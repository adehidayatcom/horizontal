# Decision Log
## Paket Lebaran Mumpuni

Dokumen ini mencatat keputusan lintas-dokumen yang sudah dianggap resmi selama fase persiapan coding.

Jika ada konflik antar dokumen, keputusan di sini menjadi pegangan cepat sampai sumber dokumen terkait ikut diperbarui.

---

## DL-001 - Stack Frontend Resmi

- status: accepted
- keputusan: frontend resmi memakai `Next.js App Router + Modernize + Material-UI v7 + Emotion + SWR + Formik/Yup`
- alasan: konsisten dengan shell repo dan kontrak frontend aktif
- implikasi:
  - jangan gunakan `Shadcn UI` sebagai base system
  - komponen baru harus tunduk ke `frontend_component_contracts.md`
- rujukan:
  - ../frontend/frontend_architecture.md
  - ../frontend/frontend_component_contracts.md

## DL-002 - Route Strategy Resmi

- status: accepted
- keputusan: route admin menggunakan segment domain yang jelas seperti `/admin/master/*`, `/admin/gudang/*`, `/admin/keuangan/*`, `/admin/laporan/*`
- alasan: mencegah drift antara shell, menu, dan blueprint modul
- implikasi:
  - `navigation_and_period_setup_ui.md` menjadi referensi final untuk route dan menu
  - dokumen plan lain harus mengikuti naming tersebut
- rujukan:
  - ../frontend/navigation_and_period_setup_ui.md
  - ../frontend/frontend_architecture.md

## DL-003 - Root Route Reseller

- status: accepted
- keputusan: dashboard reseller menggunakan route root `/reseller`
- alasan: lebih sederhana dan konsisten untuk mobile-first entry
- implikasi:
  - jangan buat `/reseller/beranda` sebagai route utama baru
- rujukan:
  - ../frontend/frontend_architecture.md
  - ../modules/dashboard_laporan.md

## DL-004 - Route Setoran Reseller

- status: accepted
- keputusan: route setoran reseller memakai `/reseller/setor`
- alasan: sudah dipakai konsisten pada menu dan blueprint modul terbaru
- implikasi:
  - hindari pembuatan route alternatif `/reseller/setoran`
- rujukan:
  - ../frontend/navigation_and_period_setup_ui.md
  - ../contracts/integration_read_model_matrix.md

## DL-005 - Reporting Read Boundary

- status: accepted
- keputusan: file boundary laporan backend memakai nama `011_functions_dashboard_reports.sql` dan `dashboard-report.service.ts`
- alasan: memisahkan domain read dashboard/laporan dari naming generik lama
- implikasi:
  - kontrak read tambahan harus masuk ke `query_contracts.md`
- rujukan:
  - ../execution/backend_plan.md
  - ../contracts/query_contracts.md
  - ../modules/dashboard_laporan.md

## DL-006 - Auth Real Tetap Masuk Fase Coding Awal

- status: accepted
- keputusan: `auth real` tetap dianggap bagian fase coding awal, bukan ditunda seluruhnya
- alasan: role guard dan approval state memengaruhi banyak flow admin/reseller
- implikasi:
  - gunakan `modules/auth.md` sebagai blueprint resmi
- rujukan:
  - ../modules/auth.md
  - ../contracts/rls_matrix.md

## DL-007 - Master Periodik Adalah Modul Terpisah

- status: accepted
- keputusan: master data lintas periode dan per-periode diperlakukan sebagai modul khusus `master_periodik`
- alasan: area ini menjadi dependency periode, Program Periode/pesanan konsumen, gudang, dan keuangan
- implikasi:
  - jangan menyebarkan implementasi master periodik sebagai task sampingan tanpa plan modul
- rujukan:
  - ../modules/master_periodik.md
  - ../frontend/navigation_and_period_setup_ui.md

## DL-008 - Recent Activity Bersifat Deferred

- status: accepted
- keputusan: `recent activity` tidak menjadi requirement fase awal yang wajib diimplementasikan
- alasan: kontrak read-nya belum cukup penting untuk memblok implementasi inti
- implikasi:
  - jika mau dibangun, kontraknya harus dipastikan dulu
  - jangan menganggap widget ini mandatory pada first working slice
- rujukan:
  - ../contracts/query_contracts.md
  - ../contracts/integration_read_model_matrix.md
  - ../modules/dashboard_laporan.md

## DL-009 - Status `SELESAI` untuk Pesanan Konsumen

- status: accepted
- keputusan: `pesanan_konsumen` berubah ke status `SELESAI` saat periode ditutup secara resmi
- alasan: status `LUNAS` sudah mewakili pelunasan uang, sedangkan `SELESAI` dipakai sebagai status lifecycle akhir saat closing periode
- implikasi:
  - pesanan tidak otomatis `SELESAI` hanya karena lunas
  - proses closing periode harus membekukan pesanan yang valid secara historis ke status `SELESAI`
- rujukan:
  - ../product/program_workflow.md
  - ../product/period_workflow.md
  - ../contracts/schema_mapping.md

## DL-010 - Koreksi pada Periode `SELESAI`

- status: accepted
- keputusan: periode `SELESAI` bersifat read-only historis; hanya koreksi administratif non-finansial yang boleh dilakukan langsung
- alasan: mencegah perubahan historis diam-diam pada uang, stok, status, dan hak reseller
- implikasi:
  - koreksi transaksi uang, stok, pesanan konsumen, dan pencairan tidak boleh mengubah row historis secara langsung
  - kebutuhan koreksi material harus lewat jalur adjustment/audit yang eksplisit
- rujukan:
  - ../product/period_workflow.md
  - ../contracts/schema_mapping.md
  - ../contracts/business_contracts.md

## DL-011 - Laporan P1 Bertahap

- status: accepted
- keputusan: tidak semua laporan P1 wajib hadir di gelombang coding pertama
- alasan: gelombang pertama cukup menutup dashboard dan laporan operasional inti tanpa menunggu report kaya atau export lengkap
- implikasi:
  - laporan inti diprioritaskan lebih dulu
  - report tambahan, export, dan widget pendukung boleh menyusul
- rujukan:
  - ../product/prd.md
  - ../modules/dashboard_laporan.md

## DL-012 - Status `AKTIF/NONAKTIF` pada `akun_kas`

- status: accepted
- keputusan: `akun_kas` memiliki status aktif/nonaktif sejak fase awal
- alasan: akun lama harus bisa disimpan untuk histori tanpa tetap tersedia untuk transaksi baru
- implikasi:
  - akun nonaktif tidak boleh dipakai transaksi baru
  - histori lama tetap boleh menampilkan akun nonaktif
- rujukan:
  - ../contracts/schema_mapping.md
  - ../modules/master_periodik.md
  - ../modules/keuangan.md

## DL-013 - Read Model Khusus untuk Monitoring Setoran Admin

- status: accepted
- keputusan: monitoring setoran admin memakai read model khusus, bukan hanya query generik
- alasan: admin monitoring membutuhkan filter, status, dan agregasi yang stabil tanpa mendorong frontend menghitung sendiri
- implikasi:
  - minimal harus ada read model untuk monitoring setoran konsumen admin
  - minimal harus ada read model untuk monitoring setoran pusat admin
- rujukan:
  - ../contracts/query_contracts.md
  - ../modules/setoran.md
  - ../contracts/integration_read_model_matrix.md

## DL-014 - Historis Read Model Berbasis Snapshot

- status: accepted
- keputusan: read model historis harus mengutamakan snapshot data transaksi, bukan join mentah ke master aktif
- alasan: nilai historis tidak boleh ikut berubah ketika master diedit di kemudian hari
- implikasi:
  - field penting saat event terjadi harus disimpan atau dibekukan pada boundary historis
  - join ke master aktif hanya untuk data non-historis yang aman
- rujukan:
  - ../contracts/schema_mapping.md
  - ../contracts/query_contracts.md
  - ../truth/02-canonical_system_brief.md

## DL-015 - Batch Pembagian Campuran Ditolak

- status: accepted
- keputusan: batch pembagian final tidak boleh mencampur item `SIAP` dan `KURANG`
- alasan: submit parsial campuran membuat audit dan hasil operasional ambigu
- implikasi:
  - UI boleh menampilkan campuran saat review
  - backend menolak submit final jika masih ada item `KURANG`
  - operator harus memecah batch bila perlu
- rujukan:
  - ../modules/gudang.md
  - ../contracts/business_contracts.md

## DL-016 - Tooling Test Final

- status: accepted
- keputusan: unit/integration test memakai `Vitest`, dan E2E memakai `Playwright`
- alasan: kombinasi ini paling cocok untuk Next.js, TypeScript, validasi, route internal, dan flow role-based lintas desktop/mobile
- implikasi:
  - baseline environment dan testing strategy harus memakai `Vitest` dan `Playwright`
  - task verifikasi modul boleh mengacu ke tool ini secara eksplisit
- rujukan:
  - ../quality/testing_strategy.md
  - ../setup/environment.md

## DL-017 - Audit Admin Satu Halaman Bertab

- status: accepted
- keputusan: `admin/laporan/audit` tetap menjadi satu halaman admin-only dengan dua tab utama: `Audit Log` dan `Riwayat Koreksi`
- alasan: audit dan koreksi adalah satu konteks baca yang sama, sehingga filter periode dan filter jenis transaksi tidak perlu dipecah ke dua route
- implikasi:
  - frontend tidak membuat dua screen audit terpisah pada fase awal
  - route audit hanya satu, dengan state tab di dalam screen
- rujukan:
  - ../ui/admin_dashboard_uiux.md
  - ../modules/dashboard_laporan.md
  - ../contracts/integration_read_model_matrix.md

## DL-018 - Riwayat Reseller Fase Awal Tetap Per Modul

- status: accepted
- keputusan: area reseller fase awal tidak memakai riwayat transaksi gabungan; riwayat tetap tersebar per modul yang relevan
- alasan: fokus fase awal reseller adalah follow-up cepat, bukan pusat histori lintas domain
- implikasi:
  - beranda reseller tidak bergantung pada unified history feed
  - jika riwayat dibangun, ia memakai query per modul seperti setoran atau pesanan konsumen
- rujukan:
  - ../ui/reseller_uiux.md
  - ../modules/setoran.md
  - ../contracts/integration_read_model_matrix.md

## DL-019 - Watchlist Fase Awal Dihitung On-Demand

- status: accepted
- keputusan: summary dashboard dan watchlist fase awal dihitung on-demand melalui route/service/workflow, bukan background job nyata
- alasan: jalur ini paling sederhana untuk first coding wave dan paling kecil risiko over-engineering
- implikasi:
  - folder `background/` tetap boleh ada sebagai jalur ekspansi
  - `watchlist-refresh.job.ts` dan job sejenis bukan deliverable wajib gelombang pertama
- rujukan:
  - ../execution/backend_plan.md
  - ../modules/dashboard_laporan.md

## DL-020 - Unified Reseller History Deferred

- status: accepted
- keputusan: unified reseller history read model tidak dibangun pada fase awal; kontrak read reseller tetap modular per layar
- alasan: read model gabungan akan menambah coupling lintas modul tanpa menjadi blocker operasional awal
- implikasi:
  - query riwayat reseller boleh ditambahkan per modul saat benar-benar dibutuhkan
  - jangan mengasumsikan adanya satu activity feed reseller lintas transaksi pada first slice
- rujukan:
  - ../contracts/query_contracts.md
  - ../ui/reseller_uiux.md
  - ../contracts/integration_read_model_matrix.md

## DL-021 - Frontend Nyata Lewat Internal API Route

- status: accepted
- keputusan: frontend nyata memakai internal API route untuk semua read dan write operasional; direct Supabase browser client hanya dipakai untuk auth/session bootstrap atau helper shell yang tidak memuat kontrak bisnis
- alasan: jalur ini paling aman untuk konsistensi contract, role guard, observability, dan mencegah query liar tersebar di komponen
- implikasi:
  - screen bisnis admin dan reseller tidak memanggil view/RPC operasional langsung dari browser
  - read sederhana tetap lewat route internal jika datanya berada dalam domain bisnis proyek
  - penggunaan browser Supabase client harus dibatasi ke auth/session atau helper non-bisnis yang sudah disetujui
- rujukan:
  - ../contracts/api_integration.md
  - ../frontend/frontend_architecture.md
  - ../contracts/integration_contract_pack.md

## DL-022 - Create Pesanan Fase Awal Berpusat di Reseller

- status: accepted
- keputusan: create `pesanan_konsumen` fase awal berpusat di reseller; admin hanya membantu manual untuk kasus pengecualian
- alasan: ini paling dekat dengan alur lapangan dan menjaga admin tidak menjadi bottleneck operasional harian
- implikasi:
  - CTA create pesanan utama berada di area reseller
  - admin boleh punya jalur bantu manual, tetapi bukan flow utama
- rujukan:
  - ../modules/program_order.md
  - ../ui/reseller_uiux.md

## DL-023 - Onboarding Reseller Dua Jalur

- status: accepted
- keputusan: fase awal memakai dua jalur onboarding reseller: admin dapat membuat reseller, dan reseller dapat self-register lalu masuk status `PENDING`
- alasan: ini paling aman untuk mendukung kebutuhan lapangan tanpa memaksa satu jalur tunggal
- implikasi:
  - auth flow dan approval flow tetap perlu memisahkan `PENDING` vs `AKTIF`
  - area reseller `PENDING` tetap dibatasi read-only sesuai kontrak auth
- rujukan:
  - ../modules/auth.md
  - ../modules/reseller.md

## DL-024 - Status Selesai Pembagian Manual

- status: accepted
- keputusan: status `SELESAI` pada pembagian tetap diisi manual admin, bukan otomatis
- alasan: status manual lebih aman untuk audit operasional dan menghindari auto-close yang salah konteks
- implikasi:
  - workflow gudang tidak boleh mengasumsikan semua item `DISERAHKAN` berarti batch otomatis selesai
  - acceptance criteria modul gudang harus menampilkan aksi final admin dengan jelas
- rujukan:
  - ../modules/gudang.md
  - ../contracts/business_contracts.md

## DL-025 - Pencairan Dipisah per Jenis

- status: accepted
- keputusan: pencairan `TABUNGAN` dan `KOMISI` tetap dipisah per jenis transaksi
- alasan: pemisahan ini paling aman untuk audit, saldo, dan penelusuran histori
- implikasi:
  - form keuangan tidak mencampur dua sumber hak dalam satu submit
  - contract `pencairan` tetap eksplisit per jenis
- rujukan:
  - ../modules/keuangan.md
  - ../contracts/business_contracts.md

## DL-026 - Mutasi Kas Lintas Periode Dilarang

- status: accepted
- keputusan: mutasi kas lintas periode dilarang pada fase awal
- alasan: larangan ini menjaga saldo periode tetap mudah diaudit dan menahan kompleksitas rekonsiliasi
- implikasi:
  - boundary mutasi kas harus menolak akun/periode silang
  - route keuangan tidak boleh membuat workaround lintas periode diam-diam
- rujukan:
  - ../modules/keuangan.md
  - ../contracts/schema_mapping.md

## DL-027 - Warning Akun Kas Tidak Perlu Tambahan

- status: accepted
- keputusan: perubahan akun kas saat periode `AKTIF` tidak memerlukan warning tambahan di luar guard umum yang sudah ada
- alasan: warning utama tetap diprioritaskan untuk master periodik yang langsung memengaruhi tagihan dan stok
- implikasi:
  - UI master periodik tidak perlu menambahkan friction berlebih pada akun kas
  - audit perubahan akun kas tetap mengikuti mekanisme umum
- rujukan:
  - ../modules/master_periodik.md
  - ../modules/keuangan.md

## DL-028 - Budget Belanja Tidak Wajib di Fase Awal

- status: accepted
- keputusan: `barang_periode.budget_belanja` tidak wajib di fase awal
- alasan: field ini bukan blocker operasional first wave dan aman ditunda
- implikasi:
  - form `barang_periode` tidak boleh memaksa input budget untuk create awal
  - setup periode tetap valid tanpa budget belanja
- rujukan:
  - ../modules/master_periodik.md
  - ../modules/periode.md

## DL-029 - Edit Pesanan Tanpa Draft Persisten

- status: accepted
- keputusan: UI perubahan `detail_pesanan_konsumen` fase awal cukup memakai selection ephemeral, tanpa draft persisten
- alasan: ini menahan kompleksitas state frontend dan cukup untuk first wave
- implikasi:
  - komponen edit pesanan tidak perlu menyimpan draft lintas session
  - perubahan dianggap final hanya saat user menekan simpan
- rujukan:
  - ../modules/program_order.md
  - ../ui/admin_dashboard_uiux.md

## DL-030 - Watchlist Fase Awal Cukup Visual Kandidat

- status: accepted
- keputusan: watchlist `PERLU_PERHATIAN` fase awal cukup tampil sebagai kandidat visual tanpa action set admin lengkap
- alasan: kebutuhan operasional awal lebih banyak pada visibilitas daripada otomasi tindakan
- implikasi:
  - layar watchlist tidak perlu memuat semua aksi admin lanjutan pada first wave
  - status final tetap berubah lewat boundary resmi
- rujukan:
  - ../modules/program_order.md
  - ../ui/admin_dashboard_uiux.md

## DL-031 - Riwayat Awal Satu Daftar Campuran Berlabel

- status: accepted
- keputusan: riwayat reseller fase awal cukup satu daftar campuran berlabel, bukan tab terpisah `setoran konsumen` dan `setor pusat`
- alasan: ini paling ringan untuk mobile tanpa memaksa unified history feed lintas domain
- implikasi:
  - UI riwayat reseller boleh memakai satu daftar modular berlabel
  - kontrak read tetap modular dan tidak berubah menjadi satu feed backend gabungan
- rujukan:
  - ../modules/setoran.md
  - ../ui/reseller_uiux.md

## DL-032 - Cloning Periode Baru

- status: accepted
- keputusan: cloning massal paket dan BOM dari periode sebelumnya masuk scope implementasi awal dan menjadi bagian setup periode
- alasan: execution plan periode dan wizard setup memang membutuhkan jalur cloning yang seragam supaya setup periode tidak terpecah ke dua pola
- implikasi:
  - setup periode, master periodik, dan wizard aktivasi boleh mengasumsikan cloning tersedia
  - teks workflow periode dapat menulis cloning sebagai langkah aktif, bukan asumsi kondisional
- rujukan:
  - ../product/period_workflow.md

## DL-036 - Otoritas Aksi Periode dan Program Periode

- status: accepted
- keputusan: pada fase awal, hanya admin yang berwenang mengaktifkan, menutup, mengunci, membuka, membatalkan, atau mengubah periode/Program Periode
- alasan: ownership aksi harus tegas supaya boundary role, approval, dan guard implementasi tidak ambigu
- implikasi:
  - UI dan backend dapat menulis guard admin-only secara eksplisit
  - reseller tidak memiliki jalur langsung untuk mengubah status periode/Program Periode
- rujukan:
  - ../product/period_workflow.md
  - ../product/program_workflow.md

## DL-037 - Istilah Resmi Pesanan Konsumen

- status: accepted
- keputusan: istilah resmi dokumen/UI untuk entitas transaksi utama adalah `Pesanan Konsumen`; istilah teknisnya `pesanan_konsumen`; istilah lama `order` hanya boleh dipakai saat menjelaskan mapping teknis atau istilah historis
- alasan: penamaan terbaru sudah dipakai sebagai arah final agar konsisten lintas dokumen dan UI
- implikasi:
  - istilah `order` tidak boleh diperlakukan sebagai source of truth
  - dokumen product dan turunan nanti harus dinormalisasi ke `Pesanan Konsumen` / `pesanan_konsumen`
- rujukan:
  - ../product/prd.md
  - ../product/program_workflow.md
  - ../product/data_flow.md
  - ../truth/04-glossary.md

## DL-038 - Istilah Program

- status: accepted
- keputusan: istilah `program` tidak dipakai sebagai sinonim longgar; jika merujuk ke template/master gunakan `Master Program`, jika merujuk ke program yang aktif dalam periode tertentu gunakan `Program Periode`, dan jika konteks umum produk dipakai harus ada definisi yang jelas
- alasan: penamaan ini sudah menutup ambiguitas istilah program tanpa mengubah business rule
- implikasi:
  - dokumen yang memakai `program` harus menjelaskan konteksnya
  - istilah `program` tanpa definisi tidak boleh menjadi source of truth
- rujukan:
  - ../product/prd.md
  - ../product/program_workflow.md
  - ../product/edge_cases.md
  - ../truth/04-glossary.md

## DL-039 - Status Kunci Resmi

- status: accepted
- keputusan: istilah UI resmi untuk status kunci adalah `Belum Dikunci` dan `Dikunci`; istilah teknis resmi adalah `belum_dikunci` dan `dikunci`; format lama `DIKUNCI` tidak dipakai sebagai enum teknis
- alasan: penamaan terbaru sudah dianggap keputusan yang diterima agar status lifecycle dan UI konsisten
- implikasi:
  - dokumen yang masih memakai format lama perlu dinormalisasi istilahnya
  - `DIKUNCI` hanya boleh muncul sebagai referensi historis atau catatan normalisasi istilah
- rujukan:
  - ../product/program_workflow.md
  - ../product/edge_cases.md
  - ../truth/04-glossary.md
