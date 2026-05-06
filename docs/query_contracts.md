# Query Contracts
## Paket Lebaran Mumpuni

Dokumen ini mendefinisikan view dan RPC read-model yang dipakai frontend. Tujuannya agar frontend
tidak membuat formula ad hoc yang berbeda-beda.

---

## 1. Aturan Umum Query

Semua query laporan dan dashboard wajib menerima atau menentukan `periode_id`.

Semua query yang menghitung item, stok, target, komisi, poin, atau tabungan wajib exclude:

```sql
status_item != 'BATAL'
```

Semua angka uang dikembalikan sebagai numeric/decimal dari database, lalu diformat di frontend.

Frontend tidak boleh:

- menghitung ulang komisi bertingkat
- menghitung target tagihan dari daftar item lokal
- menghitung saldo kas sendiri
- menentukan status lunas hanya dari state lokal

Model pesanan konsumen:

- target konsumen wajib dibaca dari read model pesanan
- target berjalan = total `detail_pesanan_konsumen` yang masih dihitung
- `pesanan_konsumen.target_tagihan_snapshot` adalah snapshot target terkini
- view yang menampilkan konsumen wajib mengekspos status finalisasi dan asal target secara jelas

---

## 2. View: `v_ringkasan_konsumen`

### Tujuan

Menampilkan daftar konsumen reseller dengan target, total bayar, sisa bayar, dan status lunas.

### Parameter / Filter

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Filter periode aktif/historis |
| `no_reseller` | Ya untuk admin, otomatis dari auth untuk reseller | Reseller hanya miliknya |

### Kolom Output

| Kolom | Tipe | Formula |
|---|---|---|
| `periode_id` | BIGINT | Periode |
| `pesanan_konsumen_id` | BIGINT | Pesanan aktif konsumen di periode |
| `no_reseller` | TEXT | Pemilik konsumen |
| `konsumen_id` | BIGINT | ID konsumen |
| `nama_konsumen` | TEXT | Dari `konsumen.nama` |
| `telepon` | TEXT | Dari `konsumen.telepon` |
| `jumlah_item_pesanan` | INT | COUNT item exclude BATAL |
| `target_tagihan` | NUMERIC | Snapshot target dari `pesanan_konsumen.target_tagihan_snapshot` |
| `target_berjalan` | NUMERIC | Target validasi setoran saat ini |
| `total_bayar` | NUMERIC | SUM `setoran_konsumen.nominal` per periode |
| `sisa_bayar` | NUMERIC | target_berjalan - total_bayar |
| `persentase_bayar` | NUMERIC | total_bayar / target_berjalan * 100 |
| `status_pesanan` | TEXT | AKTIF / PERLU_PERHATIAN / SELESAI / BATAL |
| `is_final` | BOOLEAN | `tanggal_final IS NOT NULL` |
| `status_lunas` | TEXT | `LUNAS` jika sisa = 0 dan target > 0, else `BELUM` |

### Sort Default

`sisa_bayar DESC`, lalu `nama_konsumen ASC`.

### Dipakai Oleh

- Reseller: Daftar Konsumen
- Reseller: Input Setoran
- Admin: Detail Reseller

---

## 2A. View: `v_ringkasan_pesanan_konsumen`

### Tujuan

Read model utama untuk pesanan konsumen.

### Kolom Output

| Kolom | Tipe | Formula |
|---|---|---|
| `pesanan_konsumen_id` | BIGINT | ID pesanan |
| `periode_id` | BIGINT | Periode |
| `no_reseller` | TEXT | Pemilik pesanan |
| `konsumen_id` | BIGINT | Konsumen |
| `nama_konsumen` | TEXT | Nama konsumen |
| `target_tagihan` | NUMERIC | Snapshot target pesanan |
| `target_berjalan` | NUMERIC | SUM item aktif/terhenti yang masih dihitung |
| `total_bayar` | NUMERIC | SUM setoran pesanan setelah koreksi |
| `sisa_bayar` | NUMERIC | target_berjalan - total_bayar |
| `persentase_bayar` | NUMERIC | total_bayar / target_berjalan * 100 |
| `status_pesanan` | TEXT | Status pesanan |
| `tanggal_setoran_terakhir` | DATE | Setoran terakhir |
| `hari_tanpa_setoran` | INT | Hari sejak setoran terakhir |
| `tanggal_final` | DATE | Tanggal finalisasi |
| `is_final` | BOOLEAN | `tanggal_final IS NOT NULL` |

### Sort Default

`sisa_bayar DESC`, lalu `nama_konsumen ASC`.

---

## 2B. View: `v_pesanan_perlu_perhatian`

### Tujuan

Menampilkan kandidat pesanan yang perlu ditindaklanjuti reseller/admin.

### Kolom Output

| Kolom | Tipe |
|---|---|
| `pesanan_konsumen_id` | BIGINT |
| `periode_id` | BIGINT |
| `no_reseller` | TEXT |
| `nama_reseller` | TEXT |
| `nama_konsumen` | TEXT |
| `status_pesanan` | TEXT |
| `target_berjalan` | NUMERIC |
| `total_bayar` | NUMERIC |
| `sisa_bayar` | NUMERIC |
| `tanggal_setoran_terakhir` | DATE |
| `hari_tanpa_setoran` | INT |
| `target_seharusnya` | NUMERIC |
| `selisih_dari_target` | NUMERIC |
| `alasan_watchlist` | TEXT |

View ini hanya kandidat. Perubahan status tetap melalui boundary manual.

---

## 3. View: `v_ringkasan_reseller`

### Tujuan

Menampilkan ringkasan kewajiban, setoran, komisi, tabungan, saldo, dan status lunas per reseller.

### Parameter / Filter

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Periode laporan |
| `no_reseller` | Opsional untuk admin | Reseller otomatis miliknya |

### Kolom Output

| Kolom | Tipe | Formula |
|---|---|---|
| `periode_id` | BIGINT | Periode |
| `no_reseller` | TEXT | Kode reseller |
| `nama_reseller` | TEXT | Nama reseller |
| `status_reseller` | TEXT | Status master reseller |
| `jumlah_konsumen` | INT | COUNT konsumen |
| `jumlah_pesanan_aktif` | INT | COUNT pesanan aktif/belum selesai |
| `jumlah_item_pesanan` | INT | COUNT detail item exclude BATAL |
| `nilai_paket` | NUMERIC | SUM nilai item exclude BATAL |
| `nilai_tabungan` | NUMERIC | SUM uang_tabungan item AKTIF exclude BATAL |
| `komisi` | NUMERIC | Komisi bertingkat live |
| `komisi_untuk_pelunasan` | NUMERIC | Jika flag potong komisi TRUE |
| `nilai_akhir_tabungan` | NUMERIC | Jika flag potong tabungan TRUE maka 0, else nilai_tabungan |
| `nilai_akhir_paket` | NUMERIC | nilai_paket - potong komisi - potong tabungan |
| `total_dikumpulkan` | NUMERIC | SUM setoran_konsumen |
| `total_disetor_pusat` | NUMERIC | SUM setoran |
| `saldo_belum_disetor` | NUMERIC | dikumpulkan - disetor pusat |
| `sisa_setor_pusat` | NUMERIC | nilai_akhir_paket - disetor pusat |
| `tabungan_diambil` | NUMERIC | SUM pencairan TABUNGAN |
| `sisa_tabungan` | NUMERIC | nilai_akhir_tabungan - tabungan_diambil |
| `komisi_diambil` | NUMERIC | SUM pencairan KOMISI |
| `sisa_komisi` | NUMERIC | komisi - komisi_untuk_pelunasan - komisi_diambil |
| `poin` | INT | SUM poin item AKTIF exclude BATAL |
| `status_lunas_reseller` | TEXT | `LUNAS` jika sisa_setor_pusat = 0 dan nilai_akhir_paket > 0 |

Catatan:

- `jumlah_item_pesanan` adalah total item detail yang masih dihitung
- `jumlah_pesanan_aktif` berasal dari header pesanan
- `total_dikumpulkan` berasal dari setoran pesanan konsumen

---

## 4. View: `v_stok_barang`

### Tujuan

Menghitung stok barang per periode.

### Kolom Output

| Kolom | Tipe | Formula |
|---|---|---|
| `periode_id` | BIGINT | Periode |
| `id_barang` | TEXT | Kode barang |
| `nama_barang` | TEXT | Nama barang |
| `kategori` | TEXT | Kategori barang |
| `stok_awal` | INT | Dari `barang_periode` |
| `jumlah_belanja` | INT | SUM belanja_detail |
| `jumlah_dipacking` | INT | SUM packing * BOM |
| `jumlah_kirim_tunggal` | INT | COUNT detail pesanan tunggal terkirim |
| `stok_tersedia` | INT | stok_awal + belanja - dipacking - kirim_tunggal |
| `jumlah_kebutuhan` | INT | kebutuhan item aktif |
| `harus_belanja` | INT | max(kebutuhan - stok_tersedia, 0) |
| `stok_lebih` | INT | max(stok_tersedia - kebutuhan, 0) |

---

## 5. View: `v_stok_paket_jadi`

### Tujuan

Menghitung stok paket komposit siap kirim.

### Kolom Output

| Kolom | Tipe | Formula |
|---|---|---|
| `periode_id` | BIGINT | Periode |
| `paket_id` | BIGINT | Paket komposit |
| `kode` | TEXT | Kode paket |
| `nama_paket` | TEXT | Nama paket |
| `jumlah_packing` | INT | SUM packing |
| `jumlah_terkirim` | INT | COUNT detail pesanan SUDAH |
| `stok_paket_jadi` | INT | packing - terkirim |
| `harus_packing` | INT | item aktif komposit - packing |

---

## 6. View: `v_saldo_kas`

### Tujuan

Menghitung saldo akun kas dari transaksi live.

### Parameter / Filter

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Scope periode untuk dashboard, laporan, dan validasi transaksi |
| `akun_kas_id` | Tidak | Filter satu akun bila dibutuhkan UI |

Catatan:

- `v_saldo_kas` adalah satu-satunya read model saldo kas yang dipakai dokumen ini
- tidak ada view terpisah bernama `v_saldo_kas_periode` atau `v_saldo_kas_lintas_periode`
- kebutuhan historis atau lintas periode dicapai dengan mengganti nilai `periode_id`, bukan dengan
  membuat nama view baru

### Kolom Output

| Kolom | Tipe | Formula |
|---|---|---|
| `akun_kas_id` | BIGINT | Akun kas |
| `jenis` | TEXT | Nama/jenis akun kas |
| `saldo_awal` | NUMERIC | Saldo awal akun |
| `total_setoran` | NUMERIC | SUM setoran masuk |
| `total_kas_masuk` | NUMERIC | SUM kas_masuk |
| `total_mutasi_masuk` | NUMERIC | SUM mutasi masuk |
| `total_mutasi_keluar` | NUMERIC | SUM mutasi keluar |
| `total_pencairan` | NUMERIC | SUM pencairan |
| `total_belanja` | NUMERIC | SUM belanja |
| `saldo` | NUMERIC | saldo_awal + setoran + kas_masuk + mutasi_masuk - mutasi_keluar - pencairan - belanja |

### Sort Default

`jenis ASC`.

### Dipakai Oleh

- Admin Dashboard: panel `Posisi Kas`
- Admin Keuangan: halaman saldo kas
- Admin Gudang/Keuangan: validasi saldo sumber untuk belanja, mutasi, dan pencairan

---

## 7. View: `v_detail_pesanan_aktif`

### Tujuan

Read model daftar detail item pesanan untuk admin dan reseller.

### Kolom Output

| Kolom | Tipe |
|---|---|
| `id` | BIGINT |
| `periode_id` | BIGINT |
| `pesanan_konsumen_id` | BIGINT |
| `no_reseller` | TEXT |
| `nama_reseller` | TEXT |
| `konsumen_id` | BIGINT |
| `nama_konsumen` | TEXT |
| `paket_id` | BIGINT |
| `kode_paket` | TEXT |
| `nama_paket` | TEXT |
| `kategori` | TEXT |
| `perlu_packing` | BOOLEAN |
| `qty` | INT |
| `motif_warna` | TEXT |
| `kelompok` | TEXT |
| `status_item` | TEXT |
| `status_kirim` | TEXT |
| `tgl_dikirim` | DATE |
| `uang_terhenti` | NUMERIC |
| `nilai_item` | NUMERIC |
| `nilai_komisi` | NUMERIC |
| `poin_item` | INT |
| `is_final` | BOOLEAN |

Catatan:

View ini boleh menampilkan `BATAL` jika halaman admin membutuhkan histori, tetapi semua summary harus exclude `BATAL`.

---

## 7A. View: `v_pembagian_reseller`

### Tujuan

Menampilkan status batch pembagian paket dari pusat ke reseller.

### Kolom Output

| Kolom | Tipe |
|---|---|
| `pembagian_id` | BIGINT |
| `periode_id` | BIGINT |
| `no_reseller` | TEXT |
| `nama_reseller` | TEXT |
| `tanggal` | DATE |
| `status` | TEXT |
| `jumlah_item` | INT |
| `jumlah_siap` | INT |
| `jumlah_diserahkan` | INT |
| `jumlah_kurang` | INT |
| `catatan` | TEXT |

---

## 7B. View: `v_riwayat_koreksi`

### Tujuan

Menampilkan koreksi transaksi untuk audit admin.

### Parameter / Filter

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Scope periode laporan |
| `jenis_transaksi` | Tidak | Filter jenis transaksi yang dikoreksi |
| `kata_kunci` | Tidak | Cari reseller, alasan, atau ringkasan koreksi |

### Kolom Output Minimum

| Kolom | Tipe | Catatan |
|---|---|---|
| `koreksi_transaksi_id` | BIGINT | ID row koreksi |
| `periode_id` | BIGINT | Periode terkait |
| `jenis_transaksi` | TEXT | Mis. `PESANAN`, `SETORAN_KONSUMEN`, `SETORAN_PUSAT`, `BELANJA`, `PACKING`, `PENCAIRAN`, `MUTASI_KAS`, `KAS_MASUK`, `PEMBAGIAN_PAKET` |
| `target_tabel` | TEXT | Nama tabel/transaksi yang dikoreksi |
| `target_id` | BIGINT | ID transaksi utama yang dikoreksi |
| `no_reseller` | TEXT | Null bila tidak terkait reseller |
| `nama_reseller` | TEXT | Nama reseller untuk kebutuhan baca admin |
| `alasan_koreksi` | TEXT | Alasan koreksi yang dicatat admin |
| `ringkasan_koreksi` | TEXT | Ringkasan perubahan yang mudah dibaca |
| `actor_user_id` | UUID | User admin yang melakukan koreksi |
| `actor_nama` | TEXT | Nama tampil admin pelaku aksi |
| `created_at` | TIMESTAMPTZ | Waktu koreksi dicatat |

### Sort Default

`created_at DESC`.

### Dipakai Oleh

- Admin: tab `Riwayat Koreksi` pada `admin/laporan/audit`

---

## 7B.1 View: `v_audit_log`

### Tujuan

Menampilkan jejak aksi penting lintas transaksi untuk admin tanpa membuka tabel audit mentah.

### Parameter / Filter

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Scope periode laporan |
| `jenis_transaksi` | Tidak | Filter domain transaksi yang diaudit |
| `kata_kunci` | Tidak | Cari reseller, aksi, atau ringkasan audit |

### Kolom Output Minimum

| Kolom | Tipe | Catatan |
|---|---|---|
| `audit_log_id` | BIGINT | ID row audit |
| `periode_id` | BIGINT | Periode terkait |
| `jenis_transaksi` | TEXT | Domain transaksi utama |
| `target_tabel` | TEXT | Nama tabel/transaksi yang diaudit |
| `target_id` | BIGINT | ID transaksi utama |
| `no_reseller` | TEXT | Null bila tidak terkait reseller |
| `nama_reseller` | TEXT | Nama reseller untuk kebutuhan baca admin |
| `aksi` | TEXT | Mis. `CREATE`, `UPDATE`, `FINALISASI`, `KOREKSI`, `SERAHKAN`, `AKTIVASI`, `TUTUP` |
| `ringkasan_audit` | TEXT | Ringkasan aksi yang mudah dibaca |
| `actor_user_id` | UUID | Pelaku aksi |
| `actor_nama` | TEXT | Nama tampil pelaku aksi |
| `created_at` | TIMESTAMPTZ | Waktu audit dicatat |

### Sort Default

`created_at DESC`.

### Dipakai Oleh

- Admin: tab `Audit Log` pada `admin/laporan/audit`

---

## 7C. View: `v_monitoring_setoran_konsumen_admin`

### Tujuan

Menampilkan monitoring setoran konsumen untuk admin tanpa memaksa frontend menghitung agregasi sendiri.

### Kolom Output Minimum

| Kolom | Tipe |
|---|---|
| `periode_id` | BIGINT |
| `pesanan_konsumen_id` | BIGINT |
| `konsumen_id` | BIGINT |
| `no_reseller` | TEXT |
| `nama_reseller` | TEXT |
| `nama_konsumen` | TEXT |
| `total_target` | NUMERIC |
| `total_setor` | NUMERIC |
| `sisa_bayar` | NUMERIC |
| `status_lunas_konsumen` | TEXT |
| `setor_terakhir_at` | TIMESTAMPTZ |

---

## 7D. View: `v_monitoring_setoran_pusat_admin`

### Tujuan

Menampilkan monitoring setor pusat reseller untuk admin.

### Kolom Output Minimum

| Kolom | Tipe |
|---|---|
| `periode_id` | BIGINT |
| `no_reseller` | TEXT |
| `nama_reseller` | TEXT |
| `total_dikumpulkan` | NUMERIC |
| `total_disetor_pusat` | NUMERIC |
| `saldo_belum_disetor` | NUMERIC |
| `sisa_setor_pusat` | NUMERIC |
| `status_lunas_reseller` | TEXT |
| `setor_terakhir_at` | TIMESTAMPTZ |

---

## 8. RPC Read: `get_dashboard_admin`

### Tujuan

Mengambil angka ringkas dashboard admin untuk satu periode.

### Output Minimal

| Kolom | Tipe |
|---|---|
| `total_reseller_aktif` | INT |
| `total_konsumen` | INT |
| `total_pesanan_aktif` | INT |
| `total_pesanan_perlu_perhatian` | INT |
| `total_pesanan_belum_final` | INT |
| `total_item_pesanan_aktif` | INT |
| `total_nilai_paket` | NUMERIC |
| `total_dikumpulkan` | NUMERIC |
| `total_disetor_pusat` | NUMERIC |
| `total_kas_masuk` | NUMERIC |
| `total_saldo_belum_disetor` | NUMERIC |
| `total_sisa_setor_pusat` | NUMERIC |
| `total_komisi` | NUMERIC |
| `total_tabungan` | NUMERIC |
| `jumlah_item_belum_kirim` | INT |
| `jumlah_item_sudah_kirim` | INT |
| `jumlah_item_batal` | INT |
| `jumlah_pembagian_belum_diserahkan` | INT |

---

## 9. RPC Read: `get_dashboard_reseller`

### Tujuan

Mengambil ringkasan beranda reseller.

### Output Minimal

| Kolom | Tipe |
|---|---|
| `jumlah_konsumen` | INT |
| `jumlah_pesanan_aktif` | INT |
| `jumlah_item_pesanan` | INT |
| `jumlah_konsumen_lunas` | INT |
| `total_dikumpulkan` | NUMERIC |
| `total_disetor_pusat` | NUMERIC |
| `saldo_belum_disetor` | NUMERIC |
| `sisa_setor_pusat` | NUMERIC |
| `sisa_tabungan` | NUMERIC |
| `sisa_komisi` | NUMERIC |
| `poin` | INT |

---

## 10. RPC Read Laporan

Read layer berikut tetap dipakai, dengan sumber data baru:

- `get_laporan_rekap_reseller` -> `v_ringkasan_reseller`
- `get_laporan_stok` -> `v_stok_barang`, `v_stok_paket_jadi`
- `get_laporan_pengiriman` -> `v_detail_pesanan_aktif`, `v_pembagian_reseller`
- `get_laporan_audit_koreksi` -> `v_riwayat_koreksi`, `v_audit_log`

### 10.1 RPC Read: `get_laporan_audit_koreksi`

#### Tujuan

Menjadi wrapper read resmi untuk route `GET /api/admin/laporan/audit` pada satu halaman admin
dengan dua tab: `Audit Log` dan `Riwayat Koreksi`.

#### Input / Filter

| Field | Wajib | Catatan |
|---|---|---|
| `periode_id` | Ya | Scope periode laporan |
| `tab` | Ya | `audit` atau `koreksi` |
| `page` | Tidak | Default `1` |
| `page_size` | Tidak | Default mengikuti contract report list |
| `jenis_transaksi` | Tidak | Filter lintas domain transaksi |
| `kata_kunci` | Tidak | Search server-side |

#### Perilaku

- jika `tab = 'audit'`, sumber data adalah `v_audit_log`
- jika `tab = 'koreksi'`, sumber data adalah `v_riwayat_koreksi`
- sorting default untuk kedua tab adalah `created_at DESC`
- RPC ini admin-only

#### Output

Return `ReportListEnvelope<T>` sesuai tab yang diminta:

- `tab = 'audit'` -> row `v_audit_log`
- `tab = 'koreksi'` -> row `v_riwayat_koreksi`

Frontend tetap memakai satu route `admin/laporan/audit`; perpindahan tab hanya mengganti nilai
filter `tab`, bukan route kedua.

---

## 11. Query Performance Rules

Index wajib mengacu ke `docs/schema_mapping.md`.

Query list reseller/konsumen/pesanan harus:

- pagination server-side untuk admin
- search server-side untuk data besar
- sort default dari database
- tidak mengambil semua row transaksi untuk dihitung di browser
