# Edge Cases
## Paket Lebaran Mumpuni

Dokumen ini berisi keputusan owner untuk kasus bisnis abu-abu.

Karakter bisnis utama: sistem ini adalah **tabungan target paket**, bukan order e-commerce yang kaku. Konsumen/reseller menabung menuju pilihan paket. Harga paket, BOM, dan komisi bersifat live dalam periode. Jika ada perubahan nilai atau kondisi tidak tuntas, penyesuaian dilakukan manual melalui status TERHENTI, pencairan, atau rekonsiliasi admin.

Referensi umum model pasar: program tabungan/paket Lebaran biasanya berupa setoran rutin/berjangka untuk mendapatkan paket kebutuhan Lebaran di akhir periode. Lihat contoh konsep publik seperti [Tabungan Arisan Paket Lebaran Bank CIJ](https://bankcij.co.id/produk/tabungan/tabungan-arisan-paket-lebaran/) dan [Program Tabungan Paket Lebaran Bank Subang](https://bprsubang.com/program-tabungan-paket-lebaran/).

---

## Prinsip Umum

1. Tidak ada snapshot harga di `detail_pesanan_konsumen`.
2. Konsumen mulai menabung melalui `pesanan_konsumen`; `detail_pesanan_konsumen` menjadi sumber target sejak awal.
3. Harga paket dianggap tetap sejak awal periode; revisi saat AKTIF adalah aksi darurat admin.
4. Perubahan harga paket/BOM/komisi boleh mengubah nilai akhir paket secara live jika direvisi secara sah.
5. Tidak ada konsep uang lebih permanen; jika ada selisih, admin menyesuaikan manual lewat koreksi.
6. Order yang sudah punya pembayaran tidak dibatalkan, melainkan dialihkan ke TERHENTI.
7. Order yang sudah dikirim/dibagikan tidak dibatalkan.
8. Semua transaksi tetap harus berada dalam scope `periode_id`.
9. Koreksi transaksi uang/stok wajib lewat RPC koreksi dan audit log.

---

## EC-001: Order sudah dibayar sebagian lalu dibatalkan

### Keputusan

Item pesanan konsumen yang sudah terkait setoran tidak dibatalkan bebas. Item tersebut diubah
menjadi `status_item = 'TERHENTI'`.

`uang_terhenti` diisi sesuai nilai setoran yang masuk untuk penyesuaian order tersebut.

### Dampak Database

- `status_kirim` tetap bukan `BATAL`.
- `status_item` menjadi `TERHENTI`.
- `uang_terhenti` wajib > 0.
- Nilai order memakai `uang_terhenti`, bukan `paket.nilai_paket`.
- Komisi memakai `persen_terhenti × uang_terhenti`.
- Poin = 0.
- Tidak menyumbang `nilai_tabungan`.

### Dampak UI

Jika admin mencoba membatalkan order yang sudah punya setoran, UI harus mengarahkan ke aksi "Tandai Terhenti", bukan "Batal".

### Pesan User

`Order yang sudah memiliki setoran harus ditandai terhenti`

### Test Case Wajib

- Order konsumen dengan setoran tidak bisa diubah ke `BATAL`.
- Order konsumen dengan setoran bisa diubah ke `TERHENTI` dengan `uang_terhenti`.
- Setelah TERHENTI, target konsumen berubah memakai `uang_terhenti`.

### Keputusan Tambahan Untuk Konsumen Multi-Order

Jika konsumen punya banyak order dan total setoran belum cukup melunasi semua order, admin memilih dulu paket-paket yang masih masuk dalam nilai setoran konsumen. Sisa uang yang tidak cukup untuk melunasi paket lain diarahkan ke satu paket `TERHENTI`.

Prinsip operasional:

- Admin menentukan item pesanan mana yang tetap `AKTIF`.
- Sisa setoran yang tidak cukup untuk paket penuh dipakai sebagai `uang_terhenti`.
- Target konsumen setelah rekonsiliasi tidak boleh lebih kecil dari total bayar konsumen.
- Order yang belum punya alokasi setoran dan belum dikirim boleh `BATAL` jika memang tidak dilanjutkan.

---

## EC-002: Order sudah dikirim lalu dibatalkan

### Keputusan

Order yang sudah dikirim tidak dibatalkan.

### Dampak Database

- `status_kirim = 'SUDAH'` tidak boleh diubah ke `BATAL`.
- Koreksi setelah kirim harus dilakukan lewat RPC koreksi admin dan audit log, bukan batal biasa.

### Dampak UI

Tombol batal tidak boleh tersedia untuk order `SUDAH`.

### Pesan User

`Order yang sudah dikirim tidak bisa dibatalkan`

### Test Case Wajib

- Gagal mengubah order `SUDAH` menjadi `BATAL`.
- Order `SUDAH` tetap dihitung dalam nilai, komisi, poin/tabungan sesuai status item yang masih dihitung.

---

## EC-003: Item pesanan AKTIF diubah TERHENTI ketika total bayar konsumen sudah melebihi uang_terhenti

### Keputusan

`uang_terhenti` diisi manual dan harus disesuaikan dengan total bayar konsumen agar tidak menciptakan uang lebih.

### Dampak Database

- Database wajib menolak `uang_terhenti` yang membuat total bayar konsumen lebih besar dari target konsumen baru.
- Admin harus mengisi `uang_terhenti` minimal sebesar nilai yang menjaga total bayar tidak melebihi target setelah perubahan.

### Dampak UI

Form TERHENTI harus menampilkan total bayar konsumen saat ini sebagai referensi admin.

### Pesan User

`Nominal uang terhenti harus menyesuaikan total setoran yang sudah masuk`

### Test Case Wajib

- Gagal TERHENTI jika `uang_terhenti` membuat target baru lebih kecil dari total bayar.
- Berhasil TERHENTI jika `uang_terhenti` sudah menyesuaikan total bayar.

---

## EC-004: Harga paket/BOM/komisi berubah setelah periode AKTIF

### Keputusan

Semua perubahan bersifat live. Tidak ada snapshot harga, BOM, atau komisi di transaksi.

### Dampak Database

- `detail_pesanan_konsumen` tetap tidak menyimpan harga live.
- `nilai_paket`, modal, komisi, tabungan, target, dan nilai akhir reseller mengikuti master terbaru.
- Perubahan master saat periode AKTIF boleh dilakukan oleh admin.

### Dampak UI

Admin harus diberi konfirmasi bahwa perubahan master akan memengaruhi tagihan, komisi, stok, dan laporan periode berjalan.

### Pesan User

`Perubahan ini akan memperbarui nilai hitungan periode berjalan`

### Test Case Wajib

- Perubahan `paket.nilai_paket` mengubah target konsumen.
- Perubahan `detail_paket` mengubah kebutuhan/stok.
- Perubahan `komisi_config` mengubah komisi reseller.

---

## EC-005: Pencairan sebelum reseller LUNAS

### Keputusan

Tidak bisa. Pencairan hanya boleh setelah reseller LUNAS.

### Dampak Database

RPC `buat_pencairan` wajib memvalidasi `status_lunas_reseller = 'LUNAS'`.

### Dampak UI

Tombol pencairan disabled untuk reseller yang belum LUNAS.

### Pesan User

`Pencairan hanya bisa dilakukan setelah reseller lunas`

### Test Case Wajib

- Gagal pencairan saat reseller belum lunas.
- Berhasil pencairan saat reseller sudah lunas dan sisa dana cukup.

---

## EC-006: Saldo kas tidak cukup untuk belanja/pencairan/mutasi

### Keputusan

Ada kas masuk dari luar setoran. Sistem perlu mencatat pemasukan kas eksternal agar saldo kas bisa bertambah secara resmi.

### Dampak Database

Tambahkan transaksi `kas_masuk` untuk pemasukan kas non-setoran reseller.

Contoh sumber:
- modal owner
- penyesuaian kas
- pemasukan lain

Belanja, pencairan, dan mutasi tetap harus memvalidasi saldo kas sumber setelah memperhitungkan `kas_masuk`.

### Dampak UI

Admin perlu halaman/form "Kas Masuk" atau "Pemasukan Kas".

### Pesan User

`Saldo kas tidak cukup. Tambahkan kas masuk terlebih dahulu`

### Test Case Wajib

- Saldo kas bertambah setelah input `kas_masuk`.
- Belanja/pencairan/mutasi gagal jika saldo tetap kurang.
- Belanja/pencairan/mutasi berhasil setelah kas masuk mencukupi.

---

## EC-007: Reseller setor pusat sebelum semua konsumen lunas

### Keputusan

Boleh. Reseller setor ke pusat bertahap sesuai saldo reseller yang sudah terkumpul dari konsumen.

### Dampak Database

Validasi layer 2 tetap:

```txt
total_disetor + nominal <= nilai_akhir_paket
total_disetor + nominal <= total_dikumpulkan
```

Tidak perlu menunggu semua konsumen lunas.

### Dampak UI

Halaman setor pusat menampilkan:
- total dikumpulkan
- total disetor
- saldo belum disetor
- sisa kewajiban ke pusat

### Pesan User

`Setoran Rp {nominal} ke {nama_kas} berhasil dicatat`

### Test Case Wajib

- Berhasil setor pusat saat masih ada konsumen belum lunas, selama saldo terkumpul cukup.
- Gagal setor pusat melebihi saldo terkumpul.

---

## EC-008: Periode ditutup saat masih ada reseller belum lunas

### Keputusan

Semua transaksi harus diselesaikan sebelum periode ditutup.

### Dampak Database

RPC `tutup_periode` wajib menolak penutupan jika masih ada reseller belum LUNAS atau transaksi/pencairan belum selesai.

### Dampak UI

Halaman tutup periode harus menampilkan daftar blocker:
- reseller belum lunas
- pencairan belum selesai
- transaksi operasional yang belum selesai

### Pesan User

`Masih ada {jumlah} reseller yang belum lunas`

### Test Case Wajib

- Gagal tutup periode jika ada reseller belum LUNAS.
- Berhasil tutup periode jika semua reseller LUNAS dan pencairan selesai.

---

## EC-009: Konsumen menabung sebelum memilih paket final

### Keputusan

Boleh. Konsumen dibuatkan `pesanan_konsumen` setelah memilih paket target awal. Setoran masuk ke
header pesanan. `detail_pesanan_konsumen` sudah dibuat sejak awal sebagai sumber target.

### Dampak Database

- `pesanan_konsumen.status_pesanan = 'AKTIF'`.
- `setoran_konsumen.pesanan_konsumen_id` wajib terisi.
- `target_berjalan = snapshot total pilihan paket aktif program`.

### Test Case Wajib

- Berhasil input setoran program tanpa order.
- Gagal setoran jika pesanan konsumen belum ada.
- Gagal setoran jika nominal melebihi target tagihan pesanan.

---

## EC-010: Konsumen mengganti paket sebelum dikunci

### Keputusan

Boleh. Sebelum program `DIKUNCI`, pilihan paket belum menjadi order final, tetapi pilihan paket
aktif program harus tersimpan karena menjadi acuan target tabungan.

### Dampak UI

UI menampilkan status `Belum Dikunci` dan tidak menganggap pilihan paket aktif program sebagai
stok/order final.

---

## EC-011: Total paket final lebih kecil dari setoran yang sudah masuk

### Keputusan

Tidak boleh dikunci. Admin/reseller harus memilih paket dengan total nilai minimal sebesar total setoran,
atau melakukan koreksi setoran jika memang ada salah input.

### Pesan User

`Total paket yang dipilih lebih kecil dari setoran yang sudah masuk`

### Test Case Wajib

- Gagal finalisasi pesanan jika total target < total setoran.
- Berhasil finalisasi pesanan jika total target >= total setoran.

---

## EC-012: Harga paket naik darurat saat periode aktif

### Keputusan

Boleh hanya sebagai aksi darurat admin. Harga periode secara bisnis dianggap tetap sejak awal periode,
tetapi sistem tetap mendukung revisi karena kondisi ekonomi dapat berubah.

### Dampak Database

- Tidak ada snapshot harga live di `detail_pesanan_konsumen`.
- Revisi wajib masuk `audit_log`.
- Alasan wajib.
- Target dan laporan berubah mengikuti `paket.nilai_paket` terbaru.

### Pesan User

`Perubahan harga ini akan memperbarui nilai tagihan periode berjalan`

---

## EC-013: Konsumen 3 bulan tidak setor atau tertinggal jauh

### Keputusan

Sistem tidak otomatis menghentikan program. Sistem hanya memberi kandidat watchlist.
Admin/reseller menandai manual sebagai `PERLU_PERHATIAN` atau `TERHENTI`.

### Dampak Database

- Status dapat diubah melalui RPC `tandai_program_perlu_perhatian` atau `tandai_program_terhenti`.
- Alasan wajib saat menandai `TERHENTI`.

### Test Case Wajib

- View watchlist menampilkan program lama tanpa setoran.
- Program tidak berubah otomatis tanpa RPC.

---

## EC-014: Salah input setoran/belanja/packing

### Keputusan

Tidak boleh edit langsung transaksi asli. Gunakan RPC koreksi/reversal.

### Dampak Database

- Transaksi asli tetap ada.
- Row koreksi dibuat di `koreksi_transaksi`.
- Aksi masuk `audit_log`.
- Read model memperhitungkan koreksi.

### Pesan User

`Koreksi berhasil disimpan`

---

## EC-015: Pembagian paket ke reseller kurang atau salah item

### Keputusan

Selisih pembagian dicatat pada detail pembagian dan diselesaikan lewat koreksi pembagian.

### Dampak Database

- `pembagian_paket_detail.status_item = 'KURANG'` atau `DIGANTI`.
- Koreksi wajib alasan.
- Audit log wajib.

### Test Case Wajib

- Berhasil menandai item kurang.
- Berhasil koreksi pembagian.
- Order yang sudah selesai dibagikan tidak masuk batch pembagian baru tanpa koreksi.
