# Reseller Mobile UI/UX Spec
## Paket Lebaran Mumpuni

Dokumen ini menjadi acuan desain area Reseller mobile/PWA. Fokusnya adalah alur kerja harian
reseller di HP: kelola konsumen, kelola `pesanan_konsumen`, catat `setoran_konsumen`, setor ke
pusat, dan pantau progres item pesanan.

Route dan state periode global wajib mengikuti `docs/frontend/navigation_and_period_setup_ui.md`.

Dokumen ini bukan instruksi implementasi kode.

---

## 1. Tujuan Area Reseller

Area Reseller harus membantu reseller bekerja cepat di lapangan.

Prioritas utama:

1. Input setoran konsumen secepat mungkin.
2. Pencarian konsumen cepat walau data banyak.
3. Reseller selalu tahu saldo terkumpul dan kewajiban setor pusat.
4. Pesanan konsumen, item paket, dan progres cicilan mudah dipantau.
5. Tampilan tetap nyaman dipakai satu tangan di layar HP.

---

## 2. Konteks Pengguna

Reseller kemungkinan memakai aplikasi dalam kondisi:

- Berdiri atau melayani konsumen langsung.
- Jaringan seluler tidak selalu stabil.
- Sering mengulang aksi setoran dalam jumlah banyak.
- Membutuhkan angka sisa bayar yang jelas sebelum menerima uang.
- Membutuhkan bukti bahwa transaksi sudah tersimpan.

Karena itu UI reseller harus mengurangi langkah, menghindari form panjang, dan tidak
menyembunyikan status penting.

---

## 3. Prinsip Desain

1. Mobile-first, bukan versi kecil dari dashboard admin.
2. Aksi setoran konsumen harus maksimal 3 langkah dari layar utama.
3. Search konsumen harus selalu mudah dijangkau.
4. Input uang memakai format rupiah yang jelas.
5. Tombol aksi utama sticky di bawah form.
6. Tap target minimal 44px.
7. Input mobile minimal 16px agar tidak memicu zoom browser.
8. Tidak ada horizontal scroll di halaman reseller.
9. Semua error bisnis tampil dalam Bahasa Indonesia.
10. Transaksi uang tidak boleh disimpan offline queue secara default.

Catatan offline:

- Cache daftar konsumen dan ringkasan terakhir boleh dipakai untuk mempercepat buka halaman.
- Submit setoran, perubahan pesanan, finalisasi pesanan, dan setor pusat tetap harus online kecuali
  owner menyetujui desain offline queue terpisah.

---

## 4. Struktur Navigasi

Gunakan bottom navigation 5 tab.

| Tab | Fungsi |
|---|---|
| Beranda | Ringkasan dan prioritas tagihan |
| Konsumen | Daftar konsumen dan detail pesanan |
| Setor | Input setoran konsumen |
| Pesanan | Daftar `pesanan_konsumen` milik reseller |
| Akun | Profil, setor pusat, riwayat, logout |

Tab `Setor` adalah aksi utama. Secara visual boleh lebih menonjol, tetapi tetap memakai bentuk
konsisten dengan design system, bukan tombol dekoratif yang terlalu besar.

Setor ke pusat tidak menjadi tab utama karena frekuensinya lebih rendah dari setoran konsumen.
Aksi ini ditempatkan di Beranda dan Akun.

Fase awal reseller fokus pada:

- KPI ringkas
- konsumen prioritas
- input setoran konsumen
- setor pusat
- pesanan konsumen
- akun

Riwayat transaksi gabungan lintas modul bukan requirement fase awal.

---

## 5. Layout Dasar Mobile

### Header Sticky

Header atas berisi:

- Nama reseller.
- Kode reseller.
- Periode aktif.
- Status sinkronisasi kecil: `Online`, `Offline`, `Menyinkronkan`.

Jika reseller belum di-approve, header menampilkan status `Menunggu Approval`.

### Konten

Konten memakai background `slate-50` dan card putih sederhana. Jarak bawah halaman harus cukup
agar tidak tertutup bottom nav.

### Bottom Nav

Bottom nav selalu terlihat di area reseller setelah login.

Aturan:

- Icon lucide + label pendek.
- Tab aktif memakai warna teal.
- Tinggi cukup untuk tap nyaman.
- Tidak menutupi tombol simpan sticky pada form.

---

## 6. Wireframe Mobile

### Beranda

```txt
+--------------------------------+
| RSL-023 - Siti                 |
| Periode 2026 - Online          |
+--------------------------------+
| [Dikumpulkan] [Belum Disetor]  |
| [Sisa Pusat]  [Konsumen Lunas] |
+--------------------------------+
| Input Setoran                  |
| [Search konsumen...]           |
| Konsumen prioritas             |
| - Ani      Sisa Rp 120.000     |
| - Budi     Sisa Rp 300.000     |
| - Citra    Lunas               |
+--------------------------------+
| Beranda Konsumen Setor Pesanan Akun |
+--------------------------------+
```

### Form Setoran (Ledger Payment Grid)

```txt
+--------------------------------+
| Catat Setoran Hari Ini          |
|--------------------------------|
| Ani     (Sisa: Rp 300k)        |
| Rp [   50.000 ]                |
|--------------------------------|
| Budi    (Sisa: Rp 100k)        |
| Rp [          ]                |
|--------------------------------|
| Citra   (Sisa: Rp 500k)        |
| Rp [  100.000 ]                |
+--------------------------------+
| [Simpan 2 Setoran (Rp 150k)]    |
+--------------------------------+
```

### Setor Pusat (FinTech Wallet UI)

```txt
+--------------------------------+
| UANG DI TANGAN ANDA            |
| Rp 2.000.000                   |
| [||||||||||||||||      ] 80%   |
| (Batas aman: Rp 2.500.000)     |
+--------------------------------+
| Mau setor berapa?              |
| [Semua] [500rb] [1 Juta]       |
| Rp [ 2.000.000               ] |
|--------------------------------|
| Ke rekening: [ BCA Pusat v]    |
| Upload Bukti: [ + Foto ]       |
+--------------------------------+
| [Kirim Setoran]                 |
+--------------------------------+
```

---

## 7. Beranda Reseller

Beranda bukan sekadar dashboard angka. Beranda harus langsung membantu reseller menagih dan
menginput setoran.

### KPI Ringkas

Sumber utama: `get_dashboard_reseller(periode_id)`.

| Kartu | Sumber Data |
|---|---|
| Dikumpulkan | `total_dikumpulkan` |
| Belum Disetor | `saldo_belum_disetor` |
| Sisa Setor Pusat | `sisa_setor_pusat` |
| Konsumen Lunas | `jumlah_konsumen_lunas` dari `jumlah_konsumen` |

KPI lain boleh tampil lebih kecil:

- `jumlah_pesanan_aktif`
- `jumlah_item_pesanan`
- `sisa_tabungan`
- `sisa_komisi`
- `poin`

### Area Input Setoran Cepat

Di bawah KPI, tampilkan:

- Search konsumen.
- Daftar konsumen prioritas.
- Sort default `sisa_bayar DESC`, lalu `nama_konsumen ASC`.

Klik konsumen membuka form setoran full-screen atau bottom sheet tinggi.

Dengan struktur ini, flow harian menjadi:

1. Tap konsumen.
2. Ketik nominal.
3. Tap simpan.

---

## 8. Input Setoran Konsumen

Ini adalah workflow paling penting.

Sumber daftar: `v_ringkasan_konsumen` atau `v_ringkasan_pesanan_konsumen`.

RPC submit: `buat_setoran_konsumen`.

Setoran masuk ke `pesanan_konsumen`. Konsumen selalu dianggap sudah punya pilihan paket karena
target tagihan berasal dari `detail_pesanan_konsumen`.

### Daftar Konsumen Untuk Setoran

Setiap item list menampilkan:

- Nama konsumen.
- Nomor telepon jika ada.
- Target tagihan.
- Total bayar.
- Sisa bayar.
- Status pesanan.
- Status finalisasi.
- Progress bar bayar.
- Badge `LUNAS` atau `BELUM`.

Konsumen `LUNAS` boleh tetap terlihat, tetapi default filter sebaiknya `Belum Lunas`.

### Search

Search harus sticky di atas daftar.

Aturan:

- Debounce input.
- Search berdasarkan nama dan telepon.
- Jika data lokal tersedia, boleh tampilkan hasil cache dulu lalu refetch.
- Empty state: `Konsumen tidak ditemukan`.

### Form Setoran (Ledger Payment Grid)

Menggunakan komponen `LedgerPaymentGrid`. Reseller menagih layaknya mengisi spreadsheet di mobile.

Aturan UI:
- Menampilkan daftar nama konsumen yang belum lunas sebagai baris tabel/list.
- Sel input nominal langsung terbuka untuk setiap konsumen.
- Reseller mengetik nominal, menekan `Enter` di keyboard HP, fokus berpindah ke baris di bawahnya secara mulus.
- Jika ada 5 input yang terisi, *Floating Action Button* di bawah akan berkata: `Simpan 5 Setoran (Total Rp xxx.xxx)`.
- Mengeliminasi form per-konsumen yang memakan waktu.

Tombol simpan:
- Sticky di bawah (Floating).
- Disabled saat tidak ada sel nominal yang terisi.
- Saat loading teks menjadi `Menyimpan 5 Data...`.

Setelah sukses:

- Toast: `Setoran Rp {nominal} untuk {nama_konsumen} berhasil disimpan`.
- Form tertutup.
- Daftar konsumen dan KPI refetch.
- Fokus kembali ke search/list agar input berikutnya cepat.

Jika gagal melebihi sisa:

- Tampilkan pesan inline: `Nominal melebihi sisa tagihan`.
- Tampilkan sisa bayar terbaru jika tersedia dari RPC.

---

## 9. Kelola Konsumen

Halaman Konsumen dipakai untuk tambah, edit, membuat pesanan, dan melihat progres konsumen.

Sumber utama: `v_ringkasan_konsumen`.

### Daftar Konsumen

Filter:

- Semua
- Belum Lunas
- Lunas

Sort default:

1. `sisa_bayar DESC`
2. `nama_konsumen ASC`

Card konsumen menampilkan:

- Nama.
- Telepon.
- Jumlah item pesanan.
- Total target.
- Total bayar.
- Sisa bayar.
- Persentase bayar.

### Detail Konsumen

Detail menampilkan:

- Ringkasan bayar.
- Pesanan periode aktif beserta `detail_pesanan_konsumen`.
- Status `Belum Final` atau `Sudah Final`.
- Daftar item pesanan aktif.
- Riwayat setoran konsumen.
- Aksi: `Input Setoran`, `Buat Pesanan`, `Finalisasi Pesanan`, `Edit Konsumen`.

Jika konsumen belum punya `pesanan_konsumen`, tampilkan aksi utama `Buat Pesanan`.
Jika pesanan belum final, tampilkan aksi `Finalisasi Pesanan`.

---

## 10. Kelola Pesanan

Dalam model final, reseller tidak membuat entitas item pesanan terpisah di luar header utama.
Reseller membuat dan mengelola `pesanan_konsumen`, lalu mengisi atau menyesuaikan
`detail_pesanan_konsumen`.

RPC utama:

- `buat_pesanan_konsumen`
- `ubah_detail_pesanan_konsumen`
- `finalisasi_pesanan_konsumen`

Pesanan bisa dibuat dari:

- Halaman Konsumen Detail.
- Tab Pesanan.
- Quick action Beranda.

Menggunakan komponen `PosCartLayout` (*Point of Sale*).

Field utama di area "Keranjang" (Kanan/Bottom Sheet):

| Field | Aturan UI |
|---|---|
| Konsumen | Wajib, search konsumen |
| Daftar Paket | Ditambahkan dengan sekali tap dari Katalog di sebelah kiri/atas |
| Qty | Penyesuaian instan dengan tombol `+` / `-` tanpa modal |
| Total Tagihan | Real-time kalkulasi sebelum submit |

Daftar paket harus mudah dicari karena jumlah paket bisa 500+ per periode.

Card paket minimal menampilkan:

- Kode paket.
- Nama paket.
- Harga/nilai paket snapshot.
- Kategori.
- Badge jika perlu packing.

Sebelum finalisasi, reseller harus bisa meninjau ulang daftar item aktif yang sedang menjadi
acuan target tagihan konsumen.

Setelah pesanan sukses dibuat atau diperbarui:

- Toast sesuai `docs/contracts/business_contracts.md`.
- Arahkan ke detail konsumen atau detail pesanan.

Jika total target hasil perubahan lebih kecil dari setoran yang sudah masuk, tampilkan:
`Total target pesanan lebih kecil dari setoran yang sudah masuk`.

Reseller tidak boleh melihat aksi admin seperti BATAL paksa, TERHENTI manual lintas reseller, atau
ubah status kirim.

---

## 11. Daftar Pesanan Reseller

Sumber: `v_ringkasan_pesanan_konsumen` dan `v_detail_pesanan_aktif`.

Filter header:

- Semua
- Belum Final
- Sudah Final
- Perlu Perhatian

Filter item:

- Semua
- Aktif
- Terhenti
- Batal
- Belum Kirim
- Sudah Kirim

Card/header pesanan menampilkan:

- Nama konsumen.
- Target tagihan.
- Total bayar.
- Sisa bayar.
- Status pesanan.
- Status finalisasi.

Detail item menampilkan:

- Nama paket.
- Qty.
- Status item.
- Status kirim.
- Nilai item.

Item `TERHENTI` harus terlihat jelas, tetapi tidak memakai bahasa yang menyalahkan reseller.

Label yang dipakai:

- `Aktif`
- `Terhenti`
- `Belum Final`
- `Sudah Final`
- `Belum Kirim`
- `Sudah Kirim`

---

## 12. Setor Ke Pusat

Sumber ringkasan: `get_dashboard_reseller` atau `v_ringkasan_reseller`.

RPC submit: `buat_setoran_pusat`.

Halaman ini harus menampilkan sebelum form:

- Total dikumpulkan.
- Total disetor pusat.
- Saldo belum disetor.
- Sisa setor pusat.

Field:

| Field | Aturan UI |
|---|---|
| Nominal | Wajib, format rupiah |
| Metode | `CASH` / `TRANSFER` |
| Akun Kas Tujuan | Wajib |
| Tanggal | Default hari ini |
| Keterangan | Opsional |

Validasi UI awal:

- Nominal tidak boleh lebih dari saldo belum disetor.
- Nominal tidak boleh lebih dari sisa setor pusat.

Validasi final tetap di RPC.

Setelah sukses:

- Toast: `Setoran Rp {nominal} ke {nama_kas} berhasil dicatat`.
- Ringkasan saldo refetch.

---

## 13. Riwayat

Riwayat membantu reseller mengecek transaksi yang sudah dicatat.

Jenis riwayat:

- Setoran konsumen.
- Setoran ke pusat.
- Perubahan pesanan.

Pada fase awal, riwayat tetap modular. Artinya layar riwayat tidak bergantung pada satu feed
gabungan lintas semua transaksi reseller.

Aturan:

- Filter tanggal.
- Search konsumen untuk riwayat setoran konsumen.
- Pagination atau infinite scroll.
- Tampilkan status sinkronisasi jika data dari cache.

Jika kontrak query riwayat belum dibuat, AI Coder wajib menambahkannya ke `docs/contracts/query_contracts.md`
sebelum implementasi halaman riwayat.

---

## 14. State Khusus

### Menunggu Approval

Jika reseller masih `PENDING`, tampilkan halaman khusus:

- Judul: `Pendaftaran sedang ditinjau`.
- Deskripsi singkat.
- Tombol refresh status.
- Tombol logout.

Tidak boleh menampilkan menu transaksi.

### Tidak Ada Periode Aktif

Judul: `Belum ada periode aktif`.

Isi:

- Reseller belum bisa input pesanan atau setoran.
- Tombol refresh.

### Offline

Saat offline:

- Daftar konsumen cache boleh tetap tampil.
- Tombol submit transaksi disabled.
- Tampilkan banner kecil: `Offline. Transaksi membutuhkan koneksi internet.`

### Loading

- Pakai skeleton untuk KPI dan list.
- Jangan tampilkan angka 0 ketika data masih loading.

### Error

Pesan harus Bahasa Indonesia.

Contoh:

- `Data belum bisa dimuat. Coba lagi.`
- `Setoran belum bisa disimpan. Periksa koneksi internet.`
- `Periode aktif tidak ditemukan.`

---

## 15. PWA dan Performa

Kebutuhan PWA:

- Bisa di-install ke home screen.
- App shell cepat terbuka.
- Cache aset statis.
- Cache daftar konsumen terakhir untuk akses cepat.
- Pull-to-refresh pada daftar utama.

Target performa:

- Halaman setoran load < 2 detik di jaringan 3G.
- Search konsumen < 500 ms setelah debounce.
- Submit setoran memberi feedback loading langsung.

Larangan default:

- Jangan membuat offline queue untuk transaksi uang tanpa keputusan owner.
- Jangan menyimpan data sensitif di local storage tanpa kebutuhan jelas.

---

## 16. Komponen UI

Gunakan `Material-UI v7` dan wrapper internal proyek sebagai base.

Ikuti kontrak aktif berikut:

- `docs/frontend/frontend_architecture.md`
- `docs/frontend/frontend_component_contracts.md`
- `docs/frontend/component_patterns.md`

| Komponen | Pemakaian |
|---|---|
| `Button` | Aksi simpan, tambah, refresh |
| `TextField` | Search, nominal, catatan |
| `Select`, `TextField select`, `MenuItem` | Metode, filter, akun kas |
| `Drawer` | Form mobile atau pilihan cepat |
| `Dialog` | Konfirmasi ringan jika dibutuhkan |
| `Card` | Ringkasan dan item list |
| `Chip`, `Alert`, atau `StatusBadge` | Status lunas, final, kirim, item |
| `Autocomplete` atau komponen search internal | Search konsumen/paket |
| `Tabs` + `Tab` | Filter ringkas |
| `Skeleton` | Loading |
| `Snackbar` + `Alert` atau wrapper internal | Sukses/gagal |

Komponen custom yang disarankan:

| Komponen | Fungsi |
|---|---|
| `ResellerLayout` | Header sticky + bottom nav |
| `BottomNav` | Navigasi mobile |
| `ResellerSummaryCards` | KPI beranda |
| `KonsumenSearchList` | Search dan daftar konsumen |
| `KonsumenCard` | Item konsumen mobile |
| `FormSetoran` | Form setoran 3 langkah |
| `PesananCard` | Item header pesanan mobile |
| `DetailPesananList` | Daftar item paket dalam satu pesanan |
| `SetorPusatForm` | Form setor pusat |
| `OfflineBanner` | Status koneksi |

---

## 17. Data Mapping

| Area UI | Sumber / RPC |
|---|---|
| Beranda ringkasan | `get_dashboard_reseller(periode_id)` |
| Pesanan konsumen | `v_ringkasan_pesanan_konsumen`, `buat_pesanan_konsumen`, `ubah_detail_pesanan_konsumen`, `finalisasi_pesanan_konsumen` |
| Ringkasan reseller detail | `v_ringkasan_reseller` |
| Daftar konsumen | `v_ringkasan_konsumen` |
| Input setoran konsumen | `buat_setoran_konsumen` |
| Daftar item pesanan | `v_detail_pesanan_aktif` |
| Setor ke pusat | `buat_setoran_pusat` |
| Riwayat | query tambahan per modul jika belum tersedia |

Semua query wajib berdasarkan `periode_id`.

Untuk reseller, `no_reseller` harus berasal dari auth/profile, bukan input bebas dari UI.

Frontend hanya boleh melakukan:

- format rupiah
- format tanggal
- filter tampilan ringan
- warna status
- optimistic refetch ringan setelah sukses

Frontend tidak boleh menghitung ulang target, status lunas, saldo belum disetor, komisi,
tabungan, atau sisa setor pusat sebagai sumber kebenaran.

---

## 18. Copy UI

Gunakan istilah yang konsisten.

| Konsep | Label UI |
|---|---|
| total setoran konsumen | Dikumpulkan |
| uang belum disetor ke pusat | Belum Disetor |
| sisa kewajiban ke pusat | Sisa Setor Pusat |
| pembayaran konsumen | Setoran Konsumen |
| penyetoran reseller ke pusat | Setor Pusat |
| konsumen belum lunas | Belum Lunas |
| konsumen lunas | Lunas |
| pesanan belum final | Belum Final |
| pesanan final | Sudah Final |

Contoh pesan:

- `Setoran berhasil disimpan`.
- `Nominal melebihi sisa tagihan`.
- `Transaksi membutuhkan koneksi internet`.
- `Konsumen belum memiliki pesanan`.

---

## 19. Acceptance Criteria

Area Reseller dianggap siap direview jika:

- Bottom nav mobile tersedia dan mudah dipakai.
- Header sticky menampilkan reseller dan periode aktif.
- Input setoran konsumen bisa selesai dalam 3 langkah dari layar utama.
- Search konsumen sticky, cepat, dan sorted by sisa terbesar.
- Form setoran memakai input rupiah, metode, tanggal, dan tombol simpan sticky.
- Konsumen memiliki pilihan paket sejak awal melalui `pesanan_konsumen` dan `detail_pesanan_konsumen`.
- UI membedakan pesanan `Belum Final` dan `Sudah Final`.
- Setor pusat menampilkan total dikumpulkan, total disetor, saldo belum disetor, dan sisa setor
  pusat.
- Semua submit transaksi memanggil RPC dari `docs/contracts/business_contracts.md`.
- Semua ringkasan memakai view/RPC dari `docs/contracts/query_contracts.md`.
- Error bisnis tampil dalam Bahasa Indonesia.
- Beranda fase awal tidak bergantung pada unified history feed reseller.
- Tidak ada browser alert.
- Input mobile minimal 16px.
- Tap target minimal 44px.
- Tidak ada horizontal scroll di halaman reseller.
- Offline state jelas dan tidak mengizinkan submit transaksi uang.

---

## 20. Catatan Implementasi Untuk AI Coder

Sebelum membuat area Reseller, AI Coder wajib membaca:

- `docs/product/prd.md`
- `docs/quality/implementation_guardrails.md`
- `docs/frontend/frontend_architecture.md`
- `docs/frontend/frontend_component_contracts.md`
- `docs/contracts/business_contracts.md`
- `docs/contracts/query_contracts.md`
- `docs/contracts/schema_mapping.md`
- `docs/ui/reseller_uiux.md`

Jika kebutuhan UI menemukan query yang belum ada, AI Coder wajib memperbarui
`docs/contracts/query_contracts.md` terlebih dahulu sebelum implementasi.
