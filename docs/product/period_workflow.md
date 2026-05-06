# Period Workflow
## Paket Lebaran Mumpuni

Dokumen ini menjadi kontrak konsep untuk semua fitur yang terikat periode.
Tujuannya agar frontend, database, read model, dan RPC memakai pemahaman yang sama tentang periode aktif, filter periode, persiapan periode, dan penutupan periode.

Untuk navigasi, route, menu, dan wizard setup periode, lihat `docs/navigation_and_period_setup_ui.md`.

---

## 1. Prinsip Utama

Sistem ini memakai model tabungan target paket per musim/periode. Periode bukan hanya filter tampilan, tetapi konteks kerja yang mengikat master, transaksi, stok, komisi, saldo, dan laporan.

Semua data yang memengaruhi perhitungan periode wajib berada dalam scope `periode_id`.

Contoh data yang wajib periodik:
- `barang_periode`
- `komisi_config`
- `paket`
- `detail_paket` melalui `paket`
- `reseller_periode`
- `pesanan_konsumen`
- `detail_pesanan_konsumen`
- `setoran_konsumen`
- `setoran`
- `belanja`
- `packing`
- `pencairan`
- `mutasi_kas`
- `kas_masuk`

Data identitas yang lintas periode:
- `barang`
- `akun_kas`
- `reseller`
- `profile`
- `konsumen`

---

## 2. Referensi Praktik ERP

Konsep ini mengikuti pola umum ERP/accounting:

1. Microsoft Dynamics 365 memakai fiscal calendar, fiscal year, dan period sebagai framework aktivitas finansial. Periode dipakai lintas ledger, budget, dan fixed asset.
   Referensi: https://learn.microsoft.com/en-us/dynamics365/finance/budgeting/fiscal-calendars-fiscal-years-periods

2. NetSuite memakai Period Close Checklist. Task penutupan periode berjalan berurutan, sebagian task terkunci sampai prasyarat selesai, dan periode tidak bisa ditutup jika periode sebelumnya belum selesai.
   Referensi: https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_N1455781.html

3. Odoo memakai lock date/hard lock untuk melindungi periode lama dari perubahan setelah tutup buku.
   Referensi: https://www.odoo.com/documentation/19.0/applications/finance/accounting/reporting/year_end.html

4. ERPNext memakai freeze accounting entries untuk mencegah input atau perubahan transaksi pada tanggal sebelum batas tertentu.
   Referensi: https://docs.frappe.io/erpnext/freeze-accounting-entries

Adaptasi untuk proyek ini:
- periode harus urut secara kronologis;
- aktivasi dan penutupan periode harus mengikuti langkah dan prasyarat;
- data periode selesai harus dilindungi;
- dropdown periode di dashboard adalah filter laporan, bukan pengubah periode aktif sistem.

---

## 3. Definisi Status Periode

### 3.1 PERSIAPAN

Periode sedang disiapkan oleh admin.

Karakter:
- belum boleh dipakai untuk program, order, atau setoran operasional reseller;
- master periodik boleh disiapkan;
- admin menyiapkan komisi, barang periode, dan melakukan **kloning masal paket dan BOM** dari periode lalu;
- periode boleh diedit selama belum melanggar aturan tanggal dan urutan;
- periode boleh diaktifkan hanya jika checklist persiapan minimum terpenuhi.

Contoh aktivitas:
- mengisi `barang_periode`;
- mengisi `komisi_config`;
- melakukan **kloning masal paket dan BOM** dari periode sebelumnya;

- menyiapkan `reseller_periode`.

### 3.2 AKTIF

Periode sedang berjalan dan menjadi konteks kerja utama sistem.

Karakter:
- hanya boleh ada satu periode `AKTIF`;
- menjadi default untuk dashboard, master periodik, order, setoran, gudang, dan keuangan;
- reseller aktif boleh bekerja pada periode ini;
- konsumen dapat dibuatkan pesanan berbasis paket dan menerima setoran sebelum pesanan difinalkan;
- perubahan master periodik hanya boleh melalui aksi admin terkontrol; khusus harga paket adalah revisi darurat yang wajib alasan dan audit;
- UI wajib memberi peringatan jika perubahan master memengaruhi target, stok, komisi, atau laporan periode berjalan.

Contoh aktivitas:
- reseller membuat konsumen dan pesanan konsumen berbasis paket;
- reseller mencatat setoran konsumen dan mengunci paket saat pilihan paket sudah final;
- reseller mencatat setoran konsumen;
- admin mencatat belanja, packing, pengiriman, kas masuk, mutasi kas, setoran pusat, pencairan;
- admin memperbarui harga/BOM/komisi jika memang ada perubahan bisnis.

### 3.3 SELESAI

Periode sudah ditutup dan menjadi arsip/laporan.

Karakter:
- tidak boleh ada transaksi baru;
- perubahan master/transaksi periode selesai harus dibatasi;
- data tetap bisa dibaca untuk laporan;
- hanya koreksi administratif non-finansial yang boleh dilakukan langsung;
- koreksi transaksi uang, stok, order, dan pencairan harus melalui mekanisme khusus yang eksplisit dan terekam.

Catatan:
Untuk implementasi awal, `SELESAI` diperlakukan sebagai arsip read-only historis. Koreksi material tidak dilakukan lewat CRUD biasa.

---

## 4. Aturan Transisi Status

Transisi status hanya boleh maju:

```txt
PERSIAPAN -> AKTIF -> SELESAI
```

Aturan:
- `PERSIAPAN` tidak boleh langsung menjadi `SELESAI`.
- `AKTIF` tidak boleh kembali menjadi `PERSIAPAN`.
- `SELESAI` tidak boleh kembali menjadi `AKTIF` atau `PERSIAPAN`.
- Periode baru harus dibuat dengan status awal `PERSIAPAN`.
- Status tidak boleh diubah langsung dari frontend jika nantinya sudah ada RPC bisnis. Frontend harus memanggil aksi/RPC yang memvalidasi prasyarat.

Pesan UI:
- sukses aktivasi: `Periode berhasil diaktifkan.`
- sukses penutupan: `Periode berhasil diselesaikan.`
- gagal lompat status: `Status periode harus berjalan berurutan.`
- gagal mundur status: `Periode yang sudah berjalan tidak bisa dikembalikan ke status sebelumnya.`

---

## 5. Aturan Tanggal dan Urutan Periode

Periode harus membentuk rangkaian waktu yang rapi.

Aturan wajib:
- `tgl_mulai` wajib diisi;
- `tgl_selesai` wajib diisi;
- `tgl_selesai` tidak boleh lebih awal dari `tgl_mulai`;
- rentang periode tidak boleh overlap dengan periode lain;
- periode baru tidak boleh dimulai sebelum atau sama dengan tanggal selesai periode terakhir;
- edit tanggal periode tidak boleh menyisipkan periode ke masa lalu;
- edit tanggal periode harus tetap berada di antara periode sebelumnya dan periode berikutnya.

Contoh:

```txt
Periode A:
1 Maret 2025 sampai 20 Oktober 2025

Periode B valid:
21 Oktober 2025 sampai tanggal setelahnya

Periode B tidak valid:
20 Oktober 2025 sampai tanggal setelahnya
15 Oktober 2025 sampai tanggal setelahnya
1 Januari 2025 sampai 1 Februari 2025
```

Durasi 300 hari:
- `tgl_mulai` dihitung sebagai hari ke-1;
- `tgl_selesai` untuk 300 hari adalah `tgl_mulai + 299 hari`;
- helper UI harus memakai label jelas: `Set 300 hari inklusif`.

---

## 6. Periode Aktif Sistem vs Filter Periode Tampilan

Ada dua konsep yang tidak boleh dicampur:

### 6.1 Periode Aktif Sistem

Periode aktif sistem adalah satu baris `periode.status = 'AKTIF'`.

Dipakai sebagai default untuk:
- dashboard awal;
- header admin/reseller;
- halaman master periodik;
- order;
- setoran;
- gudang;
- kas;
- laporan awal.

Mengubah periode aktif sistem berarti mengubah status data di database. Aksi ini harus dikontrol ketat melalui workflow aktivasi/penutupan.

### 6.2 Filter Periode Tampilan

Filter periode tampilan adalah pilihan user di halaman tertentu untuk melihat data periode lain.

Dipakai untuk:
- melihat dashboard periode historis;
- melihat barang periode lama;
- melihat laporan periode selesai;
- membandingkan performa antar periode.

Filter ini tidak boleh mengubah status periode.

Label UI yang disarankan:
- `Lihat periode`
- `Periode laporan`
- `Filter periode`

Label yang harus dihindari untuk filter:
- `Pilih periode aktif`
- `Ganti periode aktif`

---

## 7. Perilaku Dashboard

Dashboard admin harus default ke periode aktif sistem jika ada.

Jika ada periode aktif:
- tampilkan nama periode;
- tampilkan tanggal mulai;
- tampilkan tanggal selesai;
- tampilkan status `AKTIF`;
- semua ringkasan awal memakai `periode_id` periode aktif.

Jika tidak ada periode aktif:
- tampilkan empty state:
  `Belum ada periode aktif. Aktifkan periode terlebih dahulu.`
- jangan memakai periode acak sebagai konteks kerja;
- boleh menyediakan dropdown untuk melihat periode lain sebagai laporan, tetapi harus jelas bahwa itu bukan periode aktif.

Dashboard boleh memiliki dropdown `Lihat periode`.

Perilaku dropdown:
- default = periode aktif;
- jika user memilih periode lain, angka dashboard berubah sesuai `periode_id` terpilih;
- header tetap dapat menampilkan periode aktif sistem secara terpisah;
- jika periode terpilih bukan periode aktif, tampilkan indikator:
  `Mode lihat periode historis.`

---

## 8. Perilaku Halaman Master Periodik

Halaman master periodik adalah halaman yang datanya menempel pada periode.

Contoh:
- Barang Periode
- Komisi Config
- Paket
- Detail Paket/BOM
- Reseller Periode

Aturan UI:
- default memilih periode aktif jika ada;
- jika tidak ada periode aktif, pilih periode terbaru untuk dibaca, tetapi tampilkan pesan bahwa belum ada periode aktif;
- pilihan periode di halaman ini adalah konteks data halaman, bukan pengubah status periode;
- data list dan form wajib memakai `periode_id` terpilih;
- create/update wajib memastikan data masuk ke periode yang sedang dipilih.

Aturan perubahan pada periode `AKTIF`:
- perubahan hanya boleh melalui aksi/RPC admin terkontrol;
- harga paket dianggap tetap sejak awal periode dan hanya direvisi sebagai kondisi darurat;
- alasan dan audit log wajib untuk revisi harga/paket terkunci;
- UI wajib memberi peringatan bahwa perubahan memengaruhi hitungan periode berjalan.

Pesan peringatan:
`Perubahan ini akan memperbarui nilai hitungan periode berjalan.`

Aturan perubahan pada periode `SELESAI`:
- default UI sebaiknya read-only;
- koreksi periode selesai harus menjadi fitur khusus, bukan CRUD biasa;
- catatan audit atau metadata non-finansial boleh dikoreksi jika tidak mengubah angka, status historis, stok, atau hak reseller.

---

## 9. Checklist Persiapan Periode

Periode `PERSIAPAN` harus memiliki checklist sebelum boleh diaktifkan.

Checklist minimum:

| Langkah | Data | Wajib Aktif? | Keterangan |
|---|---|---:|---|
| 1 | Barang periode | Ya | Minimal ada barang dengan harga/stok/anggaran untuk periode |
| 2 | Komisi config | Ya | Minimal config kategori yang dipakai paket |
| 3 | Paket | Ya | Minimal ada paket untuk periode |
| 4 | Detail paket/BOM | Ya | Semua paket harus punya detail barang valid |
| 5 | Reseller periode | Sebaiknya | Reseller yang ikut periode disiapkan |
| 6 | Akun kas | Ya | Akun kas lintas periode tersedia |
| 7 | Review tanggal | Ya | Tanggal tidak overlap dan urut |

Catatan:
- `Akun kas` tidak periodik, tetapi tetap menjadi prasyarat kerja operasional.
- `Reseller periode` bisa dibuat otomatis saat reseller aktif ikut periode, tetapi status konsepnya harus jelas.
- Checklist ini dapat dihitung sebagai read model atau RPC ringan.

Status checklist:
- `Belum mulai`
- `Belum lengkap`
- `Lengkap`

Tombol `Aktifkan Periode`:
- disabled jika checklist wajib belum lengkap;
- menampilkan daftar blocker;
- hanya aktif untuk periode `PERSIAPAN`.

Contoh blocker:
- `Belum ada barang periode.`
- `Belum ada komisi config.`
- `Masih ada paket tanpa detail barang.`
- `Belum ada akun kas.`

---

## 10. Aktivasi Periode

Aktivasi periode adalah aksi bisnis, bukan update status biasa.

Validasi wajib:
- periode target harus `PERSIAPAN`;
- tidak boleh ada periode lain `AKTIF`;
- tanggal periode valid;
- checklist persiapan wajib lengkap;
- jika ada periode sebelumnya, periode sebelumnya harus `SELESAI`;
- tidak boleh melompati urutan periode.

Database write:
- update `periode.status = 'AKTIF'`;
- catat `updated_at`;
- jika nanti ada audit log, catat user admin dan waktu aktivasi.

Pesan gagal:
- `Periode lain masih aktif. Selesaikan terlebih dahulu.`
- `Setup periode belum lengkap.`
- `Tanggal periode tidak valid.`
- `Periode sebelumnya belum selesai.`

---

## 11. Penutupan Periode

Penutupan periode adalah aksi bisnis, bukan update status biasa.

Validasi wajib:
- periode target harus `AKTIF`;
- semua transaksi harus selesai;
- semua reseller harus lunas sesuai aturan periode;
- pencairan tabungan/komisi harus selesai;
- tidak ada pengiriman/packing/belanja yang masih menggantung;
- carry-over stok ke periode berikutnya harus dihitung atau disiapkan.

Database write:
- update `periode.status = 'SELESAI'`;
- catat `updated_at`;
- jika nanti ada audit log, catat user admin dan waktu penutupan.

Pesan gagal:
- `Masih ada reseller yang belum lunas.`
- `Masih ada pencairan yang belum selesai.`
- `Masih ada transaksi operasional yang belum selesai.`
- `Stok akhir belum disiapkan untuk periode berikutnya.`

---

## 12. Carry-over Stok

Stok barang antar periode tidak otomatis mengubah periode lama.

Prinsip:
- stok akhir periode lama dihitung dari read model stok;
- saat periode baru disiapkan, stok akhir tersebut dicatat sebagai `barang_periode.stok_awal` periode baru;
- setelah periode lama selesai, `barang_periode.stok_awal` periode lama tidak boleh diubah untuk menyesuaikan periode baru.

Alur yang disarankan:

```txt
Hitung stok akhir periode lama
-> review admin
-> buat/isi barang_periode periode baru
-> stok akhir lama menjadi stok_awal baru
```

Jika admin ingin koreksi stok lama, harus lewat mekanisme koreksi yang jelas, bukan edit bebas setelah periode selesai.

---

## 13. Dampak Live Change

Keputusan bisnis proyek ini: harga paket, BOM, komisi, dan barang periode dapat berubah live saat periode aktif.

Dampak:
- target konsumen berubah;
- nilai akhir paket reseller berubah;
- komisi reseller berubah;
- kebutuhan stok berubah;
- laporan dashboard berubah.

Aturan UI:
- sebelum menyimpan perubahan master periode aktif, tampilkan konfirmasi;
- konfirmasi harus menyebut dampak ke hitungan periode berjalan;
- perubahan harus invalidate query/read model terkait.

Pesan konfirmasi:
`Perubahan ini akan memperbarui nilai hitungan periode berjalan. Lanjutkan?`

---

## 14. Prinsip Query dan Read Model

Semua query laporan dan dashboard wajib eksplisit menerima atau menentukan `periode_id`.

Aturan:
- jangan menghitung lintas periode kecuali view memang dirancang lintas periode;
- dashboard default memakai periode aktif;
- dashboard historis memakai filter periode;
- view stok, saldo, order, komisi, dan target wajib filter `periode_id`;
- transaksi `BATAL` dikeluarkan dari hitungan sesuai kontrak query.

Jika tidak ada periode aktif:
- query dashboard operasional tidak boleh diam-diam memakai periode terbaru;
- UI harus menampilkan empty state yang jelas.

---

## 15. Bahasa UI

Semua label user-facing wajib Bahasa Indonesia.

Gunakan:
- `Dasbor`
- `Data Master`
- `Terakhir diperbarui`
- `Ubah`
- `Anggaran belanja`
- `Lihat periode`
- `Periode aktif`
- `Periode laporan`
- `Aktifkan Periode`
- `Selesaikan Periode`

Hindari:
- `Dashboard`
- `Master Data`
- `Updated at`
- `Edit`
- `Budget`
- `Select active period`

Nama teknis di kode dan database tetap boleh memakai istilah existing seperti `budget_belanja`, `updated_at`, `status`, dan `periode_id`.

---

## 16. Implikasi Sprint

Sebelum masuk Phase 5, Sprint 4.4 harus memastikan:
- Barang Periode tersedia sebagai master periodik;
- periode aktif tampil di admin layout;
- dashboard admin memakai periode aktif sebagai default;
- dashboard dapat memiliki filter `Lihat periode` tanpa mengubah periode aktif sistem;
- tidak ada teks `Periode aktif belum dipilih` saat ada periode `AKTIF`;
- belum ada fitur Phase 5 yang ikut terbawa.

Sprint Phase 5 harus mengikuti dokumen ini:
- Komisi Config default ke periode aktif;
- Paket & BOM default ke periode aktif;
- Approval Reseller harus mempertimbangkan apakah reseller ikut periode aktif melalui `reseller_periode`;
- semua mutation periodik wajib membawa `periode_id` terpilih.

---

## 17. Definition of Done Periode

Sebuah fitur yang terkait periode baru dianggap selesai jika:

- memakai periode aktif sebagai default konteks kerja;
- menyediakan filter periode jika halaman bersifat laporan/historis;
- tidak mencampur data lintas periode;
- menampilkan empty state jika tidak ada periode aktif;
- validasi tanggal dan status tidak hanya di frontend;
- label UI Bahasa Indonesia;
- perubahan pada periode aktif memberi peringatan jika memengaruhi hitungan live;
- periode selesai tidak menerima input operasional baru;
- pesanan konsumen pada periode selesai tidak menerima setoran baru;
- order/pembagian baru tidak boleh dibuat pada periode selesai;
- hasil smoke test membuktikan data berubah saat periode filter diganti.

