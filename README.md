# Dompetku

Aplikasi pencatatan keuangan **pribadi** untuk Android dan laptop. Data dimasukkan manual, tersinkron otomatis antar perangkat, dan memberi peringatan saat pengeluaran mulai boros.

> Proyek untuk penggunaan pribadi (satu pengguna). Status: **dalam perencanaan**.

## Fitur

- Catat pengeluaran dan pemasukan secara manual dengan cepat
- **Saldo bulanan** yang berkurang tiap pengeluaran, dengan sisa saldo bulan lalu terbawa ke bulan berikutnya
- Jatah bulanan otomatis (nominal tetap yang bisa diubah)
- **Batas boros** yang ditentukan sendiri, dengan **notifikasi peringatan** di ambang 80% dan 100% (bisa diatur)
- **Widget** homescreen Android: sisa saldo, pengeluaran bulan ini, dan progres batas
- Sinkronisasi antar perangkat, tetap bisa dipakai **offline**
- Kategori bawaan dan kategori buatan sendiri
- Tema terang dan gelap

## Teknologi

| Lapisan | Teknologi |
|---------|-----------|
| Aplikasi | Flutter (Dart) |
| State management | Riverpod |
| Database lokal | Drift (SQLite) |
| Backend & sinkronisasi | Supabase (PostgreSQL, Auth, Realtime) |
| Notifikasi | flutter_local_notifications |
| Widget Android | home_widget + Kotlin |

Detail lengkap ada di [Architecture.md](Architecture.md).

## Dokumentasi

| Dokumen | Isi |
|---------|-----|
| [PRD.md](PRD.md) | Masalah yang diselesaikan, proses bisnis, kebutuhan produk |
| [SYSTEM_DESIGN.md](SYSTEM_DESIGN.md) | Warna, tipografi, komponen UI, tata letak |
| [Architecture.md](Architecture.md) | Teknologi, alur sistem, skema database |
| [agent.md](agent.md) | Aturan kerja dan kode untuk AI coding agent |

## Prasyarat

- [Flutter SDK](https://docs.flutter.dev/get-started/install) (channel stable)
- Android Studio (untuk Android SDK dan emulator)
- Visual Studio dengan workload *Desktop development with C++* (untuk build Windows)
- Akun [Supabase](https://supabase.com) (paket gratis cukup)
- [Supabase CLI](https://supabase.com/docs/guides/cli) (opsional, untuk migrasi lokal)

## Menjalankan Proyek

```bash
# 1. Ambil dependensi
flutter pub get

# 2. Generate kode (Drift, freezed, json_serializable)
dart run build_runner build --delete-conflicting-outputs

# 3. Jalankan dengan konfigurasi Supabase
flutter run -d windows \
  --dart-define=SUPABASE_URL=https://<project-id>.supabase.co \
  --dart-define=SUPABASE_ANON_KEY=<anon-key>
```

Untuk Android, ganti `-d windows` dengan ID perangkat dari `flutter devices`.

## Konfigurasi Supabase

1. Buat proyek baru di Supabase.
2. Terapkan migrasi di folder `supabase/migrations/` (lewat `supabase db push` atau SQL Editor).
3. Pastikan **Row Level Security** aktif di semua tabel.
4. Buat satu akun (email + kata sandi), lalu **matikan pendaftaran publik** di *Authentication → Providers → Email*.
5. Salin `Project URL` dan `anon key` ke perintah `--dart-define` di atas.

> **Keamanan:** hanya gunakan `anon key` di aplikasi. Jangan pernah memasukkan `service_role` key ke kode atau repositori.

## Struktur Folder

```
lib/
├── app/          # router, tema, bootstrap
├── core/         # util, konstanta, format rupiah
├── data/         # database lokal, remote, repository
├── domain/       # model, kontrak repository, usecase
├── features/     # dashboard, transaksi, riwayat, pengaturan, auth
├── services/     # sync, notifikasi, widget
└── shared/       # komponen UI bersama
android/          # kode widget native (Kotlin)
supabase/         # migrasi SQL
test/
```

## Pengembangan

```bash
dart format .        # rapikan kode
flutter analyze      # analisis statis (harus bersih)
flutter test         # jalankan test
```

Aturan lengkap penulisan kode ada di [agent.md](agent.md).

## Rencana Rilis

| Fase | Isi |
|------|-----|
| MVP | Input transaksi, saldo bulanan, batas, sinkronisasi, notifikasi |
| v1.1 | Widget Android, riwayat dan filter |
| v1.2 | Grafik kategori, ekspor CSV, kunci aplikasi |

## Lisensi

Proyek pribadi. Hak cipta © 2026 Ndibil. Seluruh hak dilindungi.

Kode ini tidak dilisensikan untuk digunakan, disalin, atau didistribusikan pihak lain tanpa izin tertulis.
