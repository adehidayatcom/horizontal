# Execution Module Auth
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `auth`.

Dokumen ini memecah implementasi autentikasi dan otorisasi ke dua area yang terkoordinasi:

- frontend integration surface
- backend/access control logic

Dokumen ini mengacu pada:

- `docs/product/prd.md`
- `docs/contracts/schema_mapping.md`
- `docs/contracts/rls_matrix.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/frontend/frontend_architecture.md`
- `docs/execution/backend_plan.md`
- `docs/quality/implementation_guardrails.md`
- `docs/frontend/navigation_and_period_setup_ui.md`

---

## 1. Tujuan Modul

Modul `auth` bertanggung jawab untuk:

- menyediakan login yang aman untuk admin dan reseller
- menghubungkan session Supabase Auth ke `profile`
- memetakan role `ADMIN` dan `RESELLER`
- memetakan reseller login ke `profile.no_reseller`
- memproteksi route admin dan reseller
- memblok reseller `PENDING` dari area transaksi

Modul ini adalah fondasi akses sistem. Ia harus selesai secara kontrak sebelum implementasi transaksi besar bergantung pada role dan session.

---

## 2. Scope Modul

### Frontend Integration Scope

- halaman login
- guard layout admin dan reseller
- redirect berdasarkan role
- state `unauthorized`, `session expired`, `pending approval`
- logout flow

### Backend/Core Scope

- sinkronisasi `auth.users` dengan tabel `profile`
- helper auth server
- middleware atau server guard untuk route admin/reseller
- validasi role dan `no_reseller`
- policy bootstrap sesuai `rls_matrix.md`

---

## 3. Integration Contract Modul Auth

## 3.1 Session Shape Minimum

```ts
type AuthSessionSummary = {
  user_id: string;
  role: 'ADMIN' | 'RESELLER';
  no_reseller?: string | null;
  nama_tampil?: string | null;
  reseller_status?: 'PENDING' | 'AKTIF' | 'NONAKTIF' | null;
};
```

## 3.2 Route Access Rules

```txt
/admin/*    -> ADMIN only
/reseller/* -> RESELLER only
/login      -> public
```

## 3.3 Error Codes Relevan

```txt
AUTH_UNAUTHORIZED
AUTH_SESSION_EXPIRED
AUTH_PROFILE_NOT_FOUND
AUTH_ROLE_INVALID
AUTH_RESELLER_PENDING
ACCESS_FORBIDDEN
```

---

## 4. Struktur Folder Modul Auth

## 4.1 Frontend

```txt
src/app/
  login/
    page.tsx

src/app/components/auth/
  LoginScreen.tsx
  LoginForm.tsx
  PendingApprovalState.tsx
  SessionExpiredState.tsx
  UnauthorizedState.tsx
  index.ts

src/app/context/
  AuthContext.tsx

src/hooks/shared/
  useAuthSession.ts
  useLogout.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  002_rls_policies.sql
  006_functions_reseller.sql

src/lib/supabase/
  client.ts
  server.ts
  admin.ts

src/lib/server/auth/
  auth-session.ts
  auth-guards.ts
  auth-redirects.ts

src/middleware.ts

src/app/api/auth/
  session/route.ts
  logout/route.ts
```

---

## 5. Task Execution Batches

### Batch A1 - Identity Foundation

- MAU-01
- MAU-02
- MAU-03

### Batch A2 - Route Protection

- MAU-04
- MAU-05
- MAU-06A

### Batch A3 - UI and Session Feedback

- MAU-06B
- MAU-07
- MAU-08
- MAU-09

---

## 6. Hubungan Antar File

- `LoginForm.tsx` mengirim kredensial ke Supabase Auth client lalu menyerahkan hasil session ke `AuthContext.tsx`
- `AuthContext.tsx` dan `useAuthSession.ts` membaca `session/route.ts` untuk mendapatkan `AuthSessionSummary`
- `auth-session.ts` menjadi helper server tunggal untuk membaca session dan `profile`
- `auth-guards.ts` memakai output `auth-session.ts` untuk memproteksi area admin dan reseller
- `auth-redirects.ts` menentukan redirect berdasarkan role, status reseller, dan ketersediaan session
- `useLogout.ts` memanggil `logout/route.ts` lalu membersihkan state client auth
- `SessionExpiredState.tsx` dan `UnauthorizedState.tsx` hanya menampilkan feedback UI, tidak mengambil keputusan akses final

---

## 7. Breakdown Component and Function

| File | Tanggung jawab |
|---|---|
| `LoginScreen.tsx` | shell halaman login |
| `LoginForm.tsx` | form email/password, error UI, submit state |
| `PendingApprovalState.tsx` | state reseller `PENDING` |
| `SessionExpiredState.tsx` | state session habis dan CTA login ulang |
| `UnauthorizedState.tsx` | state akses ditolak |
| `AuthContext.tsx` | provider session summary frontend |
| `useAuthSession.ts` | hook baca session summary |
| `useLogout.ts` | hook logout frontend |
| `auth-session.ts` | helper server untuk gabung session + profile |
| `auth-guards.ts` | guard role dan `no_reseller` |
| `auth-redirects.ts` | aturan redirect auth-aware |
| `session/route.ts` | route baca session summary |
| `logout/route.ts` | route logout |
| `middleware.ts` | proteksi awal request untuk route utama |

---

## 8. Task Implementation Detail

## MAU-01 - Finalkan contract `profile`

- tujuan: mengunci hubungan `auth.users` dan `profile`
- file yang dibuat/diubah: `001_init_schema.sql`, `schema_mapping.md` bila perlu
- lokasi file: `supabase/migrations/001_init_schema.sql`
- langkah kerja AI agent:
  - pastikan tabel `profile` punya `id`, `role`, `no_reseller`, `nama_tampil`
  - pastikan role hanya `ADMIN` atau `RESELLER`
  - pastikan reseller profile boleh null `no_reseller` hanya jika belum linked secara sah
- dependency: `schema_mapping.md`, `rls_matrix.md`
- output yang diharapkan: kontrak profile stabil
- acceptance criteria:
  - role dan linkage ke reseller jelas
  - profile bisa dipakai sebagai sumber access model

## MAU-02 - Finalkan helper auth server

- tujuan: menyediakan helper tunggal untuk membaca session dan profile
- file yang dibuat/diubah: `src/lib/supabase/server.ts`, `src/lib/server/auth/auth-session.ts`
- lokasi file: `src/lib/supabase/server.ts`, `src/lib/server/auth/auth-session.ts`
- langkah kerja AI agent:
  - buat helper baca user dari Supabase session
  - gabungkan dengan `profile`
  - normalisasi ke `AuthSessionSummary`
- dependency: MAU-01
- output yang diharapkan: helper session stabil
- acceptance criteria:
  - admin dan reseller menghasilkan shape yang sama
  - missing profile menghasilkan error yang jelas

## MAU-03 - Sinkronkan RLS bootstrap

- tujuan: memastikan session dan RLS berbicara model akses yang sama
- file yang dibuat/diubah: `002_rls_policies.sql`, helper SQL auth
- lokasi file: `supabase/migrations/002_rls_policies.sql`
- langkah kerja AI agent:
  - buat helper `is_admin()` dan `auth_no_reseller()`
  - sinkronkan dengan `rls_matrix.md`
- dependency: MAU-01
- output yang diharapkan: policy bootstrap auth stabil
- acceptance criteria:
  - reseller tidak bisa melihat data reseller lain
  - admin bisa melihat data lintas reseller

## MAU-04 - Finalkan route protection

- tujuan: mencegah akses lintas role sejak awal request
- file yang dibuat/diubah: `src/middleware.ts`, `auth-guards.ts`, `auth-redirects.ts`
- lokasi file: `src/middleware.ts`, `src/lib/server/auth/...`
- langkah kerja AI agent:
  - proteksi `/admin/*`
  - proteksi `/reseller/*`
  - redirect user yang salah role
- dependency: MAU-02
- output yang diharapkan: route guard stabil
- acceptance criteria:
  - reseller tidak bisa masuk area admin
  - admin tidak salah dilempar ke area reseller

## MAU-05 - Finalkan pending approval flow

- tujuan: membedakan reseller `PENDING` dari reseller `AKTIF`
- file yang dibuat/diubah: `auth-guards.ts`, `reseller/page.tsx`, state screen auth
- lokasi file: `src/lib/server/auth/auth-guards.ts`, `src/app/components/auth/...`
- langkah kerja AI agent:
  - baca `reseller.status`
  - render state `Pendaftaran sedang ditinjau`
  - blok route transaksi reseller
- dependency: MAU-02
- output yang diharapkan: pending flow jelas
- acceptance criteria:
  - reseller `PENDING` tidak bisa akses transaksi
  - feedback UI jelas

## MAU-06A - Finalkan logout route

- tujuan: menyediakan jalur backend logout yang jelas dan aman
- file yang dibuat/diubah: `logout/route.ts`
- lokasi file: `src/app/api/auth/logout/route.ts`
- langkah kerja AI agent:
  - buat logout route
  - bersihkan session/cookie sesuai auth boundary yang dipakai
  - kembalikan response logout yang konsisten
- dependency: MAU-02
- output yang diharapkan: logout route stabil
- acceptance criteria:
  - logout tidak meninggalkan session aktif
  - response logout konsisten untuk consumer frontend

## MAU-06B - Finalkan frontend logout/session expired behavior

- tujuan: menghindari state auth setengah rusak di browser
- file yang dibuat/diubah: `useLogout.ts`, `SessionExpiredState.tsx`
- lokasi file: `src/hooks/shared/useLogout.ts`, `src/app/components/auth/SessionExpiredState.tsx`
- langkah kerja AI agent:
  - buat handler logout frontend
  - buat handler session expired
  - buat UX redirect ke login
- dependency: MAU-06A
- output yang diharapkan: auth exit flow stabil
- acceptance criteria:
  - session expired tidak meninggalkan UI rusak
  - logout frontend memakai route internal yang sudah tersedia

## MAU-07 - Finalkan halaman login

- tujuan: menyiapkan layar masuk yang sesuai role model proyek
- file yang dibuat/diubah: `src/app/login/page.tsx`, `LoginScreen.tsx`, `LoginForm.tsx`
- lokasi file: `src/app/login/page.tsx`, `src/app/components/auth/...`
- langkah kerja AI agent:
  - siapkan input email/password
  - tampilkan error Bahasa Indonesia
  - redirect sesudah login berdasarkan role
- dependency: MAU-04
- output yang diharapkan: login UI siap
- acceptance criteria:
  - login tidak mencampur logic transaksi
  - role redirect jelas

## MAU-08 - Finalkan AuthContext dan hook session

- tujuan: memberi frontend satu sumber baca session
- file yang dibuat/diubah: `AuthContext.tsx`, `useAuthSession.ts`
- lokasi file: `src/app/context/AuthContext.tsx`, `src/hooks/shared/useAuthSession.ts`
- langkah kerja AI agent:
  - sediakan session summary
  - sediakan loading/error state
  - hindari duplikasi fetch profile
- dependency: MAU-02
- output yang diharapkan: auth context stabil
- acceptance criteria:
  - komponen cukup membaca context/hook
  - role tidak ditebak dari route

## MAU-09 - Verifikasi auth + role scenarios

- tujuan: memastikan auth cukup aman sebelum modul transaksi dibangun
- file yang dibuat/diubah: dokumen test/checklist atau spec yang tersedia
- lokasi file: mengikuti tool test yang dipilih di `docs/quality/testing_strategy.md`
- langkah kerja AI agent:
  - uji admin access
  - uji reseller access
  - uji pending reseller
  - uji no session
- dependency: semua task auth
- output yang diharapkan: matrix scenario auth tervalidasi
- acceptance criteria:
  - semua state utama punya perilaku jelas

---

## 9. Pseudocode / Flow Penting

## 9.1 Flow `read_auth_session_summary`

```txt
read session from Supabase Auth
if no session:
  return AUTH_UNAUTHORIZED

load profile by user_id
if profile not found:
  return AUTH_PROFILE_NOT_FOUND

if profile.role not in [ADMIN, RESELLER]:
  return AUTH_ROLE_INVALID

build AuthSessionSummary
return summary
```

## 9.2 Flow `guard_dashboard_route`

```txt
read AuthSessionSummary
if error AUTH_UNAUTHORIZED:
  redirect /login

if target route starts with /admin and role != ADMIN:
  return ACCESS_FORBIDDEN

if target route starts with /reseller and role != RESELLER:
  return ACCESS_FORBIDDEN

if role == RESELLER and reseller_status == PENDING:
  render pending approval state

allow request
```

## 9.3 Flow `logout_and_session_expired`

```txt
frontend trigger logout or detect session expired
call /api/auth/logout
clear local auth state
redirect /login
render session expired feedback if trigger was expiration
```

---

## 10. First Working Slice

Slice pertama modul auth harus menutup jalur akses minimum tanpa membuka ambiguity:

1. user bisa login dari satu halaman `login`
2. server bisa membaca `AuthSessionSummary` dari session + `profile`
3. route `/admin/*` hanya bisa diakses `ADMIN`
4. route `/reseller/*` hanya bisa diakses `RESELLER`
5. reseller `PENDING` melihat state `Pendaftaran sedang ditinjau`
6. logout dan session expired kembali ke login tanpa meninggalkan UI setengah rusak

Slice ini cukup untuk membuka modul role-aware berikutnya tanpa menunggu semua polishing auth selesai.

---

## 11. Rule Khusus AI Coding Agent

- jangan menaruh keputusan role final di komponen frontend
- jangan menebak role dari path; selalu baca dari `AuthSessionSummary`
- jangan menggabungkan pending approval dengan unauthorized state
- jangan menambah provider auth baru di luar Supabase Auth
- jangan membuat registrasi publik di task auth kecuali menjadi dependency resmi

---

## 12. Urutan Eksekusi Modul

1. selesaikan kontrak `profile` dan helper session (`MAU-01`, `MAU-02`)
2. sinkronkan bootstrap RLS dan guard route (`MAU-03`, `MAU-04`)
3. kunci perilaku reseller `PENDING` dan logout route (`MAU-05`, `MAU-06A`)
4. sambungkan UX logout/session expired dan halaman login (`MAU-06B`, `MAU-07`)
5. finalkan `AuthContext` dan verifikasi skenario utama (`MAU-08`, `MAU-09`)

---

## 13. Risiko, Asumsi, dan Keputusan Terkunci

## Risiko

- auth setengah jadi akan merusak seluruh modul role-aware
- pending approval rawan bocor jika hanya di-guard UI

## Asumsi

- provider auth utama adalah Supabase Auth
- login email/password cukup untuk fase awal

## Catatan Keputusan Terkunci

- registrasi reseller fase awal memakai dua jalur: form publik `self-register` ke status `PENDING`, dan create oleh admin sesuai `DL-023` / `docs/truth/01-decision_log.md`
- halaman login dipakai bersama untuk admin dan reseller; pembedaan akses dilakukan setelah autentikasi melalui role dan redirect sesuai `DL-040` / `docs/truth/01-decision_log.md`
