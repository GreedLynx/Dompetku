# SYSTEM_DESIGN — Desain Visual

Panduan visual untuk aplikasi **Dompetku**. Semua nilai di sini adalah *design token*; di kode harus dipakai lewat token, bukan angka/warna langsung.

---

## 1. Prinsip Desain

1. **Angka adalah bintangnya.** Sisa saldo selalu jadi elemen paling besar dan paling jelas.
2. **Cepat dicatat.** Tambah transaksi maksimal 3 ketukan dari layar utama.
3. **Status terbaca sekilas.** Warna menandakan kondisi keuangan (aman, waspada, melewati batas).
4. **Tenang dan bersih.** Tanpa dekorasi berlebih; ruang kosong dipakai untuk menenangkan.
5. **Konsisten di semua perangkat.** Komponen sama di HP dan laptop; hanya tata letak yang menyesuaikan.

## 2. Warna

### 2.1 Palet utama

| Token | Light | Dark | Kegunaan |
|-------|-------|------|----------|
| `primary` | `#0F766E` | `#2DD4BF` | Aksi utama, tombol, elemen aktif |
| `onPrimary` | `#FFFFFF` | `#042F2E` | Teks di atas primary |
| `primaryContainer` | `#CCFBF1` | `#134E4A` | Latar kartu/chip aksen |
| `background` | `#F8FAFC` | `#0B1220` | Latar halaman |
| `surface` | `#FFFFFF` | `#131C2E` | Kartu, sheet, dialog |
| `surfaceVariant` | `#F1F5F9` | `#1C2740` | Input, bagian bertingkat |
| `outline` | `#E2E8F0` | `#2A3652` | Garis pemisah, border |
| `textPrimary` | `#0F172A` | `#F1F5F9` | Teks utama |
| `textSecondary` | `#64748B` | `#94A3B8` | Teks pendukung, label |

### 2.2 Warna status (semantik)

| Token | Light | Dark | Dipakai saat |
|-------|-------|------|--------------|
| `safe` | `#16A34A` | `#4ADE80` | Pengeluaran < 80% batas |
| `warning` | `#D97706` | `#FBBF24` | 80% – 99% batas |
| `danger` | `#DC2626` | `#F87171` | ≥ 100% batas / saldo negatif |
| `income` | `#0284C7` | `#38BDF8` | Pemasukan |
| `expense` | `#DC2626` | `#F87171` | Pengeluaran (teks nominal) |

Aturan: warna **tidak boleh jadi satu-satunya penanda**. Selalu sertakan ikon atau teks (mis. ikon ⚠ + "Hampir batas").

### 2.3 Warna kategori (bawaan)

Makan `#F97316` · Transportasi `#3B82F6` · Belanja `#EC4899` · Tagihan `#8B5CF6` · Hiburan `#EAB308` · Kesehatan `#10B981` · Lainnya `#64748B`

## 3. Tipografi

**Font utama:** [Plus Jakarta Sans](https://fonts.google.com/specimen/Plus+Jakarta+Sans) (fallback: Inter, system-ui).
**Angka:** gunakan *tabular figures* agar digit sejajar dan tidak "bergoyang" saat berubah.

| Token | Ukuran / Tinggi baris | Bobot | Kegunaan |
|-------|----------------------|-------|----------|
| `displayLarge` | 40 / 48 | 700 | Sisa saldo di dashboard |
| `headline` | 24 / 32 | 700 | Judul halaman |
| `title` | 18 / 26 | 600 | Judul kartu, bagian |
| `bodyLarge` | 16 / 24 | 400 | Teks utama, nominal di daftar |
| `body` | 14 / 20 | 400 | Teks biasa |
| `label` | 12 / 16 | 500 | Label, chip, keterangan kecil |

Format nominal: `Rp 1.250.000` (titik pemisah ribuan, tanpa desimal). Nilai negatif: `−Rp 50.000`.

## 4. Spasi, Bentuk, Elevasi

- **Skala spasi (px):** 4 · 8 · 12 · 16 · 24 · 32 · 48
- **Radius:** kecil 8 (chip, input) · sedang 12 (tombol) · besar 16 (kartu) · penuh 999 (pill)
- **Elevasi:** gunakan border `outline` 1px alih-alih bayangan tebal; bayangan hanya untuk sheet/dialog (`0 8 24 rgba(15,23,42,0.12)`).
- **Ukuran sentuh minimum:** 48 × 48 px.
- **Ikon:** Material Symbols Rounded, ukuran 20 / 24.

## 5. Komponen UI

### 5.1 Kartu Saldo (Balance Card)
- Label kecil "Sisa saldo bulan ini" → angka `displayLarge`.
- Di bawahnya: baris "Saldo awal" dan "Pemasukan tambahan".
- Warna angka mengikuti status (`textPrimary` normal, `danger` jika negatif).

### 5.2 Kartu Batas Pengeluaran (Limit Card)
- Progress bar horizontal tinggi 10px, radius penuh.
- Warna bar: `safe` → `warning` → `danger` sesuai persentase.
- Teks: "Rp 1.800.000 dari Rp 2.500.000 · 72%".
- Chip status di kanan: Aman / Hati-hati / Melewati batas (dengan ikon).

### 5.3 Item Transaksi (Transaction Tile)
- Kiri: lingkaran ikon kategori (warna kategori 15% opasitas).
- Tengah: kategori (bodyLarge) + catatan/tanggal (label, textSecondary).
- Kanan: nominal; pengeluaran `expense` dengan tanda "−", pemasukan `income` dengan "+".
- Geser kiri = hapus (dengan konfirmasi Undo), ketuk = ubah.

### 5.4 Form Tambah Transaksi (Bottom Sheet / Dialog)
- Toggle segmen **Pengeluaran | Pemasukan** di atas.
- Input nominal besar di tengah dengan keypad numerik, otomatis memberi titik ribuan.
- Pemilih kategori berupa chip yang bisa digulir.
- Tanggal (default hari ini), catatan opsional.
- Tombol "Simpan" lebar penuh, `primary`.

### 5.5 Tombol
| Jenis | Gaya |
|-------|------|
| Primary | Isi `primary`, teks `onPrimary`, tinggi 48 |
| Secondary | Border `outline`, teks `textPrimary` |
| Text | Tanpa latar, teks `primary` |
| Destructive | Isi `danger`, hanya untuk hapus |
| FAB | Lingkaran 56px `primary`, ikon "+" , kanan-bawah (HP) |

### 5.6 Komponen lain
- **Chip kategori:** pill, 32px tinggi, terpilih = `primaryContainer` + border `primary`.
- **Input:** `surfaceVariant`, radius 8, label mengambang, error teks `danger`.
- **Snackbar:** untuk konfirmasi dan "Urungkan".
- **Pemilih bulan:** panah kiri/kanan + nama bulan di header.
- **Empty state:** ikon besar abu-abu + satu kalimat + tombol "Tambah transaksi pertama".

## 6. Layar & Tata Letak

### HP (< 600px)
Navigasi bawah dengan 3 tab: **Beranda · Riwayat · Pengaturan**.

```
┌──────────────────────┐
│ ‹  Oktober 2026   ›  │
│ ┌──────────────────┐ │
│ │ Sisa saldo       │ │
│ │ Rp 1.450.000     │ │
│ └──────────────────┘ │
│ ┌──────────────────┐ │
│ │ Batas  ▓▓▓▓▓░░ 72%│ │
│ └──────────────────┘ │
│ Transaksi terbaru    │
│  🍜 Makan   −25.000  │
│  🚌 Transp. −10.000  │
│                  [+] │
│ [Beranda][Riwayat][⚙]│
└──────────────────────┘
```

### Laptop (≥ 840px)
Navigasi samping (rail), dua kolom: kiri ringkasan saldo + batas, kanan daftar transaksi. Form tambah muncul sebagai dialog di tengah.

### Layar
1. **Beranda** — saldo, batas, transaksi terbaru.
2. **Riwayat** — daftar per bulan, filter kategori, pencarian.
3. **Tambah/Ubah Transaksi** — bottom sheet/dialog.
4. **Pengaturan** — jatah bulanan, batas boros, ambang peringatan (default 80% & 100%), kategori, tema, ekspor, keluar.
5. **Login** — satu form email + kata sandi.

## 7. Widget Homescreen (Android)

| Ukuran | Isi |
|--------|-----|
| Kecil (2×1) | "Sisa saldo" + nominal |
| Sedang (4×2) | Sisa saldo, pengeluaran bulan ini, progress bar batas, tombol "+" |

- Warna progress bar mengikuti status `safe/warning/danger`.
- Mengikuti tema terang/gelap sistem.
- Ketuk widget → buka aplikasi; tombol "+" → langsung ke form tambah.
- Nilai terakhir tetap tampil saat offline, dengan keterangan waktu "diperbarui HH:mm".

## 8. Notifikasi

| Pemicu | Judul | Isi contoh | Ikon |
|--------|-------|------------|------|
| ≥ ambang peringatan (80%) | Hampir mencapai batas | "Pengeluaran bulan ini sudah 82% dari batas (Rp 2.050.000 / Rp 2.500.000)." | ⚠ `warning` |
| ≥ 100% batas | Batas pengeluaran terlewati | "Kamu sudah melewati batas bulan ini sebesar Rp 120.000." | ⛔ `danger` |
| Saldo negatif | Saldo habis | "Sisa saldo bulan ini −Rp 80.000." | ⛔ `danger` |

Nada bahasa: lugas, tidak menghakimi. Ketuk notifikasi → buka Beranda.

## 9. Gerak & Interaksi

- Durasi transisi 150–250 ms, kurva *ease-out*.
- Angka saldo beranimasi hitung (count) ≤ 400 ms saat berubah.
- Progress bar beranimasi halus saat nilai bertambah.
- Hormati pengaturan "kurangi animasi" dari sistem.

## 10. Aksesibilitas

- Kontras teks minimal 4.5:1 (WCAG AA).
- Semua ikon interaktif punya label semantik (screen reader).
- Dukung ukuran teks sistem hingga 200% tanpa terpotong.
- Status tidak hanya dibedakan warna (ikon + teks pendamping).