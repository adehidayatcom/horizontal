# Execution Risk Register
## Paket Lebaran Mumpuni

Dokumen ini mencatat risiko implementasi yang paling relevan untuk repo `horizontal`.

Dokumen ini fokus pada risiko eksekusi, bukan risiko bisnis eksternal.

---

## 1. Risiko: Benturan Kontrak Data

**Severity:** Tinggi  
**Probabilitas:** Menengah

Deskripsi:

- shape data frontend, query, dan RPC dapat melenceng satu sama lain
- dampaknya adalah rewrite komponen, hook, atau service layer

Mitigasi:

- jaga `schema_mapping.md`, `query_contracts.md`, dan generated types tetap sinkron
- jangan membiarkan dummy boundary berkembang liar
- review output RPC sebelum banyak UI bergantung padanya

---

## 2. Risiko: Shell Global Tercemar Kebutuhan Lokal

**Severity:** Tinggi  
**Probabilitas:** Menengah

Deskripsi:

- perubahan untuk satu halaman bisa merusak theme, navigation, atau customizer global

Mitigasi:

- bedakan pekerjaan shell vs pekerjaan feature
- setiap perubahan shell harus punya dampak lintas halaman yang jelas
- gunakan `docs/ui/modernize_shell_guide.md` sebagai pagar

---

## 3. Risiko: RLS dan Access Model Tidak Sinkron

**Severity:** Tinggi  
**Probabilitas:** Menengah

Deskripsi:

- frontend bisa terlihat benar tetapi akses data nyata salah
- reseller berisiko melihat data yang tidak semestinya atau justru terblokir total

Mitigasi:

- jadikan `rls_matrix.md` sebagai referensi aktif
- uji skenario admin vs reseller sejak awal
- jangan menunda verifikasi access model sampai akhir

---

## 4. Risiko: Timeline Terseret oleh Fitur Sekunder

**Severity:** Menengah  
**Probabilitas:** Menengah

Deskripsi:

- dashboard kaya, laporan, atau polish visual dapat menggeser fokus dari transaksi inti

Mitigasi:

- ikuti prioritas di `execution/roadmap.md`
- selesaikan modul inti lebih dulu
- tunda area sekunder bila fondasi belum aman

---

## 5. Risiko: Dokumen Aktif dan Eksekusi Kembali Bercampur

**Severity:** Menengah  
**Probabilitas:** Rendah

Deskripsi:

- kontrak aktif bisa tercampur lagi dengan catatan kerja sementara

Mitigasi:

- pertahankan pemisahan antara product truth, build contract, UI reference, dan execution plan
- update dokumen aktif dengan sengaja, bukan lewat catatan ad hoc

---

## 6. Risiko: Quality Gate Terlambat

**Severity:** Menengah  
**Probabilitas:** Menengah

Deskripsi:

- lint, typecheck, build, atau verifikasi data baru dijalankan ketika perubahan sudah terlalu besar

Mitigasi:

- jalankan quality gate bertahap
- laporkan hal yang belum bisa diverifikasi
- jangan menunggu semua fitur selesai untuk mulai verifikasi
