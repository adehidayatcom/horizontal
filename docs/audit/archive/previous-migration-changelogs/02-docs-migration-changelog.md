# 02 Docs Migration Changelog

Tanggal: 2026-05-07
Scope: eksekusi migrasi struktur berdasarkan `docs/audit/01-docs-migration-plan.md`.

## Perubahan yang Diterapkan
1. Rename file entry point:
- `docs/README.md` -> `docs/00-start-here.md`

2. Penomoran layer truth:
- `docs/truth/decision_log.md` -> `docs/truth/01-decision_log.md`
- `docs/truth/canonical_system_brief.md` -> `docs/truth/02-canonical_system_brief.md`
- `docs/truth/open_questions_register.md` -> `docs/truth/03-open_questions_register.md`

3. Arsip dokumen draft truth:
- `docs/truth/canonical-system-brief.DRAFT.md` -> `docs/truth/archive/canonical-system-brief.DRAFT.md`

4. Penomoran laporan audit content:
- `docs/audit/content/documentation_audit_report.md` -> `docs/audit/content/09-documentation-audit-report.md`

5. Update internal link di seluruh `docs/**/*.md` agar mengikuti path terbaru hasil migrasi di atas.

## Perubahan yang Sengaja Tidak Diterapkan (Ambigu)
1. Batch rename file audit top-level berikut tidak dieksekusi pada run ini:
- `docs/audit/docs_structure_audit.md` -> `docs/audit/02-docs-structure-audit.md`
- `docs/audit/docs_consistency_report.md` -> `docs/audit/03-docs-consistency-report.md`
- `docs/audit/docs_migration_changelog.md` -> `docs/audit/04-docs-migration-changelog.md`

Alasan:
- Instruksi output saat ini meminta file hasil berada di `docs/audit/02-docs-migration-changelog.md`.
- Rename batch audit di atas berpotensi bentrok urutan penomoran untuk file changelog pada eksekusi yang sama.
- Sesuai aturan "jika path meragukan, catat di changelog dan jangan tebak", perubahan ini ditunda.

2. Rename opsional berikut tidak dieksekusi:
- `docs/audit/product_truth_audit.md` -> `docs/audit/product-truth-audit.md`

Alasan:
- Di plan ditandai opsional, sehingga tidak dipaksakan pada eksekusi wajib.

## Validasi Ringkas
- File hasil rename utama terdeteksi di lokasi baru.
- Tidak ada referensi tersisa ke nama file lama untuk item yang dipindahkan pada batch wajib (truth + start-here + audit content report).
- Tidak ada perubahan isi business rules; hanya operasi struktur file dan pembaruan tautan internal.
