# GearMarket — Pasar Telegram Mini App

Aplikasi pasar (*marketplace*) serba lengkap untuk game **Car Parking Multiplayer 2 (CPM2)** dalam format Telegram Mini App. Pengguna membeli, menjual, dan bertukar setelan (transmisi/gearbox, vinyl, tuning, suspensi, dll.) menggunakan Telegram Stars.

## Stack Teknologi

| Lapisan | Teknologi |
|------|-----------|
| **Backend** | Node.js 20+, Hono v4, grammY v1.38, TypeScript 5.8 |
| **Frontend** | React 18, Vite 6, TypeScript 5.8, @telegram-apps/sdk v3 |
| **Database** | PostgreSQL 17, Drizzle ORM v0.45, postgres.js |
| **Monorepo** | npm workspaces (`@gm/server`, `@gm/web`, `@gm/shared`) |
| **Penempatan (Deploy)** | Railway (backend), Vercel (frontend) |

## Struktur Proyek

```
CPM2Hub_TMA/
├── apps/
│   ├── server/                 # Backend Hono + grammY
│   │   └── src/
│   │       ├── index.ts        # Entry point, middleware, rute, webhook
│   │       ├── bot.ts          # Perintah bot, penanganan pembayaran, escrow
│   │       ├── db/             # Skema, migrasi, seed
│   │       ├── lib/            # Logika bisnis (stars, pembayaran, gamifikasi, escrow, market)
│   │       └── routes/         # Rute API (auth, me, products, purchases, trades, admin, avatar)
│   └── web/                    # Frontend React + Vite
│       └── src/
│           ├── pages/          # 6 halaman (Market, Tools, Escrow, Sell, Profile, Admin)
│           ├── components/     # 10 komponen
│           ├── tools/          # 5 alat (kalkulator transmisi, suspensi, komparasi, warna, nama)
│           ├── api/            # Klien API
│           └── styles/         # Tema gelap (1900+ baris CSS)
├── packages/shared/            # Tipe umum dan konstanta
├── docker-compose.yml          # PostgreSQL untuk pengembangan lokal
├── railway.json                # Konfigurasi deploy backend
└── vercel.json                 # Konfigurasi deploy frontend
```

## Fitur dan Kemampuan

### Pasar (Marketplace)
- 17 kategori produk (transmisi, vinyl, tuning, nama panggilan, body kit, velg/diska, mesin, suspensi, pelat nomor, knalpot, neon, garasi, akun, layanan, asap, karakter, bundel)
- Pencarian, pemfilteran berdasarkan kategori dan model mobil
- Penilaian produk (1-5 bintang)
- Daftar keinginan (wishlist)
- Moderasi konten (pemeriksaan kata-kata kasar secara otomatis untuk pelat nomor, moderasi manual untuk kategori berisiko)

### Pembayaran
- **Mata Uang Ganda:** Telegram Stars (eksternal) + TN (internal)
- Pemilihan otomatis: TN → Saldo Stars → Faktur Telegram
- Pengisian saldo (*Top-up*) (1-10000 Stars)
- Pengembalian dana (*refund*) (atomik, dengan kompensasi jika terjadi kesalahan pada API Telegram)
- Rekonsiliasi transaksi di latar belakang setiap 60 detik

### Gamifikasi
- **Bonus Harian:** 30 Stars + bonus beruntun (+30 Stars setiap 7 hari)
- **Sistem Referal:** kode unik, 50 Stars untuk setiap pengguna yang diundang
- **Rentetan (*streak*):** pelacakan masuk harian secara terus-menerus
- **Desain Livery Harian:** konfigurasi vinyl unik setiap hari

### Perdagangan / Escrow
- Pertukaran P2P antar pengguna (uang, mobil, vinyl)
- Mesin status (*state machine*): waiting → escrow → completed / cancelled / disputed
- Tombol kontrol pada bot Telegram (accept/decline/cancel/complete/dispute)

### Panel Admin
- Statistik: saldo bot, pendapatan, penjualan platform
- Moderasi: persetujuan/penolakan daftar barang (*listing*)
- Pengembalian dana (*refund*) dengan konfirmasi
- Penambahan Stars/TN ke pengguna berdasarkan ID Telegram

### Alat & Perkakas (Tools)
- **Kalkulator Transmisi (Gearbox)** — perhitungan kecepatan berdasarkan RPM
- **Kalkulator Suspensi** — pembuatan kode konfigurasi, tag (stance/track/comfort)
- **Komparasi Mobil** — HP, torsi, bobot, 0-100 km/j, kecepatan maksimum
- **Palet RGB** — pemilih warna dengan opsi *preset*
- **Generator Nama Panggilan (Nickname)** — gaya font Unicode

### Antarmuka (UI)
- 6 halaman: Market, Tools, Escrow, Sell, Profile, Admin
- Tema gelap (desain bergaya Telegram)
- Navigasi tab di bagian bawah
- Layar tampilan awal (*splash screen*) dengan pemuatan data di awal (*preload*)
- Proksi avatar (penyimpanan tembolok/cache 24 jam)
- i18n: Bahasa Inggris + Bahasa Rusia (~340 kunci)

## Panduan Memulai Cepat

### 1. Kloning dan Instal Dependensi

```bash
git clone https://github.com/lambadaGG/CPM2Hub.git
cd CPM2Hub_TMA
npm install
```

### 2. Jalankan PostgreSQL

```bash
docker compose up -d
```

### 3. Konfigurasi Variabel Lingkungan (*Environment Variables*)

```bash
cp apps/server/.env.example apps/server/.env
```

Isi berkas `apps/server/.env`:

```env
BOT_TOKEN=token_bot_telegram_anda
DATABASE_URL=postgresql://gearmarket:gearmarket@localhost:5434/gearmarket
WEBAPP_URL=https://frontend-anda.vercel.app
ALLOW_ANON=1
ADMIN_IDS=id_telegram_anda
```

### 4. Terapkan Migrasi dan Isi Database

```bash
npm run db:push -w @gm/server
npm run db:seed -w @gm/server
```

### 5. Jalankan dalam Mode Pengembang (*Dev Mode*)

```bash
# Backend (http://localhost:8080)
npm run dev:server

# Frontend (http://localhost:5173)
npm run dev:web
```

## Variabel Lingkungan (*Environment Variables*)

| Variabel | Wajib | Deskripsi |
|-----------|:-----------:|---------|
| `BOT_TOKEN` | Ya | Token API Bot Telegram |
| `WEBAPP_URL` | Ya | URL frontend untuk tombol WebApp |
| `PUBLIC_URL` | Tidak | URL backend untuk webhook (jika tidak diatur — menggunakan polling) |
| `WEBHOOK_SECRET` | Tidak | Rahasia untuk verifikasi webhook |
| `BOT_USERNAME` | Tidak | Username bot tanpa @ (default `cpm2hub_bot`) |
| `PORT` | Tidak | Port server (default 8080) |
| `DATABASE_URL` | Ya | String koneksi PostgreSQL |
| `ALLOW_ANON` | Tidak | `1` untuk menonaktifkan otentikasi saat pengujian (*dev*) |
| `ADMIN_IDS` | Tidak | ID Telegram administrator dipisahkan dengan koma |

## API

Semua *endpoint* memiliki prefiks `/api` dan memerlukan otentikasi melalui header `X-Init-Data`.

### Produk
| Metode | Endpoint | Deskripsi |
|-------|---------|---------|
| GET | `/products` | Daftar produk (filter `?category=`) |
| GET | `/products/mine` | Produk saya |
| POST | `/products` | Buat daftar produk (*listing*) |
| PATCH | `/products/:id` | Perbarui daftar produk |
| DELETE | `/products/:id` | Hapus daftar produk |
| POST | `/products/:id/pay` | Bayar produk |
| POST | `/products/:id/rate` | Beri nilai pada produk |
| POST | `/products/:id/wishlist` | Tambah/hapus dari daftar keinginan |

### Pembelian dan Pengisian Saldo (*Top-up*)
| Metode | Endpoint | Deskripsi |
|-------|---------|---------|
| GET | `/my/downloads` | Semua unduhan |
| POST | `/products/:id/buy` | Buat faktur untuk pembelian |
| POST | `/topup` | Buat faktur untuk pengisian saldo |

### Profil
| Metode | Endpoint | Deskripsi |
|-------|---------|---------|
| GET | `/me` | Profil pengguna |
| GET | `/me/referral` | Tautan referal |
| POST | `/me/claim-daily` | Klaim bonus harian |

### Perdagangan (*Trades*)
| Metode | Endpoint | Deskripsi |
|-------|---------|---------|
| GET | `/trades` | Daftar perdagangan |
| POST | `/trades` | Buat perdagangan |
| POST | `/trades/:id/:action` | Tindakan (accept/decline/cancel/complete/dispute) |

### Admin
| Metode | Endpoint | Deskripsi |
|-------|---------|---------|
| GET | `/admin/stars` | Statistik bot |
| GET | `/admin/purchases` | Semua pembelian |
| POST | `/admin/refund` | Pengembalian dana (*refund*) |
| POST | `/admin/grant` | Berikan kredit |
| POST | `/admin/products/:id/moderate` | Moderasi |

## Bot

### Perintah
| Perintah | Deskripsi |
|---------|---------|
| `/start` | Salam + Tombol WebApp + Tautan |
| `/terms` | Syarat dan Ketentuan |
| `/support` | Dukungan |
| `/paysupport` | Dukungan Pembayaran |

### Penangan (*Handlers*)
- `pre_checkout_query` — validasi sebelum pembayaran
- `message:successful_payment` — pemrosesan pembayaran berhasil
- `callback_query` — tombol escrow (accept/decline/cancel/complete/dispute)

## Pengujian (*Tests*)

```bash
# Semua pengujian
npm test

# Khusus backend
node --import tsx --test "apps/server/src/**/*.test.ts"

# Khusus frontend
node --import tsx --test "apps/web/src/**/*.test.ts"
```

Terdapat 12 berkas pengujian: escrow, gamification, initData, market, payments, products (backend) + utils, gearbox, suspension, compare, color, nick (frontend).

## Penempatan (*Deploy*)

### Backend (Railway)

Penempatan otomatis melalui Nixpacks:

```bash
npm run build:server    # esbuild → dist/index.cjs
npm run start:prod      # node dist/index.cjs
```

### Frontend (Vercel)

```bash
npm run build -w @gm/web    # vite build → apps/web/dist
```

Pengalihan SPA (*SPA rewrites*), header CSP (frame-ancestors untuk Telegram), dan penyiapan *caching asset*.

## Lisensi

Proyek privat. Hak cipta dilindungi undang-undang.
