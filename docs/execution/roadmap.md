# Execution Roadmap
## Paket Lebaran Mumpuni

Dokumen ini menjadi roadmap implementasi lintas frontend, backend, dan integrasi.

Dokumen ini adalah panduan urutan kerja. Ia bukan kontrak produk atau arsitektur utama.

Untuk peta hubungan visual lintas dokumen dan modul, lihat:

- `docs/truth/system_maps.md`
- `docs/execution/sprint_control_board.md`
- `docs/execution/agent_prompt_templates.md`

---

## 1. Prinsip Eksekusi

- bangun fondasi dulu, fitur setelahnya
- jaga shell global tetap stabil
- jaga kontrak data tetap konsisten
- prioritaskan alur operasional inti lebih dulu
- hindari pekerjaan besar yang belum siap dependensinya
- gunakan `docs/contracts/integration_contract_pack.md` sebagai pagar kontrak lintas layer
- gunakan `docs/truth/01-decision_log.md` untuk keputusan yang sudah dikunci maupun gap yang sedang dibahas
- gunakan `docs/quality/testing_strategy.md` untuk target verifikasi minimum
- gunakan `docs/execution/sprint_control_board.md` sebagai papan kontrol sprint, packet, dan owner agen
- gunakan `docs/execution/agent_prompt_templates.md` sebagai template prompt resmi untuk Gemini dan Claude

---

## 2. Fase Eksekusi

### Fase 0: Repo dan Dokumen

- normalkan dokumen
- tetapkan package manager
- tetapkan stack aktif
- bersihkan konflik referensi

### Fase 1: Shell dan Struktur

- pisahkan shell inti dari artefak demo `Modernize`
- bersihkan demo `Modernize` secara bertahap setelah dependency inti dipindahkan
- rapikan shell `Modernize`
- tetapkan navigation admin dan reseller
- tetapkan struktur folder target

### Fase 2: Data Foundation

- siapkan Supabase
- buat migration inti
- buat seed minimum
- aktifkan RLS

### Fase 3: Shared Build Layer

- siapkan client/helper data
- siapkan query keys
- siapkan validasi
- siapkan komponen shared minimum

### Fase 4: Core Features

- periode
- reseller
- pesanan
- setoran
- gudang
- keuangan dasar

### Fase 5: Dashboard dan Laporan

- dashboard admin
- dashboard reseller
- laporan inti
- watchlist operasional

### Fase 6: Hardening

- typecheck
- build
- test yang tersedia
- verifikasi role, periode, dan state UI

---

## 3. Dependency Rules

- fitur transaksi tidak mulai sebelum kontrak data dan access model jelas
- dashboard tidak menjadi prioritas sebelum read model utama siap
- perubahan shell besar harus selesai sebelum banyak halaman domain dibangun
- dummy boundary harus tetap dekat dengan shape data akhir agar integrasi tidak mahal

---

## 4. Prioritas Operasional

Urutan modul bisnis yang paling penting:

1. periode
2. reseller
3. pesanan konsumen
4. detail pesanan dan finalisasi
5. setoran konsumen
6. setoran pusat
7. gudang
8. keuangan
9. laporan

---

## 5. Pola Eksekusi Paralel

Perencanaan tetap dipisah:

- `execution/frontend_plan.md`
- `execution/backend_plan.md`
- `execution/task_kit_standard.md`
- `integration_read_model_matrix.md`

Eksekusi dapat berjalan paralel per modul bila kontraknya sudah jelas.

### Paralel per Modul

1. `M2 Periode`
   - backend: schema, RPC, read model periode
   - frontend: daftar periode, setup UI, checklist aktivasi

2. `M3 Master Data`
   - backend: paket, barang, akun kas, reseller, konsumen
   - frontend: form, table, approval, daftar konsumen

3. `M4 Pesanan Konsumen`
   - backend: header pesanan, detail item, target berjalan, finalisasi, status item
   - frontend: pesanan list, detail, finalisasi, detail item pages

4. `M5 Setoran`
   - backend: setoran konsumen, setor pusat, status lunas, ringkasan
   - frontend: search konsumen, form setoran, setor pusat, monitoring

5. `M6-M8 Gudang dan Keuangan`
   - backend: stok, belanja, packing, pengiriman, kas, pencairan
   - frontend: form gudang, table status, kas summary, pencairan screens

6. `M9 Dashboard dan Laporan`
   - backend: read model dashboard/laporan
   - frontend: KPI, watchlist, report screens

Aturan:

- frontend boleh mulai setelah shape kontrak modul cukup jelas
- backend tidak harus menunggu semua UI selesai
- dashboard dan laporan tetap paling akhir

Module plan yang sudah aktif:

- `modules/auth.md`
- `modules/master_periodik.md`
- `modules/periode.md`
- `modules/reseller.md`
- `modules/program_order.md`
- `modules/setoran.md`
- `modules/gudang.md`
- `modules/keuangan.md`
- `modules/dashboard_laporan.md`

---

## 6. Definition of Progress

Sebuah fase dianggap bergerak benar jika:

- dependensinya jelas
- outputnya bisa diverifikasi
- tidak menciptakan kontradiksi baru di dokumen aktif
- tidak memaksa rewrite besar di fase berikutnya
