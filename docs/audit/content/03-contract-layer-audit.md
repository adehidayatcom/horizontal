# Contract readiness summary
Contract layer is substantial and close to implementation-ready, but **not coding-safe yet** due to several blocker inconsistencies. Core coverage exists for major workflows (RPC write boundaries, read models, RLS principles, API envelope), but there are unresolved mismatches in status model, naming/path authority references, and a few action-to-contract gaps.

Readiness level: **Medium-Low** until blocker items are resolved.

# Schema vs workflow consistency
## Consistent areas
- Core entities map well to product flow:
  - `pesanan_konsumen` + `detail_pesanan_konsumen` + `setoran_konsumen` + `setoran` + gudang/keuangan entities.
  - Source: `docs/contracts/schema_mapping.md` sections `2.x`, `3.x`.
- Main write actions have explicit RPC contracts.
  - Source: `docs/contracts/business_contracts.md` sections `## 2` to `## 18`.

## Inconsistencies
### C-001 (blocker)
- Issue: `reseller_periode.status_periode` lifecycle diverges from shared period status contract.
- Evidence:
  - `schema_mapping.md` `### 2.1a reseller_periode`: `AKTIF | DITUTUP_MANUAL | SELESAI`
  - `integration_contract_pack.md` `## 5.1 Status Periode`: `PERSIAPAN | AKTIF | SELESAI`
- Impact:
  - Type generation and API validation mismatch for period-related filters/state.
  - Next.js screens using one enum may reject/ignore valid DB values.
- OPEN QUESTION:
  - Is `status_periode` intentionally a separate enum from `periode.status` with its own contract?

### C-002 (high)
- Issue: Product edge-case language on “no snapshot/live pricing” conflicts with schema/business contracts that include `harga_snapshot` and `target_tagihan_snapshot`.
- Evidence:
  - Contracts: `schema_mapping.md` `detail_pesanan_konsumen.harga_snapshot`, formulas using snapshot.
  - Product edge-case text (outside contract scope) says no snapshot; contract layer assumes snapshot.
- Impact:
  - Contract-vs-product drift can break workflow calculations and regression expectations.
- Recommendation:
  - Confirm contract model as canonical and flag product docs to align before implementation.

# Permission/RLS consistency
## Strong alignment
- RLS matrix clearly defines `ADMIN` vs `RESELLER OWN` patterns, helper functions, and test cases.
  - Source: `docs/contracts/rls_matrix.md` sections `2`, `3`, `6`.
- API integration enforces actor/session derivation and forbids trusting client `no_reseller`.
  - Source: `docs/contracts/api_integration.md` section `7`.

## Gaps / risks
### P-001 (high)
- Issue: RLS matrix allows some reseller `READ terbatas` but query contracts do not always define reseller-safe projections for those datasets.
- Evidence:
  - `rls_matrix.md`: reseller read limited for `paket`, `detail_paket`, `komisi_config`, `packing`.
  - `query_contracts.md`: no explicit reseller-specific read contracts for all these datasets.
- Impact:
  - Teams may expose broader fields than intended or block needed data.
- Recommendation:
  - Add explicit reseller-safe read models (or explicit “not exposed” notes) per sensitive table.

### P-002 (medium)
- Issue: `koreksi_transaksi` reseller visibility marked `OWN READ terbatas` in RLS, but no reseller audit/koreksi read contract exists.
- Evidence:
  - `rls_matrix.md` table row `koreksi_transaksi`.
  - `query_contracts.md` audit views are admin-oriented.
- Impact:
  - Ambiguous behavior for reseller-facing dispute/history screen.
- OPEN QUESTION:
  - Should reseller have contract-level access to their correction history in phase 1?

# API/query coverage
## Coverage present
- API integration pattern is explicit (`internal API route -> service -> RPC/view`).
  - Source: `api_integration.md` sections `2`, `5`, `6`.
- Read model matrix maps many important screens to route + read contracts.
  - Source: `integration_read_model_matrix.md` sections `2`-`8`.

## Coverage gaps
### A-001 (blocker)
- Issue: Manual status actions referenced in flows are missing explicit backend contracts.
- Evidence:
  - Domain uses watchlist/manual status changes (`PERLU_PERHATIAN`, `TERHENTI` handling).
  - `business_contracts.md` lacks dedicated RPC for “mark pesanan as `PERLU_PERHATIAN`”.
- Impact:
  - Next.js UI action cannot be implemented safely without inventing contract.
- OPEN QUESTION:
  - What is canonical write boundary for status watchlist/attention tagging?

### A-002 (high)
- Issue: Some routes in matrix rely on “query tambahan per modul bila ada” (conditional availability), not fixed contract.
- Evidence:
  - `integration_read_model_matrix.md` row `reseller/konsumen/[id]`.
- Impact:
  - Screen completion depends on unspecified backend contract; high integration uncertainty.
- Recommendation:
  - Promote optional reads into explicit query contracts or mark as out-of-scope.

### A-003 (medium)
- Issue: API contract docs still reference stale truth paths and outdated authority paths.
- Evidence:
  - `api_integration.md` uses `docs/contracts/schema_mapping.md` and `docs/contracts/rls_matrix.md` path style.
  - Actual files are under `docs/contracts/*`.
- Impact:
  - Agent/tool lookup failures and wrong cross-reference resolution.

# Missing backend contracts
1. Canonical action for watchlist/manual status tagging (`PERLU_PERHATIAN`) is not explicitly contracted. (blocker)
2. Reseller-facing correction history access path (if intended) is undefined. (high)
3. Explicit close-period carry-over readiness contract is implied but not concretely modeled as a read/check boundary in contracts. (high)
- Business mentions readiness/checklist in workflow layers; contract layer should expose deterministic checker boundary.

# Naming issues
## System-term naming quality
- Positives:
  - Predominantly `snake_case` DB/API contract fields.
  - Consistent entity names: `pesanan_konsumen`, `detail_pesanan_konsumen`, `setoran_konsumen`.
- Issues:
  - Stale path namespaces (`docs/contracts/schema_mapping.md`, `support/*`) in contract docs create authority ambiguity.
  - Mixed legacy term traces (`PROGRAM_*` error prefix remains while “program” term was deprecated to `pesanan`).
    - Source: `integration_contract_pack.md` error code section.

## UI label boundary
- Contract docs mostly keep user-facing messages in Indonesian (acceptable), but there is no explicit mapping table for system term vs UI label at contract layer.
- Recommendation:
  - Add a compact bilingual mapping appendix for high-frequency terms.

# Blockers before coding
1. **C-001**: Resolve enum model for `reseller_periode.status_periode` vs shared `StatusPeriode`.
2. **A-001**: Define canonical backend contract for manual watchlist/status actions.
3. Normalize stale contract references/authority paths to actual `docs/contracts/*` and `docs/truth/*` paths.
4. Decide whether reseller correction-history read is in scope and contract it explicitly if yes.
5. Confirm close-period readiness checks as explicit backend boundary contract (not just narrative).

If these blockers remain unresolved, Next.js implementation will require assumptions on status/state transitions and backend actions, which violates implementation safety.
