# Docs Structure Audit
## Project: Paket Lebaran Mumpuni

Dokumen ini menganalisis struktur dokumentasi saat ini setelah reorganisasi awal dan memberikan rekomendasi untuk keamanan implementasi oleh AI (Codex/Agent).

---

## 1. Klasifikasi Dokumen Saat Ini

Berdasarkan struktur folder terbaru, berikut adalah klasifikasi file:

| Kategori | Folder | File Utama |
|---|---|---|
| **Truth** | `docs/truth/` | `02-canonical_system_brief.md`, `01-decision_log.md`, `03-open_questions_register.md` |
| **Product** | `docs/product/` | `prd.md`, `period_workflow.md`, `program_workflow.md`, `data_flow.md`, `edge_cases.md` |
| **Contracts** | `docs/contracts/` | `schema_mapping.md`, `business_contracts.md`, `query_contracts.md`, `rls_matrix.md`, `api_integration.md`, `integration_contract_pack.md`, `integration_read_model_matrix.md` |
| **Frontend** | `docs/frontend/` | `frontend_architecture.md`, `frontend_component_contracts.md`, `component_patterns.md`, `navigation_and_period_setup_ui.md` |
| **UI** | `docs/ui/` | `admin_dashboard_uiux.md`, `reseller_uiux.md`, `modernize_shell_guide.md` |
| **Modules** | `docs/modules/` | `auth.md`, `master_periodik.md`, `periode.md`, `reseller.md`, `program_order.md`, `setoran.md`, `gudang.md`, `keuangan.md`, `dashboard_laporan.md` |
| **Execution** | `docs/execution/` | `roadmap.md`, `sprint_control_board.md`, `frontend_plan.md`, `backend_plan.md`, `work_breakdown.md`, `task_kit_standard.md`, `agent_prompt_templates.md`, `risk_register.md`, `assessment.md` |
| **Quality** | `docs/quality/` | `testing_strategy.md`, `definition_of_done.md`, `implementation_guardrails.md` |
| **Setup** | `docs/setup/` | `environment.md`, `setup_development_environment.md` |
| **Audit** | `docs/audit/` | `product_truth_audit.md`, `docs_consistency_report.md` |

---

## 2. Temuan Audit (Issue Identification)

### A. File Yatim (Orphaned Files)
Ditemukan file yang masih berada di folder lama (`support/`) dan belum masuk ke struktur baru:
1.  **`docs/truth/system_maps.md`**: Dokumen ini sangat krusial karena berisi peta hubungan antar dokumen. Status "Support" merendahkan kepentingannya.
2.  **`docs/support/checklist.md`**: Dokumen operasional yang tidak memiliki kategori tetap di struktur baru.

### B. Ambiguitas Lokasi
1.  **`navigation_and_period_setup_ui.md`**: Berada di `frontend/`, namun isinya sangat berkaitan dengan `ui/`.
2.  **`product_truth_audit.md`**: Berada di `audit/`. Ini benar secara kategori, namun isinya sering dijadikan referensi "Truth" oleh agen.

### C. Duplikasi / Overlap
1.  **`02-canonical_system_brief.md`** vs **`prd.md`**: Terdapat overlap informasi bisnis. 
    *   *Risiko*: Agen mungkin membaca salah satu dan melewatkan detail penting di yang lain.
    *   *Rekomendasi*: `02-canonical_system_brief.md` harus secara eksplisit menyatakan dirinya sebagai "Index/Summary" dan menautkan ke `prd.md` untuk detail.

---

## 3. Usulan Struktur yang Lebih Aman (Refined Structure)

Untuk meningkatkan "Safety" bagi AI (Codex), struktur harus memisahkan secara tegas antara **Aturan (Truth)**, **Kontrak (Specs)**, dan **Rencana Kerja (Execution)**.

### Perubahan yang Diusulkan:
1.  Pindahkan `system_maps.md` ke `truth/` atau folder root `docs/` sebagai `00-start-here.md` (Sudah dilakukan sebagian di `docs/00-start-here.md`).
2.  Pindahkan `checklist.md` ke `quality/`.
3.  Hapus folder `support/` setelah file di dalamnya dipindahkan.

---

## 4. Migration Plan (Old -> New Path)

> [!IMPORTANT]
> Jangan lakukan pemindahan file sekarang. Ini hanya rencana migrasi untuk persetujuan.

| File Sumber | Path Saat Ini | Usulan Path Baru | Alasan |
|---|---|---|---|
| `system_maps.md` | `docs/truth/system_maps.md` | `docs/truth/system_maps.md` | Peta sistem adalah bagian dari kebenaran struktur proyek. |
| `checklist.md` | `docs/support/checklist.md` | `docs/quality/checklist.md` | Bagian dari standar kualitas/verifikasi. |

---

## 5. Kesimpulan Audit
Struktur saat ini sudah jauh lebih baik dan siap untuk implementasi. Dengan memindahkan 2 file yatim di atas, repositori akan memiliki kategori yang 100% konsisten.

**Rekomendasi Utama**: Selalu arahkan AI Agent untuk membaca `AGENTS.md` -> `truth/02-canonical_system_brief.md` -> `truth/system_maps.md` sebelum mengerjakan modul apapun.
