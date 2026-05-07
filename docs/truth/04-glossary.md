# Glossary
## Paket Lebaran Mumpuni

Dokumen ini menjadi glosarium istilah resmi proyek agar truth dan product layer memakai bahasa yang sama.

---

## 1. Istilah Resmi

| Istilah resmi | Istilah teknis | Istilah lama / tidak dipakai | Definisi | Contoh penggunaan | Dokumen terdampak |
|---|---|---|---|---|---|
| Pesanan Konsumen | `pesanan_konsumen` | `order` | Header tagihan/cicilan konsumen dalam satu periode | `Pesanan Konsumen` dibuat saat reseller mendaftarkan pilihan paket konsumen | `prd.md`, `program_workflow.md`, `data_flow.md`, `edge_cases.md` |
| Master Program | `master_program` bila nanti dipakai di schema / kode | `program` sebagai sinonim longgar | Template atau master yang menjadi acuan konfigurasi program | `Master Program` dipakai saat menjelaskan katalog atau definisi dasar | `prd.md`, `program_workflow.md`, `edge_cases.md` |
| Program Periode | `program_periode` bila nanti dipakai di schema / kode | `program` tanpa konteks | Program yang aktif dalam periode tertentu dan dipakai operasional | `Program Periode 2026` dipakai saat menjelaskan program aktif musim berjalan | `prd.md`, `program_workflow.md`, `period_workflow.md`, `edge_cases.md` |
| Belum Dikunci | `belum_dikunci` | `DIKUNCI` | Status UI untuk entitas yang belum final / belum dikunci | `Status: Belum Dikunci` pada layar finalisasi | `program_workflow.md`, `edge_cases.md` |
| Dikunci | `dikunci` | `DIKUNCI` | Status UI untuk entitas yang sudah final / terkunci | `Status: Dikunci` pada pesanan yang sudah final | `program_workflow.md`, `edge_cases.md` |

---

## 2. Catatan Pemakaian

- `order` hanya boleh muncul sebagai istilah lama, mapping teknis, atau penjelasan historis.
- `program` wajib diberi konteks ketika dipakai di dokumen produk.
- `DIKUNCI` tidak dipakai sebagai status teknis resmi.
- Istilah UI dan istilah teknis boleh berbeda, tetapi harus dijelaskan eksplisit.

---

## 3. Dokumen yang Paling Terdampak

- `docs/product/prd.md`
- `docs/product/program_workflow.md`
- `docs/product/period_workflow.md`
- `docs/product/data_flow.md`
- `docs/product/edge_cases.md`
- `docs/truth/01-decision_log.md`
- `docs/truth/03-open_questions_register.md`

---

## 4. Contoh Pola

- Benar: `Pesanan Konsumen` menjadi dasar tagihan reseller.
- Benar: `pesanan_konsumen` menyimpan header transaksi di database.
- Benar: `Master Program` dipakai untuk menjelaskan master/template.
- Benar: `Program Periode` dipakai untuk program aktif dalam periode berjalan.
- Benar: `Belum Dikunci` muncul sebagai label UI.
- Salah: memakai `DIKUNCI` sebagai enum teknis tanpa penjelasan.
