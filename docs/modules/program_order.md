# Execution Module Pesanan Order
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `program_order`.

Nama file tetap dipertahankan untuk kompatibilitas referensi lama, tetapi isi dokumen ini sekarang
menjadi blueprint resmi untuk model:

```txt
konsumen
-> pesanan_konsumen
-> detail_pesanan_konsumen
-> setoran_konsumen
-> finalisasi_pesanan_konsumen
-> gudang / pengiriman / pembagian
```

Dokumen ini memecah implementasi modul ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/product/prd.md`
- `docs/audit/product_truth_audit.md`
- `docs/product/program_workflow.md`
- `docs/contracts/business_contracts.md`
- `docs/contracts/schema_mapping.md`
- `docs/contracts/query_contracts.md`
- `docs/truth/system_maps.md`
- `docs/execution/task_kit_standard.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`
- `docs/ui/reseller_uiux.md`

---

## 1. Tujuan Modul

Modul `program_order` bertanggung jawab untuk:

- membuat `pesanan_konsumen` beserta item awal `detail_pesanan_konsumen`
- mengubah detail item sebelum atau sesudah ada setoran sesuai aturan audit
- menampilkan ringkasan dan watchlist pesanan
- mencatat finalisasi pesanan melalui `tanggal_final`
- menampilkan daftar `detail_pesanan_konsumen` aktif/final untuk admin dan reseller
- menangani status item `AKTIF`, `TERHENTI`, `BATAL` sesuai aturan bisnis

Modul ini adalah inti hubungan antara pilihan paket konsumen, cicilan/setoran, dan operasional
gudang.

---

## 2. Scope Modul

### Frontend Mockup Scope

- daftar pesanan konsumen admin
- daftar pesanan konsumen reseller
- create/edit `detail_pesanan_konsumen` mock
- detail `pesanan_konsumen`
- watchlist pesanan perlu perhatian
- flow finalisasi pesanan
- daftar detail pesanan aktif/final untuk reseller/admin

### Backend/Core Scope

- schema `pesanan_konsumen` dan `detail_pesanan_konsumen`
- boundary `buat_pesanan_konsumen`
- boundary `ubah_detail_pesanan_konsumen`
- boundary `finalisasi_pesanan_konsumen`
- boundary admin `ubah_status_detail_pesanan_konsumen`
- read model `v_ringkasan_pesanan_konsumen`, `v_pesanan_perlu_perhatian`, `v_detail_pesanan_aktif`
- validasi target berjalan dan transisi status header/item

---

## 3. Integration Contract Modul Pesanan

## 3.1 Entity Shape `pesanan_konsumen`

```ts
type PesananRecord = {
  id: number;
  periode_id: number;
  konsumen_id: number;
  no_reseller: string;
  target_tagihan_snapshot: number;
  nominal_harian_opsional?: number | null;
  status_pesanan: 'AKTIF' | 'PERLU_PERHATIAN' | 'SELESAI' | 'BATAL';
  tanggal_mulai?: string;
  tanggal_final?: string | null;
  catatan?: string | null;
};
```

## 3.2 Summary `pesanan_konsumen`

```ts
type PesananSummary = {
  pesanan_konsumen_id: number;
  periode_id: number;
  no_reseller: string;
  konsumen_id: number;
  nama_konsumen: string;
  target_tagihan: number;
  target_berjalan: number;
  total_bayar: number;
  sisa_bayar: number;
  persentase_bayar: number;
  status_pesanan: 'AKTIF' | 'PERLU_PERHATIAN' | 'SELESAI' | 'BATAL';
  tanggal_setoran_terakhir?: string | null;
  hari_tanpa_setoran?: number | null;
  tanggal_final?: string | null;
  is_final: boolean;
};
```

## 3.3 Summary `detail_pesanan_konsumen`

```ts
type DetailPesananSummary = {
  id: number;
  periode_id: number;
  pesanan_konsumen_id: number;
  no_reseller: string;
  nama_reseller?: string;
  konsumen_id: number;
  nama_konsumen: string;
  paket_id: number;
  nama_paket: string;
  kode_paket?: string;
  qty: number;
  status_item: 'AKTIF' | 'TERHENTI' | 'BATAL';
  status_kirim: 'BELUM' | 'SUDAH' | 'BATAL';
  nilai_item: number;
  uang_terhenti?: number | null;
  is_final: boolean;
};
```

## 3.4 Request Contract

```ts
type CreatePesananRequest = {
  periode_id: number;
  konsumen_id: number;
  items: Array<{
    paket_id: number;
    qty?: number;
    motif_warna?: string | null;
    kelompok?: string | null;
  }>;
  nominal_harian_opsional?: number | null;
  catatan?: string | null;
};

type UpdateDetailPesananRequest = {
  pesanan_konsumen_id: number;
  items: Array<{
    id?: number;
    paket_id: number;
    qty?: number;
    motif_warna?: string | null;
    kelompok?: string | null;
    status_item?: 'AKTIF' | 'TERHENTI' | 'BATAL';
    uang_terhenti?: number | null;
  }>;
  alasan?: string | null;
};

type FinalisasiPesananRequest = {
  pesanan_konsumen_id: number;
  catatan?: string | null;
};
```

## 3.5 Error Codes Relevan

```txt
PESANAN_NOT_FOUND
PESANAN_ALREADY_EXISTS
PESANAN_ALREADY_FINAL
PESANAN_INVALID_STATUS
PESANAN_TARGET_INVALID
PESANAN_TARGET_BELOW_PAYMENT
DETAIL_PESANAN_NOT_FOUND
DETAIL_PESANAN_ALREADY_SHIPPED
DETAIL_PESANAN_PERIOD_MISMATCH
DETAIL_PESANAN_TERHENTI_REQUIRES_AMOUNT
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
```

---

## 4. Struktur Folder Modul Pesanan

## 4.1 Frontend

```txt
src/app/(DashboardLayout)/admin/pesanan/
  page.tsx
  perlu-perhatian/
    page.tsx
  [id]/
    page.tsx

src/app/(DashboardLayout)/reseller/pesanan/
  page.tsx
  create/
    page.tsx
  [id]/
    page.tsx

src/app/components/admin/pesanan/
  PesananListScreen.tsx
  PesananWatchlistScreen.tsx
  PesananDetailScreen.tsx
  PesananSummaryCards.tsx
  PesananStatusBadge.tsx
  DetailPesananTable.tsx
  index.ts

src/app/components/reseller/pesanan/
  ResellerPesananScreen.tsx
  PesananCard.tsx
  PesananForm.tsx
  PesananDetailScreen.tsx
  DetailPesananEditor.tsx
  FinalisasiPesananFlow.tsx
  DetailPesananList.tsx
  index.ts

src/hooks/admin/
  usePesananListMock.ts
  usePesananWatchlistMock.ts
  useDetailPesananListMock.ts
  usePesananActionsMock.ts

src/hooks/reseller/
  usePesananListMock.ts
  usePesananDetailMock.ts
  usePesananActionsMock.ts
  useFinalisasiPesananMock.ts
  useDetailPesananListMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  004_views_ringkasan.sql
  007_functions_pesanan.sql
  012_triggers_audit.sql

src/lib/server/services/
  pesanan.service.ts
  detail-pesanan.service.ts

src/lib/server/validation/
  pesanan.validation.ts
  detail-pesanan.validation.ts

src/lib/server/workflows/
  pesanan.workflow.ts

src/app/api/admin/pesanan/
  route.ts

src/app/api/reseller/pesanan/
  route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `admin/pesanan/page.tsx` | Daftar pesanan admin |
| `admin/pesanan/perlu-perhatian/page.tsx` | Watchlist pesanan |
| `admin/pesanan/[id]/page.tsx` | Detail pesanan admin |
| `reseller/pesanan/page.tsx` | Daftar pesanan reseller |
| `reseller/pesanan/create/page.tsx` | Form create pesanan |
| `reseller/pesanan/[id]/page.tsx` | Detail pesanan reseller |
| `PesananListScreen.tsx` | Screen daftar pesanan |
| `PesananWatchlistScreen.tsx` | Screen watchlist pesanan |
| `PesananDetailScreen.tsx` | Detail header pesanan |
| `PesananForm.tsx` | Form create/edit detail pesanan |
| `DetailPesananEditor.tsx` | Editor daftar item paket |
| `FinalisasiPesananFlow.tsx` | Flow review lalu finalisasi pesanan |
| `DetailPesananList.tsx` | Daftar item pesanan untuk reseller |
| `DetailPesananTable.tsx` | Daftar item pesanan untuk admin |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Definisi tabel `pesanan_konsumen` dan `detail_pesanan_konsumen` |
| `004_views_ringkasan.sql` | View pesanan, watchlist, dan detail pesanan aktif |
| `007_functions_pesanan.sql` | Boundary pesanan dan detail pesanan |
| `012_triggers_audit.sql` | Audit perubahan pesanan/detail |
| `pesanan.service.ts` | Wrapper logic header pesanan |
| `detail-pesanan.service.ts` | Wrapper logic item pesanan |
| `pesanan.validation.ts` | Validasi input pesanan |
| `detail-pesanan.validation.ts` | Validasi input item pesanan |
| `pesanan.workflow.ts` | Workflow target berjalan, finalisasi, status item |
| `admin/pesanan/route.ts` | Route admin pesanan |
| `reseller/pesanan/route.ts` | Route reseller pesanan |

---

## 6. Hubungan Antar File

### Frontend Flow Pesanan

```txt
page.tsx
-> PesananListScreen / PesananDetailScreen
-> usePesanan*Mock
-> dummy pesanan data
-> placeholder API route dengan contract yang sama
```

### Frontend Flow Finalisasi

```txt
reseller/pesanan/[id]/page.tsx
-> FinalisasiPesananFlow
-> useFinalisasiPesananMock
-> pesanan summary + editor detail + finalisasi state
```

### Backend Flow

```txt
route.ts
-> validation
-> pesanan.service / detail-pesanan.service
-> pesanan.workflow
-> SQL boundary
-> response envelope
```

### Integration Touchpoints

- daftar pesanan harus memakai `PesananSummary`
- finalisasi harus memakai `FinalisasiPesananRequest`
- daftar item harus memakai `DetailPesananSummary`

---

## 7. Breakdown Function dan Component per File

## 7.1 `PesananForm.tsx`

Komponen:

- `PesananForm`

Tanggung jawab:

- render item awal pesanan
- render catatan dan nominal harian opsional
- submit create/update mock

## 7.2 `FinalisasiPesananFlow.tsx`

Komponen:

- `FinalisasiPesananFlow`
- `PesananSummaryPanel`
- `DetailPesananReviewPanel`
- `FinalisasiGuardPanel`

Tanggung jawab:

- menampilkan header pesanan dan item aktif
- menampilkan target berjalan vs total bayar
- mencegah submit bila target berjalan < total bayar

## 7.3 `pesanan.workflow.ts`

Functions:

- `computeTargetBerjalan`
- `ensurePesananCanBeFinalized`
- `ensureDetailBelongsToPeriode`
- `ensureTargetNotBelowPayment`
- `ensureDetailStatusChangeAllowed`

Tanggung jawab:

- menjaga aturan transisi pesanan
- menjaga validasi target berjalan
- menjaga aturan item `TERHENTI` vs `BATAL`

---

## 8. Task Implementation Detail

## TASK MPO-01

**Nama task:** Define Pesanan and Detail Types  
**Tujuan:** Menetapkan type pesanan, summary pesanan, dan summary detail lintas layer.  
**File yang dibuat/diubah:**  
- `docs/contracts/integration_contract_pack.md`
- `src/types/models.ts`
- `src/types/api.ts`
**Lokasi file:** docs + types  
**Langkah kerja AI agent:**
1. Pastikan shape pesanan dan detail konsisten dengan kontrak query.
2. Pastikan `status_pesanan`, `status_item`, dan `status_kirim` tidak menyimpang.
3. Tambahkan type request create/update/finalisasi.
**Dependency:** `program_workflow.md`, `query_contracts.md`  
**Output yang diharapkan:** kontrak pesanan stabil  
**Acceptance criteria:**
- frontend dan backend memakai shape yang sama

## TASK MPO-02

**Nama task:** Build Pesanan Mock Data Pack  
**Tujuan:** Menyediakan dataset pesanan, watchlist, detail item, dan finalisasi state.  
**File yang dibuat/diubah:**  
- `src/lib/dummy-data/pesanans.ts`
- `src/lib/dummy-data/detail-pesanans.ts`
- `src/lib/dummy-data/pakets.ts`
**Lokasi file:** `src/lib/dummy-data/`  
**Langkah kerja AI agent:**
1. Buat pesanan `AKTIF`, `PERLU_PERHATIAN`, `SELESAI`, `BATAL`.
2. Buat detail item `AKTIF`, `TERHENTI`, `BATAL`.
3. Buat dataset paket aktif untuk flow create dan finalisasi pesanan.
**Dependency:** type pesanan/detail  
**Output yang diharapkan:** dataset mock lengkap  
**Acceptance criteria:**
- semua status utama punya contoh

## TASK MPO-03

**Nama task:** Build Pesanan Mock Hooks  
**Tujuan:** Menyediakan hook list/detail/actions pesanan dan detail item.  
**File yang dibuat/diubah:**  
- `src/hooks/admin/usePesananListMock.ts`
- `src/hooks/admin/usePesananWatchlistMock.ts`
- `src/hooks/admin/useDetailPesananListMock.ts`
- `src/hooks/admin/usePesananActionsMock.ts`
- `src/hooks/reseller/usePesananListMock.ts`
- `src/hooks/reseller/usePesananDetailMock.ts`
- `src/hooks/reseller/usePesananActionsMock.ts`
- `src/hooks/reseller/useFinalisasiPesananMock.ts`
- `src/hooks/reseller/useDetailPesananListMock.ts`
**Lokasi file:** hooks admin + reseller  
**Langkah kerja AI agent:**
1. Pisahkan hook list, detail, watchlist, actions.
2. Simulasikan create pesanan, update detail, dan finalisasi.
3. Simulasikan error kontrak utama.
**Dependency:** dummy data + placeholder API  
**Output yang diharapkan:** layer data mock siap  
**Acceptance criteria:**
- screen tidak membaca dummy data langsung

## TASK MPO-04

**Nama task:** Build Reseller Pesanan Screens  
**Tujuan:** Menyediakan daftar, create, dan detail pesanan berbasis paket untuk reseller.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/reseller/pesanan/page.tsx`
- `src/app/(DashboardLayout)/reseller/pesanan/create/page.tsx`
- `src/app/(DashboardLayout)/reseller/pesanan/[id]/page.tsx`
- `src/app/components/reseller/pesanan/ResellerPesananScreen.tsx`
- `src/app/components/reseller/pesanan/PesananCard.tsx`
- `src/app/components/reseller/pesanan/PesananForm.tsx`
- `src/app/components/reseller/pesanan/PesananDetailScreen.tsx`
**Lokasi file:** reseller pesanan folders  
**Langkah kerja AI agent:**
1. Render daftar pesanan mobile-first.
2. Render form create pesanan dengan item paket awal.
3. Render detail pesanan dengan target, item aktif, dan status finalisasi.
**Dependency:** pesanan hooks + reseller UI contract  
**Output yang diharapkan:** flow pesanan dasar reseller siap  
**Acceptance criteria:**
- create pesanan bisa didemokan
- status pesanan terlihat jelas

## TASK MPO-05

**Nama task:** Build Pesanan Watchlist and Admin Pesanan Screens  
**Tujuan:** Menyediakan daftar pesanan admin dan watchlist perlu perhatian.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/pesanan/page.tsx`
- `src/app/(DashboardLayout)/admin/pesanan/perlu-perhatian/page.tsx`
- `src/app/(DashboardLayout)/admin/pesanan/[id]/page.tsx`
- `src/app/components/admin/pesanan/PesananListScreen.tsx`
- `src/app/components/admin/pesanan/PesananWatchlistScreen.tsx`
- `src/app/components/admin/pesanan/PesananDetailScreen.tsx`
- `src/app/components/admin/pesanan/PesananSummaryCards.tsx`
**Lokasi file:** admin pesanan folders  
**Langkah kerja AI agent:**
1. Buat table/list pesanan admin.
2. Buat watchlist screen.
3. Tampilkan detail pesanan dan summary ringkas.
**Dependency:** admin pesanan hooks  
**Output yang diharapkan:** area pesanan admin siap  
**Acceptance criteria:**
- watchlist bisa dibaca
- detail pesanan tidak membingungkan target berjalan

## TASK MPO-06

**Nama task:** Build Finalisasi Pesanan Flow  
**Tujuan:** Menyediakan flow review detail pesanan lalu finalisasi pesanan.  
**File yang dibuat/diubah:**  
- `src/app/components/reseller/pesanan/FinalisasiPesananFlow.tsx`
- `src/app/components/reseller/pesanan/DetailPesananEditor.tsx`
- `src/app/components/reseller/pesanan/DetailPesananList.tsx`
**Lokasi file:** reseller pesanan folders  
**Langkah kerja AI agent:**
1. Tampilkan item pesanan yang masih dihitung.
2. Tampilkan target berjalan dan total bayar.
3. Tolak submit bila target berjalan < total bayar.
4. Submit finalisasi memakai contract resmi.
**Dependency:** pesanan detail + paket dummy + finalisasi hook  
**Output yang diharapkan:** flow finalisasi bisa didemokan  
**Acceptance criteria:**
- pesanan terisi `tanggal_final`
- error kontrak utama muncul benar

## TASK MPO-07

**Nama task:** Build Detail Pesanan List and Detail Screens  
**Tujuan:** Menyediakan daftar detail pesanan untuk admin dan reseller.  
**File yang dibuat/diubah:**  
- `src/app/components/admin/pesanan/DetailPesananTable.tsx`
- `src/app/components/reseller/pesanan/DetailPesananList.tsx`
**Lokasi file:** admin + reseller pesanan folders  
**Langkah kerja AI agent:**
1. Render daftar item dengan filter status dasar.
2. Tampilkan status kirim dan status item dengan jelas.
3. Pastikan admin dan reseller melihat scope yang sesuai.
**Dependency:** detail hooks  
**Output yang diharapkan:** area item pesanan siap baca  
**Acceptance criteria:**
- reseller tidak melihat aksi admin sensitif
- item `TERHENTI` terbaca jelas

## TASK MPO-08

**Nama task:** Build Pesanan Core Schema  
**Tujuan:** Menetapkan fondasi data `pesanan_konsumen` dan `detail_pesanan_konsumen`.  
**File yang dibuat/diubah:**  
- `supabase/migrations/001_init_schema.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Definisikan tabel header pesanan dan detail item.
2. Tambahkan constraint `UNIQUE (periode_id, konsumen_id)` untuk MVP.
3. Pastikan FK pesanan-detail-konsumen-reseller-periode terkunci jelas.
**Dependency:** `schema_mapping.md`  
**Output yang diharapkan:** schema pesanan tersedia  
**Acceptance criteria:**
- relasi header ke detail tidak ambigu

## TASK MPO-09

**Nama task:** Build Pesanan Read Models  
**Tujuan:** Menyediakan view ringkasan pesanan, watchlist, dan detail item aktif.  
**File yang dibuat/diubah:**  
- `supabase/migrations/004_views_ringkasan.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Bentuk `v_ringkasan_pesanan_konsumen`.
2. Bentuk `v_pesanan_perlu_perhatian`.
3. Bentuk `v_detail_pesanan_aktif`.
**Dependency:** schema pesanan + formula target berjalan  
**Output yang diharapkan:** read model pesanan siap  
**Acceptance criteria:**
- target berjalan tidak perlu dihitung ulang di frontend

## TASK MPO-10

**Nama task:** Build Pesanan SQL Boundary  
**Tujuan:** Membuat boundary create/update/finalisasi/status untuk pesanan dan item.  
**File yang dibuat/diubah:**  
- `supabase/migrations/007_functions_pesanan.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Buat `buat_pesanan_konsumen`.
2. Buat `ubah_detail_pesanan_konsumen`.
3. Buat `finalisasi_pesanan_konsumen`.
4. Buat `ubah_status_detail_pesanan_konsumen`.
**Dependency:** schema + read model  
**Output yang diharapkan:** boundary pesanan siap  
**Acceptance criteria:**
- reseller tidak bisa bypass `finalisasi_pesanan_konsumen`

## TASK MPO-11

**Nama task:** Build Pesanan Validation Layer  
**Tujuan:** Menyediakan validasi input dan state transition pesanan/item.  
**File yang dibuat/diubah:**  
- `src/lib/server/validation/pesanan.validation.ts`
- `src/lib/server/validation/detail-pesanan.validation.ts`
**Lokasi file:** `src/lib/server/validation/`  
**Langkah kerja AI agent:**
1. Validasi create pesanan dan update detail.
2. Validasi finalisasi pesanan.
3. Validasi ubah status item.
**Dependency:** workflow assumptions + contracts  
**Output yang diharapkan:** validasi pesanan terstruktur  
**Acceptance criteria:**
- pesan error selaras dengan kontrak bisnis

## TASK MPO-12

**Nama task:** Build Pesanan Workflow Layer  
**Tujuan:** Menyediakan logic target berjalan, finalisasi, dan status item.  
**File yang dibuat/diubah:**  
- `src/lib/server/workflows/pesanan.workflow.ts`
**Lokasi file:** `src/lib/server/workflows/`  
**Langkah kerja AI agent:**
1. Definisikan `computeTargetBerjalan`.
2. Definisikan guard finalisasi pesanan.
3. Definisikan guard `TERHENTI` vs `BATAL`.
**Dependency:** validation + SQL boundary assumptions  
**Output yang diharapkan:** workflow pesanan siap  
**Acceptance criteria:**
- logic status tidak tercecer di route

## TASK MPO-13

**Nama task:** Build Pesanan Service Layer  
**Tujuan:** Membungkus boundary pesanan dan item ke service rapi.  
**File yang dibuat/diubah:**  
- `src/lib/server/services/pesanan.service.ts`
- `src/lib/server/services/detail-pesanan.service.ts`
**Lokasi file:** `src/lib/server/services/`  
**Langkah kerja AI agent:**
1. Implement operasi header pesanan.
2. Implement operasi item pesanan.
3. Pisahkan mapping error dan response.
**Dependency:** validation + workflow + SQL boundary  
**Output yang diharapkan:** service pesanan siap  
**Acceptance criteria:**
- route tetap tipis

## TASK MPO-14

**Nama task:** Build Pesanan API Routes  
**Tujuan:** Menyediakan route admin dan reseller untuk pesanan.  
**File yang dibuat/diubah:**  
- `src/app/api/admin/pesanan/route.ts`
- `src/app/api/reseller/pesanan/route.ts`
**Lokasi file:** `src/app/api/`  
**Langkah kerja AI agent:**
1. Definisikan method utama per route.
2. Panggil validation dan service yang benar.
3. Gunakan envelope respons standar.
**Dependency:** services pesanan  
**Output yang diharapkan:** route pesanan siap  
**Acceptance criteria:**
- scope actor admin vs reseller tegas

## TASK MPO-15

**Nama task:** Build Audit Coverage for Pesanan  
**Tujuan:** Menyediakan jejak audit pada create/update/finalisasi/status item pesanan.  
**File yang dibuat/diubah:**  
- `supabase/migrations/012_triggers_audit.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Tambahkan audit create pesanan.
2. Tambahkan audit update detail pesanan.
3. Tambahkan audit finalisasi pesanan.
4. Tambahkan audit perubahan status item.
**Dependency:** audit log schema  
**Output yang diharapkan:** perubahan pesanan dapat diaudit  
**Acceptance criteria:**
- aksi sensitif punya jejak audit

---

## 9. Pseudocode dan Flow Penting

## 9.1 Pseudocode Finalisasi Pesanan

```ts
async function finalisasiPesanan(input, actor) {
  const pesanan = await loadPesanan(input.pesanan_konsumen_id);
  ensurePesananBelongsToActor(pesanan, actor);
  ensurePesananCanBeFinalized(pesanan);

  const totalBayar = await getTotalBayar(pesanan.id);
  const targetBerjalan = await computeTargetBerjalan(pesanan.id);

  if (targetBerjalan < totalBayar) {
    throw domainError('PESANAN_TARGET_BELOW_PAYMENT');
  }

  await markPesananFinal(pesanan.id, actor);
  await writeAudit('PESANAN_DIFINALKAN', pesanan.id, actor.id);

  return await loadPesananSummary(pesanan.id);
}
```

## 9.2 Pseudocode Update Detail Pesanan

```ts
async function updateDetailPesanan(input, actor) {
  const pesanan = await loadPesanan(input.pesanan_konsumen_id);
  ensurePesananEditable(pesanan, actor);

  const totalBayar = await getTotalBayar(pesanan.id);
  const selectedPakets = await loadPakets(input.items);
  const totalTargetBaru = computeTargetBaru(selectedPakets, input.items);

  if (totalTargetBaru < totalBayar) {
    throw domainError('PESANAN_TARGET_BELOW_PAYMENT');
  }

  await replaceDetailPesanan(pesanan.id, input.items, input.alasan);
  return await loadPesananSummary(pesanan.id);
}
```

---

## 10. First Working Slice

`First working slice` modul `program_order` adalah versi minimum yang membuka jalur utama dari
konsumen terdaftar menuju pesanan final yang sah.

Isi slice minimum:

- reseller dapat membuat `pesanan_konsumen` dari item paket awal
- reseller dapat melihat daftar pesanan miliknya
- reseller dapat melihat detail pesanan, detail item aktif, dan target berjalan
- reseller dapat memfinalkan pesanan
- reseller dapat melihat daftar `detail_pesanan_konsumen` yang sudah final
- backend memiliki boundary aman untuk create pesanan, update detail pesanan, dan finalisasi

Yang sengaja belum diwajibkan di slice pertama:

- watchlist pesanan penuh dengan semua heuristik
- koreksi item kompleks pasca-final oleh admin
- histori gabungan lintas semua transaksi reseller
- detail admin item pesanan yang lengkap untuk semua kasus audit

Definisi selesai slice pertama:

- satu jalur demo `konsumen ada -> pesanan dibuat -> item disesuaikan -> finalisasi -> item final tampil`
  dapat dijalankan

---

## 11. Task Execution Batches

### batch_a_foundation

- `MPO-01` Define Pesanan and Detail Types
- `MPO-08` Build Pesanan Core Schema

### batch_b_mock_or_read

- `MPO-02` Build Pesanan Mock Data Pack
- `MPO-03` Build Pesanan Mock Hooks
- `MPO-09` Build Pesanan Read Models

### batch_c_primary_actions

- `MPO-04` Build Reseller Pesanan Screens
- `MPO-05` Build Pesanan Watchlist and Admin Pesanan Screens
- `MPO-06` Build Finalisasi Pesanan Flow
- `MPO-07` Build Detail Pesanan List and Detail Screens

### batch_d_backend_boundary

- `MPO-10` Build Pesanan SQL Boundary
- `MPO-11` Build Pesanan Validation Layer
- `MPO-12` Build Pesanan Workflow Layer
- `MPO-13` Build Pesanan Service Layer
- `MPO-14` Build Pesanan API Routes
- `MPO-15` Build Audit Coverage for Pesanan

---

## 12. Rule Khusus AI Coding Agent

- Jangan membuat target tagihan di frontend dari state lokal sebagai sumber kebenaran.
- Jangan membiarkan `detail_pesanan_konsumen` final lahir tanpa `pesanan_konsumen`.
- Jangan menyamakan `status_pesanan` dengan `status_item`.
- Jangan memfinalkan pesanan jika target berjalan lebih kecil dari total bayar.

---

## 13. Urutan Eksekusi Modul

1. definisikan contract pesanan
2. siapkan schema header dan detail
3. siapkan mock data pesanan, item, dan paket
4. siapkan hooks pesanan
5. bangun area reseller pesanan
6. bangun watchlist dan area pesanan admin
7. bangun flow finalisasi
8. bangun daftar item pesanan
9. bangun read model pesanan
10. bangun SQL boundary pesanan
11. bangun validation dan workflow
12. bangun service layer
13. bangun route admin dan reseller
14. tambahkan audit
15. verifikasi jalur create pesanan -> update detail -> finalisasi -> daftar item final

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- frontend menghitung target berjalan sendiri dan menyimpang dari backend
- agent salah memahami bahwa item final bisa lahir tanpa header pesanan
- penanganan `TERHENTI` dan `BATAL` membingungkan jika dicampur terlalu awal

### Asumsi

- satu konsumen hanya punya satu `pesanan_konsumen` aktif per periode pada MVP
- create pesanan aktif dipakai reseller sebagai jalur utama
- paket aktif sudah tersedia dari modul master periodik

### Catatan Keputusan Fase Awal

- create pesanan berpusat di reseller; admin hanya membantu kasus pengecualian
- UI perubahan detail item cukup memakai selection ephemeral tanpa draft persisten
- watchlist `PERLU_PERHATIAN` fase awal cukup visual kandidat
