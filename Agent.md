# agent.md — Panduan Kerja AI

Dokumen ini adalah aturan untuk AI coding agent (OpenCode, Claude Code, dan sejenisnya) yang mengerjakan proyek **Dompetku**. Baca seluruhnya sebelum menulis kode.

---

## 1. Konteks Singkat

- Aplikasi pencatatan keuangan **pribadi, satu pengguna**, berjalan di Android dan Windows (iOS opsional).
- Input transaksi manual; data sinkron antar perangkat; ada notifikasi peringatan boros, widget, dan saldo bulanan dengan carry over.
- Stack: **Flutter + Riverpod + Drift (SQLite) + Supabase**.
- Dokumen acuan (urutan prioritas jika ada konflik):
  1. `PRD.md` — apa yang dibangun dan aturan bisnisnya
  2. `Architecture.md` — teknologi, struktur, database, alur
  3. `SYSTEM_DESIGN.md` — token visual dan komponen UI
  4. `agent.md` — cara kerja (dokumen ini)

Jika permintaan bertentangan dengan dokumen di atas, **tanyakan dulu**, jangan menebak.

## 2. Cara Kerja Agent

### 2.1 Alur untuk setiap tugas
1. **Pahami:** baca dokumen dan file terkait. Jangan mengubah file yang belum dibaca.
2. **Rencanakan:** untuk tugas > 1 file, tulis rencana singkat (file yang disentuh, langkah, risiko) sebelum mengeksekusi.
3. **Kerjakan kecil:** satu fitur/perubahan per langkah. Hindari perubahan besar sekaligus.
4. **Verifikasi:** jalankan `dart format`, `flutter analyze`, dan `flutter test`. Perbaiki sampai bersih.
5. **Laporkan:** ringkas apa yang diubah, apa yang diuji, dan hal yang belum selesai.

### 2.2 Prinsip
- **Jangan berasumsi pada aturan keuangan.** Rumus saldo, carry over, dan ambang notifikasi harus mengikuti `PRD.md` §4 dan `Architecture.md` §4.2.
- **Tanya bila ambigu**, terutama soal perilaku data (hapus, konflik sinkron, migrasi).
- **Jangan menambah fitur di luar cakupan.** Hal di "Non-Tujuan" PRD tidak boleh dikerjakan.
- **Jangan menambah dependensi tanpa izin.** Usulkan dulu dengan alasan; gunakan yang sudah ada di `Architecture.md` §2.
- **Jujur soal batas:** bila tidak yakin atau tidak bisa menguji sesuatu (misal widget di perangkat nyata), katakan terus terang.
- Utamakan solusi **paling sederhana** yang memenuhi kebutuhan satu pengguna.

## 3. Aturan Kode

### 3.1 Umum
- Bahasa kode (nama kelas, fungsi, variabel, komentar kode): **Inggris**. Teks yang tampil ke pengguna dan dokumentasi: **Indonesia**.
- Ikuti `flutter_lints`/`very_good_analysis`; **nol warning** sebelum selesai.
- Format dengan `dart format` (lebar baris 80).
- Penamaan: `UpperCamelCase` kelas, `lowerCamelCase` fungsi/variabel, `snake_case` nama file, `kConstantName` hanya bila perlu.
- Fungsi pendek dan satu tanggung jawab; file > 300 baris dipecah.
- Komentar menjelaskan **mengapa**, bukan **apa**.
- Tidak ada kode mati, `print`, atau `TODO` tanpa catatan alasan.

### 3.2 Arsitektur
- Ikuti struktur folder di `Architecture.md` §3.
- Arah dependensi: `features` → `domain` → `data`. **UI dilarang** memanggil Drift atau Supabase langsung.
- Logika bisnis (hitung saldo, cek ambang, buat bulan baru) berada di `domain/usecases`, murni Dart tanpa dependensi Flutter agar mudah diuji.
- State dengan **Riverpod**; tidak mencampur pola state management lain.
- Model memakai `freezed` (imutabel).

### 3.3 Data & uang
- **Nominal selalu `int` rupiah.** Dilarang memakai `double` untuk uang.
- Waktu disimpan UTC; tanggal transaksi disimpan sebagai tanggal tanpa jam.
- ID baru dibuat di klien (UUID v4) agar bisa dibuat offline.
- Hapus data = **soft delete** (`deleted_at`), bukan hapus fisik.
- Saldo **tidak disimpan**; selalu dihitung dari transaksi dan `monthly_budgets`.
- Setiap perubahan baris wajib memperbarui `updated_at`.
- Format rupiah lewat satu fungsi util di `core/`; jangan memformat manual di UI.

### 3.4 UI
- Pakai **token** dari `SYSTEM_DESIGN.md` (warna, tipografi, spasi, radius) lewat `ThemeData`/extension. **Dilarang hardcode** warna, ukuran font, atau spasi di widget.
- Buat komponen bersama di `shared/`; jangan menduplikasi komponen yang sama di tiap fitur.
- Status tidak boleh hanya dibedakan warna; tambahkan ikon/teks.
- Dukung tema terang dan gelap, serta layout HP (< 600) dan laptop (≥ 840).
- Semua teks tampilan lewat satu tempat (siap dilokalkan), bukan tersebar acak.

### 3.5 Keamanan
- **Jangan pernah** menaruh `service_role` key, kata sandi, atau token di kode, log, atau commit.
- Konfigurasi rahasia lewat `--dart-define` / file env yang masuk `.gitignore`.
- Semua tabel baru **wajib** RLS dengan `user_id = auth.uid()`.
- Jangan mencatat data keuangan pengguna ke log.

### 3.6 Pengujian
- Setiap usecase keuangan **wajib** punya unit test (lihat kasus kunci di `Architecture.md` §7).
- Bug yang diperbaiki harus disertai test yang mereproduksinya.
- Test harus deterministik: waktu dan UUID di-*inject*, bukan dipanggil langsung.
- Jangan menghapus atau melemahkan test agar lulus.

### 3.7 Database & migrasi
- Perubahan skema lewat file migrasi di `supabase/migrations/` (bernomor, tidak diedit setelah diterapkan) dan versi skema Drift yang selaras.
- Migrasi yang berpotensi menghilangkan data **harus dikonfirmasi** ke pemilik terlebih dahulu.

## 4. Tools

| Kebutuhan | Tool | Catatan |
|-----------|------|---------|
| Framework | Flutter SDK (stable), Dart | Versi dikunci di `pubspec.yaml` / `.fvmrc` bila memakai FVM |
| Paket | `flutter pub` | Tambah dependensi hanya dengan izin |
| Generator kode | `dart run build_runner build --delete-conflicting-outputs` | Untuk Drift, freezed, json_serializable |
| Format | `dart format .` | Wajib sebelum selesai |
| Analisis | `flutter analyze` | Harus bersih |
| Test | `flutter test` | Harus lulus |
| Backend lokal | Supabase CLI (`supabase start`, `supabase db push`) | Untuk uji migrasi |
| Versi kontrol | Git | Lihat §5 |
| Desain | File Figma pemilik | Rujukan visual, token tetap di `SYSTEM_DESIGN.md` |

### Perintah rutin
```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
dart format .
flutter analyze
flutter test
flutter run -d windows     # laptop
flutter run -d <android>   # HP
```

### Dilarang tanpa izin eksplisit
- Menjalankan perintah destruktif (`rm -rf`, `git reset --hard`, `git push --force`, drop tabel).
- Mengubah file di luar folder proyek.
- Mengunggah data atau kode ke layanan eksternal di luar Supabase proyek ini.
- Mengganti stack/arsitektur.

## 5. Git

- Cabang: `main` (stabil), `feat/<nama>`, `fix/<nama>`.
- Commit kecil dan bermakna, format **Conventional Commits**: `feat: ...`, `fix: ...`, `refactor: ...`, `test: ...`, `docs: ...`, `chore: ...`.
- Jangan commit file hasil generate bila sudah di-ignore, juga file env dan kunci.
- Satu commit = satu perubahan logis yang lulus analyze dan test.

## 6. Definisi Selesai (Definition of Done)

Sebuah tugas dianggap selesai bila:
- [ ] Sesuai `PRD.md` dan aturan bisnisnya
- [ ] `dart format`, `flutter analyze` bersih; `flutter test` lulus
- [ ] Test baru ditambahkan untuk logika baru
- [ ] Tidak ada warna/ukuran hardcode; memakai token desain
- [ ] Tidak ada rahasia di kode; RLS terpasang untuk tabel baru
- [ ] Berfungsi saat offline (jika menyentuh data)
- [ ] Dokumen terkait diperbarui bila ada keputusan yang berubah
- [ ] Laporan singkat: apa yang diubah, cara menguji, hal yang belum selesai

## 7. Urutan Pengerjaan yang Disarankan

Ikuti tahap di `Architecture.md` §8. Jangan lompat ke widget atau notifikasi sebelum perhitungan saldo dan sinkronisasi stabil dan teruji.

## 8. Format Laporan Agent

```
## Ringkasan
(1–2 kalimat apa yang dikerjakan)

## Perubahan
- path/file.dart — alasan

## Pengujian
- Perintah yang dijalankan dan hasilnya

## Catatan / Perlu Keputusan
- Hal ambigu, risiko, atau yang belum selesai
```