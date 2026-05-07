# API Integration
## Paket Lebaran Mumpuni

Dokumen ini adalah panduan integrasi teknis antara frontend, internal API route, service layer, dan Supabase.

Dokumen ini **bukan** sumber kebenaran utama untuk:

- nama RPC bisnis
- payload request/response final
- aturan query/read model
- business rule

Sumber kebenaran tetap berada di:

1. `docs/contracts/schema_mapping.md`
2. `docs/contracts/business_contracts.md`
3. `docs/contracts/query_contracts.md`
4. `docs/contracts/integration_contract_pack.md`

---

## 1. Tujuan Dokumen

Panduan ini hanya menjelaskan pola integrasi yang harus diikuti agent saat implementasi:

- frontend tidak memanggil query liar
- internal route memakai envelope konsisten
- service layer menjadi pembatas antara UI dan Supabase
- read dan write mengikuti kontrak aktif

---

## 2. Arsitektur Integrasi

Pola utama:

```txt
Page / Component
-> Hook SWR atau action
-> Internal API route
-> validation
-> service
-> workflow/helper bila perlu
-> Supabase RPC / view / table
```

Aturan:

- frontend mockup boleh memakai placeholder API, tetapi shape-nya harus sama dengan internal route final
- frontend implementasi nyata wajib berbicara ke internal route untuk semua read dan write operasional, bukan memanggil Supabase browser client secara liar
- transaksi bisnis penting tetap berakhir di RPC/database function yang atomic

---

## 3. Routing Integration Pattern

Pola route internal harus mengikuti domain dan route strategy aktif.

Contoh kelompok route:

```txt
/api/admin/periode
/api/admin/reseller
/api/admin/gudang/belanja
/api/admin/gudang/packing
/api/admin/gudang/pengiriman
/api/admin/gudang/pembagian
/api/admin/keuangan/akun-kas
/api/admin/keuangan/saldo-kas
/api/admin/keuangan/kas-masuk
/api/admin/keuangan/mutasi-kas
/api/admin/keuangan/pencairan
/api/admin/laporan/rekap-reseller
/api/admin/laporan/stok
/api/admin/laporan/pengiriman
/api/admin/laporan/audit
/api/reseller/dashboard
/api/reseller/konsumen
/api/reseller/pesanan
/api/reseller/setor
/api/reseller/akun
```

---

## 4. Envelope Response

Gunakan envelope dari `docs/contracts/integration_contract_pack.md`.

Contoh sukses:

```ts
type ApiSuccess<T> = {
  success: true;
  data: T;
  meta?: {
    page?: number;
    page_size?: number;
    total_items?: number;
    total_pages?: number;
    sort_by?: string;
    sort_direction?: 'asc' | 'desc';
    applied_filters?: Record<string, unknown>;
  };
};
```

Contoh gagal:

```ts
type ApiFailure = {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
    field_errors?: Record<string, string>;
  };
};
```

---

## 5. Read Pattern

Keputusan fase awal:

- semua read bisnis admin dan reseller melewati internal API route
- direct Supabase browser client tidak dipakai untuk view/RPC operasional proyek
- pengecualian hanya untuk auth/session bootstrap atau helper shell non-bisnis yang sudah disetujui

### Read Dashboard / Laporan

- pakai internal route
- route memanggil service read
- service membaca RPC read atau view yang resmi
- pagination, sort, dan search dilakukan di backend

Contoh:

```txt
GET /api/admin/dashboard?periode_id=12
-> get_dashboard_admin(periode_id)

GET /api/admin/laporan/rekap-reseller?periode_id=12&page=1&page_size=20
-> get_laporan_rekap_reseller(...) atau wrapper v_ringkasan_reseller
```

### Read List Domain

- list besar admin wajib server-side pagination
- reseller list boleh lebih ringan, tetapi tetap scope `periode_id` dan `no_reseller`

---

## 6. Write Pattern

### Transaksi Bisnis

Semua transaksi berikut wajib lewat boundary resmi:

- periode
- pesanan
- finalisasi pesanan
- setoran konsumen
- setoran pusat
- belanja
- packing
- pengiriman
- pembagian
- kas masuk
- mutasi kas
- pencairan
- koreksi

Pola:

```txt
POST /api/admin/... atau /api/reseller/...
-> validate payload
-> resolve actor/session
-> call service
-> service call RPC resmi
-> return envelope + message
```

Nama RPC final harus mengikuti `docs/contracts/business_contracts.md` dan blueprint modul yang sudah aktif.

---

## 7. Auth dan Access Pattern

- identitas actor diambil dari session Supabase
- role dan `no_reseller` diturunkan dari `profile`
- reseller tidak boleh mengirim `no_reseller` bebas lalu dipercaya begitu saja
- admin-only route harus memvalidasi role admin sebelum menyentuh service sensitif

Referensi aktif:

- `docs/contracts/rls_matrix.md`
- `docs/quality/implementation_guardrails.md`

---

## 8. Error Handling Pattern

Error yang dikembalikan ke frontend harus:

- berbahasa Indonesia
- memakai `code` yang konsisten
- tidak membocorkan pesan teknis database mentah

Contoh kategori:

```txt
AUTH_*
ACCESS_*
VALIDATION_*
PERIODE_*
RESELLER_*
PROGRAM_*
ORDER_*
SETORAN_*
GUDANG_*
KEUANGAN_*
SYSTEM_*
```

---

## 9. Hook Integration Pattern

Hook frontend harus:

- punya key/query key yang stabil
- memanggil internal route atau placeholder route yang bentuknya sama
- tidak menyebar formula bisnis ke browser

Contoh:

```txt
useAdminDashboard
-> GET /api/admin/dashboard

useRekapResellerReport
-> GET /api/admin/laporan/rekap-reseller

useSetoranKonsumen
-> POST /api/reseller/setor
```

Nama hook disarankan mengikuti `docs/contracts/integration_read_model_matrix.md`.

---

## 10. Do-Not-Do Rules

Jangan:

- memakai nama RPC lama yang tidak ada di kontrak aktif
- membuat frontend langsung memanggil transaksi kompleks ke tabel
- membuat route internal dengan envelope berbeda-beda
- menaruh logika pelunasan, stok, komisi, atau target final di komponen
- memperlakukan contoh pada dokumen ini sebagai pengganti kontrak bisnis

---

## 11. Kapan Dokumen Ini Harus Diperbarui

Perbarui dokumen ini jika:

- pola integrasi route berubah
- envelope API berubah
- internal route strategy berubah besar

Jangan memperbarui dokumen ini hanya untuk menambah contoh RPC baru jika kontraknya belum resmi di dokumen otoritatif.
