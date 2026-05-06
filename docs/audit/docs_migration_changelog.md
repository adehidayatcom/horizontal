# Docs Migration Changelog
## Date: 2026-05-06

Dokumen ini mencatat detail migrasi struktur folder dokumentasi dari flat/semi-structured menjadi domain-structured.

---

## 1. Ringkasan Perubahan
- Seluruh file di root `docs/` dipindahkan ke subfolder berdasarkan kategori.
- Folder `support/` dihapus; isinya dipindahkan ke `truth/` dan `quality/`.
- Folder `execution/`, `modules/`, dan `ui/` dipertahankan dengan penyesuaian konten.
- File baru `02-canonical_system_brief.md` ditambahkan sebagai "Source of Truth" ringkas.
- File baru `AGENTS.md` ditambahkan di root proyek untuk instruksi AI Agent.
- Internal links di dokumen utama (`prd.md`, `01-decision_log.md`, `system_maps.md`, `00-start-here.md`) telah diperbarui.

---

## 2. Detail Perpindahan File

### Product Domain
- `docs/product/prd.md` -> `docs/product/prd.md`
- `docs/product/period_workflow.md` -> `docs/product/period_workflow.md`
- `docs/product/program_workflow.md` -> `docs/product/program_workflow.md`
- `docs/product/data_flow.md` -> `docs/product/data_flow.md`
- `docs/product/edge_cases.md` -> `docs/product/edge_cases.md`

### Contracts
- `docs/contracts/schema_mapping.md` -> `docs/contracts/schema_mapping.md`
- `docs/contracts/business_contracts.md` -> `docs/contracts/business_contracts.md`
- `docs/contracts/query_contracts.md` -> `docs/contracts/query_contracts.md`
- `docs/contracts/rls_matrix.md` -> `docs/contracts/rls_matrix.md`
- `docs/contracts/api_integration.md` -> `docs/contracts/api_integration.md`
- `docs/contracts/integration_contract_pack.md` -> `docs/contracts/integration_contract_pack.md`
- `docs/contracts/integration_read_model_matrix.md` -> `docs/contracts/integration_read_model_matrix.md`

### Truth & Audit
- `docs/truth/01-decision_log.md` -> `docs/truth/01-decision_log.md`
- `docs/truth/03-open_questions_register.md` -> `docs/truth/03-open_questions_register.md`
- `docs/truth/system_maps.md` -> `docs/truth/system_maps.md`
- `docs/truth/02-canonical_system_brief.md` (New File)
- `docs/audit/docs_structure_audit.md` (New File)
- `docs/audit/docs_migration_changelog.md` (New File)

### Quality & Setup
- `docs/testing_strategy.md` -> `docs/quality/testing_strategy.md`
- `docs/definition_of_done.md` -> `docs/quality/definition_of_done.md`
- `docs/implementation_guardrails.md` -> `docs/quality/implementation_guardrails.md`
- `docs/support/checklist.md` -> `docs/quality/checklist.md`
- `docs/environment.md` -> `docs/setup/environment.md`
- `docs/setup_development_environment.md` -> `docs/setup/setup_development_environment.md`

### Frontend & UI
- `docs/frontend_architecture.md` -> `docs/frontend/frontend_architecture.md`
- `docs/frontend_component_contracts.md` -> `docs/frontend/frontend_component_contracts.md`
- `docs/component_patterns.md` -> `docs/frontend/component_patterns.md`
- `docs/navigation_and_period_setup_ui.md` -> `docs/frontend/navigation_and_period_setup_ui.md`

---

## 3. Status Link Internal
- [x] `docs/00-start-here.md`
- [x] `docs/truth/system_maps.md`
- [x] `docs/product/prd.md`
- [x] `docs/truth/01-decision_log.md`
- [ ] Dokumen modul (`docs/modules/*.md`) - *TBD (akan diperbarui on-demand saat pengerjaan modul)*
- [ ] Dokumen eksekusi (`docs/execution/*.md`) - *TBD*

---

## 4. Validasi Final
Struktur baru ini dirancang untuk memudahkan AI Agent (Codex) dalam mengidentifikasi batasan modul dan "Truth" tanpa risiko salah tafsir karena file yang tercampur. Folder `support/` telah dikosongkan untuk menghindari ambiguitas antara "dokumen pendukung" dan "dokumen kontrak".
