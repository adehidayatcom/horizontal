# 01 Docs Migration Plan

## Tujuan
Merapikan struktur folder `docs/` agar mudah dibaca oleh Codex sebelum audit isi dokumen.

Batasan:
- Tidak audit isi dokumen.
- Tidak mengubah business rules.
- Tidak memindahkan file pada tahap ini.

## Ringkasan Struktur Usulan
Struktur target tetap memakai kategori berikut:
- `truth`
- `product`
- `contracts`
- `frontend`
- `ui`
- `modules`
- `execution`
- `quality`
- `setup`
- `audit`

## Aturan Penomoran
### File yang perlu diberi nomor
Hanya dokumen yang memang punya urutan baca ketat:
- `docs/00-start-here.md` -> `docs/00-start-here.md`
- `docs/truth/01-decision_log.md` -> `docs/truth/01-decision_log.md`
- `docs/truth/02-canonical_system_brief.md` -> `docs/truth/02-canonical_system_brief.md`
- `docs/truth/03-open_questions_register.md` -> `docs/truth/03-open_questions_register.md`

### File yang tidak perlu diberi nomor
- Semua file `docs/modules/*.md`.
- Semua file kategori `product`, `contracts`, `frontend`, `ui`, `execution`, `quality`, `setup`.
- Semua file audit kecuali yang memang sudah berfungsi sebagai urutan laporan batch (contoh di `docs/audit/content/`).

## Tabel Migrasi (Rencana, Belum Dieksekusi)
| Path lama | Path baru disarankan | Alasan perubahan |
|---|---|---|
| `docs/00-start-here.md` | `docs/00-start-here.md` | Menjadikan entry point eksplisit untuk agen. |
| `docs/truth/01-decision_log.md` | `docs/truth/01-decision_log.md` | Urutan baca paling tinggi di layer truth. |
| `docs/truth/02-canonical_system_brief.md` | `docs/truth/02-canonical_system_brief.md` | Mengikuti urutan source of truth setelah decision log. |
| `docs/truth/03-open_questions_register.md` | `docs/truth/03-open_questions_register.md` | Menutup urutan baca truth (setelah keputusan dan brief). |
| `docs/truth/archive/canonical-system-brief.DRAFT.md` | `docs/truth/archive/canonical-system-brief.DRAFT.md` | Dokumen draft perlu dipisah agar tidak bentrok dengan canonical aktif. |
| `docs/truth/system_maps.md` | `docs/truth/system_maps.md` (tetap) | Referensi peta, bukan urutan baca wajib. |
| `docs/product/prd.md` | `docs/product/prd.md` (tetap) | Dokumen inti product, nama sudah jelas. |
| `docs/product/program_workflow.md` | `docs/product/program_workflow.md` (tetap) | Nama dipertahankan untuk kompatibilitas referensi lama. |
| `docs/product/period_workflow.md` | `docs/product/period_workflow.md` (tetap) | Sudah konsisten sebagai workflow periode. |
| `docs/product/data_flow.md` | `docs/product/data_flow.md` (tetap) | Sudah sesuai kategori product. |
| `docs/product/edge_cases.md` | `docs/product/edge_cases.md` (tetap) | Sudah sesuai kategori product. |
| `docs/contracts/schema_mapping.md` | `docs/contracts/schema_mapping.md` (tetap) | Kontrak utama, penamaan sudah tepat. |
| `docs/contracts/business_contracts.md` | `docs/contracts/business_contracts.md` (tetap) | Kontrak utama, penamaan sudah tepat. |
| `docs/contracts/query_contracts.md` | `docs/contracts/query_contracts.md` (tetap) | Kontrak utama, penamaan sudah tepat. |
| `docs/contracts/integration_contract_pack.md` | `docs/contracts/integration_contract_pack.md` (tetap) | Kontrak lintas layer, sudah tepat. |
| `docs/contracts/rls_matrix.md` | `docs/contracts/rls_matrix.md` (tetap) | Kontrak keamanan data, sudah tepat. |
| `docs/contracts/integration_read_model_matrix.md` | `docs/contracts/integration_read_model_matrix.md` (tetap) | Matriks operasional baca, masih dalam layer contracts. |
| `docs/contracts/api_integration.md` | `docs/contracts/api_integration.md` (tetap) | Panduan integrasi, masih relevan di contracts. |
| `docs/frontend/frontend_architecture.md` | `docs/frontend/frontend_architecture.md` (tetap) | Sudah tepat di kategori frontend. |
| `docs/frontend/frontend_component_contracts.md` | `docs/frontend/frontend_component_contracts.md` (tetap) | Kontrak komponen frontend, sudah tepat. |
| `docs/frontend/navigation_and_period_setup_ui.md` | `docs/frontend/navigation_and_period_setup_ui.md` (tetap) | Kontrak UI flow berbasis frontend/nav, sudah tepat. |
| `docs/frontend/component_patterns.md` | `docs/frontend/component_patterns.md` (tetap) | Pola implementasi frontend, sudah tepat. |
| `docs/ui/modernize_shell_guide.md` | `docs/ui/modernize_shell_guide.md` (tetap) | Panduan UI shell, tepat di kategori ui. |
| `docs/ui/admin_dashboard_uiux.md` | `docs/ui/admin_dashboard_uiux.md` (tetap) | Spesifikasi UI admin, tepat di kategori ui. |
| `docs/ui/reseller_uiux.md` | `docs/ui/reseller_uiux.md` (tetap) | Spesifikasi UI reseller, tepat di kategori ui. |
| `docs/modules/auth.md` | `docs/modules/auth.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/dashboard_laporan.md` | `docs/modules/dashboard_laporan.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/gudang.md` | `docs/modules/gudang.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/keuangan.md` | `docs/modules/keuangan.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/master_periodik.md` | `docs/modules/master_periodik.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/periode.md` | `docs/modules/periode.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/program_order.md` | `docs/modules/program_order.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/reseller.md` | `docs/modules/reseller.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/modules/setoran.md` | `docs/modules/setoran.md` (tetap) | Dokumen modul/domain, tidak diberi nomor. |
| `docs/execution/*.md` | `docs/execution/*.md` (tetap) | Satu layer eksekusi; cukup urut via isi dokumen, tidak perlu nomor filename. |
| `docs/quality/*.md` | `docs/quality/*.md` (tetap) | Satu layer quality; tidak butuh urutan baca kaku per filename. |
| `docs/setup/*.md` | `docs/setup/*.md` (tetap) | Satu layer setup; tidak butuh urutan baca kaku per filename. |
| `docs/audit/docs_structure_audit.md` | `docs/audit/02-docs-structure-audit.md` | Supaya batch audit top-level punya urutan baca eksplisit setelah migration plan. |
| `docs/audit/docs_consistency_report.md` | `docs/audit/03-docs-consistency-report.md` | Menjaga urutan review hasil audit struktural. |
| `docs/audit/docs_migration_changelog.md` | `docs/audit/04-docs-migration-changelog.md` | Menempatkan changelog setelah plan + audit + consistency. |
| `docs/audit/product_truth_audit.md` | `docs/audit/product-truth-audit.md` (opsional rename) | Konsistensi separator nama file (`kebab-case`) tanpa ubah kategori. |
| `docs/audit/content/09-documentation-audit-report.md` | `docs/audit/content/09-documentation-audit-report.md` | Menyelaraskan pola urutan laporan di folder content. |

## File Dengan Posisi Ambigu
| File | Ambiguitas | Saran |
|---|---|---|
| `docs/contracts/api_integration.md` | Bisa dianggap `contracts` atau `frontend`/`execution` karena sifatnya panduan implementasi. | Tetap di `contracts` sebagai pagar boundary lintas layer. |
| `docs/contracts/integration_read_model_matrix.md` | Bisa dianggap `contracts` atau `frontend` karena sangat screen-oriented. | Tetap di `contracts` karena mendefinisikan kontrak sumber data per screen. |
| `docs/truth/system_maps.md` | Bisa dianggap `truth` atau `audit`/`execution` karena berisi peta lintas dokumen. | Tetap di `truth` sebagai dokumen orientasi lintas layer. |
| `docs/execution/assessment.md` | Bisa dianggap `execution` atau `audit` karena berupa penilaian kesiapan. | Tetap di `execution` karena dipakai sebagai snapshot operasional eksekusi. |
| `docs/quality/checklist.md` | Bisa dianggap `quality` atau `setup` karena memuat readiness lintas area. | Tetap di `quality` karena fungsinya quality gate. |
| `docs/truth/archive/canonical-system-brief.DRAFT.md` | Bentrok dengan canonical aktif pada layer truth. | Pindahkan ke subfolder `truth/archive/` saat eksekusi migrasi. |

## Catatan Eksekusi Tahap Lanjut
Dokumen ini hanya rencana struktur. Eksekusi rename/move dilakukan setelah approval, dengan urutan aman:
1. Rename file bernomor pada `truth`.
2. Update semua referensi path lintas dokumen.
3. Baru rapikan file audit yang disepakati perlu nomor.
4. Verifikasi ulang tidak ada broken reference.
