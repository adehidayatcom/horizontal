# Testing Strategy
## Paket Lebaran Mumpuni

Dokumen ini menetapkan strategi testing minimum sebelum proyek masuk fase coding aktif.

Tujuannya bukan memaksa semua test lengkap di hari pertama, tetapi memastikan AI coding agent tahu:

- jenis test apa yang wajib dipikirkan
- level test mana yang harus dipakai untuk tiap modul
- flow mana yang tidak boleh hanya diverifikasi manual
- kapan sebuah perubahan boleh dianggap cukup aman

---

## 1. Prinsip Umum

- test mengikuti risiko bisnis, bukan sekadar jumlah file yang berubah
- business rule uang, stok, status, dan audit tidak boleh hanya diuji manual
- frontend mockup boleh mulai dari verifikasi visual dan interaction check, tetapi contract shape tetap harus diuji
- route, validation, dan service layer harus diuji di boundary yang paling masuk akal
- jika test otomatis belum tersedia, kekosongannya harus dilaporkan sebagai risiko, bukan disembunyikan

---

## 2. Target Lapisan Test

### 2.1 Unit Test

Dipakai untuk:

- helper murni
- formatter uang/tanggal
- validator input
- mapper payload
- workflow kecil yang tidak bergantung penuh pada database

Contoh target:

- `money.ts`
- `dates.ts`
- `status.ts`
- `*.validation.ts`
- helper normalisasi dashboard/laporan

### 2.2 Integration Test

Dipakai untuk:

- API route internal
- service layer
- workflow lintas validation + service
- kontrak request/response
- query boundary yang dibungkus route server

Contoh target:

- `POST /api/admin/periode`
- `POST /api/reseller/setor`
- `POST /api/admin/gudang/pengiriman`
- `POST /api/admin/keuangan/pencairan`
- `GET /api/admin/dashboard`

### 2.3 Database / Contract Verification

Dipakai untuk:

- migration SQL
- RPC transaksi bisnis
- read model kritis
- RLS policy

Contoh target:

- `aktifkan_periode`
- `buat_setoran_konsumen`
- `buat_setoran_pusat`
- `buat_pembagian_paket`
- `buat_pencairan`
- `get_dashboard_admin`
- `get_dashboard_reseller`

### 2.4 End-to-End / Browser Verification

Dipakai untuk:

- flow role-based penting
- form uang
- approval flow
- dashboard utama
- flow yang rentan rusak karena state UI

Contoh target:

- login -> route guard -> dashboard sesuai role
- admin buat periode -> checklist -> aktifkan periode
- reseller setor ke konsumen
- admin kirim order
- admin proses pencairan

---

## 3. Matriks Prioritas Test per Modul

| Modul | Unit | Integration | DB / Contract | E2E | Catatan |
|---|---|---|---|---|---|
| `auth` | wajib | wajib | sedang | wajib | role guard dan pending approval tidak boleh hanya manual |
| `master_periodik` | sedang | wajib | wajib | opsional awal | fokus ke constraint dan readiness |
| `periode` | sedang | wajib | wajib | wajib | aktivasi periode adalah blocker modul lain |
| `reseller` | sedang | wajib | wajib | sedang | approval dan onboarding konsumen penting |
| `program_order` | sedang | wajib | wajib | sedang | transisi status harus stabil |
| `setoran` | sedang | wajib | wajib | wajib | menyentuh uang dan pelunasan |
| `gudang` | sedang | wajib | wajib | sedang | stok dan eligibility order wajib aman |
| `keuangan` | sedang | wajib | wajib | wajib | saldo kas dan pencairan berisiko tinggi |
| `dashboard_laporan` | ringan | sedang | wajib | sedang | angka utama tidak boleh dihitung liar di frontend |

---

## 4. Flow Minimum yang Wajib Punya Test

Sebelum fase coding dianggap aman berjalan, flow berikut minimal harus punya rencana verifikasi eksplisit:

1. login, bootstrap profile, dan route guard
2. create periode lalu aktivasi periode
3. approve reseller dan assign reseller ke periode
4. create pesanan konsumen lalu finalisasi pesanan
5. buat setoran konsumen sampai status lunas
6. buat setoran pusat sampai reseller lunas
7. belanja -> packing -> pengiriman/pembagian
8. pencairan reseller yang lolos eligibility
9. baca dashboard admin dan reseller tanpa menghitung formula final di browser

---

## 5. Acceptance Minimum per Layer

### Frontend Mockup

- loading, empty, error, disabled, dan success state terlihat jelas
- submit form tidak mengubah angka penting secara lokal tanpa boundary
- shape data mock mengikuti `docs/integration_contract_pack.md`

### API Route / Service

- request invalid menghasilkan envelope error yang konsisten
- role atau approval yang tidak valid ditolak di server boundary
- route hanya memanggil workflow/service yang memang menjadi domainnya

### Database / RPC

- perubahan status mengikuti rule di `docs/business_contracts.md`
- data historis tidak rusak karena update yang salah
- aksi koreksi meninggalkan jejak audit jika memang diwajibkan

---

## 6. Strategi Fase Awal

Karena repo saat ini belum punya script `test` dan `e2e`, strategi fase awal dibagi menjadi tiga milestone:

### Milestone T1 - Baseline Verification

- `pnpm run lint`
- `pnpm run typecheck`
- `pnpm run build`
- verifikasi manual pada first working slice modul yang sedang dibangun

### Milestone T2 - Contract and Route Test

- tambah runner unit/integration test berbasis `Vitest`
- mulai dari validation, helper, dan API route risiko tinggi
- tambah fixture dasar untuk contract response

### Milestone T3 - Role and Money Flow E2E

- tambah browser test berbasis `Playwright` untuk flow auth, periode, setoran, dan pencairan
- jalankan smoke test role-based sebelum merge perubahan besar

---

## 7. Rule untuk AI Coding Agent

- jangan menutup task backend uang/stok/status tanpa rencana verifikasi route atau contract
- jika menambah route baru, sebutkan minimal satu skenario sukses dan satu skenario gagal
- jika menambah validator, sertakan skenario invalid input yang harus ditolak
- jika modul masih frontend mockup saja, jelaskan mana yang diverifikasi manual dan mana yang masih deferred
- jangan mengklaim `tested` bila hanya membuka halaman tanpa memverifikasi state kritis

---

## 8. Status Repo Saat Dokumen Ini Ditulis

Status yang teramati saat review ini:

- script tersedia: `dev`, `build`, `start`, `lint`, `typecheck`
- script `test`, `test:watch`, dan `e2e` belum ada di `package.json`
- folder `supabase/` belum tersedia di repo saat ini
- generated type `src/types/database.ts` belum tersedia

Artinya:

- strategi test sudah siap sebagai kontrak
- implementasi tool test dan fixture masih menjadi pekerjaan fase coding awal

---

## 9. Dokumen Rujukan

- `docs/definition_of_done.md`
- `docs/environment.md`
- `docs/integration_contract_pack.md`
- `docs/query_contracts.md`
- `docs/business_contracts.md`
