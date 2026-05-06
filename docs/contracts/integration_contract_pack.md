# Integration Contract Pack
## Paket Lebaran Mumpuni

Dokumen ini menetapkan kontrak integrasi teknis inti antara:

- frontend mockup
- API/internal route layer
- service layer
- Supabase/core logic layer

Dokumen ini dipakai sebelum plan dipecah lebih jauh per modul prioritas.

Tujuan utamanya adalah mencegah mismatch antara frontend dan backend saat implementasi paralel dimulai.

---

## 1. Tujuan dan Scope

Kontrak ini mengatur:

- naming convention
- ID convention
- enum dan status bersama
- request/response contract
- error contract
- pagination, sorting, filtering
- format uang, tanggal, dan numerik
- aturan nullability dan optional field

Dokumen ini tidak mendefinisikan desain UI atau detail business rules penuh. Business rules tetap mengacu ke dokumen domain.

---

## 2. Urutan Otoritas

Jika ada konflik, urutannya:

1. `docs/contracts/schema_mapping.md`
2. `docs/contracts/business_contracts.md`
3. `docs/contracts/query_contracts.md`
4. `docs/contracts/integration_contract_pack.md`
5. `docs/execution/frontend_plan.md`
6. `docs/execution/backend_plan.md`

---

## 3. Naming Convention

## 3.1 Prinsip Umum

- Database layer boleh memakai `snake_case`.
- API response ke frontend harus konsisten, dan untuk proyek ini direkomendasikan tetap memakai `snake_case` agar dekat dengan output data layer.
- Frontend mockup tidak boleh mengubah nama field seenaknya per screen.
- TypeScript type harus mengikuti shape kontrak yang sama lintas layer boundary.

## 3.2 Aturan Nama Field

Gunakan pola berikut:

- foreign key: `*_id`
- status: `status_*` jika status spesifik domain, atau `status` bila entitas hanya punya satu status utama
- timestamp: `created_at`, `updated_at`
- date-only: `tgl_*`
- numeric amount: `total_*`, `sisa_*`, `saldo_*`, `nilai_*`, `nominal_*`

Contoh:

```ts
type PesananKonsumenSummary = {
  id: number;
  periode_id: number;
  konsumen_id: number;
  no_reseller: string;
  status_pesanan: 'AKTIF' | 'PERLU_PERHATIAN' | 'SELESAI' | 'BATAL';
  target_tagihan_snapshot: number;
  target_berjalan: number;
  total_bayar: number;
  sisa_bayar: number;
  created_at: string;
};
```

---

## 4. ID Convention

## 4.1 Jenis ID

- `UUID` untuk identity yang terhubung ke auth atau butuh global uniqueness tinggi
- `BIGINT` / `INTEGER` untuk entitas operasional internal jika itu keputusan schema final
- `string code` untuk identifier bisnis seperti `no_reseller`

## 4.2 Aturan Per Entitas

Rekomendasi umum:

- `profile.id` -> UUID
- `reseller.no_reseller` -> string bisnis
- `periode.id` -> numeric
- `konsumen.id` -> numeric
- `pesanan_konsumen.id` -> numeric
- `detail_pesanan_konsumen.id` -> numeric
- transaksi operasional -> numeric

## 4.3 Aturan Frontend

- frontend tidak boleh mengasumsikan semua `id` berbentuk angka bila kontraknya string
- komponen list dan table harus menerima `string | number` bila memang masih lintas entitas
- route param tetap diparse sesuai kontrak entitas, bukan asumsi global

---

## 5. Enum dan Status Bersama

## 5.1 Status Periode

```ts
type StatusPeriode = 'PERSIAPAN' | 'AKTIF' | 'SELESAI';
```

## 5.2 Status Reseller

```ts
type StatusReseller = 'PENDING' | 'AKTIF' | 'NONAKTIF';
```

## 5.3 Status Pesanan

```ts
type StatusPesanan = 'AKTIF' | 'PERLU_PERHATIAN' | 'SELESAI' | 'BATAL';
```

## 5.4 Status Detail Pesanan

```ts
type StatusItemPesanan = 'AKTIF' | 'TERHENTI' | 'BATAL';
type StatusKirimDetailPesanan = 'BELUM' | 'SUDAH' | 'BATAL';
```

## 5.5 Status Pelunasan

```ts
type StatusLunas = 'BELUM' | 'LUNAS';
```

## 5.7 Status Umum Operasional

```ts
type StatusApproval = 'PENDING' | 'APPROVED' | 'REJECTED';
type StatusRecord = 'AKTIF' | 'NONAKTIF';
```

Aturan:

- jangan membuat variasi label baru seperti `ACTIVE`, `DONE`, `IN_PROGRESS` bila domain sudah punya enum resmi
- UI boleh menampilkan label berbeda, tapi payload tetap mengikuti enum kontrak

---

## 6. Contract Request dan Response

## 6.1 Wrapper Response Standar

Semua route internal dan placeholder API harus memakai envelope konsisten:

```ts
type ApiSuccess<T> = {
  success: true;
  data: T;
  meta?: ResponseMeta;
};

type ApiFailure = {
  success: false;
  error: ApiError;
};

type ApiResponse<T> = ApiSuccess<T> | ApiFailure;
```

## 6.2 Meta Response

```ts
type ResponseMeta = {
  page?: number;
  page_size?: number;
  total_items?: number;
  total_pages?: number;
  sort_by?: string;
  sort_direction?: 'asc' | 'desc';
  applied_filters?: Record<string, unknown>;
};
```

## 6.3 Error Contract

```ts
type ApiError = {
  code: string;
  message: string;
  details?: Record<string, unknown>;
  field_errors?: Record<string, string>;
};
```

## 6.4 Action Response

Untuk create/update/delete/action transactional:

```ts
type ActionResponse<T> = {
  success: boolean;
  data?: T;
  error?: ApiError;
  message?: string;
};
```

---

## 7. Error Code Standard

Gunakan prefix kode error yang konsisten:

- `AUTH_*`
- `ACCESS_*`
- `VALIDATION_*`
- `PERIODE_*`
- `RESELLER_*`
- `PROGRAM_*`
- `ORDER_*`
- `SETORAN_*`
- `GUDANG_*`
- `KEUANGAN_*`
- `SYSTEM_*`

Contoh:

```txt
AUTH_UNAUTHORIZED
ACCESS_FORBIDDEN
VALIDATION_REQUIRED_FIELD
PERIODE_NOT_ACTIVE
PERIODE_SETUP_INCOMPLETE
RESELLER_NOT_FOUND
PROGRAM_LOCKED
ORDER_ALREADY_CANCELLED
SETORAN_MELEBIHI_TARGET
GUDANG_STOK_TIDAK_CUKUP
KEUANGAN_SALDO_TIDAK_CUKUP
SYSTEM_UNEXPECTED_ERROR
```

Aturan:

- `code` harus stabil dan machine-readable
- `message` harus siap tampil ke user atau mudah dipetakan ke copy UI
- frontend mock juga harus memakai kode ini, bukan pesan bebas

---

## 8. Pagination Contract

## 8.1 Request Pagination

```ts
type PaginationQuery = {
  page?: number;
  page_size?: number;
};
```

Aturan:

- `page` dimulai dari `1`
- `page_size` default ditentukan per endpoint

## 8.2 Response Pagination

```ts
type PaginatedResponse<T> = ApiResponse<T[]> & {
  meta?: ResponseMeta;
};
```

## 8.3 Default Page Size

Rekomendasi:

- admin table besar: `20`
- reseller list mobile: `10`
- search result cepat: `10`

---

## 9. Sorting Contract

## 9.1 Request Sorting

```ts
type SortQuery = {
  sort_by?: string;
  sort_direction?: 'asc' | 'desc';
};
```

## 9.2 Aturan Sorting

- `sort_by` hanya boleh field whitelist
- `sort_direction` hanya `asc` atau `desc`
- default sort harus ditetapkan per endpoint

Contoh:

- konsumen reseller: `sisa_bayar desc`, lalu `nama_konsumen asc`
- daftar periode: `created_at desc`
- order admin: `created_at desc`

---

## 10. Filter Contract

## 10.1 Prinsip Umum

- filter dikirim eksplisit
- nama filter harus konsisten dengan domain
- boolean filter gunakan `true/false`, bukan string bebas jika memungkinkan

## 10.2 Contoh Filter

```ts
type PeriodeFilter = {
  status?: StatusPeriode[];
};

type OrderFilter = {
  periode_id?: number;
  no_reseller?: string;
  status_kirim?: StatusKirimDetailPesanan[];
  status_item?: StatusItemPesanan[];
  q?: string;
};

type KonsumenFilter = {
  periode_id?: number;
  no_reseller?: string;
  status_lunas?: StatusLunas[];
  status_pesanan?: StatusPesanan[];
  q?: string;
};
```

## 10.3 Search Query

- gunakan field `q`
- `q` selalu opsional
- trimming dilakukan di boundary

---

## 11. Money Contract

## 11.1 Prinsip Umum

- nominal di payload dikirim sebagai `number`
- bukan string terformat rupiah
- formatting `Rp` hanya terjadi di UI layer

## 11.2 Field Uang

Contoh field:

- `nominal`
- `nilai_paket`
- `total_bayar`
- `sisa_bayar`
- `saldo_belum_disetor`
- `total_disetor_pusat`

## 11.3 Aturan Presisi

- frontend mock boleh memakai `number`
- backend/core logic harus menjaga konsistensi presisi sesuai keputusan schema
- jangan kirim string seperti `"10.000"` atau `"Rp 10.000"` di contract data

---

## 12. Date and Time Contract

## 12.1 Date-only Field

Gunakan format ISO date:

```txt
YYYY-MM-DD
```

Contoh:

- `tgl_mulai`
- `tgl_selesai`
- `tgl_setoran`

## 12.2 Timestamp Field

Gunakan ISO datetime:

```txt
YYYY-MM-DDTHH:mm:ss.sssZ
```

Contoh:

- `created_at`
- `updated_at`

## 12.3 Aturan Frontend

- frontend hanya memformat untuk tampilan
- frontend tidak boleh mengubah kontrak source menjadi format lokal dalam payload

---

## 13. Nullability dan Optional Field

## 13.1 Aturan

- `optional` berarti field boleh tidak dikirim
- `nullable` berarti field boleh ada dengan nilai `null`
- jangan mencampur keduanya tanpa alasan

## 13.2 Contoh

```ts
type KonsumenRecord = {
  id: number;
  nama_konsumen: string;
  no_telepon?: string | null;
  alamat?: string | null;
};
```

Aturan:

- field opsional harus konsisten lintas mock dan backend
- jangan buat frontend mengasumsikan string kosong padahal contract memakai `null`

---

## 14. Contract Modul Prioritas

## 14.1 Modul Periode

Field minimum response list:

```ts
type PeriodeListItem = {
  id: number;
  nama_periode: string;
  tgl_mulai: string;
  tgl_selesai: string;
  status: StatusPeriode;
  jumlah_reseller?: number;
  jumlah_konsumen?: number;
};
```

Action create:

```ts
type CreatePeriodeRequest = {
  nama_periode: string;
  tgl_mulai: string;
  tgl_selesai: string;
};
```

## 14.2 Modul Reseller

```ts
type ResellerListItem = {
  no_reseller: string;
  nama_reseller: string;
  status: StatusReseller;
  periode_id: number;
  total_konsumen?: number;
  saldo_belum_disetor?: number;
};
```

## 14.3 Modul Pesanan

```ts
type PesananSummaryItem = {
  id: number;
  periode_id: number;
  konsumen_id: number;
  nama_konsumen: string;
  no_reseller: string;
  status_pesanan: StatusPesanan;
  tanggal_final?: string | null;
  is_final: boolean;
  target_tagihan: number;
  target_berjalan: number;
  total_bayar: number;
  sisa_bayar: number;
};
```

```ts
type DetailPesananSummaryItem = {
  id: number;
  periode_id: number;
  pesanan_konsumen_id: number;
  no_reseller: string;
  nama_konsumen: string;
  nama_paket: string;
  status_item: StatusItemPesanan;
  status_kirim: StatusKirimDetailPesanan;
  nilai_item: number;
  uang_terhenti?: number | null;
};
```

## 14.4 Modul Setoran

```ts
type SetoranKonsumenRequest = {
  pesanan_konsumen_id: number;
  periode_id: number;
  nominal: number;
  tanggal: string;
  metode?: string;
  keterangan?: string | null;
};
```

```ts
type SetoranPusatRequest = {
  periode_id: number;
  no_reseller: string;
  nominal: number;
  tanggal: string;
  akun_kas_id: number;
  metode?: string;
  keterangan?: string | null;
};
```

---

## 15. Placeholder API Alignment Rule

Frontend mockup harus mengikuti kontrak ini:

- endpoint mock memakai field contract yang sama
- dummy response tidak boleh memakai shape bebas
- error mock harus memakai `ApiError`

Contoh:

```ts
const response: ApiResponse<PesananSummaryItem[]> = {
  success: true,
  data: pesanans,
  meta: {
    page: 1,
    page_size: 10,
    total_items: 25,
    total_pages: 3,
  },
};
```

---

## 16. Rule untuk AI Coding Agent

- Jangan membuat field baru tanpa memeriksa kontrak ini.
- Jangan mengganti enum hanya demi kenyamanan UI.
- Jangan mengubah payload request/response per halaman.
- Jika modul butuh field tambahan, tambahkan ke kontrak ini atau dokumen modul turunan.
- Frontend dan backend harus memakai istilah domain yang sama.

---

## 17. Risiko, Asumsi, dan Pertanyaan Konfirmasi

### Risiko

- keputusan `snake_case` vs `camelCase` dilanggar di layer frontend
- mock payload menyimpang dari payload nyata
- enum status berkembang liar saat implementasi paralel

### Asumsi

- boundary integrasi lintas layer akan memakai shape yang konsisten
- modul prioritas pertama adalah `periode`, `reseller`, `program/order`, `setoran`

### Keputusan yang Sudah Dikunci

- API ke frontend menggunakan `snake_case` penuh (DL direkam di `truth/01-decision_log.md`)
- semua route internal menggunakan envelope `ApiResponse<T>`
- `no_reseller` tetap menjadi identifier bisnis utama di semua modul yang melibatkan reseller

### Pertanyaan yang Masih Terbuka

- apakah direct Supabase browser client boleh dipakai untuk read sederhana tertentu atau semua read wajib lewat internal route (catat di `truth/03-open_questions_register.md` jika perlu formal decision)
