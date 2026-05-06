# 08 Risk to Roles / Permissions / Security

## R-01 RLS and API-layer responsibility split is clear, but execution docs can still mislead
- Evidence:
  - `docs/truth/02-canonical_system_brief.md` section `## 4. Peran dan Izin`
  - `docs/contracts/schema_mapping.md` section `### RLS Wajib`
- Risk: stale references in execution docs reduce enforcement consistency.

## R-02 Ambiguous correction rights on closed periods
- Evidence:
  - `docs/truth/01-decision_log.md` `DL-010`
  - `docs/contracts/business_contracts.md` `## 18. RPC Koreksi`
- Risk: privilege escalation by broad correction endpoints without strict allowlist.
- Status: **OPEN QUESTION**.

## R-03 Auth/onboarding consistency risk
- Evidence:
  - `docs/truth/01-decision_log.md` `DL-023`
  - `docs/modules/auth.md` references stale `docs/truth/01-decision_log.md`
- Risk: inconsistent handling of `PENDING` reseller state across route guards.
