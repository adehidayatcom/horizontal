# 07 Risk to Database / Schema / API

## D-01 Enum mismatch risk
- Evidence:
  - `docs/contracts/schema_mapping.md` (`reseller_periode.status_periode` includes `DITUTUP_MANUAL`)
  - `docs/contracts/integration_contract_pack.md` `StatusPeriode`
- Impact: incompatible API schema and frontend types.

## D-02 Historical correction guard risk
- Evidence: `DL-010`, `docs/contracts/business_contracts.md` section `## 18`.
- Impact: inconsistent correction behavior on `SELESAI` period data.

## D-03 Read model snapshot discipline risk
- Evidence: `DL-014`, `docs/contracts/query_contracts.md` read model expectations.
- Impact: reports drift if joins rely on mutable master data.

## D-04 Reference-path integrity risk for contract adoption
- Evidence: `docs/contracts/integration_contract_pack.md` mentions `docs/contracts/schema_mapping.md` and `support/*` paths.
- Impact: automation/scripts/readers may not locate authoritative docs.
