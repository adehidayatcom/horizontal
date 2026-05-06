# Frontend Component Contracts
## Paket Lebaran Mumpuni

Dokumen ini menetapkan kontrak komponen frontend untuk proyek `horizontal`.

Fokus dokumen ini:

- menjaga konsistensi komponen di atas shell `Modernize`
- memastikan admin dan reseller memakai pola UI yang selaras
- mencegah logika bisnis final bocor ke komponen presentasional

Dokumen ini melengkapi:

- `docs/frontend_architecture.md`
- `docs/component_patterns.md`
- `docs/navigation_and_period_setup_ui.md`
- `docs/ui/admin_dashboard_uiux.md`
- `docs/ui/reseller_uiux.md`

---

## 1. Prinsip Umum

- Basis komponen visual adalah `Material-UI v7`.
- Shell global mengikuti `Modernize`.
- Komponen bisnis dibangun sebagai komposisi `MUI`, wrapper internal, dan pola proyek.
- Komponen tidak menjadi sumber kebenaran logika bisnis.
- Data yang diterima komponen harus sudah siap tampil atau siap diolah untuk kebutuhan presentasi ringan.
- Semua teks user-facing memakai Bahasa Indonesia.
- Area reseller tidak boleh memaksa horizontal scroll pada viewport mobile normal.
- Area admin boleh memakai overflow horizontal untuk tabel besar jika memang diperlukan.

---

## 2. Fondasi Komponen

Komponen proyek diprioritaskan memakai fondasi berikut:

| Kebutuhan | Fondasi Utama |
|---|---|
| Tombol | `Button`, `IconButton`, wrapper internal jika perlu |
| Input teks | `TextField` |
| Select | `TextField select`, `Select`, `MenuItem` |
| Checkbox/Radio/Switch | komponen MUI standar atau wrapper internal |
| Dialog | `Dialog`, `DialogTitle`, `DialogContent`, `DialogActions` |
| Drawer/Sheet | `Drawer` |
| Table | `Table` MUI atau `@tanstack/react-table` + renderer MUI |
| Tabs | `Tabs` + `Tab` |
| Status | `Chip`, `Alert`, atau wrapper badge internal |
| Loading | `Skeleton`, `CircularProgress`, `LinearProgress` |
| Empty state | komponen internal berbasis `Stack`, `Typography`, `Button` |
| Notifikasi | `Snackbar`, `Alert`, atau wrapper internal proyek |
| Tooltip | `Tooltip` |
| Card | `Card`, `CardHeader`, `CardContent`, `CardActions` atau wrapper proyek |

Aturan:

- gunakan komponen MUI atau wrapper internal terlebih dulu sebelum membuat komponen baru
- wrapper internal dipakai untuk menyeragamkan style, spacing, atau behavior berulang
- jangan membuat markup bebas berulang jika pola sudah bisa dijadikan komponen reusable

---

## 3. Aturan Struktur Komponen

### Presentational Component

Bertanggung jawab untuk:

- render UI
- state visual lokal sederhana
- event callback keluar

Tidak bertanggung jawab untuk:

- fetch data domain langsung jika komponen sudah besar
- formula bisnis final
- hak akses final

### Container atau Feature Component

Bertanggung jawab untuk:

- menghubungkan hook, service, dan presentational component
- menyusun state halaman atau fitur
- mengelola loading, error, empty, dan success state

### Shared Component

Dipakai lintas admin dan reseller bila:

- label dan perilaku dasarnya netral
- tidak diam-diam membawa aturan bisnis khusus satu role

---

## 4. Aturan Visual

- radius kecil sampai sedang; hindari bentuk berlebihan
- border tipis dan konsisten
- shadow ringan, tidak dramatis
- spacing mengikuti skala shell/theme
- warna status mengikuti theme dan semantik bisnis
- jangan menambah ornamen yang tidak membawa fungsi
- jangan membuat card bertumpuk berlebihan tanpa alasan layout yang kuat

Aturan icon:

- gunakan `@tabler/icons-react` atau `@mui/icons-material`
- satu area UI sebaiknya konsisten memakai satu gaya icon dominan

---

## 5. State Wajib

Setiap komponen atau halaman data minimal harus mempertimbangkan:

- loading
- empty
- error
- disabled bila ada pembatasan aksi
- success feedback untuk aksi penting

Aturan:

- loading harus terlihat jelas, bukan hanya tombol diam
- empty state harus informatif, bukan layar kosong
- error state harus manusiawi dan berbahasa Indonesia

---

## 6. Shared Components Minimum

### `StatusBadge`

Tujuan:

- menampilkan status bisnis secara konsisten

Props minimum:

```ts
type StatusBadgeProps = {
  status: string;
  variant?: 'periode' | 'lunas' | 'kirim' | 'program' | 'reseller' | 'order';
}
```

Aturan:

- warna dan label ditentukan dari mapping internal
- komponen ini tidak boleh menghitung status akhir sendiri

### `SummaryCard`

Tujuan:

- menampilkan angka ringkas untuk admin atau reseller

Props minimum:

```ts
type SummaryCardProps = {
  label: string;
  value: string;
  icon?: React.ReactNode;
  tone?: 'primary' | 'success' | 'warning' | 'error' | 'info' | 'neutral';
  footer?: string;
  isLoading?: boolean;
}
```

Aturan:

- angka sudah diformat sebelum tampil atau memakai util format tampilan
- kartu tidak menghitung angka bisnis final

### `EmptyState`

Props minimum:

```ts
type EmptyStateProps = {
  title: string;
  description?: string;
  action?: React.ReactNode;
}
```

### `ErrorState`

Props minimum:

```ts
type ErrorStateProps = {
  title?: string;
  message: string;
  onRetry?: () => void;
}
```

### `LoadingState`

Props minimum:

```ts
type LoadingStateProps = {
  variant: 'page' | 'cards' | 'list' | 'table' | 'form';
  rows?: number;
}
```

### `ConfirmDialog`

Props minimum:

```ts
type ConfirmDialogProps = {
  open: boolean;
  title: string;
  description: string;
  confirmLabel: string;
  cancelLabel?: string;
  tone?: 'default' | 'danger';
  isPending?: boolean;
  onConfirm: () => void;
  onOpenChange: (open: boolean) => void;
}
```

### `RupiahInput`

Props minimum:

```ts
type RupiahInputProps = {
  value: number | null;
  onChange: (value: number | null) => void;
  disabled?: boolean;
  error?: string;
  autoFocus?: boolean;
}
```

Aturan:

- input mobile minimal 16px
- nilai keluar berupa number, bukan string terformat
- validasi final nominal tetap di server boundary atau RPC

---

## 7. Shell Components

### `AdminLayout`

Tanggung jawab:

- header global
- navigation horizontal atau sidebar sesuai mode shell
- breadcrumb
- slot periode aktif atau periode laporan
- outlet route admin

Tidak termasuk:

- fetch data bisnis halaman
- validasi transaksi

### `ResellerLayout`

Tanggung jawab:

- header ringkas reseller
- status periode aktif
- bottom navigation atau pola mobile nav yang disetujui
- outlet route reseller

Tidak termasuk:

- fetch data dashboard detail
- submit transaksi domain

### `PeriodGuard`

Tanggung jawab:

- menampilkan state yang sesuai jika belum ada periode aktif
- menandai halaman transaksi sebagai disabled/read-only saat status periode tidak mengizinkan

### `AuthGuard`

Tanggung jawab:

- menjaga UX saat auth belum siap
- menghindari render halaman protected sebelum state minimum tersedia

Catatan:

- proteksi final tetap berada di middleware atau server boundary

---

## 8. Contract Reseller Components

### `KonsumenSearchList`

Tujuan:

- mencari dan memilih konsumen atau program dengan cepat

Props minimum:

```ts
type KonsumenSearchListProps = {
  periodeId: number;
  defaultFilter?: 'belum_lunas' | 'semua' | 'lunas';
  onSelectKonsumen: (programKonsumenId: number) => void;
}
```

State minimum:

- loading
- empty
- search debounce
- indikator data cache bila dipakai

Aturan:

- search mudah dijangkau
- default sort mengikuti prioritas operasional
- tidak menimbulkan horizontal scroll

### `FormSetoran`

Props minimum:

```ts
type FormSetoranProps = {
  programKonsumenId: number;
  periodeId: number;
  onSuccess: () => void;
}
```

State minimum:

- ringkasan target, total bayar, dan sisa bayar
- submit loading
- error inline untuk nominal tidak valid
- feedback sukses

Larangan:

- submit ganda
- submit saat offline bila flow belum resmi mendukung queue
- menghitung formula final target sendiri

### `SetorPusatForm`

Props minimum:

```ts
type SetorPusatFormProps = {
  resellerId: string;
  periodeId: number;
  onSuccess: () => void;
}
```

Aturan:

- tampilkan konteks saldo terkumpul, total disetor, dan sisa setor
- validasi final tetap di RPC

### `PosCartLayout`

Tujuan:

- menggantikan form konvensional dengan antarmuka *Point of Sale* (POS) untuk pembuatan pesanan konsumen secara cepat dan intuitif

Aturan:

- layar terbelah: kiri untuk grid paket *searchable*, kanan untuk keranjang/invoice yang bersifat *sticky*
- wajib memiliki fitur penyesuaian kuantitas secara instan dengan tombol +/- tanpa *pop-up*
- kalkulasi total harga keranjang berjalan secara *real-time* di frontend sebelum sistem melakukan *submit*

### `LedgerPaymentGrid`

Tujuan:

- memungkinkan *batch input* (bulk) setoran konsumen layaknya aplikasi buku kas atau *spreadsheet* di perangkat *mobile*

Aturan:

- baris adalah daftar konsumen dengan saldo aktif, sel adalah *input* nominal setoran khusus untuk sesi saat ini
- interaksi *keyboard* (tombol `Enter`) harus memindahkan fokus ke baris konsumen berikutnya secara mulus
- wajib menyediakan aksi *Floating Button* (contoh: "Simpan 5 Setoran") untuk efisiensi pemanggilan API

### `FinTechWalletCard`

Tujuan:

- merombak tampilan *input* setor ke pusat menjadi ala dompet digital modern (contoh: OVO/Dana)

Aturan:

- menampilkan angka saldo raksasa dengan peringatan visual (*progress bar* merah) jika melebihi batas simpanan aman
- wajib menggunakan komponen *Quick Chips* untuk input nominal cepat (contoh: "Setor Semua", "Rp 500.000") guna meminimalkan ketikan berulang

---

## 9. Contract Admin Components

### `DataTableAdmin`

Tujuan:

- menampilkan data admin yang padat dan bisa discan cepat

Aturan:

- boleh memakai overflow horizontal
- header kolom harus jelas
- aksi per-row harus eksplisit
- loading, empty, dan error state wajib tersedia

### `PeriodeSetupChecklist`

Tujuan:

- menampilkan kesiapan aktivasi periode secara terstruktur

Aturan:

- setiap item jelas statusnya
- blocker dan warning dibedakan
- tombol aktivasi hanya muncul bila syarat terpenuhi

### `WatchlistCard`

Tujuan:

- menyorot area yang perlu tindakan operasional

Aturan:

- angka, label, dan aksi harus singkat
- tidak penuh narasi panjang

### `SpreadsheetDataGrid`

Tujuan:

- memungkinkan pengeditan cepat (*inline-editing*) untuk ratusan baris data (seperti Katalog Master Paket) tanpa harus berpindah halaman

Props minimum:

```ts
type SpreadsheetDataGridProps<T> = {
  rows: T[];
  columns: GridColDef[];
  loading?: boolean;
  onCellEditCommit?: (params: any) => Promise<void>;
  onMassReplace?: (find: string, replace: string) => Promise<void>;
}
```

Aturan:

- diwajibkan menggunakan komponen kelas enterprise seperti `@mui/x-data-grid`
- harus menggunakan *Virtualization* agar tidak lag saat merender 500+ baris
- perubahan sel (*cell edit*) harus langsung menembak API atau disimpan dalam *state* untuk mekanisme *batch save*
- dilarang me-render *Pop-up Modal* hanya untuk mengganti angka nominal/harga

### `TreeCategorySidebar`

Tujuan:

- mengelompokkan data yang sangat masif menjadi navigasi hierarki (Kategori > Subkategori)

Aturan:

- menggunakan komponen dari `@mui/x-tree-view`
- node yang di-klik di sidebar ini bertugas mengubah *state* parameter pemanggilan data di komponen `SpreadsheetDataGrid` sebelahnya

### `SplitPaneReview`

Tujuan:

- mempercepat proses *approval* data admin dengan navigasi layaknya aplikasi *email client* (list di sebelah kiri, detail di sebelah kanan)

Aturan:

- dilarang menggunakan *full page reload* atau *pop-up modal* murni untuk melihat detail
- aksi *Approve/Reject* di *pane* sebelah kanan wajib memindahkan seleksi secara otomatis ke entitas berikutnya di *pane* sebelah kiri

### `KanbanBoard`

Tujuan:

- manajemen status visual pada operasional gudang dan logistik menggunakan papan kolom yang intuitif

Aturan:

- mendukung aksi *Drag-and-Drop* antar kolom status (contoh: dari `Belum Di-packing` ke `Siap Bagikan`)
- komponen harus mengadopsi *optimistic UI* saat state kartu dipindah, sebelum server membalas dengan status sukses

---

## 10. Aturan Aksesibilitas

- setiap field wajib punya label yang jelas
- dialog wajib punya title
- tombol icon-only wajib punya `aria-label`
- kontras warna status harus tetap terbaca
- state disabled tidak boleh menjadi satu-satunya penanda informasi penting

---

## 11. Aturan Integrasi dengan Theme

- komponen harus kompatibel dengan dark mode
- komponen harus kompatibel dengan mode horizontal maupun sidebar jika berada dalam shell admin
- style tidak boleh hardcode melawan token theme tanpa alasan kuat
- wrapper internal lebih disukai daripada style override acak berulang

---

## 12. Larangan

- jangan memakai komponen demo template sebagai kontrak final produk
- jangan menaruh formula bisnis final di komponen
- jangan membuat pola form atau tabel yang menyimpang tanpa kebutuhan jelas
- jangan membuat feedback sukses/error yang tidak konsisten antar fitur
- jangan membuat UI reseller yang terasa seperti desktop yang dipaksa mengecil
