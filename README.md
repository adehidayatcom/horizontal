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
- arah produk: [docs/product/prd.md](docs/product/prd.md)
- keputusan penting: [docs/truth/01-decision_log.md](docs/truth/01-decision_log.md)
- pertanyaan terbuka: [docs/truth/03-open_questions_register.md](docs/truth/03-open_questions_register.md)
- peta sistem: [docs/truth/system_maps.md](docs/truth/system_maps.md)

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

- setup Supabase lokal/cloud dijelaskan di [docs/setup/setup_development_environment.md](/D:/WEBAPP/horizontal/docs/setup/setup_development_environment.md:1)
- standar environment dijelaskan di [docs/setup/environment.md](/D:/WEBAPP/horizontal/docs/setup/environment.md:1)
- strategi testing dijelaskan di [docs/quality/testing_strategy.md](/D:/WEBAPP/horizontal/docs/quality/testing_strategy.md:1)

## Struktur Dokumen yang Penting

- product truth dan domain: `docs/product/prd.md`, `docs/contracts/business_contracts.md`, `docs/contracts/schema_mapping.md`, `docs/contracts/query_contracts.md`
- build contract: `docs/frontend/frontend_architecture.md`, `docs/execution/backend_plan.md`, `docs/contracts/integration_contract_pack.md`
- blueprint modul: `docs/modules/*.md`

## Catatan Repo

- package manager resmi: `pnpm`
- repo ini masih membawa banyak modul demo dari template `Modernize`
- jangan menganggap source code saat ini sudah merepresentasikan arsitektur final; sumber kebenaran utama tetap ada di folder `docs/`
