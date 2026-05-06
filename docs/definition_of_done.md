# Definition of Done
## Paket Lebaran Mumpuni

Dokumen ini menetapkan kriteria minimal agar suatu pekerjaan dianggap siap untuk direview.

Kriteria ini berlaku untuk pekerjaan dokumentasi, frontend, backend, dan integrasi, dengan penyesuaian sesuai scope tugas.

---

## 1. Kriteria Umum

Sebuah pekerjaan dianggap selesai jika:

- scope yang dikerjakan jelas
- perubahan mengikuti dokumen kontrak yang relevan
- tidak melanggar `docs/implementation_guardrails.md`
- hasil akhirnya bisa dijelaskan dengan ringkas dan jujur
- blocker, asumsi, dan risiko tersisa dilaporkan

---

## 2. Done untuk Dokumentasi

Dokumen dianggap selesai jika:

- isi dokumen konsisten dengan keputusan stack aktif
- tidak bertabrakan dengan sumber kebenaran yang lebih tinggi
- istilah penting dipakai secara konsisten
- referensi ke dokumen lain valid
- tidak menyisakan narasi transisi yang menyesatkan

---

## 3. Done untuk Frontend

Pekerjaan frontend dianggap selesai jika:

- mengikuti `docs/frontend_architecture.md`
- mengikuti `docs/frontend_component_contracts.md`
- mengikuti `docs/navigation_and_period_setup_ui.md` bila menyentuh route atau menu
- state loading, empty, error, disabled, dan success diperhitungkan
- tidak ada formula bisnis final yang dipindahkan ke browser
- admin dan reseller dipisahkan dengan jelas jika fiturnya role-specific
- kompatibel dengan shell Modernize yang relevan

---

## 4. Done untuk Database dan Data Layer

Pekerjaan database dianggap selesai jika:

- migration dapat dijalankan ulang dengan aman sesuai konteksnya
- aturan bisnis mengikuti `business_contracts.md`
- struktur mengikuti `schema_mapping.md`
- akses mengikuti `rls_matrix.md`
- query atau RPC tidak memaksa frontend menghitung formula final
- kebutuhan audit dan koreksi diperhitungkan untuk aksi penting

---

## 5. Done untuk UI Admin

Pekerjaan UI admin dianggap selesai jika:

- desktop-first
- informasi padat tetapi tetap mudah discan
- tabel besar masih usable
- status penting terlihat jelas
- perilaku periode mengikuti `navigation_and_period_setup_ui.md`

---

## 6. Done untuk UI Reseller

Pekerjaan UI reseller dianggap selesai jika:

- mobile-first
- aksi utama cepat dijangkau
- tidak ada horizontal scroll pada viewport mobile normal
- form nominal dan aksi simpan nyaman dipakai di HP
- feedback transaksi jelas

---

## 7. Quality Gate Minimum

Jika relevan dengan perubahan dan tool-nya tersedia, jalankan:

```bash
pnpm run lint
pnpm run typecheck
pnpm run build
```

Jika test sudah tersedia:

```bash
pnpm run test
```

Jika E2E sudah tersedia dan perubahannya menyentuh flow UI penting:

```bash
pnpm run e2e
```

Jika perubahan menyentuh Supabase:

```bash
pnpm exec supabase db reset
pnpm exec supabase gen types typescript --local > src/types/database.ts
```

Jika quality gate belum bisa dijalankan, alasannya harus dilaporkan.

Strategi test yang dipakai harus mengikuti:

- `docs/testing_strategy.md`

---

## 8. Laporan Selesai

Sebuah pekerjaan baru benar-benar dianggap selesai bila hasilnya dilaporkan dengan menyebut:

- apa yang dikerjakan
- apa yang belum dikerjakan
- file yang diubah
- command yang dijalankan
- verifikasi yang berhasil
- verifikasi yang belum bisa dijalankan
- risiko tersisa
