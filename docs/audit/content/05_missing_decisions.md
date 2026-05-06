# 05 Missing Decisions

## OPEN QUESTION MQ-01
Which authority order is canonical for conflict resolution?
- Candidates in conflict:
  - AGENTS source-of-truth order
  - `docs/contracts/integration_contract_pack.md` `## 2. Urutan Otoritas`
- Risk: silent conflict resolution by different teams.

## OPEN QUESTION MQ-02
Is `reseller_periode.status_periode` intentionally separate from core `StatusPeriode`?
- Evidence:
  - `docs/contracts/schema_mapping.md` includes `DITUTUP_MANUAL`
  - `docs/contracts/integration_contract_pack.md` shared enum does not
- Risk: broken type generation/contracts.

## OPEN QUESTION MQ-03
For first wave, is `barang_periode.budget_belanja` required or optional?
- Evidence conflict:
  - `DL-028` optional
  - `docs/modules/master_periodik.md` mandatory

## OPEN QUESTION MQ-04
For first wave, do we need warning UX on akun kas update during `AKTIF` period?
- Evidence conflict:
  - `DL-027` no extra warning
  - `docs/modules/master_periodik.md` warning needed
