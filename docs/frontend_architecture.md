# Frontend Architecture
## Paket Lebaran Mumpuni

Dokumen ini adalah kontrak implementasi frontend untuk repo `horizontal`.

Tujuannya:

- menjaga konsistensi antara shell Modernize dan kebutuhan bisnis proyek
- memberi pedoman yang jelas untuk route, layout, komponen, hook, dan integrasi data
- memastikan logika bisnis final tidak bocor ke browser

---

## 1. Baseline Arsitektur

Frontend proyek ini dibangun dengan stack berikut:

- `Next.js 16` dengan `App Router`
- `React 19`
- `Material-UI v7`
- `Emotion`
- `SWR`
- `Formik` + `Yup`
- `TypeScript`
- `Supabase` untuk auth, data access, RPC, dan RLS

Shell visual proyek berasal dari template `Modernize`, khususnya untuk:

- provider theme
- customizer
- layout horizontal dan vertical
- header dan sidebar pattern
- dark mode dan layout width mode

---

## 2. Prinsip Utama

- Frontend adalah lapisan presentasi, interaksi, dan navigasi.
- Kebenaran bisnis berada di database dan server boundary.
- Semua transaksi penting harus melalui `Supabase RPC` atau server action yang aman.
- Perhitungan final seperti stok, saldo, target, status lunas, dan komisi tidak boleh menjadi sumber kebenaran di browser.
- Reseller hanya boleh melihat data miliknya sendiri.
- Setiap query operasional harus jelas konteks periodenya.
- Shell Modernize tidak dibuang; ia dijadikan fondasi UI resmi proyek.

---

## 3. Lapisan Frontend

Frontend dibagi menjadi lima lapisan:

### A. App Routing Layer

Berisi route, layout, loading, dan error boundary berbasis `App Router`.

### B. Shell Layer

Berisi theme provider, customizer, header, navigation, sidebar, breadcrumb, dan pengaturan layout global dari Modernize.

### C. Feature UI Layer

Berisi halaman dan komponen bisnis untuk admin dan reseller.

### D. Data Access Layer

Berisi client Supabase, server helpers, query keys, hooks SWR, dan action wrapper.

Keputusan akses data frontend nyata:

- semua read dan write operasional admin/reseller melewati internal API route
- browser Supabase client tidak dipakai langsung untuk view/RPC bisnis proyek
- browser Supabase client hanya boleh dipakai untuk auth/session bootstrap atau helper shell non-bisnis yang sudah disetujui

### E. Domain Contract Layer

Berisi type, schema validasi, dan kontrak data yang merujuk pada dokumen bisnis dan schema database.

---

## 4. Struktur Folder Target

Struktur berikut adalah target resmi untuk frontend proyek:

```txt
src/
  app/
    (DashboardLayout)/
      admin/
        dashboard/page.tsx
        periode/page.tsx
        reseller/page.tsx
        reseller-periode/page.tsx
        konsumen/page.tsx
        master/
          akun-kas/page.tsx
          barang/page.tsx
          barang-periode/page.tsx
          komisi/page.tsx
          paket/page.tsx
        pesanan/
          page.tsx
          perlu-perhatian/
            page.tsx
          [id]/
            page.tsx
        setoran-konsumen/page.tsx
        setoran-pusat/page.tsx
        gudang/
          belanja/page.tsx
          packing/page.tsx
          pembagian/page.tsx
          pengiriman/page.tsx
        keuangan/
          kas-masuk/page.tsx
          mutasi-kas/page.tsx
          pencairan/page.tsx
        laporan/
          rekap-reseller/page.tsx
          stok/page.tsx
          audit/page.tsx
          pembagian/page.tsx
          keuangan/page.tsx
        koreksi/page.tsx
      reseller/
        page.tsx
        konsumen/page.tsx
        pesanan/
          page.tsx
          create/
            page.tsx
          [id]/
            page.tsx
        setor/page.tsx
        akun/page.tsx
      layout.tsx
    api/
    layout.tsx
    loading.tsx
    not-found.tsx

  app/components/
    admin/
    reseller/
    shared/
    ui-components/
    tables/

  app/context/
    customizerContext.tsx
    AuthContext.tsx
    PeriodContext.tsx

  lib/
    supabase/
      client.ts
      server.ts
    query/
      keys.ts
      fetchers.ts
      errors.ts
    validasi/
      schemas.ts
    services/
      admin/
      reseller/
      shared/

  hooks/
    admin/
    reseller/
    shared/
    mutations/

  types/
    database.ts
    models.ts
    api.ts

  utils/
    theme/
    i18n.ts
    markdown.ts
    format.ts
```

Catatan:

- folder existing dari shell Modernize dipertahankan selama masih relevan
- folder domain bisnis baru ditambahkan secara bertahap dan terstruktur
- nama folder bisnis harus mencerminkan domain proyek, bukan demo template
- alias folder frontend `program/` dan `order/` tidak dipakai pada target final; semua route dan target file resmi memakai `pesanan/`

---

## 5. Routing

Routing menggunakan `Next.js App Router`.

Pola route utama:

```txt
/                              -> landing/redirect
/login                         -> autentikasi
/admin                         -> redirect ke dashboard admin
/admin/dashboard               -> dashboard admin
/admin/periode                 -> kelola periode
/admin/reseller                -> daftar reseller
/admin/reseller-periode        -> reseller per periode
/admin/konsumen                -> daftar konsumen admin
/admin/master/akun-kas         -> master akun kas
/admin/master/barang           -> master barang
/admin/master/barang-periode   -> barang per periode
/admin/master/komisi           -> komisi per periode
/admin/master/paket            -> master paket dan BOM
/admin/pesanan                -> pesanan konsumen
/admin/setoran-konsumen        -> monitoring setoran konsumen
/admin/setoran-pusat           -> monitoring setoran pusat
/admin/gudang/belanja          -> belanja
/admin/gudang/packing          -> packing
/admin/gudang/pembagian        -> pembagian paket
/admin/gudang/pengiriman       -> pengiriman
/admin/keuangan/kas-masuk      -> kas masuk
/admin/keuangan/mutasi-kas     -> mutasi kas
/admin/keuangan/pencairan      -> pencairan
/admin/laporan/rekap-reseller  -> laporan rekap reseller
/admin/laporan/stok            -> laporan stok
/admin/laporan/audit           -> audit log dan riwayat koreksi admin
/reseller                      -> dashboard reseller
/reseller/konsumen             -> konsumen reseller
/reseller/pesanan             -> pesanan reseller
/reseller/setor                -> setoran reseller
/reseller/akun                 -> akun reseller
```

Aturan:

- admin dan reseller harus dipisah jelas pada route level
- route tidak boleh mengikuti nama demo template yang tidak relevan
- layout shell harus reusable, tetapi menu dan akses harus role-aware
- route final harus mengikuti `docs/navigation_and_period_setup_ui.md` bila ada konflik dengan dokumen lain
- route audit admin first wave mengikuti `docs/support/decision_log.md` dan memakai `/admin/laporan/audit`

---

## 6. Layout dan Shell

### Root Layout

`src/app/layout.tsx` bertanggung jawab untuk:

- metadata global
- theme bootstrap
- provider global
- top loader jika dipakai

### Dashboard Layout

`src/app/(DashboardLayout)/layout.tsx` bertanggung jawab untuk:

- header
- sidebar atau horizontal navigation
- container content
- breadcrumb
- slot halaman admin dan reseller

### Shell Behavior

Shell Modernize menjadi fondasi perilaku UI berikut:

- mode horizontal navigation
- mode sidebar navigation
- dark mode
- fluid vs contained layout
- responsive collapse behavior

Aturan shell:

- perubahan shell harus dilakukan sebagai capability global, bukan hardcode per halaman
- halaman bisnis harus mengikuti shell, bukan membuat shell alternatif baru tanpa alasan kuat

---

## 7. Role dan Access Model

Frontend mengenal minimal dua role:

- `ADMIN`
- `RESELLER`

Informasi auth dan role dibagikan lewat `AuthContext`.

Informasi periode aktif dibagikan lewat `PeriodContext`.

Aturan:

- reseller tidak boleh mengontrol identifier akses sensitif dari browser jika bisa diambil dari session/profile
- redirect dan proteksi utama dilakukan di middleware atau server boundary
- guard tambahan di komponen hanya sebagai lapisan UX, bukan lapisan keamanan utama

---

## 8. Data Flow

Pola data flow utama:

```txt
Page
-> Feature component
-> Hook SWR / action
-> Service wrapper
-> Supabase client
-> RPC / view / table
```

Prioritas akses data:

1. `RPC` untuk transaksi dan agregasi bisnis
2. `View` atau query terstruktur untuk read model kompleks
3. `Direct table read` hanya untuk master sederhana yang aman oleh RLS

Mutasi bisnis:

- create/update/delete yang berdampak ke uang, stok, status, atau audit harus lewat RPC atau server action aman

---

## 9. SWR dan Query Keys

Semua fetching frontend menggunakan `SWR`.

Setiap domain wajib punya query key factory yang konsisten, misalnya:

```ts
export const periodeKeys = {
  all: () => ['periode'],
  active: () => ['periode', 'active'],
  byId: (id: number) => ['periode', id],
};
```

Aturan:

- jangan hardcode key acak di komponen
- invalidation harus mengikuti domain key
- fetcher umum dan error mapping diletakkan di `lib/query/`

---

## 10. Forms dan Validasi

Form menggunakan:

- `Formik` untuk state form
- `Yup` untuk validasi client-side

Aturan:

- validasi frontend hanya untuk UX awal
- validasi final tetap di RPC/server boundary
- pesan validasi harus jelas dan konsisten dalam Bahasa Indonesia
- form admin dan reseller boleh berbeda pola interaksinya sesuai konteks perangkat

---

## 11. Component Strategy

Komponen dibagi menjadi:

- `app/components/admin/` untuk UI khusus admin
- `app/components/reseller/` untuk UI khusus reseller
- `app/components/shared/` untuk komponen lintas role
- `app/components/ui-components/` untuk wrapper atau adaptasi komponen MUI

Aturan:

- jangan campur komponen admin dan reseller dalam satu folder fitur tanpa alasan kuat
- presentational component dipisah dari data access jika ukurannya mulai besar
- reusable component harus generik dan tidak diam-diam membawa aturan domain spesifik

---

## 12. Responsive Rules

Prinsip responsive proyek:

- admin: desktop-first
- reseller: mobile-first

Aturan penting:

- halaman reseller tidak boleh memaksa horizontal scroll pada viewport mobile normal
- tabel admin boleh memakai horizontal overflow jika memang diperlukan
- elemen aksi utama reseller harus bisa dijangkau cepat dengan interaksi minimum

---

## 13. Theme Rules

Theme mengikuti shell Modernize yang disesuaikan untuk kebutuhan proyek.

Yang harus tetap konsisten:

- design token warna
- typography
- spacing
- elevation
- dark mode behavior
- horizontal vs sidebar navigation behavior
- fluid/container switching

Aturan:

- jangan membuat theme alternatif per halaman
- kustomisasi theme harus masuk ke lapisan theme/provider, bukan inline style yang tersebar

---

## 14. Integrasi Supabase

Frontend membutuhkan minimal:

- browser client
- server-side helper
- generated database types

Lokasi target:

```txt
src/lib/supabase/client.ts
src/lib/supabase/server.ts
src/types/database.ts
```

Aturan:

- kredensial client memakai `NEXT_PUBLIC_SUPABASE_URL` dan `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- service role key tidak boleh bocor ke browser
- semua type database yang generated tidak diedit manual

---

## 15. Dokumen Rujukan

Dokumen ini dipakai bersama:

- `docs/prd.md`
- `docs/business_contracts.md`
- `docs/query_contracts.md`
- `docs/schema_mapping.md`
- `docs/rls_matrix.md`
- `docs/frontend_component_contracts.md`
- `docs/component_patterns.md`
- `docs/navigation_and_period_setup_ui.md`
- `docs/ui/admin_dashboard_uiux.md`
- `docs/ui/reseller_uiux.md`
- `docs/ui/modernize_shell_guide.md`
- `docs/definition_of_done.md`

Jika ada konflik, urutan prioritasnya:

1. kontrak bisnis dan schema
2. dokumen arsitektur frontend ini
3. kontrak komponen dan UI
4. dokumen rencana eksekusi

---

## 16. Definition of Good Frontend

Frontend dianggap berada di jalur yang benar jika:

- shell Modernize berhasil dipakai sebagai fondasi resmi
- route admin dan reseller bersih dan konsisten
- theme global mengontrol dark mode, layout width, dan navigation mode
- data access terpusat dan aman
- validasi frontend membantu UX, bukan menggantikan business rules
- halaman bisnis tidak lagi bergantung pada mock demo template yang tidak relevan
