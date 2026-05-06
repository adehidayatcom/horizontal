# Sprint Control Board
## Paket Lebaran Mumpuni

Dokumen ini menjadi papan kendali utama untuk eksekusi repo dengan tiga AI coder:

- Codex
- Gemini
- Claude

Dokumen ini tidak mengubah scope produk. Dokumen ini hanya mengatur:

- urutan sprint
- pembagian kerja antar agen
- status kontrol
- gate review dan audit
- packet eksekusi kecil per sprint

Pola kerja yang dikunci:

```txt
Codex buat task packet
-> Gemini implement
-> Codex cek hasil vs docs
-> Claude audit modul kritis
-> Codex putuskan merge / rework
-> lanjut ke task berikutnya
```

---

## 1. Aturan Kontrol Utama

| Aturan | Isi |
|---|---|
| Orchestrator | Codex |
| Builder default | Gemini |
| Auditor modul kritis | Claude |
| Source of truth | `docs/product/prd.md`, `docs/contracts/business_contracts.md`, `docs/contracts/schema_mapping.md`, `docs/contracts/query_contracts.md`, `docs/truth/01-decision_log.md` |
| Larangan utama | dua agen menulis file yang sama dalam packet aktif |
| Jalur keputusan merge | hanya Codex |
| Audit wajib | hanya pada modul kritis atau packet yang ditandai `AUDIT_REQUIRED` |

### Guardrail Eksekusi Wajib

| Guardrail | Aturan |
|---|---|
| Mock-to-Real Contract Rule | dummy data, mock context, placeholder API, dan transformer harus memakai shape contract resmi; dilarang membuat shape inline yang menyimpang dari kontrak akhir |
| Stable SQL Error Code Rule | validasi SQL/RPC yang bersifat bisnis harus mengembalikan error code stabil agar service layer dan UI tidak menebak pesan error |
| Concurrency Safety Rule | transaksi kritis harus divalidasi dan dieksekusi secara atomic di boundary yang sama; dilarang memisahkan validasi sensitif dan write akhir ke dua langkah yang rawan race condition |
| Git Checkpoint Rule | setiap menemukan titik aman packet atau sprint, perubahan harus di-stage dan di-commit sebelum lanjut ke packet besar berikutnya atau sebelum pindah thread eksekusi |

---

## 2. Status Sprint

| Status | Arti | Owner Aktif | Output Wajib |
|---|---|---|---|
| `PLANNED` | sprint sudah didefinisikan, belum dimulai | Codex | scope, owner, gate |
| `ACTIVE` | implementasi sedang berjalan | Gemini atau Codex | patch/diff kerja |
| `IN_REVIEW` | hasil implementasi sedang dicek terhadap docs | Codex | review note |
| `AUDIT` | modul sedang diaudit secara correctness | Claude | audit finding / approval |
| `MERGE_READY` | siap digabung tanpa blocker | Codex | keputusan merge |
| `DONE` | sprint ditutup | Codex | summary hasil |
| `BLOCKED` | sprint tertahan keputusan/dependency | Codex | blocker note |

---

## 3. Status Packet

| Status | Arti |
|---|---|
| `TODO` | belum dikerjakan |
| `DOING` | sedang diimplementasikan |
| `REVIEW` | menunggu cek Codex |
| `AUDIT` | menunggu audit Claude |
| `REWORK` | harus diperbaiki |
| `DONE` | selesai |
| `BLOCKED` | tertahan dependency atau keputusan |

---

## 4. Peta Peran Agen

| Agen | Peran Utama | Pekerjaan Paling Cocok | Larangan Utama |
|---|---|---|---|
| Codex | control tower, prompter, integrator | task packet, review, merge decision, cross-doc consistency, final integration | jangan membiarkan dua agen overlap file |
| Gemini | builder utama | frontend screens, hooks, dummy layer, route/service scaffolding, repetitive implementation | jangan menebak contract atau memperluas scope |
| Claude | auditor modul kritis | SQL/RLS/business rule audit, edge case review, refactor risky logic, final correctness check | jangan dipakai untuk bulk scaffolding (kecuali ditunjuk khusus sebagai Builder komponen UI kompleks tinggi seperti Spreadsheet DataGrid, Kloning, POS Cart, Ledger Grid, FinTech UI, Split-Pane, dan Kanban) |

---

## 5. Sprint Map Utama

| Sprint ID | Nama Sprint | Tujuan | Owner Builder | Auditor | Status Awal | Gate Keluar |
|---|---|---|---|---|---|---|
| `S0` | Foundation Lock | kunci docs, task packet, workflow agen, branch discipline | Codex | - | `PLANNED` | semua aturan eksekusi terkunci |
| `S1` | Frontend Shell Foundation | shell admin/reseller, shared UI, context mock, dummy API skeleton | Gemini | Codex review only | `PLANNED` | shell dan shared layer stabil |
| `S1.5` | Periode & Katalog Paket | setup periode, wizard kloning, master paket (Spreadsheet Mode) | Claude / Gemini | Codex / Claude | `PLANNED` | kloning dan data grid ratusan paket berjalan |
| `S2` | Pesanan Core | `konsumen`, `pesanan_konsumen`, `detail_pesanan_konsumen`, finalisasi pesanan | Gemini | Claude | `PLANNED` | flow pesanan valid end-to-end |
| `S3` | Setoran Core | `setoran_konsumen`, `setoran`, monitoring setoran | Gemini | Claude | `PLANNED` | status lunas dan saldo konsisten |
| `S4` | Gudang | belanja, packing, pengiriman, pembagian | Gemini | Claude | `PLANNED` | flow gudang valid dan dapat diaudit |
| `S5` | Keuangan | akun kas, kas masuk, mutasi kas, pencairan | Gemini | Claude | `PLANNED` | flow keuangan valid dan audit-safe |
| `S6` | Dashboard dan Laporan | dashboard admin/reseller, laporan inti, audit screen | Gemini | Claude selective | `PLANNED` | semua read model dipakai konsisten |
| `S7` | Integration Hardening | sinkronisasi route, type, response, dead path cleanup | Codex | Claude selective | `PLANNED` | tidak ada mismatch lintas modul |
| `S8` | Testing dan Release Readiness | baseline test, smoke flow, readiness akhir | Codex | Claude final pass | `PLANNED` | verifikasi minimum lolos |

---

## 6. Packet Map per Sprint

### `S0` Foundation Lock

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S0-T01` | Lock source of truth | cek docs aktif dan dependency | Codex | - | `docs/` | source of truth final tercatat |
| `S0-T02` | Define packet workflow | format packet, status, owner, gate | Codex | - | `docs/execution/` | workflow agen terdokumentasi |
| `S0-T03` | Lock sprint board | buat dan tautkan board sprint | Codex | - | `docs/execution/` | board aktif dipakai |

### `S1` Frontend Shell Foundation

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S1-T01` | Admin shell | admin layout, nav, header | Gemini | Codex | `src/app/components/layout/**` | shell admin stabil |
| `S1-T02` | Reseller shell | reseller mobile shell, bottom nav | Gemini | Codex | `src/app/components/layout/**` | shell reseller stabil |
| `S1-T03` | Shared UI primitives | badge, state, header, input rupiah | Gemini | Codex | `src/app/components/shared/**` | shared UI reusable |
| `S1-T04` | Mock contexts | auth, period, UI, mock API context | Gemini | Codex | `src/app/context/**` | context mock siap |
| `S1-T05` | Dummy API skeleton | fetcher, handlers, endpoints | Gemini | Codex | `src/lib/dummy-api/**` | placeholder API konsisten |

### `S1.5` Periode & Katalog Paket

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S1.5-T01` | Periode Backend & Mock | API, RPC kloning, mock list periode | Gemini | Claude | `src/app/api/admin/periode/**`, `src/lib/server/**` | RPC kloning masal siap dipanggil UI |
| `S1.5-T02` | Wizard Setup Periode | UI create & wizard kloning periode | Gemini | Claude | `src/app/(DashboardLayout)/admin/periode/**` | flow wizard kloning paket bisa didemokan |
| `S1.5-T03` | Katalog Paket (Spreadsheet) | UI data grid massal, mass replace, tree-view | Claude | Codex | `src/app/(DashboardLayout)/admin/master/paket/**` | inline-editing ratusan baris stabil tanpa lag |

### `S2` Pesanan Core

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S2-T01` | Pesanan schema/read model | migration dan read model pesanan | Gemini | Claude | `supabase/migrations/001*`, `004*`, `007*` | schema + views valid |
| `S2-T02` | Pesanan backend layer | validation, workflow, service, route | Gemini | Claude | `src/lib/server/**`, `src/app/api/**` | contract backend konsisten |
| `S2-T03` | Reseller pesanan UI | list, create, detail, finalisasi | Gemini | Codex -> Claude | `src/app/(DashboardLayout)/reseller/pesanan/**`, components/hooks terkait | flow reseller jalan |
| `S2-T04` | Admin pesanan UI | list, watchlist, detail pesanan | Gemini | Codex -> Claude | `src/app/(DashboardLayout)/admin/pesanan/**`, components/hooks terkait | flow admin jalan |

### `S3` Setoran Core

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S3-T01` | Setoran schema/boundary | SQL setoran, monitoring views | Gemini | Claude | `supabase/migrations/008*`, `004*` | boundary setoran valid |
| `S3-T02` | Setoran backend layer | validation, workflow, service, route | Gemini | Claude | `src/lib/server/**`, `src/app/api/**` | status lunas konsisten |
| `S3-T03` | Reseller setor UI | setor konsumen dan setor pusat | Gemini | Codex -> Claude | `src/app/(DashboardLayout)/reseller/setor/**`, components/hooks terkait | alur 3 langkah siap |
| `S3-T04` | Admin monitoring setoran | layar monitoring layer 1 dan 2 | Gemini | Codex | `src/app/(DashboardLayout)/admin/setoran-*/**` | monitoring admin jelas |

### `S4` Gudang

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S4-T01` | Gudang backend | belanja, packing, kirim, pembagian | Gemini | Claude | `supabase/migrations/009*`, `src/lib/server/**`, `src/app/api/admin/gudang/**` | eligibility dan stok valid |
| `S4-T02` | Gudang UI | screen gudang admin | Gemini | Codex -> Claude | `src/app/(DashboardLayout)/admin/gudang/**`, components/hooks terkait | UI gudang lengkap |

### `S5` Keuangan

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S5-T01` | Keuangan backend | kas masuk, mutasi, pencairan | Gemini | Claude | `supabase/migrations/010*`, `src/lib/server/**`, `src/app/api/admin/keuangan/**` | saldo dan audit valid |
| `S5-T02` | Keuangan UI | screen keuangan admin | Gemini | Codex -> Claude | `src/app/(DashboardLayout)/admin/keuangan/**`, components/hooks terkait | UI keuangan lengkap |

### `S6` Dashboard dan Laporan

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S6-T01` | Dashboard/report backend | read route, payload, filter | Gemini | Claude selective | `supabase/migrations/011*`, `src/lib/server/**`, `src/app/api/admin/laporan/**`, `src/app/api/reseller/dashboard/**` | payload read stabil |
| `S6-T02` | Admin dashboard/laporan UI | KPI, watchlist, report screens | Gemini | Codex | `src/app/(DashboardLayout)/admin/dashboard/**`, `admin/laporan/**` | dashboard admin siap |
| `S6-T03` | Reseller dashboard UI | KPI, priority list, shortcut setor | Gemini | Codex | `src/app/(DashboardLayout)/reseller/**`, components/hooks terkait | dashboard reseller siap |

### `S7` Integration Hardening

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S7-T01` | Type and contract hardening | types, response envelope, naming | Codex | Claude selective | lintas `src/types/**`, route/service response | tidak ada drift contract |
| `S7-T02` | Route and state cleanup | dead route, dead state, mismatch cleanup | Codex | Claude selective | lintas app/hooks/components | tidak ada dead path besar |
| `S7-T03` | Cross-module merge pass | pesanan-setoran-gudang-keuangan-dashboard | Codex | Claude | lintas modul | integrasi konsisten |

### `S8` Testing dan Release Readiness

| Packet ID | Nama | Scope | Builder | Audit | Write Scope | Exit Criteria |
|---|---|---|---|---|---|---|
| `S8-T01` | Unit/integration baseline | validation, service, route tests | Codex + Gemini | Claude | `tests/**`, config terkait | baseline test hijau |
| `S8-T02` | E2E smoke flow | login, periode, reseller, pesanan, setoran | Codex + Gemini | Claude | `e2e/**` | smoke flow lolos |
| `S8-T03` | Final readiness pass | bug triage dan merge gate akhir | Codex | Claude | lintas repo | siap masuk release |

---

## 7. Current Control Marker

Gunakan blok ini setiap kali status sprint diperbarui:

```txt
CURRENT SPRINT : S0 Foundation Lock
SPRINT STATUS  : DONE
ACTIVE PACKET  : -
BUILDER        : Codex
AUDITOR        : -
NEXT GATE      : Prepare S1-T01 packet for Gemini
```

Jika sprint aktif pindah, blok ini wajib diperbarui.

---

## 8. Sprint Status Board

| Sprint ID | Nama | Status | Active Packet | Builder | Auditor | Last Decision | Next Gate |
|---|---|---|---|---|---|---|---|
| `S0` | Foundation Lock | `DONE` | - | Codex | - | source of truth, board sprint, prompt template, dan guardrail eksekusi sudah terkunci | siapkan packet `S1-T01` |
| `S1` | Frontend Shell Foundation | `PLANNED` | - | Gemini | Codex | belum mulai | bekukan packet `S1-T01` lalu buka thread eksekusi |
| `S1.5` | Periode & Katalog Paket | `PLANNED` | - | Claude / Gemini | Codex / Claude | belum mulai | tunggu S1 selesai |
| `S2` | Pesanan Core | `PLANNED` | - | Gemini | Claude | belum mulai | tunggu S1.5 stabil |
| `S3` | Setoran Core | `PLANNED` | - | Gemini | Claude | belum mulai | tunggu S2 stabil |
| `S4` | Gudang | `PLANNED` | - | Gemini | Claude | belum mulai | tunggu S3 stabil |
| `S5` | Keuangan | `PLANNED` | - | Gemini | Claude | belum mulai | tunggu S4 dependency siap |
| `S6` | Dashboard dan Laporan | `PLANNED` | - | Gemini | Claude selective | belum mulai | tunggu read model inti siap |
| `S7` | Integration Hardening | `PLANNED` | - | Codex | Claude selective | belum mulai | tunggu S1-S6 baseline selesai |
| `S8` | Testing dan Release Readiness | `PLANNED` | - | Codex + Gemini | Claude | belum mulai | tunggu integrasi selesai |

---

## 9. Packet Log Template

Gunakan tabel ini untuk mencatat packet yang sedang dikerjakan.

| Packet ID | Sprint | Scope | Status | Builder | Auditor | Write Scope | Acceptance Gate | Notes |
|---|---|---|---|---|---|---|---|---|
| `S0-T01` | `S0` | lock source of truth | `DONE` | Codex | `-` | `docs/` | source of truth final tercatat | selesai |
| `S0-T02` | `S0` | define packet workflow | `DONE` | Codex | `-` | `docs/execution/` | workflow agen terdokumentasi | selesai |
| `S0-T03` | `S0` | lock sprint board | `DONE` | Codex | `-` | `docs/execution/` | board, template prompt, dan guardrail dipakai | checkpoint resmi: `f7a425f` |
| `S1-T01` | `S1` | admin shell | `TODO` | Gemini | Codex | `src/app/components/layout/**` | shell admin stabil | packet belum dibekukan |

---

## 10. Checkpoint Log

Setiap checkpoint git yang dianggap aman harus dicatat di board ini.

| Checkpoint | Sprint/Packet | Commit | Tanggal | Scope | Catatan |
|---|---|---|---|---|---|
| `CP-001` | `S0-T03` | `f7a425f` | `2026-05-06 02:38:19 +0700` | docs foundation lock | source of truth, board sprint, dan workflow agen terkunci |

---

## 11. Rule Operasional Sprint

1. Codex selalu menulis atau memperbarui packet sebelum implementasi dimulai.
2. Gemini hanya bekerja pada packet aktif yang write scope-nya jelas.
3. Claude hanya mengaudit packet yang sudah lolos review awal Codex.
4. Jika satu packet menyentuh file milik packet lain, Codex wajib memecah ulang atau menunda packet.
5. Tidak ada sprint yang naik ke `DONE` sebelum gate keluar sprint terpenuhi.
6. Jika terjadi drift terhadap docs, status packet turun ke `REWORK`.
7. Jika ditemukan keputusan produk baru, sprint pindah ke `BLOCKED` sampai `decision_log` dan docs terkait diperbarui.
8. Packet yang membuat dummy layer harus lolos `Mock-to-Real Contract Rule`.
9. Packet backend kritis harus mendefinisikan mapping `stable SQL error code` yang dipakai service layer.
10. Packet transaksi kritis harus menyebut strategi `atomic write` atau `locking boundary` pada acceptance gate.
11. Packet yang sudah lolos gate aman harus masuk checkpoint git sebelum packet besar berikutnya dibuka.
12. Sprint yang berpindah dari `ACTIVE` ke `DONE` harus punya keputusan apakah perlu commit checkpoint saat itu juga.

### Aturan Checkpoint Git

| Kondisi | Aksi Minimum |
|---|---|
| packet selesai dan lolos review | stage perubahan packet dan siapkan commit checkpoint |
| packet selesai lalu akan diaudit | boleh stage dulu; commit setelah audit lolos atau setelah rework kecil selesai |
| sprint selesai | evaluasi perubahan yang sudah stabil, lalu commit checkpoint sebelum pindah fokus |
| pindah thread dari kontrol ke eksekusi | pastikan status git bersih atau ada checkpoint yang disengaja |

Format ringkas keputusan checkpoint:

```txt
GIT CHECKPOINT:
STATE        : REQUIRED | OPTIONAL | HOLD
REASON       : <alasan singkat>
SCOPE        : <packet/sprint yang dicakup>
NEXT ACTION  : <stage | commit | hold>
```

---

## 12. Bacaan Wajib Sebelum Sprint Aktif

| Sprint | Dokumen Minimum |
|---|---|
| `S1` | `docs/frontend_architecture.md`, `docs/frontend_component_contracts.md`, `docs/execution/frontend_plan.md` |
| `S2` | `docs/product/prd.md`, `docs/product/program_workflow.md`, `docs/contracts/business_contracts.md`, `docs/contracts/schema_mapping.md`, `docs/contracts/query_contracts.md`, `docs/modules/program_order.md` |
| `S3` | `docs/contracts/business_contracts.md`, `docs/contracts/query_contracts.md`, `docs/modules/setoran.md` |
| `S4` | `docs/contracts/schema_mapping.md`, `docs/contracts/business_contracts.md`, `docs/modules/gudang.md` |
| `S5` | `docs/contracts/schema_mapping.md`, `docs/contracts/business_contracts.md`, `docs/modules/keuangan.md` |
| `S6` | `docs/contracts/query_contracts.md`, `docs/contracts/integration_read_model_matrix.md`, `docs/modules/dashboard_laporan.md` |
| `S7` | semua dokumen modul aktif + `docs/truth/01-decision_log.md` |
| `S8` | `docs/testing_strategy.md`, `docs/definition_of_done.md`, `docs/environment.md` |
