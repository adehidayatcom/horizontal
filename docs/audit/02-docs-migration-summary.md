# 02 Docs Migration Summary

## Ringkasan
Migrasi struktur `docs/` memecah dokumen ke layer yang lebih jelas: `truth`, `product`, `contracts`, `frontend`, `ui`, `modules`, `execution`, `quality`, `setup`, dan `audit`.

Dokumen ini menggantikan changelog migrasi lama yang sebelumnya terpisah dan tumpang tindih. Dokumen ini sengaja menjadi satu-satunya ringkasan migrasi yang dipertahankan secara aktif.

## Struktur Sebelum Migrasi
- Banyak referensi masih memakai path root-level lama seperti `docs/prd.md` dan `docs/schema_mapping.md`.
- Namespace lama seperti `docs/support/*` masih muncul di sejumlah dokumen.
- Layer truth, product, contracts, frontend, quality, dan setup belum konsisten dipisah per folder.
- Folder audit berisi campuran audit struktur, changelog migrasi, dan batch audit isi lama.

## Struktur Setelah Migrasi
- Dokumen dipisah ke layer yang jelas: `truth`, `product`, `contracts`, `frontend`, `ui`, `modules`, `execution`, `quality`, `setup`, dan `audit`.
- Source of truth aktif memakai nama file aktual dan bernomor konsisten.
- Folder `docs/audit/` dipersempit menjadi area audit aktif yang minimal.
- Audit isi lama tidak dipakai lagi dan audit berikutnya harus dimulai ulang dari nol.

## File dan Folder Utama yang Dipindahkan
- Dokumen truth dipusatkan di `docs/truth/`.
- Dokumen product dipusatkan di `docs/product/`.
- Dokumen contracts dipusatkan di `docs/contracts/`.
- Dokumen frontend/UI dipisah ke `docs/frontend/` dan `docs/ui/`.
- Dokumen kualitas dipusatkan di `docs/quality/`.
- Dokumen setup dipusatkan di `docs/setup/`.
- Entry docs aktif memakai `docs/README.md`.
- Area audit lama dan content audit mentah tidak lagi dipakai sebagai area aktif.

## Perubahan Penamaan Penting
- Source of truth aktif memakai nama file aktual:
  - `docs/truth/01-decision_log.md`
  - `docs/truth/02-canonical_system_brief.md`
  - `docs/truth/03-open_questions_register.md`
- Dokumen truth utama dinormalkan ke pola nomor + nama file aktual.
- Dokumen audit struktur aktif dinormalkan menjadi `docs/audit/01-docs-structure-audit.md`.
- Ringkasan migrasi aktif dinormalkan menjadi `docs/audit/02-docs-migration-summary.md`.
- Changelog migrasi lama digabung menjadi dokumen ini.

## Aturan Penomoran yang Diterapkan
- Penomoran dipakai untuk dokumen yang memang punya urutan baca jelas pada layer truth dan audit aktif.
- Dokumen modul/domain tidak diberi nomor.
- Audit isi lama tidak dipertahankan sebagai batch aktif.

## Stale Link yang Diperbaiki
- Referensi namespace support lama diarahkan ke path truth yang aktif.
- Referensi root-level lama untuk product dan contracts diarahkan ke folder aktual masing-masing.
- Referensi source of truth dengan nama file lama/non-aktual diperbaiki ke:
  - `docs/truth/01-decision_log.md`
  - `docs/truth/02-canonical_system_brief.md`
  - `docs/truth/03-open_questions_register.md`

## Status Akhir Migrasi
- Struktur `docs/` sudah terpisah per layer.
- `docs/audit/` sudah dibersihkan agar hanya menyisakan file audit aktif minimal.
- Audit isi lama sudah dibuang dari area aktif dan harus diulang dari nol saat siklus audit berikutnya dimulai.
