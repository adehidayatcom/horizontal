# Navigation and Period Setup UI
## Paket Lebaran Mumpuni

Dokumen ini adalah kontrak UI/UX untuk navigasi, route, dan setup periode.

Tujuannya:

- periode diperlakukan sebagai pusat kendali operasional;
- admin tidak salah masuk ke transaksi sebelum periode siap;
- AI Coder tidak menafsir sendiri link/menu dan state halaman;
- setiap route jelas boleh dibuka pada status periode apa;
- setup awal periode dibuat sebagai wizard/checklist, bukan CRUD terpisah yang membingungkan.

---

## 1. Prinsip Utama

Periode adalah gerbang operasional.

```txt
PERSIAPAN -> AKTIF -> SELESAI
```

Makna UI:

- `PERSIAPAN`: admin menyiapkan semua data sebelum operasional dimulai.
- `AKTIF`: reseller dan admin menjalankan transaksi periode berjalan.
- `SELESAI`: data menjadi arsip/laporan, transaksi baru ditutup.

Dashboard, menu, tombol, dan form harus selalu sadar status periode.

AI Coder dilarang membuat halaman transaksi yang tetap aktif ketika:

- belum ada periode aktif;
- periode masih `PERSIAPAN`;
- periode sudah `SELESAI`;
- data prasyarat periode belum lengkap.

---

## 2. Periode Aktif vs Periode Dipilih

### Periode Aktif Sistem

Satu periode dengan:

```txt
periode.status = 'AKTIF'
```

Dipakai untuk transaksi periode berjalan: pesanan konsumen, setoran, finalisasi pesanan, belanja,
packing, pembagian, kas, dan pencairan.

### Periode Dipilih / Periode Laporan

Filter tampilan untuk membaca data periode tertentu.

Label UI yang benar:

- `Periode aktif`
- `Periode laporan`
- `Lihat periode`
- `Periode persiapan`

Label UI yang dilarang:

- `Ganti periode aktif` untuk filter laporan;
- `Pilih periode aktif` untuk dropdown laporan;
- `Periode aktif belum dipilih` ketika sebenarnya ada periode `AKTIF`.

---

## 3. State Global Aplikasi Admin

Admin shell harus menentukan state periode sebelum render menu utama.

### Tidak Ada Periode

- tampilkan empty state `Belum ada periode`;
- aksi utama `Buat Periode`;
- menu transaksi disabled/hidden;
- dashboard operasional tidak ditampilkan.

### Ada Periode PERSIAPAN, Belum Ada AKTIF

- tampilkan `Periode sedang dipersiapkan`;
- aksi utama `Lanjutkan Setup Periode`;
- menu setup aktif;
- menu transaksi disabled;
- dashboard operasional diganti checklist persiapan.

### Ada Periode AKTIF

- dashboard operasional tersedia;
- transaksi tersedia sesuai role;
- setup master periodik tetap bisa dibuka dengan warning dampak;
- header menampilkan badge `Periode Aktif`.

### Melihat Periode SELESAI

- tampilkan badge `Mode laporan historis`;
- tombol transaksi disembunyikan atau disabled;
- koreksi hanya lewat menu koreksi khusus admin jika diizinkan;
- semua halaman bersifat read-only kecuali koreksi/audit.

---

## 4. Admin Navigation

Sidebar admin wajib memakai grup dan route berikut.

### Ringkasan

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Dashboard | `/admin/dashboard` | AKTIF / laporan historis | Jika belum aktif, arahkan ke setup periode |

### Setup Periode

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Daftar Periode | `/admin/periode` | Semua | CRUD periode terbatas |
| Wizard Persiapan | `/admin/periode/:periodeId/setup` | PERSIAPAN | Step-by-step setup |
| Checklist Aktivasi | `/admin/periode/:periodeId/checklist` | PERSIAPAN | Review prasyarat |
| Review Penutupan | `/admin/periode/:periodeId/closing` | AKTIF | Checklist sebelum SELESAI |

### Data Master

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Akun Kas | `/admin/master/akun-kas` | Semua | Lintas periode |
| Barang | `/admin/master/barang` | Semua | Identitas barang |
| Barang Periode | `/admin/master/barang-periode` | PERSIAPAN / AKTIF | Periodik |
| Komisi | `/admin/master/komisi` | PERSIAPAN / AKTIF | Periodik |
| Paket & BOM | `/admin/master/paket` | PERSIAPAN / AKTIF | Periodik |

### Relasi

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Reseller | `/admin/reseller` | Semua | Approval dan status |
| Reseller Periode | `/admin/reseller-periode` | PERSIAPAN / AKTIF | Keikutsertaan periode |
| Konsumen | `/admin/konsumen` | AKTIF / laporan historis | Admin view |

### Pesanan

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Pesanan Konsumen | `/admin/pesanan` | AKTIF / laporan historis | Header tagihan/cicilan |
| Pesanan Perlu Perhatian | `/admin/pesanan/perlu-perhatian` | AKTIF | Watchlist |
| Detail Pesanan | `/admin/pesanan/:pesananId` | AKTIF / laporan historis | Header + item pesanan |

### Setoran

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Setoran Konsumen | `/admin/setoran-konsumen` | AKTIF / laporan historis | Read/admin koreksi |
| Setoran Pusat | `/admin/setoran-pusat` | AKTIF / laporan historis | Read/admin koreksi |

### Gudang

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Belanja / PO | `/admin/gudang/belanja` | AKTIF | RPC `buat_belanja` |
| Packing | `/admin/gudang/packing` | AKTIF | RPC `buat_packing` |
| Pembagian Paket | `/admin/gudang/pembagian` | AKTIF / laporan historis | Serah terima ke reseller |

### Keuangan

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Kas Masuk | `/admin/keuangan/kas-masuk` | AKTIF | Modal/penyesuaian |
| Mutasi Kas | `/admin/keuangan/mutasi-kas` | AKTIF | Antar akun kas |
| Pencairan | `/admin/keuangan/pencairan` | AKTIF | Setelah reseller lunas |

### Koreksi & Audit

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Koreksi Transaksi | `/admin/koreksi` | AKTIF / SELESAI khusus | Admin only |
| Audit Log | `/admin/laporan/audit` | Semua | Admin only, read-only; satu route dengan tab `Audit Log` dan `Riwayat Koreksi` |

### Laporan

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Rekap Reseller | `/admin/laporan/rekap-reseller` | Semua dengan periode | Read-only |
| Laporan Stok | `/admin/laporan/stok` | Semua dengan periode | Read-only |
| Laporan Pembagian | `/admin/laporan/pembagian` | Semua dengan periode | Read-only |
| Laporan Keuangan | `/admin/laporan/keuangan` | Semua dengan periode | Read-only |

---

## 5. Reseller Navigation

Reseller tidak melihat setup periode.

Bottom navigation:

| Label | Route | Status Periode | Catatan |
|---|---|---|---|
| Beranda | `/reseller` | AKTIF | Ringkasan |
| Konsumen | `/reseller/konsumen` | AKTIF | Konsumen dan pesanan |
| Setor | `/reseller/setor` | AKTIF | Setoran konsumen |
| Pesanan | `/reseller/pesanan` | AKTIF | Pesanan dan finalisasi |
| Akun | `/reseller/akun` | AKTIF | Profil, setor pusat, logout |

Jika reseller `PENDING`:

- tampilkan halaman `Pendaftaran sedang ditinjau`;
- semua tab transaksi disembunyikan.

Jika tidak ada periode `AKTIF`:

- tampilkan halaman `Belum ada periode aktif`;
- tombol transaksi disabled;
- sediakan tombol refresh.

---

## 6. Wizard Persiapan Periode

Route:

```txt
/admin/periode/:periodeId/setup
```

Wizard wajib berbentuk stepper/checklist.

| Step | Nama | Output Wajib |
|---|---|---|
| 1 | Buat / Review Periode | Periode `PERSIAPAN` dengan tanggal valid |
| 2 | Akun Kas | Minimal satu akun kas tersedia |
| 3 | Barang Periode | Barang, harga referensi, budget, stok awal |
| 4 | Paket & BOM | Paket periode dan semua detail paket valid |
| 5 | Komisi | Config per kategori paket |
| 6 | Reseller Periode | Reseller yang ikut periode |
| 7 | Review Checklist | Semua blocker dan warning terlihat |
| 8 | Aktifkan Periode | RPC `aktifkan_periode` |

Tombol `Aktifkan Periode` disabled jika:

- checklist wajib belum lengkap;
- ada periode lain `AKTIF`;
- periode sebelumnya belum `SELESAI`;
- tanggal tidak valid.

---

## 7. Checklist Aktivasi

| Item | Wajib | Blocker Jika |
|---|---:|---|
| Periode valid | Ya | Tanggal overlap/tidak urut |
| Akun kas tersedia | Ya | Tidak ada akun kas |
| Barang periode tersedia | Ya | Tidak ada barang periode |
| Paket tersedia | Ya | Tidak ada paket |
| Semua paket punya BOM | Ya | Ada paket tanpa detail |
| Komisi config lengkap | Ya | Ada kategori paket tanpa komisi |
| Reseller periode tersedia | Sebaiknya | Warning jika kosong |
| Tidak ada periode aktif lain | Ya | Ada periode AKTIF |

Status checklist:

| Status | Arti |
|---|---|
| `Belum Mulai` | Data belum ada |
| `Belum Lengkap` | Data ada tapi belum memenuhi syarat |
| `Lengkap` | Syarat terpenuhi |
| `Ada Peringatan` | Boleh lanjut, tapi perlu perhatian admin |

---

## 8. Review Penutupan Periode

Route:

```txt
/admin/periode/:periodeId/closing
```

Checklist penutupan:

| Item | Wajib |
|---|---:|
| Semua reseller lunas | Ya |
| Semua pencairan selesai | Ya |
| Pembagian paket selesai atau selisih tercatat | Ya |
| Koreksi besar sudah selesai | Ya |
| Stok akhir siap carry-over | Ya |
| Tidak ada pesanan aktif menggantung | Ya |

Tombol:

```txt
Selesaikan Periode
```

RPC:

```txt
tutup_periode
```

---

## 9. Route Guard dan Menu State

| Kondisi | Perilaku |
|---|---|
| Anonymous | Redirect login |
| Role salah | Redirect sesuai role |
| Reseller PENDING | Halaman menunggu approval |
| Tidak ada periode aktif untuk transaksi | Empty state, transaksi disabled |
| Route butuh PERSIAPAN tapi periode bukan PERSIAPAN | Tampilkan read-only atau redirect checklist |
| Route butuh AKTIF tapi tidak ada AKTIF | Redirect setup/empty |
| Periode SELESAI | Read-only kecuali koreksi khusus |

Menu item boleh:

- disabled dengan tooltip alasan;
- disembunyikan jika role tidak punya akses;
- tetap tampil read-only untuk laporan.

Jangan menampilkan tombol aksi aktif jika RPC akan menolak karena status periode.

---

## 10. Empty, Loading, Error

### Empty Tidak Ada Periode

Judul:

```txt
Belum ada periode
```

Aksi:

```txt
Buat Periode
```

### Empty Periode Persiapan

Judul:

```txt
Periode sedang dipersiapkan
```

Aksi:

```txt
Lanjutkan Setup Periode
```

### Empty Belum Ada Periode Aktif

Judul:

```txt
Belum ada periode aktif
```

Admin aksi:

```txt
Buka Setup Periode
```

Reseller:

```txt
Transaksi belum tersedia
```

### Error

Pesan harus Bahasa Indonesia:

- `Data periode belum bisa dimuat. Coba lagi.`
- `Setup periode belum lengkap.`
- `Periode lain masih aktif. Selesaikan terlebih dahulu.`

---

## 11. Aksi Cepat Berdasarkan Status

### PERSIAPAN

Aksi utama:

- `Lanjutkan Setup`
- `Review Checklist`
- `Aktifkan Periode`

Tidak boleh tampil:

- `Input Setoran`
- `Buat Pesanan Konsumen`
- `Belanja`
- `Packing`
- `Pembagian`
- `Pencairan`

### AKTIF

Aksi utama:

- `Buat Pesanan Konsumen`
- `Input Setoran`
- `Finalisasi Pesanan`
- `Setor Pusat`
- `Belanja`
- `Packing`
- `Pembagian Paket`

Setup master periodik boleh tampil dengan warning dampak.

### SELESAI

Aksi utama:

- `Lihat Laporan`
- `Lihat Audit`
- `Koreksi Khusus` jika diizinkan.

Tidak boleh tampil:

- aksi transaksi baru.

---

## 12. Acceptance Criteria

Navigasi dan setup periode dianggap siap jika:

- semua route admin dan reseller mengikuti tabel di dokumen ini;
- admin shell selalu mengetahui state periode sebelum render menu;
- setup periode memakai wizard/checklist;
- transaksi disabled sebelum periode aktif;
- reseller tidak melihat setup periode;
- periode selesai tampil sebagai mode laporan/read-only;
- tombol aktivasi periode disabled sampai checklist wajib lengkap;
- semua label periode memakai Bahasa Indonesia yang tidak ambigu;
- tidak ada route transaksi yang bisa submit saat status periode tidak valid;
- UI menampilkan alasan disabled/empty state, bukan halaman kosong.
