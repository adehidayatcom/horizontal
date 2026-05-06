# Pesanan Workflow
## Paket Lebaran Mumpuni

Dokumen ini adalah kontrak utama untuk model bisnis final:

```txt
konsumen memilih paket
-> reseller membuat pesanan konsumen
-> reseller mencatat cicilan/setoran
-> detail pesanan dapat disesuaikan jika kebutuhan atau budget berubah
-> pesanan difinalkan untuk operasional
-> gudang, pengiriman, dan pembagian berjalan dari detail pesanan final
```

Nama file tetap `program_workflow.md` untuk menjaga kompatibilitas referensi dokumentasi lama.
Isi dokumen ini sekarang menjadi source of truth untuk workflow `pesanan_konsumen`.

---

## 1. Prinsip Utama

Alur utama sistem:

```txt
Konsumen
-> Pesanan Konsumen
-> Detail Pesanan Konsumen
-> Setoran Konsumen
-> Finalisasi Pesanan Konsumen
-> Gudang / PO
-> Packing
-> Pembagian Paket ke Reseller
-> Pencairan / Penutupan Periode
```

Konsekuensi penting:

- `konsumen` adalah master orang milik reseller.
- `pesanan_konsumen` adalah header tagihan/cicilan konsumen dalam satu periode.
- `detail_pesanan_konsumen` adalah daftar paket yang dipesan di dalam `pesanan_konsumen`.
- `setoran_konsumen` masuk ke `pesanan_konsumen`, bukan ke item detail.
- target tagihan konsumen berasal dari total `detail_pesanan_konsumen` yang masih dihitung.
- jika budget berubah, detail pesanan boleh diubah sebelum penutupan periode sesuai aturan audit.
- item yang tidak lagi masuk budget dapat ditandai `TERHENTI`.
- finalisasi pesanan ditandai oleh `tanggal_final`, bukan status `DIKUNCI`.

---

## 2. Definisi Entitas Utama

### 2.1 `konsumen`

`konsumen` adalah master identitas orang.

Ia menyimpan:

- reseller pemilik;
- nama;
- telepon;
- alamat;
- identitas dasar lain yang memang disetujui produk.

`konsumen` tidak menyimpan angka tagihan hidup.

### 2.2 `pesanan_konsumen`

`pesanan_konsumen` adalah header tagihan/cicilan konsumen dalam satu periode.

Ia menyimpan:

- periode;
- konsumen;
- reseller pemilik;
- status pesanan;
- `target_tagihan_snapshot`;
- `nominal_harian_opsional`;
- `tanggal_mulai`;
- `tanggal_final`;
- catatan operasional.

Aturan MVP:

```txt
Satu konsumen hanya boleh memiliki satu pesanan_konsumen aktif dalam satu periode.
```

Constraint yang disarankan:

```sql
UNIQUE (periode_id, konsumen_id)
```

### 2.3 `detail_pesanan_konsumen`

`detail_pesanan_konsumen` adalah item paket di dalam satu `pesanan_konsumen`.

Ia menyimpan:

- referensi ke `pesanan_konsumen`;
- paket yang dipilih;
- qty;
- snapshot harga saat item dibuat atau diubah;
- status item;
- `uang_terhenti` bila item tidak tuntas normal;
- atribut operasional seperti motif/warna/kelompok bila relevan.

`detail_pesanan_konsumen` adalah sumber target tagihan sebelum dan sesudah finalisasi.

### 2.4 `setoran_konsumen`

`setoran_konsumen` adalah cicilan pembayaran konsumen ke reseller.

Ia wajib mengacu ke:

- `pesanan_konsumen`
- `konsumen`
- `no_reseller`
- `periode_id`

---

## 3. Status yang Dipakai

### 3.1 Status `pesanan_konsumen`

| Status | Arti | Dampak |
|---|---|---|
| `AKTIF` | Pesanan berjalan normal | Boleh menerima setoran dan perubahan item |
| `PERLU_PERHATIAN` | Perlu follow-up karena ritme setor atau kondisi budget | Boleh menerima setoran, masuk watchlist |
| `SELESAI` | Pesanan dibekukan saat closing periode selesai | Read-only historis |
| `BATAL` | Pesanan dibatalkan sebelum ada setoran | Tidak dihitung dalam target |

Transisi normal:

```txt
AKTIF -> PERLU_PERHATIAN -> AKTIF
AKTIF -> BATAL
AKTIF / PERLU_PERHATIAN -> SELESAI
```

Larangan:

- pesanan dengan setoran tidak boleh langsung `BATAL`;
- pesanan `SELESAI` tidak boleh menerima transaksi operasional baru;
- perubahan ke `SELESAI` terjadi saat closing periode resmi, bukan otomatis saat lunas.

### 3.2 Status `detail_pesanan_konsumen`

| Status | Arti | Dampak ke target |
|---|---|---|
| `AKTIF` | Item masih dihitung normal | Nilai paket penuh dihitung |
| `TERHENTI` | Item tidak tuntas normal, tetapi sebagian uang tetap diakui | Nilai item dihitung dari `uang_terhenti` |
| `BATAL` | Item dibatalkan dan tidak lagi dihitung | Tidak ikut target |

### 3.3 Finalisasi

Pesanan dianggap final secara operasional jika:

```txt
pesanan_konsumen.tanggal_final IS NOT NULL
```

Efek finalisasi:

- pesanan siap dipakai modul gudang/pengiriman/pembagian;
- perubahan item setelah finalisasi hanya boleh lewat jalur admin yang diaudit;
- frontend harus membedakan pesanan `belum final` vs `sudah final`.

---

## 4. Target Tagihan Konsumen

Sumber target harus tunggal dan mudah diaudit.

Formula:

```txt
target_berjalan =
  SUM(detail_pesanan_konsumen yang masih dihitung)
```

Aturan per item:

```txt
AKTIF     -> nilai item = qty x harga_snapshot atau nilai paket snapshot
TERHENTI  -> nilai item = uang_terhenti
BATAL     -> nilai item = 0
```

Snapshot header:

```txt
pesanan_konsumen.target_tagihan_snapshot
```

Kolom ini menyimpan snapshot target terkini agar:

- validasi setoran tidak tergantung hitung liar di frontend;
- ringkasan cepat tetap konsisten;
- audit perubahan target mudah dibaca.

Validasi setoran konsumen:

```txt
total_setoran_pesanan + nominal_baru <= target_berjalan
```

Jika target berjalan = 0, setoran ditolak.

---

## 5. Buat Pesanan Konsumen

Pembuatan pesanan berarti:

1. pilih atau buat `konsumen`
2. buat `pesanan_konsumen`
3. isi minimal satu `detail_pesanan_konsumen`
4. hitung `target_tagihan_snapshot`

Boundary utama:

```txt
buat_pesanan_konsumen
```

Input minimal:

- `periode_id`
- `konsumen_id`
- daftar item paket
- `nominal_harian_opsional` opsional
- catatan opsional

Validasi:

1. periode harus `AKTIF`;
2. konsumen harus milik reseller yang benar;
3. reseller harus `AKTIF`;
4. belum ada pesanan aktif untuk pasangan (`periode_id`, `konsumen_id`) pada MVP;
5. minimal ada satu item paket;
6. semua paket harus berada pada periode yang sama;
7. total target hasil item harus > 0.

---

## 6. Perubahan Detail Pesanan

Sebelum `SELESAI`, detail pesanan masih boleh berubah sesuai kebutuhan lapangan.

Kasus umum:

- konsumen menambah paket;
- konsumen mengganti paket ke yang lebih murah;
- konsumen mengurangi item;
- sebagian item tidak masuk budget dan harus `TERHENTI`.

Boundary utama:

```txt
ubah_detail_pesanan_konsumen
```

Aturan:

- perubahan item harus memperbarui `target_tagihan_snapshot`;
- perubahan tidak boleh membuat target berjalan lebih kecil dari total setoran yang sudah masuk;
- jika sudah ada setoran, alasan perubahan wajib disimpan;
- perubahan setelah `tanggal_final` hanya boleh oleh admin dan wajib diaudit.

Contoh logika penyesuaian budget:

```txt
total setoran = 450.000
item A aktif  = 300.000
item B aktif  = 250.000
=> total target 550.000

jika konsumen tidak sanggup:
- item A tetap AKTIF 300.000
- item B diubah TERHENTI 150.000
=> target baru 450.000
```

---

## 7. Finalisasi Pesanan

Finalisasi adalah titik ketika pesanan dianggap siap menjadi dasar operasional gudang.

Boundary utama:

```txt
finalisasi_pesanan_konsumen
```

Input minimal:

- `pesanan_konsumen_id`
- catatan finalisasi opsional

Validasi:

1. periode harus `AKTIF`;
2. pesanan harus milik reseller yang benar atau dikerjakan admin;
3. pesanan harus berstatus `AKTIF` atau `PERLU_PERHATIAN`;
4. pesanan harus punya minimal satu `detail_pesanan_konsumen` yang masih dihitung;
5. target berjalan tidak boleh lebih kecil dari total setoran yang sudah masuk.

Write:

- isi `pesanan_konsumen.tanggal_final`;
- tulis `audit_log`;
- pesanan siap dibaca modul gudang/pengiriman/pembagian.

Pesan gagal utama:

| Kondisi | Pesan |
|---|---|
| Tidak ada item aktif | `Pesanan harus memiliki minimal satu item yang masih dihitung` |
| Target di bawah setoran | `Total target pesanan lebih kecil dari setoran yang sudah masuk` |
| Pesanan sudah final | `Pesanan ini sudah difinalkan` |

---

## 8. Pesanan Perlu Perhatian

Di lapangan, konsumen yang lama tidak setor atau tertinggal jauh dari target ditandai berdasarkan
informasi reseller dan penilaian admin. Sistem tidak boleh otomatis mengubah pesanan menjadi
`BATAL` atau `SELESAI`.

Sistem hanya boleh memberi watchlist kandidat.

Read model yang disarankan:

```txt
v_pesanan_perlu_perhatian
```

Kolom bantu:

- `tanggal_setoran_terakhir`
- `hari_tanpa_setoran`
- `nominal_harian_opsional`
- `target_seharusnya`
- `total_setoran`
- `selisih_dari_target`

Keputusan status tetap manual melalui boundary resmi.

---

## 9. Dampak ke Modul Gudang dan Pembagian

Gudang dan pembagian tidak lagi bergantung pada `order_paket` sebagai entitas utama.

Sekarang mereka membaca:

- `pesanan_konsumen` yang sudah final
- `detail_pesanan_konsumen` yang masih dihitung

Aturan:

- item `AKTIF` masuk kebutuhan normal;
- item `TERHENTI` dihitung sesuai `uang_terhenti` atau rule operasional yang disetujui;
- item `BATAL` tidak ikut kebutuhan;
- semua perubahan setelah finalisasi wajib diaudit.

---

## 10. Koreksi dan Audit

Koreksi transaksi uang atau perubahan item tidak boleh dilakukan dengan edit bebas pada row historis.

Prinsip:

- transaksi asli tetap menjadi riwayat;
- koreksi dibuat lewat boundary resmi;
- alasan wajib;
- semua koreksi masuk `audit_log`;
- read model harus memperhitungkan transaksi koreksi.

Koreksi wajib tersedia untuk:

- setoran konsumen;
- setoran pusat;
- belanja;
- packing;
- pencairan;
- mutasi kas;
- pembagian paket;
- perubahan detail pesanan setelah finalisasi.

---

## 11. Dampak ke Istilah Lama

| Istilah Lama | Istilah Final |
|---|---|
| `program_konsumen` | `pesanan_konsumen` |
| `program_pilihan_paket` | dihapus |
| `order_paket` | `detail_pesanan_konsumen` |
| kunci paket program | finalisasi pesanan konsumen |
| target program | target tagihan pesanan |

Dokumen lain wajib mengikuti dokumen ini jika membahas:

- target konsumen;
- setoran konsumen;
- detail paket pesanan;
- finalisasi pesanan;
- perubahan budget;
- `TERHENTI`;
- audit log.
