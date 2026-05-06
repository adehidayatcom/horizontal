# Agent Prompt Templates
## Paket Lebaran Mumpuni

Dokumen ini menjadi template prompt operasional tetap untuk:

- Gemini sebagai builder utama
- Claude sebagai auditor modul kritis

Dokumen ini harus dipakai bersama:

- `docs/execution/sprint_control_board.md`
- `docs/support/decision_log.md`
- source of truth modul yang relevan

Dokumen ini tidak menetapkan scope baru. Dokumen ini hanya menstandarkan cara Codex mengirim packet ke agen lain.

---

## 1. Prinsip Pemakaian

| Prinsip | Aturan |
|---|---|
| Orchestrator tunggal | hanya Codex yang mengirim packet resmi |
| Builder default | Gemini |
| Auditor default | Claude pada packet kritis |
| Format kontrol | semua prompt wajib menyebut `Sprint ID`, `Packet ID`, `Write Scope`, `Acceptance Gate` |
| Source of truth | prompt tidak boleh menggantikan dokumen inti; prompt hanya menurunkan instruksi dari docs |
| Larangan utama | agen lain tidak boleh memperluas scope, mengganti contract, atau menyentuh file di luar `Write Scope` |

### Guardrail Teknis Wajib

| Guardrail | Aturan |
|---|---|
| Mock-to-Real Contract Rule | jika packet menyentuh dummy data, mock context, hook placeholder, atau placeholder API, semua shape harus mengacu ke contract resmi atau type tunggal yang diturunkan dari contract |
| Stable SQL Error Code Rule | jika packet menyentuh SQL/RPC validation, hasil error bisnis harus punya code stabil yang bisa dipetakan oleh service layer |
| Concurrency Safety Rule | jika packet menyentuh transaksi sensitif, prompt harus menyebut bahwa validasi dan write akhir tidak boleh dipisah secara rawan race condition |
| Git Checkpoint Rule | setelah packet mencapai titik aman, hasil harus siap masuk checkpoint git sebelum packet besar berikutnya dibuka |

---

## 2. Field Wajib di Setiap Packet

| Field | Wajib | Keterangan |
|---|---|---|
| `Sprint ID` | ya | contoh `S2` |
| `Packet ID` | ya | contoh `S2-T03` |
| `Packet Name` | ya | nama kerja singkat |
| `Current Status` | ya | `TODO`, `DOING`, `REVIEW`, `AUDIT`, `REWORK` |
| `Objective` | ya | hasil yang harus dicapai |
| `Write Scope` | ya | file/folder yang boleh disentuh |
| `Read Scope` | ya | dokumen yang wajib dibaca sebelum kerja |
| `Hard Constraints` | ya | larangan tegas |
| `Acceptance Criteria` | ya | kondisi lolos packet |
| `Output Format` | ya | format jawaban agen |
| `Escalation Rule` | ya | kapan agen harus berhenti dan mengembalikan blocker |

---

## 3. Template Prompt Standar untuk Gemini

### Kapan dipakai

- implementasi packet aktif
- kerja lebar tapi terarah
- frontend screens
- hooks
- route/service scaffolding
- boilerplate yang mengikuti contract jelas

### Template

```txt
ROLE: Gemini Builder

PROJECT:
Paket Lebaran Mumpuni

CONTROL:
CURRENT SPRINT : <Sprint ID dan nama sprint>
SPRINT STATUS  : ACTIVE
ACTIVE PACKET  : <Packet ID>
BUILDER        : Gemini
AUDITOR        : <Claude atau ->
NEXT GATE      : Codex review

PACKET:
Sprint ID      : <Sx>
Packet ID      : <Sx-Txx>
Packet Name    : <nama packet>
Current Status : DOING

OBJECTIVE:
<jelaskan hasil packet secara literal>

WRITE SCOPE:
- <folder/file 1>
- <folder/file 2>

DO NOT TOUCH:
- <folder/file yang dilarang>

READ SCOPE:
- docs/prd.md
- docs/support/decision_log.md
- docs/execution/sprint_control_board.md
- <dokumen modul/contract yang relevan>

SOURCE OF TRUTH:
- <doc 1>
- <doc 2>
- <doc 3>

IMPLEMENTATION RULES:
- Ikuti source of truth apa adanya.
- Jangan menambah fitur, state, endpoint, atau field di luar dokumen.
- Jangan mengubah nama entity, route, atau response shape tanpa dasar dokumen.
- Jangan menyentuh file di luar Write Scope.
- Jika menemukan konflik dokumen, berhenti dan laporkan, jangan menyelesaikan dengan asumsi sendiri.
- Jika butuh helper kecil yang implisit dari packet, buat hanya jika langsung mendukung objective packet ini.
- Jika packet menyentuh dummy layer, gunakan contract resmi atau type tunggal yang diturunkan dari contract, bukan shape inline.
- Jika packet menyentuh transaksi kritis, jangan memisahkan validasi sensitif dan write akhir ke langkah yang rawan race condition.
- Jika packet menyentuh SQL/RPC validation, kembalikan error bisnis dengan code stabil.

TASKS:
1. <langkah kerja 1>
2. <langkah kerja 2>
3. <langkah kerja 3>

DEPENDENCIES:
- <dependency 1>
- <dependency 2>

ACCEPTANCE CRITERIA:
- <kriteria 1>
- <kriteria 2>
- <kriteria 3>

OUTPUT FORMAT:
- Summary:
  <ringkasan hasil implementasi>
- Files changed:
  - <path>
  - <path>
- Notes for Codex review:
  - <catatan penting>
- Git checkpoint note:
  - <required | optional | hold>
- Blockers:
  - <isi `none` jika tidak ada>

ESCALATION RULE:
- Jika contract tidak cukup jelas, return blocker.
- Jika Write Scope tidak cukup, usulkan tambahan scope, jangan ambil sendiri.
- Jika ada keputusan produk baru, stop.
```

### Catatan penggunaan Gemini

| Situasi | Aturan |
|---|---|
| packet frontend | minta Gemini fokus pada page, component, hook, dan state yang tertulis |
| packet backend scaffolding | minta Gemini hanya membuat wrapper, route, validation, service shape yang sudah didefinisikan |
| packet multi-file | tetap batasi ke satu domain packet |
| packet kritis | tetap kirim ke Claude setelah review awal Codex |

---

## 4. Template Prompt Standar untuk Claude

### Kapan dipakai

- packet berstatus `AUDIT`
- modul kritis: pesanan, setoran, gudang, keuangan, SQL, RLS
- final correctness check
- review edge case dan auditability

### Template

```txt
ROLE: Claude Auditor

PROJECT:
Paket Lebaran Mumpuni

CONTROL:
CURRENT SPRINT : <Sprint ID dan nama sprint>
SPRINT STATUS  : AUDIT
ACTIVE PACKET  : <Packet ID>
BUILDER        : <Gemini atau Codex>
AUDITOR        : Claude
NEXT GATE      : Codex merge decision

PACKET:
Sprint ID      : <Sx>
Packet ID      : <Sx-Txx>
Packet Name    : <nama packet>
Current Status : AUDIT

AUDIT OBJECTIVE:
Audit packet ini terhadap source of truth. Fokus pada correctness, contract compliance, edge case, dan scope discipline.

AUDIT SCOPE:
- <file hasil implementasi 1>
- <file hasil implementasi 2>

READ SCOPE:
- docs/prd.md
- docs/support/decision_log.md
- docs/execution/sprint_control_board.md
- <dokumen modul/contract yang relevan>

SOURCE OF TRUTH:
- <doc 1>
- <doc 2>
- <doc 3>

AUDIT RULES:
- Jangan rewrite besar. Fokus audit.
- Jangan menilai dari preferensi pribadi. Nilai terhadap dokumen aktif.
- Cari mismatch entity, route, payload, validation, status flow, audit trail, dan edge case.
- Tandai jika builder over-engineer, under-build, atau menyentuh file di luar scope.
- Pisahkan finding menjadi blocking vs non-blocking.
- Jika packet menyentuh dummy layer, audit kepatuhan terhadap contract resmi.
- Jika packet menyentuh SQL/RPC validation, audit kestabilan error code bisnis.
- Jika packet menyentuh transaksi kritis, audit atomicity dan risiko race condition.

CHECKLIST:
1. Contract cocok dengan docs.
2. Naming entity/route/type konsisten.
3. Validation rule sesuai business rule.
4. State transition aman dan dapat diaudit.
5. Tidak ada scope creep.
6. Acceptance criteria packet terpenuhi.

OUTPUT FORMAT:
- Verdict:
  PASS | PASS WITH MINOR FIX | REWORK REQUIRED
- Blocking findings:
  - <isi `none` jika tidak ada>
- Non-blocking findings:
  - <isi `none` jika tidak ada>
- Files reviewed:
  - <path>
  - <path>
- Merge recommendation:
  - <siap merge / perlu rework / perlu packet baru>
- Git checkpoint recommendation:
  - <required | optional | hold>

ESCALATION RULE:
- Jika docs bertentangan, jangan memutuskan sendiri. Kembalikan conflict note ke Codex.
- Jika ada risiko data integrity, tandai sebagai blocking.
- Jika masalah hanya kosmetik, tandai non-blocking.
```

### Catatan penggunaan Claude

| Situasi | Aturan |
|---|---|
| audit SQL/RLS | minta Claude fokus pada correctness, access control, dan audit trail |
| audit frontend kritis | fokus ke state flow, contract usage, dan scope compliance, bukan selera visual |
| audit modul keuangan/gudang | prioritaskan edge case, status transition, dan integritas historis |
| token terbatas | kirim hanya file hasil packet dan source of truth minimal |

---

## 5. Template Ringkas Packet Header

Gunakan header ini sebelum mengirim prompt ke Gemini atau Claude.

```txt
CURRENT SPRINT : <Sx Nama Sprint>
SPRINT STATUS  : <ACTIVE | AUDIT | IN_REVIEW>
ACTIVE PACKET  : <Sx-Txx>
BUILDER        : <Gemini | Codex>
AUDITOR        : <Claude | ->
NEXT GATE      : <Codex review | Claude audit | Merge decision>
```

---

## 6. Tabel Mapping Prompt per Status Packet

| Packet Status | Agen Tujuan | Template yang Dipakai | Output yang Diharapkan |
|---|---|---|---|
| `TODO` | Codex internal | packet drafting internal | packet siap dikirim |
| `DOING` | Gemini | template builder | implementasi packet |
| `REVIEW` | Codex internal | review checklist internal | keputusan review |
| `AUDIT` | Claude | template auditor | verdict audit |
| `REWORK` | Gemini | template builder + bagian temuan audit | patch perbaikan |
| `DONE` | Codex internal | summary closeout | packet ditutup |

---

## 7. Do-Not-Do Rules Lintas Agen

1. Jangan kirim prompt tanpa `Write Scope`.
2. Jangan kirim prompt tanpa `Source of Truth`.
3. Jangan biarkan Gemini memutuskan konflik dokumen.
4. Jangan kirim Claude untuk boilerplate massal.
5. Jangan audit file yang belum lolos review awal Codex.
6. Jangan ubah `Sprint Status` di board tanpa keputusan Codex.
7. Jangan membuka packet sprint berikutnya jika packet aktif belum punya gate keluar yang jelas.

---

## 8. Checklist Sebelum Prompt Dikirim

| Check | Ya/Tidak |
|---|---|
| Sprint dan packet sudah ada di `sprint_control_board.md` |  |
| Write Scope sudah spesifik |  |
| Read Scope sudah minimal tapi cukup |  |
| Source of truth sudah jelas |  |
| Acceptance criteria sudah literal |  |
| Guardrail mock/API/error code/concurrency yang relevan sudah disebut |  |
| Keputusan git checkpoint untuk packet ini sudah jelas |  |
| Do-not-do rules sudah disebut |  |
| Output format agen sudah jelas |  |
| Gate setelah packet ini sudah jelas |  |

Jika salah satu kosong, Codex belum boleh mengirim packet.
