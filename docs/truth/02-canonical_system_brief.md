# Canonical System Brief
## Paket Lebaran Mumpuni

Dokumen ini adalah ringkasan final ("Source of Truth") yang mengintegrasikan PRD, schema, workflow, dan keputusan arsitektur. Dokumen ini berfungsi sebagai acuan cepat untuk memahami cara sistem bekerja secara utuh tanpa menciptakan aturan baru.

---

## 1. Ringkasan Sistem
Sistem manajemen tabungan berjangka berbasis web untuk mengelola paket Lebaran melalui jaringan reseller. Fokus utama adalah **auditabilitas**, **konsistensi data** (uang, stok, status), dan **efisiensi operasional** (admin desktop-first, reseller mobile-first).

- **Core Engine**: Sistem berjalan berbasis `periode`. Hanya satu periode yang boleh `AKTIF` secara operasional pada satu waktu.
- **Data Isolation**: Data dipisahkan antar reseller menggunakan RLS (Row Level Security).
- **Business Logic Ownership**: Frontend tidak menjadi sumber kebenaran logic bisnis. Semua operasi tulis dan read model kompleks dilakukan via Backend (RPC/Internal API Routes).

---

## 2. Struktur Data Utama (Entity Summary)

### Hierarki Transaksi
1.  **Periode**: Kontainer waktu operasional (misal: "Lebaran 2026").
2.  **Reseller**: Mitra lapangan yang mengelola konsumen.
3.  **Konsumen**: Pelanggan akhir milik reseller.
4.  **Pesanan Konsumen**: Header tagihan/cicilan konsumen per periode (1 konsumen = 1 pesanan aktif per periode).
5.  **Detail Pesanan Konsumen**: Item paket yang dipesan di dalam satu header pesanan.
6.  **Setoran Konsumen**: Layer 1 uang (Konsumen -> Reseller). Mengacu ke header pesanan.
7.  **Setoran Pusat**: Layer 2 uang (Reseller -> Admin/Pusat).

### Pendukung Operasional
- **Paket & Barang**: Master data per periode yang menentukan target tagihan dan kebutuhan stok.
- **Gudang & Packing**: Pengelolaan stok fisik dan perakitan paket komposit.
- **Keuangan**: Pengelolaan kas, mutasi, dan pencairan hak reseller (tabungan/komisi).

---

## 3. Workflow Utama (Alur Program)

1.  **Inisiasi**: Admin menyiapkan periode, master paket, dan reseller.
2.  **Pesanan Konsumen**: Reseller mendaftarkan konsumen dan membuat `pesanan_konsumen` berisi daftar `detail_pesanan_konsumen`.
3.  **Tabungan**: Konsumen mencicil (`setoran_konsumen`) ke reseller. Reseller menyetorkan dana terkumpul ke pusat (`setoran`).
4.  **Penyesuaian**: Detail pesanan dapat diubah (tambah/kurang/`TERHENTI`) sebelum periode ditutup jika budget konsumen berubah.
5.  **Finalisasi**: Reseller/Admin memfinalkan pesanan (`tanggal_final`). Pesanan masuk ke antrean gudang.
6.  **Operasional Gudang**: Belanja barang -> Packing -> Pembagian Paket ke Reseller.
7.  **Closing**: Periode ditutup, pesanan berubah menjadi `SELESAI` (read-only historis), hak reseller dihitung.

---

## 4. Peran dan Izin (Roles & Permissions)

### Admin (Desktop-First)
- **Akses**: Penuh ke semua modul.
- **Tanggung Jawab**: Master data, monitoring dashboard, approval, keuangan pusat, gudang, dan audit log.
- **Koreksi**: Admin adalah satu-satunya role yang bisa melakukan koreksi administratif pada data historis melalui boundary resmi.

### Reseller (Mobile-First)
- **Akses**: Terbatas pada data miliknya sendiri (OWN).
- **Tanggung Jawab**: Registrasi konsumen, input pesanan, catat setoran, dan pantau progres pelunasan.
- **Pembatasan**: Tidak bisa melihat data reseller lain, tidak bisa melihat detail biaya/budget internal admin, hanya bisa baca data periode yang aktif.

---

## 5. Aturan Bisnis & Teknis Krusial

### Perhitungan Target Tagihan
- **Formula**: `SUM(qty * harga_snapshot)` dari semua item aktif.
- **Status TERHENTI**: Item yang tidak tuntas tetap dihitung sebesar `uang_terhenti` (tidak dianggap 0).
- **Status BATAL**: Item tidak dihitung dalam target.

### Konsistensi Keuangan
- **Uang Snapshot**: Semua transaksi menggunakan snapshot harga/nilai saat kejadian untuk mencegah drift jika master data berubah di masa depan.
- **Audit Trail**: Setiap perubahan status penting atau transaksi uang wajib tercatat di `audit_log`.
- **Saldo Kas**: Kas dipisahkan per akun (Tunai, Bank, dll) dan mutasi lintas periode dilarang pada fase awal.

### Aturan Lifecycle
- **LUNAS vs SELESAI**: Pesanan yang lunas uangnya tetap berstatus `AKTIF` sampai periode ditutup secara resmi menjadi `SELESAI`.
- **Read-Only History**: Periode `SELESAI` bersifat beku; tidak boleh ada CRUD biasa yang mengubah nilai finansial atau stok.

---

## 6. Stack Teknologi
- **Frontend**: Next.js (App Router), Modernize Shell (MUI v7), SWR.
- **Backend**: Supabase (PostgreSQL, Auth, RLS, RPC).
- **Integration**: Internal API Routes sebagai boundary antara UI dan Supabase RPC.
- **Testing**: Vitest (Unit/Integration), Playwright (E2E).
