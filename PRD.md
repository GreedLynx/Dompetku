# PRD — Aplikasi Pencatatan Keuangan Pribadi

> Nama kerja: **Dompetku** (bisa diganti)
> Status: Draft v1 · Pengguna: 1 orang (pemilik) · Penggunaan: pribadi

---

## 1. Latar Belakang & Masalah

Pencatatan keuangan sering gagal karena tiga hal:

1. **Data tercecer.** Catatan ada di HP, di laptop, atau hanya di kepala, sehingga tidak pernah utuh.
2. **Tidak ada gambaran cepat.** Untuk tahu "bulan ini sudah habis berapa dan sisa berapa", harus membuka aplikasi dan menghitung.
3. **Baru sadar setelah terlambat.** Tidak ada peringatan saat pengeluaran mendekati batas, jadi pemborosan baru terasa di akhir bulan.

### Masalah yang diselesaikan

| # | Masalah | Solusi produk |
|---|---------|---------------|
| 1 | Data tidak konsisten antar perangkat | Sinkronisasi otomatis laptop ↔ HP |
| 2 | Tidak tahu sisa uang bulan ini | Saldo bulanan yang berkurang tiap pengeluaran + widget |
| 3 | Boros tanpa sadar | Notifikasi peringatan saat mencapai batas yang ditentukan |
| 4 | Aplikasi keuangan lain terlalu rumit / penuh iklan / data dipegang pihak lain | Aplikasi sederhana, input manual, data milik sendiri |

## 2. Tujuan & Non-Tujuan

### Tujuan
- Mencatat pemasukan dan pengeluaran secara manual dengan cepat (≤ 10 detik per transaksi).
- Menampilkan **sisa saldo bulan ini** dan **total pengeluaran bulan ini** secara real-time.
- Memberi peringatan boros berdasarkan batas yang ditetapkan sendiri.
- Data identik di semua perangkat, dan tetap bisa dipakai saat offline.

### Non-Tujuan (di luar cakupan v1)
- Multi-user, berbagi data, atau fitur keluarga.
- Integrasi rekening bank / e-wallet / scan struk otomatis.
- Investasi, utang-piutang, atau pencatatan multi-mata-uang.
- Monetisasi, iklan, atau publikasi ke Play Store / App Store.

## 3. Pengguna

**Satu persona:** pemilik aplikasi. Mencatat sendiri, melihat sendiri, dan menentukan sendiri aturan batasnya. Tidak ada peran lain.

## 4. Proses Bisnis

### 4.1 Konsep inti

- **Saldo awal bulanan** = nominal tetap otomatis (misal uang bulanan), dapat diubah kapan saja.
- **Sisa saldo bawaan** dari bulan lalu **dibawa** ke bulan berikutnya (carry over).
- **Pemasukan tambahan** boleh dicatat kapan saja dan menambah saldo.
- **Pengeluaran** mengurangi saldo.
- **Batas boros** ditetapkan manual dalam nominal pengeluaran per bulan.

### 4.2 Rumus

```
Saldo awal bulan M  = Sisa saldo akhir bulan (M-1) + Jatah bulanan M
Sisa saldo saat ini = Saldo awal bulan M + Pemasukan tambahan M − Pengeluaran M
Persentase batas    = Pengeluaran M ÷ Batas boros M × 100%
```

Bulan pertama memakai saldo awal 0 + jatah bulanan. Sisa saldo bisa negatif (defisit) dan ikut terbawa ke bulan berikutnya.

### 4.3 Alur proses

**A. Pergantian bulan (otomatis)**
1. Saat aplikasi dibuka pada bulan baru, sistem membuat catatan bulan tersebut.
2. Jatah bulanan dan batas boros disalin dari pengaturan terkini (menjadi *snapshot*, sehingga perubahan pengaturan tidak mengubah bulan lalu).
3. Sisa saldo bulan lalu otomatis menjadi bagian saldo awal bulan baru.

**B. Mencatat transaksi**
1. Pengguna menekan tombol tambah.
2. Mengisi: jenis (pengeluaran/pemasukan), nominal, kategori, tanggal (default hari ini), catatan opsional.
3. Simpan → tersimpan lokal seketika → disinkronkan ke cloud di latar belakang.
4. Sistem menghitung ulang sisa saldo dan persentase batas.
5. Jika ambang terlewati → notifikasi peringatan muncul.
6. Widget diperbarui.

**C. Peringatan boros**
- Ambang default: **80%** (peringatan) dan **100%** (batas tercapai); keduanya bisa diatur manual.
- Setiap ambang hanya memicu notifikasi **sekali per bulan** (tidak berulang tiap transaksi).
- Jika batas dinaikkan lalu pengeluaran kembali di bawah ambang, status ambang di-reset.

**D. Mengubah / menghapus transaksi**
- Perubahan langsung menghitung ulang saldo semua bulan setelahnya.
- Penghapusan bersifat *soft delete* agar sinkronisasi aman.

**E. Sinkronisasi**
- Offline-first: semua operasi berjalan di database lokal.
- Saat online, perubahan dikirim dan diterima otomatis. Konflik diselesaikan dengan aturan *last write wins* berdasarkan waktu perubahan.

## 5. Kebutuhan Fungsional

| ID | Kebutuhan | Prioritas |
|----|-----------|-----------|
| F1 | Login satu akun (email + kata sandi) | Must |
| F2 | CRUD transaksi (pengeluaran & pemasukan) | Must |
| F3 | Kategori transaksi (bawaan + dapat ditambah) | Must |
| F4 | Dashboard: sisa saldo, total pengeluaran bulan ini, progres batas | Must |
| F5 | Pengaturan: jatah bulanan, batas boros, ambang peringatan | Must |
| F6 | Saldo bawaan antar bulan (carry over) | Must |
| F7 | Sinkronisasi antar perangkat + mode offline | Must |
| F8 | Notifikasi peringatan boros | Must |
| F9 | Widget homescreen (sisa saldo, pengeluaran, progres) | Should |
| F10 | Riwayat per bulan + filter kategori | Should |
| F11 | Grafik pengeluaran per kategori | Could |
| F12 | Ekspor CSV | Could |
| F13 | Kunci aplikasi (biometrik/PIN) | Could |

## 6. Kebutuhan Non-Fungsional

- **Platform:** Android (wajib), Windows/laptop (wajib), iOS (opsional).
- **Performa:** pembukaan aplikasi < 2 detik; simpan transaksi terasa instan.
- **Offline:** seluruh fitur inti berjalan tanpa internet.
- **Keamanan:** data hanya bisa diakses akun pemilik; transport terenkripsi (HTTPS).
- **Biaya:** target Rp0 per bulan (paket gratis layanan cloud).
- **Akurasi:** nominal disimpan sebagai bilangan bulat rupiah, tanpa pecahan desimal.

## 7. Kriteria Keberhasilan

- Data di HP dan laptop identik dalam hitungan detik setelah online.
- Sisa saldo selalu cocok dengan perhitungan manual.
- Notifikasi muncul tepat saat ambang terlewati, dan tidak muncul berulang.
- Widget menampilkan angka yang sama dengan aplikasi.

## 8. Rencana Rilis

| Fase | Isi |
|------|-----|
| **MVP** | F1–F8: input, saldo, batas, sinkronisasi, notifikasi |
| **v1.1** | F9 widget Android, F10 riwayat |
| **v1.2** | F11–F13 sesuai kebutuhan |

## 9. Asumsi & Risiko

| Risiko | Mitigasi |
|--------|----------|
| Widget butuh kode native | Dikerjakan terakhir, setelah inti stabil |
| Layanan gratis berubah kebijakan | Data bisa diekspor CSV; skema database standar (PostgreSQL) |
| Konflik sinkronisasi | Soft delete + `updated_at` + last write wins |
| Notifikasi iOS terbatas | Fokus Android dan desktop dulu |