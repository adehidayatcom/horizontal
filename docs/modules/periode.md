# Execution Module Periode
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `periode`.

Dokumen ini memecah implementasi modul periode ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/prd.md`
- `docs/product_truth_audit.md`
- `docs/business_contracts.md`
- `docs/schema_mapping.md`
- `docs/query_contracts.md`
- `docs/support/system_maps.md`
- `docs/execution/task_kit_standard.md`
- `docs/frontend_architecture.md`
- `docs/frontend_component_contracts.md`
- `docs/navigation_and_period_setup_ui.md`
- `docs/integration_contract_pack.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`

---

## 1. Tujuan Modul

Modul `periode` bertanggung jawab untuk:

- membuat periode baru
- menampilkan daftar dan detail periode
- memandu setup periode sebelum aktivasi
- memvalidasi readiness aktivasi
- mengubah status periode secara terkendali
- membedakan periode aktif operasional dan periode laporan

Modul ini adalah fondasi sistem karena banyak modul lain bergantung pada konteks `periode_id`.

---

## 2. Scope Modul

### Frontend Mockup Scope

- daftar periode admin
- form create/edit periode
- detail periode
- wizard setup periode
- checklist aktivasi
- state empty saat belum ada periode
- state aktif, persiapan, selesai

### Backend/Core Scope

- schema dan field periode
- boundary create/update/read periode
- readiness checklist logic
- guard hanya satu periode aktif
- query list/detail/ringkasan periode
- audit status periode

---

## 3. Integration Contract Modul Periode

## 3.1 Entity Shape

```ts
type PeriodeRecord = {
  id: number;
  nama_periode: string;
  tgl_mulai: string;
  tgl_selesai: string;
  status: 'PERSIAPAN' | 'AKTIF' | 'SELESAI';
  created_at: string;
  updated_at?: string | null;
};
```

## 3.2 Ringkasan Periode

```ts
type PeriodeSummary = {
  id: number;
  nama_periode: string;
  tgl_mulai: string;
  tgl_selesai: string;
  status: 'PERSIAPAN' | 'AKTIF' | 'SELESAI';
  jumlah_reseller: number;
  jumlah_konsumen: number;
  jumlah_program: number;
  jumlah_order: number;
  setup_progress_persen: number;
  semua_syarat_aktivasi_terpenuhi: boolean;
};
```

## 3.3 Request Contract

```ts
type CreatePeriodeRequest = {
  nama_periode: string;
  tgl_mulai: string;
  tgl_selesai: string;
};

type UpdatePeriodeStatusRequest = {
  periode_id: number;
  status: 'PERSIAPAN' | 'AKTIF' | 'SELESAI';
};
```

## 3.4 Error Codes Relevan

```txt
PERIODE_NOT_FOUND
PERIODE_INVALID_DATE_RANGE
PERIODE_OVERLAP
PERIODE_ALREADY_ACTIVE_EXISTS
PERIODE_SETUP_INCOMPLETE
PERIODE_INVALID_STATUS_TRANSITION
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
```

---

## 4. Struktur Folder Modul Periode

## 4.1 Frontend

```txt
src/app/(DashboardLayout)/admin/periode/
  page.tsx
  create/
    page.tsx
  [id]/
    page.tsx
    setup/
      page.tsx
    checklist/
      page.tsx

src/app/components/admin/periode/
  PeriodeListScreen.tsx
  PeriodeForm.tsx
  PeriodeTable.tsx
  PeriodeDetailScreen.tsx
  PeriodeSetupWizard.tsx
  PeriodeChecklist.tsx
  PeriodeSummaryCards.tsx
  PeriodeStatusActions.tsx
  PeriodeEmptyState.tsx
  index.ts

src/hooks/admin/
  usePeriodeListMock.ts
  usePeriodeDetailMock.ts
  usePeriodeChecklistMock.ts
  usePeriodeActionsMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  004_views_ringkasan.sql
  005_functions_periode.sql
  012_triggers_audit.sql

src/lib/server/services/
  periode.service.ts

src/lib/server/validation/
  periode.validation.ts

src/lib/server/workflows/
  periode.workflow.ts

src/app/api/admin/periode/
  route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `admin/periode/page.tsx` | Entry route daftar periode |
| `admin/periode/create/page.tsx` | Entry route create periode |
| `admin/periode/[id]/page.tsx` | Entry route detail periode |
| `admin/periode/[id]/setup/page.tsx` | Entry route wizard setup |
| `admin/periode/[id]/checklist/page.tsx` | Entry route checklist aktivasi |
| `PeriodeListScreen.tsx` | Screen utama daftar periode |
| `PeriodeForm.tsx` | Form create/edit mock |
| `PeriodeTable.tsx` | Table daftar periode |
| `PeriodeDetailScreen.tsx` | Screen detail periode |
| `PeriodeSetupWizard.tsx` | Wizard setup multi-step |
| `PeriodeChecklist.tsx` | Checklist aktivasi |
| `PeriodeSummaryCards.tsx` | Ringkasan mini untuk detail periode |
| `PeriodeStatusActions.tsx` | Tombol aksi status |
| `PeriodeEmptyState.tsx` | Empty state jika belum ada periode |
| `usePeriodeListMock.ts` | Hook data mock daftar periode |
| `usePeriodeDetailMock.ts` | Hook data mock detail periode |
| `usePeriodeChecklistMock.ts` | Hook data mock checklist |
| `usePeriodeActionsMock.ts` | Hook action mock create/activate/close |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Definisi tabel `periode` |
| `004_views_ringkasan.sql` | View ringkasan readiness periode |
| `005_functions_periode.sql` | Boundary create/update/list/detail/checklist periode |
| `012_triggers_audit.sql` | Audit untuk perubahan periode |
| `periode.service.ts` | Wrapper boundary periode |
| `periode.validation.ts` | Validasi input periode |
| `periode.workflow.ts` | Orkestrasi readiness dan transisi status |
| `admin/periode/route.ts` | Route HTTP internal admin periode |

---

## 6. Hubungan Antar File

### Frontend Flow

```txt
page.tsx
-> screen component
-> mock hook
-> query keys / mock fetch
-> dummy periode data
```

### Backend Flow

```txt
route.ts
-> validation
-> service
-> workflow
-> SQL boundary
-> response envelope
```

### Integration Touchpoints

- frontend mock memakai shape `PeriodeRecord`, `PeriodeSummary`
- backend route dan service harus mengeluarkan shape yang sama
- readiness checklist di mock harus mengikuti struktur backend final

---

## 7. Breakdown Function dan Component per File

## 7.1 `PeriodeForm.tsx`

Komponen:

- `PeriodeForm`

Tanggung jawab:

- render field `nama_periode`, `tgl_mulai`, `tgl_selesai`
- validasi UI awal
- submit ke action mock

State:

- `isSubmitting`
- `serverError`
- `formMode` (`create` | `edit`)

## 7.2 `PeriodeSetupWizard.tsx`

Komponen:

- `PeriodeSetupWizard`
- `WizardStepHeader`
- `WizardStepContent`
- `WizardNavigation`

Tanggung jawab:

- memandu admin melalui step setup
- menandai step complete/incomplete
- menahan aktivasi bila syarat belum terpenuhi

## 7.3 `PeriodeChecklist.tsx`

Komponen:

- `PeriodeChecklist`
- `ChecklistItem`
- `ChecklistSummary`

Tanggung jawab:

- render item readiness
- membedakan blocker dan warning
- menampilkan CTA aktivasi bila memenuhi syarat

## 7.4 `usePeriodeActionsMock.ts`

Functions:

- `createPeriodeMock`
- `activatePeriodeMock`
- `closePeriodeMock`
- `duplicateFromPreviousPeriodeMock`

Tanggung jawab:

- simulasi perubahan state periode
- simulasi error kontrak
- update local dataset mock

## 7.5 `periode.service.ts`

Functions:

- `createPeriode`
- `updatePeriodeStatus`
- `getPeriodeList`
- `getPeriodeDetail`
- `getPeriodeChecklist`

Tanggung jawab:

- menjadi lapisan tunggal akses modul periode dari route
- memanggil workflow/helper bila dibutuhkan

## 7.6 `periode.workflow.ts`

Functions:

- `validatePeriodeDateRange`
- `ensureNoOtherActivePeriode`
- `buildPeriodeChecklist`
- `validatePeriodeStatusTransition`
- `ensurePeriodeReadyForActivation`

---

## 8. Task Implementation Detail

## TASK MP-01

**Nama task:** Define Periode Types and Contracts  
**Tujuan:** Menetapkan type dan contract modul periode untuk frontend dan backend.  
**File yang dibuat/diubah:**  
- `docs/integration_contract_pack.md`
- `src/types/models.ts`
- `src/types/api.ts`
**Lokasi file:** docs + types  
**Langkah kerja AI agent:**
1. Pastikan shape `PeriodeRecord`, `PeriodeSummary`, dan request contract sudah konsisten.
2. Tambahkan type turunan yang dibutuhkan screen atau service.
3. Pastikan status hanya `PERSIAPAN`, `AKTIF`, `SELESAI`.
**Dependency:** `schema_mapping.md`, `query_contracts.md`  
**Output yang diharapkan:** kontrak modul periode stabil  
**Acceptance criteria:**
- semua type utama modul periode terdefinisi
- frontend dan backend merujuk ke shape yang sama

## TASK MP-02

**Nama task:** Build Periode Mock Data Pack  
**Tujuan:** Menyediakan dataset mock untuk semua state modul periode.  
**File yang dibuat/diubah:**  
- `src/lib/dummy-data/periodes.ts`
**Lokasi file:** `src/lib/dummy-data/`  
**Langkah kerja AI agent:**
1. Buat data periode kosong, persiapan, aktif, selesai.
2. Tambahkan item readiness per periode persiapan.
3. Tambahkan skenario error seperti overlap atau aktivasi gagal.
**Dependency:** tipe modul periode  
**Output yang diharapkan:** dataset mock periode lengkap  
**Acceptance criteria:**
- state empty, normal, warning, dan blocked tersedia

## TASK MP-03

**Nama task:** Build Periode Mock Hooks and Actions  
**Tujuan:** Menyediakan layer hook/action mock untuk modul periode.  
**File yang dibuat/diubah:**  
- `src/hooks/admin/usePeriodeListMock.ts`
- `src/hooks/admin/usePeriodeDetailMock.ts`
- `src/hooks/admin/usePeriodeChecklistMock.ts`
- `src/hooks/admin/usePeriodeActionsMock.ts`
**Lokasi file:** `src/hooks/admin/`  
**Langkah kerja AI agent:**
1. Bungkus dummy API/fetcher dengan query keys.
2. Pisahkan list, detail, checklist, dan actions.
3. Tambahkan pending/error/success state.
**Dependency:** dummy data periode + placeholder API layer  
**Output yang diharapkan:** layer data mock modul periode siap  
**Acceptance criteria:**
- page dan screen tidak akses dummy data langsung

## TASK MP-04

**Nama task:** Build Periode List Route and Screen  
**Tujuan:** Menampilkan daftar periode admin.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/periode/page.tsx`
- `src/app/components/admin/periode/PeriodeListScreen.tsx`
- `src/app/components/admin/periode/PeriodeTable.tsx`
- `src/app/components/admin/periode/PeriodeEmptyState.tsx`
**Lokasi file:** route + admin periode components  
**Langkah kerja AI agent:**
1. Bentuk page entry.
2. Buat screen wrapper.
3. Buat table daftar periode.
4. Buat empty state jika belum ada periode.
**Dependency:** mock hooks + shared components  
**Output yang diharapkan:** daftar periode bisa dinavigasi  
**Acceptance criteria:**
- list tampil
- empty state tampil saat data kosong
- state loading/error tersedia

## TASK MP-05

**Nama task:** Build Periode Form Create/Edit  
**Tujuan:** Menyediakan form create/edit periode mock.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/periode/create/page.tsx`
- `src/app/components/admin/periode/PeriodeForm.tsx`
**Lokasi file:** route + component  
**Langkah kerja AI agent:**
1. Buat form fields.
2. Tambahkan validation schema.
3. Hubungkan ke action mock create.
4. Tampilkan feedback sukses/gagal.
**Dependency:** validation UI + mock actions  
**Output yang diharapkan:** create/edit mock berjalan  
**Acceptance criteria:**
- invalid date range ditolak secara UI mock
- submit state jelas

## TASK MP-06

**Nama task:** Build Periode Detail Screen  
**Tujuan:** Menampilkan detail periode dan ringkasannya.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/periode/[id]/page.tsx`
- `src/app/components/admin/periode/PeriodeDetailScreen.tsx`
- `src/app/components/admin/periode/PeriodeSummaryCards.tsx`
- `src/app/components/admin/periode/PeriodeStatusActions.tsx`
**Lokasi file:** route + component  
**Langkah kerja AI agent:**
1. Buat detail layout.
2. Tampilkan ringkasan periode.
3. Tampilkan aksi status mock.
4. Tampilkan tautan ke setup/checklist.
**Dependency:** detail hook + summary type  
**Output yang diharapkan:** detail periode informatif  
**Acceptance criteria:**
- status periode terlihat jelas
- user bisa menuju setup/checklist

## TASK MP-07

**Nama task:** Build Periode Setup Wizard Mock  
**Tujuan:** Menyediakan wizard persiapan periode.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/periode/[id]/setup/page.tsx`
- `src/app/components/admin/periode/PeriodeSetupWizard.tsx`
**Lokasi file:** route + component  
**Langkah kerja AI agent:**
1. Definisikan step setup.
2. Render progress/step indicator.
3. Tampilkan state complete/incomplete per step.
4. Hubungkan ke data mock readiness.
**Dependency:** checklist mock + navigation contract  
**Output yang diharapkan:** setup periode dapat didemokan  
**Acceptance criteria:**
- wizard step terlihat
- progress dan blocker dapat dipahami

## TASK MP-08

**Nama task:** Build Periode Activation Checklist Mock  
**Tujuan:** Menyediakan checklist readiness aktivasi.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/periode/[id]/checklist/page.tsx`
- `src/app/components/admin/periode/PeriodeChecklist.tsx`
**Lokasi file:** route + component  
**Langkah kerja AI agent:**
1. Render item checklist.
2. Bedakan blocker dan warning.
3. Tampilkan CTA aktivasi mock jika syarat terpenuhi.
**Dependency:** checklist hook  
**Output yang diharapkan:** aktivasi mock terstruktur  
**Acceptance criteria:**
- admin bisa melihat alasan periode belum siap
- tombol aktivasi tidak muncul sembarangan

## TASK MP-09

**Nama task:** Build Periode Schema and Migration  
**Tujuan:** Menetapkan fondasi data periode pada layer engine/backend.  
**File yang dibuat/diubah:**  
- `supabase/migrations/001_init_schema.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Definisikan tabel `periode`.
2. Definisikan constraint tanggal.
3. Tambahkan status enum atau check.
4. Tambahkan index yang relevan.
**Dependency:** `schema_mapping.md`  
**Output yang diharapkan:** struktur data periode tersedia  
**Acceptance criteria:**
- tabel periode memiliki field minimum yang disetujui
- status terkendali secara struktural

## TASK MP-10

**Nama task:** Build Periode Read Model  
**Tujuan:** Menyediakan list/detail/ringkasan periode untuk konsumsi frontend.  
**File yang dibuat/diubah:**  
- `supabase/migrations/004_views_ringkasan.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Bentuk view atau query helper ringkasan periode.
2. Tambahkan count dan progress readiness.
3. Pastikan hasil siap konsumsi screen detail.
**Dependency:** schema periode + entitas terkait minimum  
**Output yang diharapkan:** read model periode siap  
**Acceptance criteria:**
- frontend tidak perlu menghitung readiness dari nol

## TASK MP-11

**Nama task:** Build Periode SQL Boundary  
**Tujuan:** Membuat create/list/detail/checklist/update status untuk periode.  
**File yang dibuat/diubah:**  
- `supabase/migrations/005_functions_periode.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Buat boundary create periode.
2. Buat boundary list/detail periode.
3. Buat boundary checklist readiness.
4. Buat boundary perubahan status.
**Dependency:** schema periode + read model periode  
**Output yang diharapkan:** SQL boundary periode siap  
**Acceptance criteria:**
- semua aksi utama periode punya boundary resmi

## TASK MP-12

**Nama task:** Build Periode Validation Layer  
**Tujuan:** Menyediakan validasi input dan validasi transisi periode.  
**File yang dibuat/diubah:**  
- `src/lib/server/validation/periode.validation.ts`
**Lokasi file:** `src/lib/server/validation/`  
**Langkah kerja AI agent:**
1. Validasi request create.
2. Validasi request status transition.
3. Definisikan pesan error domain.
**Dependency:** integration contract + business contracts  
**Output yang diharapkan:** validasi periode terstruktur  
**Acceptance criteria:**
- validation terpisah dari route
- pesan error konsisten

## TASK MP-13

**Nama task:** Build Periode Workflow Layer  
**Tujuan:** Menyediakan helper readiness dan status transition rule.  
**File yang dibuat/diubah:**  
- `src/lib/server/workflows/periode.workflow.ts`
**Lokasi file:** `src/lib/server/workflows/`  
**Langkah kerja AI agent:**
1. Buat helper date range rule.
2. Buat helper no active periode conflict.
3. Buat helper checklist readiness evaluation.
4. Buat helper status transition validation.
**Dependency:** validation + SQL boundary assumptions  
**Output yang diharapkan:** workflow helper periode siap  
**Acceptance criteria:**
- logic transisi tidak tercecer di banyak file

## TASK MP-14

**Nama task:** Build Periode Service Layer  
**Tujuan:** Membungkus boundary SQL ke service yang rapi.  
**File yang dibuat/diubah:**  
- `src/lib/server/services/periode.service.ts`
**Lokasi file:** `src/lib/server/services/`  
**Langkah kerja AI agent:**
1. Implement fungsi create/list/detail/checklist/updateStatus.
2. Hubungkan ke workflow bila perlu.
3. Standarkan response dan error mapping.
**Dependency:** validation + workflow + SQL boundary  
**Output yang diharapkan:** service periode siap dipakai route  
**Acceptance criteria:**
- route tidak memuat logic periode final

## TASK MP-15

**Nama task:** Build Admin Periode API Route  
**Tujuan:** Menyediakan route internal untuk modul periode.  
**File yang dibuat/diubah:**  
- `src/app/api/admin/periode/route.ts`
**Lokasi file:** `src/app/api/admin/periode/`  
**Langkah kerja AI agent:**
1. Definisikan method GET/POST/PATCH sesuai kebutuhan.
2. Panggil validation dan service.
3. Keluarkan response envelope standar.
**Dependency:** service periode  
**Output yang diharapkan:** endpoint internal periode siap  
**Acceptance criteria:**
- response memakai contract standar
- error memakai code yang konsisten

## TASK MP-16

**Nama task:** Build Audit Hook for Periode Changes  
**Tujuan:** Mencatat perubahan create/status periode.  
**File yang dibuat/diubah:**  
- `supabase/migrations/012_triggers_audit.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Tambahkan trigger audit untuk periode.
2. Definisikan aksi create/update status.
3. Pastikan field inti perubahan tersimpan.
**Dependency:** audit log schema  
**Output yang diharapkan:** perubahan periode bisa diaudit  
**Acceptance criteria:**
- create dan perubahan status punya jejak audit

---

## 9. Pseudocode dan Flow Penting

## 9.1 Pseudocode Aktivasi Periode

```ts
async function activatePeriode(periodeId, actor) {
  ensureActorIsAdmin(actor);

  const periode = await getPeriodeDetail(periodeId);
  if (!periode) throw error('PERIODE_NOT_FOUND');

  ensureStatusTransitionAllowed(periode.status, 'AKTIF');
  await ensureNoOtherActivePeriode();

  const checklist = await buildPeriodeChecklist(periodeId);
  if (!checklist.semua_syarat_aktivasi_terpenuhi) {
    throw error('PERIODE_SETUP_INCOMPLETE');
  }

  await updatePeriodeStatus(periodeId, 'AKTIF');
  await writeAudit('PERIODE_DIAKTIFKAN', periodeId, actor.id);

  return successPayload();
}
```

## 9.2 Pseudocode Create Periode Mock

```ts
function createPeriodeMock(input) {
  validateDateRange(input.tgl_mulai, input.tgl_selesai);

  const periodeBaru = {
    id: generateMockId(),
    nama_periode: input.nama_periode,
    tgl_mulai: input.tgl_mulai,
    tgl_selesai: input.tgl_selesai,
    status: 'PERSIAPAN',
  };

  dummyPeriodes.unshift(periodeBaru);
  return periodeBaru;
}
```

---

## 10. Urutan Eksekusi Modul

1. definisikan contract tipe modul periode
2. siapkan dummy data periode
3. siapkan hook dan action mock
4. bangun list periode
5. bangun form create/edit
6. bangun detail periode
7. bangun wizard setup
8. bangun checklist aktivasi
9. bangun schema dan read model periode
10. bangun SQL boundary periode
11. bangun validation/workflow/service
12. bangun route internal admin periode
13. tambahkan audit perubahan
14. verifikasi alignment frontend-backend modul periode

---

## 11. First Working Slice

`First working slice` modul periode adalah versi minimum yang sudah layak didemokan dan dijadikan
fondasi modul lain.

Isi slice minimum:

- admin dapat melihat daftar periode
- admin dapat membuat periode `PERSIAPAN`
- admin dapat membuka detail periode
- admin dapat melihat checklist readiness minimum
- backend memiliki boundary create/list/detail/checklist
- backend menolak aktivasi bila readiness minimum belum terpenuhi

Yang sengaja belum diwajibkan di slice pertama:

- duplicasi setup dari periode sebelumnya
- koreksi kompleks periode `SELESAI`
- wizard setup penuh lintas semua master periodik

Definisi selesai slice pertama:

- frontend mock dan backend boundary memakai shape periode yang sama
- satu demo end-to-end create -> list -> detail -> checklist -> activate guard dapat dijalankan

---

## 12. Task Execution Batches

### batch_a_foundation

- `MP-01` Define Periode Types and Contracts
- `MP-09` Build Periode Schema and Migration

### batch_b_mock_or_read

- `MP-02` Build Periode Mock Data Pack
- `MP-03` Build Periode Mock Hooks and Actions
- `MP-10` Build Periode Read Model

### batch_c_primary_actions

- `MP-04` Build Periode List Route and Screen
- `MP-05` Build Periode Form Create/Edit
- `MP-06` Build Periode Detail Screen
- `MP-08` Build Periode Activation Checklist Mock

### batch_d_backend_boundary

- `MP-11` Build Periode SQL Boundary
- `MP-12` Build Periode Validation Layer
- `MP-13` Build Periode Workflow Layer
- `MP-14` Build Periode Service Layer
- `MP-15` Build Admin Periode API Route
- `MP-16` Build Audit Hook for Periode Changes

### batch_e_extended_demo

- `MP-07` Build Periode Setup Wizard Mock

---

## 13. Rule Khusus AI Coding Agent

- Jangan mulai dari wizard setup jika list/detail/checklist belum ada.
- Jangan membuat toggle status periode langsung di UI tanpa guard backend.
- Jangan menambah status periode baru di luar `PERSIAPAN`, `AKTIF`, `SELESAI`.
- Jika field readiness belum final di modul lain, render sebagai checklist minimum yang stabil, bukan kalkulasi liar di UI.

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- readiness checklist terlalu UI-driven dan tidak punya padanan backend
- field readiness berubah-ubah saat modul lain belum stabil
- status periode bocor menjadi toggle UI biasa tanpa aturan transisi

### Asumsi

- periode tetap menjadi gerbang operasional utama
- hanya satu periode aktif pada satu waktu
- wizard setup tetap menjadi pengalaman admin utama sebelum aktivasi

### Catatan dan Deferred Scope

- perubahan pada periode `AKTIF` dibatasi ke aksi admin terkontrol dengan warning dampak, bukan edit bebas
- duplikasi data dari periode sebelumnya tetap deferred sesuai `PQ-004`
- frontend mockup awal boleh menghitung checklist dari dummy layer selama shape-nya tetap tunduk ke kontrak backend final
