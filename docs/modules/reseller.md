# Execution Module Reseller
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `reseller`.

Dokumen ini memecah implementasi modul reseller ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/product/prd.md`
- `docs/truth/01-decision_log.md`
- `docs/contracts/business_contracts.md`
- `docs/contracts/schema_mapping.md`
- `docs/contracts/query_contracts.md`
- `docs/contracts/rls_matrix.md`
- `docs/truth/system_maps.md`
- `docs/execution/task_kit_standard.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`
- `docs/ui/reseller_uiux.md`

---

## 1. Tujuan Modul

Modul `reseller` bertanggung jawab untuk:

- mengelola identitas reseller
- mengelola status reseller `PENDING`, `AKTIF`, `NONAKTIF`
- menyiapkan keterikatan reseller ke periode melalui `reseller_periode`
- menyediakan area kerja reseller yang aman berbasis `profile.no_reseller`
- mengelola data konsumen milik reseller
- menyediakan read model ringkasan reseller untuk admin dan reseller

Modul ini menjadi jembatan antara fondasi `periode` dan rantai operasional `program -> order -> setoran`.

---

## 2. Scope Modul

### Frontend Mockup Scope

- admin list reseller
- admin detail reseller
- admin approval / status action reseller
- admin create/edit reseller mock
- reseller mobile shell state `PENDING` vs `AKTIF`
- reseller list konsumen
- reseller detail konsumen sederhana
- reseller create/edit konsumen mock

### Backend/Core Scope

- schema `reseller`, `profile`, `reseller_periode`, `konsumen`
- boundary admin create/update/list/detail reseller
- boundary reseller self profile read/update terbatas
- boundary create/update/list konsumen milik reseller
- validasi scope `no_reseller`
- read model ringkasan reseller dan konsumen
- RLS helper dan policy reseller/konsumen

---

## 3. Integration Contract Modul Reseller

## 3.1 Entity Shape Reseller

```ts
type ResellerRecord = {
  no_reseller: string;
  nama_reseller: string;
  alamat?: string | null;
  telepon?: string | null;
  status: 'PENDING' | 'AKTIF' | 'NONAKTIF';
  created_at?: string;
  updated_at?: string | null;
};
```

## 3.2 Entity Shape Konsumen

```ts
type KonsumenRecord = {
  id: number;
  no_reseller: string;
  nama_konsumen: string;
  telepon?: string | null;
  alamat?: string | null;
  created_at?: string;
  updated_at?: string | null;
};
```

## 3.3 Ringkasan Reseller

```ts
type ResellerSummary = {
  periode_id: number;
  no_reseller: string;
  nama_reseller: string;
  status_reseller: 'PENDING' | 'AKTIF' | 'NONAKTIF';
  jumlah_konsumen: number;
  jumlah_program_aktif?: number;
  jumlah_order?: number;
  total_dikumpulkan?: number;
  total_disetor_pusat?: number;
  saldo_belum_disetor?: number;
  status_lunas_reseller?: 'BELUM' | 'LUNAS';
};
```

## 3.4 Request Contract

```ts
type CreateResellerRequest = {
  no_reseller: string;
  nama_reseller: string;
  alamat?: string | null;
  telepon?: string | null;
};

type UpdateResellerStatusRequest = {
  no_reseller: string;
  status: 'PENDING' | 'AKTIF' | 'NONAKTIF';
};

type CreateKonsumenRequest = {
  nama_konsumen: string;
  telepon?: string | null;
  alamat?: string | null;
};
```

## 3.5 Error Codes Relevan

```txt
RESELLER_NOT_FOUND
RESELLER_ALREADY_EXISTS
RESELLER_STATUS_INVALID
RESELLER_NOT_ACTIVE
RESELLER_ACCESS_DENIED
KONSUMEN_NOT_FOUND
KONSUMEN_ACCESS_DENIED
VALIDATION_REQUIRED_FIELD
ACCESS_FORBIDDEN
```

---

## 4. Struktur Folder Modul Reseller

## 4.1 Frontend

```txt
src/app/(Admin)/admin/reseller/
  page.tsx
  [no_reseller]/
    page.tsx
  create/
    page.tsx

src/app/(Reseller)/reseller/
  page.tsx
  konsumen/
    page.tsx
    [id]/
      page.tsx
    create/
      page.tsx
  akun/
    page.tsx

src/app/components/admin/reseller/
  ResellerListScreen.tsx
  ResellerTable.tsx
  ResellerForm.tsx
  ResellerDetailScreen.tsx
  ResellerStatusActions.tsx
  ResellerSummaryCards.tsx
  index.ts

src/app/components/reseller/konsumen/
  ResellerKonsumenScreen.tsx
  KonsumenCard.tsx
  KonsumenSearchList.tsx
  KonsumenForm.tsx
  KonsumenDetailScreen.tsx
  ResellerPendingState.tsx
  index.ts

src/hooks/admin/
  useResellerListMock.ts
  useResellerDetailMock.ts
  useResellerActionsMock.ts

src/hooks/reseller/
  useKonsumenListMock.ts
  useKonsumenDetailMock.ts
  useKonsumenActionsMock.ts
  useResellerDashboardMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  002_rls_policies.sql
  004_views_ringkasan.sql
  006_functions_reseller.sql

src/lib/server/services/
  reseller.service.ts

src/lib/server/validation/
  reseller.validation.ts

src/app/api/admin/reseller/
  route.ts

src/app/api/reseller/konsumen/
  route.ts

src/app/api/reseller/akun/
  route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `admin/reseller/page.tsx` | Entry route daftar reseller admin |
| `admin/reseller/create/page.tsx` | Entry route create reseller |
| `admin/reseller/[no_reseller]/page.tsx` | Entry route detail reseller |
| `reseller/page.tsx` | Beranda reseller mock |
| `reseller/konsumen/page.tsx` | Daftar konsumen reseller |
| `reseller/konsumen/create/page.tsx` | Form create konsumen mock |
| `reseller/konsumen/[id]/page.tsx` | Detail konsumen |
| `reseller/akun/page.tsx` | Profil reseller, status, shortcut setor pusat |
| `ResellerListScreen.tsx` | Screen utama admin untuk daftar reseller |
| `ResellerTable.tsx` | Table daftar reseller |
| `ResellerForm.tsx` | Form create/edit reseller |
| `ResellerDetailScreen.tsx` | Detail reseller untuk admin |
| `ResellerStatusActions.tsx` | Aksi approve/nonaktifkan reseller |
| `ResellerSummaryCards.tsx` | Ringkasan reseller terpilih |
| `ResellerKonsumenScreen.tsx` | Screen list konsumen di area reseller |
| `KonsumenCard.tsx` | Card konsumen mobile |
| `KonsumenSearchList.tsx` | Search + list konsumen |
| `KonsumenForm.tsx` | Form create/edit konsumen |
| `KonsumenDetailScreen.tsx` | Detail konsumen sederhana |
| `ResellerPendingState.tsx` | State khusus reseller belum approve |
| `useResellerListMock.ts` | Hook mock list reseller |
| `useResellerDetailMock.ts` | Hook mock detail reseller |
| `useResellerActionsMock.ts` | Hook mock create/update status reseller |
| `useKonsumenListMock.ts` | Hook mock list konsumen |
| `useKonsumenDetailMock.ts` | Hook mock detail konsumen |
| `useKonsumenActionsMock.ts` | Hook mock create/edit konsumen |
| `useResellerDashboardMock.ts` | Hook mock ringkasan reseller mobile |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Definisi tabel `profile`, `reseller`, `reseller_periode`, `konsumen` |
| `002_rls_policies.sql` | Helper auth dan policy reseller/konsumen |
| `004_views_ringkasan.sql` | View `v_ringkasan_reseller`, `v_ringkasan_konsumen` minimum |
| `006_functions_reseller.sql` | Boundary reseller dan konsumen |
| `reseller.service.ts` | Wrapper boundary reseller/konsumen |
| `reseller.validation.ts` | Validasi input reseller/konsumen |
| `admin/reseller/route.ts` | Route admin untuk reseller |
| `reseller/konsumen/route.ts` | Route reseller untuk konsumen |
| `reseller/akun/route.ts` | Route profile/ringkasan reseller sendiri |

---

## 6. Hubungan Antar File

### Frontend Flow Admin

```txt
page.tsx
-> ResellerListScreen
-> useResellerListMock
-> dummy reseller data
```

### Frontend Flow Reseller

```txt
reseller route
-> ResellerKonsumenScreen / akun / beranda
-> reseller hooks
-> dummy konsumen dan summary data
```

### Backend Flow

```txt
route.ts
-> reseller.validation.ts
-> reseller.service.ts
-> SQL boundary / RLS-safe query
-> response envelope
```

### Integration Touchpoints

- admin list dan detail reseller memakai shape `ResellerRecord` / `ResellerSummary`
- area reseller mobile memakai `profile.no_reseller` sebagai sumber scope
- route reseller tidak boleh menerima `no_reseller` bebas dari browser/client untuk scope utama

---

## 7. Breakdown Function dan Component per File

## 7.1 `ResellerStatusActions.tsx`

Komponen:

- `ResellerStatusActions`
- `ApproveResellerButton`
- `DeactivateResellerButton`

Tanggung jawab:

- render aksi status admin
- menampilkan dialog konfirmasi
- memblok aksi yang tidak valid untuk status saat ini

## 7.2 `ResellerPendingState.tsx`

Komponen:

- `ResellerPendingState`

Tanggung jawab:

- menampilkan state khusus reseller `PENDING`
- menjelaskan bahwa input operasional belum tersedia
- tetap memberi akses terbatas ke profil dan info approval

## 7.3 `KonsumenForm.tsx`

Komponen:

- `KonsumenForm`

Tanggung jawab:

- render field `nama_konsumen`, `telepon`, `alamat`
- submit create/edit konsumen mock
- menjaga form tetap pendek dan mobile-first

## 7.4 `reseller.service.ts`

Functions:

- `createReseller`
- `updateResellerStatus`
- `getResellerList`
- `getResellerDetail`
- `getOwnResellerSummary`
- `getKonsumenListByActor`
- `getKonsumenDetailByActor`
- `createKonsumenByActor`
- `updateKonsumenByActor`

Tanggung jawab:

- menjadi lapisan tunggal akses modul reseller
- memetakan route admin dan reseller ke boundary yang benar
- memastikan scope actor aman sebelum query/write

---

## 8. Task Implementation Detail

## TASK MR-01

**Nama task:** Define Reseller Module Types and Contracts  
**Tujuan:** Menetapkan type reseller, konsumen, dan summary reseller lintas frontend-backend.  
**File yang dibuat/diubah:**  
- `docs/contracts/integration_contract_pack.md`
- `src/types/models.ts`
- `src/types/api.ts`
**Lokasi file:** docs + types  
**Langkah kerja AI agent:**
1. Pastikan shape reseller, konsumen, dan summary reseller konsisten.
2. Pastikan enum status reseller hanya `PENDING`, `AKTIF`, `NONAKTIF`.
3. Tambahkan type helper untuk screen admin dan mobile reseller.
**Dependency:** `schema_mapping.md`, `query_contracts.md`  
**Output yang diharapkan:** kontrak modul reseller stabil  
**Acceptance criteria:**
- shape admin dan reseller tidak liar
- `no_reseller` tetap jadi identifier bisnis utama

## TASK MR-02

**Nama task:** Build Reseller Mock Data Pack  
**Tujuan:** Menyediakan data mock reseller, profile, reseller summary, dan konsumen.  
**File yang dibuat/diubah:**  
- `src/lib/dummy-data/resellers.ts`
- `src/lib/dummy-data/konsumens.ts`
**Lokasi file:** `src/lib/dummy-data/`  
**Langkah kerja AI agent:**
1. Buat reseller dengan status `PENDING`, `AKTIF`, `NONAKTIF`.
2. Buat konsumen per reseller aktif.
3. Buat ringkasan KPI reseller minimum.
**Dependency:** type reseller dan konsumen  
**Output yang diharapkan:** dataset mock reseller lengkap  
**Acceptance criteria:**
- ada skenario admin list
- ada skenario reseller pending
- ada skenario reseller aktif dengan konsumen

## TASK MR-03

**Nama task:** Build Admin Reseller Mock Hooks and Actions  
**Tujuan:** Menyediakan hook mock untuk list/detail/create/update status reseller.  
**File yang dibuat/diubah:**  
- `src/hooks/admin/useResellerListMock.ts`
- `src/hooks/admin/useResellerDetailMock.ts`
- `src/hooks/admin/useResellerActionsMock.ts`
**Lokasi file:** `src/hooks/admin/`  
**Langkah kerja AI agent:**
1. Pisahkan list, detail, dan action hooks.
2. Simulasikan approve dan nonaktifkan reseller.
3. Simulasikan loading/error/success.
**Dependency:** dummy reseller data  
**Output yang diharapkan:** layer data admin reseller siap  
**Acceptance criteria:**
- screen admin tidak akses dummy data langsung

## TASK MR-04

**Nama task:** Build Admin Reseller List Screen  
**Tujuan:** Menyediakan daftar reseller untuk admin.  
**File yang dibuat/diubah:**  
- `src/app/(Admin)/admin/reseller/page.tsx`
- `src/app/components/admin/reseller/ResellerListScreen.tsx`
- `src/app/components/admin/reseller/ResellerTable.tsx`
**Lokasi file:** route + admin reseller components  
**Langkah kerja AI agent:**
1. Buat page entry.
2. Tampilkan table reseller dengan status badge.
3. Tambahkan search/sort minimum.
**Dependency:** admin reseller hooks  
**Output yang diharapkan:** daftar reseller dapat dipakai admin  
**Acceptance criteria:**
- status terlihat jelas
- navigasi ke detail tersedia

## TASK MR-05

**Nama task:** Build Admin Reseller Form and Detail  
**Tujuan:** Menyediakan create/edit/detail reseller.  
**File yang dibuat/diubah:**  
- `src/app/(Admin)/admin/reseller/create/page.tsx`
- `src/app/(Admin)/admin/reseller/[no_reseller]/page.tsx`
- `src/app/components/admin/reseller/ResellerForm.tsx`
- `src/app/components/admin/reseller/ResellerDetailScreen.tsx`
- `src/app/components/admin/reseller/ResellerStatusActions.tsx`
- `src/app/components/admin/reseller/ResellerSummaryCards.tsx`
**Lokasi file:** admin reseller folders  
**Langkah kerja AI agent:**
1. Buat form reseller.
2. Buat detail reseller dengan summary cards.
3. Tambahkan action approve/nonaktifkan.
**Dependency:** list screen + detail hook  
**Output yang diharapkan:** admin dapat mengelola reseller mock  
**Acceptance criteria:**
- state `PENDING` bisa diapprove
- state action tidak liar

## TASK MR-06

**Nama task:** Build Reseller Pending and Active Shell States  
**Tujuan:** Menyediakan perilaku area reseller untuk status pending dan aktif.  
**File yang dibuat/diubah:**  
- `src/app/(Reseller)/reseller/page.tsx`
- `src/app/components/reseller/konsumen/ResellerPendingState.tsx`
- `src/hooks/reseller/useResellerDashboardMock.ts`
**Lokasi file:** reseller route + components + hooks  
**Langkah kerja AI agent:**
1. Tampilkan pending state khusus jika reseller belum approve.
2. Tampilkan beranda ringkas jika reseller aktif.
3. Pastikan header dan periode tetap terlihat.
**Dependency:** auth/profile mock + reseller summary mock  
**Output yang diharapkan:** state reseller pending/aktif dapat didemokan  
**Acceptance criteria:**
- reseller pending tidak bisa masuk flow operasional
- reseller aktif melihat ringkasan minimum

## TASK MR-07

**Nama task:** Build Reseller Konsumen List and Search  
**Tujuan:** Menyediakan daftar konsumen mobile-first untuk reseller.  
**File yang dibuat/diubah:**  
- `src/app/(Reseller)/reseller/konsumen/page.tsx`
- `src/app/components/reseller/konsumen/ResellerKonsumenScreen.tsx`
- `src/app/components/reseller/konsumen/KonsumenCard.tsx`
- `src/app/components/reseller/konsumen/KonsumenSearchList.tsx`
- `src/hooks/reseller/useKonsumenListMock.ts`
**Lokasi file:** reseller konsumen folders  
**Langkah kerja AI agent:**
1. Render search sticky.
2. Render card konsumen mobile.
3. Default sort `sisa_bayar desc`, lalu `nama_konsumen asc` bila memakai summary data.
**Dependency:** dummy konsumen + reseller UI contract  
**Output yang diharapkan:** daftar konsumen cepat dan mobile-first  
**Acceptance criteria:**
- tidak ada horizontal scroll
- search mudah dijangkau

## TASK MR-08

**Nama task:** Build Konsumen Create/Edit/Detail Mock  
**Tujuan:** Menyediakan CRUD ringan konsumen milik reseller.  
**File yang dibuat/diubah:**  
- `src/app/(Reseller)/reseller/konsumen/create/page.tsx`
- `src/app/(Reseller)/reseller/konsumen/[id]/page.tsx`
- `src/app/components/reseller/konsumen/KonsumenForm.tsx`
- `src/app/components/reseller/konsumen/KonsumenDetailScreen.tsx`
- `src/hooks/reseller/useKonsumenDetailMock.ts`
- `src/hooks/reseller/useKonsumenActionsMock.ts`
**Lokasi file:** reseller konsumen folders  
**Langkah kerja AI agent:**
1. Buat form create/edit konsumen.
2. Buat detail konsumen sederhana.
3. Pastikan scope konsumen tetap milik reseller aktif.
**Dependency:** konsumen list + hooks  
**Output yang diharapkan:** flow konsumen dasar siap  
**Acceptance criteria:**
- create/edit konsumen bisa didemokan
- detail konsumen tidak bocor lintas reseller

## TASK MR-09

**Nama task:** Build Reseller Core Schema  
**Tujuan:** Menetapkan fondasi data reseller, profile, reseller_periode, dan konsumen.  
**File yang dibuat/diubah:**  
- `supabase/migrations/001_init_schema.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Definisikan tabel inti reseller-related.
2. Tambahkan constraint `UNIQUE (periode_id, no_reseller)` untuk `reseller_periode`.
3. Pastikan `profile` menjadi basis auth scope.
**Dependency:** `schema_mapping.md`  
**Output yang diharapkan:** fondasi schema reseller tersedia  
**Acceptance criteria:**
- `no_reseller` menjadi key bisnis yang jelas
- `profile` dan `reseller` tidak saling ambigu

## TASK MR-10

**Nama task:** Build Reseller RLS and Auth Helpers  
**Tujuan:** Menyediakan helper auth dan policy aman untuk reseller/konsumen.  
**File yang dibuat/diubah:**  
- `supabase/migrations/002_rls_policies.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Tambahkan helper `auth_no_reseller()` dan `is_admin()`.
2. Bentuk policy reseller dan konsumen.
3. Pastikan reseller tidak bisa melihat data reseller lain.
**Dependency:** schema reseller + `rls_matrix.md`  
**Output yang diharapkan:** access model reseller aman  
**Acceptance criteria:**
- test case matrix utama dapat dipetakan
- route reseller tidak butuh input bebas `no_reseller`

## TASK MR-11

**Nama task:** Build Reseller Read Models  
**Tujuan:** Menyediakan ringkasan reseller dan daftar konsumen yang siap dipakai UI.  
**File yang dibuat/diubah:**  
- `supabase/migrations/004_views_ringkasan.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Bentuk `v_ringkasan_reseller`.
2. Bentuk `v_ringkasan_konsumen`.
3. Pastikan hasil mudah dipakai admin dan reseller.
**Dependency:** schema reseller + schema program minimum  
**Output yang diharapkan:** read model reseller siap  
**Acceptance criteria:**
- admin dapat membaca ringkasan reseller
- reseller dapat membaca daftar konsumennya sendiri

## TASK MR-12

**Nama task:** Build Reseller SQL Boundary  
**Tujuan:** Membuat boundary create/update/list/detail reseller dan konsumen.  
**File yang dibuat/diubah:**  
- `supabase/migrations/006_functions_reseller.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Buat boundary create/list/detail reseller admin.
2. Buat boundary update status reseller.
3. Buat boundary list/create/update konsumen by actor.
**Dependency:** schema + RLS + read model  
**Output yang diharapkan:** SQL boundary reseller siap  
**Acceptance criteria:**
- admin dan reseller punya jalur boundary berbeda

## TASK MR-13

**Nama task:** Build Reseller Validation Layer  
**Tujuan:** Menyediakan validasi input reseller dan konsumen.  
**File yang dibuat/diubah:**  
- `src/lib/server/validation/reseller.validation.ts`
**Lokasi file:** `src/lib/server/validation/`  
**Langkah kerja AI agent:**
1. Validasi create reseller.
2. Validasi update status reseller.
3. Validasi create/update konsumen.
**Dependency:** integration contract + SQL boundary assumptions  
**Output yang diharapkan:** validasi reseller terstruktur  
**Acceptance criteria:**
- route tidak memuat validasi detail

## TASK MR-14

**Nama task:** Build Reseller Service Layer  
**Tujuan:** Membungkus boundary reseller ke service yang rapi.  
**File yang dibuat/diubah:**  
- `src/lib/server/services/reseller.service.ts`
**Lokasi file:** `src/lib/server/services/`  
**Langkah kerja AI agent:**
1. Implement fungsi admin dan self-service reseller.
2. Pisahkan mapping error admin vs reseller.
3. Pastikan scope actor selalu diperiksa.
**Dependency:** validation + SQL boundary  
**Output yang diharapkan:** service reseller siap  
**Acceptance criteria:**
- service admin dan reseller dapat dipanggil route tanpa logic liar

## TASK MR-15

**Nama task:** Build Admin Reseller API Route  
**Tujuan:** Menyediakan route internal admin untuk modul reseller.  
**File yang dibuat/diubah:**  
- `src/app/api/admin/reseller/route.ts`
**Lokasi file:** `src/app/api/admin/reseller/`  
**Langkah kerja AI agent:**
1. Definisikan GET/POST/PATCH untuk admin.
2. Panggil validation dan service.
3. Gunakan response envelope standar.
**Dependency:** service reseller  
**Output yang diharapkan:** endpoint admin reseller siap  
**Acceptance criteria:**
- code error konsisten
- response shape seragam

## TASK MR-16

**Nama task:** Build Reseller Konsumen and Akun Routes  
**Tujuan:** Menyediakan route reseller untuk data miliknya sendiri.  
**File yang dibuat/diubah:**  
- `src/app/api/reseller/konsumen/route.ts`
- `src/app/api/reseller/akun/route.ts`
**Lokasi file:** `src/app/api/reseller/`  
**Langkah kerja AI agent:**
1. Definisikan GET/POST/PATCH konsumen by actor.
2. Definisikan GET profile/ringkasan reseller sendiri.
3. Tolak input scope `no_reseller` bebas dari client.
**Dependency:** service reseller + auth helper  
**Output yang diharapkan:** self-service reseller aman  
**Acceptance criteria:**
- route reseller hanya membuka data miliknya

---

## 9. Pseudocode dan Flow Penting

## 9.1 Pseudocode Approve Reseller

```ts
async function approveReseller(noReseller, actor) {
  ensureActorIsAdmin(actor);

  const reseller = await getResellerDetail(noReseller);
  if (!reseller) throw error('RESELLER_NOT_FOUND');
  if (reseller.status === 'AKTIF') return reseller;

  await updateResellerStatus(noReseller, 'AKTIF');
  await ensureResellerPeriodeExistsForActivePeriode(noReseller);

  return await getResellerDetail(noReseller);
}
```

## 9.2 Pseudocode Create Konsumen by Reseller

```ts
async function createKonsumenByActor(input, actor) {
  const noReseller = requireAuthNoReseller(actor);
  ensureResellerIsOperational(actor);

  const payload = {
    no_reseller: noReseller,
    nama_konsumen: input.nama_konsumen,
    telepon: input.telepon ?? null,
    alamat: input.alamat ?? null,
  };

  return await insertKonsumen(payload);
}
```

---

## 10. First Working Slice

`First working slice` modul reseller adalah versi minimum yang sudah cukup untuk membuka jalur
operasional reseller tanpa memaksakan seluruh transaksi selesai.

Isi slice minimum:

- admin dapat melihat daftar reseller
- admin dapat approve reseller `PENDING` menjadi `AKTIF`
- reseller `PENDING` melihat halaman pending state
- reseller `AKTIF` dapat melihat daftar konsumen miliknya
- reseller `AKTIF` dapat membuat konsumen baru
- backend memiliki boundary aman untuk list/update status reseller dan list/create konsumen

Yang sengaja belum diwajibkan di slice pertama:

- edit profile reseller lengkap
- summary keuangan reseller penuh
- operasi program/order/setoran
- automation approval atau onboarding kompleks

Definisi selesai slice pertama:

- satu jalur demo admin approve reseller -> reseller aktif -> reseller tambah konsumen dapat dijalankan

---

## 11. Task Execution Batches

### batch_a_foundation

- `MR-01` Define Reseller Module Types and Contracts
- `MR-09` Build Reseller Core Schema
- `MR-10` Build Reseller RLS and Auth Helpers

### batch_b_mock_or_read

- `MR-02` Build Reseller Mock Data Pack
- `MR-03` Build Admin Reseller Mock Hooks and Actions
- `MR-11` Build Reseller Read Models

### batch_c_primary_actions

- `MR-04` Build Admin Reseller List Screen
- `MR-05` Build Admin Reseller Form and Detail
- `MR-06` Build Reseller Pending and Active Shell States
- `MR-07` Build Reseller Konsumen List and Search
- `MR-08` Build Konsumen Create/Edit/Detail Mock

### batch_d_backend_boundary

- `MR-12` Build Reseller SQL Boundary
- `MR-13` Build Reseller Validation Layer
- `MR-14` Build Reseller Service Layer
- `MR-15` Build Admin Reseller API Route
- `MR-16` Build Reseller Konsumen and Akun Routes

---

## 12. Rule Khusus AI Coding Agent

- Jangan izinkan reseller mengirim `no_reseller` bebas untuk scope data utamanya.
- Jangan anggap reseller `PENDING` bisa masuk ke flow operasional program/order/setoran.
- Jangan campur detail keuangan reseller penuh ke modul ini sebelum read model dasarnya stabil.
- Untuk area mobile reseller, utamakan layar sempit dan alur cepat, bukan tabel admin mini.

---

## 13. Urutan Eksekusi Modul

1. definisikan contract tipe modul reseller
2. siapkan schema reseller, profile, reseller_periode, konsumen
3. siapkan helper auth dan policy
4. siapkan dummy data reseller dan konsumen
5. siapkan hooks admin reseller
6. bangun daftar reseller admin
7. bangun detail dan aksi status reseller
8. bangun pending state dan beranda reseller minimum
9. bangun daftar konsumen reseller
10. bangun create/edit/detail konsumen mock
11. bangun read model reseller dan konsumen
12. bangun SQL boundary reseller
13. bangun validation dan service layer
14. bangun route admin reseller
15. bangun route reseller konsumen dan akun
16. verifikasi alignment admin-reseller-scope actor

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- scope `no_reseller` bocor dan membuat reseller bisa membaca data orang lain
- modul reseller berkembang terlalu jauh menjadi modul program/setoran sebelum waktunya
- admin summary reseller terlalu bergantung pada read model modul lain yang belum siap

### Asumsi

- `no_reseller` tetap menjadi identifier bisnis utama
- `profile` menjadi basis auth scope reseller
- approval reseller tetap aksi admin, bukan self-approve

### Catatan Keputusan Fase Awal

- create reseller memakai dua jalur: admin create dan self-register `PENDING`
- admin mengelola `reseller_periode` sebagai area tersendiri karena route dan modulnya sudah aktif
- detail konsumen awal cukup menyimpan nama, telepon, dan alamat sampai ada kontrak field tambahan resmi
