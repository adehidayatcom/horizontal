# Canonical System Brief (DRAFT)

Status: **DRAFT**
Reason: unresolved blockers remain from cross-layer reconciliation (`docs/audit/content/07-cross-layer-reconciliation.md`).

## 1. Scope and authority

This brief consolidates currently accepted system direction for implementation planning.

Canonical precedence (current intended model):
1. `docs/truth/01-decision_log.md`
2. `docs/truth/canonical_system_brief.DRAFT.md` (this document while draft)
3. `docs/product/prd.md`
4. `docs/contracts/schema_mapping.md`
5. `docs/contracts/business_contracts.md`
6. `docs/contracts/query_contracts.md`
7. `docs/contracts/integration_contract_pack.md`
8. `docs/modules/*.md`
9. `docs/execution/*.md`

If conflict is not resolved by accepted decisions, mark as OPEN QUESTION and do not assume.

## 2. Confirmed implementation baseline (accepted)

### 2.1 Stack and architecture
- Frontend stack: Next.js App Router + TypeScript + MUI/Modernize + SWR + Formik/Yup.
- Integration boundary: operational business data must go through internal API routes.
- Supabase: data platform with RPC and RLS boundaries.

### 2.2 Domain and lifecycle anchors
- Core transaction model uses `pesanan_konsumen` and `detail_pesanan_konsumen`.
- `pesanan_konsumen` reaches final lifecycle `SELESAI` at official period closing, not merely at payment completion.
- Historical read behavior should prioritize snapshot-based values where defined by accepted contracts.

### 2.3 Role model
- Primary actors: `ADMIN` and `RESELLER`.
- Onboarding policy accepted: admin-create path and self-register path both exist; self-register enters pending approval state.

### 2.4 First-wave boundaries
- Watchlist/dashboard computation in first wave: on-demand (no required production cron job).
- Unified reseller history feed: deferred; first wave stays modular by screen/domain.
- Reporting first wave: operational core first, richer reports can follow.

## 3. System terms and UI labels

Use English for system/internal terms. Use Indonesian for user-facing UI labels.

Examples:
- System status: `pending_approval`
- UI label: `Menunggu Persetujuan`

- System status: `completed`
- UI label: `Selesai`

- System entity: `pesanan_konsumen`
- UI label/entity: `Pesanan Konsumen`

- System action: `submit_payment`
- UI button label: `Simpan Setoran`

- System route segment: `/admin/payment-verification`
- UI page title: `Verifikasi Pembayaran`

Note: final bilingual glossary source is still OPEN QUESTION (see section 7).

## 4. Module-level operating model (draft-stable)

- `auth`: session + role guard + pending approval behavior.
- `periode`: period setup/activation/closing gate.
- `master_periodik`: period-dependent master data support.
- `reseller`: reseller and konsumen operational ownership.
- `program_order` (domain now aligned to order model): order lifecycle and detail states.
- `setoran`: consumer and center payment flows.
- `gudang`: purchase/packing/distribution/shipping operations.
- `keuangan`: cash mutation and withdrawal boundaries.
- `dashboard_laporan`: read-model-driven operational visibility.

Cross-module behavior remains subject to unresolved contract and mapping blockers.

## 5. Implementation constraints

- Do not invent business rules beyond accepted decisions.
- Do not bypass internal API routes for operational business data.
- Do not place final business formulas only in browser code.
- Do not manually edit generated database types.
- Do not silently resolve document conflicts.

## 6. Blockers still unresolved (keeps this brief in DRAFT)

1. Authority hierarchy is not fully synchronized across all docs.
2. Period status model mismatch (`reseller_periode.status_periode` vs shared status set).
3. Pricing/target calculation conflict (snapshot vs live model in product docs).
4. Missing canonical backend contract for manual watchlist/attention status actions.
5. UI action-to-backend-contract mapping is incomplete for critical actions.
6. Role visibility mapping to permission/RLS references is incomplete.
7. Historical correction allowlist/denylist at field/action level is not explicit.
8. Setup/test gate baseline for transactional coding is not fully closed.

Source: `docs/audit/content/07-cross-layer-reconciliation.md`.

## 7. Final open questions (must be resolved before final brief)

- OQ-CL-01: Final canonical authority hierarchy across all docs.
- OQ-CL-02: Distinct enum policy for `reseller_periode.status_periode`.
- OQ-CL-03: Canonical target/pricing source (`harga_snapshot` vs live master).
- OQ-CL-04: Canonical command contract for manual watchlist/attention status handling.
- OQ-CL-05: Exact correction scope on closed periods for `ADMIN`.
- OQ-CL-06: Canonical onboarding transition ownership across `auth` and `reseller`.
- OQ-CL-07: Canonical permission key naming standard for UI conditional rendering.
- OQ-CL-08: Canonical bilingual glossary source (`system_term` vs UI label).

## 8. Conditions to promote DRAFT -> final

This document can become `docs/truth/canonical-system-brief.md` when:
- All blocker items in section 6 are resolved in accepted truth/contract artifacts.
- Open questions in section 7 are closed or explicitly deferred out-of-scope with guardrails.
- Cross-references are normalized to canonical paths.
