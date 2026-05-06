# Execution Module Gudang
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `gudang`.

Dokumen ini memecah implementasi modul gudang ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/prd.md`
- `docs/product_truth_audit.md`
- `docs/data_flow.md`
- `docs/business_contracts.md`
- `docs/schema_mapping.md`
- `docs/query_contracts.md`
- `docs/edge_cases.md`
- `docs/support/system_maps.md`
- `docs/execution/task_kit_standard.md`
- `docs/modules/program_order.md`
- `docs/modules/setoran.md`
- `docs/integration_contract_pack.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`
- `docs/ui/admin_dashboard_uiux.md`

---

## 1. Tujuan Modul

Modul `gudang` bertanggung jawab untuk:

- mencatat pembelian barang melalui `belanja` dan `belanja_detail`
- mengubah bahan mentah menjadi stok siap kirim melalui `packing`
- memproses pengiriman order dengan validasi stok yang benar
- mencatat pembagian paket dari pusat ke reseller
- menampilkan posisi stok barang, stok paket jadi, backlog packing, dan status pembagian
- menjaga agar semua perubahan stok berjalan atomic dan bisa diaudit

Modul ini menjadi jembatan antara order aktif dan operasional fisik lapangan.

---

## 2. Scope Modul

### Frontend Mockup Scope

- halaman daftar belanja admin
- form belanja dengan detail item dinamis
- halaman packing paket komposit
- halaman pengiriman order
- halaman pembagian paket ke reseller
- tabel stok barang dan stok paket jadi
- badge status item pembagian
- flow koreksi operasional terbatas di level UI state

### Backend/Core Scope

- schema `belanja`, `belanja_detail`, `packing`, `pembagian_paket`, `pembagian_paket_detail`
- boundary `buat_belanja`
- boundary `buat_packing`
- boundary `proses_kirim_detail_pesanan`
- boundary `buat_pembagian_paket`
- boundary `serahkan_pembagian_paket`
- read model `v_stok_barang`, `v_stok_paket_jadi`, `v_detail_pesanan_aktif`, `v_pembagian_reseller`
- validasi saldo kas belanja, stok BOM, stok paket jadi, dan eligibility detail pesanan untuk pembagian
- alur koreksi admin untuk salah input belanja, packing, atau pembagian
- batch pembagian final harus ditolak jika masih mencampur item `SIAP` dan `KURANG`

---

## 3. Integration Contract Modul Gudang

## 3.1 Entity Shape Belanja

```ts
type BelanjaRecord = {
  id: number;
  periode_id: number;
  tanggal: string;
  supplier: string;
  no_bukti: string;
  akun_kas_id: number;
  total_belanja: number;
  created_at?: string;
};

type BelanjaItem = {
  id?: number;
  id_barang: string;
  nama_barang?: string;
  harga: number;
  jumlah: number;
  subtotal?: number;
};
```

## 3.2 Entity Shape Packing

```ts
type PackingRecord = {
  id: number;
  periode_id: number;
  tanggal: string;
  paket_id: number;
  nama_paket?: string;
  jumlah_packing: number;
  stok_paket_jadi_terbaru?: number;
};
```

## 3.3 Entity Shape Pengiriman Order

```ts
type KirimOrderResult = {
  order_id: number;
  periode_id: number;
  paket_id: number;
  nama_paket: string;
  no_reseller: string;
  nama_konsumen: string;
  status_kirim: 'SUDAH';
  tgl_dikirim: string;
  sumber_stok: 'PAKET_JADI' | 'BARANG_TUNGGAL';
};
```

## 3.4 Entity Shape Pembagian

```ts
type PembagianRecord = {
  id: number;
  periode_id: number;
  no_reseller: string;
  tanggal: string;
  status: 'DRAFT' | 'DISIAPKAN' | 'DISERAHKAN' | 'SELESAI' | 'BATAL';
  catatan?: string | null;
};

type PembagianItem = {
  detail_pesanan_konsumen_id: number;
  paket_id: number;
  nama_paket?: string;
  status_item: 'SIAP' | 'DISERAHKAN' | 'KURANG' | 'DIGANTI';
  catatan?: string | null;
};
```

## 3.5 Read Model Shape

```ts
type StokBarangSummary = {
  periode_id: number;
  id_barang: string;
  nama_barang: string;
  kategori?: string;
  stok_awal: number;
  jumlah_belanja: number;
  jumlah_dipacking: number;
  jumlah_kirim_tunggal: number;
  stok_tersedia: number;
  jumlah_kebutuhan: number;
  harus_belanja: number;
  stok_lebih: number;
};

type StokPaketJadiSummary = {
  periode_id: number;
  paket_id: number;
  kode?: string;
  nama_paket: string;
  jumlah_packing: number;
  jumlah_terkirim: number;
  stok_paket_jadi: number;
  harus_packing: number;
};

type PembagianResellerSummary = {
  pembagian_id: number;
  periode_id: number;
  no_reseller: string;
  nama_reseller: string;
  tanggal: string;
  status: 'DRAFT' | 'DISIAPKAN' | 'DISERAHKAN' | 'SELESAI' | 'BATAL';
  jumlah_order: number;
  jumlah_siap: number;
  jumlah_diserahkan: number;
  jumlah_kurang: number;
  catatan?: string | null;
};
```

## 3.6 Request Contract

```ts
type CreateBelanjaRequest = {
  periode_id: number;
  tanggal: string;
  supplier: string;
  no_bukti: string;
  akun_kas_id: number;
  items: Array<{
    id_barang: string;
    harga: number;
    jumlah: number;
  }>;
};

type CreatePackingRequest = {
  periode_id: number;
  tanggal: string;
  paket_id: number;
  jumlah_packing: number;
};

type ProsesKirimOrderRequest = {
  periode_id: number;
  order_id: number;
  tanggal?: string | null;
};

type CreatePembagianRequest = {
  periode_id: number;
  no_reseller: string;
  tanggal: string;
  catatan?: string | null;
  items: Array<{
    detail_pesanan_konsumen_id: number;
    status_item?: 'SIAP' | 'KURANG' | 'DIGANTI';
    catatan?: string | null;
  }>;
};

type SerahkanPembagianRequest = {
  periode_id: number;
  pembagian_id: number;
  tanggal: string;
  catatan?: string | null;
};
```

## 3.7 Error Codes Relevan

```txt
BELANJA_INVALID_ITEM
BELANJA_EMPTY_ITEMS
BELANJA_SALDO_KAS_TIDAK_CUKUP
BELANJA_BARANG_NOT_FOUND
PACKING_INVALID_PACKAGE
PACKING_STOK_BAHAN_TIDAK_CUKUP
PACKING_PACKAGE_NOT_COMPOSITE
ORDER_NOT_FOUND
ORDER_ALREADY_SHIPPED
ORDER_CANNOT_CANCEL_AFTER_SHIP
ORDER_STOK_PAKET_JADI_NOT_READY
ORDER_STOK_BARANG_NOT_READY
PEMBAGIAN_NOT_FOUND
PEMBAGIAN_EMPTY_ITEMS
PEMBAGIAN_ORDER_NOT_ELIGIBLE
PEMBAGIAN_ALREADY_DISERAHKAN
KOREKSI_REQUIRES_REASON
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
```

---

## 4. Struktur Folder Modul Gudang

## 4.1 Frontend

```txt
src/app/(DashboardLayout)/admin/gudang/belanja/
  page.tsx
  create/
    page.tsx
  [id]/
    page.tsx

src/app/(DashboardLayout)/admin/gudang/packing/
  page.tsx
  create/
    page.tsx

src/app/(DashboardLayout)/admin/gudang/pengiriman/
  page.tsx
  [id]/
    page.tsx

src/app/(DashboardLayout)/admin/gudang/pembagian/
  page.tsx
  create/
    page.tsx
  [id]/
    page.tsx

src/app/(DashboardLayout)/admin/laporan/stok/
  barang/
    page.tsx
  paket-jadi/
    page.tsx

src/app/components/admin/gudang/
  BelanjaListScreen.tsx
  BelanjaForm.tsx
  BelanjaItemTable.tsx
  BelanjaDetailScreen.tsx
  PackingListScreen.tsx
  PackingForm.tsx
  PengirimanOrderScreen.tsx
  OrderKirimTable.tsx
  PembagianListScreen.tsx
  PembagianForm.tsx
  PembagianDetailScreen.tsx
  PembagianItemsTable.tsx
  StokBarangTable.tsx
  StokPaketJadiTable.tsx
  GudangAlertPanel.tsx
  index.ts

src/hooks/admin/
  useBelanjaListMock.ts
  useBelanjaActionsMock.ts
  usePackingListMock.ts
  usePackingActionsMock.ts
  usePengirimanOrderMock.ts
  usePembagianListMock.ts
  usePembagianActionsMock.ts
  useStokBarangMock.ts
  useStokPaketJadiMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  004_views_ringkasan.sql
  009_functions_gudang.sql
  012_triggers_audit.sql

src/lib/server/services/
  gudang.service.ts

src/lib/server/validation/
  gudang.validation.ts

src/lib/server/workflows/
  gudang.workflow.ts

src/app/api/admin/gudang/belanja/
  route.ts

src/app/api/admin/gudang/packing/
  route.ts

src/app/api/admin/gudang/pengiriman/
  route.ts

src/app/api/admin/gudang/pembagian/
  route.ts

src/app/api/admin/laporan/stok/
  barang/route.ts
  paket-jadi/route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `admin/gudang/belanja/page.tsx` | Entry route daftar belanja |
| `admin/gudang/belanja/create/page.tsx` | Entry route form belanja |
| `admin/gudang/belanja/[id]/page.tsx` | Detail belanja dan audit ringkas |
| `admin/gudang/packing/page.tsx` | Daftar aktivitas packing |
| `admin/gudang/packing/create/page.tsx` | Form packing komposit |
| `admin/gudang/pengiriman/page.tsx` | Daftar order belum/sudah kirim |
| `admin/gudang/pengiriman/[id]/page.tsx` | Detail order untuk proses kirim |
| `admin/gudang/pembagian/page.tsx` | Daftar batch pembagian |
| `admin/gudang/pembagian/create/page.tsx` | Form batch pembagian reseller |
| `admin/gudang/pembagian/[id]/page.tsx` | Detail pembagian dan status item |
| `admin/laporan/stok/barang/page.tsx` | Tabel stok barang |
| `admin/laporan/stok/paket-jadi/page.tsx` | Tabel stok paket jadi |
| `BelanjaListScreen.tsx` | Screen utama daftar belanja |
| `BelanjaForm.tsx` | Form header dan detail belanja |
| `BelanjaItemTable.tsx` | Tabel item belanja editable |
| `BelanjaDetailScreen.tsx` | Detail belanja dan saldo kas sesudah transaksi |
| `PackingListScreen.tsx` | Daftar packing dan backlog |
| `PackingForm.tsx` | Form input packing |
| `PengirimanOrderScreen.tsx` | Screen kirim order |
| `OrderKirimTable.tsx` | Tabel order siap/belum siap kirim |
| `PembagianListScreen.tsx` | Daftar pembagian per reseller |
| `PembagianForm.tsx` | Form draft pembagian |
| `PembagianDetailScreen.tsx` | Detail status batch pembagian |
| `PembagianItemsTable.tsx` | Tabel detail item pembagian |
| `StokBarangTable.tsx` | Tabel stok barang dan indikator belanja |
| `StokPaketJadiTable.tsx` | Tabel stok paket komposit |
| `GudangAlertPanel.tsx` | Panel backlog, stok kritis, pembagian belum selesai |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Definisi tabel belanja, packing, pembagian |
| `004_views_ringkasan.sql` | View stok, order aktif, dan pembagian reseller |
| `009_functions_gudang.sql` | Function atomic untuk belanja, packing, kirim, pembagian |
| `012_triggers_audit.sql` | Audit log dan koreksi gudang |
| `gudang.service.ts` | Service wrapper gudang |
| `gudang.validation.ts` | Validasi payload gudang |
| `gudang.workflow.ts` | Orkestrasi rule stok dan eligibility |
| `admin/gudang/belanja/route.ts` | API belanja admin |
| `admin/gudang/packing/route.ts` | API packing admin |
| `admin/gudang/pengiriman/route.ts` | API pengiriman admin |
| `admin/gudang/pembagian/route.ts` | API pembagian admin |
| `admin/laporan/stok/barang/route.ts` | API read stok barang |
| `admin/laporan/stok/paket-jadi/route.ts` | API read stok paket jadi |

---

## 6. Hubungan Antar File

- `BelanjaForm.tsx` mengirim payload `CreateBelanjaRequest` ke `admin/gudang/belanja/route.ts`
- `admin/gudang/belanja/route.ts` memanggil `gudang.validation.ts`, lalu `gudang.service.ts`
- `gudang.service.ts` meneruskan eksekusi atomic ke `gudang.workflow.ts` dan function `buat_belanja`
- `PackingForm.tsx` memakai `usePackingActionsMock.ts` pada mock stage dan meniru shape `CreatePackingRequest`
- `PengirimanOrderScreen.tsx` membaca daftar dari `v_detail_pesanan_aktif` melalui `admin/pengiriman/route.ts`
- `PembagianForm.tsx` memakai order yang sudah `status_kirim = 'SUDAH'` sebagai kandidat item
- `GudangAlertPanel.tsx` menggabungkan data dari `v_stok_barang`, `v_stok_paket_jadi`, dan `v_pembagian_reseller`
- `gudang.workflow.ts` menjadi boundary utama agar UI tidak menghitung stok atau eligibility sendiri

---

## 7. Breakdown Function/Component/Class per File

## 7.1 Frontend Breakdown

### `BelanjaForm.tsx`

- `BelanjaForm`
- `handleAddItemRow`
- `handleChangeItem`
- `handleRemoveItem`
- `calculateDraftTotal`
- `handleSubmit`

### `PackingForm.tsx`

- `PackingForm`
- `handleSelectPaket`
- `handleChangeJumlahPacking`
- `handleSubmit`

### `PengirimanOrderScreen.tsx`

- `PengirimanOrderScreen`
- `handleFilterStatus`
- `handleProcessShip`
- `renderStockSourceBadge`

### `PembagianForm.tsx`

- `PembagianForm`
- `handleSelectReseller`
- `handleToggleOrderItem`
- `handleUpdateItemStatus`
- `handleSubmitDraft`

### `GudangAlertPanel.tsx`

- `GudangAlertPanel`
- `buildAlertItems`
- `renderAlertState`

## 7.2 Backend Breakdown

### `gudang.validation.ts`

- `validateCreateBelanjaRequest`
- `validateCreatePackingRequest`
- `validateProsesKirimOrderRequest`
- `validateCreatePembagianRequest`
- `validateSerahkanPembagianRequest`

### `gudang.service.ts`

- `createBelanja`
- `createPacking`
- `prosesKirimOrder`
- `createPembagianPaket`
- `serahkanPembagianPaket`
- `getStokBarang`
- `getStokPaketJadi`
- `getPembagianList`

### `gudang.workflow.ts`

- `assertBelanjaKasSufficient`
- `assertPackingPackageComposite`
- `assertPackingBahanSufficient`
- `assertOrderCanBeShipped`
- `resolveSumberStokKirim`
- `assertPembagianItemsEligible`
- `assertPembagianCanBeDelivered`

### `009_functions_gudang.sql`

- `buat_belanja(...)`
- `buat_packing(...)`
- `proses_kirim_detail_pesanan(...)`
- `buat_pembagian_paket(...)`
- `serahkan_pembagian_paket(...)`
- `koreksi_belanja(...)`
- `koreksi_packing(...)`
- `koreksi_pembagian_paket(...)`

---

## 8. First Working Slice

Slice pertama modul gudang harus sesederhana mungkin tetapi menutup satu alur fisik nyata:

1. admin input `belanja`
2. stok barang bertambah di `v_stok_barang`
3. admin input `packing` untuk paket komposit
4. stok paket jadi terbaca di `v_stok_paket_jadi`
5. admin proses kirim order
6. admin buat batch pembagian
7. admin tandai pembagian `DISERAHKAN`

Yang belum wajib di slice pertama:

- koreksi gudang lengkap
- penggantian item pembagian yang kompleks
- bulk processing lintas reseller
- dashboard gudang penuh

---

## 9. Task Execution Batches

### Batch G1 - Stok Foundation

- MGD-01
- MGD-02
- MGD-03
- MGD-04

### Batch G2 - Belanja dan Packing

- MGD-05
- MGD-06
- MGD-07
- MGD-08

### Batch G3 - Pengiriman dan Pembagian

- MGD-09
- MGD-10
- MGD-11
- MGD-12
- MGD-13

### Batch G4 - Read Model, Audit, dan Hardening

- MGD-14
- MGD-15
- MGD-16
- MGD-17

---

## 10. Task Implementation Detail

## MGD-01 - Finalkan schema gudang

- tujuan: memastikan tabel gudang dan relasinya lengkap untuk belanja, packing, dan pembagian
- file yang dibuat/diubah: `001_init_schema.sql`
- lokasi file: `supabase/migrations/001_init_schema.sql`
- langkah kerja AI agent:
  - cek definisi tabel yang sudah ada
  - tambahkan kolom wajib yang belum ada
  - validasi FK `periode_id`, `paket_id`, `detail_pesanan_konsumen_id`, `akun_kas_id`
  - tambahkan enum status pembagian dan status item
- dependency: `modules/program_order.md`, `modules/setoran.md`
- output yang diharapkan: schema siap untuk RPC gudang
- acceptance criteria:
  - semua tabel gudang punya PK/FK jelas
  - semua transaksi gudang membawa `periode_id`
  - status enum pembagian sesuai kontrak

## MGD-02 - Buat read model stok barang

- tujuan: menyediakan satu sumber baca untuk keputusan belanja
- file yang dibuat/diubah: `004_views_ringkasan.sql`
- lokasi file: `supabase/migrations/004_views_ringkasan.sql`
- langkah kerja AI agent:
  - buat atau lengkapi view `v_stok_barang`
  - pastikan formula belanja, packing, dan kirim tunggal sesuai query contract
  - exclude `status_kirim = 'BATAL'` pada kebutuhan
- dependency: MGD-01
- output yang diharapkan: `v_stok_barang`
- acceptance criteria:
  - kolom `harus_belanja` dan `stok_tersedia` tersedia
  - hasil view bisa dipakai langsung frontend tanpa formula tambahan

## MGD-03 - Buat read model stok paket jadi

- tujuan: menyediakan dasar validasi pengiriman komposit
- file yang dibuat/diubah: `004_views_ringkasan.sql`
- lokasi file: `supabase/migrations/004_views_ringkasan.sql`
- langkah kerja AI agent:
  - buat atau lengkapi view `v_stok_paket_jadi`
  - hitung `stok_paket_jadi` dari packing minus order terkirim
  - hitung `harus_packing`
- dependency: MGD-01
- output yang diharapkan: `v_stok_paket_jadi`
- acceptance criteria:
  - hanya paket komposit yang muncul
  - `jumlah_terkirim` hanya menghitung `status_kirim = 'SUDAH'`

## MGD-04 - Buat daftar order siap kirim dan pembagian

- tujuan: memastikan pengiriman dan pembagian membaca kandidat detail pesanan yang sama
- file yang dibuat/diubah: `004_views_ringkasan.sql`
- lokasi file: `supabase/migrations/004_views_ringkasan.sql`
- langkah kerja AI agent:
  - pastikan `v_detail_pesanan_aktif` memuat `perlu_packing`, `status_kirim`, `status_item`
  - buat atau lengkapi `v_pembagian_reseller`
  - siapkan kolom jumlah item siap, diserahkan, kurang
- dependency: MGD-01
- output yang diharapkan: read model pengiriman dan pembagian
- acceptance criteria:
  - admin bisa filter detail pesanan `BELUM` dan `SUDAH`
  - pembagian punya summary per batch
  - batch final tidak boleh lolos jika masih ada campuran item `SIAP` dan `KURANG`

## MGD-05 - Implementasi RPC belanja

- tujuan: mencatat pembelian barang secara atomic dan aman terhadap saldo kas
- file yang dibuat/diubah: `009_functions_gudang.sql`, `gudang.validation.ts`, `gudang.service.ts`, `gudang.workflow.ts`
- lokasi file: `supabase/migrations/009_functions_gudang.sql`, `src/lib/server/validation/gudang.validation.ts`, `src/lib/server/services/gudang.service.ts`, `src/lib/server/workflows/gudang.workflow.ts`
- langkah kerja AI agent:
  - validasi item minimal satu
  - hitung total belanja dari detail
  - cek `v_saldo_kas` atau helper saldo kas sumber
  - insert header dan detail dalam satu transaksi
  - kembalikan saldo kas terbaru
- dependency: MGD-01, `modules/keuangan.md` akan memakai hasil saldo
- output yang diharapkan: function `buat_belanja`
- acceptance criteria:
  - gagal jika saldo kas kurang
  - gagal jika ada item invalid
  - sukses mengembalikan total dan saldo terbaru

## MGD-06 - Implementasi UI dan hook mock belanja

- tujuan: menyiapkan alur admin input belanja dari screen sampai hook mock yang meniru kontrak route internal
- file yang dibuat/diubah: `admin/gudang/belanja/page.tsx`, `admin/gudang/belanja/create/page.tsx`, `BelanjaListScreen.tsx`, `BelanjaForm.tsx`, `BelanjaItemTable.tsx`, `useBelanjaListMock.ts`, `useBelanjaActionsMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun list screen dan form screen
  - buat row dinamis item belanja
  - pakai dummy data yang meniru shape kontrak
  - sediakan action mock yang meniru payload POST ke route internal
- dependency: MGD-05
- output yang diharapkan: UI mock belanja lengkap
- acceptance criteria:
  - admin bisa tambah/hapus item form
  - total draft tampil dari state UI
  - payload yang dikirim sesuai `CreateBelanjaRequest`

## MGD-07 - Implementasi RPC packing

- tujuan: mencatat packing komposit dengan validasi BOM dan stok bahan
- file yang dibuat/diubah: `009_functions_gudang.sql`, `gudang.validation.ts`, `gudang.service.ts`, `gudang.workflow.ts`
- lokasi file: sama dengan MGD-05
- langkah kerja AI agent:
  - validasi paket `perlu_packing = TRUE`
  - ambil BOM `detail_paket`
  - hitung kebutuhan bahan
  - tolak jika salah satu bahan kurang
  - insert packing dan return stok paket jadi terbaru
- dependency: MGD-02, MGD-03
- output yang diharapkan: function `buat_packing`
- acceptance criteria:
  - paket tunggal ditolak
  - stok bahan kurang ditolak dengan pesan jelas
  - sukses mengembalikan stok paket jadi terbaru

## MGD-08 - Implementasi UI packing dan backlog

- tujuan: memberi admin area kerja packing yang cepat dan jelas
- file yang dibuat/diubah: `admin/packing/page.tsx`, `admin/packing/create/page.tsx`, `PackingListScreen.tsx`, `PackingForm.tsx`, `usePackingListMock.ts`, `usePackingActionsMock.ts`, `StokPaketJadiTable.tsx`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan daftar paket dengan `harus_packing`
  - sediakan form input jumlah packing
  - tampilkan stok paket jadi setelah submit mock
- dependency: MGD-07
- output yang diharapkan: UI mock packing
- acceptance criteria:
  - backlog packing terlihat jelas
  - form menolak input kosong/0 di level UI
  - response mock meniru `PackingRecord`

## MGD-09 - Implementasi RPC pengiriman order

- tujuan: memproses pengiriman order berdasarkan tipe paket dan stok aktual
- file yang dibuat/diubah: `009_functions_gudang.sql`, `gudang.validation.ts`, `gudang.service.ts`, `gudang.workflow.ts`
- lokasi file: sama
- langkah kerja AI agent:
  - cek order ada dan belum `SUDAH`
  - cek `paket.perlu_packing`
  - jika komposit, cek `v_stok_paket_jadi > 0`
  - jika tunggal, cek `v_stok_barang` komponen tunggal cukup
  - update `status_kirim` dan `tgl_dikirim`
- dependency: MGD-03, MGD-04, `modules/program_order.md`
- output yang diharapkan: function `proses_kirim_detail_pesanan`
- acceptance criteria:
  - order `SUDAH` tidak bisa dikirim ulang
  - komposit tanpa stok paket jadi ditolak
  - tunggal tanpa stok barang ditolak

## MGD-10 - Implementasi UI pengiriman

- tujuan: memberi admin daftar order siap kirim dan aksi kirim yang aman
- file yang dibuat/diubah: `admin/pengiriman/page.tsx`, `admin/pengiriman/[id]/page.tsx`, `PengirimanOrderScreen.tsx`, `OrderKirimTable.tsx`, `usePengirimanOrderMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan filter `BELUM` dan `SUDAH`
  - buat aksi kirim per baris atau detail
  - tampilkan badge sumber stok
- dependency: MGD-09
- output yang diharapkan: UI pengiriman mock
- acceptance criteria:
  - daftar order memuat paket, reseller, konsumen, status kirim
  - aksi kirim menghasilkan state sukses/gagal yang jelas

## MGD-11 - Implementasi RPC pembagian paket

- tujuan: membuat batch pembagian paket ke reseller setelah order dikirim
- file yang dibuat/diubah: `009_functions_gudang.sql`, `gudang.validation.ts`, `gudang.service.ts`, `gudang.workflow.ts`
- lokasi file: sama
- langkah kerja AI agent:
  - validasi item minimal satu
  - validasi semua order milik reseller yang sama dan sudah eligible
  - insert header dan detail pembagian
  - set status awal `DISIAPKAN` atau `DRAFT` sesuai payload
- dependency: MGD-09
- output yang diharapkan: function `buat_pembagian_paket`
- acceptance criteria:
  - order yang belum layak tidak bisa masuk batch
  - batch punya header dan detail konsisten

## MGD-12 - Implementasi RPC serah terima pembagian

- tujuan: menandai batch pembagian sudah benar-benar diserahkan
- file yang dibuat/diubah: `009_functions_gudang.sql`, `gudang.validation.ts`, `gudang.service.ts`, `gudang.workflow.ts`
- lokasi file: sama
- langkah kerja AI agent:
  - validasi batch ada dan belum `DISERAHKAN`
  - update status header
  - update item `SIAP` menjadi `DISERAHKAN` jika belum punya status akhir
  - catat audit
- dependency: MGD-11
- output yang diharapkan: function `serahkan_pembagian_paket`
- acceptance criteria:
  - batch yang sudah diserahkan tidak bisa diserahkan ulang
  - status summary `v_pembagian_reseller` ikut berubah

## MGD-13 - Implementasi UI pembagian reseller

- tujuan: memberi admin flow draft sampai serah terima
- file yang dibuat/diubah: `admin/pembagian/page.tsx`, `admin/pembagian/create/page.tsx`, `admin/pembagian/[id]/page.tsx`, `PembagianListScreen.tsx`, `PembagianForm.tsx`, `PembagianDetailScreen.tsx`, `PembagianItemsTable.tsx`, `usePembagianListMock.ts`, `usePembagianActionsMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - buat daftar batch
  - buat form pilih reseller dan daftar order
  - buat detail batch dengan aksi serahkan
- dependency: MGD-11, MGD-12
- output yang diharapkan: UI pembagian lengkap
- acceptance criteria:
  - admin bisa membuat batch dari order eligible
  - detail batch menampilkan status item
  - aksi serahkan mengubah state batch

## MGD-14 - Implementasi halaman stok dan alert panel

- tujuan: menjadikan backlog operasional gudang mudah discan
- file yang dibuat/diubah: `admin/stok/barang/page.tsx`, `admin/stok/paket-jadi/page.tsx`, `StokBarangTable.tsx`, `StokPaketJadiTable.tsx`, `GudangAlertPanel.tsx`, `useStokBarangMock.ts`, `useStokPaketJadiMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan tabel stok barang
  - tampilkan tabel stok paket jadi
  - buat panel alert untuk `harus_belanja`, `harus_packing`, dan pembagian belum selesai
- dependency: MGD-02, MGD-03, MGD-04
- output yang diharapkan: halaman stok operasional
- acceptance criteria:
  - warna/status backlog mudah dikenali
  - tidak ada formula bisnis kompleks di komponen

## MGD-15 - Tambahkan audit dan koreksi gudang

- tujuan: menutup kebutuhan salah input operasional
- file yang dibuat/diubah: `009_functions_gudang.sql`, `012_triggers_audit.sql`
- lokasi file: `supabase/migrations/...`
- langkah kerja AI agent:
  - sediakan RPC `koreksi_belanja`, `koreksi_packing`, `koreksi_pembagian_paket`
  - wajibkan alasan koreksi
  - tulis ke `koreksi_transaksi` dan `audit_log`
- dependency: MGD-05, MGD-07, MGD-11
- output yang diharapkan: jalur koreksi admin
- acceptance criteria:
  - koreksi tanpa alasan ditolak
  - riwayat koreksi bisa dilacak

## MGD-16 - Verifikasi edge case gudang

- tujuan: memastikan perilaku gudang mengikuti keputusan owner
- file yang dibuat/diubah: `gudang.workflow.ts`, test/spec bila tersedia
- lokasi file: `src/lib/server/workflows/gudang.workflow.ts`
- langkah kerja AI agent:
  - uji salah input belanja/packing
  - uji pembagian kurang atau diganti
  - uji order yang sudah dikirim tidak bisa dibatalkan biasa
- dependency: MGD-09, MGD-15
- output yang diharapkan: rule edge case stabil
- acceptance criteria:
  - EC-014 dan EC-015 tercermin dalam workflow
  - pengiriman tidak membuka celah double-send

## MGD-17 - Rapikan folder exports dan integrasi modul

- tujuan: memastikan modul gudang mudah dipanggil modul dashboard/laporan nanti
- file yang dibuat/diubah: `src/app/components/admin/gudang/index.ts`, route index bila perlu, docs terkait
- lokasi file: `src/app/components/admin/gudang/index.ts`
- langkah kerja AI agent:
  - ekspor komponen inti
  - rapikan import path
  - cek konsistensi naming screen/table/form
- dependency: semua task UI gudang
- output yang diharapkan: modul gudang stabil untuk integrasi
- acceptance criteria:
  - tidak ada import path kacau
  - komponen mudah dipakai ulang

---

## 11. Pseudocode Task Penting

## 11.1 Pseudocode `buat_belanja`

```txt
validate admin role
validate periode aktif
validate items minimal 1

total = sum(item.harga * item.jumlah)
saldo = getSaldoKas(akun_kas_id, periode_id)

if saldo < total:
  reject BELANJA_SALDO_KAS_TIDAK_CUKUP

begin transaction
  insert belanja header
  insert belanja_detail items
  write audit_log
commit

return belanja + items + saldo_terbaru
```

## 11.2 Pseudocode `buat_packing`

```txt
validate admin role
validate periode aktif
validate paket composite
validate jumlah_packing > 0

bom = getDetailPaket(paket_id)
for each bom item:
  need = bom.jumlah * jumlah_packing
  stok = getStokBarangTersedia(id_barang, periode_id)
  if stok < need:
    reject PACKING_STOK_BAHAN_TIDAK_CUKUP

insert packing
return stok_paket_jadi_terbaru
```

## 11.3 Flow `proses_kirim_detail_pesanan`

```txt
load order + paket
if order.status_kirim == SUDAH:
  reject ORDER_ALREADY_SHIPPED

if paket.perlu_packing:
  check v_stok_paket_jadi > 0
else:
  check v_stok_barang komponen tunggal cukup

update order.status_kirim = SUDAH
update order.tgl_dikirim = tanggal_input_or_today
write audit

return hasil kirim
```

## 11.4 Flow `serahkan_pembagian_paket`

```txt
load pembagian header + items
if header.status in (DISERAHKAN, SELESAI):
  reject PEMBAGIAN_ALREADY_DISERAHKAN

update detail item SIAP -> DISERAHKAN
update header.status = DISERAHKAN
write audit

return summary batch terbaru
```

---

## 12. Rule untuk AI Coding Agent

- jangan hitung stok final di frontend
- jangan izinkan UI memproses kirim tanpa response backend yang berhasil
- bedakan jelas paket komposit dan tunggal
- pembagian hanya memakai order yang sudah melewati rule eligibility yang sama
- koreksi gudang tidak boleh edit transaksi asli diam-diam
- semua query summary harus scope `periode_id`
- semua angka uang dan stok harus berasal dari read model atau response backend, bukan cache lokal semata

---

## 13. Urutan Eksekusi dari Awal sampai Akhir

1. finalkan schema gudang
2. buat view stok dan pembagian
3. implementasikan RPC belanja
4. implementasikan UI belanja
5. implementasikan RPC packing
6. implementasikan UI packing
7. implementasikan RPC pengiriman
8. implementasikan UI pengiriman
9. implementasikan RPC pembagian
10. implementasikan UI pembagian
11. implementasikan halaman stok
12. tambahkan audit dan koreksi
13. verifikasi edge case

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

## Risiko

- stok bisa salah bila rule paket tunggal vs komposit bercampur
- pengiriman dan pembagian rawan double action bila status tidak dijaga ketat
- koreksi gudang yang terlambat bisa memengaruhi laporan stok dan kas

## Asumsi

- pembagian paket hanya untuk order yang sudah siap serah secara operasional
- belanja selalu admin-only
- pengiriman order dilakukan satu per satu lebih dulu pada slice pertama
- batch pembagian boleh dibuat setelah sebagian order reseller siap

## Catatan Keputusan Terkunci

- status `SELESAI` pada pembagian tetap aksi manual admin sesuai `BQ-004` / `docs/support/decision_log.md`
- pengiriman massal per reseller tetap ditunda; first slice memakai alur satu per satu
