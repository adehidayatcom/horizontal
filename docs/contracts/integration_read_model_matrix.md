# Integration Read Model Matrix
## Paket Lebaran Mumpuni

Dokumen ini adalah matriks operasional untuk menghubungkan:

- halaman / screen
- komponen penting
- route internal
- view / RPC read model
- dependency modul
- state UI

Tujuannya agar AI coding agent tidak menebak sumber data per halaman dan tidak membuat hook atau
route liar di luar kontrak yang sudah ada.

Dokumen ini bukan kontrak bisnis baru. Semua field tetap tunduk pada:

- `docs/contracts/query_contracts.md`
- `docs/contracts/integration_contract_pack.md`
- `docs/modules/*.md`

---

## 1. Rule Umum

- semua halaman baca wajib scope `periode_id` kecuali halaman yang memang global
- jika data perlu paging, sorting, atau search untuk list besar, lakukan di backend
- frontend hanya melakukan format tampilan, state loading/empty/error, dan navigasi
- jika read model belum tersedia, agent wajib menambahkan kontraknya ke `docs/contracts/query_contracts.md`
  sebelum implementasi

---

## 2. Matrix Admin Dashboard dan Laporan

| Halaman / Screen | Komponen Utama | Route Internal | View / RPC Utama | Dependency Modul | State UI Kritis | Catatan |
|---|---|---|---|---|---|---|
| `admin/dashboard` | `AdminDashboardScreen`, `DashboardKpiGrid`, `OperationalWatchlist` | `GET /api/admin/dashboard` | `get_dashboard_admin`, `v_pesanan_perlu_perhatian`, `v_ringkasan_reseller`, `v_stok_barang`, `v_stok_paket_jadi`, `v_saldo_kas`, `v_pembagian_reseller` | periode, reseller, program_order, setoran, gudang, keuangan | loading, empty_no_periode, empty_no_data, stale, error | semua panel kas dashboard tetap scope `periode_id` |
| `admin/laporan/rekap-reseller` | `RekapResellerReportScreen`, `ReportTableToolbar` | `GET /api/admin/laporan/rekap-reseller` | `v_ringkasan_reseller` atau `get_laporan_rekap_reseller` | reseller, setoran, keuangan | loading, empty, error, paginated | wajib pagination server-side |
| `admin/laporan/stok` | `LaporanStokScreen`, `StockRiskPanel` | `GET /api/admin/laporan/stok` | `v_stok_barang`, `v_stok_paket_jadi` atau `get_laporan_stok` | gudang, program_order | loading, empty, error | tampilkan tab barang vs paket jadi |
| `admin/laporan/pengiriman` | `LaporanPengirimanScreen` | `GET /api/admin/laporan/pengiriman` | `v_detail_pesanan_aktif`, `v_pembagian_reseller` atau `get_laporan_pengiriman` | program_order, gudang | loading, empty, error | filter status kirim dan pembagian |
| `admin/laporan/audit` | `AuditKoreksiScreen` | `GET /api/admin/laporan/audit` | `get_laporan_audit_koreksi` -> `v_riwayat_koreksi`, `v_audit_log` | semua modul transaksi | loading, empty, error | admin-only, satu route dengan tab `Audit Log` dan `Riwayat Koreksi` |

---

## 3. Matrix Reseller Dashboard

| Halaman / Screen | Komponen Utama | Route Internal | View / RPC Utama | Dependency Modul | State UI Kritis | Catatan |
|---|---|---|---|---|---|---|
| `reseller` | `ResellerDashboardScreen`, `ResellerSummaryCards`, `SetorPusatQuickPanel` | `GET /api/reseller/dashboard` | `get_dashboard_reseller` | reseller, setoran, keuangan | pending, no_periode, loading, offline, error | mobile-first, tidak bergantung pada unified history feed |
| `reseller` priority list | `PriorityKonsumenList` | disatukan atau route read khusus | `v_ringkasan_konsumen` atau `v_ringkasan_pesanan_konsumen` | reseller, program_order, setoran | loading, empty, stale | sort `sisa_bayar DESC` |

---

## 4. Matrix Auth dan Access Bootstrap

| Halaman / Screen | Komponen Utama | Route Internal | View / RPC Utama | Dependency Modul | State UI Kritis | Catatan |
|---|---|---|---|---|---|---|
| `login` | `LoginForm`, `AuthShell`, `SessionExpiredState` | `POST /api/auth/login`, `GET /api/auth/session` | auth session + lookup `profile` | auth | loading, invalid_credentials, session_expired, error | jangan campurkan approval logic ke komponen form |
| `pending-approval` | `PendingApprovalState`, `LogoutButton` | `GET /api/auth/session`, `POST /api/auth/logout` | auth session + `profile.status_reseller` | auth, reseller | loading, pending, error | dipakai untuk reseller yang belum aktif |
| protected route bootstrap | `AuthGate`, `RoleGate` | `GET /api/auth/session` | auth session summary | auth | loading, unauthorized, forbidden | redirect ditentukan server/middleware, bukan page |

---

## 5. Matrix Modul Relasi dan Master

| Halaman / Screen | Komponen Utama | Route Internal | View / RPC Utama | Dependency Modul | State UI Kritis | Catatan |
|---|---|---|---|---|---|---|
| `admin/reseller` | `ResellerListScreen`, `ResellerTable` | `GET /api/admin/reseller` | read reseller list, `v_ringkasan_reseller` ringkas bila perlu | reseller | loading, empty, error | detail operasional jangan dihitung di tabel |
| `admin/reseller/[no_reseller]` | `ResellerDetailScreen`, `ResellerSummaryCards` | `GET /api/admin/reseller` detail route | `v_ringkasan_reseller`, `v_ringkasan_konsumen` | reseller, setoran | loading, empty, error | detail reseller admin |
| `reseller/konsumen` | `ResellerKonsumenScreen`, `KonsumenSearchList` | `GET /api/reseller/konsumen` | `v_ringkasan_konsumen` | reseller, setoran | loading, empty, offline, error | search by nama/telepon |
| `reseller/konsumen/[id]` | `KonsumenDetailScreen` | `GET /api/reseller/konsumen` detail route | `v_ringkasan_konsumen`, `v_ringkasan_pesanan_konsumen`, `v_detail_pesanan_aktif`, riwayat setoran query tambahan per modul bila ada | reseller, program_order, setoran | loading, empty, error | jika riwayat belum ada, kontrak harus ditambah dulu |
| `admin/master/akun-kas` | `AkunKasScreen`, `AkunKasFormDialog` | `GET/POST /api/admin/master/akun-kas` | read akun kas + `v_saldo_kas` bila butuh saldo | master_periodik, keuangan | loading, empty, error, success_refetch | lintas periode |
| `admin/master/barang` | `BarangScreen`, `BarangFormDialog` | `GET/POST /api/admin/master/barang` | read barang master | master_periodik | loading, empty, error, success_refetch | lintas periode |
| `admin/master/barang-periode` | `BarangPeriodeScreen`, `BarangPeriodeSelector` | `GET/POST /api/admin/master/barang-periode` | read barang periode scoped `periode_id` | master_periodik, periode | loading, empty, no_periode, error | periodik |
| `admin/master/komisi` | `KomisiScreen`, `KomisiForm` | `GET/POST /api/admin/master/komisi` | read komisi periodik scoped `periode_id` | master_periodik, periode | loading, empty, no_periode, error | periodik |
| `admin/master/paket` | `PaketScreen`, `PaketBomEditor` | `GET/POST /api/admin/master/paket` | read paket + detail BOM scoped `periode_id` | master_periodik, periode | loading, empty, error, success_refetch | wajib jaga aturan BOM |
| `admin/reseller-periode` | `ResellerPeriodeScreen`, `ResellerPeriodeAssignment` | `GET/POST /api/admin/reseller-periode` | read assignment reseller periode | master_periodik, reseller, periode | loading, empty, error | digunakan untuk readiness periode |

---

## 6. Matrix Modul Pesanan dan Setoran

| Halaman / Screen | Komponen Utama | Route Internal | View / RPC Utama | Dependency Modul | State UI Kritis | Catatan |
|---|---|---|---|---|---|---|
| `admin/pesanan` | `PesananListScreen`, `PesananWatchlistScreen` | `GET /api/admin/pesanan` | `v_ringkasan_pesanan_konsumen`, `v_pesanan_perlu_perhatian` | program_order | loading, empty, error | watchlist admin |
| `reseller/pesanan` | `ResellerPesananScreen`, `PesananCard` | `GET /api/reseller/pesanan` | `v_ringkasan_pesanan_konsumen` | program_order | loading, empty, offline, error | list pesanan pemilik |
| `admin/pesanan/[id]` | `PesananDetailScreen`, `DetailPesananTable` | `GET /api/admin/pesanan` detail route | `v_ringkasan_pesanan_konsumen`, `v_detail_pesanan_aktif` | program_order, gudang | loading, empty, error | detail header + item pesanan |
| `reseller/pesanan/[id]` | `PesananDetailScreen`, `DetailPesananList`, `FinalisasiPesananFlow` | `GET /api/reseller/pesanan` detail route | `v_ringkasan_pesanan_konsumen`, `v_detail_pesanan_aktif` scoped reseller | program_order, gudang | loading, empty, offline, error | tidak hitung status lokal |
| `reseller/setor` | `ResellerSetorScreen`, `KonsumenSearchList`, `SetoranFormMock` | `GET/POST /api/reseller/setor` | `v_ringkasan_konsumen`, `buat_setoran_konsumen` | setoran | loading, empty, offline, error, success_refetch | alur 3 langkah |
| `reseller/setor/pusat` | `SetoranPusatScreen`, `SetoranPusatForm` | `GET/POST /api/reseller/setor` | `get_dashboard_reseller`, `v_ringkasan_reseller`, `buat_setoran_pusat` | setoran, keuangan | loading, error, success_refetch | nominal dibatasi UI, final tetap RPC |
| `admin/setoran-konsumen` | `SetoranKonsumenAdminScreen` | `GET /api/admin/setoran` | `v_monitoring_setoran_konsumen_admin` | setoran | loading, empty, error | monitor layer 1 |
| `admin/setoran-pusat` | `SetoranPusatAdminScreen` | `GET /api/admin/setoran` | `v_monitoring_setoran_pusat_admin` | setoran, keuangan | loading, empty, error | monitor layer 2 |

---

## 7. Matrix Modul Gudang dan Keuangan

| Halaman / Screen | Komponen Utama | Route Internal | View / RPC Utama | Dependency Modul | State UI Kritis | Catatan |
|---|---|---|---|---|---|---|
| `admin/gudang/belanja` | `BelanjaListScreen`, `BelanjaForm` | `GET/POST /api/admin/gudang/belanja` | `buat_belanja`, summary saldo dari `v_saldo_kas` | gudang, keuangan | loading, error, success_refetch | form item dinamis |
| `admin/gudang/packing` | `PackingListScreen`, `PackingForm` | `GET/POST /api/admin/gudang/packing` | `v_stok_paket_jadi`, `buat_packing` | gudang | loading, empty, error | paket komposit only |
| `admin/gudang/pengiriman` | `PengirimanOrderScreen`, `OrderKirimTable` | `GET/POST /api/admin/gudang/pengiriman` | `v_detail_pesanan_aktif`, `v_stok_paket_jadi`, `v_stok_barang`, `proses_kirim_detail_pesanan` | gudang, program_order | loading, error, success_refetch | komposit vs tunggal |
| `admin/gudang/pembagian` | `PembagianListScreen`, `PembagianForm` | `GET/POST /api/admin/gudang/pembagian` | `v_pembagian_reseller`, `v_detail_pesanan_aktif`, `buat_pembagian_paket`, `serahkan_pembagian_paket` | gudang | loading, empty, error | eligibility item harus backend |
| `admin/laporan/stok/barang` | `StokBarangTable` | `GET /api/admin/laporan/stok/barang` | `v_stok_barang` | gudang | loading, empty, error | report operasional |
| `admin/laporan/stok/paket-jadi` | `StokPaketJadiTable` | `GET /api/admin/laporan/stok/paket-jadi` | `v_stok_paket_jadi` | gudang | loading, empty, error | backlog packing |
| `admin/keuangan/akun-kas` | `AkunKasListScreen`, `SaldoKasTable` | `GET /api/admin/keuangan/akun-kas`, `GET /api/admin/keuangan/saldo-kas` | `v_saldo_kas` | keuangan, setoran, gudang | loading, empty, error | saldo lintas transaksi |
| `admin/keuangan/kas-masuk` | `KasMasukListScreen`, `KasMasukForm` | `GET/POST /api/admin/keuangan/kas-masuk` | `buat_kas_masuk`, `v_saldo_kas` | keuangan | loading, error, success_refetch | sumber kas eksternal |
| `admin/keuangan/mutasi-kas` | `MutasiKasListScreen`, `MutasiKasForm` | `GET/POST /api/admin/keuangan/mutasi-kas` | `buat_mutasi_kas`, `v_saldo_kas` | keuangan | loading, error, success_refetch | akun asal/tujuan berbeda |
| `admin/keuangan/pencairan` | `PencairanListScreen`, `PencairanForm`, `PencairanEligibilityPanel` | `GET/POST /api/admin/keuangan/pencairan` | `buat_pencairan`, `v_ringkasan_reseller`, `v_saldo_kas` | keuangan, setoran | loading, empty, error, disabled_not_lunas | reseller harus `LUNAS` |

---

## 8. Matrix State Global

| State | Muncul di | Sumber Penentu | Respons UI |
|---|---|---|---|
| `loading` | hampir semua halaman | fetch belum selesai | skeleton, bukan angka 0 |
| `empty_no_periode` | dashboard admin, reseller beranda, laporan periode aktif | periode aktif tidak ada | CTA buat periode atau refresh |
| `empty_no_data` | dashboard/laporan | periode ada tetapi data belum tersedia | tampilkan CTA kerja berikutnya |
| `pending` | reseller area | status reseller `PENDING` | blok transaksi, tampilkan halaman approval |
| `offline` | area reseller | status jaringan | disable submit transaksi uang |
| `error` | semua halaman | route/RPC gagal | tampilkan pesan Bahasa Indonesia |
| `stale` | dashboard/laporan | data lama tetapi masih valid | tampilkan label terakhir diperbarui |

---

## 9. Matrix Hook Naming Recommendation

| Kebutuhan | Hook Disarankan |
|---|---|
| dashboard admin | `useAdminDashboard` / `useAdminDashboardMock` |
| dashboard reseller | `useResellerDashboard` / `useResellerDashboardMock` |
| laporan rekap reseller | `useRekapResellerReport` |
| laporan stok | `useStokReport` |
| laporan pengiriman | `usePengirimanReport` |
| laporan audit | `useAuditKoreksiReport` |
| konsumen prioritas | `usePriorityKonsumen` |
| setoran konsumen | `useSetoranKonsumen` |
| setor pusat | `useSetoranPusat` |
| saldo kas | `useSaldoKas` |
| auth session | `useAuthSession` / `useAuthSessionMock` |
| akun kas master | `useAkunKasMaster` / `useAkunKasMasterMock` |
| barang master | `useBarangMaster` / `useBarangMasterMock` |
| barang periode | `useBarangPeriode` / `useBarangPeriodeMock` |
| komisi periodik | `useKomisi` / `useKomisiMock` |
| paket master | `usePaketMaster` / `usePaketMasterMock` |
| reseller periode | `useResellerPeriode` / `useResellerPeriodeMock` |
| pesanan konsumen | `usePesananList` / `usePesananListMock` |
| detail item pesanan | `useDetailPesananList` / `useDetailPesananListMock` |

Catatan:

- jika masih mockup, suffix `Mock`
- saat masuk integrasi nyata, pertahankan shape return sedekat mungkin

---

## 10. Gaps yang Perlu Dijaga

- `recent activity` belum punya kontrak final; jangan diimplementasikan seolah sudah resmi
- riwayat reseller fase awal tetap modular; jangan asumsikan ada satu read model histori gabungan
- beberapa halaman admin monitor setoran mungkin masih memakai placeholder query; jangan biarkan ini
  menjadi sumber kebenaran permanen
- read model auth session bukan pengganti authorization rule; role guard final tetap tunduk ke
  middleware dan `profile`

---

## 11. Rule untuk AI Coding Agent

- sebelum membuat hook baru, cari dulu apakah halaman tersebut sudah punya read model di matriks
  ini
- jangan membuat satu hook yang mencampur terlalu banyak route lintas domain tanpa alasan kuat
- kalau satu screen butuh banyak panel, lebih baik service/route yang merakit payload daripada
  frontend menghitung sendiri
- jika halaman yang dibangun tidak punya baris di matriks ini, tambahkan dulu dokumen ini atau
  `query_contracts.md` sesuai kebutuhan
