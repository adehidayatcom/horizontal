# Execution Task Kit Standard
## Paket Lebaran Mumpuni

Dokumen ini menetapkan format baku `RTK + Caveman` untuk semua blueprint modul di folder `modules/`.

Makna istilah:

- `RTK` = `Ready-to-Code Task Kit`
- `Caveman` = versi kerja paling sederhana yang tetap valid secara arsitektur

Tujuan:

- membuat blueprint modul lebih siap dieksekusi AI coding agent
- mencegah task terlalu besar dan terlalu kabur
- menjaga setiap modul punya `first_working_slice` yang realistis

---

## 1. Struktur Minimum Dokumen Modul

Setiap dokumen modul di folder `modules/` ke depan minimal harus punya bagian berikut:

1. tujuan modul
2. scope frontend dan backend
3. integration contract modul
4. struktur folder dan file
5. hubungan antar file
6. breakdown component/function/class
7. task implementation detail
8. pseudocode / flow penting
9. `first_working_slice`
10. `task_execution_batches`
11. rule khusus AI coding agent
12. urutan eksekusi modul
13. risiko, asumsi, pertanyaan konfirmasi

---

## 2. Aturan RTK

Task implementation dianggap `RTK-ready` bila:

- nama task jelas
- tujuan task tunggal
- file yang dibuat/diubah disebut eksplisit
- lokasi file disebut eksplisit
- langkah kerja AI agent berurutan
- dependency disebut
- output yang diharapkan disebut
- acceptance criteria bisa diverifikasi

Task tidak boleh:

- mencampur frontend dan backend tanpa alasan kuat
- meminta rewrite modul lain yang belum jadi dependency resmi
- bergantung pada “nanti lihat saja”

---

## 3. Aturan Caveman

Setiap modul wajib punya `first_working_slice` yang:

- sempit
- demonstrable
- tidak palsu secara arsitektur
- tidak memaksa semua edge case selesai di awal

`first_working_slice` bukan prototipe asal jalan. Ia harus:

- memakai istilah domain yang benar
- mengikuti kontrak aktif
- cukup kecil untuk diselesaikan cepat

Contoh pola:

- modul `periode`: list, create, activate readiness minimum
- modul `reseller`: list, detail singkat, approval status, create/edit konsumen dasar
- modul `setoran`: cari konsumen, input nominal, simpan setoran konsumen

---

## 4. Aturan Batch Eksekusi

Setiap modul sebaiknya membagi task ke batch:

1. `batch_a_foundation`
2. `batch_b_mock_or_read`
3. `batch_c_primary_actions`
4. `batch_d_backend_boundary`
5. `batch_e_alignment_and_hardening`

Tidak semua modul harus memakai nama batch yang persis sama, tetapi urutannya harus mudah dipahami.

---

## 5. Rule untuk AI Coding Agent

Saat membaca dokumen modul di folder `modules/`, agent harus:

1. kerjakan `first_working_slice` dulu bila user meminta mulai cepat
2. ikuti batch eksekusi, jangan lompat ke task jauh bila dependency belum siap
3. baca dokumen otoritatif sebelum coding
4. jangan memperluas scope di luar task tanpa alasan yang jelas
5. jika menemukan konflik dokumen, berhenti di kontrak yang lebih tinggi

---

## 6. Catatan Format Modul

Berdasarkan standar ini:

- `modules/periode.md` harus memiliki section `first_working_slice` dan batch eksekusi
- modul-modul berikutnya harus langsung mengikuti format RTK + Caveman ini sejak dibuat
