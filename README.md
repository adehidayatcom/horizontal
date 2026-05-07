# Paket Lebaran Mumpuni

Repo ini adalah fondasi aplikasi operasional Paket Lebaran Mumpuni

## Status Saat Ini

Yang sudah siap di level dokumen:

- product truth dan kontrak bisnis inti
- kontrak frontend, backend, dan integrasi lintas layer
- blueprint modul prioritas dari `auth` sampai `dashboard_laporan`
- decision log dan testing strategy

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

Quality gate lokal baseline repo saat ini:

```bash
pnpm run lint
pnpm run typecheck
pnpm run build
```

Status saat ini:

- `pnpm run build` lolos
- `pnpm run lint` lolos
- `pnpm run typecheck` lolos

Catatan:

- setup Supabase lokal/cloud dijelaskan di [docs/setup/setup_development_environment.md](docs/setup/setup_development_environment.md)
- standar environment dijelaskan di [docs/setup/environment.md](docs/setup/environment.md)
- strategi testing dijelaskan di [docs/quality/testing_strategy.md](docs/quality/testing_strategy.md)

## Struktur Dokumen yang Penting

- product truth dan domain: `docs/product/prd.md`, `docs/contracts/business_contracts.md`, `docs/contracts/schema_mapping.md`, `docs/contracts/query_contracts.md`
- build contract: `docs/frontend/frontend_architecture.md`, `docs/execution/backend_plan.md`, `docs/contracts/integration_contract_pack.md`
- blueprint modul: `docs/modules/*.md`

## Catatan Repo

- package manager resmi: `pnpm`
- repo ini masih membawa banyak modul demo dari template `Modernize`
- jangan menganggap source code saat ini sudah merepresentasikan arsitektur final; sumber kebenaran utama tetap ada di folder `docs/`
