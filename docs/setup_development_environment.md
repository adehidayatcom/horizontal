# Setup Development Environment
## Paket Lebaran Mumpuni

Panduan ini menjelaskan langkah setup lokal untuk repo `horizontal` dengan baseline `Next.js + Modernize + Supabase`.

Dokumen ini menjelaskan target setup yang ingin dicapai, lalu menandai bagian yang saat ini masih berupa rencana bootstrap.

---

## 1. Prasyarat

Pastikan tool berikut tersedia:

- Node.js `20+`, direkomendasikan `22 LTS`
- pnpm `10.x`
- Git
- Docker Desktop
- Supabase CLI

Cek cepat:

```bash
node --version
pnpm --version
git --version
pnpm exec supabase --version
```

---

## 2. Clone dan Masuk ke Repo

```bash
git clone <repo-url> horizontal
cd horizontal
```

Jika repo sudah ada secara lokal, cukup masuk ke folder kerjanya.

---

## 3. Install Dependency

```bash
pnpm install
```

Hasil yang diharapkan:

- dependency terpasang tanpa error fatal
- folder `node_modules/` terbentuk

---

## 4. Siapkan Environment Variables

Buat file:

```txt
.env.local
```

Isi minimal:

```env
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_APP_ENV=local
```

Untuk Supabase cloud:

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_APP_ENV=local
```

Aturan penting:

- gunakan `NEXT_PUBLIC_*` untuk value yang dibaca browser
- jangan letakkan `SERVICE_ROLE_KEY` di file yang dibaca client

---

## 5. Siapkan Supabase

### Opsi A: Supabase Local

Inisialisasi jika folder `supabase/` belum ada:

```bash
pnpm exec supabase init
```

Jalankan local stack:

```bash
pnpm exec supabase start
```

Ambil nilai `API URL` dan `anon key`, lalu masukkan ke `.env.local`.

### Opsi B: Supabase Cloud

1. Buat project di Supabase
2. Ambil `Project URL`
3. Ambil `anon key`
4. Masukkan ke `.env.local`

Catatan status repo saat dokumen ini ditulis:

- folder `supabase/` belum tersedia
- langkah Supabase pada dokumen ini masih bersifat target bootstrap

---

## 6. Generate Database Types

Untuk local:

```bash
pnpm exec supabase gen types typescript --local > src/types/database.ts
```

Untuk cloud:

```bash
pnpm exec supabase gen types typescript --project-id your-project-id > src/types/database.ts
```

File `src/types/database.ts` dianggap generated file dan tidak diedit manual.

Catatan status repo saat dokumen ini ditulis:

- file `src/types/database.ts` belum tersedia
- generation type belum bisa dianggap bagian dari setup harian sampai bootstrap Supabase selesai

---

## 7. Struktur Minimum yang Diharapkan

Setelah setup dasar, proyek diharapkan memiliki area berikut:

```txt
src/
  app/
  utils/
  types/
  lib/
supabase/
  migrations/
```

Catatan:

- `app/` dan `utils/` adalah basis shell Modernize + Next.js
- `lib/` dipakai untuk integrasi Supabase, query, validasi, dan utilitas tambahan
- `supabase/` menyimpan migration dan seed

Catatan:

- struktur di atas adalah target arsitektur, bukan cerminan penuh repo saat ini
- repo saat ini masih membawa banyak modul demo template yang belum dipangkas

---

## 8. Jalankan Development Server

```bash
pnpm run dev
```

Hasil yang diharapkan:

```txt
Next.js ready on http://localhost:3000
```

Lalu buka:

```txt
http://localhost:3000
```

---

## 9. Verifikasi Awal

Checklist minimum:

- dependency sukses terpasang
- `.env.local` tersedia
- `NEXT_PUBLIC_SUPABASE_URL` dan `NEXT_PUBLIC_SUPABASE_ANON_KEY` terisi
- dev server bisa menyala
- halaman utama bisa dibuka
- tidak ada error fatal di terminal saat boot

Jika Supabase sudah aktif:

- types berhasil di-generate
- migration bisa dijalankan
- koneksi ke Supabase berhasil

---

## 10. Quality Gate Dasar

Jalankan:

```bash
pnpm run lint
pnpm run build
```

Jika script sudah tersedia, lanjutkan:

```bash
pnpm run typecheck
pnpm run test
```

Jika proyek sudah punya E2E:

```bash
pnpm run e2e
```

Status repo saat dokumen ini ditulis:

- `lint`, `build`, dan `typecheck` tersedia
- `test` dan `e2e` belum tersedia
- strategi penambahannya dijelaskan di `docs/testing_strategy.md`

---

## 11. Troubleshooting Umum

### `NEXT_PUBLIC_SUPABASE_URL is required`

Periksa `.env.local` dan restart dev server.

### Supabase local gagal start

Pastikan Docker Desktop sedang aktif, lalu jalankan ulang:

```bash
pnpm exec supabase stop
pnpm exec supabase start
```

### Types tidak tergenerate

Pastikan Supabase local aktif atau project cloud Supabase dapat diakses, lalu ulangi command generate types.

### Build gagal karena struktur belum lengkap

Itu menandakan setup dependency berhasil, tetapi baseline folder, env, atau implementasi proyek belum lengkap.

---

## 12. Langkah Setelah Setup

Urutan kerja yang direkomendasikan:

1. verifikasi shell Modernize dan theme berjalan baik
2. siapkan Supabase schema dan generated types
3. pasang layer `lib/` untuk client, query, dan validasi
4. bangun route admin dan reseller di atas shell yang sudah ada
5. aktifkan quality gate sebelum pekerjaan fitur besar dimulai
