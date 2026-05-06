# 07 - Cross-Layer Reconciliation

## Overall readiness verdict
**Verdict: partially ready**

Documentation has strong breadth and structure, but cross-layer blockers remain unresolved. Coding can start only for low-risk foundation work; transactional and permission-sensitive implementation is not safe until blocker decisions are closed.

## Blocker summary

### Blockers that must be resolved before coding (or before S2+ transactional coding)

1. **Authority hierarchy conflict**
- Area: source of truth
- Core issue: document authority order is inconsistent between AGENTS and some contract/quality/execution docs.
- Impact: conflicting conflict-resolution behavior across agents.

2. **Period status model mismatch**
- Area: schema/database + API/query contract
- Core issue: `reseller_periode.status_periode` includes `DITUTUP_MANUAL` while shared period status contract lists `PERSIAPAN | AKTIF | SELESAI`.
- Impact: enum/type/API/frontend filter mismatch.

3. **Pricing/target calculation conflict**
- Area: product workflow + schema/database
- Core issue: snapshot-based vs live master pricing assumptions across product docs.
- Impact: different totals, validation paths, audit outcomes.

4. **Missing canonical backend contract for manual watchlist/status actions**
- Area: API/query contract + module boundary
- Core issue: operational status actions (e.g., attention/watchlist handling) are referenced but not fully contracted.
- Impact: UI or service teams may invent commands.

5. **UI action-to-contract mapping gap**
- Area: frontend/UI + API/query contract
- Core issue: critical UI actions do not consistently reference contract IDs, payload schemas, and state outcomes.
- Impact: unsafe implementation assumptions, inconsistent error handling.

6. **Role-visibility to permission/RLS traceability gap**
- Area: permission/RLS + frontend/UI
- Core issue: UI visibility rules are not fully mapped to permission keys and RLS/API references.
- Impact: unauthorized UI exposure or hidden valid actions.

7. **Historical correction policy not operationally explicit**
- Area: permission/RLS + schema/database + contracts
- Core issue: principle exists, but field/action allowlist per correction boundary is incomplete.
- Impact: over-permissive or under-permissive correction behavior.

8. **Setup/test baseline incompleteness for integrated development**
- Area: execution/testing/setup
- Core issue: documented baseline mentions missing `supabase/` bootstrap state, generated DB types, and automated test harness maturity.
- Impact: transactional coding can proceed without reliable verification.

## Consolidated findings

### 1) Source of truth
- Deduplicated issues:
  - Authority order conflict (AGENTS vs other docs).
  - Stale references (`docs/support/*`, root-level contract paths).
  - Truth consumption ambiguity (summary docs vs decision log precedence).
- Severity: high.
- Fix timing: **before broad implementation**.

### 2) Product workflow
- Deduplicated issues:
  - `pricing_snapshot` vs live pricing conflict for `target_calculation`.
  - onboarding/login state machine incomplete.
  - sequential vs parallel interpretation of `setoran_pusat` vs gudang flow.
  - legacy `program` terminology still present in some workflow/edge-case phrasing.
- Severity: blocker/high.
- Fix timing: **before module coding that touches order/setoran/gudang/keuangan**.

### 3) Schema/database
- Deduplicated issues:
  - status enum mismatch (`reseller_periode.status_periode`).
  - correction-policy operational matrix missing by table/field/action.
  - carry-over close-period checker boundary not explicit enough.
- Severity: blocker/high.
- Fix timing: **before migration/RPC implementation for affected domains**.

### 4) Permission/RLS
- Deduplicated issues:
  - reseller-safe projection/read coverage incomplete for some RLS-permitted reads.
  - reseller correction-history scope not explicitly decided.
  - UI visibility rules not mapped to permission keys/RLS IDs.
- Severity: high.
- Fix timing: **before role-sensitive UI and route wiring**.

### 5) API/query contract
- Deduplicated issues:
  - missing contract for manual watchlist/status action.
  - optional/conditional query usage (“query tambahan bila ada”) for important screens.
  - stale cross-reference paths reduce contract traceability.
- Severity: blocker/high.
- Fix timing: **before backend route + frontend action integration**.

### 6) Module boundary
- Deduplicated issues:
  - `auth` vs `reseller` onboarding ownership boundary unclear.
  - `periode` vs `master_periodik` readiness/checklist ownership split unclear.
  - `program_order` vs `setoran` status/watchlist boundary incomplete.
  - repeated legacy `program_*` system naming in module artifacts.
- Severity: high.
- Fix timing: **before sprint packet activation for affected modules**.

### 7) Frontend/UI
- Deduplicated issues:
  - route hierarchy vs period setup route map unsynchronized.
  - no canonical route ownership matrix.
  - no canonical UI action-contract matrix.
  - no canonical role visibility matrix.
  - state behavior matrix (especially success/recovery) incomplete.
- Severity: blocker/high.
- Fix timing: **before S1.5/S2 UI packets**.

### 8) Execution/testing/setup
- Deduplicated issues:
  - sprint packets are structured but not explicitly gated by unresolved decision IDs.
  - test strategy is strong conceptually but not yet hard-enforced in execution gates.
  - setup baseline and deterministic seed/test profiles need formal gating.
- Severity: high.
- Fix timing: **before S2+ transactional coding**.

## Final open questions

1. **OQ-CL-01 (Authority)**
- What is the final canonical conflict-resolution hierarchy across all docs?
- Candidate: AGENTS order (`decision_log` first) as global standard.

2. **OQ-CL-02 (Period status)**
- Is `reseller_periode.status_periode` a distinct enum from `periode.status`?
- If yes, what is canonical cross-mapping and UI display behavior?

3. **OQ-CL-03 (Pricing model)**
- For `pesanan_konsumen` target calculation, is canonical source `harga_snapshot` or current master package value?

4. **OQ-CL-04 (Watchlist action contract)**
- What is canonical backend command contract for manual watchlist/attention status handling?

5. **OQ-CL-05 (Historical correction scope)**
- Which exact correction actions/fields are allowed on closed periods by `ADMIN`?

6. **OQ-CL-06 (Onboarding boundary)**
- What is canonical transition map for registration -> `PENDING` -> approved operational access, and which module owns each transition?

7. **OQ-CL-07 (Role visibility mapping)**
- What permission key naming convention is canonical for UI conditional rendering, and where is its source artifact?

8. **OQ-CL-08 (Language mapping source)**
- Where is the canonical glossary for `system_term` (English) to UI label (Indonesian)?

## Decision sequence

1. **Lock authority model** (OQ-CL-01) and normalize stale path references.
2. **Lock domain status model** (OQ-CL-02) including enum mapping policy.
3. **Lock pricing/target model** (OQ-CL-03).
4. **Lock correction policy matrix** (OQ-CL-05).
5. **Lock missing action contracts** for watchlist/manual status and optional reads (OQ-CL-04).
6. **Lock onboarding ownership/state machine** (OQ-CL-06).
7. **Lock permission-to-UI key standard** (OQ-CL-07).
8. **Lock bilingual system/UI glossary source** (OQ-CL-08).
9. **Then** activate S2+ packets with blocker-indexed test plans.

## Docs update plan

### Phase A: Canonicalization (no business-rule change)
1. Normalize all stale references to canonical paths (`docs/truth/*`, `docs/contracts/*`, etc.).
2. Align authority-order sections in execution/quality/contracts docs with AGENTS canonical order.
3. Remove duplicate or overlapping sections that can drift (especially UI docs).

### Phase B: Contract closure
1. Add explicit enum model for period-related statuses.
2. Add action-contract registry for critical UI/backend operations.
3. Add correction-policy allowlist/denylist matrix by boundary/table/field.
4. Resolve optional query contracts into explicit in-scope/out-of-scope contracts.

### Phase C: Cross-layer mapping
1. Add route ownership matrix (route, actor, guard, period-state dependency, module owner).
2. Add role visibility matrix (component/action -> permission key -> RLS/API reference).
3. Add state behavior matrix (loading/empty/error/success/offline/recovery) for key UI actions.
4. Add bilingual glossary:
- System term: `payment_verification`
- UI label: `Verifikasi Pembayaran`
- System term: `pending_verification`
- UI status shown: `Menunggu Verifikasi`

### Phase D: Execution gate hardening
1. Add packet fields: `decision_log_ref`, `open_question_ref`, `contract_gate_passed`, `rtk_ready`.
2. Add setup gate checklist (Supabase bootstrap, generated types, env matrix, deterministic seeds).
3. Add blocker-indexed testing plan linking each blocker to required unit/integration/E2E checks.

## Coding readiness checklist

### Must be true before transactional coding
- [ ] Authority hierarchy is uniform across truth/contracts/quality/execution docs.
- [ ] Status enum model for period and reseller-period is finalized.
- [ ] Pricing/target calculation source is finalized.
- [ ] Watchlist/manual status backend contract is finalized.
- [ ] UI action-contract matrix exists for critical screens.
- [ ] Role visibility matrix mapped to permission/RLS references exists.
- [ ] Correction policy matrix for closed-period data is finalized.
- [ ] Route ownership matrix is finalized.
- [ ] Bilingual glossary source is finalized (English system terms, Indonesian UI labels).
- [ ] Setup gate passed (Supabase/bootstrap/types/env baseline).
- [ ] Test gate passed for first high-risk flows (permissions, money/status workflows, contract envelopes, UI states).

### Issues that can be fixed during implementation (non-blocking)
- Encoding and text cleanup artifacts.
- Duplicate narrative sections in UI docs.
- Minor terminology polish where canonical term is already decided.
- Additional convenience test coverage beyond minimum blocker-indexed set.
