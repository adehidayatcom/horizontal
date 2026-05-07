# Integration Readiness Checklist
## Paket Lebaran Mumpuni

Checklist ini dipakai untuk menilai kesiapan integrasi repo sebelum implementasi besar dijalankan.

Dokumen ini bukan kontrak arsitektur utama. Fungsinya adalah checklist operasional singkat.

---

## 1. Product Alignment

- [ ] `prd.md` menjadi acuan kebutuhan produk
- [ ] `business_contracts.md` menjadi acuan aturan bisnis
- [ ] `schema_mapping.md` menjadi acuan struktur data
- [ ] `query_contracts.md` menjadi acuan kebutuhan query

---

## 2. Frontend Baseline

- [ ] frontend mengikuti `Next.js 16 App Router`
- [ ] shell UI resmi adalah `Modernize`
- [ ] integrasi shell mengikuti `frontend/frontend_architecture.md` dan `frontend/navigation_and_period_setup_ui.md`
- [ ] UI library utama adalah `Material-UI v7 + Emotion`
- [ ] data fetching frontend memakai `SWR`
- [ ] form handling memakai `Formik + Yup`

---

## 3. Package Manager

- [ ] package manager resmi adalah `pnpm`
- [ ] `package.json` memiliki field `packageManager`
- [ ] `pnpm-lock.yaml` tersedia
- [ ] `package-lock.json` tidak ada

---

## 4. Integration Contract

- [ ] `integration_contract_pack.md` sudah menjadi acuan lintas layer
- [ ] naming convention konsisten
- [ ] enum/status bersama konsisten
- [ ] request/response envelope konsisten
- [ ] error code standard konsisten

---

## 5. Data Readiness

- [ ] strategi backend mengacu ke `Supabase + RPC + RLS`
- [ ] folder `supabase/` direncanakan atau tersedia
- [ ] generated types diarahkan ke `src/types/database.ts`
- [ ] service role key tidak direncanakan masuk ke browser

---

## 6. UI Contract

- [ ] `frontend_component_contracts.md` menjadi acuan perilaku komponen
- [ ] `component_patterns.md` menjadi acuan pola implementasi
- [ ] `navigation_and_period_setup_ui.md` menjadi acuan navigasi dan periode
- [ ] `ui/admin_dashboard_uiux.md` menjadi acuan UI admin
- [ ] `ui/reseller_uiux.md` menjadi acuan UI reseller

---

## 7. Quality Gate

- [ ] repo memiliki `pnpm run lint`
- [ ] repo memiliki `pnpm run typecheck`
- [ ] repo memiliki `pnpm run build`
- [ ] strategi test tersedia sesuai tahap implementasi

---

## 8. Decision and Question Control

- [ ] keputusan lintas-dokumen penting tercatat di `truth/01-decision_log.md`
- [ ] tidak ada gap penting yang hanya hidup di catatan sementara
- [ ] modul yang belum diimplementasikan diberi status jelas

---

## 9. Status Review Saat Ini

Ringkasan status saat checklist ini diperbarui:

- kontrak produk, modul inti, dan route strategy sudah jauh lebih siap
- Priority 1 document blockers sudah ditutup
- Priority 2 document cleanup sudah ditutup
- strategi testing sekarang sudah punya dokumen khusus
- keputusan penting sudah terpusat di `truth/01-decision_log.md`
- final docs polish untuk Priority 1 dan Priority 2 sudah selesai: terminology konsisten, boundary frontend/backend jelas, artefak teks dibersihkan
- bootstrap Supabase, generated types, dan script test masih belum tersedia di repo fisik saat ini (pekerjaan fase coding)
