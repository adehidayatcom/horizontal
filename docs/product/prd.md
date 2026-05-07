# PRD
## Paket Lebaran Mumpuni

Dokumen ini menjadi ringkasan kebutuhan produk utama untuk proyek `horizontal`.

PRD ini bersifat product truth. Dokumen implementasi dan eksekusi harus tunduk pada isi dokumen ini.

---

## 1. Ringkasan Produk

Paket Lebaran Mumpuni adalah sistem manajemen tabungan berjangka berbasis web untuk mengelola penjualan paket Lebaran melalui jaringan reseller.

Masalah utama yang ingin diselesaikan:

- pencatatan tabungan, Pesanan Konsumen, dan setoran reseller sering terpecah dan sulit diaudit
- admin pusat sulit memantau progres operasional satu periode secara utuh
- reseller membutuhkan alur mobile yang cepat untuk setoran, Pesanan Konsumen, dan follow-up konsumen
- perubahan status uang, stok, dan pelunasan harus tetap konsisten antar modul

Inti bisnis:

- konsumen memilih satu atau beberapa paket sejak awal
- reseller membuat `pesanan_konsumen` sebagai header tagihan/cicilan per periode
- reseller mengisi `detail_pesanan_konsumen` sebagai daftar paket yang dipesan
- reseller mengelola konsumen dan menerima setoran
- reseller menyesuaikan detail pesanan jika kebutuhan atau budget berubah
- reseller/admin memfinalkan pesanan saat siap masuk operasional gudang
- reseller menyetor dana ke pusat
- sistem berjalan berdasarkan periode operasional

---

## 2. Pengguna

### Admin

Admin pusat mengelola:

- periode
- reseller
- master paket dan barang
- Pesanan Konsumen
- gudang
- keuangan
- laporan

### Reseller

Reseller lapangan mengelola:

- pendaftaran
- konsumen
- pesanan konsumen
- detail pesanan paket
- finalisasi pesanan
- setoran konsumen
- setor pusat
- progres pelunasan

---

## 3. Prinsip Operasional

- sistem berjalan per `periode`
- hanya satu periode yang aktif secara operasional pada satu waktu
- `pesanan_konsumen` adalah titik awal tagihan/cicilan konsumen
- target tagihan konsumen berasal dari total `detail_pesanan_konsumen` yang masih dihitung
- finalisasi pesanan ditandai oleh `tanggal_final`, bukan status lock terpisah
- item yang tidak masuk budget dapat ditandai `TERHENTI`
- transaksi uang, stok, dan status penting harus terkendali dan dapat diaudit

---

## 4. Modul Utama

### P0

- autentikasi dan otorisasi
- kelola periode
- kelola reseller
- master data paket dan barang
- pesanan konsumen
- detail pesanan konsumen
- setoran konsumen
- setoran pusat
- gudang dasar

### P1

- keuangan lanjutan
- dashboard penuh
- laporan lengkap
- pencairan

Catatan implementasi:

- item P1 boleh hadir bertahap
- gelombang coding pertama cukup menutup dashboard dan laporan operasional inti
- report kaya, export lengkap, dan panel tambahan boleh menyusul

### Di luar scope gelombang coding pertama

- export laporan penuh
- recent activity kaya yang belum punya kontrak query final
- background job nyata untuk precompute watchlist
- unified reseller history feed lintas semua modul

---

## 5. Outcome yang Diharapkan

Sistem harus mampu:

- membantu admin memantau operasional periode berjalan
- membantu reseller bekerja cepat di HP
- menjaga konsistensi transaksi dan ringkasan
- memisahkan data antar reseller
- menyediakan jalur audit dan koreksi

### Success Criteria Minimum

- admin dapat memantau status operasional periode berjalan dari dashboard dan laporan inti
- reseller dapat menjalankan alur utama `konsumen -> pesanan -> detail pesanan -> setor -> finalisasi -> setor pusat` tanpa ambiguity UI
- transaksi penting tunduk ke kontrak dan audit trail yang konsisten
- periodisasi, role guard, dan ringkasan lintas modul tidak saling bertentangan

---

## 6. Prinsip UX

- admin desktop-first
- reseller mobile-first
- status periode harus memengaruhi perilaku halaman
- frontend tidak menjadi sumber kebenaran logic bisnis final

---

## 7. Asumsi dan Constraint

Asumsi:

- hanya satu periode operasional `AKTIF` pada satu waktu
- admin desktop-first dan reseller mobile-first tetap menjadi arah utama
- frontend nyata memakai internal API route untuk read/write operasional

Constraint:

- shell visual tetap memakai `Modernize`
- perubahan historis pada periode `SELESAI` tidak dilakukan lewat CRUD biasa
- first wave menghindari over-engineering seperti background job wajib atau read model gabungan yang belum perlu

---

## 8. External Dependencies

- `Supabase` untuk auth, database, RPC, dan RLS
- `Next.js` App Router untuk frontend dan internal API route
- `Material-UI` dan shell `Modernize` untuk fondasi UI
- `Vitest` untuk unit/integration test
- `Playwright` untuk E2E

---

## 9. Open Questions dan Decision Tracking

Pertanyaan yang masih valid dan keputusan lintas dokumen tidak disimpan di PRD ini. Gunakan:

- `../truth/03-open_questions_register.md`
- `../truth/01-decision_log.md`

---

## 10. Dokumen Turunan

Dokumen ini dibaca bersama:

- `../contracts/business_contracts.md`
- `../contracts/schema_mapping.md`
- `../contracts/query_contracts.md`
- `../contracts/rls_matrix.md`
- `../frontend/frontend_architecture.md`
- `../truth/01-decision_log.md`
