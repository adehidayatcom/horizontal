# Business Contracts
## Paket Lebaran Mumpuni

Dokumen ini adalah kontrak implementasi untuk aksi bisnis utama. AI coding agent wajib mengikuti
dokumen ini sebelum membuat RPC, service, form, atau halaman transaksi.

---

## 1. Aturan Umum

### 1.1 Prinsip Wajib

Semua aksi yang mengubah uang, stok, status pesanan, status periode, atau hak reseller harus
diproses lewat Supabase RPC / database function yang atomic.

Frontend hanya boleh:

- mengirim input
- menampilkan loading, success, error
- membaca hasil query/view

Frontend tidak boleh:

- menghitung validasi final
- insert langsung ke tabel transaksi kompleks
- menentukan target tagihan dari state lokal
- menampilkan error teknis database ke user

Model pesanan wajib:

- konsumen memilih paket sejak awal
- reseller membuat `pesanan_konsumen` sebagai header tagihan/cicilan per periode
- `detail_pesanan_konsumen` menjadi sumber target tagihan
- `setoran_konsumen` wajib membawa `pesanan_konsumen_id`
- target setoran selalu berasal dari total `detail_pesanan_konsumen` yang masih dihitung
- finalisasi pesanan ditandai oleh `tanggal_final`
- penyesuaian budget dilakukan lewat perubahan detail item, bukan layer program terpisah

### 1.2 Scope Wajib

Semua aksi transaksi wajib membawa:

- `periode_id`
- `created_by` dari `auth.uid()` bila aksi dari aplikasi
- `no_reseller` jika transaksi terkait reseller

Semua hitungan transaksi wajib filter:

```sql
WHERE periode_id = :periode_id
```

Semua hitungan item, komisi, poin, tabungan, stok, target, dan kebutuhan wajib exclude:

```sql
AND status_item != 'BATAL'
```

### 1.3 Format Kontrak

Setiap RPC harus jelas dalam hal:

- aktor
- input
- validasi
- database write
- output
- pesan sukses
- pesan gagal
- test case wajib

Jika ada edge case yang belum memiliki keputusan owner, implementasi wajib berhenti dan mencatat
kebutuhan keputusan di `docs/product/edge_cases.md` atau `docs/truth/03-open_questions_register.md`.

---

## 2. RPC: `buat_pesanan_konsumen`

### Aktor

Reseller aktif atau admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Harus periode AKTIF |
| `konsumen_id` | Ya | Konsumen milik reseller |
| `items` | Ya | Daftar paket, minimal 1 |
| `nominal_harian_opsional` | Tidak | Untuk watchlist, bukan validasi final |
| `catatan` | Tidak | Catatan operasional |

`items` berisi:

- `paket_id`
- `qty` default `1`
- `motif_warna` opsional
- `kelompok` opsional

### Validasi

1. Periode harus `AKTIF`.
2. Konsumen harus milik reseller login atau admin.
3. Reseller harus `AKTIF`.
4. Belum ada pesanan aktif untuk pasangan (`periode_id`, `konsumen_id`) pada MVP.
5. Minimal ada satu item paket.
6. Semua paket harus berada pada periode yang sama.
7. Total target hasil item harus > 0.

### Database Write

Insert ke `pesanan_konsumen`:

- `periode_id`
- `konsumen_id`
- `no_reseller`
- `status_pesanan = 'AKTIF'`
- `target_tagihan_snapshot`
- `nominal_harian_opsional`
- `tanggal_mulai`
- `catatan`
- `created_by`

Insert ke `detail_pesanan_konsumen` untuk setiap item.

### Output

Return row `pesanan_konsumen` terbaru dan daftar item yang dibuat.

### Pesan Sukses

`Pesanan {nama_konsumen} berhasil dibuat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Tidak ada item | `Pilih minimal satu paket` |
| Total target <= 0 | `Total target pesanan harus lebih dari Rp 0` |
| Pesanan sudah ada | `Konsumen sudah memiliki pesanan pada periode ini` |
| Konsumen bukan milik reseller | `Konsumen tidak ditemukan` |

### Test Case Wajib

- berhasil membuat pesanan dengan satu item
- berhasil membuat pesanan dengan banyak item
- gagal jika item kosong
- gagal jika paket beda periode
- gagal jika reseller belum aktif

---

## 3. RPC: `ubah_detail_pesanan_konsumen`

### Aktor

Reseller pemilik pesanan atau admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `pesanan_konsumen_id` | Ya | Header yang diubah |
| `items` | Ya | Daftar item target baru, minimal 1 yang masih dihitung |
| `alasan` | Kondisional | Wajib jika pesanan sudah punya setoran atau sudah final |

`items` berisi:

- `id` opsional untuk item lama
- `paket_id`
- `qty`
- `motif_warna` opsional
- `kelompok` opsional
- `status_item` (`AKTIF` / `TERHENTI` / `BATAL`)
- `uang_terhenti` kondisional jika `TERHENTI`

### Validasi

1. Pesanan belum `SELESAI` atau `BATAL`.
2. Minimal ada satu item yang masih dihitung (`AKTIF` atau `TERHENTI` dengan nominal > 0).
3. Semua paket harus berada pada periode yang sama dengan pesanan.
4. Jika `status_item = 'TERHENTI'`, `uang_terhenti` wajib > 0.
5. Target baru tidak boleh lebih kecil dari total setoran yang sudah masuk.
6. Jika pesanan sudah punya setoran, alasan wajib dan masuk `audit_log`.
7. Jika `tanggal_final` sudah terisi, hanya admin yang boleh mengubah detail.

### Database Write

- update / insert / tandai batal item di `detail_pesanan_konsumen`
- update `pesanan_konsumen.target_tagihan_snapshot`
- tulis `audit_log`

### Output

Return row `pesanan_konsumen` terbaru, item terbaru, dan target berjalan baru.

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Target kurang dari setoran | `Total target pesanan lebih kecil dari setoran yang sudah masuk` |
| Uang terhenti kosong | `Wajib isi nominal uang terhenti untuk item ini` |
| Item kosong | `Pesanan harus memiliki minimal satu item yang masih dihitung` |
| Pesanan sudah selesai | `Pesanan yang sudah selesai tidak bisa diubah` |

### Test Case Wajib

- berhasil menambah item baru
- berhasil mengganti item ke paket lebih murah
- berhasil menandai item `TERHENTI`
- gagal jika target baru di bawah total setoran

---

## 4. RPC: `finalisasi_pesanan_konsumen`

### Aktor

Reseller aktif atau admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `pesanan_konsumen_id` | Ya | Pesanan yang difinalkan |
| `catatan` | Tidak | Catatan finalisasi |

### Validasi

1. Pesanan harus `AKTIF` atau `PERLU_PERHATIAN`.
2. Periode pesanan harus `AKTIF`.
3. Pesanan harus punya minimal satu item yang masih dihitung.
4. Target berjalan tidak boleh lebih kecil dari total setoran yang sudah masuk.
5. Reseller hanya boleh memfinalkan pesanan miliknya sendiri.
6. Pesanan yang sudah final tidak boleh difinalkan ulang.

### Database Write

- set `pesanan_konsumen.tanggal_final`
- tulis `audit_log`

### Output

Return row `pesanan_konsumen` terbaru.

### Pesan Sukses

`Pesanan untuk {nama_konsumen} berhasil difinalkan`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Tidak ada item aktif | `Pesanan harus memiliki minimal satu item yang masih dihitung` |
| Target kurang dari setoran | `Total target pesanan lebih kecil dari setoran yang sudah masuk` |
| Sudah final | `Pesanan ini sudah difinalkan` |

### Test Case Wajib

- berhasil finalisasi pesanan valid
- gagal finalisasi jika item kosong
- gagal finalisasi jika target di bawah total setoran

---

## 5. RPC: `ubah_status_detail_pesanan_konsumen`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `detail_pesanan_id` | Ya | Item yang diubah |
| `status_item` | Ya | `AKTIF` / `TERHENTI` / `BATAL` |
| `uang_terhenti` | Kondisional | Wajib jika `TERHENTI` |
| `alasan` | Ya | Catatan admin |

### Validasi

1. Hanya admin.
2. Jika `TERHENTI`, `uang_terhenti` wajib > 0.
3. Jika item sudah dipakai pembagian selesai, perubahan harus lewat jalur koreksi yang lebih ketat.
4. Jika perubahan membuat target pesanan turun di bawah total bayar, `uang_terhenti` harus disesuaikan.

### Database Write

- update item terkait
- update `pesanan_konsumen.target_tagihan_snapshot`
- tulis `audit_log`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Bukan admin | `Anda tidak memiliki akses untuk mengubah status item pesanan` |
| TERHENTI tanpa nominal | `Wajib isi nominal uang terhenti untuk item ini` |
| Target turun di bawah bayar | `Nominal uang terhenti harus menyesuaikan total setoran yang sudah masuk` |

---

## 6. RPC: `buat_setoran_konsumen`

### Aktor

Reseller aktif.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode aktif |
| `pesanan_konsumen_id` | Ya | Header yang menerima setoran |
| `konsumen_id` | Ya | Konsumen yang membayar |
| `nominal` | Ya | Harus > 0 |
| `metode` | Ya | `CASH` / `TRANSFER` |
| `tanggal` | Ya | Default hari ini di UI |
| `keterangan` | Tidak | Catatan opsional |

### Validasi

1. Periode harus `AKTIF`.
2. Reseller login harus pemilik pesanan/konsumen.
3. `nominal > 0`.
4. `total_bayar_pesanan + nominal <= target_berjalan`.
5. Target berjalan dihitung dari total item aktif/terhenti yang masih dihitung.
6. Jika target berjalan = 0, setoran ditolak.

### Database Write

Insert ke `setoran_konsumen`:

- `periode_id`
- `pesanan_konsumen_id`
- `tanggal`
- `konsumen_id`
- `no_reseller`
- `nominal`
- `metode`
- `keterangan`
- `created_by`

### Output

Return:

- row setoran
- `sisa_bayar_konsumen`
- `status_lunas_konsumen`

### Pesan Sukses

`Setoran Rp {nominal} untuk {nama_konsumen} berhasil disimpan`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Nominal <= 0 | `Nominal setoran harus lebih dari Rp 0` |
| Melebihi target | `Setoran melebihi sisa tagihan {nama_konsumen} (sisa Rp {sisa})` |
| Sudah lunas | `{nama_konsumen} sudah lunas, tidak bisa menerima setoran` |
| Pesanan tidak aktif/tidak ada | `{nama_konsumen} belum memiliki pesanan aktif` |

### Test Case Wajib

- berhasil input setoran sebagian
- berhasil input setoran tepat lunas
- gagal input setoran lebih dari sisa
- gagal nominal 0/negatif
- gagal jika konsumen milik reseller lain

---

## 7. RPC: `buat_setoran_pusat`

### Aktor

Reseller aktif atau admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode aktif |
| `no_reseller` | Ya | Reseller penyetor |
| `tanggal` | Ya | Tanggal setor |
| `nominal` | Ya | Harus > 0 |
| `metode` | Ya | `CASH` / `TRANSFER` |
| `akun_kas_id` | Ya | Kas penerima pusat |
| `keterangan` | Tidak | Catatan opsional |

### Validasi

1. Periode harus `AKTIF`.
2. Pasangan (`periode_id`, `no_reseller`) harus ada di `reseller_periode`.
3. `nominal > 0`.
4. `total_disetor_pusat + nominal <= nilai_akhir_paket`.
5. `total_disetor_pusat + nominal <= total_dikumpulkan`.
6. Reseller hanya boleh input setoran untuk dirinya sendiri.

### Database Write

Insert ke `setoran`.

### Output

Return:

- row setoran
- `saldo_belum_disetor`
- `sisa_setor_pusat`
- `status_lunas_reseller`

---

## 8. RPC: `buat_belanja`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `tanggal` | Ya | Tanggal transaksi |
| `supplier` | Ya | Nama supplier |
| `no_bukti` | Ya | Nomor bukti transaksi |
| `akun_kas_id` | Ya | Kas sumber |
| `items` | Ya | Minimal satu item |

`items` berisi:

- `id_barang`
- `harga`
- `jumlah`

### Validasi

1. Hanya admin.
2. Periode harus `AKTIF`.
3. `items` minimal satu.
4. Semua `id_barang` harus valid.
5. `harga > 0` dan `jumlah > 0`.
6. Total belanja harus sama dengan penjumlahan detail.
7. `akun_kas_id` harus aktif dan saldo kas sumber cukup.

### Database Write

- insert `belanja`
- insert `belanja_detail`
- tulis `audit_log`

### Output

Return:

- row `belanja`
- daftar `belanja_detail`
- `saldo_kas_terbaru`

### Pesan Sukses

`Belanja berhasil dicatat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Item kosong | `Belanja harus memiliki minimal satu item` |
| Barang tidak ditemukan | `Barang belanja tidak ditemukan` |
| Saldo kas kurang | `Saldo kas tidak cukup untuk belanja ini` |
| Nominal tidak valid | `Harga dan jumlah item harus lebih dari 0` |

### Test Case Wajib

- berhasil membuat belanja dengan banyak item
- gagal jika item kosong
- gagal jika salah satu barang tidak valid
- gagal jika saldo kas kurang

---

## 9. RPC: `buat_packing`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `tanggal` | Ya | Tanggal packing |
| `paket_id` | Ya | Paket komposit |
| `jumlah_packing` | Ya | Harus > 0 |

### Validasi

1. Hanya admin.
2. Periode harus `AKTIF`.
3. Paket harus ada pada periode yang sama.
4. Paket harus `perlu_packing = TRUE`.
5. `jumlah_packing > 0`.
6. Semua bahan BOM harus cukup untuk jumlah packing yang diminta.

### Database Write

- insert `packing`
- tulis `audit_log`

### Output

Return:

- row `packing`
- `stok_paket_jadi_terbaru`

### Pesan Sukses

`Packing berhasil dicatat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Paket bukan komposit | `Paket ini tidak memerlukan proses packing` |
| Stok bahan kurang | `Stok bahan tidak cukup untuk packing ini` |
| Jumlah tidak valid | `Jumlah packing harus lebih dari 0` |

### Test Case Wajib

- berhasil packing paket komposit
- gagal jika paket tunggal
- gagal jika stok bahan kurang

---

## 10. RPC: `proses_kirim_detail_pesanan`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `detail_pesanan_konsumen_id` | Ya | Item yang dikirim |
| `tanggal` | Tidak | Default hari ini bila kosong |

### Validasi

1. Hanya admin.
2. Detail pesanan harus ada pada periode yang sama.
3. `status_item` tidak boleh `BATAL`.
4. `status_kirim` belum `SUDAH`.
5. Jika paket komposit, stok paket jadi harus cukup.
6. Jika paket tunggal, stok barang sumber harus cukup.

### Database Write

- update `detail_pesanan_konsumen.status_kirim = 'SUDAH'`
- update `detail_pesanan_konsumen.tgl_dikirim`
- tulis `audit_log`

### Output

Return row detail pesanan terbaru beserta sumber stok yang dipakai.

### Pesan Sukses

`Item pesanan berhasil dikirim`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Item tidak ditemukan | `Detail pesanan tidak ditemukan` |
| Sudah dikirim | `Item pesanan ini sudah dikirim` |
| Stok paket jadi tidak siap | `Stok paket jadi belum siap untuk pengiriman` |
| Stok barang tidak siap | `Stok barang tidak cukup untuk pengiriman` |

### Test Case Wajib

- berhasil kirim item komposit dengan stok paket jadi cukup
- berhasil kirim item tunggal dengan stok barang cukup
- gagal jika item sudah dikirim
- gagal jika stok tidak cukup

---

## 11. RPC: `buat_pembagian_paket`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `no_reseller` | Ya | Reseller penerima |
| `tanggal` | Ya | Tanggal batch |
| `catatan` | Tidak | Catatan batch |
| `items` | Ya | Minimal satu detail item |

`items` berisi:

- `detail_pesanan_konsumen_id`
- `status_item` opsional (`SIAP` / `KURANG` / `DIGANTI`)
- `catatan` opsional

### Validasi

1. Hanya admin.
2. Periode harus `AKTIF`.
3. Reseller harus valid pada periode yang sama.
4. `items` minimal satu.
5. Semua item harus milik reseller yang sama.
6. Semua item harus eligible untuk pembagian dan sudah melewati rule pengiriman.
7. Batch final tidak boleh mencampur item `SIAP` dan `KURANG`.

### Database Write

- insert `pembagian_paket`
- insert `pembagian_paket_detail`
- tulis `audit_log`

### Output

Return row `pembagian_paket` terbaru beserta detail batch.

### Pesan Sukses

`Batch pembagian reseller berhasil dibuat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Item kosong | `Pilih minimal satu item untuk pembagian` |
| Item tidak eligible | `Ada item yang belum layak masuk batch pembagian` |
| Campuran SIAP/KURANG | `Batch final tidak boleh mencampur item siap dan kurang` |

### Test Case Wajib

- berhasil membuat batch pembagian valid
- gagal jika item kosong
- gagal jika item reseller bercampur
- gagal jika batch final mencampur `SIAP` dan `KURANG`

---

## 12. RPC: `serahkan_pembagian_paket`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode transaksi |
| `pembagian_id` | Ya | Batch pembagian |
| `tanggal` | Ya | Tanggal serah |
| `catatan` | Tidak | Catatan serah terima |

### Validasi

1. Hanya admin.
2. Batch pembagian harus ada.
3. Batch belum `DISERAHKAN` atau `SELESAI`.
4. Item yang masih `SIAP` boleh ditandai `DISERAHKAN`.
5. Status `SELESAI` tetap aksi manual terpisah dan tidak diisi otomatis.

### Database Write

- update `pembagian_paket_detail` yang relevan ke `DISERAHKAN`
- update `pembagian_paket.status = 'DISERAHKAN'`
- tulis `audit_log`

### Output

Return summary batch pembagian terbaru.

### Pesan Sukses

`Pembagian paket berhasil diserahkan`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Batch tidak ditemukan | `Batch pembagian tidak ditemukan` |
| Sudah diserahkan | `Batch pembagian ini sudah diserahkan` |

### Test Case Wajib

- berhasil menyerahkan batch valid
- gagal jika batch sudah diserahkan
- status summary ikut berubah setelah serah terima

---

## 13. RPC: `buat_kas_masuk`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `tanggal` | Ya | Tanggal kas masuk |
| `akun_kas_id` | Ya | Kas penerima |
| `sumber` | Ya | `MODAL_OWNER` / `PENYESUAIAN` / `LAINNYA` |
| `nominal` | Ya | Harus > 0 |
| `keterangan` | Tidak | Catatan tambahan |

### Validasi

1. Hanya admin.
2. Periode harus `AKTIF`.
3. `akun_kas_id` harus valid dan aktif.
4. `nominal > 0`.
5. `sumber` harus sesuai enum yang diizinkan.

### Database Write

- insert `kas_masuk`
- tulis `audit_log`

### Output

Return row `kas_masuk` terbaru dan `saldo_kas_terbaru`.

### Pesan Sukses

`Kas masuk berhasil dicatat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Sumber tidak valid | `Sumber kas masuk tidak valid` |
| Nominal tidak valid | `Nominal kas masuk harus lebih dari Rp 0` |
| Akun kas tidak ditemukan | `Akun kas tidak ditemukan` |

### Test Case Wajib

- berhasil mencatat kas masuk
- gagal jika nominal 0/negatif
- gagal jika akun kas tidak valid

---

## 14. RPC: `buat_mutasi_kas`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `tanggal` | Ya | Tanggal mutasi |
| `dari_akun_id` | Ya | Akun sumber |
| `ke_akun_id` | Ya | Akun tujuan |
| `jumlah` | Ya | Harus > 0 |
| `keterangan` | Tidak | Catatan tambahan |

### Validasi

1. Hanya admin.
2. Periode harus `AKTIF`.
3. `dari_akun_id` dan `ke_akun_id` harus valid dan berbeda.
4. `jumlah > 0`.
5. Saldo akun sumber harus cukup.
6. Mutasi kas lintas periode dilarang.

### Database Write

- insert `mutasi_kas`
- tulis `audit_log`

### Output

Return row `mutasi_kas` terbaru beserta saldo akun asal/tujuan terbaru.

### Pesan Sukses

`Mutasi kas berhasil dicatat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Akun sama | `Akun asal dan tujuan tidak boleh sama` |
| Saldo kurang | `Saldo akun sumber tidak cukup untuk mutasi ini` |
| Jumlah tidak valid | `Jumlah mutasi harus lebih dari Rp 0` |

### Test Case Wajib

- berhasil mutasi kas valid
- gagal jika akun asal dan tujuan sama
- gagal jika saldo sumber kurang

---

## 15. RPC: `buat_pencairan`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` |
| `tanggal` | Ya | Tanggal pencairan |
| `no_reseller` | Ya | Reseller penerima |
| `jenis` | Ya | `TABUNGAN` atau `KOMISI` |
| `nominal` | Ya | Harus > 0 |
| `metode` | Ya | `CASH` / `TRANSFER` |
| `akun_kas_id` | Ya | Kas sumber |

### Validasi

1. Hanya admin.
2. Periode harus `AKTIF`.
3. Reseller harus `LUNAS`.
4. `jenis` hanya satu per transaksi.
5. `nominal > 0`.
6. `nominal <= sisa_tabungan` atau `sisa_komisi` sesuai jenis.
7. Saldo akun kas sumber harus cukup.

### Database Write

- insert `pencairan`
- tulis `audit_log`

### Output

Return:

- row `pencairan`
- `saldo_kas_terbaru`
- `sisa_hak_reseller_terbaru`

### Pesan Sukses

`Pencairan reseller berhasil dicatat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Reseller belum lunas | `Pencairan hanya bisa dilakukan setelah reseller lunas` |
| Sisa hak tidak cukup | `Nominal pencairan melebihi sisa hak reseller` |
| Saldo kas kurang | `Saldo kas tidak cukup untuk pencairan ini` |

### Test Case Wajib

- berhasil mencairkan tabungan reseller lunas
- berhasil mencairkan komisi reseller lunas
- gagal jika reseller belum lunas
- gagal jika nominal melebihi sisa hak
- gagal jika saldo kas kurang

---

## 16. RPC: `aktifkan_periode`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `PERSIAPAN` yang akan diaktifkan |

### Validasi

1. Hanya admin.
2. Periode harus ada dan berstatus `PERSIAPAN`.
3. Tidak boleh ada periode lain yang masih `AKTIF`.
4. Checklist readiness wajib lengkap:
   - akun kas tersedia
   - barang periode tersedia
   - paket tersedia
   - semua paket punya BOM
   - komisi config lengkap
5. Jika periode sebelumnya belum `SELESAI`, aktivasi ditolak.

### Database Write

- update `periode.status = 'AKTIF'`
- tulis `audit_log`

### Output

Return row `periode` terbaru dan ringkasan checklist aktivasi.

### Pesan Sukses

`Periode berhasil diaktifkan`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Periode tidak siap | `Setup periode belum lengkap` |
| Ada periode aktif lain | `Masih ada periode lain yang aktif` |
| Transisi status tidak valid | `Periode ini tidak bisa diaktifkan` |

### Test Case Wajib

- berhasil aktivasi periode siap
- gagal jika checklist belum lengkap
- gagal jika masih ada periode aktif lain

---

## 17. RPC: `tutup_periode`

### Aktor

Admin.

### Input

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode `AKTIF` yang akan ditutup |
| `catatan` | Tidak | Catatan penutupan |

### Validasi

1. Hanya admin.
2. Periode harus berstatus `AKTIF`.
3. Semua reseller harus `LUNAS`.
4. Pencairan belum selesai harus nol.
5. Pembagian paket harus selesai atau selisihnya sudah tercatat resmi.
6. Tidak boleh ada transaksi operasional menggantung yang masih memblok closing.

### Database Write

- update `periode.status = 'SELESAI'`
- update `pesanan_konsumen.status_pesanan = 'SELESAI'` untuk pesanan valid pada periode itu
- tulis `audit_log`

### Output

Return row `periode` terbaru dan ringkasan blocker = kosong.

### Pesan Sukses

`Periode berhasil ditutup`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Masih ada reseller belum lunas | `Masih ada reseller yang belum lunas` |
| Masih ada pencairan/transaksi terbuka | `Masih ada transaksi operasional yang belum selesai` |
| Status periode tidak valid | `Periode ini belum bisa ditutup` |

### Test Case Wajib

- gagal tutup periode jika ada reseller belum lunas
- gagal tutup periode jika ada transaksi operasional belum selesai
- berhasil tutup periode jika semua blocker bersih

---

## 18. RPC Koreksi: `koreksi_*`

### Aktor

Admin.

### Scope

Koreksi berlaku untuk transaksi operasional yang salah input, minimal:

- `koreksi_setoran_konsumen`
- `koreksi_setoran`
- `koreksi_belanja`
- `koreksi_packing`
- `koreksi_pembagian_paket`
- `koreksi_kas_masuk`
- `koreksi_mutasi_kas`
- `koreksi_pencairan`

### Input Minimum

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode transaksi asal |
| `target_tabel` | Ya | Entitas transaksi yang dikoreksi |
| `target_id` | Ya | ID transaksi asal |
| `alasan` | Ya | Alasan koreksi wajib literal |
| `aksi_koreksi` | Ya | `REVERSAL`, `ADJUSTMENT`, atau aksi koreksi resmi lain |
| `payload_koreksi` | Ya | Detail perubahan yang dibutuhkan boundary koreksi |

### Validasi

1. Hanya admin.
2. Alasan koreksi wajib terisi.
3. Transaksi asli tidak boleh dihapus diam-diam.
4. Koreksi tidak boleh membuat saldo, stok, target, atau status turunan menjadi inkonsisten.
5. Jika transaksi asal sudah berada pada periode `SELESAI`, hanya koreksi yang memang diizinkan oleh rule historis yang boleh berjalan.

### Database Write

- tulis row koreksi ke `koreksi_transaksi`
- tulis `audit_log`
- buat reversal/adjustment sesuai boundary transaksi asal

### Output

Return:

- row `koreksi_transaksi`
- referensi transaksi asal
- ringkasan nilai sesudah koreksi

### Pesan Sukses

`Koreksi transaksi berhasil dicatat`

### Pesan Gagal

| Kondisi | Pesan |
|---|---|
| Alasan kosong | `Alasan koreksi wajib diisi` |
| Target transaksi tidak ditemukan | `Transaksi yang akan dikoreksi tidak ditemukan` |
| Koreksi membuat data tidak konsisten | `Koreksi ini membuat data turunan tidak konsisten` |

### Test Case Wajib

- berhasil mencatat koreksi dengan alasan valid
- gagal jika alasan kosong
- gagal jika target transaksi tidak ditemukan
- gagal jika koreksi membuat saldo/stok/target tidak konsisten

---

## 19. Prinsip Error dan Audit

- semua perubahan item setelah ada setoran wajib alasan
- semua perubahan item setelah finalisasi wajib alasan dan admin-only kecuali diputuskan lain
- transaksi asli tidak boleh dihapus untuk koreksi
- semua aksi sensitif wajib masuk `audit_log`

---

## 20. Istilah Final

| Istilah Lama | Istilah Final |
|---|---|
| `program_konsumen` | `pesanan_konsumen` |
| `program_pilihan_paket` | dihapus |
| `order_paket` | `detail_pesanan_konsumen` |
| kunci paket program | finalisasi pesanan konsumen |
| target program | target tagihan pesanan |
