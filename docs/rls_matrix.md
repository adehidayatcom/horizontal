# RLS Matrix
## Paket Lebaran Mumpuni

Dokumen ini mendefinisikan hak akses database. Implementasi RLS wajib mengikuti matrix ini dan memakai `auth.uid()` melalui tabel `profile`.

---

## 1. Prinsip Auth

### 1.1 Tabel Profile

`profile.id` harus sama dengan `auth.users.id`.

```sql
profile.id = auth.uid()
```

Admin:

```sql
EXISTS (
  SELECT 1 FROM profile
  WHERE profile.id = auth.uid()
    AND profile.role = 'ADMIN'
)
```

Reseller:

```sql
SELECT profile.no_reseller
FROM profile
WHERE profile.id = auth.uid()
  AND profile.role = 'RESELLER'
```

### 1.2 Larangan

- Jangan memakai klaim JWT custom seperti `auth.jwt() ->> 'no_reseller'` sebagai satu-satunya sumber akses.
- Jangan expose service role key ke frontend.
- Jangan membuat policy `USING (true)` pada tabel operasional.
- Jangan memberikan reseller akses ke data reseller lain melalui view.

---

## 2. Matrix Akses Tabel

Legend:
- `ALL`: boleh semua operasi sesuai kebutuhan admin.
- `OWN`: hanya data milik reseller login.
- `READ`: hanya baca.
- `NO`: tidak boleh.
- `RPC`: operasi tulis hanya boleh lewat RPC.

| Tabel | Admin SELECT | Admin INSERT | Admin UPDATE | Admin DELETE | Reseller SELECT | Reseller INSERT | Reseller UPDATE | Reseller DELETE | Catatan |
|---|---|---|---|---|---|---|---|---|---|
| `profile` | ALL | ALL | ALL | NO | OWN | NO | OWN terbatas | NO | Reseller hanya boleh lihat/update profil sendiri yang aman |
| `periode` | ALL | ALL | ALL | NO | READ aktif | NO | NO | NO | Reseller butuh periode aktif untuk UI |
| `reseller` | ALL | ALL | ALL | NO | OWN | NO | OWN terbatas | NO | Status hanya admin yang ubah |
| `reseller_periode` | ALL | ALL | ALL | NO | OWN READ | NO | NO | NO | Flag pelunasan hanya admin |
| `konsumen` | ALL | ALL | ALL | NO | OWN | OWN | OWN | NO | OWN berdasarkan `no_reseller` |
| `pesanan_konsumen` | ALL | RPC | RPC | NO | OWN | RPC | RPC terbatas | NO | Setoran berjalan melalui header pesanan |
| `paket` | ALL | ALL | ALL | NO | READ periode aktif | NO | NO | NO | Harga live; perubahan saat AKTIF memengaruhi tagihan berjalan |
| `detail_paket` | ALL | ALL | ALL | NO | READ periode aktif | NO | NO | NO | BOM live; perubahan saat AKTIF memengaruhi stok/kebutuhan |
| `barang` | ALL | ALL | ALL | NO | READ terbatas | NO | NO | NO | Reseller tidak wajib lihat semua detail biaya |
| `barang_periode` | ALL | ALL | ALL | NO | NO langsung | NO | NO | NO | Stok/harga/budget sensitif; reseller perlu interface aman terpisah bila perlu stok/availability |
| `akun_kas` | ALL | ALL | ALL | NO | NO langsung | NO | NO | NO | Reseller perlu akun tujuan setoran via interface aman terpisah agar `saldo_awal` tidak terekspos |
| `komisi_config` | ALL | ALL | ALL | NO | READ terbatas | NO | NO | NO | Komisi live; perubahan saat AKTIF memengaruhi hak reseller |
| `detail_pesanan_konsumen` | ALL | RPC | RPC | NO | OWN | RPC | NO | NO | Reseller kelola item pesanan lewat RPC |
| `setoran_konsumen` | ALL | RPC | Koreksi admin | NO | OWN | RPC | NO | NO | Reseller input lewat RPC |
| `setoran` | ALL | RPC | Koreksi admin | NO | OWN | RPC | NO | NO | Reseller setor pusat lewat RPC |
| `pencairan` | ALL | RPC | Koreksi admin | NO | OWN READ | NO | NO | NO | Pencairan hanya admin |
| `belanja` | ALL | ALL/RPC | ALL | NO | NO | NO | NO | NO | Operasional admin |
| `belanja_detail` | ALL | ALL/RPC | ALL | NO | NO | NO | NO | NO | Operasional admin |
| `packing` | ALL | RPC | Koreksi admin | NO | READ terbatas | NO | NO | NO | Reseller boleh lihat status siap kirim bila diperlukan |
| `mutasi_kas` | ALL | RPC | Koreksi admin | NO | NO | NO | NO | NO | Keuangan pusat |
| `kas_masuk` | ALL | RPC | Koreksi admin | NO | NO | NO | NO | NO | Pemasukan kas luar setoran |
| `pembagian_paket` | ALL | RPC | RPC/koreksi | NO | OWN READ | NO | NO | NO | Reseller boleh lihat paket yang diserahkan kepadanya |
| `pembagian_paket_detail` | ALL | RPC | RPC/koreksi | NO | OWN READ | NO | NO | NO | Mengikuti pembagian_paket |
| `koreksi_transaksi` | ALL | RPC | NO | NO | OWN READ terbatas | NO | NO | NO | Reseller hanya boleh melihat koreksi yang terkait datanya bila dibutuhkan UI |
| `audit_log` | ALL | RPC/internal | NO | NO | NO langsung | NO | NO | NO | Audit internal admin |

---

## 3. Policy Helper yang Disarankan

Gunakan helper SQL agar policy tidak berulang dan mudah diaudit.

```sql
CREATE OR REPLACE FUNCTION auth_role()
RETURNS TEXT
LANGUAGE SQL
STABLE
SECURITY DEFINER
AS $$
  SELECT role::TEXT
  FROM profile
  WHERE id = auth.uid()
$$;
```

```sql
CREATE OR REPLACE FUNCTION auth_no_reseller()
RETURNS TEXT
LANGUAGE SQL
STABLE
SECURITY DEFINER
AS $$
  SELECT no_reseller
  FROM profile
  WHERE id = auth.uid()
    AND role = 'RESELLER'
$$;
```

```sql
CREATE OR REPLACE FUNCTION is_admin()
RETURNS BOOLEAN
LANGUAGE SQL
STABLE
SECURITY DEFINER
AS $$
  SELECT EXISTS (
    SELECT 1
    FROM profile
    WHERE id = auth.uid()
      AND role = 'ADMIN'
  )
$$;
```

> Catatan: pastikan function helper tidak membuka data sensitif dan search_path dikunci dalam migrasi final.

---

## 4. Contoh Policy

### 4.1 Konsumen

```sql
CREATE POLICY "admin_kelola_semua_konsumen"
ON konsumen
FOR ALL
USING (is_admin())
WITH CHECK (is_admin());
```

```sql
CREATE POLICY "reseller_lihat_konsumen_sendiri"
ON konsumen
FOR SELECT
USING (no_reseller = auth_no_reseller());
```

```sql
CREATE POLICY "reseller_tambah_konsumen_sendiri"
ON konsumen
FOR INSERT
WITH CHECK (no_reseller = auth_no_reseller());
```

```sql
CREATE POLICY "reseller_edit_konsumen_sendiri"
ON konsumen
FOR UPDATE
USING (no_reseller = auth_no_reseller())
WITH CHECK (no_reseller = auth_no_reseller());
```

### 4.2 Detail Pesanan Konsumen

```sql
CREATE POLICY "admin_lihat_semua_detail_pesanan"
ON detail_pesanan_konsumen
FOR SELECT
USING (is_admin());
```

```sql
CREATE POLICY "reseller_lihat_detail_pesanan_sendiri"
ON detail_pesanan_konsumen
FOR SELECT
USING (no_reseller = auth_no_reseller());
```

Tulis detail pesanan harus lewat `buat_pesanan_konsumen`, `ubah_detail_pesanan_konsumen`, atau
`finalisasi_pesanan_konsumen` sesuai kontrak, bukan direct insert dari frontend.

### 4.3 Pesanan Konsumen

```sql
CREATE POLICY "admin_kelola_semua_pesanan"
ON pesanan_konsumen
FOR ALL
USING (is_admin())
WITH CHECK (is_admin());
```

```sql
CREATE POLICY "reseller_lihat_pesanan_sendiri"
ON pesanan_konsumen
FOR SELECT
USING (no_reseller = auth_no_reseller());
```

Tulis pesanan, ubah detail item, tandai perhatian, dan finalisasi pesanan harus lewat RPC agar
validasi target, status, dan audit berjalan konsisten.

---

## 5. View dan RPC

Semua view yang menampilkan data reseller harus salah satu dari:

1. RLS-safe karena membaca tabel dengan RLS aktif.
2. Memakai parameter `periode_id` dan filter `no_reseller = auth_no_reseller()` untuk reseller.
3. Dipisahkan antara view admin dan view reseller.

RPC transaksi harus:
- cek `is_admin()` untuk aksi admin
- cek `auth_no_reseller()` untuk aksi reseller
- tidak menerima `no_reseller` bebas dari client reseller tanpa validasi
- mengembalikan pesan error bisnis dalam Bahasa Indonesia

---

## 6. Test RLS Wajib

| Skenario | Harapan |
|---|---|
| Admin membaca semua konsumen | Berhasil |
| Reseller A membaca konsumen A | Berhasil |
| Reseller A membaca konsumen B | Tidak ada row |
| Reseller A insert konsumen dengan no_reseller A | Berhasil |
| Reseller A insert konsumen dengan no_reseller B | Ditolak |
| Reseller mengakses belanja | Ditolak |
| Reseller mengakses mutasi_kas | Ditolak |
| Reseller mengakses kas_masuk | Ditolak |
| Anonymous mengakses tabel operasional | Ditolak |
| Reseller A membaca pesanan konsumen B | Tidak ada row |
| Reseller membuat setoran tanpa program miliknya | Ditolak |
| Reseller mengakses audit_log langsung | Ditolak |
| Reseller melihat pembagian reseller lain | Tidak ada row |
