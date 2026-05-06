# Product Truth Audit
## Paket Lebaran Mumpuni

Dokumen ini merekam hasil audit terhadap dokumen product truth utama untuk memastikan
AI coding agent bekerja di atas kontrak yang konsisten.

Untuk peta hubungan visual antar dokumen, modul, dan entitas, lihat:

- `docs/truth/system_maps.md`

Scope audit tahap 1:

- `docs/product/prd.md`
- `docs/contracts/business_contracts.md`
- `docs/contracts/schema_mapping.md`
- `docs/contracts/query_contracts.md`
- `docs/product/program_workflow.md`
- `docs/product/period_workflow.md`
- `docs/product/edge_cases.md`
- `docs/contracts/integration_contract_pack.md`

---

## 1. Ringkasan Status

Status umum: **cukup sehat sebagai truth-layer domain**, tetapi keputusan readiness coding final tetap mengikuti audit kesiapan dokumen terbaru.

Kesimpulan audit:

- fondasi model bisnis sudah konsisten pada alur `konsumen -> pesanan_konsumen -> detail_pesanan_konsumen -> setoran_konsumen -> finalisasi_pesanan_konsumen`
- kontrak enum inti sekarang sudah searah antara domain docs dan integration contract
- aturan periode aktif tunggal sudah konsisten di level PRD, workflow, schema, dan API guidance
- kontrak lintas layer untuk `snake_case`, response envelope, dan error code sudah cukup kuat
- masih ada beberapa area yang perlu dijaga sebagai keputusan eksplisit agar agent tidak berasumsi liar

---

## 2. Kebenaran Produk yang Sudah Terkunci

Hal-hal berikut dapat dianggap sebagai product truth aktif:

1. Sistem berjalan berbasis `periode`, dan hanya satu `periode.status = 'AKTIF'` pada satu waktu.
2. Konsumen tidak mulai dari item order lepas, tetapi dari `pesanan_konsumen` yang sudah memuat detail paket.
3. `setoran_konsumen` selalu masuk ke `pesanan_konsumen`, bukan langsung ke item detail.
4. `detail_pesanan_konsumen` sudah ada sejak awal dan pesanan menjadi siap operasional setelah `finalisasi_pesanan_konsumen`.
5. Semua transaksi operasional penting wajib membawa `periode_id`.
6. Semua hitungan yang menyangkut order, komisi, poin, tabungan, stok, target, dan kebutuhan wajib mengecualikan `status_kirim = 'BATAL'`.
7. Harga paket, BOM, dan komisi bersifat live dalam periode aktif, dengan perubahan hanya lewat aksi admin terkontrol.
8. Program dengan setoran tidak boleh langsung `BATAL`; jalur koreksi utamanya adalah `TERHENTI` bila bisnis tidak lanjut normal.
9. Reseller hanya boleh melihat dan mengubah data miliknya melalui `profile -> no_reseller`.
10. Frontend bukan sumber kebenaran business logic; validasi final berada di RPC / database function / read model.

---

## 3. Koreksi Aman yang Sudah Diterapkan

Audit ini juga menemukan beberapa frasa yang masih membawa jejak model lama. Koreksi aman yang
sudah diterapkan:

- wording di `docs/product/period_workflow.md` diselaraskan agar alur reseller tidak lagi terkesan "pesanan dulu"
- wording di `docs/product/data_flow.md` diselaraskan agar reseller `PENDING` diblok pada `pesanan/setoran`, bukan hanya `setoran`
- wording di `docs/contracts/schema_mapping.md` diselaraskan agar pendaftaran reseller dan tabel `konsumen` tidak menyiratkan order sebagai langkah awal tunggal

---

## 4. Gap yang Masih Ada

Gap berikut belum cukup berbahaya untuk memblokir planning, tetapi perlu dijaga saat implementasi:

### 4.1 Status akhir pesanan vs status periode

Yang sudah jelas:

- `pesanan_konsumen.status_pesanan` memiliki status domain sendiri
- `periode.status` memiliki lifecycle sendiri

Yang masih perlu disiplin implementasi:

- `pesanan_konsumen` dibekukan ke status `SELESAI` saat closing periode resmi
- status `SELESAI` dipakai sebagai lifecycle akhir historis, bukan sekadar indikator lunas

### 4.2 Batas koreksi periode selesai

Yang sudah jelas:

- periode `SELESAI` menolak input operasional baru
- koreksi admin masih mungkin melalui mekanisme khusus

Keputusan yang sekarang sudah dikunci:

- periode `SELESAI` hanya mengizinkan koreksi administratif non-finansial secara langsung
- koreksi material wajib lewat jalur audit/adjustment khusus, bukan edit row historis

### 4.3 Snapshot historis vs live recalculation

Yang sudah jelas:

- harga/BOM/komisi live dalam periode aktif
- histori periode lama dilindungi lewat pemisahan data periodik

Keputusan yang sekarang sudah dikunci:

- read model historis tidak boleh ikut berubah karena join ke master aktif
- boundary historis harus memakai snapshot data transaksi untuk field penting
- agent backend tetap harus ketat memfilter semua hitungan ke `periode_id` yang benar

### 4.4 Satu pesanan per konsumen per periode

Yang sudah jelas:

- MVP memakai `UNIQUE (periode_id, konsumen_id)`

Yang berarti untuk agent:

- frontend mock dan backend plan tidak boleh diam-diam mendesain multi-pesanan per konsumen dalam satu periode
- extensibility boleh disiapkan, tetapi perilaku aktif saat ini tetap single-pesanan per periode

---

## 5. Risiko Salah Tafsir yang Paling Besar

Jika tidak diawasi, AI coding agent paling mungkin salah di area berikut:

1. Menganggap reseller boleh membuat item pesanan liar tanpa `pesanan_konsumen`.
2. Menganggap `pesanan_konsumen.target_tagihan_snapshot` sebagai angka input bebas, padahal ia harus selalu berasal dari total `detail_pesanan_konsumen` yang masih dihitung.
3. Menghitung order `BATAL` ke dalam ringkasan reseller, stok, atau komisi.
4. Menggunakan `camelCase` atau enum baru yang tidak ada di kontrak integrasi.
5. Menganggap filter periode tampilan sama dengan mengganti periode aktif sistem.

---

## 6. Rule Kerja untuk AI Coding Agent

Saat membaca docs, agent harus memakai urutan mental berikut:

1. `docs/product/prd.md` untuk arah produk.
2. `docs/product/program_workflow.md` dan `docs/product/period_workflow.md` untuk lifecycle domain.
3. `docs/contracts/schema_mapping.md` untuk struktur data dan formula.
4. `docs/contracts/business_contracts.md` untuk aksi write/RPC.
5. `docs/contracts/query_contracts.md` untuk read model.
6. `docs/contracts/integration_contract_pack.md` untuk boundary lintas layer.

Agent harus berhenti dan menandai pertanyaan jika menemukan kebutuhan yang:

- meminta multi-pesanan per konsumen pada satu periode
- meminta input operasional baru pada periode `SELESAI`
- meminta bypass `finalisasi_pesanan_konsumen`
- meminta payload/status baru yang belum ada di kontrak

---

## 7. Kesimpulan Audit

Hasil audit ini:

- product truth sudah stabil dan siap digunakan sebagai fondasi dokumen implementasi
- tidak perlu rewrite besar pada dokumen inti
- semua blueprint modul harus tetap mengacu ke dokumen ini agar asumsi lama tidak kembali masuk

---

## 8. Dokumen Referensi Lanjutan

Dokumen yang relevan untuk dibaca setelah audit ini:

1. `support/system_maps.md` untuk peta hubungan antar dokumen dan modul
2. `truth/01-decision_log.md` untuk keputusan lintas-dokumen yang sudah dikunci
3. `execution/frontend_plan.md` dan `execution/backend_plan.md` untuk blueprint eksekusi
4. `modules/*.md` untuk blueprint modul prioritas
