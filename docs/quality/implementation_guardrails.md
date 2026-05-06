# Implementation Guardrails
## Paket Lebaran Mumpuni

Dokumen ini menetapkan pagar implementasi agar agen dan developer tidak salah menafsir kebutuhan proyek.

Dokumen ini dipakai bersama kontrak bisnis, kontrak frontend, dan dokumen UI reference yang aktif di folder `docs/`.

---

## 1. Urutan Otoritas Dokumen

Jika ada konflik, gunakan urutan berikut:

1. `docs/product/prd.md`
2. `docs/contracts/business_contracts.md`
3. `docs/contracts/schema_mapping.md`
4. `docs/contracts/query_contracts.md`
5. `docs/contracts/rls_matrix.md`
6. `docs/product/edge_cases.md`
7. `docs/frontend_architecture.md`
8. `docs/frontend_component_contracts.md`
9. `docs/navigation_and_period_setup_ui.md`
10. `docs/ui/admin_dashboard_uiux.md` dan `docs/ui/reseller_uiux.md`
11. `docs/component_patterns.md`
12. `docs/definition_of_done.md`

Jika konflik tetap tidak bisa diselesaikan, pekerjaan harus dihentikan sampai ada keputusan owner.

---

## 2. Definisi yang Tidak Boleh Ditafsir Ulang

| Istilah | Definisi Final |
|---|---|
| Pesanan konsumen | Header tagihan/cicilan konsumen dalam satu periode |
| Detail pesanan konsumen | Paket yang dipesan di dalam satu pesanan konsumen |
| Target berjalan | Target yang dipakai untuk validasi setoran saat ini |
| Target tagihan | Snapshot total item pesanan yang masih dihitung di `pesanan_konsumen.target_tagihan_snapshot` |
| Detail pesanan aktif | `status_item != 'BATAL'` |
| Koreksi | Transaksi pembalik atau penyesuaian melalui mekanisme resmi, bukan edit row bebas |
| Audit | Catatan wajib untuk aksi penting, koreksi, dan revisi |

---

## 3. Larangan Implementasi

Jangan:

- membuat setoran konsumen langsung ke item detail pesanan
- menjadikan item detail final sebagai syarat awal konsumen mulai menabung
- memperlakukan `target_nominal` sebagai angka bebas tanpa daftar pilihan paket aktif
- menghitung formula bisnis final hanya di frontend
- insert langsung ke tabel transaksi kompleks dari frontend
- mengubah transaksi uang atau stok dengan update langsung tanpa mekanisme resmi
- memakai service role key di browser
- menampilkan error teknis library atau database langsung ke user
- membuat dummy data terlihat seperti data produksi
- menganggap halaman demo template sebagai deliverable produk
- melakukan fetch read sederhana (contoh: list) via Supabase browser client sebelum ada instruksi resmi (defaultkan semua read via internal API route untuk saat ini sesuai BQ-003)

---

## 4. Checklist Sebelum Mengerjakan Fitur

Sebelum mulai menulis kode, pastikan:

1. Fitur ini mengacu ke modul mana di `prd.md`?
2. Aturan bisnisnya sudah tercermin di `business_contracts.md`?
3. Jika menyentuh data, apakah sumber kebenarannya tabel, view, atau RPC?
4. Jika menyentuh uang, stok, status, atau audit, apakah ada mekanisme resmi yang aman?
5. Apakah halaman atau komponen ini perlu sadar `periode_id`?
6. Apakah role admin dan reseller sudah dibedakan?
7. Apakah route dan navigation mengikuti `navigation_and_period_setup_ui.md`?
8. Apakah UI mengikuti `frontend_architecture.md` dan `frontend_component_contracts.md`?
9. Apakah state loading, empty, error, disabled, dan success sudah direncanakan?
10. Apakah perubahan ini menyentuh shell global atau hanya komponen bisnis?

Jika salah satu jawaban belum jelas, jangan lanjut implementasi penuh.

---

## 5. Guardrail Shell vs Feature

### Perubahan Shell Global

Layak dilakukan jika:

- berdampak lintas banyak halaman
- menyangkut theme, layout, navigation mode, dark mode, atau fluid/container behavior
- memperbaiki konsistensi global

### Perubahan Feature Lokal

Harus tetap lokal jika:

- hanya menyelesaikan satu modul bisnis
- hanya mengubah satu tampilan tabel atau form
- tidak memerlukan perubahan perilaku layout global

Jangan memodifikasi shell global untuk mengatasi masalah kecil yang seharusnya selesai di layer komponen fitur.

---

## 6. Checklist Database

Migration baru wajib:

- bisa dijalankan dari database kosong
- memakai enum atau constraint untuk status penting
- menambahkan index untuk query penting
- mengaktifkan RLS pada tabel operasional
- mengikuti `rls_matrix.md`
- menyediakan mekanisme transaksi resmi untuk aksi kompleks
- menjaga semua transaksi utama membawa konteks periode

---

## 7. Checklist RPC atau Server Boundary

Setiap aksi bisnis penting wajib mendefinisikan:

- aktor
- input
- validasi
- write database
- output
- pesan gagal
- kebutuhan audit
- perilaku saat terjadi data tidak valid

Aksi yang menyentuh uang, stok, status, atau koreksi harus bersifat atomic.

---

## 8. Checklist Query atau Read Model

Read model wajib:

- menerima atau menentukan `periode_id`
- tidak mencampur data lintas periode tanpa alasan eksplisit
- mengecualikan `BATAL` dari formula yang relevan
- tidak memaksa frontend menghitung formula bisnis final

---

## 9. Checklist Frontend

Setiap implementasi frontend wajib:

- memisahkan admin dan reseller dengan jelas
- kompatibel dengan shell `Modernize`
- kompatibel dengan dark mode bila berada dalam shell umum
- memakai Bahasa Indonesia untuk user-facing copy
- menggunakan komponen MUI atau wrapper internal secara konsisten
- tidak menyalin pola demo template mentah-mentah

---

## 10. Safe Commit Discipline

Jangan membuat commit tanpa instruksi owner.

Jika diminta commit:

- pastikan perubahan berada dalam scope yang jelas
- cek `git status`
- jangan campurkan perubahan unrelated
- gunakan pesan commit yang jelas

---

## 11. Laporan Hasil Kerja

Setelah pekerjaan selesai, minimal laporkan:

- scope yang dikerjakan
- file yang diubah
- command yang dijalankan
- hasil quality gate yang sempat dijalankan
- blocker yang masih ada
- risiko tersisa

---

## 12. Prinsip Akhir

Dokumen ini ada untuk menjaga disiplin:

- shell tetap menjadi shell
- domain tetap menjadi domain
- data tetap aman
- UI tetap konsisten
- agen tidak mengarang kontrak baru di luar dokumen resmi
