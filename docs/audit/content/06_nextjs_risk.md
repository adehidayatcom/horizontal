# 06 Risk to Next.js Implementation

## N-01 Boundary bypass risk
- Evidence: `docs/truth/01-decision_log.md` `DL-021`; `docs/contracts/api_integration.md` boundary rules.
- Risk: accidental direct Supabase browser reads/writes from client components.

## N-02 Type contract drift risk (`snake_case`)
- Evidence: `docs/contracts/integration_contract_pack.md` `## 3.1` and `## 17 Risiko`.
- Risk: DTO adapters or UI types silently convert naming, causing runtime mismatches.

## N-03 Route/menu drift risk
- Evidence: `docs/truth/01-decision_log.md` `DL-002`, `DL-003`, `DL-004`; stale references in execution/frontend docs.
- Risk: app router structure diverges from agreed route map.
