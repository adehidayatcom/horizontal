# Execution Work Breakdown
## Paket Lebaran Mumpuni

Dokumen ini memecah pekerjaan implementasi menjadi area tanggung jawab, bukan per nama agent tertentu.

Tujuannya agar pekerjaan bisa didelegasikan ke siapa pun tanpa mengikat ke tooling atau model tertentu.

---

## 1. Workstream Utama

### Workstream A: Shell dan Frontend Foundation

Tanggung jawab:

- layout global
- navigation
- theme integration
- komponen shared
- route foundation

Output:

- shell siap dipakai banyak halaman
- admin dan reseller terpisah jelas

### Workstream B: Data Foundation

Tanggung jawab:

- migration
- RLS
- seed
- generated types

Output:

- fondasi data aman dan bisa diuji

### Workstream C: Business Transaction Layer

Tanggung jawab:

- RPC transaksi
- validasi boundary
- service wrappers

Output:

- transaksi inti dapat dipanggil secara konsisten

### Workstream D: Feature Screens

Tanggung jawab:

- halaman admin
- halaman reseller
- wiring UI ke data layer

Output:

- flow bisnis utama bisa dijalankan dari UI

### Workstream E: Verification dan Hardening

Tanggung jawab:

- lint
- typecheck
- build
- test yang tersedia
- review state dan risiko

Output:

- kualitas minimum sebelum review atau handoff

---

## 2. Pola Delegasi

Pekerjaan boleh didelegasikan bila:

- scope file dan ownership jelas
- dependensi sudah diketahui
- output yang dibutuhkan spesifik

Pekerjaan tidak layak didelegasikan bila:

- masih kabur secara kontrak
- sangat bergantung pada keputusan arsitektur yang belum dibuat
- blokirnya ada pada dokumen atau keputusan, bukan coding

---

## 3. Handover Minimum antar Workstream

Setiap handoff minimal menyebut:

- output yang sudah selesai
- file yang relevan
- asumsi aktif
- risiko tersisa
- pekerjaan lanjutan yang dibutuhkan

---

## 4. Checklist Tugas

Setiap tugas sebaiknya memiliki:

- tujuan
- input dan dependensi
- file target
- definisi selesai
- verifikasi minimum

Jika salah satu belum jelas, tugas belum matang untuk dikerjakan.
