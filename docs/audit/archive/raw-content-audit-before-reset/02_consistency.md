# 02 Consistency

## Finding S-01: Source-of-truth precedence conflict
- Checked:
  - AGENTS instruction order
  - `docs/contracts/integration_contract_pack.md` section `## 2. Urutan Otoritas`
- Conflict:
  - AGENTS: truth docs first.
  - Integration contract pack: `schema_mapping` first.
- Implementation risk: inconsistent conflict resolution in implementation and reviews.
- Status: **OPEN QUESTION**.

## Finding S-02: Enum/status inconsistency for period-related lifecycle
- Checked:
  - `docs/contracts/schema_mapping.md` section `### 2.1a reseller_periode`
  - `docs/contracts/integration_contract_pack.md` section `## 5.1 Status Periode`
- Conflict:
  - `reseller_periode.status_periode` includes `DITUTUP_MANUAL`.
  - shared status type omits `DITUTUP_MANUAL`.
- Risk: DTO/type mismatch and broken filters.
- Status: **OPEN QUESTION**.

## Finding S-03: Decision drift in module docs
- Checked:
  - `docs/truth/01-decision_log.md` (`DL-027`, `DL-028`)
  - `docs/modules/master_periodik.md` (tail decision notes)
- Conflict:
  - `DL-027`: no extra akun kas warning needed.
  - module doc: warning still required.
  - `DL-028`: `budget_belanja` not mandatory first wave.
  - module doc: `budget_belanja` mandatory first wave.
- Risk: frontend form validation and backend requirement diverge.
- Status: **OPEN QUESTION**.
