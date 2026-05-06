# 03 Ambiguity

## Finding A-01: Historical correction scope is not operationally precise
- Checked:
  - `docs/truth/01-decision_log.md` `DL-010`
  - `docs/contracts/business_contracts.md` `## 18. RPC Koreksi: koreksi_*`
- Ambiguity: "koreksi administratif non-finansial" is not mapped to an explicit allow/deny matrix by entity and field.
- Risk: different teams can classify the same correction differently.
- Status: **OPEN QUESTION**.

## Finding A-02: Internal API route rule has edge ambiguity for non-operational reads
- Checked:
  - `docs/truth/01-decision_log.md` `DL-021`
  - `docs/contracts/api_integration.md` boundary notes
- Ambiguity: scope of "helper non-bisnis" is not concretely enumerated.
- Risk: accidental direct browser Supabase queries in production screens.
- Status: **OPEN QUESTION**.
