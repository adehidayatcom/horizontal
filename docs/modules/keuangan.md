# Execution Module Keuangan
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `keuangan`.

Dokumen ini memecah implementasi modul keuangan ke dua area yang terkoordinasi:

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
- `docs/modules/setoran.md`
- `docs/modules/gudang.md`
- `docs/integration_contract_pack.md`
- `docs/execution/frontend_plan.md`
- `docs/execution/backend_plan.md`
- `docs/ui/admin_dashboard_uiux.md`

---

## 1. Tujuan Modul

Modul `keuangan` bertanggung jawab untuk:

- mengelola `akun_kas` sebagai sumber dan tujuan transaksi kas
- menjaga `akun_kas` aktif/nonaktif tanpa menghapus histori transaksi lama
- mencatat `kas_masuk` untuk pemasukan eksternal di luar setoran reseller
- mencatat `mutasi_kas` antar akun kas
- mencatat `pencairan` tabungan atau komisi reseller
- menampilkan posisi saldo kas yang konsisten per `periode_id`, termasuk saat admin membuka periode historis yang berbeda
- menjaga agar belanja, mutasi, dan pencairan tidak melewati saldo kas yang tersedia

Modul ini adalah pusat kontrol likuiditas dan penyelesaian hak reseller.

---

## 2. Scope Modul

### Frontend Mockup Scope

- halaman daftar akun kas
- halaman kas masuk
- halaman mutasi kas
- halaman pencairan reseller
- halaman ringkasan saldo kas
- filter periode dan akun kas
- form nominal, metode, dan catatan
- state disabled pencairan jika reseller belum lunas

### Backend/Core Scope

- schema `akun_kas`, `kas_masuk`, `mutasi_kas`, `pencairan`
- boundary `buat_kas_masuk`
- boundary `buat_mutasi_kas`
- boundary `buat_pencairan`
- read model `v_saldo_kas`, `v_ringkasan_reseller`
- validasi saldo kas sumber
- validasi reseller `LUNAS` sebelum pencairan
- validasi sisa tabungan dan sisa komisi

---

## 3. Integration Contract Modul Keuangan

## 3.1 Entity Shape Akun Kas

```ts
type AkunKasRecord = {
  id: number;
  jenis: string;
  nama_akun?: string | null;
  saldo_awal: number;
  aktif: boolean;
};
```

## 3.2 Entity Shape Kas Masuk

```ts
type KasMasukRecord = {
  id: number;
  periode_id: number;
  tanggal: string;
  akun_kas_id: number;
  sumber: 'MODAL_OWNER' | 'PENYESUAIAN' | 'LAINNYA';
  nominal: number;
  keterangan?: string | null;
};
```

## 3.3 Entity Shape Mutasi Kas

```ts
type MutasiKasRecord = {
  id: number;
  periode_id: number;
  tanggal: string;
  dari_akun_id: number;
  ke_akun_id: number;
  jumlah: number;
  keterangan?: string | null;
};
```

## 3.4 Entity Shape Pencairan

```ts
type PencairanRecord = {
  id: number;
  periode_id: number;
  tanggal: string;
  no_reseller: string;
  jenis: 'TABUNGAN' | 'KOMISI';
  nominal: number;
  metode: 'CASH' | 'TRANSFER';
  akun_kas_id: number;
};
```

## 3.5 Read Model Shape

```ts
type SaldoKasSummary = {
  akun_kas_id: number;
  jenis: string;
  saldo_awal: number;
  total_setoran: number;
  total_kas_masuk: number;
  total_mutasi_masuk: number;
  total_mutasi_keluar: number;
  total_pencairan: number;
  total_belanja: number;
  saldo: number;
};

type PencairanEligibility = {
  periode_id: number;
  no_reseller: string;
  nama_reseller: string;
  status_lunas_reseller: 'BELUM' | 'LUNAS';
  sisa_tabungan: number;
  sisa_komisi: number;
};
```

## 3.6 Request Contract

```ts
type CreateKasMasukRequest = {
  periode_id: number;
  tanggal: string;
  akun_kas_id: number;
  sumber: 'MODAL_OWNER' | 'PENYESUAIAN' | 'LAINNYA';
  nominal: number;
  keterangan?: string | null;
};

type CreateMutasiKasRequest = {
  periode_id: number;
  tanggal: string;
  dari_akun_id: number;
  ke_akun_id: number;
  jumlah: number;
  keterangan?: string | null;
};

type CreatePencairanRequest = {
  periode_id: number;
  tanggal: string;
  no_reseller: string;
  jenis: 'TABUNGAN' | 'KOMISI';
  nominal: number;
  metode: 'CASH' | 'TRANSFER';
  akun_kas_id: number;
};
```

## 3.7 Error Codes Relevan

```txt
AKUN_KAS_NOT_FOUND
KAS_MASUK_INVALID_SOURCE
KAS_MASUK_INVALID_AMOUNT
MUTASI_AKUN_SAMA
MUTASI_SALDO_TIDAK_CUKUP
MUTASI_INVALID_AMOUNT
PENCAIRAN_RESELLER_NOT_LUNAS
PENCAIRAN_SISA_DANA_TIDAK_CUKUP
PENCAIRAN_SALDO_KAS_TIDAK_CUKUP
PENCAIRAN_INVALID_AMOUNT
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
```

---

## 4. Struktur Folder Modul Keuangan

## 4.1 Frontend

```txt
src/app/(DashboardLayout)/admin/keuangan/
  akun-kas/
    page.tsx
  kas-masuk/
    page.tsx
    create/
      page.tsx
  mutasi-kas/
    page.tsx
    create/
      page.tsx
  pencairan/
    page.tsx
    create/
      page.tsx

src/app/components/admin/keuangan/
  AkunKasListScreen.tsx
  KasSaldoSummaryCards.tsx
  KasMasukListScreen.tsx
  KasMasukForm.tsx
  MutasiKasListScreen.tsx
  MutasiKasForm.tsx
  PencairanListScreen.tsx
  PencairanForm.tsx
  PencairanEligibilityPanel.tsx
  SaldoKasTable.tsx
  KeuanganAlertPanel.tsx
  index.ts

src/hooks/admin/
  useAkunKasMock.ts
  useSaldoKasMock.ts
  useKasMasukMock.ts
  useKasMasukActionsMock.ts
  useMutasiKasMock.ts
  useMutasiKasActionsMock.ts
  usePencairanMock.ts
  usePencairanActionsMock.ts
  usePencairanEligibilityMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  004_views_ringkasan.sql
  010_functions_keuangan.sql
  012_triggers_audit.sql

src/lib/server/services/
  keuangan.service.ts

src/lib/server/validation/
  keuangan.validation.ts

src/lib/server/workflows/
  keuangan.workflow.ts

src/app/api/admin/keuangan/
  akun-kas/route.ts
  saldo-kas/route.ts
  kas-masuk/route.ts
  mutasi-kas/route.ts
  pencairan/route.ts
```

---

## 5. Daftar File dan Fungsi Tiap File

## 5.1 Frontend Files

| File | Fungsi |
|---|---|
| `admin/keuangan/akun-kas/page.tsx` | Daftar akun kas |
| `admin/keuangan/kas-masuk/page.tsx` | Daftar kas masuk |
| `admin/keuangan/kas-masuk/create/page.tsx` | Form kas masuk |
| `admin/keuangan/mutasi-kas/page.tsx` | Daftar mutasi kas |
| `admin/keuangan/mutasi-kas/create/page.tsx` | Form mutasi kas |
| `admin/keuangan/pencairan/page.tsx` | Daftar pencairan |
| `admin/keuangan/pencairan/create/page.tsx` | Form pencairan |
| `AkunKasListScreen.tsx` | Screen daftar akun kas |
| `KasSaldoSummaryCards.tsx` | Kartu ringkasan saldo |
| `KasMasukListScreen.tsx` | Screen daftar pemasukan kas |
| `KasMasukForm.tsx` | Form kas masuk |
| `MutasiKasListScreen.tsx` | Screen daftar mutasi |
| `MutasiKasForm.tsx` | Form mutasi kas |
| `PencairanListScreen.tsx` | Screen daftar pencairan |
| `PencairanForm.tsx` | Form pencairan reseller |
| `PencairanEligibilityPanel.tsx` | Panel status lunas dan sisa hak reseller |
| `SaldoKasTable.tsx` | Tabel saldo kas |
| `KeuanganAlertPanel.tsx` | Panel alert saldo rendah dan reseller siap cair |

## 5.2 Backend Files

| File | Fungsi |
|---|---|
| `001_init_schema.sql` | Definisi tabel kas dan pencairan |
| `004_views_ringkasan.sql` | View `v_saldo_kas` dan pendukung pencairan |
| `010_functions_keuangan.sql` | Function kas masuk, mutasi, pencairan |
| `012_triggers_audit.sql` | Audit keuangan dan koreksi |
| `keuangan.service.ts` | Service wrapper keuangan |
| `keuangan.validation.ts` | Validasi payload keuangan |
| `keuangan.workflow.ts` | Orkestrasi rule saldo dan eligibility |
| `akun-kas/route.ts` | API daftar akun kas |
| `saldo-kas/route.ts` | API saldo kas |
| `kas-masuk/route.ts` | API kas masuk |
| `mutasi-kas/route.ts` | API mutasi kas |
| `pencairan/route.ts` | API pencairan |

---

## 6. Hubungan Antar File

- `KasMasukForm.tsx` mengirim `CreateKasMasukRequest` ke `kas-masuk/route.ts`
- `MutasiKasForm.tsx` mengirim `CreateMutasiKasRequest` ke `mutasi-kas/route.ts`
- `PencairanForm.tsx` mengirim `CreatePencairanRequest` ke `pencairan/route.ts`
- semua route keuangan memanggil `keuangan.validation.ts` lalu `keuangan.service.ts`
- `keuangan.service.ts` menggunakan `keuangan.workflow.ts` untuk cek saldo dan eligibility reseller
- `SaldoKasTable.tsx` dan `KasSaldoSummaryCards.tsx` membaca `v_saldo_kas`
- `PencairanEligibilityPanel.tsx` membaca ringkasan reseller untuk memastikan status `LUNAS`
- `KeuanganAlertPanel.tsx` membaca saldo kas dan reseller yang siap pencairan

---

## 7. Breakdown Function/Component/Class per File

## 7.1 Frontend Breakdown

### `KasMasukForm.tsx`

- `KasMasukForm`
- `handleSelectAkunKas`
- `handleSubmit`

### `MutasiKasForm.tsx`

- `MutasiKasForm`
- `handleSelectDariAkun`
- `handleSelectKeAkun`
- `handleSubmit`

### `PencairanForm.tsx`

- `PencairanForm`
- `handleSelectReseller`
- `handleSelectJenis`
- `handleSubmit`

### `PencairanEligibilityPanel.tsx`

- `PencairanEligibilityPanel`
- `renderLunasBadge`
- `renderDanaAvailability`

### `KeuanganAlertPanel.tsx`

- `KeuanganAlertPanel`
- `buildFinanceAlerts`
- `renderAlertCard`

## 7.2 Backend Breakdown

### `keuangan.validation.ts`

- `validateCreateKasMasukRequest`
- `validateCreateMutasiKasRequest`
- `validateCreatePencairanRequest`

### `keuangan.service.ts`

- `getAkunKasList`
- `getSaldoKas`
- `createKasMasuk`
- `createMutasiKas`
- `createPencairan`
- `getPencairanEligibility`

### `keuangan.workflow.ts`

- `assertAkunKasExists`
- `assertKasSourceAmountValid`
- `assertMutasiAccountsDifferent`
- `assertSaldoKasCukup`
- `assertResellerSudahLunas`
- `assertSisaDanaPencairanCukup`

### `010_functions_keuangan.sql`

- `buat_kas_masuk(...)`
- `buat_mutasi_kas(...)`
- `buat_pencairan(...)`
- `koreksi_kas_masuk(...)`
- `koreksi_mutasi_kas(...)`
- `koreksi_pencairan(...)`

---

## 8. First Working Slice

Slice pertama modul keuangan harus menutup satu alur likuiditas minimum:

1. admin melihat `v_saldo_kas`
2. admin mencatat `kas_masuk`
3. saldo akun bertambah
4. admin mencatat `mutasi_kas`
5. admin melihat reseller yang sudah `LUNAS`
6. admin mencairkan `TABUNGAN` atau `KOMISI`

Yang belum wajib di slice pertama:

- koreksi keuangan lengkap
- export laporan kas
- multi-step approval pencairan
- audit visual mendalam di UI

---

## 9. Task Execution Batches

### Batch K1 - Kas Foundation

- MKE-01
- MKE-02
- MKE-03

### Batch K2 - Kas Masuk dan Mutasi

- MKE-04
- MKE-05
- MKE-06
- MKE-07

### Batch K3 - Pencairan Reseller

- MKE-08
- MKE-09
- MKE-10
- MKE-11

### Batch K4 - Hardening dan Audit

- MKE-12
- MKE-13
- MKE-14

---

## 10. Task Implementation Detail

## MKE-01 - Finalkan schema keuangan

- tujuan: memastikan tabel kas dan pencairan siap dipakai lintas periode
- file yang dibuat/diubah: `001_init_schema.sql`
- lokasi file: `supabase/migrations/001_init_schema.sql`
- langkah kerja AI agent:
  - cek definisi `akun_kas`, `kas_masuk`, `mutasi_kas`, `pencairan`
  - tambah constraint nominal positif
  - tambah FK dan index penting
- dependency: `modules/setoran.md`, `modules/gudang.md`
- output yang diharapkan: schema keuangan siap
- acceptance criteria:
  - `akun_kas` lintas periode tetap konsisten
  - transaksi kas/pencairan tetap membawa `periode_id`

## MKE-02 - Lengkapi read model saldo kas

- tujuan: menjadikan saldo kas satu sumber kebenaran untuk UI dan validasi
- file yang dibuat/diubah: `004_views_ringkasan.sql`
- lokasi file: `supabase/migrations/004_views_ringkasan.sql`
- langkah kerja AI agent:
  - buat atau lengkapi `v_saldo_kas`
  - hitung setoran, kas masuk, mutasi masuk/keluar, pencairan, dan belanja
  - gunakan `periode_id` sebagai filter wajib tanpa membuat view saldo kas kedua
- dependency: MKE-01
- output yang diharapkan: `v_saldo_kas`
- acceptance criteria:
  - saldo mengikuti formula query contract
  - bisa dipakai oleh belanja dan pencairan

## MKE-03 - Buat eligibility pencairan reseller

- tujuan: memastikan pencairan membaca status reseller dari sumber yang benar
- file yang dibuat/diubah: `004_views_ringkasan.sql`, `keuangan.workflow.ts`
- lokasi file: `supabase/migrations/004_views_ringkasan.sql`, `src/lib/server/workflows/keuangan.workflow.ts`
- langkah kerja AI agent:
  - gunakan `v_ringkasan_reseller`
  - siapkan helper `getPencairanEligibility`
  - hitung `sisa_tabungan` dan `sisa_komisi`
- dependency: `modules/setoran.md`
- output yang diharapkan: eligibility pencairan konsisten
- acceptance criteria:
  - pencairan hanya melihat reseller lunas
  - sisa dana sesuai ringkasan reseller

## MKE-04 - Implementasi RPC kas masuk

- tujuan: mencatat pemasukan kas resmi di luar setoran reseller
- file yang dibuat/diubah: `010_functions_keuangan.sql`, `keuangan.validation.ts`, `keuangan.service.ts`, `keuangan.workflow.ts`
- lokasi file: `supabase/migrations/010_functions_keuangan.sql`, `src/lib/server/...`
- langkah kerja AI agent:
  - validasi nominal > 0
  - validasi akun kas valid
  - insert transaksi dan audit
- dependency: MKE-01, MKE-02
- output yang diharapkan: function `buat_kas_masuk`
- acceptance criteria:
  - saldo kas bertambah setelah kas masuk
  - sumber wajib sesuai enum

## MKE-05 - Implementasi UI kas masuk

- tujuan: memberi admin jalur cepat menambah likuiditas resmi
- file yang dibuat/diubah: `admin/keuangan/kas-masuk/page.tsx`, `admin/keuangan/kas-masuk/create/page.tsx`, `KasMasukListScreen.tsx`, `KasMasukForm.tsx`, `useKasMasukMock.ts`, `useKasMasukActionsMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - buat list dan form
  - tampilkan akun kas dan sumber pemasukan
  - pastikan payload meniru kontrak final
- dependency: MKE-04
- output yang diharapkan: UI kas masuk
- acceptance criteria:
  - nominal diformat jelas
  - state sukses/gagal mudah dipahami

## MKE-06 - Implementasi RPC mutasi kas

- tujuan: memindahkan saldo antar akun tanpa mengubah total likuiditas
- file yang dibuat/diubah: `010_functions_keuangan.sql`, `keuangan.validation.ts`, `keuangan.service.ts`, `keuangan.workflow.ts`
- lokasi file: sama
- langkah kerja AI agent:
  - validasi akun asal dan tujuan berbeda
  - validasi jumlah > 0
  - cek saldo akun sumber cukup
  - insert transaksi dan audit
- dependency: MKE-02
- output yang diharapkan: function `buat_mutasi_kas`
- acceptance criteria:
  - mutasi akun sama ditolak
  - mutasi saldo kurang ditolak
  - total gabungan antar akun tidak berubah secara konsep

## MKE-07 - Implementasi UI mutasi kas

- tujuan: memberi admin flow aman untuk pindah saldo antar akun
- file yang dibuat/diubah: `admin/keuangan/mutasi-kas/page.tsx`, `admin/keuangan/mutasi-kas/create/page.tsx`, `MutasiKasListScreen.tsx`, `MutasiKasForm.tsx`, `useMutasiKasMock.ts`, `useMutasiKasActionsMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan dua selector akun
  - beri warning jika akun sama
  - tampilkan saldo akun sumber sebagai referensi
- dependency: MKE-06
- output yang diharapkan: UI mutasi kas
- acceptance criteria:
  - user tidak bisa submit dengan akun sama
  - info saldo sumber terlihat

## MKE-08 - Implementasi RPC pencairan

- tujuan: mencairkan hak reseller setelah lunas dan dana cukup
- file yang dibuat/diubah: `010_functions_keuangan.sql`, `keuangan.validation.ts`, `keuangan.service.ts`, `keuangan.workflow.ts`
- lokasi file: sama
- langkah kerja AI agent:
  - validasi reseller `LUNAS`
  - validasi `jenis` tabungan/komisi
  - validasi nominal <= sisa dana sesuai jenis
  - validasi saldo akun kas cukup
  - insert pencairan dan audit
- dependency: MKE-02, MKE-03, `modules/setoran.md`
- output yang diharapkan: function `buat_pencairan`
- acceptance criteria:
  - reseller belum lunas ditolak
  - nominal melebihi sisa dana ditolak
  - saldo kas kurang ditolak

## MKE-09 - Implementasi UI pencairan dan eligibility

- tujuan: memberi admin halaman pencairan yang tidak membuka transaksi salah
- file yang dibuat/diubah: `admin/keuangan/pencairan/page.tsx`, `admin/keuangan/pencairan/create/page.tsx`, `PencairanListScreen.tsx`, `PencairanForm.tsx`, `PencairanEligibilityPanel.tsx`, `usePencairanMock.ts`, `usePencairanActionsMock.ts`, `usePencairanEligibilityMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan daftar reseller eligible
  - tampilkan sisa tabungan/komisi
  - disable submit jika belum lunas
- dependency: MKE-08
- output yang diharapkan: UI pencairan
- acceptance criteria:
  - state disabled untuk reseller `BELUM`
  - form membedakan pencairan tabungan vs komisi

## MKE-10 - Implementasi halaman akun kas dan saldo

- tujuan: menjadikan posisi kas mudah dibaca lintas transaksi
- file yang dibuat/diubah: `admin/keuangan/akun-kas/page.tsx`, `AkunKasListScreen.tsx`, `KasSaldoSummaryCards.tsx`, `SaldoKasTable.tsx`, `useAkunKasMock.ts`, `useSaldoKasMock.ts`
- lokasi file: `src/app/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - tampilkan daftar akun kas
  - tampilkan summary total saldo
  - buat filter periode bila dibutuhkan
- dependency: MKE-02
- output yang diharapkan: halaman posisi kas
- acceptance criteria:
  - saldo tiap akun terlihat
  - komponen tidak menghitung saldo sendiri

## MKE-11 - Tambahkan alert panel keuangan

- tujuan: memudahkan admin melihat saldo kritis dan reseller siap cair
- file yang dibuat/diubah: `KeuanganAlertPanel.tsx`
- lokasi file: `src/app/components/admin/keuangan/KeuanganAlertPanel.tsx`
- langkah kerja AI agent:
  - bangun alert untuk saldo rendah
  - bangun alert untuk reseller `LUNAS` dengan sisa hak > 0
  - tautkan ke halaman terkait
- dependency: MKE-03, MKE-10
- output yang diharapkan: panel alert operasional
- acceptance criteria:
  - alert mudah discan
  - tidak membuat keputusan bisnis baru

## MKE-12 - Tambahkan audit dan koreksi keuangan

- tujuan: menjaga jejak transaksi kas dan pencairan
- file yang dibuat/diubah: `010_functions_keuangan.sql`, `012_triggers_audit.sql`
- lokasi file: `supabase/migrations/...`
- langkah kerja AI agent:
  - sediakan koreksi `kas_masuk`, `mutasi_kas`, `pencairan`
  - wajibkan alasan
  - tulis `audit_log` dan `koreksi_transaksi`
- dependency: MKE-04, MKE-06, MKE-08
- output yang diharapkan: jalur koreksi keuangan
- acceptance criteria:
  - koreksi tanpa alasan ditolak
  - perubahan tercatat untuk audit

## MKE-13 - Verifikasi edge case saldo dan pencairan

- tujuan: menutup rule penting dari edge cases
- file yang dibuat/diubah: `keuangan.workflow.ts`, test/spec bila tersedia
- lokasi file: `src/lib/server/workflows/keuangan.workflow.ts`
- langkah kerja AI agent:
  - uji EC-005 reseller belum lunas
  - uji EC-006 saldo kas tidak cukup
  - uji mutasi akun sama dan saldo kurang
- dependency: MKE-08, MKE-12
- output yang diharapkan: workflow keuangan stabil
- acceptance criteria:
  - pencairan tidak lolos sebelum lunas
  - kas masuk benar-benar bisa membuka jalan transaksi berikutnya

## MKE-14 - Rapikan exports dan integrasi ke dashboard

- tujuan: memastikan modul keuangan siap dipakai dashboard/laporan nanti
- file yang dibuat/diubah: `src/app/components/admin/keuangan/index.ts`, docs terkait
- lokasi file: `src/app/components/admin/keuangan/index.ts`
- langkah kerja AI agent:
  - ekspor komponen inti
  - rapikan naming saldo/kas/pencairan
  - pastikan import path bersih
- dependency: semua task UI keuangan
- output yang diharapkan: modul keuangan stabil untuk integrasi
- acceptance criteria:
  - komponen mudah dipakai ulang
  - naming antar file konsisten

---

## 11. Pseudocode Task Penting

## 11.1 Pseudocode `buat_kas_masuk`

```txt
validate admin role
validate nominal > 0
validate akun kas exists

insert kas_masuk
write audit

return transaksi + saldo kas terbaru
```

## 11.2 Pseudocode `buat_mutasi_kas`

```txt
validate admin role
validate dari_akun_id != ke_akun_id
validate jumlah > 0

saldo_sumber = getSaldoKas(dari_akun_id, periode_id)
if saldo_sumber < jumlah:
  reject MUTASI_SALDO_TIDAK_CUKUP

insert mutasi_kas
write audit

return mutasi + saldo akun asal/tujuan terbaru
```

## 11.3 Flow `buat_pencairan`

```txt
validate admin role
validate reseller status_lunas_reseller == LUNAS
validate nominal > 0

eligibility = getPencairanEligibility(no_reseller, periode_id)
available = jenis == TABUNGAN ? eligibility.sisa_tabungan : eligibility.sisa_komisi

if nominal > available:
  reject PENCAIRAN_SISA_DANA_TIDAK_CUKUP

saldo_kas = getSaldoKas(akun_kas_id, periode_id)
if saldo_kas < nominal:
  reject PENCAIRAN_SALDO_KAS_TIDAK_CUKUP

insert pencairan
write audit
return pencairan + saldo terbaru + sisa dana terbaru
```

---

## 12. Rule untuk AI Coding Agent

- jangan hitung saldo kas di frontend
- jangan izinkan pencairan hanya berdasarkan state lokal list reseller
- semua validasi saldo harus memakai source backend yang sama dengan belanja
- kas masuk adalah audit trail resmi, bukan edit saldo awal
- mutasi kas tidak boleh dipakai untuk mengakali saldo negatif tersembunyi
- pencairan harus tunduk ke `status_lunas_reseller`
- semua nominal uang harus diperlakukan sebagai decimal/numeric yang diformat di UI

---

## 13. Urutan Eksekusi dari Awal sampai Akhir

1. finalkan schema keuangan
2. buat `v_saldo_kas`
3. buat helper eligibility pencairan
4. implementasikan RPC kas masuk
5. implementasikan UI kas masuk
6. implementasikan RPC mutasi kas
7. implementasikan UI mutasi kas
8. implementasikan RPC pencairan
9. implementasikan UI pencairan
10. implementasikan halaman akun kas dan saldo
11. tambahkan audit dan koreksi
12. verifikasi edge case

---

## 14. Risiko, Asumsi, dan Pertanyaan Konfirmasi

## Risiko

- mismatch saldo bisa terjadi bila belanja, mutasi, dan pencairan tidak membaca formula saldo yang sama
- pencairan rawan salah bila sisa tabungan/komisi tidak memakai ringkasan reseller terbaru
- kas masuk bisa disalahgunakan sebagai koreksi liar jika alasan tidak dijaga

## Asumsi

- akun kas dikelola admin-only
- pencairan dilakukan admin, bukan reseller self-service
- metode pencairan cukup `CASH` dan `TRANSFER` pada fase awal
- saldo kas per periode tetap dibaca dari transaksi live, bukan snapshot
- kebutuhan historis atau lintas periode memakai `v_saldo_kas` dengan `periode_id` yang dipilih

## Catatan Keputusan Terkunci

- pencairan `TABUNGAN` dan `KOMISI` tetap dipisah per jenis transaksi sesuai `BQ-005` / `docs/support/decision_log.md`
- mutasi kas lintas periode dilarang sesuai `BQ-006` / `docs/support/decision_log.md`
