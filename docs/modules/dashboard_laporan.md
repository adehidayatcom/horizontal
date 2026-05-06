# Execution Module Dashboard Laporan
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `dashboard_laporan`.

Dokumen ini memecah implementasi modul dashboard dan laporan ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/product/prd.md`
- `docs/audit/product_truth_audit.md`
- `docs/contracts/query_contracts.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/contracts/integration_read_model_matrix.md`
- `docs/truth/system_maps.md`
- `docs/execution/task_kit_standard.md`
- `docs/modules/periode.md`
- `docs/modules/reseller.md`
- `docs/modules/program_order.md`
- `docs/modules/setoran.md`
- `docs/modules/gudang.md`
- `docs/modules/keuangan.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`
- `docs/ui/admin_dashboard_uiux.md`
- `docs/ui/reseller_uiux.md`

---

## 1. Tujuan Modul

Modul `dashboard_laporan` bertanggung jawab untuk:

- menampilkan ringkasan operasional admin untuk satu periode
- menampilkan beranda reseller yang fokus pada follow-up harian
- menyajikan watchlist lintas modul tanpa menghitung ulang bisnis di frontend
- menyediakan halaman laporan inti yang siap dipakai operasional dan review
- menggabungkan read model yang tersebar menjadi pengalaman baca yang cepat, jujur, dan terarah

Modul ini adalah lapisan baca lintas sistem, bukan lapisan transaksi.

---

## 2. Scope Modul

### Frontend Mockup Scope

- halaman dashboard admin
- halaman dashboard/beranda reseller
- panel KPI utama dan pendukung
- watchlist operasional
- panel progres reseller
- panel stok dan kas
- halaman laporan rekap reseller
- halaman laporan stok
- halaman laporan pengiriman
- halaman audit/koreksi ringan untuk admin
- state loading, empty, stale, dan error

### Backend/Core Scope

- RPC read `get_dashboard_admin`
- RPC read `get_dashboard_reseller`
- konsumsi view `v_ringkasan_reseller`, `v_ringkasan_pesanan_konsumen`, `v_pesanan_perlu_perhatian`, `v_detail_pesanan_aktif`, `v_stok_barang`, `v_stok_paket_jadi`, `v_saldo_kas`, `v_pembagian_reseller`, `v_riwayat_koreksi`, `v_audit_log`
- pagination, sorting, filtering server-side untuk list besar
- normalisasi envelope read-only untuk dashboard dan laporan
- tambahan view/RPC opsional `v_aktivitas_admin` bila aktivitas terbaru benar-benar dibutuhkan

---

## 3. Integration Contract Modul Dashboard Laporan

## 3.1 Admin Dashboard Payload

```ts
type AdminDashboardSummary = {
  periode_id: number;
  total_reseller_aktif: number;
  total_konsumen: number;
  total_program_aktif: number;
  total_program_belum_dikunci: number;
  total_program_perlu_perhatian: number;
  total_order_aktif: number;
  total_nilai_paket: number;
  total_dikumpulkan: number;
  total_disetor_pusat: number;
  total_kas_masuk: number;
  total_saldo_belum_disetor: number;
  total_sisa_setor_pusat: number;
  total_komisi: number;
  total_tabungan: number;
  jumlah_order_belum_kirim: number;
  jumlah_order_sudah_kirim: number;
  jumlah_order_batal: number;
  jumlah_pembagian_belum_diserahkan: number;
};
```

## 3.2 Reseller Dashboard Payload

```ts
type ResellerDashboardSummary = {
  periode_id: number;
  jumlah_konsumen: number;
  jumlah_pesanan_aktif: number;
  jumlah_item_pesanan: number;
  jumlah_konsumen_lunas: number;
  total_dikumpulkan: number;
  total_disetor_pusat: number;
  saldo_belum_disetor: number;
  sisa_setor_pusat: number;
  sisa_tabungan: number;
  sisa_komisi: number;
  poin: number;
};
```

## 3.3 Report List Envelope

```ts
type ReportListEnvelope<T> = {
  data: T[];
  meta: {
    periode_id: number;
    page: number;
    page_size: number;
    total_items: number;
    total_pages: number;
    sort_by?: string | null;
    sort_direction?: 'asc' | 'desc' | null;
    search?: string | null;
    filters?: Record<string, string | number | boolean | null>;
  };
};
```

## 3.4 Dashboard Page State Contract

```ts
type DashboardPageState =
  | { kind: 'loading' }
  | { kind: 'ready'; refreshed_at?: string | null }
  | { kind: 'empty_no_periode' }
  | { kind: 'empty_no_data' }
  | { kind: 'error'; message: string };
```

## 3.5 Error Codes Relevan

```txt
DASHBOARD_PERIODE_REQUIRED
DASHBOARD_DATA_NOT_FOUND
REPORT_INVALID_FILTER
REPORT_INVALID_SORT
REPORT_PAGE_OUT_OF_RANGE
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
```

---

## 4. Struktur Folder Modul Dashboard Laporan

## 4.1 Frontend

```txt
src/app/(DashboardLayout)/admin/dashboard/
  page.tsx

src/app/(DashboardLayout)/admin/laporan/
  rekap-reseller/
    page.tsx
  stok/
    page.tsx
  pengiriman/
    page.tsx
  audit/
    page.tsx

src/app/(DashboardLayout)/reseller/
  page.tsx

src/app/components/admin/dashboard/
  AdminDashboardScreen.tsx
  AdminPageHeader.tsx
  DashboardKpiGrid.tsx
  DashboardKpiCard.tsx
  OperationalWatchlist.tsx
  ResellerProgressPanel.tsx
  StockRiskPanel.tsx
  CashPositionPanel.tsx
  RecentActivityFeed.tsx
  DashboardEmptyState.tsx
  DashboardErrorState.tsx
  index.ts

src/app/components/admin/laporan/
  RekapResellerReportScreen.tsx
  LaporanStokScreen.tsx
  LaporanPengirimanScreen.tsx
  AuditKoreksiScreen.tsx
  ReportTableToolbar.tsx
  ReportPaginationBar.tsx
  ExportPlaceholderBar.tsx
  index.ts

src/app/components/reseller/dashboard/
  ResellerDashboardScreen.tsx
  ResellerSummaryCards.tsx
  PriorityKonsumenList.tsx
  SetorPusatQuickPanel.tsx
  ResellerDashboardEmptyState.tsx
  index.ts

src/hooks/admin/
  useAdminDashboardMock.ts
  useRekapResellerReportMock.ts
  useStokReportMock.ts
  usePengirimanReportMock.ts
  useAuditKoreksiMock.ts
  useRecentActivityMock.ts

src/hooks/reseller/
  useResellerDashboardMock.ts
  usePriorityKonsumenMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  004_views_ringkasan.sql
  011_functions_dashboard_reports.sql

src/lib/server/services/
  dashboard-report.service.ts

src/lib/server/validation/
  dashboard-report.validation.ts

src/lib/server/workflows/
  dashboard-report.workflow.ts

src/app/api/admin/dashboard/
  route.ts

src/app/api/admin/laporan/
  rekap-reseller/route.ts
  stok/route.ts
  pengiriman/route.ts
  audit/route.ts

src/app/api/reseller/dashboard/
  route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `admin/dashboard/page.tsx` | Entry route dashboard admin |
| `admin/laporan/rekap-reseller/page.tsx` | Halaman laporan ringkasan reseller |
| `admin/laporan/stok/page.tsx` | Halaman laporan stok barang dan paket |
| `admin/laporan/pengiriman/page.tsx` | Halaman laporan pengiriman dan pembagian |
| `admin/laporan/audit/page.tsx` | Halaman audit admin satu route dengan tab `Audit Log` dan `Riwayat Koreksi` |
| `reseller/page.tsx` | Entry route beranda reseller |
| `AdminDashboardScreen.tsx` | Orkestrasi section dashboard admin |
| `AdminPageHeader.tsx` | Header gradient dashboard admin |
| `DashboardKpiGrid.tsx` | Grid KPI utama dan pendukung |
| `DashboardKpiCard.tsx` | Kartu KPI reusable |
| `OperationalWatchlist.tsx` | Watchlist lintas modul |
| `ResellerProgressPanel.tsx` | Panel progres lunas reseller |
| `StockRiskPanel.tsx` | Panel stok dan packing kritis |
| `CashPositionPanel.tsx` | Panel posisi kas |
| `RecentActivityFeed.tsx` | Feed aktivitas terbaru opsional |
| `DashboardEmptyState.tsx` | Empty state admin |
| `DashboardErrorState.tsx` | Error state admin |
| `RekapResellerReportScreen.tsx` | Screen laporan rekap reseller |
| `LaporanStokScreen.tsx` | Screen laporan stok |
| `LaporanPengirimanScreen.tsx` | Screen laporan pengiriman |
| `AuditKoreksiScreen.tsx` | Screen audit admin dengan tab `Audit Log` dan `Riwayat Koreksi` |
| `ReportTableToolbar.tsx` | Search/filter/sort toolbar |
| `ReportPaginationBar.tsx` | Pagination list report |
| `ExportPlaceholderBar.tsx` | Placeholder aksi export atau print |
| `ResellerDashboardScreen.tsx` | Orkestrasi beranda reseller |
| `ResellerSummaryCards.tsx` | KPI ringkas reseller |
| `PriorityKonsumenList.tsx` | Konsumen prioritas follow-up |
| `SetorPusatQuickPanel.tsx` | Ringkasan setor pusat dan shortcut |
| `ResellerDashboardEmptyState.tsx` | Empty state reseller |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `004_views_ringkasan.sql` | View pendukung dashboard/laporan |
| `011_functions_dashboard_reports.sql` | RPC read dashboard dan laporan |
| `dashboard-report.service.ts` | Service layer untuk dashboard/laporan |
| `dashboard-report.validation.ts` | Validasi query param report |
| `dashboard-report.workflow.ts` | Workflow normalisasi read-model dan state |
| `admin/dashboard/route.ts` | API dashboard admin |
| `admin/laporan/rekap-reseller/route.ts` | API laporan rekap reseller |
| `admin/laporan/stok/route.ts` | API laporan stok |
| `admin/laporan/pengiriman/route.ts` | API laporan pengiriman |
| `admin/laporan/audit/route.ts` | API audit dan koreksi |
| `reseller/dashboard/route.ts` | API beranda reseller |

---

## 6. Hubungan Antar File

- `admin/dashboard/page.tsx` menggunakan `useAdminDashboardMock.ts` dan nantinya route `admin/dashboard/route.ts`
- `AdminDashboardScreen.tsx` merangkai `DashboardKpiGrid`, `OperationalWatchlist`, `ResellerProgressPanel`, `StockRiskPanel`, `CashPositionPanel`, dan `RecentActivityFeed`
- `OperationalWatchlist.tsx` menggabungkan data dari `get_dashboard_admin`, `v_pesanan_perlu_perhatian`, `v_detail_pesanan_aktif`, `v_stok_barang`, `v_stok_paket_jadi`, `v_pembagian_reseller`
- `RekapResellerReportScreen.tsx` membaca `v_ringkasan_reseller` dengan pagination server-side
- `LaporanStokScreen.tsx` membaca `v_stok_barang` dan `v_stok_paket_jadi`
- `LaporanPengirimanScreen.tsx` membaca `v_detail_pesanan_aktif` dan `v_pembagian_reseller`
- `AuditKoreksiScreen.tsx` membaca `get_laporan_audit_koreksi` dalam satu route dengan state tab internal; RPC ini membungkus `v_riwayat_koreksi` dan `v_audit_log`
- `ResellerDashboardScreen.tsx` membaca `get_dashboard_reseller` dan `PriorityKonsumenList.tsx` membaca `v_ringkasan_konsumen` atau `v_ringkasan_pesanan_konsumen`
- semua route laporan memanggil `dashboard-report.validation.ts`, `dashboard-report.service.ts`, lalu `dashboard-report.workflow.ts`

---

## 7. Breakdown Function/Component/Class per File

## 7.1 Frontend Breakdown

### `AdminDashboardScreen.tsx`

- `AdminDashboardScreen`
- `resolveAdminDashboardState`
- `renderDashboardBody`

### `DashboardKpiGrid.tsx`

- `DashboardKpiGrid`
- `buildPrimaryKpis`
- `buildSecondaryKpis`

### `OperationalWatchlist.tsx`

- `OperationalWatchlist`
- `buildWatchlistItems`
- `renderWatchlistCard`

### `RekapResellerReportScreen.tsx`

- `RekapResellerReportScreen`
- `handleSearch`
- `handleFilterStatus`
- `handlePageChange`

### `LaporanStokScreen.tsx`

- `LaporanStokScreen`
- `handleSwitchTab`
- `handleSort`

### `LaporanPengirimanScreen.tsx`

- `LaporanPengirimanScreen`
- `handleFilterStatusKirim`
- `handleFilterPembagianStatus`

### `ResellerDashboardScreen.tsx`

- `ResellerDashboardScreen`
- `renderPendingGuard`
- `renderResellerDashboard`

### `PriorityKonsumenList.tsx`

- `PriorityKonsumenList`
- `buildPriorityItems`
- `handleSelectKonsumen`

## 7.2 Backend Breakdown

### `dashboard-report.validation.ts`

- `validateDashboardQuery`
- `validateReportPaginationQuery`
- `validateReportFilterQuery`

### `dashboard-report.service.ts`

- `getAdminDashboardSummary`
- `getResellerDashboardSummary`
- `getRekapResellerReport`
- `getStokReport`
- `getPengirimanReport`
- `getAuditKoreksiReport`
- `getRecentActivity`

### `dashboard-report.workflow.ts`

- `resolveAdminDashboardEmptyState`
- `resolveResellerDashboardState`
- `normalizeReportMeta`
- `buildOperationalWatchlistPayload`
- `mergeStockPanelsPayload`

### `011_functions_dashboard_reports.sql`

- `get_dashboard_admin(...)`
- `get_dashboard_reseller(...)`
- `get_laporan_rekap_reseller(...)`
- `get_laporan_stok(...)`
- `get_laporan_pengiriman(...)`
- `get_laporan_audit_koreksi(...)`
- `get_recent_activity(...)`

---

## 8. First Working Slice

Slice pertama modul dashboard_laporan harus memberi nilai operasional nyata tanpa menunggu semua laporan lengkap:

1. admin dashboard menampilkan KPI utama dari `get_dashboard_admin`
2. admin watchlist menampilkan minimum:
   - reseller belum lunas
   - detail pesanan belum kirim
   - stok perlu belanja
   - paket perlu packing
3. reseller beranda menampilkan KPI ringkas dari `get_dashboard_reseller`
4. reseller beranda menampilkan daftar konsumen prioritas
5. halaman laporan rekap reseller tampil dengan filter dasar dan pagination

Yang belum wajib di slice pertama:

- recent activity penuh
- export/print final
- audit feed yang kompleks
- drill-down lintas banyak tabel dalam satu halaman
- unified history feed reseller lintas semua modul
- laporan P1 tambahan di luar laporan operasional inti

---

## 9. Task Execution Batches

### Batch D1 - Dashboard Foundation

- MDL-01
- MDL-02
- MDL-03

### Batch D2 - Admin Dashboard

- MDL-04
- MDL-05
- MDL-06
- MDL-07

### Batch D3 - Reseller Dashboard

- MDL-08
- MDL-09
- MDL-10

### Batch D4 - Laporan Inti

- MDL-11
- MDL-12
- MDL-13
- MDL-14

### Batch D5 - Audit dan Hardening

- MDL-15
- MDL-16
- MDL-17

---

## 10. Task Implementation Detail

## MDL-01 - Finalkan kontrak read-only dashboard

- tujuan: mengunci payload dashboard admin dan reseller agar frontend tidak menebak field
- file yang dibuat/diubah: `011_functions_dashboard_reports.sql`, `dashboard-report.service.ts`, `dashboard-report.validation.ts`
- lokasi file: `supabase/migrations/011_functions_dashboard_reports.sql`, `src/lib/server/services/dashboard-report.service.ts`, `src/lib/server/validation/dashboard-report.validation.ts`
- langkah kerja AI agent:
  - cocokkan output `get_dashboard_admin` dan `get_dashboard_reseller` dengan `query_contracts.md`
  - pastikan semua field memakai naming yang konsisten
  - pastikan `periode_id` selalu hadir
- dependency: `query_contracts.md`, modul sebelumnya
- output yang diharapkan: contract dashboard final
- acceptance criteria:
  - tidak ada field KPI yang ambigu
  - payload admin dan reseller bisa dikonsumsi langsung UI

## MDL-02 - Finalkan state dan envelope laporan

- tujuan: menyatukan pola loading, empty, error, dan pagination laporan
- file yang dibuat/diubah: `dashboard-report.validation.ts`, `dashboard-report.workflow.ts`
- lokasi file: `src/lib/server/validation/dashboard-report.validation.ts`, `src/lib/server/workflows/dashboard-report.workflow.ts`
- langkah kerja AI agent:
  - definisikan query page/page_size/sort/filter
  - buat normalizer `meta`
  - buat resolver state empty untuk admin dan reseller
- dependency: MDL-01
- output yang diharapkan: pola laporan seragam
- acceptance criteria:
  - semua route read list bisa memakai envelope yang sama
  - empty state tidak bercampur dengan error state

## MDL-03 - Tambahkan read model yang masih kurang untuk laporan

- tujuan: menutup kekurangan data untuk panel dan laporan inti
- file yang dibuat/diubah: `004_views_ringkasan.sql`, `011_functions_dashboard_reports.sql`
- lokasi file: `supabase/migrations/...`
- langkah kerja AI agent:
  - review `v_riwayat_koreksi` dan `v_audit_log`
  - tambah helper report atau view agregat bila perlu
  - putuskan apakah `get_recent_activity` sudah layak atau ditunda
- dependency: MDL-01
- output yang diharapkan: read layer cukup untuk dashboard/laporan
- acceptance criteria:
  - semua panel inti punya sumber data yang jelas
  - aktivitas terbaru disembunyikan bila belum punya kontrak jujur

## MDL-04 - Implementasi API dashboard admin

- tujuan: menyediakan satu endpoint admin dashboard yang ringan dan stabil
- file yang dibuat/diubah: `011_functions_dashboard_reports.sql`, `dashboard-report.service.ts`, `dashboard-report.workflow.ts`, `admin/dashboard/route.ts`
- lokasi file: `supabase/migrations/011_functions_dashboard_reports.sql`, `src/lib/server/...`, `src/app/api/admin/dashboard/route.ts`
- langkah kerja AI agent:
  - buat route GET admin dashboard
  - ambil `get_dashboard_admin`
  - gabungkan payload panel tambahan bila diperlukan
  - kirim envelope yang jujur untuk empty/error
- dependency: MDL-01, MDL-03
- output yang diharapkan: endpoint dashboard admin
- acceptance criteria:
  - query berbasis `periode_id`
  - response bisa men-drive KPI dan watchlist

## MDL-05 - Implementasi UI dashboard admin

- tujuan: menerjemahkan UI/UX dashboard admin menjadi screen nyata
- file yang dibuat/diubah: `admin/dashboard/page.tsx`, `AdminDashboardScreen.tsx`, `AdminPageHeader.tsx`, `DashboardKpiGrid.tsx`, `DashboardKpiCard.tsx`, `DashboardEmptyState.tsx`, `DashboardErrorState.tsx`, `useAdminDashboardMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun layout screen utama
  - tampilkan KPI utama dan pendukung
  - buat loading dan empty state
  - gunakan mock payload yang sama dengan contract final
- dependency: MDL-04
- output yang diharapkan: dashboard admin base
- acceptance criteria:
  - KPI utama terlihat tanpa scroll di desktop umum
  - tidak ada angka palsu saat loading

## MDL-06 - Implementasi watchlist dan panel progres admin

- tujuan: membuat dashboard admin benar-benar operasional, bukan hanya angka
- file yang dibuat/diubah: `OperationalWatchlist.tsx`, `ResellerProgressPanel.tsx`, `StockRiskPanel.tsx`, `CashPositionPanel.tsx`, `useRekapResellerReportMock.ts`, `useStokReportMock.ts`
- lokasi file: `src/app/components/admin/dashboard/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun panel watchlist
  - bangun panel progres reseller
  - bangun panel stok dan kas
  - tautkan CTA ke halaman modul terkait
- dependency: MDL-05
- output yang diharapkan: dashboard admin operasional
- acceptance criteria:
  - watchlist minimum terpenuhi
  - panel tidak menghitung formula bisnis sendiri

## MDL-07 - Tambahkan feed aktivitas terbaru opsional

- tujuan: memberi ruang ritme operasional tanpa memblokir slice awal
- file yang dibuat/diubah: `RecentActivityFeed.tsx`, `useRecentActivityMock.ts`
- lokasi file: `src/app/components/admin/dashboard/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - cek apakah kontrak `get_recent_activity` tersedia dari backend read layer
  - jika belum, tampilkan placeholder jujur atau sembunyikan section
  - jika ada, bangun feed dengan format waktu singkat tanpa membuat kontrak backend baru di task ini
- dependency: MDL-03
- output yang diharapkan: recent activity opsional
- acceptance criteria:
  - tidak ada data dummy yang berpura-pura produksi
  - section tidak merusak layout jika belum aktif

## MDL-08 - Implementasi API beranda reseller

- tujuan: menyediakan sumber ringkasan cepat untuk area reseller
- file yang dibuat/diubah: `011_functions_dashboard_reports.sql`, `dashboard-report.service.ts`, `dashboard-report.workflow.ts`, `reseller/dashboard/route.ts`
- lokasi file: `supabase/migrations/...`, `src/lib/server/...`, `src/app/api/reseller/dashboard/route.ts`
- langkah kerja AI agent:
  - buat route GET reseller dashboard
  - ambil `get_dashboard_reseller`
  - gabungkan daftar prioritas konsumen jika dibutuhkan
- dependency: MDL-01, modul reseller, pesanan, setoran
- output yang diharapkan: endpoint beranda reseller
- acceptance criteria:
  - `no_reseller` berasal dari auth/profile
  - response cukup untuk KPI dan shortcut setor

## MDL-09 - Implementasi UI beranda reseller

- tujuan: membuat beranda reseller fokus pada follow-up dan input setoran cepat
- file yang dibuat/diubah: `reseller/page.tsx`, `ResellerDashboardScreen.tsx`, `ResellerSummaryCards.tsx`, `SetorPusatQuickPanel.tsx`, `ResellerDashboardEmptyState.tsx`, `useResellerDashboardMock.ts`
- lokasi file: `src/app/...`, `src/hooks/reseller/...`
- langkah kerja AI agent:
  - bangun KPI ringkas reseller
  - buat panel shortcut setor pusat
  - buat pending/no periode/offline state
- dependency: MDL-08
- output yang diharapkan: beranda reseller base
- acceptance criteria:
  - beranda tidak terasa seperti mini admin dashboard
  - state `PENDING` dan `tidak ada periode aktif` jelas

## MDL-10 - Implementasi daftar konsumen prioritas

- tujuan: menghubungkan dashboard reseller dengan alur setoran harian
- file yang dibuat/diubah: `PriorityKonsumenList.tsx`, `usePriorityKonsumenMock.ts`, integrasi `v_ringkasan_konsumen` atau `v_ringkasan_pesanan_konsumen`
- lokasi file: `src/app/components/reseller/dashboard/PriorityKonsumenList.tsx`, `src/hooks/reseller/usePriorityKonsumenMock.ts`
- langkah kerja AI agent:
  - buat sort default `sisa_bayar DESC`, lalu `nama_konsumen ASC`
  - buat CTA ke halaman setor atau detail konsumen
  - tampilkan badge `LUNAS/BELUM`
- dependency: MDL-09, modul reseller/setoran
- output yang diharapkan: list prioritas konsumen
- acceptance criteria:
  - daftar mendorong action cepat
  - field target, total bayar, dan sisa bayar terbaca jelas

## MDL-11 - Implementasi laporan rekap reseller

- tujuan: memberi admin laporan ringkasan reseller yang stabil dan bisa ditindaklanjuti
- file yang dibuat/diubah: `admin/laporan/rekap-reseller/page.tsx`, `RekapResellerReportScreen.tsx`, `ReportTableToolbar.tsx`, `ReportPaginationBar.tsx`, `useRekapResellerReportMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - gunakan `v_ringkasan_reseller`
  - buat search reseller
  - buat filter status lunas
  - buat pagination server-side
- dependency: MDL-02
- output yang diharapkan: laporan rekap reseller
- acceptance criteria:
  - tidak render semua reseller sekaligus
  - formula tetap dari backend

## MDL-12 - Implementasi laporan stok

- tujuan: menyediakan satu halaman baca untuk keputusan gudang
- file yang dibuat/diubah: `admin/laporan/stok/page.tsx`, `LaporanStokScreen.tsx`, `useStokReportMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan tab stok barang dan stok paket jadi
  - buat sort default yang sesuai risiko
  - tampilkan indikator `harus_belanja` dan `harus_packing`
- dependency: modul gudang, MDL-02
- output yang diharapkan: laporan stok
- acceptance criteria:
  - halaman mudah dipakai untuk keputusan operasional
  - tidak ada perhitungan stok di browser

## MDL-13 - Implementasi laporan pengiriman dan pembagian

- tujuan: menyediakan visibilitas detail pesanan terkirim dan pembagian reseller
- file yang dibuat/diubah: `admin/laporan/pengiriman/page.tsx`, `LaporanPengirimanScreen.tsx`, `usePengirimanReportMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan detail pesanan by `status_kirim`
  - tampilkan status batch pembagian
  - buat filter `BELUM`, `SUDAH`, `DISERAHKAN`, `SELESAI`
- dependency: modul gudang, MDL-02
- output yang diharapkan: laporan pengiriman
- acceptance criteria:
  - admin bisa melihat backlog kirim dan pembagian
  - sumber data tetap satu arah dari view/report API

## MDL-14 - Implementasi laporan audit dan koreksi

- tujuan: memberi admin visibilitas kesalahan operasional dan audit penting
- file yang dibuat/diubah: `admin/laporan/audit/page.tsx`, `AuditKoreksiScreen.tsx`, `useAuditKoreksiMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun satu screen audit admin dengan dua tab: `Audit Log` dan `Riwayat Koreksi`
  - gunakan filter bersama untuk periode, jenis transaksi, dan kata kunci
  - konsumsi `get_laporan_audit_koreksi` melalui route internal yang dibangun terpisah
  - pastikan `tab = audit` membaca `v_audit_log` dan `tab = koreksi` membaca `v_riwayat_koreksi`
- dependency: MDL-03
- output yang diharapkan: laporan audit
- acceptance criteria:
  - audit admin-only
  - informasi alasan koreksi terlihat
  - frontend tidak membuat dua route audit terpisah

## MDL-15 - Sinkronkan state dan mock seluruh modul

- tujuan: menjaga semua halaman dashboard/laporan memakai shape data yang sama
- file yang dibuat/diubah: semua hook mock modul ini
- lokasi file: `src/hooks/admin/...`, `src/hooks/reseller/...`
- langkah kerja AI agent:
  - cek semua hook mock
  - cocokan dengan payload final
  - rapikan empty/loading/error state
- dependency: semua task sebelumnya
- output yang diharapkan: konsistensi state dan payload
- acceptance criteria:
  - tidak ada mock shape yang liar
  - halaman tidak saling bertentangan

## MDL-16 - Tambahkan placeholder export yang jujur

- tujuan: menyiapkan titik ekstensi export tanpa pura-pura sudah final
- file yang dibuat/diubah: `ExportPlaceholderBar.tsx`, screen report terkait
- lokasi file: `src/app/components/admin/laporan/ExportPlaceholderBar.tsx`
- langkah kerja AI agent:
  - buat placeholder button export/print
  - beri label jika fitur belum aktif
  - pastikan tidak mengganggu flow utama
- dependency: MDL-11, MDL-12, MDL-13
- output yang diharapkan: placeholder export rapi
- acceptance criteria:
  - tidak ada tombol bohong
  - UX tetap jujur soal status fitur

## MDL-17 - Verifikasi performa dan dependency report

- tujuan: memastikan dashboard/laporan tidak menjadi bottleneck
- file yang dibuat/diubah: `dashboard-report.service.ts`, route terkait, docs bila perlu
- lokasi file: `src/lib/server/services/dashboard-report.service.ts`, `src/app/api/...`
- langkah kerja AI agent:
  - cek pagination server-side
  - cek search/sort berada di backend
  - cek section yang boleh disembunyikan jika query belum siap
- dependency: semua task modul ini
- output yang diharapkan: dashboard/laporan siap dibangun bertahap
- acceptance criteria:
  - list besar tidak diambil semua ke browser
  - dashboard admin tetap ringan

---

## 11. Pseudocode Task Penting

## 11.1 Pseudocode `get_dashboard_admin`

```txt
validate periode_id
summary = call get_dashboard_admin(periode_id)

if no active/historical periode found:
  return empty_no_periode

if summary exists but major panels still empty:
  return empty_no_data with allowed quick actions

load top reseller ringkasan
load stok barang kritis
load stok paket jadi kritis
load watchlist pesanan

return merged dashboard payload
```

## 11.2 Flow watchlist admin

```txt
start with empty array

if reseller belum lunas > 0:
  add watchlist item

if detail pesanan belum kirim > 0:
  add watchlist item

if stok barang harus belanja > 0:
  add watchlist item

if stok paket harus packing > 0:
  add watchlist item

if pembagian belum diserahkan > 0:
  add watchlist item

if no items:
  show "Operasional periode ini terkendali"
```

## 11.3 Pseudocode `get_dashboard_reseller`

```txt
resolve no_reseller from auth/profile
validate periode_id
summary = call get_dashboard_reseller(periode_id)
priority_konsumen = query v_ringkasan_konsumen sorted by sisa_bayar desc

return dashboard summary + priority list
```

## 11.4 Flow laporan rekap reseller

```txt
validate periode_id, page, page_size, sort, filter
query v_ringkasan_reseller
apply search server-side
apply filter status lunas
apply pagination
return ReportListEnvelope
```

---

## 12. Rule untuk AI Coding Agent

- dashboard dan laporan harus read-only
- jangan hitung KPI, komisi, saldo, atau stok di frontend
- semua panel harus tunduk ke `periode_id`
- kalau query belum siap, tampilkan state kosong yang jujur atau sembunyikan panel
- jangan isi recent activity dengan dummy yang terlihat seperti produksi
- admin dashboard dan reseller dashboard tidak boleh memakai struktur interaksi yang sama persis
- list laporan besar wajib pagination/search/sort di backend
- beranda reseller first slice tidak bergantung pada unified history feed

---

## 13. Urutan Eksekusi dari Awal sampai Akhir

1. finalkan kontrak dashboard dan report envelope
2. siapkan read model atau RPC pendukung
3. implementasikan endpoint dashboard admin
4. implementasikan UI dashboard admin
5. implementasikan watchlist dan panel progres admin
6. implementasikan endpoint reseller dashboard
7. implementasikan UI reseller dashboard
8. implementasikan konsumen prioritas
9. implementasikan laporan rekap reseller
10. implementasikan laporan stok
11. implementasikan laporan pengiriman
12. implementasikan laporan audit
13. sinkronkan mock, route, dan placeholder export
14. verifikasi performa dan dependency

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

## Risiko

- dashboard mudah menjadi berat jika terlalu banyak query paralel tanpa batas
- laporan mudah menyalin formula bisnis ke frontend jika kontrak read model longgar
- recent activity rawan menjadi area paling ambigu bila kontraknya tidak dikunci

## Asumsi

- dashboard admin desktop-first
- beranda reseller mobile-first
- laporan inti yang paling penting adalah rekap reseller, stok, pengiriman, dan audit
- export/print boleh ditunda setelah layer baca inti stabil

## Pertanyaan yang masih terbuka

- apakah laporan reseller versi admin butuh drill-down ke detail konsumen langsung dari tabel rekap (`UX-003`, status `deferred`)
