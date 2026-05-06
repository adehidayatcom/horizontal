# ENGINE / BACKEND / CORE LOGIC PLAN
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint teknis backend dan core logic untuk AI coding agent.

Scope dokumen ini mencakup:

- database schema
- API endpoints
- service layer
- business logic
- validation
- background process future-ready
- workflow orchestration sederhana
- struktur folder dan file backend

Dokumen ini tidak menjadi tempat untuk detail UI.

Kontrak lintas layer yang wajib dijaga selama implementasi plan ini:

- `docs/contracts/integration_contract_pack.md`

---

## 1. Tujuan Plan

Tujuan backend/core logic adalah membangun fondasi sistem yang:

- aman secara data
- konsisten secara kontrak
- siap dikonsumsi frontend
- memusatkan logic bisnis final di boundary yang tepat
- mendukung audit, koreksi, dan kontrol role

---

## 2. Aturan Umum untuk AI Coding Agent

### Aturan Scope

- Business rule final tidak diletakkan di frontend.
- Perubahan uang, stok, status, dan audit harus lewat boundary resmi.
- Semua struktur data harus tunduk ke kontrak domain.
- Semua operasi sensitif harus sadar role dan periode.

### Aturan Data

- Gunakan Supabase sebagai fondasi data.
- Jadikan migration SQL sebagai sumber perubahan schema.
- Hindari skema yang sulit diuji ulang.

### Aturan Service Layer

- Service layer adalah wrapper terstruktur terhadap Supabase boundary.
- Jangan biarkan page frontend memanggil query liar tanpa service/hook layer yang jelas.

### Aturan Validation

- Validation dibagi antara:
  - input validation
  - business validation
  - access validation
  - consistency validation

---

## 3. Struktur Folder Backend

Struktur target backend/core logic:

```txt
supabase/
  migrations/
    001_init_schema.sql
    002_rls_policies.sql
    003_seed_reference_data.sql
    004_views_ringkasan.sql
    005_functions_periode.sql
    006_functions_reseller.sql
    007_functions_pesanan.sql
    008_functions_setoran.sql
    009_functions_gudang.sql
    010_functions_keuangan.sql
    011_functions_dashboard_reports.sql
    012_triggers_audit.sql
  seed.sql

src/
  lib/
    supabase/
      client.ts
      server.ts
      admin.ts
    server/
      auth/
        auth-session.ts
        auth-guards.ts
        profile-access.ts
      services/
        auth.service.ts
        master-periodik.service.ts
        periode.service.ts
        reseller.service.ts
        pesanan.service.ts
        detail-pesanan.service.ts
        setoran.service.ts
        gudang.service.ts
        keuangan.service.ts
        dashboard-report.service.ts
      validation/
        auth.validation.ts
        master-periodik.validation.ts
        periode.validation.ts
        reseller.validation.ts
        pesanan.validation.ts
        detail-pesanan.validation.ts
        setoran.validation.ts
        gudang.validation.ts
        keuangan.validation.ts
        dashboard-report.validation.ts
      workflows/
        auth.workflow.ts
        master-periodik.workflow.ts
        periode.workflow.ts
        pesanan.workflow.ts
        setoran.workflow.ts
        gudang.workflow.ts
        pencairan.workflow.ts
        dashboard-report.workflow.ts
      background/
        audit-sync.job.ts
        period-check.job.ts
        watchlist-refresh.job.ts
      utils/
        errors.ts
        money.ts
        status.ts
        dates.ts
      contracts/
        actions.ts
        responses.ts
        events.ts

  app/
    api/
      auth/
        login/route.ts
        logout/route.ts
        session/route.ts
      admin/
        periode/route.ts
        reseller/route.ts
        pesanan/route.ts
        setoran/route.ts
        master/
          akun-kas/route.ts
          barang/route.ts
          barang-periode/route.ts
          komisi/route.ts
          paket/route.ts
        reseller-periode/route.ts
        gudang/
          belanja/route.ts
          packing/route.ts
          pengiriman/route.ts
          pembagian/route.ts
        keuangan/
          akun-kas/route.ts
          saldo-kas/route.ts
          kas-masuk/route.ts
          mutasi-kas/route.ts
          pencairan/route.ts
        laporan/
          rekap-reseller/route.ts
          stok/route.ts
          pengiriman/route.ts
          audit/route.ts
      reseller/
        dashboard/route.ts
        konsumen/route.ts
        pesanan/route.ts
        setor/route.ts
        akun/route.ts
      system/
        health/route.ts
        jobs/route.ts

  types/
    database.ts
    server.ts
    contracts.ts
```

---

## 4. Struktur File Backend dan Fungsi Setiap File

## 4.1 Migration Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Membuat schema inti |
| `002_rls_policies.sql` | Mengaktifkan dan mengatur RLS |
| `003_seed_reference_data.sql` | Seed reference data minimum |
| `004_views_ringkasan.sql` | Membuat read model/view inti |
| `005_functions_periode.sql` | Boundary periode (termasuk RPC `clone_paket_periode_lalu` & `mass_replace_item_paket`) |
| `006_functions_reseller.sql` | Boundary reseller |
| `007_functions_pesanan.sql` | Boundary pesanan dan detail pesanan |
| `008_functions_setoran.sql` | Boundary setoran |
| `009_functions_gudang.sql` | Boundary belanja, packing, pembagian, pengiriman |
| `010_functions_keuangan.sql` | Boundary kas, mutasi, pencairan |
| `011_functions_dashboard_reports.sql` | Query/report boundary |
| `012_triggers_audit.sql` | Trigger audit dan perubahan status terkait |

## 4.2 Service Layer Files

| File | Fungsi |
|---|---|
| `auth.service.ts` | Wrapper auth session, profile, dan approval state |
| `master-periodik.service.ts` | Wrapper master lintas periode dan per-periode (termasuk `massReplacePaketItem`) |
| `periode.service.ts` | Wrapper logic periode (termasuk `clonePaketPeriodeLalu`) |
| `reseller.service.ts` | Wrapper logic reseller |
| `pesanan.service.ts` | Wrapper logic header pesanan konsumen |
| `detail-pesanan.service.ts` | Wrapper logic detail item pesanan |
| `setoran.service.ts` | Wrapper logic setoran |
| `gudang.service.ts` | Wrapper logic gudang |
| `keuangan.service.ts` | Wrapper logic keuangan |
| `dashboard-report.service.ts` | Wrapper read/report dashboard dan laporan |

## 4.3 Validation Files

| File | Fungsi |
|---|---|
| `auth.validation.ts` | Validasi login, session, dan approval state |
| `master-periodik.validation.ts` | Validasi master lintas periode dan periodik |
| `periode.validation.ts` | Validasi input periode |
| `reseller.validation.ts` | Validasi input reseller |
| `pesanan.validation.ts` | Validasi input pesanan |
| `detail-pesanan.validation.ts` | Validasi input detail item pesanan |
| `setoran.validation.ts` | Validasi input setoran |
| `gudang.validation.ts` | Validasi input gudang |
| `keuangan.validation.ts` | Validasi input keuangan |
| `dashboard-report.validation.ts` | Validasi filter dan query param dashboard/laporan |

## 4.4 Workflow Files

| File | Fungsi |
|---|---|
| `auth.workflow.ts` | Orkestrasi login, pending approval, dan logout |
| `master-periodik.workflow.ts` | Orkestrasi warning perubahan master dan readiness periode |
| `periode.workflow.ts` | Orkestrasi status periode dan readiness |
| `pesanan.workflow.ts` | Flow target berjalan, finalisasi pesanan, status item, pengiriman |
| `setoran.workflow.ts` | Flow layer setoran dan pelunasan |
| `gudang.workflow.ts` | Flow belanja, packing, pembagian |
| `pencairan.workflow.ts` | Flow pencairan komisi/tabungan |
| `dashboard-report.workflow.ts` | Orkestrasi payload dashboard dan laporan agar frontend tetap tipis |

## 4.5 Background Files

| File | Fungsi |
|---|---|
| `audit-sync.job.ts` | Jalur ekstensi bila nanti dibutuhkan agregasi audit non-blocking |
| `period-check.job.ts` | Jalur ekstensi untuk pemeriksaan periode terjadwal di fase lanjutan |
| `watchlist-refresh.job.ts` | Jalur ekstensi jika watchlist kelak perlu precompute, bukan deliverable wajib first wave |

## 4.6 API Route Files

| File | Fungsi |
|---|---|
| `auth/login/route.ts` | Endpoint login server-side |
| `auth/logout/route.ts` | Endpoint logout aman |
| `auth/session/route.ts` | Endpoint baca session dan approval state |
| `admin/periode/route.ts` | Endpoint admin periode |
| `admin/reseller/route.ts` | Endpoint admin reseller |
| `admin/master/akun-kas/route.ts` | Endpoint admin akun kas |
| `admin/master/barang/route.ts` | Endpoint admin barang |
| `admin/master/barang-periode/route.ts` | Endpoint admin barang per periode |
| `admin/master/komisi/route.ts` | Endpoint admin komisi periodik |
| `admin/master/paket/route.ts` | Endpoint admin paket dan BOM |
| `admin/master/paket/clone/route.ts` | Endpoint kloning masal paket dari periode lalu |
| `admin/master/paket/mass-replace/route.ts` | Endpoint mass replace komponen paket |
| `admin/reseller-periode/route.ts` | Endpoint admin assignment reseller periode |
| `admin/pesanan/route.ts` | Endpoint admin pesanan |
| `admin/setoran/route.ts` | Endpoint admin setoran |
| `admin/gudang/belanja/route.ts` | Endpoint admin belanja |
| `admin/gudang/packing/route.ts` | Endpoint admin packing |
| `admin/gudang/pengiriman/route.ts` | Endpoint admin pengiriman |
| `admin/gudang/pembagian/route.ts` | Endpoint admin pembagian |
| `admin/keuangan/akun-kas/route.ts` | Endpoint admin akun kas dan saldo kas |
| `admin/keuangan/saldo-kas/route.ts` | Endpoint admin saldo kas |
| `admin/keuangan/kas-masuk/route.ts` | Endpoint admin kas masuk |
| `admin/keuangan/mutasi-kas/route.ts` | Endpoint admin mutasi kas |
| `admin/keuangan/pencairan/route.ts` | Endpoint admin pencairan |
| `admin/laporan/rekap-reseller/route.ts` | Endpoint admin laporan rekap reseller |
| `admin/laporan/stok/route.ts` | Endpoint admin laporan stok |
| `admin/laporan/pengiriman/route.ts` | Endpoint admin laporan pengiriman |
| `admin/laporan/audit/route.ts` | Endpoint admin laporan audit koreksi |
| `reseller/dashboard/route.ts` | Endpoint reseller dashboard |
| `reseller/konsumen/route.ts` | Endpoint reseller konsumen |
| `reseller/pesanan/route.ts` | Endpoint reseller pesanan |
| `reseller/setor/route.ts` | Endpoint reseller setor |
| `reseller/akun/route.ts` | Endpoint reseller akun |

---

## 5. Hubungan Antar File

Relasi utama:

```txt
API Route
-> validation
-> service
-> workflow/helper
-> Supabase client
-> SQL boundary (RPC/view/table)
```

Untuk background process yang benar-benar dipakai setelah fase awal:

```txt
job
-> service/workflow
-> Supabase boundary
-> audit/log/result
```

Contoh:

- `admin/periode/route.ts`
  -> `periode.validation.ts`
  -> `periode.service.ts`
  -> `periode.workflow.ts`
  -> `005_functions_periode.sql`

- `reseller/setor/route.ts`
  -> `setoran.validation.ts`
  -> `setoran.service.ts`
  -> `setoran.workflow.ts`
  -> `008_functions_setoran.sql`

---

## 6. Breakdown Function, Class, dan Boundary per File

## 6.1 `periode.service.ts`

Functions:

- `createPeriode`
- `updatePeriodeStatus`
- `getPeriodeList`
- `getPeriodeDetail`
- `getPeriodeChecklist`

Tanggung jawab:

- memanggil boundary periode
- memastikan response terstruktur
- menerjemahkan error teknis ke error domain

## 6.2 `setoran.service.ts`

Functions:

- `createSetoranKonsumen`
- `createSetoranPusat`
- `getRingkasanSetoranReseller`
- `getRiwayatSetoranKonsumen`
- `getRiwayatSetoranPusat`

Tanggung jawab:

- membungkus transaksi setoran
- memberi response siap konsumsi route

## 6.3 `setoran.workflow.ts`

Functions:

- `validateSetoranAgainstTarget`
- `recomputePesananPaymentSummary`
- `recomputeResellerPaymentSummary`
- `buildSetoranSuccessPayload`

Tanggung jawab:

- orkestrasi logic flow setoran
- sinkronisasi state turunan

## 6.4 `010_functions_keuangan.sql`

Boundary:

- `buat_kas_masuk`
- `buat_mutasi_kas`
- `buat_pencairan_tabungan`
- `buat_pencairan_komisi`
- `get_posisi_kas`

Tanggung jawab:

- menjaga logic finansial tetap di boundary resmi

---

## 7. Database Schema Detail

Entitas inti:

- `profile`
- `periode`
- `reseller`
- `konsumen`
- `pesanan_konsumen`
- `detail_pesanan_konsumen`
- `paket`
- `detail_paket`
- `barang`
- `barang_periode`
- `setoran_konsumen`
- `setoran`
- `belanja`
- `belanja_detail`
- `packing`
- `pembagian_paket`
- `pembagian_paket_detail`
- `akun_kas`
- `kas_masuk`
- `mutasi_kas`
- `pencairan`
- `audit_log`

Prinsip schema:

- setiap entitas penting punya penanda waktu
- tabel transaksi penting sadar `periode_id`
- status penting ditopang enum atau constraint
- audit tersedia untuk tindakan penting

---

## 8. API Endpoint Design

Contoh endpoint:

```txt
POST /api/admin/periode
PATCH /api/admin/periode
GET /api/admin/periode
POST /api/admin/master/paket/clone
POST /api/admin/master/paket/mass-replace

GET /api/admin/pesanan
POST /api/admin/gudang
POST /api/admin/keuangan

GET /api/reseller/pesanan
POST /api/reseller/pesanan
POST /api/reseller/setor
GET /api/reseller/dashboard
```

Response shape:

```ts
type ApiSuccess<T> = {
  success: true;
  data: T;
  meta?: Record<string, unknown>;
};

type ApiFailure = {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
  };
};
```

---

## 9. Business Logic Scope

Business logic yang harus hidup di engine/backend:

- validasi status periode
- validasi target berjalan
- validasi pelunasan
- validasi item batal vs aktif vs terhenti
- validasi saldo belum disetor
- validasi stok, packing, dan pembagian
- validasi pencairan
- pembentukan watchlist operasional

Jangan menaruh logic ini sebagai final source di frontend.

---

## 10. Validation Scope

Validation dibagi menjadi empat:

### Input Validation

- tipe data
- field wajib
- numeric positif
- enum status valid

### Access Validation

- role admin/reseller
- akses reseller terhadap datanya sendiri

### Business Validation

- target tidak terlampaui
- status tidak melompat liar
- stok cukup
- periode sesuai

### Consistency Validation

- entity terkait benar-benar ada
- item `BATAL` dikecualikan dari formula relevan
- transaksi tidak membuat turunan data inkonsisten

---

## 11. Background Process dan Workflow Engine

Pada fase coding awal, summary dashboard dan watchlist dihitung on-demand melalui route -> service -> workflow -> SQL boundary.

Background process hanya disiapkan sebagai abstraksi future-ready untuk fase lanjutan, misalnya:

- refresh ringkasan watchlist jika query on-demand nanti terbukti berat
- pemeriksaan periode mendekati penutupan jika butuh notifikasi terjadwal
- sinkronisasi audit atau agregat tambahan yang tidak wajib untuk first wave

Istilah workflow pada konteks ini berarti orkestrasi proses, bukan model AI generatif.

Contoh workflow:

```txt
Setoran Konsumen
-> validasi input
-> validasi akses
-> validasi target berjalan
-> tulis transaksi
-> hitung ulang ringkasan pesanan
-> hitung ulang ringkasan reseller
-> tulis audit
-> return payload sukses
```

---

## 12. Task Implementation Detail

## TASK B-01

**Nama task:** Build Initial Database Schema  
**Tujuan:** Membuat tabel inti, relasi, enum, dan constraint dasar.  
**File yang dibuat/diubah:**  
- `supabase/migrations/001_init_schema.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Definisikan enum status.
2. Definisikan tabel master.
3. Definisikan tabel transaksi.
4. Tambahkan FK dan constraint.
5. Tambahkan index minimum.
**Dependency:** `docs/contracts/schema_mapping.md`  
**Output yang diharapkan:** schema inti siap di-migrate  
**Acceptance criteria:**
- migration dapat dibaca dan ditelusuri jelas
- relasi utama terpenuhi
- status penting memiliki guard struktural

## TASK B-02

**Nama task:** Implement Access Control Layer  
**Tujuan:** Mengaktifkan RLS dan policy sesuai role.  
**File yang dibuat/diubah:**  
- `supabase/migrations/002_rls_policies.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Aktifkan RLS pada tabel operasional.
2. Buat policy admin.
3. Buat policy reseller.
4. Pastikan access scope reseller aman.
**Dependency:** schema inti + `docs/contracts/rls_matrix.md`  
**Output yang diharapkan:** policy siap ditinjau  
**Acceptance criteria:**
- policy ada untuk tabel penting
- admin dan reseller dibedakan jelas
- tidak ada pendekatan akses yang terlalu longgar

## TASK B-03

**Nama task:** Create Reference Seed and Test Seed Skeleton  
**Tujuan:** Menyediakan seed minimum untuk pengembangan dan validasi.  
**File yang dibuat/diubah:**  
- `supabase/migrations/003_seed_reference_data.sql`
- `supabase/seed.sql`
**Lokasi file:** `supabase/`  
**Langkah kerja AI agent:**
1. Buat seed reference entity.
2. Buat seed skenario bisnis minimum.
3. Pisahkan reference seed dan scenario seed jika perlu.
**Dependency:** schema inti  
**Output yang diharapkan:** seed dasar siap pakai  
**Acceptance criteria:**
- skenario minimum dari environment doc dapat diwujudkan
- seed mudah diulang

## TASK B-04

**Nama task:** Build Read Model Views  
**Tujuan:** Membuat view dan agregat utama untuk dashboard, pesanan, dan laporan.  
**File yang dibuat/diubah:**  
- `supabase/migrations/004_views_ringkasan.sql`
**Lokasi file:** `supabase/migrations/`  
**Langkah kerja AI agent:**
1. Tentukan output read model utama.
2. Bangun view ringkasan reseller.
3. Bangun view ringkasan pesanan dan watchlist pesanan.
4. Bangun view detail item pesanan, stok, dan pengiriman.
**Dependency:** schema inti + query contracts  
**Output yang diharapkan:** read model siap dipanggil  
**Acceptance criteria:**
- field read model cukup untuk dashboard/laporan
- formula final tidak ditinggalkan ke frontend

## TASK B-05

**Nama task:** Implement Periode Boundary  
**Tujuan:** Membuat boundary untuk lifecycle periode.  
**File yang dibuat/diubah:**  
- `supabase/migrations/005_functions_periode.sql`
- `src/lib/server/services/periode.service.ts`
- `src/lib/server/validation/periode.validation.ts`
- `src/lib/server/workflows/periode.workflow.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Definisikan fungsi create/update/read periode.
2. Definisikan validasi input.
3. Definisikan checklist readiness helper.
4. Bungkus ke service layer.
**Dependency:** schema + RLS + view dasar  
**Output yang diharapkan:** boundary periode siap  
**Acceptance criteria:**
- service periode punya API internal yang jelas
- boundary periode tidak bocor ke frontend

## TASK B-06

**Nama task:** Implement Reseller Boundary  
**Tujuan:** Membuat boundary admin untuk reseller dan statusnya.  
**File yang dibuat/diubah:**  
- `supabase/migrations/006_functions_reseller.sql`
- `src/lib/server/services/reseller.service.ts`
- `src/lib/server/validation/reseller.validation.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Definisikan fungsi create/update/list reseller.
2. Validasi status dan periode.
3. Bungkus ke service.
**Dependency:** periode boundary  
**Output yang diharapkan:** management reseller siap  
**Acceptance criteria:**
- list dan status reseller dapat dibaca terstruktur
- semua akses tetap role-aware

## TASK B-07

**Nama task:** Implement Pesanan Boundary  
**Tujuan:** Membuat core flow `pesanan_konsumen`, `detail_pesanan_konsumen`, dan finalisasi pesanan.  
**File yang dibuat/diubah:**  
- `supabase/migrations/007_functions_pesanan.sql`
- `src/lib/server/services/pesanan.service.ts`
- `src/lib/server/services/detail-pesanan.service.ts`
- `src/lib/server/validation/pesanan.validation.ts`
- `src/lib/server/validation/detail-pesanan.validation.ts`
- `src/lib/server/workflows/pesanan.workflow.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Buat boundary create header pesanan beserta item awal.
2. Buat boundary update detail item pesanan.
3. Buat boundary finalisasi pesanan.
4. Tangani status item `AKTIF`, `TERHENTI`, `BATAL`.
5. Bungkus ke service dan workflow.
**Dependency:** schema + reseller + periode  
**Output yang diharapkan:** flow pesanan inti siap  
**Acceptance criteria:**
- header pesanan dan detail item dipisah jelas
- logic finalisasi pesanan tidak tercampur dengan UI

## TASK B-08

**Nama task:** Implement Setoran Boundary  
**Tujuan:** Membuat layer setoran konsumen dan setoran pusat.  
**File yang dibuat/diubah:**  
- `supabase/migrations/008_functions_setoran.sql`
- `src/lib/server/services/setoran.service.ts`
- `src/lib/server/validation/setoran.validation.ts`
- `src/lib/server/workflows/setoran.workflow.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Buat fungsi setoran konsumen.
2. Buat fungsi setoran pusat.
3. Buat helper pelunasan dan ringkasan.
4. Bungkus ke service dan workflow.
**Dependency:** pesanan boundary  
**Output yang diharapkan:** layer setoran siap  
**Acceptance criteria:**
- boundary setoran mencakup dua layer
- status lunas dan sisa bayar dapat dihitung konsisten

## TASK B-09

**Nama task:** Implement Gudang Boundary  
**Tujuan:** Membuat flow belanja, packing, pembagian, pengiriman.  
**File yang dibuat/diubah:**  
- `supabase/migrations/009_functions_gudang.sql`
- `src/lib/server/services/gudang.service.ts`
- `src/lib/server/validation/gudang.validation.ts`
- `src/lib/server/workflows/gudang.workflow.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Definisikan boundary belanja.
2. Definisikan boundary packing.
3. Definisikan boundary pembagian dan pengiriman.
4. Bungkus ke workflow.
**Dependency:** schema + pesanan  
**Output yang diharapkan:** gudang core flow siap  
**Acceptance criteria:**
- stok dan perpindahan barang tidak dipecah liar
- flow gudang punya audit trail yang bisa dilanjutkan

## TASK B-10

**Nama task:** Implement Keuangan Boundary  
**Tujuan:** Membuat flow kas masuk, mutasi kas, dan pencairan.  
**File yang dibuat/diubah:**  
- `supabase/migrations/010_functions_keuangan.sql`
- `src/lib/server/services/keuangan.service.ts`
- `src/lib/server/validation/keuangan.validation.ts`
- `src/lib/server/workflows/pencairan.workflow.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Definisikan boundary kas masuk.
2. Definisikan boundary mutasi kas.
3. Definisikan boundary pencairan.
4. Tambahkan orkestrasi pencairan.
**Dependency:** setoran boundary + gudang boundary  
**Output yang diharapkan:** keuangan dasar siap  
**Acceptance criteria:**
- flow keuangan bisa diaudit
- pencairan tidak menjadi write liar

## TASK B-11

**Nama task:** Implement Reporting Boundary  
**Tujuan:** Menyediakan query/report endpoint yang siap dipakai frontend.  
**File yang dibuat/diubah:**  
- `supabase/migrations/011_functions_dashboard_reports.sql`
- `src/lib/server/services/dashboard-report.service.ts`
- `src/lib/server/validation/dashboard-report.validation.ts`
- `src/lib/server/workflows/dashboard-report.workflow.ts`
**Lokasi file:** migration + server layer  
**Langkah kerja AI agent:**
1. Definisikan kebutuhan dashboard admin dan reseller.
2. Definisikan read/report boundary.
3. Bungkus ke validation, service, dan workflow.
**Dependency:** view ringkasan + semua boundary utama  
**Output yang diharapkan:** laporan siap konsumsi  
**Acceptance criteria:**
- dashboard dan laporan inti bisa dipanggil dari satu layer terpusat

## TASK B-12

**Nama task:** Implement Audit Trigger Layer  
**Tujuan:** Menyediakan audit otomatis untuk aksi penting.  
**File yang dibuat/diubah:**  
- `supabase/migrations/012_triggers_audit.sql`
- `src/lib/server/background/audit-sync.job.ts`
**Lokasi file:** migration + background layer  
**Langkah kerja AI agent:**
1. Definisikan trigger insert audit.
2. Tentukan tabel yang wajib audit.
3. Siapkan jalur ekstensi job hanya jika agregasi audit tambahan benar-benar dibutuhkan.
**Dependency:** schema inti  
**Output yang diharapkan:** audit siap  
**Acceptance criteria:**
- aksi penting punya jalur audit
- struktur audit konsisten
- implementasi first wave tetap valid tanpa mewajibkan job audit terpisah

## TASK B-13A

**Nama task:** Build Admin Route Layer  
**Tujuan:** Menyediakan route internal untuk area admin yang hanya membungkus validation dan service layer.  
**File yang dibuat/diubah:** route admin di `src/app/api/admin/**/route.ts`  
**Lokasi file:** `src/app/api/admin/`  
**Langkah kerja AI agent:**
1. Bentuk route admin per domain sesuai kontrak domain resmi.
2. Pastikan handler hanya membaca session/role, validasi input, dan memanggil service.
3. Gunakan response envelope backend yang sudah disepakati.
4. Pastikan route admin tidak menanam logic bisnis final.
**Dependency:** service layer admin dan validation layer admin  
**Output yang diharapkan:** route admin siap dipakai integrasi internal  
**Acceptance criteria:**
- route admin tidak memuat logic bisnis final
- tiap route admin hanya memanggil validation + service yang sesuai
- area admin yang belum menjadi dependency resmi tidak ikut disentuh

## TASK B-13B

**Nama task:** Build Reseller Route Layer  
**Tujuan:** Menyediakan route internal untuk area reseller yang hanya membungkus validation dan service layer.  
**File yang dibuat/diubah:** route reseller di `src/app/api/reseller/**/route.ts`  
**Lokasi file:** `src/app/api/reseller/`  
**Langkah kerja AI agent:**
1. Bentuk route reseller per domain sesuai kontrak domain resmi.
2. Pastikan handler hanya membaca session/role, validasi input, dan memanggil service.
3. Gunakan response envelope backend yang sama dengan area admin.
4. Pastikan route reseller tidak menanam logic bisnis final.
**Dependency:** service layer reseller dan validation layer reseller  
**Output yang diharapkan:** route reseller siap dipakai integrasi internal  
**Acceptance criteria:**
- route reseller tidak memuat logic bisnis final
- tiap route reseller hanya memanggil validation + service yang sesuai
- kontrak response tidak menyimpang dari area admin

## TASK B-13C

**Nama task:** Stabilize Shared Response Envelope  
**Tujuan:** Menormalkan response success/failure yang dipakai route internal.  
**File yang dibuat/diubah:** helper response backend yang dipakai lintas route  
**Lokasi file:** `src/lib/server/`, `src/app/api/`  
**Langkah kerja AI agent:**
1. Bentuk helper response success/failure yang reusable.
2. Samakan field wajib untuk success, error, dan metadata ringan.
3. Pastikan helper dapat dipakai route admin dan reseller tanpa branch liar.
4. Hindari memasukkan detail UI ke dalam response envelope.
**Dependency:** B-13A, B-13B  
**Output yang diharapkan:** response envelope backend konsisten  
**Acceptance criteria:**
- route admin dan reseller memakai shape response yang sama
- tidak ada route baru yang membuat envelope inline berbeda sendiri
- response tetap cukup sederhana untuk dibaca service consumer

## TASK B-13D

**Nama task:** Stabilize Shared Error Mapping  
**Tujuan:** Menormalkan mapping error validation, access, dan business rule di route internal.  
**File yang dibuat/diubah:** helper error mapping backend yang dipakai lintas route  
**Lokasi file:** `src/lib/server/`, `src/app/api/`  
**Langkah kerja AI agent:**
1. Bentuk helper mapping error code ke HTTP status dan payload error.
2. Pastikan validation error, access error, dan domain error dipisahkan jelas.
3. Pastikan route admin dan reseller memanggil helper yang sama.
4. Hindari pesan error bebas yang memaksa frontend menebak.
**Dependency:** B-13A, B-13B, B-13C  
**Output yang diharapkan:** error mapping backend konsisten  
**Acceptance criteria:**
- error route dipetakan dengan helper yang sama
- validation dan access error tidak bercampur dengan domain error
- frontend tidak perlu membaca pesan bebas untuk mengenali jenis error

## TASK B-14

**Nama task:** Build Background Jobs and Workflow Helpers  
**Tujuan:** Menyediakan proses non-UI yang mendukung sistem.  
**File yang dibuat/diubah:**  
- `src/lib/server/background/period-check.job.ts`
- `src/lib/server/background/watchlist-refresh.job.ts`
- `src/lib/server/workflows/*.ts`
**Lokasi file:** server background/workflow  
**Langkah kerja AI agent:**
1. Buat file skeleton untuk `period-check` dan `watchlist-refresh`.
2. Buat sebagai abstraction/placeholder saja tanpa mengimplementasikan full cron scheduler, karena keputusan fase awal sudah mengunci watchlist dan summary dihitung on-demand.
3. Hubungkan logika abstraction ke service layer yang relevan jika nanti dipanggil.
**Dependency:** service layer stabil, `DL-019` di `docs/truth/01-decision_log.md`  
**Output yang diharapkan:** Abstraction jalur ekstensi proses terjadwal tersedia tanpa over-engineering scheduler di wave pertama.  
**Acceptance criteria:**
- background concerns tidak bercampur dengan route
- workflow helper direpresentasikan sebagai placeholder fungsi/service terpisah
- agent tidak mengimplementasikan cron library (seperti node-cron) sebelum ada arahan tegas

## TASK B-15A

**Nama task:** Backend Naming Hardening  
**Tujuan:** Merapikan naming entity, function, dan helper backend agar konsisten dengan source of truth dokumen.  
**File yang dibuat/diubah:** file backend yang masih menyimpan naming lama atau naming drift  
**Lokasi file:** `src/lib/server/`, `src/app/api/`, `supabase/migrations/` bila diperlukan oleh kontrak resmi  
**Langkah kerja AI agent:**
1. Audit naming entity, route helper, dan boundary function terhadap dokumen kontrak resmi.
2. Samakan istilah backend ke vocabulary final yang sudah dikunci.
3. Hindari rename area yang belum menjadi dependency task aktif.
4. Pastikan tidak ada dua nama berbeda untuk boundary yang sama.
**Dependency:** semua task backend domain inti selesai  
**Output yang diharapkan:** naming backend konsisten dengan dokumen  
**Acceptance criteria:**
- tidak ada naming drift aktif pada file backend yang disentuh
- boundary function, service, dan helper memakai vocabulary final yang sama
- rename tidak memperluas scope ke modul yang belum menjadi dependency resmi

## TASK B-15B

**Nama task:** Backend Response Contract Hardening  
**Tujuan:** Memastikan response route dan service wrapper konsisten dengan contract pack.  
**File yang dibuat/diubah:** file backend yang membentuk response route/service  
**Lokasi file:** `src/lib/server/`, `src/app/api/`  
**Langkah kerja AI agent:**
1. Audit response success dan failure terhadap response envelope resmi.
2. Samakan field wajib response pada route yang sudah menjadi dependency aktif.
3. Pastikan payload read dan write tidak memakai shape ad hoc.
4. Hindari perubahan ke route yang belum dibangun pada wave aktif.
**Dependency:** B-13C, B-13D  
**Output yang diharapkan:** response backend stabil untuk integrasi  
**Acceptance criteria:**
- response success/failure konsisten lintas route yang aktif
- tidak ada route aktif yang memakai shape inline berbeda sendiri
- perubahan tetap terbatas pada area dependency resmi

## TASK B-15C

**Nama task:** Backend Validation Boundary Hardening  
**Tujuan:** Memastikan validation tidak tumpang tindih dan tetap berada di boundary yang tepat.  
**File yang dibuat/diubah:** validation dan service file yang overlap  
**Lokasi file:** `src/lib/server/validation/`, `src/lib/server/services/`  
**Langkah kerja AI agent:**
1. Audit overlap antara input validation, business validation, dan access validation.
2. Pindahkan validation yang salah boundary ke layer yang tepat.
3. Pastikan route tidak mengulang business validation yang sudah dijaga boundary resmi.
4. Hindari menambah business rule baru di luar dokumen kontrak.
**Dependency:** validation layer dan service layer domain aktif  
**Output yang diharapkan:** validation backend konsisten  
**Acceptance criteria:**
- input, access, dan business validation terpisah jelas
- route tidak mengulang validation bisnis yang sudah hidup di boundary resmi
- tidak ada business rule baru yang muncul dari asumsi implementor

## TASK B-15D

**Nama task:** Backend Access and Period Safety Hardening  
**Tujuan:** Memastikan route dan service aktif tetap sadar role dan periode.  
**File yang dibuat/diubah:** guard, route, dan service file yang menyentuh role/periode  
**Lokasi file:** `src/lib/server/auth/`, `src/lib/server/services/`, `src/app/api/`  
**Langkah kerja AI agent:**
1. Audit guard role pada route dan service aktif.
2. Audit penggunaan `periode_id` pada operasi yang harus period-aware.
3. Pastikan area admin/reseller tidak bocor akses lintas boundary.
4. Hindari memperluas audit ke modul yang belum menjadi dependency resmi.
**Dependency:** service layer aktif, auth guard aktif  
**Output yang diharapkan:** backend aman untuk integrasi role/periode  
**Acceptance criteria:**
- route aktif memakai guard role yang benar
- operasi sensitif tetap period-aware
- area admin dan reseller tidak saling memakai akses yang salah

---

## 13. Pseudocode dan Flow Script Penting

## 13.1 Pseudocode Setoran Konsumen

```ts
async function createSetoranKonsumen(input, actor) {
  validateInput(input);
  validateActor(actor, 'RESELLER_OR_ADMIN');

  const pesanan = await loadPesanan(input.pesananKonsumenId);
  ensurePesananBelongsToActor(pesanan, actor);

  const targetBerjalan = computeTargetBerjalan(pesanan);
  const totalBayarSebelumnya = await getTotalBayar(pesanan.id);
  const totalBaru = totalBayarSebelumnya + input.nominal;

  if (totalBaru > targetBerjalan) {
    throw domainError('TARGET_TERLAMPAUI', 'Setoran melebihi target berjalan');
  }

  const trx = await insertSetoran(pesanan.id, input.nominal, input.tanggal);
  await recomputePesananSummary(pesanan.id);
  await recomputeResellerSummary(pesanan.noReseller, pesanan.periodeId);
  await writeAudit('SETORAN_KONSUMEN_DIBUAT', trx.id, actor.id);

  return buildSuccessPayload(trx.id);
}
```

## 13.2 Pseudocode Finalisasi Pesanan

```ts
async function finalisasiPesananKonsumen(input, actor) {
  validateActor(actor, 'RESELLER_OR_ADMIN');
  const pesanan = await loadPesanan(input.pesananKonsumenId);

  ensurePesananCanBeFinalized(pesanan);

  const totalBayar = await getTotalBayar(pesanan.id);
  const targetBerjalan = await computeTargetBerjalan(pesanan.id);

  if (targetBerjalan < totalBayar) {
    throw domainError('TARGET_PESANAN_LEBIH_KECIL', 'Total target pesanan lebih kecil dari total setoran');
  }

  const result = await markPesananFinal(pesanan.id, actor);
  await recomputePesananSummary(pesanan.id);
  await writeAudit('PESANAN_DIFINALKAN', pesanan.id, actor.id);

  return result;
}
```

---

## 14. Urutan Eksekusi

Urutan kerja backend dari awal sampai akhir:

1. bangun schema inti
2. aktifkan RLS
3. siapkan seed minimum
4. bangun read model dasar
5. implement boundary periode
6. implement boundary reseller
7. implement boundary pesanan
8. implement boundary setoran
9. implement boundary gudang
10. implement boundary keuangan
11. implement boundary laporan
12. implement audit trigger
13. bentuk service layer
14. bentuk API route layer
15. bentuk background jobs dan workflow helper bila benar-benar dibutuhkan setelah baseline on-demand stabil
16. hardening backend

---

## 15. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- schema terlalu sering berubah setelah service layer mulai dibangun
- query output dan kebutuhan frontend tidak sinkron
- RLS menjadi sumber bug integrasi yang terlambat terlihat
- background concern tercampur ke route synchronous

### Asumsi

- Supabase tetap menjadi data engine resmi
- API internal tetap perlu walau boundary utama ada di Supabase
- service layer Next.js tetap dipakai sebagai anti-corruption layer
- model transaksi konsumen final memakai `pesanan_konsumen` dan `detail_pesanan_konsumen`

### Keputusan First Wave

- background jobs fase awal cukup sebagai abstraction path, bukan implementasi wajib
- watchlist operasional fase awal cukup berupa query agregat on-demand
