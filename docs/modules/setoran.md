# Execution Module Setoran
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `setoran`.

Dokumen ini memecah implementasi modul setoran ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/product/prd.md`
- `docs/truth/01-decision_log.md`
- `docs/product/program_workflow.md`
- `docs/contracts/business_contracts.md`
- `docs/contracts/schema_mapping.md`
- `docs/contracts/query_contracts.md`
- `docs/truth/system_maps.md`
- `docs/execution/task_kit_standard.md`
- `docs/modules/program_order.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`
- `docs/ui/reseller_uiux.md`

---

## 1. Tujuan Modul

Modul `setoran` bertanggung jawab untuk:

- mencatat `setoran_konsumen` sebagai layer 1 konsumen ke reseller
- mencatat `setoran` sebagai layer 2 reseller ke pusat
- menampilkan ringkasan pelunasan konsumen
- menampilkan ringkasan pelunasan reseller
- menjaga validasi nominal agar tidak melebihi target berjalan atau saldo terkumpul
- memberi jalur UI mobile-first untuk input setoran cepat

Modul ini adalah pusat pergerakan uang operasional harian.

---

## 2. Scope Modul

### Frontend Mockup Scope

- search konsumen untuk setor cepat
- form setoran konsumen mobile-first
- daftar riwayat setoran konsumen sederhana
- halaman setor pusat reseller
- halaman ringkasan saldo belum disetor
- feedback sukses/gagal yang cepat untuk alur lapangan

Riwayat reseller pada fase awal tetap modular dan tidak bergantung pada satu feed transaksi gabungan lintas domain.

### Backend/Core Scope

- schema `setoran_konsumen` dan `setoran`
- boundary `buat_setoran_konsumen`
- boundary `buat_setoran_pusat`
- read model `v_ringkasan_konsumen` dan `v_ringkasan_reseller` yang siap untuk setoran
- read model khusus `v_monitoring_setoran_konsumen_admin` dan `v_monitoring_setoran_pusat_admin` untuk monitoring admin
- validasi `target_berjalan` pesanan
- validasi `total_dikumpulkan` dan `sisa_setor_pusat` reseller
- update status lunas konsumen dan reseller pada response/read model

---

## 3. Integration Contract Modul Setoran

## 3.1 Setoran Konsumen Request

```ts
type SetoranKonsumenRequest = {
  periode_id: number;
  pesanan_konsumen_id: number;
  konsumen_id: number;
  nominal: number;
  metode: 'CASH' | 'TRANSFER';
  tanggal: string;
  keterangan?: string | null;
};
```

## 3.2 Setoran Pusat Request

```ts
type SetoranPusatRequest = {
  periode_id: number;
  no_reseller: string;
  nominal: number;
  metode: 'CASH' | 'TRANSFER';
  tanggal: string;
  akun_kas_id: number;
  keterangan?: string | null;
};
```

## 3.3 Setoran Konsumen Response

```ts
type SetoranKonsumenResult = {
  id: number;
  periode_id: number;
  pesanan_konsumen_id: number;
  konsumen_id: number;
  no_reseller: string;
  nominal: number;
  metode: 'CASH' | 'TRANSFER';
  tanggal: string;
  sisa_bayar_konsumen: number;
  status_lunas_konsumen: 'BELUM' | 'LUNAS';
};
```

## 3.4 Setoran Pusat Response

```ts
type SetoranPusatResult = {
  id: number;
  periode_id: number;
  no_reseller: string;
  nominal: number;
  metode: 'CASH' | 'TRANSFER';
  tanggal: string;
  akun_kas_id: number;
  saldo_belum_disetor: number;
  sisa_setor_pusat: number;
  status_lunas_reseller: 'BELUM' | 'LUNAS';
};
```

## 3.5 Error Codes Relevan

```txt
SETORAN_INVALID_AMOUNT
SETORAN_PROGRAM_NOT_FOUND
SETORAN_PROGRAM_NOT_ACTIVE
SETORAN_MELEBIHI_TARGET
SETORAN_KONSUMEN_ALREADY_LUNAS
SETORAN_RESELLER_SCOPE_INVALID
SETORAN_PUSAT_MELEBIHI_TARGET
SETORAN_PUSAT_MELEBIHI_SALDO_TERKUMPUL
KEUANGAN_AKUN_KAS_NOT_FOUND
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
```

---

## 4. Struktur Folder Modul Setoran

## 4.1 Frontend

```txt
src/app/(Reseller)/reseller/setor/
  page.tsx
  pusat/
    page.tsx
  riwayat/
    page.tsx

src/app/(Admin)/admin/setoran-konsumen/
  page.tsx

src/app/(Admin)/admin/setoran-pusat/
  page.tsx

src/app/components/reseller/setor/
  ResellerSetorScreen.tsx
  KonsumenSearchList.tsx
  SetoranFormMock.tsx
  QuickNominalChips.tsx
  SetoranPusatScreen.tsx
  SetoranPusatForm.tsx
  RiwayatSetoranScreen.tsx
  index.ts

src/app/components/admin/setoran/
  SetoranKonsumenAdminScreen.tsx
  SetoranPusatAdminScreen.tsx
  SetoranAuditPanel.tsx
  index.ts

src/hooks/reseller/
  useKonsumenSetorListMock.ts
  useSetoranKonsumenMock.ts
  useSetoranPusatMock.ts
  useRiwayatSetoranMock.ts

src/hooks/admin/
  useSetoranKonsumenAdminMock.ts
  useSetoranPusatAdminMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  004_views_ringkasan.sql
  008_functions_setoran.sql
  012_triggers_audit.sql

src/lib/server/services/
  setoran.service.ts

src/lib/server/validation/
  setoran.validation.ts

src/lib/server/workflows/
  setoran.workflow.ts

src/app/api/admin/setoran/
  route.ts

src/app/api/reseller/setor/
  route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `reseller/setor/page.tsx` | Entry route input setoran konsumen |
| `reseller/setor/pusat/page.tsx` | Entry route setor pusat |
| `reseller/setor/riwayat/page.tsx` | Riwayat setoran reseller |
| `admin/setoran-konsumen/page.tsx` | Monitoring admin setoran konsumen |
| `admin/setoran-pusat/page.tsx` | Monitoring admin setor pusat |
| `ResellerSetorScreen.tsx` | Screen utama input setoran konsumen |
| `KonsumenSearchList.tsx` | Search konsumen untuk setor |
| `SetoranFormMock.tsx` | Form setoran konsumen |
| `QuickNominalChips.tsx` | Shortcut nominal cepat |
| `SetoranPusatScreen.tsx` | Screen setor pusat |
| `SetoranPusatForm.tsx` | Form setor pusat |
| `RiwayatSetoranScreen.tsx` | Riwayat transaksi setoran |
| `SetoranKonsumenAdminScreen.tsx` | Monitoring admin layer 1 |
| `SetoranPusatAdminScreen.tsx` | Monitoring admin layer 2 |
| `SetoranAuditPanel.tsx` | Panel ringkas audit/koreksi |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Definisi tabel `setoran_konsumen` dan `setoran` |
| `004_views_ringkasan.sql` | View ringkasan konsumen dan reseller untuk setoran |
| `008_functions_setoran.sql` | Boundary setoran konsumen dan setoran pusat |
| `012_triggers_audit.sql` | Audit perubahan setoran |
| `setoran.service.ts` | Wrapper logic setoran |
| `setoran.validation.ts` | Validasi input setoran |
| `setoran.workflow.ts` | Workflow validasi target dan saldo |
| `admin/setoran/route.ts` | Route admin monitoring setoran berbasis read model khusus |
| `reseller/setor/route.ts` | Route reseller untuk setor |

---

## 6. Hubungan Antar File

### Frontend Flow Setoran Konsumen

```txt
reseller/setor/page.tsx
-> ResellerSetorScreen
-> KonsumenSearchList
-> SetoranFormMock
-> useSetoranKonsumenMock
```

### Frontend Flow Setor Pusat

```txt
reseller/setor/pusat/page.tsx
-> SetoranPusatScreen
-> SetoranPusatForm
-> useSetoranPusatMock
```

### Backend Flow

```txt
route.ts
-> setoran.validation.ts
-> setoran.service.ts
-> setoran.workflow.ts
-> SQL boundary
-> response envelope
```

### Integration Touchpoints

- layer 1 selalu bergantung pada `pesanan_konsumen_id`
- layer 2 selalu bergantung pada `no_reseller` dan `periode_id`
- kedua layer mengembalikan sisa/status lunas yang siap dibaca UI

---

## 7. Breakdown Function dan Component per File

## 7.1 `SetoranFormMock.tsx`

Komponen:

- `SetoranFormMock`

Tanggung jawab:

- render nominal, metode, tanggal, catatan
- menampilkan target, total bayar, sisa bayar
- submit setoran konsumen mock

## 7.2 `SetoranPusatForm.tsx`

Komponen:

- `SetoranPusatForm`

Tanggung jawab:

- render nominal setor pusat
- render akun kas tujuan
- menampilkan saldo belum disetor dan sisa setor pusat
- submit setor pusat mock

## 7.3 `setoran.workflow.ts`

Functions:

- `ensureProgramCanReceivePayment`
- `ensurePaymentDoesNotExceedTargetBerjalan`
- `ensureResellerCanSetorPusat`
- `ensureSetoranPusatDoesNotExceedSaldoTerkumpul`
- `recomputeProgramAndResellerSummary`

Tanggung jawab:

- menjaga dua layer setoran tetap konsisten
- menjaga validasi sisa dan status lunas
- menjaga recalculation summary setelah transaksi

---

## 8. Task Implementation Detail

## TASK MST-01

**Nama task:** Define Setoran Module Types  
**Tujuan:** Menetapkan contract request/response untuk setoran konsumen dan pusat.  
**File yang dibuat/diubah:**  
- `docs/contracts/integration_contract_pack.md`
- `src/types/models.ts`
- `src/types/api.ts`
**Lokasi file:** docs + types  
**Langkah kerja AI agent:**
1. Pastikan shape request/response setoran konsumen dan pusat konsisten.
2. Pastikan field uang tetap `number`.
3. Pastikan status lunas mengikuti kontrak resmi.
**Dependency:** `business_contracts.md`, `query_contracts.md`  
**Output yang diharapkan:** kontrak setoran stabil  
**Acceptance criteria:**
- frontend dan backend bicara dalam shape yang sama

## TASK MST-02

**Nama task:** Build Setoran Mock Data Pack  
**Tujuan:** Menyediakan data mock transaksi setoran layer 1 dan layer 2.  
**File yang dibuat/diubah:**  
- `src/lib/dummy-data/setoran.ts`
**Lokasi file:** `src/lib/dummy-data/`  
**Langkah kerja AI agent:**
1. Buat riwayat setoran konsumen sebagian dan lunas.
2. Buat riwayat setor pusat sebagian dan lunas reseller.
3. Tambahkan skenario melebihi target dan melebihi saldo terkumpul.
**Dependency:** pesanan/detail summary data  
**Output yang diharapkan:** dataset mock setoran lengkap  
**Acceptance criteria:**
- skenario gagal dan sukses tersedia

## TASK MST-03

**Nama task:** Build Setoran Mock Hooks  
**Tujuan:** Menyediakan hook mock untuk input dan monitoring setoran.  
**File yang dibuat/diubah:**  
- `src/hooks/reseller/useKonsumenSetorListMock.ts`
- `src/hooks/reseller/useSetoranKonsumenMock.ts`
- `src/hooks/reseller/useSetoranPusatMock.ts`
- `src/hooks/reseller/useRiwayatSetoranMock.ts`
- `src/hooks/admin/useSetoranKonsumenAdminMock.ts`
- `src/hooks/admin/useSetoranPusatAdminMock.ts`
**Lokasi file:** hooks admin + reseller  
**Langkah kerja AI agent:**
1. Pisahkan list calon setor, submit layer 1, submit layer 2, riwayat.
2. Simulasikan response sukses dan error code utama.
3. Simulasikan refetch ringkasan setelah transaksi.
**Dependency:** dummy data + placeholder API  
**Output yang diharapkan:** layer data mock setoran siap  
**Acceptance criteria:**
- screen tidak membaca data setoran langsung

## TASK MST-04

**Nama task:** Build Reseller Setoran Konsumen Screen  
**Tujuan:** Menyediakan flow 3 langkah setoran konsumen dari area reseller.  
**File yang dibuat/diubah:**  
- `src/app/(Reseller)/reseller/setor/page.tsx`
- `src/app/components/reseller/setor/ResellerSetorScreen.tsx`
- `src/app/components/reseller/setor/KonsumenSearchList.tsx`
- `src/app/components/reseller/setor/SetoranFormMock.tsx`
- `src/app/components/reseller/setor/QuickNominalChips.tsx`
**Lokasi file:** reseller setor folders  
**Langkah kerja AI agent:**
1. Render search konsumen sticky.
2. Render daftar konsumen prioritas.
3. Render form nominal cepat.
4. Simulasikan toast sukses dan error inline.
**Dependency:** konsumen/pesanan summary + setoran hooks  
**Output yang diharapkan:** flow setoran konsumen cepat siap  
**Acceptance criteria:**
- maksimal 3 langkah dari list ke simpan
- tidak ada horizontal scroll

## TASK MST-05

**Nama task:** Build Reseller Setor Pusat Screen  
**Tujuan:** Menyediakan flow setor pusat untuk reseller.  
**File yang dibuat/diubah:**  
- `src/app/(Reseller)/reseller/setor/pusat/page.tsx`
- `src/app/components/reseller/setor/SetoranPusatScreen.tsx`
- `src/app/components/reseller/setor/SetoranPusatForm.tsx`
**Lokasi file:** reseller setor pusat folders  
**Langkah kerja AI agent:**
1. Tampilkan `total_dikumpulkan`, `total_disetor_pusat`, `saldo_belum_disetor`, `sisa_setor_pusat`.
2. Render field akun kas tujuan.
3. Simulasikan submit setor pusat.
**Dependency:** reseller summary + setoran pusat hooks  
**Output yang diharapkan:** flow setor pusat siap  
**Acceptance criteria:**
- reseller paham sisa setor sebelum submit

## TASK MST-06

**Nama task:** Build Riwayat Setoran Screen  
**Tujuan:** Menyediakan riwayat setoran untuk reseller.  
**File yang dibuat/diubah:**  
- `src/app/(Reseller)/reseller/setor/riwayat/page.tsx`
- `src/app/components/reseller/setor/RiwayatSetoranScreen.tsx`
**Lokasi file:** reseller riwayat folders  
**Langkah kerja AI agent:**
1. Tampilkan riwayat setoran konsumen.
2. Tampilkan riwayat setor pusat.
3. Tambahkan filter sederhana.
**Dependency:** riwayat hooks  
**Output yang diharapkan:** riwayat setoran terbaca  
**Acceptance criteria:**
- transaksi terakhir mudah dicek
- layar riwayat tidak bergantung pada unified reseller history feed

## TASK MST-07

**Nama task:** Build Admin Setoran Monitoring Screens  
**Tujuan:** Menyediakan monitoring admin untuk layer 1 dan layer 2.  
**File yang dibuat/diubah:**  
- `src/app/(Admin)/admin/setoran-konsumen/page.tsx`
- `src/app/(Admin)/admin/setoran-pusat/page.tsx`
- `src/app/components/admin/setoran/SetoranKonsumenAdminScreen.tsx`
- `src/app/components/admin/setoran/SetoranPusatAdminScreen.tsx`
- `src/app/components/admin/setoran/SetoranAuditPanel.tsx`
**Lokasi file:** admin setoran folders  
**Langkah kerja AI agent:**
1. Tampilkan list transaksi layer 1.
2. Tampilkan list transaksi layer 2.
3. Tambahkan panel audit ringkas.
**Dependency:** admin hooks  
**Output yang diharapkan:** monitoring admin siap  
**Acceptance criteria:**
- admin bisa membedakan dua layer dengan jelas

## TASK MST-08

**Nama task:** Build Setoran Core Schema  
**Tujuan:** Menetapkan fondasi data `setoran_konsumen` dan `setoran`.  
**File yang dibuat/diubah:**  
- `supabase/migrations/001_init_schema.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Definisikan tabel setoran layer 1 dan layer 2.
2. Pastikan FK pesanan/konsumen/reseller/periode jelas.
3. Tambahkan check nominal > 0.
**Dependency:** `schema_mapping.md`  
**Output yang diharapkan:** schema setoran tersedia  
**Acceptance criteria:**
- dua layer transaksi terpisah jelas

## TASK MST-09

**Nama task:** Build Setoran Read Models  
**Tujuan:** Menyediakan read model yang siap dipakai untuk input dan ringkasan setoran.  
**File yang dibuat/diubah:**  
- `supabase/migrations/004_views_ringkasan.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Pastikan `v_ringkasan_konsumen` siap untuk flow setor cepat.
2. Pastikan `v_ringkasan_pesanan_konsumen` siap untuk detail pesanan.
3. Pastikan `v_ringkasan_reseller` siap untuk setor pusat.
**Dependency:** schema setoran + pesanan/detail read models  
**Output yang diharapkan:** read model setoran siap  
**Acceptance criteria:**
- frontend tidak perlu menghitung status lunas sendiri

## TASK MST-10

**Nama task:** Build Setoran SQL Boundary  
**Tujuan:** Membuat boundary setoran konsumen dan setor pusat.  
**File yang dibuat/diubah:**  
- `supabase/migrations/008_functions_setoran.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Buat `buat_setoran_konsumen`.
2. Buat `buat_setoran_pusat`.
3. Pastikan output mengembalikan sisa dan status lunas.
**Dependency:** schema + read models  
**Output yang diharapkan:** boundary setoran siap  
**Acceptance criteria:**
- validation final hidup di boundary

## TASK MST-11

**Nama task:** Build Setoran Validation Layer  
**Tujuan:** Menyediakan validasi input dua layer setoran.  
**File yang dibuat/diubah:**  
- `src/lib/server/validation/setoran.validation.ts`
**Lokasi file:** `src/lib/server/validation/`  
**Langkah kerja AI agent:**
1. Validasi setoran konsumen.
2. Validasi setor pusat.
3. Definisikan pesan error domain.
**Dependency:** contracts + workflow assumptions  
**Output yang diharapkan:** validasi setoran terstruktur  
**Acceptance criteria:**
- route tidak memegang validasi detail

## TASK MST-12

**Nama task:** Build Setoran Workflow Layer  
**Tujuan:** Menyediakan workflow target berjalan, saldo terkumpul, dan pelunasan.  
**File yang dibuat/diubah:**  
- `src/lib/server/workflows/setoran.workflow.ts`
**Lokasi file:** `src/lib/server/workflows/`  
**Langkah kerja AI agent:**
1. Validasi target berjalan pesanan.
2. Validasi saldo terkumpul reseller.
3. Recompute summary pesanan dan reseller setelah transaksi.
**Dependency:** program_order boundary + read models  
**Output yang diharapkan:** workflow setoran siap  
**Acceptance criteria:**
- status lunas konsumen dan reseller konsisten

## TASK MST-13

**Nama task:** Build Setoran Service Layer  
**Tujuan:** Membungkus boundary setoran ke service rapi.  
**File yang dibuat/diubah:**  
- `src/lib/server/services/setoran.service.ts`
**Lokasi file:** `src/lib/server/services/`  
**Langkah kerja AI agent:**
1. Implement operasi layer 1 dan layer 2.
2. Mapping error dan response.
3. Jaga actor scope reseller/admin.
**Dependency:** validation + workflow + boundary  
**Output yang diharapkan:** service setoran siap  
**Acceptance criteria:**
- service bisa dipakai route tanpa logic liar

## TASK MST-14

**Nama task:** Build Setoran API Routes  
**Tujuan:** Menyediakan route reseller dan admin untuk setoran.  
**File yang dibuat/diubah:**  
- `src/app/api/reseller/setor/route.ts`
- `src/app/api/admin/setoran/route.ts`
**Lokasi file:** `src/app/api/`  
**Langkah kerja AI agent:**
1. Definisikan method utama route reseller.
2. Definisikan monitoring route admin.
3. Gunakan response envelope standar.
**Dependency:** service setoran  
**Output yang diharapkan:** route setoran siap  
**Acceptance criteria:**
- dua layer setoran tidak tercampur

## TASK MST-15

**Nama task:** Build Audit Coverage for Setoran  
**Tujuan:** Menyediakan jejak audit transaksi setoran.  
**File yang dibuat/diubah:**  
- `supabase/migrations/012_triggers_audit.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Tambahkan audit setoran konsumen.
2. Tambahkan audit setor pusat.
3. Siapkan kaitan ke koreksi transaksi nanti.
**Dependency:** audit log schema  
**Output yang diharapkan:** setoran dapat diaudit  
**Acceptance criteria:**
- layer 1 dan layer 2 sama-sama punya jejak

---

## 9. Pseudocode dan Flow Penting

## 9.1 Pseudocode Setoran Konsumen

```ts
async function createSetoranKonsumen(input, actor) {
  const pesanan = await loadPesanan(input.pesanan_konsumen_id);
  ensurePesananBelongsToActor(pesanan, actor);
  ensurePesananCanReceivePayment(pesanan);

  const targetBerjalan = await computeTargetBerjalan(pesanan.id);
  const totalBayar = await getTotalBayar(pesanan.id);

  if (totalBayar + input.nominal > targetBerjalan) {
    throw domainError('SETORAN_MELEBIHI_TARGET');
  }

  const trx = await insertSetoranKonsumen(input, actor);
  await recomputePesananAndResellerSummary(pesanan.id, pesanan.no_reseller, pesanan.periode_id);
  return trx;
}
```

## 9.2 Pseudocode Setor Pusat

```ts
async function createSetoranPusat(input, actor) {
  ensureResellerScopeAllowed(input.no_reseller, actor);

  const summary = await getResellerSummary(input.periode_id, input.no_reseller);
  if (summary.total_disetor_pusat + input.nominal > summary.nilai_akhir_paket) {
    throw domainError('SETORAN_PUSAT_MELEBIHI_TARGET');
  }
  if (summary.total_disetor_pusat + input.nominal > summary.total_dikumpulkan) {
    throw domainError('SETORAN_PUSAT_MELEBIHI_SALDO_TERKUMPUL');
  }

  const trx = await insertSetoranPusat(input, actor);
  await recomputeResellerSummary(input.no_reseller, input.periode_id);
  return trx;
}
```

---

## 10. First Working Slice

`First working slice` modul `setoran` adalah versi minimum yang sudah memungkinkan reseller melakukan transaksi harian paling penting dengan aman.

Isi slice minimum:

- reseller dapat mencari konsumen dan melihat sisa bayar
- reseller dapat menyimpan setoran konsumen
- reseller dapat melihat perubahan sisa bayar dan status lunas
- reseller dapat membuka halaman setor pusat
- reseller dapat menyimpan setor pusat bila saldo mencukupi
- backend memiliki boundary aman untuk layer 1 dan layer 2

Yang sengaja belum diwajibkan di slice pertama:

- riwayat lengkap multi-filter
- monitoring admin yang sangat kaya
- koreksi transaksi setoran
- integrasi audit panel yang penuh

Definisi selesai slice pertama:

- satu jalur demo `pilih konsumen -> input setoran -> status lunas berubah -> setor pusat -> saldo belum disetor berkurang` dapat dijalankan

---

## 11. Task Execution Batches

### batch_a_foundation

- `MST-01` Define Setoran Module Types
- `MST-08` Build Setoran Core Schema

### batch_b_mock_or_read

- `MST-02` Build Setoran Mock Data Pack
- `MST-03` Build Setoran Mock Hooks
- `MST-09` Build Setoran Read Models

### batch_c_primary_actions

- `MST-04` Build Reseller Setoran Konsumen Screen
- `MST-05` Build Reseller Setor Pusat Screen
- `MST-06` Build Riwayat Setoran Screen
- `MST-07` Build Admin Setoran Monitoring Screens

### batch_d_backend_boundary

- `MST-10` Build Setoran SQL Boundary
- `MST-11` Build Setoran Validation Layer
- `MST-12` Build Setoran Workflow Layer
- `MST-13` Build Setoran Service Layer
- `MST-14` Build Setoran API Routes
- `MST-15` Build Audit Coverage for Setoran

---

## 12. Rule Khusus AI Coding Agent

- Jangan izinkan setoran konsumen tanpa `pesanan_konsumen_id`.
- Jangan izinkan setor pusat melebihi `total_dikumpulkan`.
- Jangan hitung status lunas final di UI.
- Jangan campur layar setoran konsumen dan setor pusat tanpa label layer yang jelas.

---

## 13. Urutan Eksekusi Modul

1. definisikan contract setoran
2. siapkan schema setoran dua layer
3. siapkan dummy data dan hooks
4. bangun screen setor konsumen reseller
5. bangun screen setor pusat reseller
6. bangun riwayat setoran
7. bangun monitoring admin
8. bangun read models setoran
9. bangun SQL boundary setoran
10. bangun validation dan workflow
11. bangun service layer
12. bangun route reseller dan admin
13. tambahkan audit
14. verifikasi jalur setor konsumen -> setor pusat

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- agent salah memahami bahwa layer 1 dan layer 2 punya validasi yang sama
- ringkasan reseller tidak sinkron jika recompute sesudah transaksi tidak konsisten
- UI terlalu lambat jika daftar calon setor tidak dioptimalkan dari read model

### Asumsi

- modul `program_order` sudah menyediakan pesanan aktif yang valid untuk menerima setoran
- route reseller tetap memakai scope `profile.no_reseller`
- akun kas tujuan untuk setor pusat tersedia dari modul master

### Catatan Keputusan Fase Awal

- riwayat setoran awal cukup satu daftar campuran berlabel
- admin pada fase awal cukup monitoring setoran; input koreksi setoran belum tersedia pada fase awal
- metode pembayaran awal cukup `CASH` dan `TRANSFER`
