# Paket Lebaran Mumpuni

Repo ini adalah fondasi aplikasi operasional Paket Lebaran Mumpuni yang dibangun di atas:

- `Next.js 16` App Router
- shell `Modernize`
- `Material-UI v7`
- `SWR`
- `Formik + Yup`
- `Supabase + RPC + RLS`

Saat ini repo sudah melewati fase **document readiness** dan masuk ke fase **final docs polish**. Sumber kebenaran utama proyek ada di folder `docs/`, sementara source code masih membawa banyak artefak demo dari template `Modernize`.

## Status Saat Ini

Yang sudah siap di level dokumen:

- product truth dan kontrak bisnis inti
- kontrak frontend, backend, dan integrasi lintas layer
- blueprint modul prioritas dari `auth` sampai `dashboard_laporan`
- decision log, open questions register, dan testing strategy

Yang masih berupa target baseline repo:

- bootstrap `supabase/`
- generated type `src/types/database.ts`
- script `test` dan `e2e`
- pembersihan modul demo template di source code

## Entry Docs

Mulai dari dokumen ini:

- index docs: [docs/README.md](docs/README.md)
- arah produk: [docs/prd.md](docs/prd.md)
- keputusan penting: [docs/support/decision_log.md](docs/support/decision_log.md)
- pertanyaan terbuka: [docs/support/open_questions_register.md](docs/support/open_questions_register.md)
- peta sistem: [docs/support/system_maps.md](docs/support/system_maps.md)

## Setup Dasar

Install dependency:

```bash
pnpm install
```

Jalankan development server:

```bash
pnpm run dev
```

Quality gate yang saat ini tersedia:

```bash
pnpm run lint
pnpm run typecheck
pnpm run build
```

Catatan:

- setup Supabase lokal/cloud dijelaskan di [docs/setup_development_environment.md](/D:/WEBAPP/horizontal/docs/setup_development_environment.md:1)
- standar environment dijelaskan di [docs/environment.md](/D:/WEBAPP/horizontal/docs/environment.md:1)
- strategi testing dijelaskan di [docs/testing_strategy.md](/D:/WEBAPP/horizontal/docs/testing_strategy.md:1)

## Struktur Dokumen yang Penting

- product truth dan domain: `docs/prd.md`, `docs/business_contracts.md`, `docs/schema_mapping.md`, `docs/query_contracts.md`
- build contract: `docs/frontend_architecture.md`, `docs/execution/backend_plan.md`, `docs/integration_contract_pack.md`
- blueprint modul: `docs/modules/*.md`

## Catatan Repo

- package manager resmi: `pnpm`
- repo ini masih membawa banyak modul demo dari template `Modernize`
- jangan menganggap source code saat ini sudah merepresentasikan arsitektur final; sumber kebenaran utama tetap ada di folder `docs/`
