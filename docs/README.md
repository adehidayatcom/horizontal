# Docs Index
## Paket Lebaran Mumpuni

Dokumen ini adalah pintu masuk utama untuk membaca folder `docs/`.

Tujuannya:

- membantu developer atau AI coding agent menemukan dokumen yang tepat lebih cepat
- memisahkan sumber kebenaran dari dokumen pendukung
- menjaga folder `docs/` tetap rapi saat jumlah dokumen bertambah

Dokumen ini tidak menetapkan aturan bisnis baru. Ia hanya menjadi peta baca dan peta peran dokumen.

---

## 1. Cara Memakai Folder `docs/`

Gunakan prinsip berikut:

- mulai dari dokumen produk dan domain lebih dulu
- lanjut ke kontrak build dan kontrak integrasi
- baru masuk ke execution blueprint atau blueprint modul
- jika ada konflik, cek `support/decision_log.md`
- jika ada area yang masih belum jelas, cek `support/open_questions_register.md`

---

## 2. Kelompok Dokumen

### Product Truth

Dokumen inti kebutuhan bisnis dan aturan domain:

- `prd.md`
- `product_truth_audit.md`
- `business_contracts.md`
- `schema_mapping.md`
- `query_contracts.md`
- `rls_matrix.md`
- `edge_cases.md`
- `period_workflow.md`
- `program_workflow.md`
- `data_flow.md`

### Build Contract

Dokumen kontrak implementasi lintas frontend, backend, dan integrasi:

- `frontend_architecture.md`
- `frontend_component_contracts.md`
- `component_patterns.md`
- `navigation_and_period_setup_ui.md`
- `api_integration.md`
- `integration_contract_pack.md`
- `integration_read_model_matrix.md`
- `environment.md`
- `setup_development_environment.md`
- `testing_strategy.md`
- `implementation_guardrails.md`
- `definition_of_done.md`

### UI Reference

Dokumen referensi bentuk halaman, density, dan interaksi:

- `ui/admin_dashboard_uiux.md`
- `ui/reseller_uiux.md`
- `ui/modernize_shell_guide.md`

### Execution Blueprint

Dokumen blueprint eksekusi lintas area:

- `execution/frontend_plan.md`
- `execution/backend_plan.md`
- `execution/roadmap.md`
- `execution/sprint_control_board.md`
- `execution/agent_prompt_templates.md`
- `execution/task_kit_standard.md`
- `execution/work_breakdown.md`
- `execution/risk_register.md`
- `execution/assessment.md`

### Module Blueprint

Dokumen blueprint modul prioritas:

- `modules/auth.md`
- `modules/master_periodik.md`
- `modules/periode.md`
- `modules/reseller.md`
- `modules/program_order.md`
- `modules/setoran.md`
- `modules/gudang.md`
- `modules/keuangan.md`
- `modules/dashboard_laporan.md`

### Control and Support

Dokumen kontrol keputusan, pertanyaan, dan orientasi cepat:

- `support/decision_log.md`
- `support/open_questions_register.md`
- `support/system_maps.md`
- `support/checklist.md`

---

## 3. Urutan Baca yang Disarankan

### Untuk Orchestrator / Reviewer

1. `prd.md`
2. `product_truth_audit.md`
3. `support/decision_log.md`
4. `support/open_questions_register.md`
5. `support/system_maps.md`
6. `execution/roadmap.md`
7. `execution/sprint_control_board.md`
8. `execution/agent_prompt_templates.md`

### Untuk Frontend Agent

1. `prd.md`
2. `query_contracts.md`
3. `frontend_architecture.md`
4. `frontend_component_contracts.md`
5. `navigation_and_period_setup_ui.md`
6. `integration_contract_pack.md`
7. `integration_read_model_matrix.md`
8. blueprint modul yang relevan

### Untuk Backend Agent

1. `prd.md`
2. `business_contracts.md`
3. `schema_mapping.md`
4. `query_contracts.md`
5. `rls_matrix.md`
6. `execution/backend_plan.md`
7. `integration_contract_pack.md`
8. blueprint modul yang relevan

---

## 4. Aturan Pengelolaan Docs

- nama file memakai `lowercase_snake_case`
- kontrak aktif tidak boleh dicampur dengan catatan kerja sementara
- dokumen support tidak boleh menyaingi sumber kebenaran utama
- dokumen baru harus masuk kelompok yang jelas
- execution blueprint boleh berkembang, tetapi product truth dan build contract harus lebih stabil

---

## 5. Dokumen yang Paling Sering Dibuka

Jika hanya butuh orientasi cepat, buka:

1. `prd.md`
2. `support/decision_log.md`
3. `support/system_maps.md`
4. `integration_contract_pack.md`
5. `execution/roadmap.md`
6. `execution/sprint_control_board.md`

---

## 6. Status Folder `docs/` Saat Ini

Kondisi yang sudah tercapai:

- nama file sudah konsisten `lowercase_snake_case`
- kontrak frontend dan backend sudah dipisahkan
- blueprint modul prioritas utama sudah tersedia
- decision log, open questions, dan testing strategy sudah tersedia
- dokumen historis berbasis agent lama sudah disingkirkan

Catatan:

- `README.md` di root repo adalah entry point umum repo
- dokumen ini adalah entry point khusus untuk folder `docs/`
