# 02 Product Layer Audit

## Scope audited
- `docs/product/prd.md`
- `docs/product/data_flow.md`
- `docs/product/period_workflow.md`
- `docs/product/program_workflow.md`
- `docs/product/edge_cases.md`
- Product truth references used only for consistency checks: `docs/truth/01-decision_log.md`, `docs/truth/02-canonical_system_brief.md`

## Overall result
Product goal is broadly consistent (period-based savings/order operations with auditability), but there are critical conflicts across workflow/dataflow/edge-cases that can break implementation behavior if unresolved.

## Findings

### Finding ID: P-001
- Severity: blocker
- Conflicting documents:
  - `docs/product/edge_cases.md` (Prinsip Umum #1, EC-004, EC-012)
  - `docs/product/program_workflow.md` (sections `2.3 detail_pesanan_konsumen`, `4. Target Tagihan Konsumen`)
  - `docs/product/data_flow.md` (target uses item value + `TERHENTI` rule)
  - `docs/truth/02-canonical_system_brief.md` (`SUM(qty * harga_snapshot)`)
- Product impact:
  Core value model is inconsistent: snapshot-based pricing vs live master pricing.
- Implementation impact:
  Validation, recalculation, reporting, and audit logic will diverge between services; impossible to produce one correct target formula.
- Recommended decision:
  Decide one canonical pricing model for `detail_pesanan_konsumen` (snapshot or live) and align all product docs.
- UI wording impact, if any:
  Confirmation text about live changes may need removal or rewording if snapshot model is chosen.
- OPEN QUESTION:
  Is target calculation source `harga_snapshot` per item or current master `paket.nilai_paket`?

### Finding ID: P-002
- Severity: high
- Conflicting documents:
  - `docs/product/prd.md` (Reseller manages pendaftaran + core operations)
  - `docs/product/data_flow.md` section `(2) Pendaftaran Reseller` (Reseller or Admin)
  - product scope docs do not include explicit onboarding/login workflow file
- Product impact:
  Onboarding/login flow is not fully defined as a complete product workflow in product layer.
- Implementation impact:
  Auth guard states (`PENDING`, `AKTIF`, `NONAKTIF`) and first-screen routing behavior can be implemented inconsistently.
- Recommended decision:
  Add explicit onboarding/login state machine at product layer, including transitions and blocked actions by status.
- UI wording impact, if any:
  Needed for user-facing status copy consistency (`PENDING`, approval notifications, blocked action messages).
- OPEN QUESTION:
  What is the canonical onboarding/login transition map from registration to active operational access?

### Finding ID: P-003
- Severity: high
- Conflicting documents:
  - `docs/product/data_flow.md` (main path puts `Setoran Pusat` after `Pembagian` in narrative, but diagram also shows loop independent from packing)
  - `docs/product/program_workflow.md` (core flow ends with `Pencairan / Penutupan Periode` after pembagian)
  - `docs/product/edge_cases.md` EC-007 (setor pusat allowed before all consumers lunas)
- Product impact:
  Relative ordering of `setoran pusat` vs gudang/pembagian is ambiguous in narrative.
- Implementation impact:
  Teams may enforce incorrect workflow dependency (hard sequencing) and block valid operations.
- Recommended decision:
  Define workflow as dependency graph (not strict linear sequence) with explicit prerequisites per action.
- UI wording impact, if any:
  Step indicator labels and progress timelines may need non-linear wording.
- OPEN QUESTION:
  Is `setoran pusat` an independent parallel flow with financial constraints only, or a late-stage sequential step?

### Finding ID: P-004
- Severity: medium
- Conflicting documents:
  - `docs/product/edge_cases.md` EC-009 and EC-010 (uses legacy phrasing: “tanpa order”, “sebelum program DIKUNCI”)
  - `docs/product/program_workflow.md` (final model uses `pesanan_konsumen` + `tanggal_final`, no `DIKUNCI` status)
- Product impact:
  Legacy terms remain in exception narratives and can confuse product intent.
- Implementation impact:
  Risk of reintroducing deprecated state (`DIKUNCI`) and wrong domain term mapping in API/contracts.
- Recommended decision:
  Rewrite edge-case wording to final entity/state names only.
- UI wording impact, if any:
  Replace legacy user-facing terms with consistent Indonesian labels (e.g., `Difinalkan`, not `Dikunci` unless intentionally retained as UI copy alias).

### Finding ID: P-005
- Severity: medium
- Conflicting documents:
  - `docs/product/period_workflow.md` (section `11. Penutupan Periode` requires carry-over stock prepared)
  - `docs/product/edge_cases.md` EC-008 (closing blockers list does not explicitly include carry-over readiness)
- Product impact:
  Closing period blocker criteria are not fully harmonized.
- Implementation impact:
  `tutup_periode` readiness checks may differ between teams/services.
- Recommended decision:
  Create one canonical closing checklist with exact blocking conditions and reference it from all product docs.
- UI wording impact, if any:
  Blocker list text in “Selesaikan Periode” screen must match canonical checklist.

### Finding ID: P-006
- Severity: high
- Conflicting documents:
  - `docs/product/edge_cases.md` EC-013 (`tandai_program_perlu_perhatian`, `tandai_program_terhenti`)
  - `docs/product/program_workflow.md` (`PERLU_PERHATIAN` is watchlist/manual status, no product-level canonical RPC naming)
- Product impact:
  Exception flow references RPC names that are not clearly canonical in product layer and use legacy “program” naming.
- Implementation impact:
  API boundary naming can drift early; product-to-contract traceability becomes weak.
- Recommended decision:
  Define product-level action names independent of technical RPC names, then map to contract names later.
- UI wording impact, if any:
  Ensure Indonesian action labels map consistently to final system actions.
- OPEN QUESTION:
  What are canonical product actions for watchlist/status handling (term-level), independent of RPC naming?

### Finding ID: P-007
- Severity: medium
- Conflicting documents:
  - `docs/product/period_workflow.md` section `15. Bahasa UI`
  - `docs/product/prd.md` and `docs/product/program_workflow.md` mixed English/Indonesian labels without mapping table
- Product impact:
  Language policy exists locally (period workflow) but not as cross-product-layer standard.
- Implementation impact:
  Inconsistent terminology across workflow docs can leak into API naming or UI copy.
- Recommended decision:
  Add product-layer glossary with two columns: `System term (English)` and `UI label (Indonesian)`.
- UI wording impact, if any:
  High; affects menu/page/form/status wording consistency.

### Finding ID: P-008
- Severity: low
- Conflicting documents:
  - `docs/product/data_flow.md`, `docs/product/edge_cases.md` (encoding artifacts like `â€”`, `Ã—`)
- Product impact:
  No direct business change, but readability and search reliability are reduced.
- Implementation impact:
  Minor tooling/search mismatch risk for exact term queries.
- Recommended decision:
  Normalize encoding to UTF-8 across product docs.
- UI wording impact, if any:
  None directly.

## Workflow completeness checklist (product layer only)
- Onboarding/login: partial, not fully defined as explicit transition map. (See P-002)
- Period setup: mostly complete in `period_workflow.md`.
- Program/order flow: mostly complete in `program_workflow.md`, but conflicts with edge cases. (P-001, P-004)
- Reseller flow: present across PRD/data flow/program flow, but onboarding status transitions still incomplete. (P-002)
- Setoran/payment flow: largely complete, sequencing ambiguity remains. (P-003)
- Gudang/inventory flow: present at high level; dependency to setoran flow should be clarified as parallel vs sequential. (P-003)
- Keuangan/finance flow: present at high level through setoran pusat/pencairan + edge cases.
- Dashboard/reporting flow: referenced in PRD and period workflow; detailed report interaction intentionally not deep-audited here.

## System terms vs UI labels (observed)
- System terms (should be English):
  - `period_status`, `order_status`, `target_calculation`, `pricing_snapshot`, `audit_log`, `internal_api_route`, `row_level_security`.
- UI labels/entities (Indonesian, observed):
  - `Periode`, `Setoran Konsumen`, `Setor Pusat`, `Selesaikan Periode`, `Lihat periode`, `Perlu Perhatian`.
- Gap:
  - No single product-layer glossary binds each system term to one UI label.
  - OPEN QUESTION: Should one canonical bilingual mapping table be mandatory before implementation?
