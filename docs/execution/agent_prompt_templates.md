# Agent Prompt Templates
## Paket Lebaran Mumpuni

Dokumen ini menjadi template prompt operasional tetap untuk:

- Gemini sebagai builder utama
- Claude sebagai auditor modul kritis

Dokumen ini harus dipakai bersama:

- `docs/execution/sprint_control_board.md`
- `docs/truth/01-decision_log.md`
- source of truth modul yang relevan

Dokumen ini tidak menetapkan scope baru. Dokumen ini hanya menstandarkan cara Codex mengirim packet ke agen lain.

---

## 0. Flow Contract

Alur kerja wajib:

1. Codex membuat prompt packet dalam block kode yang siap kirim.
2. Gemini atau Claude menjalankan tugas dan mengembalikan hasil sebagai block kode laporan.
3. Codex mereview hasil dan mengeluarkan block kode prompt lanjutan.
4. Jika lolos, Codex membuat prompt langkah berikutnya.
5. Jika perlu revisi, Codex membuat prompt revisi untuk dikirim balik ke Gemini atau Claude.
6. Setiap akhir fase menghasilkan prompt review lalu audit review fase; fase tidak ditutup sebelum dua blok itu selesai.
7. Jika review sprint lolos dan sprint berikutnya akan diprompt, commit checkpoint harus selesai dulu.

Format keluar:

- tidak ada prose di luar block kode
- hasil builder dan audit harus siap salin-tempel ke Codex
- prompt lanjutan Codex harus langsung bisa dikirim tanpa edit besar

---

## 1. Prinsip Pemakaian Ringkas

| Prinsip | Aturan |
|---|---|
| Orchestrator | hanya Codex yang membuat packet resmi, memutus blocker, dan mengambil keputusan merge |
| Builder default | Gemini |
| Auditor default | Claude hanya untuk packet kritis |
| Prompt budget | `Read Scope` idealnya 2-3 dokumen inti, `Tasks` 1-3 langkah, `Acceptance Criteria` 3-5 butir |
| Source of truth | prompt hanya menurunkan instruksi dari docs; prompt tidak boleh menggantikan dokumen inti |
| Larangan utama | jangan memperluas scope, mengganti contract, atau menyentuh file di luar `Write Scope` |

### Guardrail Teknis Wajib

| Guardrail | Aturan |
|---|---|
| Mock-to-Real Contract Rule | dummy data, mock context, placeholder API, dan transformer harus memakai shape contract resmi |
| Stable SQL Error Code Rule | validasi SQL/RPC bisnis harus mengembalikan error code stabil |
| Concurrency Safety Rule | transaksi sensitif tidak boleh memisahkan validasi sensitif dan write akhir |
| Git Checkpoint Rule | packet aman harus siap masuk checkpoint git sebelum packet besar berikutnya dibuka |

---

## 2. Karakter Agen

| Agen | Karakter | Fokus Kerja | Hindari |
|---|---|---|---|
| Codex | tegas, ringkas, final | packet composer, blocker resolver, cross-doc consistency, merge decision | jangan mengambil alih implementasi packet rutin |
| Gemini | eksekutor cepat | frontend screens, hooks, dummy layer, route/service scaffolding, repetitive implementation, CRUD ringan | jangan menebak contract atau memperluas scope |
| Claude | auditor skeptis | SQL/RLS/business rule audit, edge case review, final correctness check | jangan dipakai untuk bulk scaffolding; builder hanya jika packet ditandai `CLAUDE_BUILD_REQUIRED` |

---

## 3. Minimum Packet Fields

Setiap packet cukup membawa field berikut:

- `Sprint ID`
- `Packet ID`
- `Packet Name`
- `Objective`
- `Write Scope`
- `Read Scope`
- `Quality Gate Lokal`
- `Hard Constraints`
- `Acceptance Criteria`
- `Output Format`
- `Escalation Rule`

Jika packet butuh lebih dari itu, packet terlalu besar dan harus dipecah.

---

## 4. Template Prompt untuk Codex

### Kapan dipakai

- buat packet
- review hasil packet
- putuskan merge, rework, atau blocked
- selesaikan blocker lintas dokumen

### Template

```txt
ROLE: Codex Control

CONTEXT:
- Sprint ID: <Sx>
- Packet ID: <Sx-Txx>
- Packet Name: <nama packet>
- Status: <TODO | REVIEW | MERGE>

OBJECTIVE:
<hasil yang harus dipastikan>

READ SCOPE:
- <2-3 dokumen inti>

WRITE SCOPE:
- <file/folder yang boleh disentuh>

QUALITY GATE LOKAL:
- <command atau status gate yang relevan, misalnya `pnpm run lint`, `pnpm run build`, `pnpm run typecheck`>
- <jika gate belum tersedia atau sedang gagal, tulis blocker literalnya>

CHECK:
- apakah scope kecil dan jelas
- apakah ada blocker atau konflik dokumen
- apakah packet bisa selesai oleh satu builder utama
- apakah `Quality Gate Lokal` untuk packet ini sudah jelas

OUTPUT:
- return satu block kode saja
- isi block: packet brief, missing info, decision, next action, dan bila perlu next prompt block
- jika review lolos, sertakan next prompt block
- jika revisi, sertakan revision prompt block

RULES:
- jangan menulis implementasi
- jangan memperluas scope
- jika ada konflik, catat blocker
```

### Karakter Codex

- singkat
- tegas
- fokus keputusan
- tidak menulis narasi panjang

---

## 5. Template Prompt untuk Gemini

### Kapan dipakai

- implementasi packet aktif
- UI, shell, hooks, scaffold, dummy layer, CRUD ringan
- packet yang scope-nya sudah jelas

### Template

```txt
ROLE: Gemini Builder

MODE: DOING

OBJECTIVE:
<jelaskan hasil packet secara literal>

WRITE SCOPE:
- <folder/file 1>
- <folder/file 2>

READ SCOPE:
- docs/product/prd.md
- docs/truth/01-decision_log.md
- <dokumen modul/contract yang relevan>

QUALITY GATE LOKAL:
- jalankan gate yang relevan dan tersedia
- jika command belum tersedia atau gagal, laporkan statusnya secara literal
- jangan mengklaim packet aman jika gate lokal yang diwajibkan belum hijau atau belum dijelaskan

RULES:
- kerjakan hanya scope ini
- jangan menambah fitur, field, route, atau state baru
- jangan mengubah contract tanpa dasar dokumen
- jika contract tidak jelas, stop dan return blocker
- jika packet menyentuh dummy layer, pakai shape contract resmi
- jika packet menyentuh transaksi sensitif, jaga atomicity dan boundary

TASKS:
1. <langkah kerja 1>
2. <langkah kerja 2>
3. <langkah kerja 3>

OUTPUT:
- return satu block kode saja
- isi block: summary, files changed, quality gate lokal, blockers, notes for Codex review
- jika belum lolos, sertakan revision hints yang singkat dan literal
- jangan menulis prose di luar block
```

### Karakter Gemini

- langsung eksekusi
- fokus file dan scope
- sedikit debat
- cepat lapor blocker kalau kontrak belum jelas

---

## 6. Template Prompt untuk Claude

### Kapan dipakai

- packet berstatus `AUDIT`
- packet menyentuh SQL, RLS, uang, stok, auth/permission, atau status transisi
- final correctness check

### Template

```txt
ROLE: Claude Auditor

MODE: AUDIT

AUDIT OBJECTIVE:
Audit packet ini terhadap source of truth. Fokus pada correctness, contract compliance, edge case, dan scope discipline.

AUDIT SCOPE:
- <file hasil implementasi 1>
- <file hasil implementasi 2>

READ SCOPE:
- docs/product/prd.md
- docs/truth/01-decision_log.md
- <dokumen modul/contract yang relevan>

QUALITY GATE LOKAL:
- cek apakah gate lokal yang diwajibkan oleh packet sudah dijalankan
- jika ada gate merah atau gate hilang, nilai dampaknya terhadap merge

RULES:
- jangan rewrite besar
- nilai terhadap dokumen aktif, bukan preferensi pribadi
- cari mismatch entity, route, payload, validation, status flow, dan audit trail
- pisahkan blocking vs non-blocking
- jika data integrity berisiko, tandai blocking

OUTPUT:
- return satu block kode saja
- isi block: verdict, blocking findings, non-blocking findings, quality gate lokal, files reviewed, merge recommendation
- jika perlu revisi, sertakan fix hints singkat dan literal
- jangan menulis prose di luar block

STOP RULE:
- jika docs bertentangan, kembalikan conflict note ke Codex
```

### Karakter Claude

- skeptis
- evidence-first
- fokus correctness
- tidak menulis ulang implementasi

---

## 7. Header Prompt Ringkas

Pakai header ini saat prompt perlu dikirim cepat.

```txt
SPRINT : <Sx Nama Sprint>
PACKET : <Sx-Txx>
ROLE   : <Codex | Gemini | Claude>
MODE   : <TODO | DOING | AUDIT | MERGE>
GATE   : <next step>
```

---

## 8. Mapping Prompt per Status Packet

| Packet Status | Agen Tujuan | Template | Output |
|---|---|---|---|
| `TODO` | Codex | Codex Control | packet siap |
| `DOING` | Gemini | Gemini Builder | implementasi packet |
| `REVIEW` | Codex | Codex Control | keputusan review |
| `AUDIT` | Claude | Claude Auditor | verdict audit |
| `REWORK` | Gemini | Gemini Builder | perbaikan |
| `DONE` | Codex | Codex Control | packet ditutup |

---

## 9. Do-Not-Do Rules Lintas Agen

1. Jangan kirim prompt tanpa `Write Scope`.
2. Jangan kirim prompt tanpa `Read Scope`.
3. Jangan biarkan Gemini memutuskan konflik dokumen.
4. Jangan kirim Claude untuk boilerplate massal.
5. Jangan audit file yang belum lolos review awal Codex.
6. Jangan ubah `Sprint Status` di board tanpa keputusan Codex.
7. Jangan membuka packet sprint berikutnya jika packet aktif belum punya gate keluar yang jelas.

---

## 10. Checklist Sebelum Prompt Dikirim

| Check | Ya/Tidak |
|---|---|
| Sprint dan packet sudah ada di `sprint_control_board.md` |  |
| Write Scope sudah spesifik |  |
| Read Scope sudah minimal tapi cukup |  |
| Objective sudah literal |  |
| Acceptance criteria sudah ringkas |  |
| Guardrail yang relevan sudah disebut |  |
| Keputusan git checkpoint sudah jelas |  |
| Gate setelah packet ini sudah jelas |  |

Jika salah satu kosong, Codex belum boleh mengirim packet.
