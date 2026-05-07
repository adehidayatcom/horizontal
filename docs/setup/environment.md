# Environment Standard
## Paket Lebaran Mumpuni

Dokumen ini menetapkan standar environment kerja lokal untuk proyek `horizontal`.

Semua developer dan agen harus mengacu ke dokumen ini saat menyiapkan runtime, environment variable, quality gate, dan dependency tooling.

Dokumen ini membedakan dengan jelas antara:

- target baseline yang harus dituju repo
- status repo saat review dokumen ini ditulis

---

## 1. Baseline Teknis

Proyek ini dibangun dengan baseline berikut:

- Framework: `Next.js 16` dengan `App Router`
- UI shell: `Modernize` sebagai basis theme, layout, dan customizer
- UI library: `Material-UI v7` + `Emotion`
- Frontend data layer: `SWR`
- Form handling: `Formik` + `Yup`
- Backend/data source: `Supabase` dengan `RPC`, `RLS`, dan schema PostgreSQL

---

## 2. Runtime Wajib

| Kebutuhan | Versi | Catatan |
|---|---|---|
| Node.js | `22 LTS` direkomendasikan, minimal `20` | Untuk Next.js 16 dan tooling modern |
| pnpm | `10.x` direkomendasikan | Package manager standar proyek |
| Git | Stabil terbaru | Wajib untuk branch, review, dan diff |
| Docker Desktop | Stabil terbaru | Dibutuhkan jika memakai Supabase local |
| Supabase CLI | Stabil terbaru | Untuk init, start, migration, dan type generation |
| Browser Chromium/Chrome | Stabil terbaru | Untuk verifikasi UI dan testing manual |

---

## 3. Struktur File Environment

File lokal yang dipakai:

```txt
.env.local
```

Template yang harus tersedia di repo:

```txt
.env.example
```

Aturan:

- `.env.local` tidak boleh di-commit.
- Semua env yang dibaca browser harus memakai prefix `NEXT_PUBLIC_`.
- Secret server-only tidak boleh memakai `NEXT_PUBLIC_`.
- Nilai environment staging dan production harus dipisah dari local.

---

## 4. Environment Variables Minimum

| Nama | Wajib | Dipakai di | Catatan |
|---|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Ya | Browser + server | URL project Supabase local atau cloud |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Ya | Browser + server | Anon key untuk akses aman dengan RLS |
| `NEXT_PUBLIC_APP_ENV` | Tidak | Browser | Contoh: `local`, `staging`, `production` |

Larangan:

- Jangan menaruh `SUPABASE_SERVICE_ROLE_KEY` di browser.
- Jangan commit credential pribadi ke repo.
- Jangan membuat env browser tanpa prefix `NEXT_PUBLIC_`.

---

## 5. Setup Dependency

Command dasar:

```bash
pnpm install
```

Repo harus memakai `pnpm-lock.yaml` sebagai satu-satunya lockfile aktif.

Status repo saat dokumen ini ditulis:

- `pnpm-lock.yaml` sudah ada
- `package-lock.json` tidak ada
- `package.json` sudah memiliki `packageManager`

---

## 6. Supabase Local

Jika development memakai Supabase local, command yang dipakai:

```bash
pnpm exec supabase init
pnpm exec supabase start
pnpm exec supabase stop
pnpm exec supabase status
pnpm exec supabase db reset
pnpm exec supabase gen types typescript --local > src/types/database.ts
```

Aturan database:

- Semua perubahan schema harus berupa migration SQL di `supabase/migrations/`.
- Seed data lokal disimpan di `supabase/seed.sql`.
- RLS harus aktif sejak awal, bukan ditambahkan belakangan.
- RPC transaksi bisnis harus dibuat sebagai database function, bukan logika final di browser.

Status repo saat dokumen ini ditulis:

- folder `supabase/` belum tersedia di repo
- generated type `src/types/database.ts` belum tersedia
- semua perintah Supabase di dokumen ini masih berstatus target baseline, bukan bukti bahwa bootstrap sudah selesai

---

## 7. Script Package Minimum

Script minimum yang diharapkan:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint src",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "e2e": "playwright test"
  }
}
```

Jika sebagian script belum ada, itu berarti repo belum mencapai baseline siap-kembang penuh.

Status repo saat dokumen ini ditulis:

- tersedia: `dev`, `build`, `start`, `lint`, `typecheck`
- belum tersedia: `test`, `test:watch`, `e2e`
- script `lint` sudah dinormalisasi ke `eslint src` yang kompatibel dengan baseline Next.js 16 saat ini

Tool test yang dikunci:

- unit/integration test: `Vitest`
- E2E test: `Playwright`

---

## 8. Quality Gate Lokal

Sebelum perubahan dianggap siap review:

```bash
pnpm run lint
pnpm run typecheck
pnpm run build
```

Status baseline repo saat dokumen ini diperbarui:

- `pnpm run lint` lolos
- `pnpm run typecheck` lolos
- `pnpm run build` lolos dengan warning non-blocking dari route demo dan MUI legacy component

Untuk unit/integration test:

```bash
pnpm run test
```

Untuk fitur UI penting:

```bash
pnpm run e2e
```

Untuk perubahan database:

```bash
pnpm exec supabase db reset
pnpm exec supabase gen types typescript --local > src/types/database.ts
```

---

## 9. Seed Data Minimum

Seed lokal harus mampu mewakili skenario berikut:

- 1 admin
- reseller `PENDING`, `AKTIF`, `NONAKTIF`
- periode `PERSIAPAN`, `AKTIF`, `SELESAI`
- paket komposit dan paket tunggal
- barang dengan stok cukup dan stok kurang
- konsumen tanpa pesanan
- pesanan konsumen belum final
- pesanan konsumen sudah final
- pesanan perlu perhatian
- konsumen belum lunas
- konsumen lunas
- item pesanan `AKTIF`, `TERHENTI`, `BATAL`
- setoran konsumen sebagian dan lunas
- setoran pusat sebagian dan lunas
- pencairan tabungan dan komisi
- kas masuk dari luar setoran reseller
- koreksi setoran
- pembagian paket ke reseller
- audit log untuk penyesuaian atau koreksi transaksi

---

## 10. Deployment Environment

Standar minimum environment:

- local untuk development
- staging untuk verifikasi integrasi
- production untuk go-live

Aturan:

- staging wajib ada sebelum production digunakan serius
- migration tidak boleh langsung naik ke production tanpa lolos local atau staging
- konfigurasi environment harus terpisah per target

---

## 11. Prinsip Kerja

- Browser hanya mengonsumsi data yang aman melalui RLS dan RPC.
- Logika bisnis final tetap berada di database atau server boundary.
- Shell Modernize dipertahankan untuk theme, layout, mode horizontal, dark mode, dan customizer.
- Dokumen ini menjadi acuan utama environment sampai ada keputusan resmi baru.

Dokumen terkait:

- `docs/setup/setup_development_environment.md`
- `docs/quality/testing_strategy.md`
- `docs/quality/checklist.md`
