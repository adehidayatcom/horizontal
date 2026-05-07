# System Maps
## Paket Lebaran Mumpuni

Dokumen ini menyediakan peta hubungan antar dokumen, modul bisnis, dan entitas data utama proyek.

Tujuannya:

- memberi peta hubungan antar dokumen inti
- memberi peta hubungan antar modul bisnis utama
- memberi peta dependency antar entitas data inti
- membantu AI coding agent memahami konteks tanpa membaca semua file dari nol

Dokumen ini bukan kontrak bisnis baru. Semua keputusan domain tetap tunduk pada dokumen product truth.

Dokumen acuan utama:

- `../product/prd.md`
- `../truth/02-canonical_system_brief.md`
- `../truth/01-decision_log.md`
- `../truth/03-open_questions_register.md`
- `../product/program_workflow.md`
- `../product/period_workflow.md`
- `../contracts/schema_mapping.md`
- `../contracts/business_contracts.md`
- `../contracts/query_contracts.md`
- `../contracts/integration_contract_pack.md`
- `../quality/testing_strategy.md`
- `../execution/frontend_plan.md`
- `../execution/backend_plan.md`

---

## 1. Cara Membaca Peta Ini

Urutan bacaan yang disarankan untuk AI coding agent:

1. baca `../product/prd.md` untuk arah produk
2. baca `../truth/02-canonical_system_brief.md` untuk ringkasan sistem final
3. baca `../truth/01-decision_log.md` untuk keputusan aman
4. baca `../truth/03-open_questions_register.md` untuk gap aktif
5. lihat peta dokumen pada dokumen ini
6. lihat peta modul bisnis pada dokumen ini
7. lihat peta entitas data pada dokumen ini
8. baru masuk ke execution blueprint atau blueprint modul

---

## 2. Peta Otoritas Dokumen

```mermaid
flowchart TD
    PRD["prd.md<br/>arah produk"] --> BRIEF["02-canonical_system_brief.md<br/>ringkasan sistem final"]
    PRD --> PESANAN["program_workflow.md<br/>lifecycle pesanan_konsumen"]
    PRD --> PERIODE["period_workflow.md<br/>lifecycle periode"]
    PRD --> SCHEMA["schema_mapping.md<br/>struktur data dan formula"]
    PRD --> BUSINESS["business_contracts.md<br/>aksi write / RPC"]
    PRD --> QUERY["query_contracts.md<br/>read model"]

    BRIEF --> INTEGRATION["integration_contract_pack.md<br/>boundary lintas layer"]
    PESANAN --> BUSINESS
    PESANAN --> QUERY
    PERIODE --> BUSINESS
    PERIODE --> QUERY
    SCHEMA --> BUSINESS
    SCHEMA --> QUERY

    BUSINESS --> BACKEND["execution/backend_plan.md"]
    QUERY --> BACKEND
    INTEGRATION --> BACKEND

    INTEGRATION --> FRONTEND["execution/frontend_plan.md"]
    QUERY --> FRONTEND
    PESANAN --> FRONTEND
    PERIODE --> FRONTEND

    FRONTEND --> MODULES["modules/*.md"]
    BACKEND --> MODULES
```

### Ringkasan Makna

- `../product/prd.md` tetap menjadi pintu masuk produk
- `../truth/02-canonical_system_brief.md` adalah ringkasan cepat sistem yang stabil
- `../truth/01-decision_log.md` adalah keputusan lintas-dokumen yang sudah dikunci
- `../truth/03-open_questions_register.md` adalah gap aktif yang belum boleh ditebak
- `../contracts/schema_mapping.md`, `../contracts/business_contracts.md`, dan `../contracts/query_contracts.md` adalah trio inti domain build
- `../contracts/integration_contract_pack.md` adalah pagar teknis sebelum frontend dan backend berjalan paralel
- dokumen di `modules/*.md` tidak boleh membuat keputusan domain baru yang melawan dokumen di atasnya

---

## 3. Peta Peran Dokumen

| Dokumen | Peran utama | Dipakai oleh |
|---|---|---|
| `../product/prd.md` | arah produk dan ruang lingkup | semua agent |
| `../truth/02-canonical_system_brief.md` | ringkasan final sistem | semua agent |
| `../truth/01-decision_log.md` | keputusan lintas-dokumen yang sudah dikunci | semua agent |
| `../truth/03-open_questions_register.md` | pertanyaan terbuka yang masih perlu keputusan | semua agent |
| `../product/program_workflow.md` | lifecycle `pesanan_konsumen`, `detail_pesanan_konsumen`, dan finalisasi pesanan | frontend, backend |
| `../product/period_workflow.md` | lifecycle periode aktif dan penutupan | frontend, backend |
| `../contracts/schema_mapping.md` | tabel, relasi, formula, constraint | backend, reviewer data |
| `../contracts/business_contracts.md` | kontrak RPC/action write | backend, integrator |
| `../contracts/query_contracts.md` | kontrak view/read model | frontend, backend |
| `../contracts/integration_contract_pack.md` | shape payload, enum, envelope, error code | frontend, backend |
| `./01-decision_log.md` | keputusan lintas-dokumen yang sudah dikunci | orchestrator, implementer |
| `./03-open_questions_register.md` | pertanyaan terbuka yang tidak boleh ditebak diam-diam | orchestrator, implementer |
| `../quality/testing_strategy.md` | pagar verifikasi per modul dan per layer | implementer, reviewer |
| `execution/frontend_plan.md` | blueprint UI mockup | frontend agent |
| `execution/backend_plan.md` | blueprint engine/core logic | backend agent |
| `execution/roadmap.md` | urutan eksekusi lintas area | orchestrator |
| `modules/*.md` | blueprint modul prioritas | implementer modul |

---

## 4. Peta Modul Bisnis

```mermaid
flowchart LR
    PER["Periode"] --> RES["Reseller"]
    PER --> MASTER["Master Periodik<br/>paket barang komisi akun kas"]
    RES --> KON["Konsumen"]
    PER --> PESANAN["Pesanan Konsumen"]
    KON --> PESANAN
    RES --> PESANAN
    MASTER --> DETAIL["Detail Pesanan / Finalisasi"]
    PESANAN --> DETAIL
    DETAIL --> SETK["Setoran Konsumen"]
    PESANAN --> SETK
    SETK --> SETP["Setoran Pusat"]
    RES --> SETP
    MASTER --> GUD["Gudang<br/>belanja packing pembagian pengiriman"]
    DETAIL --> GUD
    SETP --> KEU["Keuangan<br/>kas mutasi pencairan"]
    GUD --> KEU
    KEU --> LAP["Dashboard / Laporan"]
    SETK --> LAP
    SETP --> LAP
    DETAIL --> LAP
```

### Urutan Ketergantungan Nyata

1. `periode` adalah gerbang semua modul operasional
2. `reseller` dan `master periodik` harus cukup siap sebelum volume transaksi naik
3. `pesanan konsumen` adalah titik masuk operasional reseller
4. detail pesanan dan finalisasi tidak berdiri sendiri; keduanya bergantung pada pesanan konsumen dan master periode
5. `setoran konsumen` dan `setoran pusat` membentuk dua layer pelunasan
6. `gudang` bergantung pada pesanan aktif dan master barang/paket
7. `keuangan` bergantung pada setoran, kas, gudang, dan pencairan
8. `dashboard/laporan` datang paling akhir karena bergantung pada semua read model inti

---

## 5. Peta Modul Implementasi

```mermaid
flowchart TD
    M0["Foundation Docs + Contracts"] --> M1["Module Periode"]
    M1 --> M2["Module Reseller"]
    M2 --> M3["Module Pesanan Konsumen"]
    M3 --> M4["Module Detail Pesanan / Finalisasi"]
    M4 --> M5["Module Setoran"]
    M5 --> M6["Module Gudang"]
    M6 --> M7["Module Keuangan"]
    M7 --> M8["Module Dashboard / Laporan"]
```

### Implikasi untuk Eksekusi

- blueprint modul tidak perlu dibuat semuanya sekaligus
- rantai modul prioritas sekarang sudah lengkap dari `periode` sampai `dashboard/laporan`
- fokus berikutnya yang paling alami adalah eksekusi implementasi dan sinkronisasi read model
- `pesanan`, `detail/finalisasi`, dan `setoran` sebaiknya dipandang sebagai rantai kerja, bukan tiga pulau terpisah

Blueprint modul aktif saat ini:

- `modules/auth.md`
- `modules/master_periodik.md`
- `modules/periode.md`
- `modules/reseller.md`
- `modules/program_order.md`
- `modules/setoran.md`
- `modules/gudang.md`
- `modules/keuangan.md`
- `modules/dashboard_laporan.md`

---

## 6. Peta Entitas Data Inti

```mermaid
erDiagram
    PERIODE ||--o{ RESELLER_PERIODE : "mengikat"
    PERIODE ||--o{ PAKET : "memiliki"
    PERIODE ||--o{ BARANG_PERIODE : "memiliki"
    PERIODE ||--o{ KOMISI_CONFIG : "memiliki"
    PERIODE ||--o{ PESANAN_KONSUMEN : "melingkupi"
    PERIODE ||--o{ DETAIL_PESANAN_KONSUMEN : "melingkupi"
    PERIODE ||--o{ SETORAN_KONSUMEN : "melingkupi"
    PERIODE ||--o{ SETORAN : "melingkupi"
    PERIODE ||--o{ PENCAIRAN : "melingkupi"
    PERIODE ||--o{ BELANJA : "melingkupi"
    PERIODE ||--o{ PACKING : "melingkupi"
    PERIODE ||--o{ KAS_MASUK : "melingkupi"

    RESELLER ||--o{ KONSUMEN : "memiliki"
    RESELLER ||--o{ PESANAN_KONSUMEN : "mengelola"
    RESELLER ||--o{ SETORAN_KONSUMEN : "menerima"
    RESELLER ||--o{ SETORAN : "menyetor"
    RESELLER ||--o{ PENCAIRAN : "mencairkan"

    KONSUMEN ||--o{ PESANAN_KONSUMEN : "punya"
    PESANAN_KONSUMEN ||--o{ DETAIL_PESANAN_KONSUMEN : "berisi"
    PESANAN_KONSUMEN ||--o{ SETORAN_KONSUMEN : "menerima"

    PAKET ||--o{ DETAIL_PESANAN_KONSUMEN : "dipilih"
    PAKET ||--o{ DETAIL_PAKET : "punya bom"
    BARANG ||--o{ DETAIL_PAKET : "komponen"
    BARANG ||--o{ BARANG_PERIODE : "punya stok harga"

    DETAIL_PESANAN_KONSUMEN ||--o{ PEMBAGIAN_PAKET_DETAIL : "dibagikan"
    PEMBAGIAN_PAKET ||--o{ PEMBAGIAN_PAKET_DETAIL : "berisi"
```

### Makna Praktis untuk Agent

- `pesanan_konsumen` adalah parent operasional untuk tagihan/cicilan konsumen
- `detail_pesanan_konsumen` adalah daftar item paket di dalam pesanan
- `setoran_konsumen` harus selalu bisa dilacak ke `pesanan_konsumen`, `reseller`, dan `periode`
- `setoran` layer pusat harus selalu bisa dilacak ke `reseller` dan `periode`
- data gudang dan keuangan tidak boleh lepas dari konteks `periode`

---

## 7. Peta Boundary Frontend dan Backend

```mermaid
flowchart LR
    UI["Frontend Mockup<br/>pages components state"] --> MOCK["Dummy Data / Placeholder API"]
    UI --> CONTRACT["integration_contract_pack.md"]
    MOCK --> CONTRACT
    CONTRACT --> API["Internal API Route"]
    API --> SERVICE["Service Layer"]
    SERVICE --> WORKFLOW["Workflow / Validation"]
    WORKFLOW --> SQL["Supabase RPC / View / Table"]
```

### Aturan Boundary

- frontend hanya boleh bicara dalam shape kontrak yang sama
- placeholder API harus meniru envelope yang nanti dipakai route internal
- service layer tidak boleh bocor langsung ke UI
- business validation final tetap hidup di workflow / SQL boundary

---

## 8. Peta Dependency per Workstream

| Workstream | Bergantung pada | Menjadi fondasi untuk |
|---|---|---|
| `periode` | product truth + kontrak status | semua modul operasional |
| `reseller` | periode + auth/profile | konsumen, pesanan, setoran |
| `pesanan_konsumen` | periode + reseller + konsumen | detail item, setoran konsumen |
| `detail_pesanan_konsumen` | pesanan + paket periode | gudang, laporan, komisi |
| `setoran_konsumen` | pesanan + reseller + periode | status lunas konsumen, saldo reseller |
| `setoran` | reseller + periode + saldo terkumpul | status lunas reseller, kas |
| `gudang` | detail pesanan aktif/final + paket + barang periode | pengiriman, laporan stok |
| `keuangan` | setoran + gudang + kas masuk | pencairan, saldo, laporan |
| `laporan` | semua read model inti | dashboard dan audit operasional |

---

## 9. Rule Graphify untuk AI Coding Agent

Saat agent menerima task baru, lakukan cek cepat:

1. task ini berada di modul apa
2. modul itu bergantung pada modul mana
3. dokumen otoritatif apa yang harus dibaca dulu
4. entity apa yang menjadi pusat perubahan
5. apakah task menyentuh boundary frontend, backend, atau keduanya

Checklist minimum:

- jika task menyentuh status, cek workflow domain lebih dulu
- jika task menyentuh payload, cek `../contracts/integration_contract_pack.md`
- jika task menyentuh formula, cek `../contracts/schema_mapping.md` dan `../contracts/query_contracts.md`
- jika task menyentuh write action, cek `../contracts/business_contracts.md`
- jika task menyentuh modul prioritas, cek juga dokumen yang relevan di `modules/`

---

## 10. Catatan Arsitektur

Dokumen ini menetapkan:

- `./system_maps.md` menjadi index visual utama untuk hubungan antar dokumen, modul, dan entitas
- `../product/data_flow.md` tetap fokus ke alur operasional bisnis
- semua blueprint modul berikutnya sebaiknya menaut ke dokumen ini agar agent baru cepat orientasi

---

## 11. Catatan Penggunaan

Saat menggunakan dokumen ini:

1. baca peta otoritas dokumen untuk orientasi lintas layer
2. baca peta modul bisnis untuk memahami ketergantungan antar modul
3. baca peta entitas data untuk memahami relasi data inti
4. baru masuk ke blueprint modul atau execution plan yang relevan
