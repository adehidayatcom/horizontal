# Open Questions Register
## Paket Lebaran Mumpuni

Dokumen ini mengumpulkan pertanyaan yang masih terbuka agar tidak tercecer di banyak execution plan.

Status yang dipakai:

- `open`
- `needs product decision`
- `needs technical decision`
- `deferred`
- `resolved`

---

## 1. Product Questions

| ID | Pertanyaan | Status | Dampak | Dokumen Terkait |
|---|---|---|---|---|
| `PQ-001` | Kapan tepatnya `pesanan_konsumen` berubah ke status `SELESAI` saat periode ditutup? | resolved | dipakai untuk workflow closing dan laporan historis | `program_workflow.md`, `period_workflow.md`, `schema_mapping.md`, `truth/01-decision_log.md` |
| `PQ-002` | Koreksi apa saja yang masih boleh dilakukan pada periode `SELESAI`? | resolved | dipakai untuk rule historis dan audit | `edge_cases.md`, `business_contracts.md`, `period_workflow.md`, `truth/01-decision_log.md` |
| `PQ-003` | Apakah semua laporan P1 wajib hadir di fase coding pertama atau boleh bertahap? | resolved | dipakai untuk sequencing dashboard/laporan | `prd.md`, `modules/dashboard_laporan.md`, `truth/01-decision_log.md` |
| `PQ-004` | Apakah duplikasi data dari periode sebelumnya termasuk scope modul periode awal, atau ditunda ke modul turunan? | deferred | memengaruhi helper setup periode dan scope first wave | `modules/periode.md`, `period_workflow.md` |
| `PQ-005` | Apakah create pesanan awal hanya dilakukan reseller, atau admin juga aktif memakainya sejak fase pertama? | resolved | memengaruhi boundary create pesanan dan CTA admin | `modules/program_order.md`, `ui/reseller_uiux.md`, `truth/01-decision_log.md` |
| `PQ-006` | Apakah create reseller dilakukan oleh admin, self-register, atau keduanya sejak fase awal? | resolved | memengaruhi auth flow, admin flow, dan onboarding reseller | `modules/auth.md`, `modules/reseller.md`, `execution/frontend_plan.md`, `truth/01-decision_log.md` |

---

## 2. UX Questions

| ID | Pertanyaan | Status | Dampak | Dokumen Terkait |
|---|---|---|---|---|
| `UX-001` | Apakah audit/koreksi admin lebih baik satu halaman dengan tab atau dua screen terpisah? | resolved | dipakai untuk route audit admin dan komposisi tab laporan | `ui/admin_dashboard_uiux.md`, `modules/dashboard_laporan.md`, `truth/01-decision_log.md` |
| `UX-002` | Apakah halaman reseller butuh riwayat transaksi gabungan pada fase awal atau cukup status ringkas? | resolved | dipakai untuk menjaga beranda reseller tetap ringan dan tidak bergantung pada histori gabungan | `ui/reseller_uiux.md`, `integration_read_model_matrix.md`, `truth/01-decision_log.md` |
| `UX-003` | Apakah laporan reseller versi admin butuh drill-down langsung ke detail konsumen dari tabel rekap? | deferred | memengaruhi CTA dan navigasi laporan admin | `modules/dashboard_laporan.md` |
| `UX-004` | Apakah UI perlu menyimpan draft pilihan paket sementara sebelum lock, atau cukup selection ephemeral? | resolved | memengaruhi kompleksitas UI edit pesanan reseller | `modules/program_order.md`, `truth/01-decision_log.md` |
| `UX-005` | Apakah watchlist `PERLU_PERHATIAN` pada fase awal perlu aksi admin lengkap, atau cukup visual kandidat? | resolved | memengaruhi action set admin pada watchlist pesanan | `modules/program_order.md`, `ui/admin_dashboard_uiux.md`, `truth/01-decision_log.md` |
| `UX-006` | Apakah riwayat setoran awal perlu dipisah menjadi tab `setoran konsumen` dan `setor pusat`, atau cukup satu daftar campuran berlabel? | resolved | memengaruhi layout halaman riwayat reseller | `modules/setoran.md`, `ui/reseller_uiux.md`, `truth/01-decision_log.md` |

---

## 3. Backend Questions

| ID | Pertanyaan | Status | Dampak | Dokumen Terkait |
|---|---|---|---|---|
| `BQ-001` | Apakah background jobs seperti `watchlist-refresh` benar-benar dibutuhkan di fase awal, atau cukup dihitung on-demand? | resolved | dipakai untuk menahan kompleksitas backend gelombang pertama | `execution/backend_plan.md`, `truth/01-decision_log.md` |
| `BQ-002` | Apakah `akun_kas` memerlukan status aktif/nonaktif eksplisit sejak awal? | resolved | dipakai untuk schema, validation, dan pencairan | `schema_mapping.md`, `modules/master_periodik.md`, `modules/keuangan.md`, `truth/01-decision_log.md` |
| `BQ-003` | Apakah direct Supabase browser client boleh dipakai untuk read sederhana tertentu, atau semua read wajib melewati internal API route? | resolved | dipakai untuk mengunci boundary akses data frontend nyata | `api_integration.md`, `integration_contract_pack.md`, `frontend_architecture.md`, `truth/01-decision_log.md` |
| `BQ-004` | Apakah status `SELESAI` pada pembagian diisi otomatis setelah semua item `DISERAHKAN`, atau tetap aksi manual admin? | resolved | memengaruhi workflow pembagian dan acceptance criteria gudang | `modules/gudang.md`, `truth/01-decision_log.md` |
| `BQ-005` | Apakah pencairan `TABUNGAN` dan `KOMISI` boleh digabung dalam satu transaksi, atau tetap dipisah per jenis? | resolved | memengaruhi contract `pencairan` dan form keuangan | `modules/keuangan.md`, `business_contracts.md`, `truth/01-decision_log.md` |
| `BQ-006` | Apakah mutasi kas lintas periode dilarang sepenuhnya, atau cukup dicatat pada periode aktif saat transaksi dibuat? | resolved | memengaruhi boundary mutasi kas dan saldo periode | `modules/keuangan.md`, `schema_mapping.md`, `truth/01-decision_log.md` |
| `BQ-007` | Apakah perubahan akun kas juga perlu warning saat periode `AKTIF`, atau cukup untuk master periodik yang memengaruhi tagihan/stok? | resolved | memengaruhi UX warning di master periodik dan keuangan | `modules/master_periodik.md`, `modules/keuangan.md`, `truth/01-decision_log.md` |
| `BQ-008` | Apakah `barang_periode` wajib memuat `budget_belanja` di fase awal, atau boleh ditunda? | resolved | memengaruhi field minimum barang periode dan setup periode | `modules/master_periodik.md`, `modules/periode.md`, `truth/01-decision_log.md` |

---

## 4. API Questions

| ID | Pertanyaan | Status | Dampak | Dokumen Terkait |
|---|---|---|---|---|
| `AQ-001` | Apakah admin monitoring setoran akan memakai read model khusus yang terpisah dari query generik? | resolved | dipakai untuk kontrak route admin setoran | `integration_read_model_matrix.md`, `modules/setoran.md`, `query_contracts.md`, `truth/01-decision_log.md` |
| `AQ-002` | Apakah riwayat transaksi reseller akan dibuat satu read model gabungan atau tetap tersebar per modul? | resolved | dipakai untuk menjaga kontrak read reseller tetap modular pada fase awal | `query_contracts.md`, `ui/reseller_uiux.md`, `integration_read_model_matrix.md`, `truth/01-decision_log.md` |

---

## 5. Data Questions

| ID | Pertanyaan | Status | Dampak | Dokumen Terkait |
|---|---|---|---|---|
| `DQ-001` | Bagaimana menjaga read model historis agar tidak berubah karena join ke master yang diedit? | resolved | dipakai untuk desain snapshot dan audit | `schema_mapping.md`, `query_contracts.md`, `product_truth_audit.md`, `truth/01-decision_log.md` |
| `DQ-002` | Apakah batch pembagian boleh mencampur item dengan status stok `SIAP` dan `KURANG` dalam satu submit? | resolved | dipakai untuk validation gudang | `business_contracts.md`, `modules/gudang.md`, `truth/01-decision_log.md` |

---

## 6. Testing Questions

| ID | Pertanyaan | Status | Dampak | Dokumen Terkait |
|---|---|---|---|---|
| `TQ-001` | Framework test apa yang akan dipakai untuk unit/integration: `Vitest` atau alternatif lain? | resolved | dipakai untuk script package dan task verifikasi | `environment.md`, `testing_strategy.md`, `truth/01-decision_log.md` |
| `TQ-002` | Framework E2E apa yang akan dipakai: `Playwright` atau alternatif lain? | resolved | dipakai untuk script `e2e` dan browser verification | `environment.md`, `testing_strategy.md`, `truth/01-decision_log.md` |

---

## 7. Rule untuk AI Coding Agent

- jika sebuah task menyentuh keputusan yang belum tertulis di register ini, agent wajib melaporkannya sebelum memutuskan implementasi final
- jika task tetap harus berjalan meski keputusan detailnya belum tertulis, agent harus memilih jalur paling konservatif dan menandainya sebagai asumsi
- jangan menyembunyikan keputusan lokal yang seharusnya masuk register ini
