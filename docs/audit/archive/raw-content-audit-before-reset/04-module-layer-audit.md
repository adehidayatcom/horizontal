# 04 Module Layer Audit

## Module: auth
- Readiness: partial
- Blockers:
  - Stale truth path references (`docs/product/prd.md`, `docs/contracts/schema_mapping.md`, `docs/contracts/rls_matrix.md`, `docs/truth/01-decision_log.md`) reduce source-of-truth traceability.
  - Self-register flow is referenced, but module contract does not fully define onboarding transition map in one place.
- Conflicts:
  - References conflict with current truth/contract locations (`docs/truth/*`, `docs/contracts/*`).
- Missing details:
  - Explicit boundary for self-register create flow and approval transition contract naming.
- Contract dependencies:
  - `docs/contracts/rls_matrix.md`, `docs/contracts/api_integration.md`, `docs/contracts/integration_contract_pack.md`, `docs/contracts/schema_mapping.md`.
- UI language issues:
  - User-facing labels mostly Indonesian and acceptable (`Pendaftaran sedang ditinjau`).
  - Implementation-facing terms are mixed Indonesian/English in prose; acceptable if code contracts remain system-term consistent.
- Recommended actions:
  - Normalize references to actual truth/contract paths.
  - Add explicit onboarding state transition matrix in module doc.

## Module: dashboard_laporan
- Readiness: partial
- Blockers:
  - Payload uses legacy system terms (`total_program_aktif`, `total_program_belum_dikunci`) while contract layer uses `pesanan` nomenclature.
  - Route/data references include stale paths and deferred scope uncertainty (UX-003 drill-down).
- Conflicts:
  - Naming drift vs `query_contracts.md` and `program_order` final terminology.
- Missing details:
  - Canonical mapping for metrics from read models to UI cards when names differ.
- Contract dependencies:
  - `docs/contracts/query_contracts.md`, `docs/contracts/integration_read_model_matrix.md`, `docs/contracts/api_integration.md`.
- UI language issues:
  - UI labels are Indonesian in component intent; good.
  - System fields should avoid legacy `program_*` names in implementation contract surfaces.
- Recommended actions:
  - Replace legacy `program_*` metric names with final `pesanan_*` system terms.
  - Mark deferred report drill-down as out-of-scope until decided.

## Module: gudang
- Readiness: partial
- Blockers:
  - Mixed term usage `order_id` in contracts while final entity is `detail_pesanan_konsumen_id` for shipment-level action.
  - Multiple stale doc references (`docs/product/data_flow.md`, `docs/contracts/business_contracts.md`, `docs/truth/system_maps.md`) path drift.
- Conflicts:
  - Some contract examples use `order` naming inconsistent with contract layer’s final terms.
- Missing details:
  - Explicit admin-only correction UX/route for `koreksi_belanja/packing/pembagian` (not just mentioned).
- Contract dependencies:
  - `business_contracts` RPCs (`buat_belanja`, `buat_packing`, `proses_kirim_detail_pesanan`, `buat_pembagian_paket`, `serahkan_pembagian_paket`), `query_contracts` stock/pembagian views.
- UI language issues:
  - User-facing labels are mostly Indonesian and aligned.
  - System naming should standardize away from generic `order` where detail entity is required.
- Recommended actions:
  - Align request/response identifiers to final entity naming.
  - Add explicit correction operation route contracts or mark deferred.

## Module: keuangan
- Readiness: partial
- Blockers:
  - Decision drift in references: module cites `BQ-007`/`BQ-008` from stale path and may conflict with truth decisions (known conflict in prior audits).
  - Uses stale contract paths (`docs/contracts/schema_mapping.md`, etc.).
- Conflicts:
  - Potential mismatch with truth decisions on `akun_kas` warning and `budget_belanja` implications in related module notes.
- Missing details:
  - Explicit rule boundary for historical period correction allowlist at keuangan-transaction field level.
- Contract dependencies:
  - `buat_kas_masuk`, `buat_mutasi_kas`, `buat_pencairan`, `v_saldo_kas`, `v_ringkasan_reseller`.
- UI language issues:
  - Indonesian labels for user flows are good.
  - System-term structure is fairly consistent.
- Recommended actions:
  - Reconcile decision references with `docs/truth/01-decision_log.md`.
  - Add explicit historical-correction policy references per transaction type.

## Module: master_periodik
- Readiness: partial
- Blockers:
  - Direct contradiction with truth decisions documented elsewhere (warning akun kas and `budget_belanja` mandatory note).
  - Stale path references to non-canonical files.
- Conflicts:
  - Cross-module conflict with truth decision log (`DL-027`, `DL-028`) already identified.
- Missing details:
  - Clear boundary ownership split with `periode` module for readiness evaluation and activation blockers.
- Contract dependencies:
  - `schema_mapping` master tables, `periode` readiness helpers, integration contract naming.
- UI language issues:
  - UI-facing language mostly Indonesian and acceptable.
  - System terms are mixed but largely aligned with table names.
- Recommended actions:
  - Resolve decision conflict explicitly before coding.
  - Add boundary statement: what master_periodik owns vs what periode workflow owns.

## Module: periode
- Readiness: partial
- Blockers:
  - Uses stale source references (`docs/truth/system_maps.md`, root-level path style).
  - Some task narratives use legacy `program` count fields in summaries.
- Conflicts:
  - Minor nomenclature drift with final `pesanan` terminology.
- Missing details:
  - Explicit backend contract for carry-over stock readiness check used by close-period gate.
- Contract dependencies:
  - `aktifkan_periode`, `tutup_periode`, period read models/checklist helpers.
- UI language issues:
  - Strong Indonesian UI label orientation is good.
  - Implementation-facing types should avoid legacy `program` naming.
- Recommended actions:
  - Standardize summary fields to final system terms.
  - Define close-period carry-over checker contract explicitly.

## Module: program_order
- Readiness: partial
- Blockers:
  - Path drift in references (`docs/product/program_workflow.md`, `docs/contracts/business_contracts.md`, etc. without `/product` or `/contracts`).
  - Legacy wording persists in places (`order`) despite final term migration.
- Conflicts:
  - Cross-layer conflict risk with product edge-cases on snapshot/live pricing assumptions.
- Missing details:
  - Explicit backend contract for marking `PERLU_PERHATIAN` action (manual status handling).
- Contract dependencies:
  - `buat_pesanan_konsumen`, `ubah_detail_pesanan_konsumen`, `finalisasi_pesanan_konsumen`, `ubah_status_detail_pesanan_konsumen`, read views for ringkasan/watchlist/detail.
- UI language issues:
  - UI entities Indonesian and mostly consistent.
  - System terms should strictly use final `pesanan/detail` terms in code-facing contracts.
- Recommended actions:
  - Add explicit status-action contract for watchlist/manual attention handling.
  - Normalize all references to canonical paths.

## Module: reseller
- Readiness: partial
- Blockers:
  - Heavy stale path usage (`docs/truth/system_maps.md`, root-level contract paths).
  - Summary field uses `jumlah_program_aktif` legacy naming.
- Conflicts:
  - Terminology drift vs final `pesanan` domain naming.
- Missing details:
  - Explicit boundary for self-register flow handoff to auth module (create->pending->approval).
- Contract dependencies:
  - `reseller`, `profile`, `reseller_periode`, `konsumen` schemas; RLS matrix; read models `v_ringkasan_reseller`, `v_ringkasan_konsumen`.
- UI language issues:
  - UI labels and entities are Indonesian and aligned for frontend.
  - System fields should remove legacy `program` terms from implementation contracts.
- Recommended actions:
  - Align summary/system field names to final domain.
  - Add explicit cross-module onboarding boundary with auth.

## Module: setoran
- Readiness: partial
- Blockers:
  - Uses legacy term in error code (`SETORAN_PROGRAM_NOT_FOUND`, `SETORAN_PROGRAM_NOT_ACTIVE`) while domain now uses `pesanan`.
  - Path/reference drift to non-canonical docs.
- Conflicts:
  - Cross-module naming inconsistency with `program_order` final entities and contract naming standards.
- Missing details:
  - Explicit admin correction boundary availability for first wave is marked deferred, but module should state fallback behavior clearly.
- Contract dependencies:
  - `buat_setoran_konsumen`, `buat_setoran_pusat`, read models `v_ringkasan_konsumen`, `v_ringkasan_reseller`, admin monitoring views.
- UI language issues:
  - Indonesian UI labels and mobile-first entities are strong.
  - System errors/contracts should use final entity term (`pesanan`) instead of `program`.
- Recommended actions:
  - Rename system error codes/fields to final entity terminology.
  - Clarify first-wave correction handling path explicitly.

## Duplicate rules across modules
- Repeated rule: “frontend must not calculate business final values” appears in many modules (`dashboard_laporan`, `program_order`, `setoran`, `gudang`, `keuangan`).
- Repeated rule: “scope by `periode_id`” duplicated across almost all modules.
- Repeated rule: “admin-only sensitive actions” repeated with varying phrasing.
- Recommendation:
  - Create one shared module-layer guardrail appendix and reference it to reduce drift.

## Missing module boundaries (cross-module)
1. `auth` vs `reseller`: onboarding/self-register ownership boundary is not sharply documented in one contract point.
2. `periode` vs `master_periodik`: readiness checklist ownership split is not explicit enough.
3. `program_order` vs `setoran`: watchlist/manual status action boundary missing explicit contract.
4. `gudang` vs `keuangan`: correction policy boundaries across period-closed states need explicit shared policy reference.

## Cross-module conflicts
1. Decision conflict propagation in `master_periodik` notes vs truth decisions (`DL-027`, `DL-028`).
2. Legacy `program` terminology still used in several modules (`dashboard_laporan`, `reseller`, `setoran`, `periode`) while final entity is `pesanan_konsumen`.
3. Stale path references (`docs/support/*`, root-level contract paths) across nearly all modules create inconsistent truth lookup.

## Language compliance summary
- Frontend UI-facing entities/labels: generally Indonesian and compliant.
- Implementation/system entities: mostly technical naming is acceptable, but legacy Indonesian business terms and deprecated `program` system terms remain in code-facing contracts.
- Recommendation:
  - Add per-module bilingual mapping table:
    - `System term (English/technical)`
    - `UI label (Indonesian)`

## Final readiness signal
- Overall module layer readiness: **partial**.
- Safe-to-code threshold not met until blocker items are resolved:
  - path canonicalization,
  - legacy naming cleanup (`program` -> `pesanan` where system-facing),
  - explicit missing action boundaries,
  - truth decision conflicts reconciled in module docs.
