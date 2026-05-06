# Execution Module Master Periodik
## Paket Lebaran Mumpuni

Dokumen ini adalah blueprint eksekusi modul `master_periodik`.

Dokumen ini memecah implementasi area master data lintas dan per-periode ke dua area yang terkoordinasi:

- frontend mockup
- backend/core logic

Dokumen ini mengacu pada:

- `docs/product/prd.md`
- `docs/contracts/schema_mapping.md`
- `docs/contracts/business_contracts.md`
- `docs/contracts/query_contracts.md`
- `docs/product/period_workflow.md`
- `docs/navigation_and_period_setup_ui.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/frontend_architecture.md`
- `docs/execution/backend_plan.md`

---

## 1. Tujuan Modul

Modul `master_periodik` bertanggung jawab untuk:

- mengelola `akun_kas`
- mengelola `barang`
- mengelola `barang_periode`
- mengelola `komisi_config`
- mengelola `paket` dan `detail_paket`
- mengelola `reseller_periode`
- menyediakan fondasi data untuk wizard persiapan periode

Modul ini harus siap sebelum periode `PERSIAPAN` dapat naik ke `AKTIF`.

---

## 2. Scope Modul

### Frontend Mockup Scope

- halaman master akun kas
- halaman master barang
- halaman barang periode
- halaman komisi per periode
- halaman paket dan detail BOM
- halaman reseller periode
- state warning saat mengubah master pada periode `AKTIF`

### Backend/Core Scope

- schema master lintas periode dan periodik
- boundary create/update/list/detail master
- validasi relasi periode
- validasi paket wajib punya BOM
- validasi reseller periode unik
- helper readiness untuk checklist periode

---

## 3. Integration Contract Modul

## 3.1 Entitas Minimum

```txt
akun_kas
barang
barang_periode
komisi_config
paket
detail_paket
reseller_periode
```

## 3.2 Rule Operasional Minimum

```txt
- akun_kas dan barang bersifat lintas periode
- barang_periode, komisi_config, paket, detail_paket, reseller_periode bersifat period-aware
- reseller_periode harus unik per pasangan (periode_id, no_reseller)
- paket komposit wajib punya detail_paket yang valid
- warning perubahan master aktif hanya UI/contract aid; keputusan final tetap di backend boundary
```

## 3.3 Contract Readiness Checklist Periode

```txt
checklist master periodik minimal harus bisa membaca:
- akun kas tersedia
- barang periode tersedia
- komisi_config tersedia
- paket dengan BOM valid tersedia
- reseller_periode tersedia
```

---

## 4. Struktur Folder Modul

## 4.1 Frontend

```txt
src/app/(DashboardLayout)/admin/master/
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

src/app/(DashboardLayout)/admin/reseller-periode/
  page.tsx

src/app/components/admin/master/
  AkunKasScreen.tsx
  BarangScreen.tsx
  BarangPeriodeScreen.tsx
  KomisiScreen.tsx
  PaketScreen.tsx
  PaketBomEditor.tsx
  ResellerPeriodeScreen.tsx
  MasterPeriodWarning.tsx
  index.ts

src/hooks/admin/
  useAkunKasMasterMock.ts
  useBarangMasterMock.ts
  useBarangPeriodeMock.ts
  useKomisiMock.ts
  usePaketMasterMock.ts
  useResellerPeriodeMock.ts
```

## 4.2 Backend

```txt
supabase/migrations/
  001_init_schema.sql
  003_seed_reference_data.sql
  004_views_ringkasan.sql
  005_functions_periode.sql

src/lib/server/services/
  master-periodik.service.ts

src/lib/server/validation/
  master-periodik.validation.ts

src/lib/server/workflows/
  master-periodik.workflow.ts

src/app/api/admin/master/
  akun-kas/route.ts
  barang/route.ts
  barang-periode/route.ts
  komisi/route.ts
  paket/route.ts

src/app/api/admin/reseller-periode/
  route.ts
```

---

## 5. Hubungan Antar File

- `AkunKasScreen.tsx` memakai `useAkunKasMasterMock.ts` dan meniru shape route `admin/master/akun-kas/route.ts`
- `BarangScreen.tsx` memakai `useBarangMasterMock.ts` dan menjadi sumber pilihan untuk `BarangPeriodeScreen.tsx`
- `BarangPeriodeScreen.tsx`, `KomisiScreen.tsx`, dan `ResellerPeriodeScreen.tsx` wajib sadar `periode_id`
- `PaketScreen.tsx` dan `PaketBomEditor.tsx` mengirim payload yang meniru boundary paket periodik
- `master-periodik.validation.ts` memvalidasi payload input area master sebelum `master-periodik.service.ts`
- `master-periodik.service.ts` meneruskan rule period-aware ke `master-periodik.workflow.ts` dan SQL boundary
- `periode.workflow.ts` membaca helper readiness hasil modul ini untuk menentukan checklist aktivasi periode

---

## 6. Breakdown Component and Function

| File | Tanggung jawab |
|---|---|
| `AkunKasScreen.tsx` | list/filter akun kas |
| `BarangScreen.tsx` | list/filter barang lintas periode |
| `BarangPeriodeScreen.tsx` | setup barang periode |
| `KomisiScreen.tsx` | setup komisi periodik |
| `PaketScreen.tsx` | list/filter paket periodik |
| `PaketBomEditor.tsx` | editor BOM paket |
| `ResellerPeriodeScreen.tsx` | setup reseller aktif per periode |
| `MasterPeriodWarning.tsx` | warning saat master periodik disentuh pada periode `AKTIF` |
| `useAkunKasMasterMock.ts` | hook mock akun kas |
| `useBarangMasterMock.ts` | hook mock barang |
| `useBarangPeriodeMock.ts` | hook mock barang periode |
| `useKomisiMock.ts` | hook mock komisi |
| `usePaketMasterMock.ts` | hook mock paket dan BOM |
| `useResellerPeriodeMock.ts` | hook mock reseller periode |
| `master-periodik.validation.ts` | validasi input master periodik |
| `master-periodik.service.ts` | service wrapper area master periodik |
| `master-periodik.workflow.ts` | rule period-aware dan readiness helper |
| `admin/master/*/route.ts` | route internal admin area master |
| `admin/reseller-periode/route.ts` | route internal reseller periode |

---

## 7. Task Execution Batches

### Batch M1 - Schema and Contracts

- MMP-01
- MMP-02
- MMP-03

### Batch M2 - UI Master Baseline

- MMP-04
- MMP-05
- MMP-06

### Batch M3 - Period Readiness Integration

- MMP-07
- MMP-08
- MMP-09

---

## 8. Task Implementation Detail

## MMP-01 - Finalkan schema master lintas periode

- tujuan: memastikan `akun_kas` dan `barang` stabil sebagai fondasi lintas periode
- file yang dibuat/diubah: `001_init_schema.sql`
- lokasi file: `supabase/migrations/001_init_schema.sql`
- langkah kerja AI agent:
  - pastikan tabel `akun_kas` punya identitas, status aktif, dan field display minimum yang stabil
  - pastikan tabel `barang` punya identitas, satuan, dan field dasar yang dipakai modul gudang/pesanan
  - kunci constraint dasar agar tidak ada duplikasi liar pada master lintas periode
- dependency: `schema_mapping.md`
- output yang diharapkan: schema master dasar siap
- acceptance criteria:
  - `akun_kas` dan `barang` punya constraint yang jelas
  - tidak tercampur dengan transaksi periode

## MMP-02 - Finalkan schema master periodik

- tujuan: memastikan `barang_periode`, `komisi_config`, `paket`, `detail_paket`, `reseller_periode` siap
- file yang dibuat/diubah: `001_init_schema.sql`
- lokasi file: `supabase/migrations/001_init_schema.sql`
- langkah kerja AI agent:
  - pastikan semua entitas periodik terhubung jelas ke `periode_id`
  - pastikan relasi `paket` ke `detail_paket` cukup untuk validasi BOM
  - pastikan `reseller_periode` hanya mengizinkan pasangan unik `(periode_id, no_reseller)`
- dependency: MMP-01
- output yang diharapkan: schema periodik siap
- acceptance criteria:
  - relasi ke `periode_id` jelas
  - paket dan BOM bisa divalidasi

## MMP-03 - Finalkan boundary dan validation master

- tujuan: menyediakan create/update/list/detail yang aman untuk area master
- file yang dibuat/diubah: `master-periodik.service.ts`, `master-periodik.validation.ts`, `master-periodik.workflow.ts`, route master terkait
- lokasi file: `src/lib/server/...`, `src/app/api/admin/...`
- langkah kerja AI agent:
  - bentuk validation input per domain master
  - bentuk service wrapper create/update/list/detail untuk akun kas, barang, barang periode, komisi, paket, reseller periode
  - bentuk workflow rule untuk warning periode `AKTIF`, validasi BOM, dan uniqueness reseller periode
  - sambungkan route internal admin ke validation dan service yang tepat
- dependency: MMP-02
- output yang diharapkan: boundary master siap
- acceptance criteria:
  - warning perubahan saat periode `AKTIF` bisa dipicu dari backend
  - reseller periode unik per periode

## MMP-04 - Finalkan UI akun kas dan barang

- tujuan: memberi admin entry point untuk data lintas periode
- file yang dibuat/diubah: `admin/master/akun-kas/page.tsx`, `admin/master/barang/page.tsx`, `AkunKasScreen.tsx`, `BarangScreen.tsx`, `useAkunKasMasterMock.ts`, `useBarangMasterMock.ts`
- lokasi file: `src/app/(DashboardLayout)/admin/master/...`, `src/app/components/admin/master/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun screen akun kas dan barang dengan list/filter dasar
  - hubungkan screen ke hook mock masing-masing
  - tampilkan loading, empty, dan error state yang jujur
- dependency: MMP-03
- output yang diharapkan: UI akun kas dan barang siap
- acceptance criteria:
  - list/form dasar jelas
  - state loading/empty/error tersedia

## MMP-05 - Finalkan UI barang periode, komisi, dan reseller periode

- tujuan: memberi admin area setup periodik yang langsung dipakai wizard periode
- file yang dibuat/diubah: `admin/master/barang-periode/page.tsx`, `admin/master/komisi/page.tsx`, `admin/reseller-periode/page.tsx`, `BarangPeriodeScreen.tsx`, `KomisiScreen.tsx`, `ResellerPeriodeScreen.tsx`, `useBarangPeriodeMock.ts`, `useKomisiMock.ts`, `useResellerPeriodeMock.ts`
- lokasi file: `src/app/(DashboardLayout)/admin/...`, `src/app/components/admin/master/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun screen period-aware untuk barang periode, komisi, dan reseller periode
  - tampilkan state `no_periode` bila konteks periode belum siap
  - pastikan filter/header menunjukkan konteks periode aktif atau periode yang dipilih
- dependency: MMP-03
- output yang diharapkan: UI master periodik siap
- acceptance criteria:
  - semua page sadar `periode_id`
  - tidak ada formula bisnis final di browser

## MMP-06 - Finalkan UI paket dan BOM

- tujuan: mengamankan fondasi paket sebelum program/order berjalan
- file yang dibuat/diubah: `admin/master/paket/page.tsx`, `PaketScreen.tsx`, `PaketBomEditor.tsx`, `usePaketMasterMock.ts`
- lokasi file: `src/app/(DashboardLayout)/admin/master/paket/page.tsx`, `src/app/components/admin/master/...`, `src/hooks/admin/...`
- langkah kerja AI agent:
  - bangun screen paket periodik dan editor BOM
  - tampilkan perbedaan paket tunggal dan komposit secara jujur
  - pastikan payload mock meniru kontrak paket dan detail BOM final
- dependency: MMP-03
- output yang diharapkan: UI paket/BOM siap
- acceptance criteria:
  - paket tanpa BOM bisa terdeteksi
  - perubahan pada periode `AKTIF` memunculkan warning yang benar

## MMP-07 - Hubungkan master periodik ke checklist periode

- tujuan: menjadikan readiness periode benar-benar berbasis data
- file yang dibuat/diubah: `005_functions_periode.sql`, `periode.workflow.ts`
- lokasi file: `supabase/migrations/005_functions_periode.sql`, `src/lib/server/workflows/periode.workflow.ts`
- langkah kerja AI agent:
  - sambungkan helper checklist periode ke data akun kas, barang periode, komisi, paket/BOM, dan reseller periode
  - pastikan checklist membaca data nyata, bukan flag manual frontend
  - tampilkan hasil checklist per item readiness minimum
- dependency: MMP-02
- output yang diharapkan: checklist periode sinkron
- acceptance criteria:
  - item akun kas, barang periode, paket/BOM, komisi, reseller periode bisa dibaca checklist

## MMP-08 - Tambahkan warning master aktif

- tujuan: mencegah admin mengubah master periodik tanpa sadar dampaknya
- file yang dibuat/diubah: `MasterPeriodWarning.tsx`
- lokasi file: `src/app/components/admin/master/MasterPeriodWarning.tsx`
- langkah kerja AI agent:
  - bangun komponen warning reusable untuk area master periodik
  - tampilkan warning berdasarkan flag/field yang sudah dikirim backend boundary
  - pastikan warning tidak menghitung sendiri status periode final di browser
- dependency: MMP-04, MMP-05, MMP-06
- output yang diharapkan: warning konsisten
- acceptance criteria:
  - perubahan master saat `AKTIF` tidak terasa silent

## MMP-09 - Verifikasi dependency dengan modul transaksi

- tujuan: memastikan master data benar-benar cukup sebelum coding program/order/setoran
- file yang dibuat/diubah: dokumentasi check atau spec yang tersedia
- lokasi file: mengikuti tool test yang dipilih di `docs/testing_strategy.md`
- dependency: semua task modul ini
- output yang diharapkan: modul siap jadi fondasi transaksi
- acceptance criteria:
  - periode tidak bisa aktif jika master minimum belum lengkap

---

## 9. Pseudocode / Flow Penting

## 9.1 Flow `validate_paket_bom`

```txt
load paket payload
if paket marked komposit:
  require at least one detail_paket row
  for each detail_paket row:
    require barang_id valid
    require qty > 0
  if any row invalid:
    reject PAKET_BOM_INVALID

if paket marked tunggal:
  do not require BOM rows

allow save
```

## 9.2 Flow `validate_reseller_periode_unique`

```txt
read periode_id and no_reseller
if reseller not found:
  reject RESELLER_NOT_FOUND

check existing reseller_periode by (periode_id, no_reseller)
if already exists:
  reject RESELLER_PERIODE_DUPLICATE

allow save
```

## 9.3 Flow `build_period_readiness_from_master`

```txt
check akun_kas exists
check barang_periode exists for active/selected periode
check komisi_config exists for active/selected periode
check paket periodik exists
check every paket komposit has valid BOM
check reseller_periode exists

return checklist:
  akun_kas_ready
  barang_periode_ready
  komisi_ready
  paket_bom_ready
  reseller_periode_ready
```

---

## 10. First Working Slice

Slice pertama modul ini:

1. admin punya minimal satu akun kas
2. admin punya daftar barang
3. admin bisa membuat barang periode
4. admin bisa membuat paket dengan BOM valid
5. admin bisa mengaitkan reseller ke periode
6. checklist periode membaca semua itu

---

## 11. Rule Khusus AI Coding Agent

- jangan menaruh validasi BOM final di komponen frontend
- jangan menebak readiness periode dari jumlah card atau state UI; baca dari helper checklist resmi
- jangan mencampur master lintas periode dengan transaksi operasional
- jangan membuat warning periode `AKTIF` hanya dari path; pakai flag/contract backend
- jangan memperluas scope ke modul transaksi jika dependency belum masuk batch aktif

---

## 12. Urutan Eksekusi Modul

1. kunci schema master lintas periode dan periodik (`MMP-01`, `MMP-02`)
2. bentuk boundary backend master (`MMP-03`)
3. bangun screen admin master dasar (`MMP-04`, `MMP-05`, `MMP-06`)
4. sambungkan readiness checklist periode (`MMP-07`)
5. tampilkan warning master aktif dan verifikasi dependency transaksi (`MMP-08`, `MMP-09`)

---

## 13. Risiko, Asumsi, dan Pertanyaan Konfirmasi

## Risiko

- master periodik yang tidak ketat akan merusak semua modul transaksi di belakangnya
- paket tanpa BOM atau reseller_periode tidak lengkap akan membuat aktivasi periode menyesatkan

## Asumsi

- master periodik dikelola admin-only
- wizard periode akan menjadi consumer utama modul ini

## Catatan Keputusan Terkunci

- warning perubahan akun kas saat periode `AKTIF` tetap diperlukan sesuai `BQ-007` / `docs/truth/01-decision_log.md`
- `barang_periode` tetap wajib memuat `budget_belanja` pada fase awal sesuai `BQ-008` / `docs/truth/01-decision_log.md`
