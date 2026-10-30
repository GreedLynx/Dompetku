# Architecture — Aplikasi Dompetku

Dokumen teknis: teknologi, alur sistem, dan database.

---

## 1. Ringkasan

Aplikasi **offline-first** dengan satu basis kode Flutter untuk Android dan Windows (iOS opsional). Setiap perangkat menyimpan salinan data di database lokal (SQLite); cloud (Supabase/PostgreSQL) bertugas sebagai sumber sinkronisasi antar perangkat. Karena penggunanya satu orang, arsitektur sengaja dibuat sederhana.

```
┌───────────────┐        ┌───────────────┐
│  HP (Android) │        │ Laptop (Win)  │
│  Flutter App  │        │  Flutter App  │
│  ┌─────────┐  │        │  ┌─────────┐  │
│  │ SQLite  │  │        │  │ SQLite  │  │
│  │ (Drift) │  │        │  │ (Drift) │  │
│  └────┬────┘  │        │  └────┬────┘  │
└───────┼───────┘        └───────┼───────┘
        │   HTTPS / Realtime     │
        └──────────┬─────────────┘
                   ▼
        ┌─────────────────────┐
        │      Supabase       │
        │ Auth · Postgres ·   │
        │ Realtime · RLS      │
        └─────────────────────┘
```

## 2. Teknologi

| Lapisan | Teknologi | Alasan |
|---------|-----------|--------|
| UI & logika | **Flutter (Dart)** | Satu kode untuk Android, Windows, iOS |
| State management | **Riverpod** | Sederhana, mudah diuji, reaktif |
| Navigasi | **go_router** | Standar deklaratif |
| Database lokal | **Drift** (SQLite) | Type-safe, reaktif (stream), mendukung semua platform |
| Backend | **Supabase** (PostgreSQL + Auth + Realtime) | Gratis untuk skala pribadi, SQL standar |
| Klien backend | `supabase_flutter` | SDK resmi |
| Notifikasi | `flutter_local_notifications` | Notifikasi lokal, tanpa server push |
| Widget Android | `home_widget` + Kotlin (Glance / AppWidget) | Jembatan data Flutter → widget native |
| Penyimpanan aman | `flutter_secure_storage` | Menyimpan sesi/token |
| Format & lokal | `intl` | Format rupiah dan tanggal Indonesia |
| Model & serialisasi | `freezed` + `json_serializable` | Model imutabel, aman |
| Pengujian | `flutter_test`, `mocktail` | Unit & widget test |
| Lint | `flutter_lints` / `very_good_analysis` | Konsistensi kode |

**Catatan notifikasi:** karena input manual, pengecekan batas cukup dilakukan di perangkat **saat transaksi disimpan**. Tidak perlu server push, cron, atau proses latar belakang.

## 3. Struktur Proyek

```
lib/
├── main.dart
├── app/                      # router, tema, bootstrap
├── core/                     # util, konstanta, format rupiah, error
├── data/
│   ├── local/                # Drift: tabel, DAO, database
│   ├── remote/               # Supabase client, mapper
│   └── repositories/         # implementasi repository
├── domain/
│   ├── models/               # Transaction, Category, MonthlyBudget, Settings
│   ├── repositories/         # kontrak (abstract)
│   └── usecases/             # hitung saldo, cek ambang, buat bulan baru
├── features/
│   ├── dashboard/
│   ├── transactions/
│   ├── history/
│   ├── settings/
│   └── auth/
├── services/
│   ├── sync_service.dart
│   ├── notification_service.dart
│   └── widget_service.dart
└── shared/                   # widget UI bersama (design system)
android/app/src/main/kotlin/  # widget native
supabase/migrations/          # SQL migrasi
test/
```

Pola: **Clean-ish layered** — `features` → `domain` (usecase) → `data` (repository). UI tidak pernah menyentuh Drift atau Supabase langsung.

## 4. Model Data

### 4.1 Tabel

Semua nominal berupa **integer rupiah** (`bigint`). Semua ID memakai **UUID** yang dibuat di klien agar bisa dibuat saat offline.

**`categories`**
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | uuid PK | |
| user_id | uuid | `auth.uid()` |
| name | text | |
| icon | text | nama ikon |
| color | text | hex |
| is_default | bool | kategori bawaan |
| updated_at | timestamptz | untuk sinkronisasi |
| deleted_at | timestamptz null | soft delete |

**`transactions`**
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | uuid PK | |
| user_id | uuid | |
| type | text | `expense` \| `income` |
| amount | bigint | > 0 |
| category_id | uuid FK | boleh null untuk pemasukan |
| note | text null | |
| occurred_at | date | tanggal transaksi |
| created_at | timestamptz | |
| updated_at | timestamptz | |
| deleted_at | timestamptz null | |

Indeks: `(user_id, occurred_at)`, `(user_id, updated_at)`.

**`settings`** (satu baris per pengguna)
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| user_id | uuid PK | |
| monthly_allowance | bigint | jatah bulanan default |
| spending_limit | bigint | batas boros default |
| warn_percent | int | default 80 |
| limit_percent | int | default 100 |
| start_month | date | bulan pertama pemakaian |
| updated_at | timestamptz | |

**`monthly_budgets`** (snapshot per bulan)
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| id | uuid PK | |
| user_id | uuid | |
| month | date | selalu tanggal 1 |
| allowance | bigint | jatah bulan itu (disalin dari settings) |
| spending_limit | bigint | batas bulan itu |
| warn_notified_at | timestamptz null | penanda notifikasi 80% sudah terkirim |
| limit_notified_at | timestamptz null | penanda notifikasi 100% sudah terkirim |
| updated_at | timestamptz | |

Unik: `(user_id, month)`.

### 4.2 Perhitungan (tidak disimpan, selalu dihitung)

Saldo **tidak disimpan** sebagai kolom, tetapi dihitung dari data mentah agar tidak pernah tidak cocok antar perangkat.

```
Sisa saldo akhir bulan M =
    Σ allowance (semua bulan ≤ M)
  + Σ income  (semua transaksi ≤ akhir bulan M)
  − Σ expense (semua transaksi ≤ akhir bulan M)

Saldo awal bulan M = Sisa saldo akhir bulan (M−1) + allowance M
Sisa saldo saat ini = Saldo akhir bulan berjalan
Persentase batas    = Σ expense bulan M ÷ spending_limit M
```

Pendekatan kumulatif ini membuat carry over otomatis benar, termasuk saat transaksi lama diubah atau dihapus.

### 4.3 SQL Supabase (ringkas)

```sql
create table transactions (
  id uuid primary key,
  user_id uuid not null default auth.uid(),
  type text not null check (type in ('expense','income')),
  amount bigint not null check (amount > 0),
  category_id uuid references categories(id),
  note text,
  occurred_at date not null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  deleted_at timestamptz
);

alter table transactions enable row level security;

create policy "own rows" on transactions
  for all using (user_id = auth.uid())
  with check (user_id = auth.uid());

-- trigger untuk menjaga updated_at
create function set_updated_at() returns trigger as $$
begin new.updated_at = now(); return new; end; $$ language plpgsql;

create trigger trg_tx_updated before update on transactions
  for each row execute function set_updated_at();
```

Tabel lain memakai pola RLS yang sama. Pendaftaran akun baru **dimatikan** di dashboard Supabase setelah akun pemilik dibuat.

## 5. Alur Sistem

### 5.1 Menyimpan transaksi
```
UI form ──▶ AddTransactionUseCase
              │ 1. validasi (nominal > 0, kategori)
              │ 2. tulis ke SQLite (updated_at = now)
              │ 3. hitung ulang saldo & persentase (stream Drift)
              │ 4. CheckThresholdUseCase
              │      └─ jika ambang terlewati & belum diberi tanda
              │         → NotificationService.show()
              │         → tandai warn_notified_at / limit_notified_at
              │ 5. WidgetService.update(saldo, pengeluaran, persen)
              └ 6. tandai "perlu sinkron" → SyncService
```

### 5.2 Sinkronisasi
- **Push:** ambil baris lokal dengan `updated_at` > `last_pushed_at`, kirim `upsert` ke Supabase.
- **Pull:** ambil baris server dengan `updated_at` > `last_pulled_at`, `upsert` ke SQLite.
- **Realtime:** berlangganan perubahan tabel agar perangkat lain terbarui seketika saat online.
- **Konflik:** *last write wins* berdasarkan `updated_at`.
- **Hapus:** `deleted_at` diisi (soft delete); baris dibersihkan permanen secara berkala bila perlu.
- **Pemicu:** saat aplikasi dibuka, setelah setiap perubahan lokal, saat koneksi kembali, dan lewat realtime.
- **Jam perangkat:** `updated_at` pada upsert sebaiknya ditentukan server (trigger) agar selisih jam HP/laptop tidak merusak urutan.

### 5.3 Pergantian bulan
```
App dibuka ──▶ EnsureMonthUseCase
   ada monthly_budgets untuk bulan ini? ── ya ──▶ selesai
        │ tidak
        ▼
   salin allowance & spending_limit dari settings
   (buat juga baris untuk bulan terlewat bila ada celah)
```

### 5.4 Notifikasi ambang
```
persen = expense_bulan ÷ spending_limit
jika persen ≥ limit_percent & limit_notified_at null → notifikasi "terlewati"
jika persen ≥ warn_percent  & warn_notified_at  null → notifikasi "hampir"
jika persen turun di bawah ambang (karena edit/hapus/ubah batas)
   → tanda notified dikosongkan agar bisa memicu lagi
```

### 5.5 Widget (Android)
```
WidgetService.update() ─▶ home_widget.saveWidgetData(...)
                       ─▶ home_widget.updateWidget(...)
Widget native (Kotlin) membaca SharedPreferences yang sama
dan menampilkan: sisa saldo, pengeluaran, persen, waktu diperbarui.
```
Widget tidak melakukan query jaringan; ia hanya menampilkan data terakhir yang didorong aplikasi.

### 5.6 Autentikasi
- Satu akun email + kata sandi di Supabase Auth, pendaftaran publik dimatikan.
- Sesi disimpan di `flutter_secure_storage`; masuk otomatis berikutnya.
- Aplikasi tetap berfungsi offline selama sesi lokal ada.

## 6. Keamanan

- RLS aktif di semua tabel (`user_id = auth.uid()`).
- Hanya `anon key` Supabase di aplikasi; **`service_role` key tidak pernah masuk kode klien atau repositori**.
- Kunci dan URL lewat konfigurasi env (`--dart-define`), bukan di-commit.
- HTTPS saja. Opsional: kunci aplikasi biometrik.
- Cadangan: ekspor CSV rutin + fitur backup bawaan Supabase bila tersedia.

## 7. Strategi Pengujian

| Level | Cakupan |
|-------|---------|
| Unit | Rumus saldo, carry over, ambang notifikasi, format rupiah (**wajib lengkap**) |
| Repository | Drift in-memory, logika sinkron dengan remote palsu |
| Widget | Dashboard, form transaksi |
| Manual | Sinkron 2 perangkat, mode offline, widget, notifikasi |

Kasus uji kunci: bulan tanpa transaksi, saldo negatif terbawa, edit transaksi bulan lalu, ganti batas di tengah bulan, hapus lalu sinkron, transaksi dibuat offline di dua perangkat.

## 8. Tahap Pengerjaan

1. Setup proyek, tema, struktur folder, Supabase + migrasi + RLS
2. Database lokal + model + usecase perhitungan saldo (dengan test)
3. UI: Dashboard, form transaksi, riwayat, pengaturan
4. Autentikasi + sinkronisasi + realtime
5. Notifikasi ambang
6. Widget Android
7. Polesan: tema gelap, ekspor CSV, kunci aplikasi