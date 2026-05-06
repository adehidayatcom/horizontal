# AGENTS.md

## Bahasa kerja
Dokumentasi produk, aturan bisnis, label UI, dan istilah aplikasi menggunakan bahasa Indonesia.

Instruksi teknis boleh memakai istilah Inggris jika istilah tersebut lebih umum dalam pengembangan software, misalnya:
- schema
- dataflow
- API contract
- permission
- role
- module
- frontend
- backend

## Aturan utama
Jangan mulai coding sebelum audit dokumentasi selesai.

## Hierarki sumber kebenaran
1. `docs/truth/01-decision_log.md`
2. `docs/truth/02-canonical_system_brief.md`
3. `docs/product/prd.md`
4. `docs/product/data_flow.md`
5. `docs/contracts/schema_mapping.md`
6. `docs/contracts/business_contracts.md`
7. Dokumen modul terkait di `docs/modules/`

## Jika ada konflik
Jika PRD, dataflow, schema, contract, atau module docs bertentangan:
- jangan memilih diam-diam
- tulis sebagai OPEN QUESTION
- beri opsi keputusan
- jelaskan dampak implementasi
- jangan mulai coding sampai keputusan masuk decision log

## Larangan
- Jangan mengarang business rules.
- Jangan mengganti istilah bisnis tanpa alasan.
- Jangan membuat arsitektur baru tanpa mendokumentasikan.
- Jangan mengubah isi dokumen saat task hanya meminta audit struktur.
