# FRONTEND / MOCKUP PLAN
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint teknis frontend mockup untuk AI coding agent.

Scope dokumen ini sengaja dibatasi hanya untuk area frontend mockup:

- halaman
- komponen
- layout
- navigasi
- state UI
- dummy data
- interaction flow
- placeholder API
- struktur folder dan file frontend

Dokumen ini tidak menjadi tempat untuk:

- backend real
- database real
- auth real
- engine logic real

Kontrak lintas layer yang wajib dijaga selama implementasi plan ini:

- `docs/integration_contract_pack.md`

---

## 1. Tujuan Plan

Tujuan frontend mockup adalah membangun representasi UI lengkap yang:

- bisa dipakai untuk validasi alur operasional
- konsisten dengan shell `Modernize`
- cukup rinci untuk implementasi bertahap
- aman dipisahkan dari backend real
- mudah dihubungkan ke service layer nyata di fase integrasi

Mockup frontend harus terasa seperti aplikasi nyata, tetapi sumber datanya tetap dummy atau placeholder API yang terkontrol.

---

## 2. Aturan Umum untuk AI Coding Agent

### Aturan Scope

- Fokus hanya pada presentasi dan interaksi UI.
- Jangan menulis logic bisnis final.
- Jangan membuat koneksi database real.
- Jangan membuat auth real.
- Jangan membuat RPC real.
- Jangan mengunci struktur data dummy ke shape yang liar; shape dummy harus sedekat mungkin ke kontrak data akhir.

### Aturan Integrasi Shell

- Gunakan shell `Modernize` sebagai fondasi global.
- Pertahankan theme, customizer, dark mode, layout width mode, navigation mode.
- Jangan membuat shell kedua yang bersaing dengan shell global.

### Aturan Komponen

- Gunakan `Material-UI v7` dan wrapper internal.
- Gunakan pola dari `docs/frontend_component_contracts.md`.
- Semua copy user-facing dalam Bahasa Indonesia.
- Semua halaman data wajib punya loading, empty, error, dan success feedback yang masuk akal.

### Aturan Dummy

- Dummy data harus modular.
- Dummy fetcher harus reusable.
- Dummy data shape harus menggunakan penamaan field yang konsisten.
- State UI harus siap diganti ke source real nanti tanpa rewrite besar.

---

## 3. Struktur Folder Frontend

Struktur target frontend mockup:

```txt
src/
  app/
    (DashboardLayout)/
      admin/
        dashboard/
          page.tsx
        periode/
          page.tsx
          create/
            page.tsx
          [id]/
            page.tsx
          [id]/
            setup/
              page.tsx
            checklist/
              page.tsx
        reseller/
          page.tsx
        master/
          akun-kas/
            page.tsx
          barang/
            page.tsx
          barang-periode/
            page.tsx
          komisi/
            page.tsx
          paket/
            page.tsx
        pesanan/
          page.tsx
          perlu-perhatian/
            page.tsx
          [id]/
            page.tsx
        setoran-konsumen/
          page.tsx
        setoran-pusat/
          page.tsx
        gudang/
          belanja/
            page.tsx
          packing/
            page.tsx
          pembagian/
            page.tsx
          pengiriman/
            page.tsx
        keuangan/
          kas-masuk/
            page.tsx
          mutasi-kas/
            page.tsx
          pencairan/
            page.tsx
        laporan/
          rekap-reseller/
            page.tsx
          stok/
            page.tsx
          audit/
            page.tsx
          pembagian/
            page.tsx
          keuangan/
            page.tsx
      reseller/
        page.tsx
        konsumen/
          page.tsx
        pesanan/
          page.tsx
          create/
            page.tsx
          [id]/
            page.tsx
        setor/
          page.tsx
        akun/
          page.tsx
      layout.tsx
    auth/
      login/
        page.tsx
      register/
        page.tsx
    layout.tsx
    loading.tsx
    not-found.tsx

  app/components/
    admin/
      dashboard/
      periode/
      reseller/
      master/
      pesanan/
      setoran/
      gudang/
      keuangan/
      laporan/
    reseller/
      dashboard/
      konsumen/
      pesanan/
      setor/
      akun/
    shared/
    ui-components/
    layout/

  app/context/
    AuthContext.tsx
    PeriodContext.tsx
    MockApiContext.tsx
    UIContext.tsx
    customizerContext.tsx

  hooks/
    admin/
    reseller/
    shared/
    dummy/

  lib/
    dummy-data/
    dummy-api/
    query/
    validasi/
    utils/

  types/
    models.ts
    api.ts
    ui.ts
    forms.ts
```

---

## 4. Struktur File Frontend dan Fungsi Setiap File

## 4.1 Global App Files

| File | Fungsi |
|---|---|
| `src/app/layout.tsx` | Root layout, provider chain, global theme bootstrap |
| `src/app/loading.tsx` | Loading global sederhana |
| `src/app/not-found.tsx` | Halaman 404 |
| `src/app/(DashboardLayout)/layout.tsx` | Shell utama admin dan reseller |
| `src/app/auth/login/page.tsx` | Halaman login mock |
| `src/app/auth/register/page.tsx` | Halaman registrasi mock |

## 4.2 Context Files

| File | Fungsi |
|---|---|
| `src/app/context/AuthContext.tsx` | Auth mock, role switching, profile mock |
| `src/app/context/PeriodContext.tsx` | Periode aktif dan periode laporan mock |
| `src/app/context/MockApiContext.tsx` | Toggle latency, error scenario, mock behavior |
| `src/app/context/UIContext.tsx` | Snackbar, dialog helper, UI transient state |

## 4.3 Shared UI Files

| File | Fungsi |
|---|---|
| `src/app/components/shared/StatusBadge.tsx` | Badge status konsisten |
| `src/app/components/shared/SummaryCard.tsx` | KPI card reusable |
| `src/app/components/shared/EmptyState.tsx` | Empty state reusable |
| `src/app/components/shared/ErrorState.tsx` | Error state reusable |
| `src/app/components/shared/LoadingState.tsx` | Loading skeleton reusable |
| `src/app/components/shared/ConfirmDialog.tsx` | Dialog konfirmasi reusable |
| `src/app/components/shared/RupiahInput.tsx` | Input nominal UI-only |
| `src/app/components/shared/PeriodeSelector.tsx` | Selector periode aktif/laporan |
| `src/app/components/shared/PageSection.tsx` | Wrapper section halaman |
| `src/app/components/shared/PageHeader.tsx` | Header halaman dengan aksi |

## 4.4 Layout Files

| File | Fungsi |
|---|---|
| `src/app/components/layout/AdminShell.tsx` | Komposisi header, nav, content admin |
| `src/app/components/layout/ResellerShell.tsx` | Komposisi header, bottom nav, content reseller |
| `src/app/components/layout/AdminSidebar.tsx` | Sidebar admin |
| `src/app/components/layout/AdminHorizontalNav.tsx` | Nav horizontal admin |
| `src/app/components/layout/AdminHeader.tsx` | Header admin |
| `src/app/components/layout/ResellerHeader.tsx` | Header reseller |
| `src/app/components/layout/ResellerBottomNav.tsx` | Bottom nav reseller |
| `src/app/components/layout/BreadcrumbsBar.tsx` | Breadcrumb UI |

## 4.5 Dummy Layer Files

| File | Fungsi |
|---|---|
| `src/lib/dummy-data/periodes.ts` | Data periode mock |
| `src/lib/dummy-data/resellers.ts` | Data reseller mock |
| `src/lib/dummy-data/konsumens.ts` | Data konsumen mock |
| `src/lib/dummy-data/pesanans.ts` | Data header pesanan mock |
| `src/lib/dummy-data/detail-pesanans.ts` | Data item/detail pesanan mock |
| `src/lib/dummy-data/setoran.ts` | Data setoran mock |
| `src/lib/dummy-data/gudang.ts` | Data belanja, packing, pengiriman mock |
| `src/lib/dummy-data/keuangan.ts` | Data kas, pencairan, mutasi mock |
| `src/lib/dummy-data/dashboard.ts` | KPI dan watchlist mock |
| `src/lib/dummy-api/fetcher.ts` | Placeholder fetcher dengan latency dan error injection |
| `src/lib/dummy-api/endpoints.ts` | Peta endpoint mock |
| `src/lib/dummy-api/handlers.ts` | Handler mock berdasarkan endpoint |
| `src/lib/query/keys.ts` | Query key mock untuk SWR |

## 4.6 Types Files

| File | Fungsi |
|---|---|
| `src/types/models.ts` | Domain model mock |
| `src/types/api.ts` | Contract placeholder API |
| `src/types/ui.ts` | Type UI state |
| `src/types/forms.ts` | Type input form |

---

## 5. Hubungan Antar File

Relasi utama:

```txt
Page
-> Feature screen component
-> Shared components
-> Hook
-> query keys + dummy fetcher
-> dummy API handlers
-> dummy data
```

Contoh hubungan:

- `admin/dashboard/page.tsx`
  -> `AdminDashboardScreen.tsx`
  -> `useAdminDashboardMock.ts`
  -> `dummy-api/fetcher.ts`
  -> `dummy-api/handlers.ts`
  -> `dummy-data/dashboard.ts`

- `reseller/setor/page.tsx`
  -> `ResellerSetorScreen.tsx`
  -> `KonsumenSearchList.tsx`
  -> `FormSetoranMock.tsx`
  -> `useResellerSetoranMock.ts`
  -> `dummy-data/pesanans.ts` + `dummy-data/setoran.ts`

---

## 6. Breakdown Component dan Function per File

## 6.1 `StatusBadge.tsx`

Komponen:

- `StatusBadge`
- `getStatusTone`
- `getStatusLabel`

Tanggung jawab:

- mapping status ke tone
- mapping status ke label human-readable

## 6.2 `SummaryCard.tsx`

Komponen:

- `SummaryCard`
- `SummaryCardSkeleton`

Tanggung jawab:

- render KPI
- handle loading visual

## 6.3 `AdminSidebar.tsx`

Komponen:

- `AdminSidebar`
- `AdminSidebarGroup`
- `AdminSidebarItem`

Tanggung jawab:

- menampilkan grup menu admin
- sinkron dengan route aktif
- sinkron dengan role dan periode state mock

## 6.4 `ResellerBottomNav.tsx`

Komponen:

- `ResellerBottomNav`
- `BottomNavItem`

Tanggung jawab:

- menampilkan 5 tab utama
- sinkron dengan route aktif

## 6.5 `dummy-api/fetcher.ts`

Function:

- `mockFetch`
- `withLatency`
- `withScenarioError`
- `resolveMockEndpoint`

Tanggung jawab:

- simulasi request/response
- simulasi loading
- simulasi error state

---

## 7. Dummy Data Design

Dummy data harus memuat skenario:

- periode `PERSIAPAN`, `AKTIF`, `SELESAI`
- reseller `PENDING`, `AKTIF`, `NONAKTIF`
- konsumen lunas dan belum lunas
- pesanan belum final dan sudah final
- detail pesanan aktif, terhenti, batal
- setoran konsumen dan setoran pusat
- stok aman dan stok kritis
- watchlist operasional

Aturan dummy:

- jumlah data cukup untuk menunjukkan state kosong, normal, dan padat
- jangan hanya punya happy path
- semua halaman penting harus bisa diuji dengan minimal 3 state

---

## 8. Placeholder API Design

Endpoint placeholder:

```txt
GET /mock/admin/dashboard
GET /mock/admin/periode
GET /mock/admin/pesanan
GET /mock/admin/reseller
GET /mock/reseller/dashboard
GET /mock/reseller/konsumen
GET /mock/reseller/pesanan
POST /mock/reseller/setor
POST /mock/reseller/pesanan
```

Aturan:

- response shape stabil
- error shape konsisten
- success shape mudah diganti ke service real

Contoh shape:

```ts
type MockApiResponse<T> = {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
};
```

---

## 9. Interaction Flow Penting

## 9.1 Flow Admin Periode Setup

```txt
Admin buka daftar periode
-> pilih periode persiapan
-> masuk wizard setup
-> cek checklist
-> aktivasi mock
-> UI pindah ke dashboard aktif
```

## 9.2 Flow Reseller Input Setoran

```txt
Reseller buka tab Setor
-> cari konsumen
-> pilih konsumen/pesanan
-> isi nominal
-> submit mock
-> tampil feedback sukses
-> list dan ringkasan diperbarui
```

## 9.3 Flow Admin Pesanan Review

```txt
Admin buka daftar pesanan
-> filter status
-> buka detail
-> review detail item dan status finalisasi
-> tampilkan watchlist atau histori item
```

---

## 10. Task Implementation Detail

Di bawah ini adalah task blueprint yang bisa langsung dipakai AI coding agent.

## TASK F-01

**Nama task:** Setup Root Frontend Mockup Foundation  
**Tujuan:** Menyiapkan fondasi layout root, provider, dan route base.  
**File yang dibuat/diubah:**  
- `src/app/layout.tsx`
- `src/app/loading.tsx`
- `src/app/not-found.tsx`
- `src/app/(DashboardLayout)/layout.tsx`
**Lokasi file:** `src/app/`  
**Langkah kerja AI agent:**
1. Review provider dan shell existing.
2. Pastikan root layout memuat provider global.
3. Buat loading global sederhana.
4. Pastikan dashboard layout siap dipakai admin dan reseller.
**Dependency:** shell Modernize existing  
**Output yang diharapkan:** fondasi route dan provider siap  
**Acceptance criteria:**
- root layout render tanpa error
- dashboard layout bisa menjadi wadah admin dan reseller
- loading dan not-found tersedia

## TASK F-02

**Nama task:** Build Mock Context Layer  
**Tujuan:** Membuat auth, period, UI, dan mock API context untuk kebutuhan mockup.  
**File yang dibuat/diubah:**  
- `src/app/context/AuthContext.tsx`
- `src/app/context/PeriodContext.tsx`
- `src/app/context/UIContext.tsx`
- `src/app/context/MockApiContext.tsx`
**Lokasi file:** `src/app/context/`  
**Langkah kerja AI agent:**
1. Definisikan state auth mock.
2. Definisikan active period mock.
3. Definisikan snackbar/dialog helper.
4. Definisikan latency/error scenario switch.
**Dependency:** `src/types/ui.ts`, `src/types/models.ts`  
**Output yang diharapkan:** state global mock siap  
**Acceptance criteria:**
- role admin/reseller dapat disimulasikan
- periode aktif dapat diubah
- error scenario dapat ditrigger

## TASK F-03

**Nama task:** Create Dummy Data Domain Packs  
**Tujuan:** Menyediakan data dummy modular untuk semua modul inti.  
**File yang dibuat/diubah:**  
- `src/lib/dummy-data/periodes.ts`
- `src/lib/dummy-data/resellers.ts`
- `src/lib/dummy-data/konsumens.ts`
- `src/lib/dummy-data/pesanans.ts`
- `src/lib/dummy-data/detail-pesanans.ts`
- `src/lib/dummy-data/setoran.ts`
- `src/lib/dummy-data/gudang.ts`
- `src/lib/dummy-data/keuangan.ts`
- `src/lib/dummy-data/dashboard.ts`
**Lokasi file:** `src/lib/dummy-data/`  
**Langkah kerja AI agent:**
1. Definisikan type mock lebih dulu.
2. Susun dataset per domain.
3. Pastikan ada state normal, empty, warning, dan overloaded.
4. Pastikan naming antar file konsisten.
**Dependency:** `src/types/models.ts`  
**Output yang diharapkan:** dataset mock reusable  
**Acceptance criteria:**
- semua domain inti punya data mock
- tidak ada field naming yang saling bertabrakan
- state UI utama bisa didemokan

## TASK F-04

**Nama task:** Build Placeholder API Layer  
**Tujuan:** Menyediakan placeholder API untuk SWR dan interaksi mock.  
**File yang dibuat/diubah:**  
- `src/lib/dummy-api/endpoints.ts`
- `src/lib/dummy-api/handlers.ts`
- `src/lib/dummy-api/fetcher.ts`
- `src/types/api.ts`
**Lokasi file:** `src/lib/dummy-api/`, `src/types/`  
**Langkah kerja AI agent:**
1. Buat daftar endpoint mock.
2. Buat handler per endpoint.
3. Tambahkan latency simulator.
4. Tambahkan error simulator.
**Dependency:** dummy data domain packs  
**Output yang diharapkan:** request mock konsisten  
**Acceptance criteria:**
- mock fetcher bisa return success
- mock fetcher bisa return error
- response shape konsisten lintas endpoint

## TASK F-05

**Nama task:** Build Shared Component Library  
**Tujuan:** Menyediakan shared components minimum untuk semua screen.  
**File yang dibuat/diubah:**  
- `src/app/components/shared/StatusBadge.tsx`
- `src/app/components/shared/SummaryCard.tsx`
- `src/app/components/shared/EmptyState.tsx`
- `src/app/components/shared/ErrorState.tsx`
- `src/app/components/shared/LoadingState.tsx`
- `src/app/components/shared/ConfirmDialog.tsx`
- `src/app/components/shared/RupiahInput.tsx`
- `src/app/components/shared/PeriodeSelector.tsx`
- `src/app/components/shared/PageSection.tsx`
- `src/app/components/shared/PageHeader.tsx`
**Lokasi file:** `src/app/components/shared/`  
**Langkah kerja AI agent:**
1. Implement status badge mapping.
2. Implement KPI card.
3. Implement state components.
4. Implement rupiah input UI-only.
**Dependency:** theme + types + MUI  
**Output yang diharapkan:** shared building blocks siap  
**Acceptance criteria:**
- komponen shared dapat dipakai di admin dan reseller
- dark mode tidak merusak tampilan
- semua komponen mengikuti kontrak frontend component

## TASK F-06

**Nama task:** Build Admin Shell Components  
**Tujuan:** Membentuk shell admin yang siap memuat halaman bisnis.  
**File yang dibuat/diubah:**  
- `src/app/components/layout/AdminShell.tsx`
- `src/app/components/layout/AdminSidebar.tsx`
- `src/app/components/layout/AdminHorizontalNav.tsx`
- `src/app/components/layout/AdminHeader.tsx`
- `src/app/components/layout/BreadcrumbsBar.tsx`
**Lokasi file:** `src/app/components/layout/`  
**Langkah kerja AI agent:**
1. Buat struktur shell admin.
2. Hubungkan ke customizer mode.
3. Render menu bisnis admin.
4. Tambahkan periode selector di header.
**Dependency:** context auth + period + shell existing  
**Output yang diharapkan:** admin shell stabil  
**Acceptance criteria:**
- sidebar/horizontal mode bisa ditampilkan
- route admin terlihat terstruktur
- header menampilkan periode aktif/laporan

## TASK F-07

**Nama task:** Build Reseller Shell Components  
**Tujuan:** Membentuk shell reseller mobile-first.  
**File yang dibuat/diubah:**  
- `src/app/components/layout/ResellerShell.tsx`
- `src/app/components/layout/ResellerHeader.tsx`
- `src/app/components/layout/ResellerBottomNav.tsx`
**Lokasi file:** `src/app/components/layout/`  
**Langkah kerja AI agent:**
1. Buat shell ringan reseller.
2. Tambahkan bottom nav 5 tab.
3. Tampilkan status sinkronisasi mock.
4. Pastikan aman pada viewport mobile.
**Dependency:** auth context + period context  
**Output yang diharapkan:** shell reseller mobile siap  
**Acceptance criteria:**
- bottom nav berfungsi
- halaman tidak menimbulkan horizontal scroll
- header reseller ringkas dan jelas

## TASK F-08

**Nama task:** Build Admin Dashboard Mock Screen  
**Tujuan:** Menampilkan dashboard admin berdasarkan data mock.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/dashboard/page.tsx`
- `src/app/components/admin/dashboard/AdminDashboardScreen.tsx`
- `src/app/components/admin/dashboard/KpiGrid.tsx`
- `src/app/components/admin/dashboard/WatchlistPanel.tsx`
- `src/app/components/admin/dashboard/ResellerProgressPanel.tsx`
- `src/app/components/admin/dashboard/WarehousePanel.tsx`
- `src/app/components/admin/dashboard/CashPanel.tsx`
**Lokasi file:** admin dashboard folders  
**Langkah kerja AI agent:**
1. Bentuk page shell.
2. Pecah dashboard menjadi section reusable.
3. Hubungkan ke hook mock dashboard.
4. Siapkan loading/error/empty scenario.
**Dependency:** shared components + dummy dashboard data  
**Output yang diharapkan:** dashboard admin mock interaktif  
**Acceptance criteria:**
- KPI muncul
- watchlist tampil
- state kosong dan error bisa ditrigger

## TASK F-09

**Nama task:** Build Periode Management Mock Module  
**Tujuan:** Menyediakan list, detail, setup wizard, dan checklist periode.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/periode/page.tsx`
- `src/app/(DashboardLayout)/admin/periode/create/page.tsx`
- `src/app/(DashboardLayout)/admin/periode/[id]/page.tsx`
- `src/app/(DashboardLayout)/admin/periode/[id]/setup/page.tsx`
- `src/app/(DashboardLayout)/admin/periode/[id]/checklist/page.tsx`
- `src/app/components/admin/periode/*`
**Lokasi file:** periode folders  
**Langkah kerja AI agent:**
1. Buat table list periode.
2. Buat form create/edit mock.
3. Buat wizard setup.
4. Buat checklist activation mock.
**Dependency:** periode dummy data + shared components  
**Output yang diharapkan:** flow periode bisa didemokan  
**Acceptance criteria:**
- create/edit mock berjalan
- checklist dan setup state jelas
- transisi visual status periode bisa disimulasikan

## TASK F-10

**Nama task:** Build Reseller Daily Setor Flow Mock  
**Tujuan:** Menyediakan flow utama harian reseller untuk input setoran.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/reseller/setor/page.tsx`
- `src/app/components/reseller/setor/ResellerSetorScreen.tsx`
- `src/app/components/reseller/setor/KonsumenSearchList.tsx`
- `src/app/components/reseller/setor/SetoranFormMock.tsx`
- `src/app/components/reseller/setor/QuickNominalChips.tsx`
**Lokasi file:** reseller setor folders  
**Langkah kerja AI agent:**
1. Buat daftar konsumen prioritas.
2. Buat search dan select flow.
3. Buat form nominal.
4. Buat feedback sukses dan update local state.
**Dependency:** konsumen/pesanan/setoran dummy data  
**Output yang diharapkan:** flow 3 langkah setoran bisa dicoba  
**Acceptance criteria:**
- search bekerja
- nominal bisa diinput
- submit mock mengubah tampilan ringkasan

## TASK F-11A1

**Nama task:** Build Admin Akun Kas Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `master/akun-kas`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/master/akun-kas/page.tsx`
- komponen admin akun kas terkait
**Lokasi file:** admin master akun-kas folders  
**Langkah kerja AI agent:**
1. Buat header, filter dasar, dan content panel untuk akun kas.
2. Gunakan presentasi tabel/list yang cocok untuk data rekening/kas.
3. Hubungkan ke hook mock akun kas.
4. Pastikan empty, loading, dan error state dasar tersedia.
**Dependency:** shared components + dummy pack akun kas  
**Output yang diharapkan:** modul akun kas dapat dinavigasi  
**Acceptance criteria:**
- route `admin/master/akun-kas` tersedia
- halaman akun kas tidak dead end
- state dasar akun kas terlihat jelas

## TASK F-11A2

**Nama task:** Build Admin Barang Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `master/barang`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/master/barang/page.tsx`
- komponen admin barang terkait
**Lokasi file:** admin master barang folders  
**Langkah kerja AI agent:**
1. Buat header, filter dasar, dan content panel untuk daftar barang.
2. Gunakan presentasi tabel/list yang cocok untuk master barang.
3. Hubungkan ke hook mock barang.
4. Pastikan empty, loading, dan error state dasar tersedia.
**Dependency:** shared components + dummy pack barang  
**Output yang diharapkan:** modul barang dapat dinavigasi  
**Acceptance criteria:**
- route `admin/master/barang` tersedia
- halaman barang tidak dead end
- state dasar barang terlihat jelas

## TASK F-11A3

**Nama task:** Build Admin Barang Periode Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `master/barang-periode`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/master/barang-periode/page.tsx`
- komponen admin barang periode terkait
**Lokasi file:** admin master barang-periode folders  
**Langkah kerja AI agent:**
1. Buat header, filter dasar, dan content panel untuk barang periode.
2. Hubungkan ke hook mock barang periode.
3. Tampilkan state `no_periode` bila periode belum aktif atau belum dipilih.
4. Pastikan warning konteks periode terlihat jujur.
**Dependency:** shared components + dummy pack barang periode + period context  
**Output yang diharapkan:** modul barang periode dapat dinavigasi  
**Acceptance criteria:**
- route `admin/master/barang-periode` tersedia
- state `no_periode` tampil bila relevan
- halaman barang periode tidak dead end

## TASK F-11A4

**Nama task:** Build Admin Komisi Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `master/komisi`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/master/komisi/page.tsx`
- komponen admin komisi terkait
**Lokasi file:** admin master komisi folders  
**Langkah kerja AI agent:**
1. Buat header, filter dasar, dan content panel untuk master komisi.
2. Gunakan presentasi tabel/list yang sesuai untuk level komisi.
3. Hubungkan ke hook mock komisi.
4. Pastikan empty, loading, dan error state dasar tersedia.
**Dependency:** shared components + dummy pack komisi  
**Output yang diharapkan:** modul komisi dapat dinavigasi  
**Acceptance criteria:**
- route `admin/master/komisi` tersedia
- halaman komisi tidak dead end
- state dasar komisi terlihat jelas

## TASK F-11A5

**Nama task:** Build Admin Paket Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `master/paket`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/master/paket/page.tsx`
- komponen admin paket terkait
**Lokasi file:** admin master paket folders  
**Langkah kerja AI agent:**
1. Buat header, filter dasar, dan content panel untuk master paket.
2. Gunakan presentasi yang membantu membedakan paket tunggal dan komposit.
3. Hubungkan ke hook mock paket.
4. Pastikan badge/status paket terbaca konsisten dengan kontrak UI.
**Dependency:** shared components + dummy pack paket  
**Output yang diharapkan:** modul paket dapat dinavigasi  
**Acceptance criteria:**
- route `admin/master/paket` tersedia
- halaman paket tidak dead end
- status/badge paket terbaca jelas

## TASK F-11A6

**Nama task:** Build Admin Reseller Periode Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `reseller-periode`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/reseller-periode/page.tsx`
- komponen admin reseller-periode terkait
**Lokasi file:** admin reseller-periode folders  
**Langkah kerja AI agent:**
1. Buat header, filter dasar, dan content panel untuk reseller per periode.
2. Hubungkan ke hook mock reseller periode.
3. Tampilkan state `no_periode` bila periode belum aktif atau belum dipilih.
4. Pastikan konteks relasi reseller-periode tidak bercampur dengan master reseller global.
**Dependency:** shared components + dummy pack reseller-periode + period context  
**Output yang diharapkan:** modul reseller-periode dapat dinavigasi  
**Acceptance criteria:**
- route `admin/reseller-periode` tersedia
- state `no_periode` tampil bila relevan
- halaman reseller-periode tidak dead end

## TASK F-11B

**Nama task:** Build Admin Relasi Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `reseller` dan `konsumen`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/reseller/page.tsx`
- `src/app/(DashboardLayout)/admin/konsumen/page.tsx`
- komponen admin relasi terkait
**Lokasi file:** admin relation folders  
**Langkah kerja AI agent:**
1. Buat list reseller dan list konsumen dengan search/filter dasar.
2. Pecah ke page header, filter bar, dan content table/list.
3. Hubungkan ke hook mock reseller dan konsumen.
4. Siapkan empty, loading, dan error state yang konsisten.
**Dependency:** shared components + reseller/konsumen dummy packs  
**Output yang diharapkan:** modul relasi admin dapat dinavigasi  
**Acceptance criteria:**
- route reseller dan konsumen tersedia
- tiap halaman punya state dasar
- tidak ada halaman relasi admin yang kosong tanpa arah

## TASK F-11C

**Nama task:** Build Admin Pesanan Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `pesanan` dan daftar detail item pesanan.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/pesanan/page.tsx`
- `src/app/(DashboardLayout)/admin/pesanan/perlu-perhatian/page.tsx`
- `src/app/(DashboardLayout)/admin/pesanan/[id]/page.tsx`
- komponen admin pesanan terkait
**Lokasi file:** admin pesanan folders  
**Langkah kerja AI agent:**
1. Buat list dan panel status pesanan.
2. Buat detail pesanan beserta daftar item mock.
3. Hubungkan ke hook mock pesanan dan detail item.
4. Pastikan watchlist dan status badge konsisten dengan kontrak UI.
**Dependency:** shared components + pesanan/detail-pesanan dummy packs  
**Output yang diharapkan:** modul pesanan admin dapat dinavigasi  
**Acceptance criteria:**
- route pesanan tersedia
- status utama terbaca jelas
- detail item pesanan tidak dead end

## TASK F-11D

**Nama task:** Build Admin Setoran Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `setoran-konsumen` dan `setoran-pusat`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/setoran-konsumen/page.tsx`
- `src/app/(DashboardLayout)/admin/setoran-pusat/page.tsx`
- komponen admin setoran terkait
**Lokasi file:** admin setoran folders  
**Langkah kerja AI agent:**
1. Buat dua screen monitoring yang terpisah jelas per layer.
2. Tambahkan filter dasar dan state read-only.
3. Hubungkan ke hook mock setoran admin.
4. Pastikan admin dapat membedakan layer 1 dan layer 2 tanpa kebingungan.
**Dependency:** shared components + setoran dummy packs  
**Output yang diharapkan:** modul setoran admin dapat dinavigasi  
**Acceptance criteria:**
- dua route setoran admin tersedia
- layer konsumen dan pusat tidak tercampur
- menu admin setoran tidak dead end

## TASK F-11E

**Nama task:** Build Admin Gudang Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `gudang`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/gudang/belanja/page.tsx`
- `src/app/(DashboardLayout)/admin/gudang/packing/page.tsx`
- `src/app/(DashboardLayout)/admin/gudang/pembagian/page.tsx`
- `src/app/(DashboardLayout)/admin/gudang/pengiriman/page.tsx`
- komponen admin gudang terkait
**Lokasi file:** admin gudang folders  
**Langkah kerja AI agent:**
1. Buat screen belanja, packing, pembagian, dan pengiriman.
2. Gunakan header, filter, dan content panel yang sesuai konteks gudang.
3. Hubungkan ke hook mock gudang.
4. Pastikan status backlog dan alert visual terbaca jelas.
**Dependency:** shared components + gudang dummy packs  
**Output yang diharapkan:** modul gudang admin dapat dinavigasi  
**Acceptance criteria:**
- semua route gudang tersedia
- screen punya state dasar
- tidak ada halaman gudang yang kosong tanpa konteks

## TASK F-11F

**Nama task:** Build Admin Keuangan Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `keuangan`.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/keuangan/kas-masuk/page.tsx`
- `src/app/(DashboardLayout)/admin/keuangan/mutasi-kas/page.tsx`
- `src/app/(DashboardLayout)/admin/keuangan/pencairan/page.tsx`
- komponen admin keuangan terkait
**Lokasi file:** admin keuangan folders  
**Langkah kerja AI agent:**
1. Buat screen kas masuk, mutasi kas, dan pencairan.
2. Pisahkan ringkasan saldo, filter, dan list transaksi.
3. Hubungkan ke hook mock keuangan.
4. Pastikan warning saldo dan eligibility pencairan terlihat jujur.
**Dependency:** shared components + keuangan dummy packs  
**Output yang diharapkan:** modul keuangan admin dapat dinavigasi  
**Acceptance criteria:**
- semua route keuangan tersedia
- state warning dan error dasar tersedia
- menu keuangan tidak dead end

## TASK F-11G

**Nama task:** Build Admin Laporan dan Audit Skeleton  
**Tujuan:** Menyediakan skeleton untuk modul admin `laporan` dan audit.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/admin/laporan/rekap-reseller/page.tsx`
- `src/app/(DashboardLayout)/admin/laporan/stok/page.tsx`
- `src/app/(DashboardLayout)/admin/laporan/audit/page.tsx`
- `src/app/(DashboardLayout)/admin/laporan/pembagian/page.tsx`
- `src/app/(DashboardLayout)/admin/laporan/keuangan/page.tsx`
- komponen admin laporan/audit terkait
**Lokasi file:** admin laporan folders  
**Langkah kerja AI agent:**
1. Buat screen laporan inti dan audit dengan state baca yang jujur.
2. Siapkan tabel/list/card sesuai konteks laporan.
3. Hubungkan ke hook mock laporan dan audit.
4. Pastikan route audit memakai `admin/laporan/audit` dan tidak menjadi dead end.
**Dependency:** shared components + dashboard/laporan dummy packs  
**Output yang diharapkan:** modul laporan admin dapat dinavigasi  
**Acceptance criteria:**
- route laporan dan audit tersedia
- tiap route punya loading/empty/error dasar
- modul audit tidak berdiri sebagai placeholder kosong

## TASK F-12A

**Nama task:** Build Reseller Dashboard & Akun Skeleton  
**Tujuan:** Menyediakan halaman utama (dashboard) dan pengaturan (akun) untuk reseller.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/reseller/page.tsx`
- `src/app/(DashboardLayout)/reseller/akun/page.tsx`
- komponen screen terkait
**Lokasi file:** reseller route folders  
**Langkah kerja AI agent:**
1. Buat layout dashboard dengan summary cards dasar.
2. Buat halaman akun dengan mock data profil reseller.
3. Pastikan state loading dan error disimulasikan.
**Dependency:** reseller dummy data  
**Output yang diharapkan:** Dashboard dan Akun dapat dinavigasi.  
**Acceptance criteria:**
- halaman dashboard me-render komponen KPI mock
- halaman akun tersedia tanpa error

## TASK F-12B

**Nama task:** Build Reseller Konsumen & Pesanan Skeleton  
**Tujuan:** Menyediakan halaman manajemen konsumen dan pesanan untuk reseller.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/reseller/konsumen/page.tsx`
- `src/app/(DashboardLayout)/reseller/pesanan/page.tsx`
- `src/app/(DashboardLayout)/reseller/pesanan/create/page.tsx`
- `src/app/(DashboardLayout)/reseller/pesanan/[id]/page.tsx`
- komponen screen terkait
**Lokasi file:** reseller route folders  
**Langkah kerja AI agent:**
1. Buat daftar konsumen dengan search/filter mock.
2. Buat daftar pesanan berjalan.
3. Buat detail pesanan dengan daftar item dan state finalisasi.
4. Hubungkan ke mock hooks terkait.
**Dependency:** reseller dummy data  
**Output yang diharapkan:** Area konsumen dan pesanan dapat dilihat.  
**Acceptance criteria:**
- route konsumen dan pesanan tidak kosong
- search/filter mock UI tersedia
- detail pesanan dapat menampilkan status finalisasi

## TASK F-12C

**Nama task:** Build Reseller Riwayat Modular Skeleton  
**Tujuan:** Menyediakan area riwayat modular untuk reseller tanpa unified history feed.  
**File yang dibuat/diubah:**  
- `src/app/(DashboardLayout)/reseller/akun/page.tsx`
- komponen screen terkait
**Lokasi file:** reseller route folders  
**Langkah kerja AI agent:**
1. Buat section riwayat setor pusat dan ringkasan perubahan pesanan yang tersedia.
2. Tampilkan bahwa riwayat fase awal tetap modular per domain.
3. Hindari asumsi adanya satu feed gabungan lintas semua transaksi reseller.
**Dependency:** reseller dummy data  
**Output yang diharapkan:** Riwayat modular bisa dinavigasi dari area akun.  
**Acceptance criteria:**
- tidak ada dead end pada area riwayat reseller
- tidak ada unified history dependency pada first slice

## TASK F-13

**Nama task:** Build Frontend Validation and Form Types  
**Tujuan:** Menormalkan type dan validasi frontend mock.  
**File yang dibuat/diubah:**  
- `src/types/forms.ts`
- `src/lib/validasi/*.ts`
**Lokasi file:** types and validation folders  
**Langkah kerja AI agent:**
1. Definisikan type form.
2. Definisikan schema validasi UI.
3. Pastikan bisa dipakai ulang.
**Dependency:** domain types  
**Output yang diharapkan:** form types stabil  
**Acceptance criteria:**
- form pages tidak mendefinisikan type liar per file
- error validation konsisten

## TASK F-14

**Nama task:** Build Frontend Hook Layer for Mock Data  
**Tujuan:** Menyediakan hooks domain untuk semua screen penting.  
**File yang dibuat/diubah:**  
- `src/hooks/admin/*`
- `src/hooks/reseller/*`
- `src/hooks/shared/*`
- `src/hooks/dummy/*`
**Lokasi file:** hooks folders  
**Langkah kerja AI agent:**
1. Buat query hooks per domain.
2. Bungkus SWR dengan query keys.
3. Tambahkan helper mutation mock.
**Dependency:** dummy-api + query keys  
**Output yang diharapkan:** page tidak memanggil dummy fetcher langsung  
**Acceptance criteria:**
- layer hook reusable
- page lebih bersih
- mutation mock punya state pending/success/error

## TASK F-15A

**Nama task:** Frontend Route and Dead-End Hardening  
**Tujuan:** Menutup dead route dan dead-end screen pada frontend mockup.  
**File yang dibuat/diubah:** page dan navigasi frontend yang masih dead end  
**Lokasi file:** `src/app/`, `src/components/`, `src/config/` bila diperlukan oleh navigasi  
**Langkah kerja AI agent:**
1. Audit route yang sudah tercantum di plan tetapi belum punya screen usable.
2. Audit link, menu, dan CTA yang menuju placeholder kosong.
3. Rapikan fallback agar route belum aktif tidak tampak seperti rusak.
4. Hindari menyentuh domain yang belum menjadi dependency resmi.
**Dependency:** semua task frontend screen utama selesai  
**Output yang diharapkan:** frontend tidak punya dead route besar  
**Acceptance criteria:**
- tidak ada route utama yang berakhir di placeholder kosong
- menu dan CTA utama tidak mengarah ke halaman buntu
- perubahan tetap terbatas pada route aktif

## TASK F-15B

**Nama task:** Frontend Copy and Label Hardening  
**Tujuan:** Menormalkan copy Bahasa Indonesia dan label UI agar konsisten lintas screen.  
**File yang dibuat/diubah:** screen dan komponen frontend yang masih punya copy tidak konsisten  
**Lokasi file:** `src/app/`, `src/components/`  
**Langkah kerja AI agent:**
1. Audit copy user-facing yang masih campur istilah atau nada.
2. Samakan label tombol, badge, helper text, dan heading ke vocabulary final.
3. Hindari mengganti istilah yang sudah dikunci source of truth.
4. Pastikan copy tetap pendek dan operasional.
**Dependency:** route dan screen utama tersedia  
**Output yang diharapkan:** copy UI frontend konsisten  
**Acceptance criteria:**
- istilah user-facing konsisten lintas screen aktif
- tidak ada label yang memakai vocabulary lama
- heading, tombol, dan helper text tetap operasional

## TASK F-15C

**Nama task:** Frontend State Coverage Hardening  
**Tujuan:** Memastikan loading, empty, error, disabled, dan success feedback hadir di screen utama.  
**File yang dibuat/diubah:** screen dan komponen state frontend aktif  
**Lokasi file:** `src/app/`, `src/components/`, `src/hooks/` bila perlu untuk mock state  
**Langkah kerja AI agent:**
1. Audit screen utama untuk state loading, empty, error, disabled, dan success feedback.
2. Tambahkan state yang hilang tanpa mengubah kontrak domain.
3. Pastikan state period-aware seperti `no_periode` tetap dipisah dari empty biasa.
4. Hindari membuat state yang tidak didukung kontrak UI.
**Dependency:** semua screen utama aktif  
**Output yang diharapkan:** state coverage frontend siap diverifikasi  
**Acceptance criteria:**
- screen utama tidak kosong tanpa state yang jujur
- state `no_periode` tidak tercampur dengan empty biasa
- feedback disabled/success tampil konsisten pada flow aktif

## TASK F-15D

**Nama task:** Frontend Reseller Mobile Hardening  
**Tujuan:** Memastikan area reseller aman dipakai di viewport mobile utama.  
**File yang dibuat/diubah:** screen reseller yang masih rawan overflow atau layout rusak  
**Lokasi file:** `src/app/(DashboardLayout)/reseller/`, komponen reseller terkait  
**Langkah kerja AI agent:**
1. Audit overflow, wrapping, dan tap target pada screen reseller.
2. Rapikan list, form, dan action area agar tetap usable di mobile.
3. Pastikan summary card dan tabel ringkas tidak memaksa scroll horizontal liar.
4. Hindari mengubah area admin pada task ini.
**Dependency:** screen reseller utama selesai  
**Output yang diharapkan:** area reseller aman di mobile  
**Acceptance criteria:**
- tidak ada overflow liar pada screen reseller utama
- action penting tetap mudah disentuh di mobile
- layout reseller tetap terbaca tanpa horizontal scroll yang tidak perlu

## TASK F-15E

**Nama task:** Frontend Theme Compatibility Hardening  
**Tujuan:** Memastikan dark mode dan mode shell utama tidak merusak keterbacaan screen aktif.  
**File yang dibuat/diubah:** screen dan komponen frontend aktif yang bermasalah di theme mode  
**Lokasi file:** `src/app/`, `src/components/`, theme wrapper terkait  
**Langkah kerja AI agent:**
1. Audit kontras warna, badge, border, dan panel pada mode terang/gelap.
2. Rapikan komponen yang hilang contrast atau state hover/focus-nya.
3. Pastikan area admin desktop dan reseller mobile tetap terbaca.
4. Hindari redesign visual di luar masalah kompatibilitas theme.
**Dependency:** shared components dan screen utama selesai  
**Output yang diharapkan:** frontend aman di theme mode utama  
**Acceptance criteria:**
- dark mode tidak membuat teks/panel utama sulit dibaca
- area admin tetap usable di desktop
- area reseller tetap usable di mobile

---

## 11. Pseudocode Task Penting

## 11.1 Pseudocode Mock Fetcher

```ts
async function mockFetch<T>(endpoint: string, options?: MockOptions): Promise<MockApiResponse<T>> {
  await withLatency(options?.latency ?? defaultLatency);

  if (shouldTriggerError(endpoint, options)) {
    return {
      success: false,
      error: {
        code: 'MOCK_ERROR',
        message: 'Terjadi kesalahan simulasi',
      },
    };
  }

  const handler = resolveMockEndpoint(endpoint);
  return handler(options);
}
```

## 11.2 Pseudocode Reseller Setoran Mock

```ts
function submitMockSetoran(input) {
  validateUiInput(input);
  const currentPesanan = findPesanan(input.pesananKonsumenId);
  const nextTotal = currentPesanan.totalBayar + input.nominal;

  updateMockPesanan({
    ...currentPesanan,
    totalBayar: nextTotal,
    sisaBayar: Math.max(0, currentPesanan.targetBerjalan - nextTotal),
  });

  pushMockTransaction(input);

  return {
    success: true,
    message: 'Setoran berhasil disimpan',
  };
}
```

---

## 12. Urutan Eksekusi

Urutan kerja frontend dari awal sampai akhir:

1. setup root layout dan dashboard layout
2. buat context layer
3. buat types dan dummy data
4. buat placeholder API layer
5. buat shared component library
6. bentuk admin shell
7. bentuk reseller shell
8. bangun dashboard admin
9. bangun modul periode
10. bangun flow setor reseller
11. bangun modul admin lain per domain, bukan sekaligus
12. bangun modul reseller lain
13. rapikan hook dan validasi
14. lakukan hardening frontend

---

## 13. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- mock data shape terlalu jauh dari data nyata
- shell global berubah terlalu banyak karena kebutuhan lokal
- terlalu banyak halaman dibangun sebelum komponen shared stabil

### Asumsi

- shell Modernize tetap dipakai
- frontend mockup boleh memakai placeholder auth dan placeholder API
- role admin dan reseller disimulasikan di context mock

### Catatan Plan Fase Awal

- semua route admin pada plan ini dianggap tampil sebagai skeleton mockup awal sesuai pembagian task per domain
- halaman auth mock perlu mensimulasikan approval reseller agar konsisten dengan modul `auth`
- bottom nav reseller final tetap 5 tab sesuai `docs/ui/reseller_uiux.md`
- mode horizontal admin cukup tersedia sebagai capability shell; tidak wajib menjadi mode default demonstrasi
