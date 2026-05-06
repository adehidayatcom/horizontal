# Modernize Shell Guide
## Paket Lebaran Mumpuni

Dokumen ini menjelaskan cara memakai shell `Modernize` sebagai fondasi UI proyek `horizontal`.

Dokumen ini bukan kontrak arsitektur utama. Sumber kebenaran frontend tetap berada di:

- `docs/frontend_architecture.md`
- `docs/frontend_component_contracts.md`
- `docs/navigation_and_period_setup_ui.md`

Peran dokumen ini adalah menjelaskan apa yang dipertahankan dari shell Modernize dan bagaimana shell itu dipakai oleh modul bisnis proyek.

---

## 1. Peran Shell Modernize

Shell Modernize dipakai sebagai fondasi untuk:

- theme provider
- customizer global
- layout container
- header
- navigation horizontal
- sidebar
- dark mode
- fluid vs contained layout
- responsive layout behavior

Shell ini tidak dipakai sebagai sumber domain bisnis.

Yang termasuk domain bisnis tetap dibangun khusus untuk proyek ini:

- halaman admin
- halaman reseller
- komponen form dan tabel domain
- integrasi Supabase
- rules akses berbasis role dan periode

---

## 2. Prinsip Adopsi Shell

- Pertahankan capability global yang sudah matang dari shell.
- Jangan mengikat route bisnis ke nama menu demo template.
- Jangan menjadikan halaman demo sebagai sumber kebenaran produk.
- Gunakan struktur dan pola shell untuk mempercepat implementasi, bukan untuk membawa scope demo ke proyek inti.

---

## 3. Area Shell yang Dipertahankan

### Theme System

Pertahankan:

- provider theme
- palet warna dan token dasar
- typography setup
- shadow, shape, spacing, dan component overrides

Yang boleh disesuaikan:

- branding proyek
- warna utama dan status
- metadata title/description
- default appearance untuk admin dan reseller jika dibutuhkan

### Customizer

Pertahankan customizer sebagai pusat kontrol untuk:

- dark mode
- navigation mode
- layout width
- direction dan preferensi layout lain jika dipakai

Aturan:

- behavior ini harus tetap dikelola secara global
- jangan membuat toggle liar per halaman yang mem-bypass customizer

### Layout Engine

Pertahankan:

- root layout pattern
- dashboard layout pattern
- header wrapper
- navigation wrapper
- content container pattern

Yang diubah:

- isi menu
- label
- route mapping
- akses per role

---

## 4. Area Template yang Tidak Menjadi Acuan Produk

Yang tidak boleh dianggap sebagai kontrak proyek:

- halaman demo apps seperti ecommerce, invoice, blog, chat, dan sejenisnya
- mock API bawaan template
- sample data presentational yang tidak mewakili domain proyek
- penamaan modul demo yang tidak relevan dengan bisnis

Hal-hal itu boleh dipakai sebagai referensi pola teknis sementara, tetapi bukan sebagai struktur produk final.

---

## 5. Mapping Shell ke Kebutuhan Proyek

### Admin

Admin bersifat desktop-first dan memanfaatkan shell untuk:

- navigation yang lebih lebar
- tabel data besar
- filter kompleks
- halaman dashboard dan operasional

Mode yang paling masuk akal untuk admin:

- sidebar mode untuk kerja harian yang banyak modul
- horizontal mode bila dibutuhkan untuk demonstrasi atau kebutuhan tertentu

### Reseller

Reseller bersifat mobile-first dan memanfaatkan shell untuk:

- layout yang ringan
- aksi cepat
- daftar dan form ringkas
- navigasi yang tidak membebani layar kecil

Aturan:

- halaman reseller tidak boleh menjadi turunan langsung dari asumsi layout desktop admin
- jika satu shell dipakai bersama, detail komponen tetap harus role-aware

---

## 6. Aturan Navigation

Navigation proyek harus dibangun dari struktur bisnis resmi, bukan dari menu demo.

Prinsip:

- menu admin dan reseller dipisah
- label menu harus mengikuti bahasa domain
- route harus mengikuti kontrak di `frontend_architecture.md`
- urutan menu harus mendukung alur kerja operasional, bukan sekadar meniru template

Contoh kelompok menu admin:

- dashboard
- periode
- reseller
- paket
- barang
- order
- belanja
- packing
- pengiriman
- laporan

Contoh kelompok menu reseller:

- dashboard
- program
- order
- setoran
- profil

---

## 7. Aturan Dark Mode dan Layout Width

Dark mode dan fluid layout adalah capability shell, bukan fitur domain.

Aturan:

- implementasi tetap global
- style komponen bisnis harus kompatibel dengan kedua mode
- tabel, card, badge, dan form tidak boleh pecah saat mode berubah
- keputusan visual domain tidak boleh merusak konsistensi theme global

---

## 8. Aturan Integrasi Komponen Bisnis

Komponen bisnis dibangun di atas shell dengan pembagian:

- shell component untuk struktur global
- feature component untuk domain admin dan reseller
- shared component untuk elemen lintas halaman

Jangan:

- mengedit komponen shell hanya untuk menyelesaikan satu kebutuhan halaman sempit
- menaruh logika bisnis di komponen theme/layout
- membuat duplikasi shell baru padahal cukup extend pola yang ada

---

## 9. Kapan Shell Boleh Diubah

Perubahan shell layak dilakukan jika:

- menyelesaikan kebutuhan lintas banyak halaman
- memperbaiki konsistensi global
- mendukung mode horizontal/sidebar/dark mode/fluid layout secara umum
- tidak mengikat shell ke satu modul bisnis spesifik

Perubahan shell tidak layak jika:

- hanya untuk menyiasati masalah satu halaman
- hanya memindahkan kerumitan dari komponen bisnis ke komponen global
- membuat perilaku customizer menjadi tidak konsisten

---

## 10. Ringkasan Praktis

Shell Modernize di proyek ini berfungsi sebagai:

- fondasi visual
- fondasi layout
- fondasi state layout global

Shell Modernize bukan:

- kontrak domain
- kontrak route bisnis
- kontrak data
- kontrak validasi

Dengan batas ini, shell tetap berguna sebagai percepatan implementasi tanpa mengaburkan arsitektur produk.
