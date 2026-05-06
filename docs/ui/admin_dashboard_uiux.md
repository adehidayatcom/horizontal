# Admin Dashboard UI/UX Spec
## Paket Lebaran Mumpuni

Dokumen ini menjadi acuan desain tampilan Dashboard Admin. Fokusnya adalah rancangan informasi, komponen, state, dan mapping data. Dokumen ini bukan instruksi implementasi kode.

---

## 1. Tujuan Halaman

Dashboard Admin adalah pusat pantau operasional periode berjalan.

Navigasi, route, dan perilaku saat belum ada periode aktif wajib mengikuti
`docs/navigation_and_period_setup_ui.md`.

Admin harus bisa melihat dalam 1 layar:

- Apakah uang konsumen sudah terkumpul sesuai target paket.
- Berapa pesanan konsumen yang belum difinalkan.
- Pesanan mana yang perlu perhatian karena lama tidak setor atau tertinggal target.
- Berapa uang yang sudah disetor reseller ke pusat.
- Reseller mana yang perlu ditindaklanjuti.
- Order mana yang belum dikirim.
- Stok atau packing mana yang perlu perhatian.
- Posisi kas pusat dan pemasukan tambahan.

Dashboard bersifat read-only. Aksi cepat hanya membawa admin ke halaman transaksi terkait, bukan langsung mengubah data dari dashboard.

---

## 2. Prinsip Desain

1. Desktop-first untuk admin pusat.
2. Padat tetapi tetap mudah discan.
3. Semua angka berasal dari view/RPC di `docs/query_contracts.md`.
4. Tidak ada kalkulasi bisnis kompleks di frontend.
5. Semua data wajib berada dalam scope `periode_id`.
6. Dashboard tidak memakai kartu dekoratif yang tidak membawa keputusan operasional.
7. Status penting harus terlihat dari warna, badge, dan angka, bukan dari paragraf penjelasan panjang.

---

## 3. Struktur Layout

### Sidebar Admin

Sidebar kiri memakai background `teal-900`, logo sederhana, dan navigasi berkelompok.

| Grup | Menu |
|---|---|
| Ringkasan | Dashboard |
| Master Data | Periode, Paket, Barang, Akun Kas, Komisi |
| Relasi | Reseller, Konsumen |
| Order & Setoran | Order, Setoran Konsumen, Setoran Pusat |
| Gudang | Belanja, Packing, Pengiriman |
| Keuangan | Kas Masuk, Mutasi Kas, Pencairan |
| Laporan | Rekap Reseller, Laporan Stok, Laporan Pengiriman, Audit |
| Sistem | Pengaturan |

Menu aktif: `bg-white/10`, teks putih, border kiri `emerald-400`.

### Header Atas

Header atas berisi:

- Breadcrumb: `Admin / Dashboard`
- Selector periode atau badge periode aktif.
- Status periode: `PERSIAPAN`, `AKTIF`, `SELESAI`.
- Tombol refresh data.
- User menu.

Jika tidak ada periode aktif, tampilkan state kosong yang meminta admin memilih periode historis atau membuat periode baru.
Jika ada periode `PERSIAPAN`, arahkan admin ke `Wizard Persiapan` atau `Checklist Aktivasi`,
bukan dashboard operasional.

### Konten Utama

Konten memakai background `slate-50`, container maksimal lebar `1440px`, dan spacing konsisten.

Urutan section:

1. Header ringkasan periode.
2. KPI utama.
3. Watchlist operasional.
4. Progres setoran reseller.
5. Gudang dan pengiriman.
6. Posisi kas.
7. Rekap reseller.
8. Aktivitas terbaru.

---

## 4. Wireframe Desktop

```txt
+------------------+----------------------------------------------------------+
| Sidebar          | Header: Admin / Dashboard      Periode Aktif     User    |
|                  +----------------------------------------------------------+
| Dashboard        | [Gradient Header: Dashboard Admin + quick actions]       |
| Master Data      |                                                          |
| Order & Setoran  | [Nilai Paket] [Dikumpulkan] [Disetor Pusat] [Belum Setor]|
| Gudang           |                                                          |
| Keuangan         | [Reseller Aktif] [Konsumen] [Order Aktif] [Sisa Pusat]   |
| Laporan          |                                                          |
|                  | [Watchlist: Belum Lunas | Belum Kirim | Stok Kritis]     |
|                  |                                                          |
|                  | [Progres Setoran Reseller]      [Keuangan Kas]           |
|                  | [table/top reseller]             [akun kas + saldo]      |
|                  |                                                          |
|                  | [Gudang & Pengiriman]            [Aktivitas Terbaru]     |
|                  |                                                          |
|                  | [Rekap Reseller Table]                                     |
+------------------+----------------------------------------------------------+
```

---

## 5. Header Ringkasan Periode

Visual:

- Full-width band dengan gradient `from-teal-600 to-emerald-500`.
- Radius kecil (`rounded-sm` atau `rounded-md`), shadow tipis, tanpa ornamen berlebihan.
- Judul: `Dashboard Admin`
- Subtitle: `Ringkasan operasional periode {nama_periode}`.
- Informasi kecil: tanggal mulai, tanggal selesai, jumlah hari berjalan.

Quick action:

| Aksi | Tujuan |
|---|---|
| Input Belanja | Ke halaman Belanja |
| Packing | Ke halaman Packing |
| Pengiriman | Ke halaman Pengiriman |
| Kas Masuk | Ke halaman Kas Masuk |

Quick action memakai tombol kecil dengan icon lucide:

- `ShoppingCart` untuk Belanja
- `PackageCheck` untuk Packing
- `Truck` untuk Pengiriman
- `CirclePlus` atau `WalletCards` untuk Kas Masuk

---

## 6. KPI Utama

KPI utama harus muncul langsung setelah header.

| Kartu | Sumber Data | Catatan Tampilan |
|---|---|---|
| Nilai Paket | `total_nilai_paket` | Total nilai order aktif exclude BATAL |
| Dikumpulkan | `total_dikumpulkan` | Setoran konsumen yang sudah masuk ke reseller |
| Disetor Pusat | `total_disetor_pusat` | Uang reseller yang sudah masuk kas pusat |
| Belum Disetor | `total_saldo_belum_disetor` | Selisih dikumpulkan vs disetor pusat |

Setiap kartu:

- Ukuran stabil, tidak berubah saat angka panjang.
- Format uang: `Rp 12.345.678`.
- Label kecil di atas, angka besar di bawah.
- Icon kecil kanan atas.
- Footer mini: status atau konteks singkat, bukan penjelasan panjang.

Warna:

- Nilai Paket: teal.
- Dikumpulkan: emerald.
- Disetor Pusat: sky.
- Belum Disetor: amber jika > 0, slate jika 0.

---

## 7. KPI Pendukung

KPI pendukung lebih kecil dari KPI utama.

| Kartu | Sumber Data |
|---|---|
| Reseller Aktif | `total_reseller_aktif` |
| Konsumen | `total_konsumen` |
| Order Aktif | `total_order_aktif` |
| Sisa Setor Pusat | `total_sisa_setor_pusat` |
| Komisi | `total_komisi` |
| Tabungan | `total_tabungan` |
| Kas Masuk | `total_kas_masuk` |

KPI pendukung boleh dalam grid 7 kartu kecil di desktop, atau 3-4 kolom tergantung lebar layar.

---

## 8. Watchlist Operasional

Watchlist adalah area untuk hal yang perlu tindakan.

| Item | Data | Aksi |
|---|---|---|
| Reseller belum lunas | `v_ringkasan_reseller.status_lunas_reseller = BELUM` | Lihat Rekap Reseller |
| Pesanan belum final | `v_ringkasan_pesanan_konsumen.tanggal_final IS NULL` | Lihat Pesanan |
| Pesanan perlu perhatian | `v_pesanan_perlu_perhatian` | Follow up Reseller |
| Detail pesanan belum kirim | `jumlah_item_belum_kirim` atau `v_detail_pesanan_aktif.status_kirim = BELUM` | Ke Pengiriman |
| Stok perlu belanja | `v_stok_barang.harus_belanja > 0` | Ke Belanja |
| Paket perlu packing | `v_stok_paket_jadi.harus_packing > 0` | Ke Packing |
| Pembagian belum diserahkan | `v_pembagian_reseller.status != SELESAI` | Ke Pembagian |
| Pencairan siap | reseller `LUNAS` dengan sisa komisi/tabungan > 0 | Ke Pencairan |

Tampilan:

- Gunakan card horizontal berisi icon, label, angka, dan tombol teks pendek.
- Warna hanya sebagai sinyal: amber untuk perlu perhatian, rose untuk blocker, emerald untuk aman.
- Jika semua aman, tampilkan state ringkas: `Operasional periode ini terkendali`.

---

## 9. Progres Setoran Reseller

Section ini menjawab pertanyaan: uang sudah sampai mana?

Komponen:

- Progress bar total: `total_disetor_pusat / total_nilai_paket`.
- Sub progress: `total_dikumpulkan / total_nilai_paket`.
- Tabel top reseller yang perlu ditindaklanjuti.

Kolom tabel:

| Kolom | Sumber |
|---|---|
| Reseller | `nama_reseller`, `no_reseller` |
| Konsumen | `jumlah_konsumen` |
| Nilai Paket | `nilai_akhir_paket` |
| Dikumpulkan | `total_dikumpulkan` |
| Disetor | `total_disetor_pusat` |
| Belum Disetor | `saldo_belum_disetor` |
| Sisa Pusat | `sisa_setor_pusat` |
| Status | `status_lunas_reseller` |

Sort default:

1. `sisa_setor_pusat DESC`
2. `saldo_belum_disetor DESC`
3. `nama_reseller ASC`

Klik baris membuka detail reseller.

---

## 10. Gudang dan Pengiriman

Section ini dibagi 2 panel.

### Panel Stok Barang

Sumber: `v_stok_barang`.

Kolom ringkas:

- Barang
- Stok tersedia
- Kebutuhan
- Harus belanja
- Stok lebih

Tampilkan maksimal 5 barang paling kritis berdasarkan `harus_belanja DESC`.

### Panel Paket Jadi

Sumber: `v_stok_paket_jadi`.

Kolom ringkas:

- Paket
- Sudah packing
- Terkirim
- Stok paket jadi
- Harus packing

Tampilkan maksimal 5 paket dengan `harus_packing > 0`.

---

## 11. Posisi Kas

Sumber: `v_saldo_kas`.

Default dashboard memakai `v_saldo_kas` dengan `periode_id` periode aktif. Jika admin melihat
periode historis, panel kas tetap memakai view yang sama dengan `periode_id` historis tersebut.

Kolom:

| Kolom | Sumber |
|---|---|
| Akun Kas | `jenis` |
| Saldo Awal | `saldo_awal` |
| Setoran | `total_setoran` |
| Kas Masuk | `total_kas_masuk` |
| Belanja | `total_belanja` |
| Pencairan | `total_pencairan` |
| Saldo | `saldo` |

Visual:

- Akun kas ditampilkan sebagai list ringkas, bukan tabel besar jika jumlah akun sedikit.
- Saldo negatif atau saldo tidak cukup diberi warna rose.
- Total kas masuk eksternal harus terlihat agar tidak bercampur dengan setoran reseller.

---

## 12. Rekap Reseller

Dashboard menampilkan versi ringkas dari halaman Rekap Reseller.

Sumber: `v_ringkasan_reseller`.

Fitur:

- Search reseller.
- Filter status lunas: Semua, Belum, Lunas.
- Pagination server-side.
- Link ke halaman detail reseller.

Kolom minimum:

- No Reseller
- Nama Reseller
- Jumlah Konsumen
- Nilai Akhir Paket
- Total Dikumpulkan
- Total Disetor Pusat
- Sisa Setor Pusat
- Status Lunas

Catatan:

- Jangan render semua 200 reseller sekaligus jika query sudah mendukung pagination.
- Tabel ini tidak menghitung formula sendiri.

---

## 13. Aktivitas Terbaru

Aktivitas terbaru membantu admin membaca ritme operasional.

Sumber data bisa dibuat sebagai view/RPC tambahan nanti, misalnya `v_aktivitas_admin`.

Jenis aktivitas:

- Setoran konsumen masuk.
- Setoran reseller ke pusat.
- Kas masuk.
- Belanja.
- Packing.
- Pengiriman.
- Pencairan.

Format item:

```txt
10:42 - Setoran pusat - RSL-023 - Rp 2.500.000
10:15 - Pengiriman - 18 order - Paket Sembako A
09:58 - Kas masuk - Modal owner - Rp 20.000.000
```

Jika view ini belum tersedia di fase awal, section boleh disembunyikan sampai kontrak query dibuat.

---

## 13A. Audit dan Koreksi Admin

Route audit admin memakai satu halaman: `admin/laporan/audit`.

Komposisi halaman:

- tab `Audit Log`
- tab `Riwayat Koreksi`
- filter bersama: `periode`, `jenis transaksi`, `kata kunci`

Aturan:

- halaman ini admin-only
- tab hanya mengubah isi panel, bukan berpindah ke route audit kedua
- `Audit Log` fokus pada jejak aksi penting
- `Riwayat Koreksi` fokus pada koreksi yang sudah tercatat dengan alasan dan waktu
- jika salah satu tab belum punya data, tampilkan empty state yang jujur tanpa menyembunyikan tab lain

Halaman audit ini bersifat read-only untuk fase awal. Koreksi material tetap mengikuti jalur audit khusus dan tidak menjadi inline action di dashboard.

---

## 14. State Halaman

### Loading

- Gunakan skeleton untuk KPI, tabel, dan panel.
- Jangan tampilkan angka 0 saat data masih loading.

### Empty: Belum Ada Periode Aktif

Judul: `Belum ada periode aktif`

Aksi:

- `Buat Periode`
- `Pilih Periode Historis`

### Empty: Data Periode Masih Kosong

Judul: `Periode sudah aktif, data belum tersedia`

Aksi:

- `Kelola Paket`
- `Approve Reseller`

### Error

Pesan harus Bahasa Indonesia.

Contoh:

- `Dashboard belum bisa dimuat. Coba muat ulang.`
- `Data periode tidak ditemukan.`

Tidak boleh memakai browser alert.

### Stale / Refresh

Header menampilkan status kecil:

- `Diperbarui barusan`
- `Diperbarui 5 menit lalu`

Tombol refresh harus memicu refetch query.

---

## 15. Responsive Behavior

### Desktop >= 1280px

- Sidebar selalu terlihat.
- KPI utama 4 kolom.
- Section bawah 2 kolom.
- Rekap reseller full width.

### Tablet 768px - 1279px

- Sidebar boleh collapse menjadi icon rail.
- KPI utama 2 kolom.
- Section bawah 1 kolom jika lebar tidak cukup.

### Mobile < 768px

Admin bukan mobile-first, tetapi tetap harus bisa dibuka:

- Sidebar menjadi sheet/drawer.
- KPI 1 kolom.
- Tabel berubah menjadi horizontal scroll hanya untuk admin, bukan reseller.
- Quick action menjadi 2 kolom.

---

## 16. Komponen UI

Gunakan `Material-UI v7` dan wrapper internal proyek sebagai base.

Ikuti kontrak aktif berikut:

- `docs/frontend_architecture.md`
- `docs/frontend_component_contracts.md`
- `docs/component_patterns.md`

| Komponen | Pemakaian |
|---|---|
| `Card` / wrapper card internal | KPI, panel watchlist, panel kas |
| `Table` MUI / `@tanstack/react-table` renderer MUI | Rekap reseller, stok, paket jadi |
| `Chip`, `Alert`, atau `StatusBadge` | Status periode, status lunas, status kirim |
| `Button` | Quick action, refresh, link halaman |
| `Select`, `TextField select`, `MenuItem` | Pilih periode, filter status |
| `TextField` | Search reseller |
| `Tabs` + `Tab` | Stok Barang / Paket Jadi, Audit Log / Riwayat Koreksi |
| `Skeleton` | Loading state |
| `Snackbar` + `Alert` atau wrapper internal | Error refetch atau aksi navigasi gagal |

Komponen custom yang disarankan:

| Komponen | Fungsi |
|---|---|
| `AdminPageHeader` | Header gradient + quick action |
| `DashboardKpiCard` | Kartu angka utama |
| `OperationalWatchlist` | Alert operasional |
| `ResellerProgressPanel` | Progress setoran reseller |
| `CashPositionPanel` | Ringkasan saldo kas |
| `StockRiskPanel` | Stok barang dan paket jadi |
| `RecentActivityFeed` | Aktivitas terbaru |

---

## 17. Data Mapping

| Area UI | Sumber Utama |
|---|---|
| KPI utama dan pendukung | `get_dashboard_admin(periode_id)` |
| Pesanan konsumen | `v_ringkasan_pesanan_konsumen` |
| Watchlist pesanan | `v_pesanan_perlu_perhatian` |
| Progres reseller | `v_ringkasan_reseller` |
| Rekap reseller | `v_ringkasan_reseller` |
| Stok barang kritis | `v_stok_barang` |
| Paket perlu packing | `v_stok_paket_jadi` |
| Posisi kas | `v_saldo_kas` |
| Detail pesanan belum kirim | `v_detail_pesanan_aktif` atau output dashboard |
| Pembagian paket | `v_pembagian_reseller` |
| Audit admin | `get_laporan_audit_koreksi` (wrapper `v_riwayat_koreksi` / `v_audit_log`) |
| Aktivitas terbaru | View/RPC tambahan jika dibutuhkan |

Query dashboard wajib diambil berdasarkan `periode_id`.

Frontend hanya boleh melakukan format tampilan:

- format rupiah
- format tanggal
- pembulatan persentase tampilan
- pewarnaan status

Frontend tidak boleh menghitung ulang target, komisi, tabungan, stok, saldo kas, atau status lunas.

---

## 18. Copy UI

Gunakan istilah yang konsisten:

| Konsep | Label UI |
|---|---|
| total nilai paket | Nilai Paket |
| uang konsumen masuk ke reseller | Dikumpulkan |
| uang reseller masuk pusat | Disetor Pusat |
| dikumpulkan belum disetor | Belum Disetor |
| kewajiban akhir reseller | Nilai Akhir Paket |
| sisa kewajiban reseller ke pusat | Sisa Setor Pusat |
| kas eksternal | Kas Masuk |
| detail pesanan belum dikirim | Belum Kirim |

Hindari teks panjang yang menjelaskan cara kerja sistem di dalam dashboard. Penjelasan detail cukup di tooltip atau dokumentasi.

---

## 19. Acceptance Criteria

Dashboard Admin dianggap siap direview jika:

- Layout mengikuti sidebar gelap, header atas, dan konten card-based.
- KPI utama langsung terlihat tanpa scroll pada desktop umum.
- Semua angka dashboard berasal dari query contract.
- Semua query memakai `periode_id`.
- Tidak ada formula bisnis kompleks di frontend.
- Watchlist menampilkan minimal reseller belum lunas, detail pesanan belum kirim, stok perlu belanja, dan paket perlu packing.
- Watchlist menampilkan pesanan belum final/perlu perhatian dan pembagian belum diserahkan.
- Rekap reseller memakai pagination/filter server-side.
- Loading, empty, error, dan refresh state tersedia.
- Copy UI memakai Bahasa Indonesia.
- Tidak ada browser alert.
- Tidak ada elemen visual dengan border tebal, shadow berat, `rounded-lg`, atau radius lebih besar.

---

## 20. Catatan Implementasi Untuk AI Coder

Sebelum membuat halaman Dashboard Admin, AI Coder wajib membaca:

- `docs/prd.md`
- `docs/implementation_guardrails.md`
- `docs/frontend_architecture.md`
- `docs/frontend_component_contracts.md`
- `docs/query_contracts.md`
- `docs/schema_mapping.md`
- `docs/ui/admin_dashboard_uiux.md`

Dashboard boleh dibangun setelah query `get_dashboard_admin` dan view pendukung tersedia. Jika view pendukung belum siap, tampilkan state kosong yang jujur, bukan data dummy yang terlihat seperti data produksi.
