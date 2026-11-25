# Backend · Admin Web Museum Mpu Purwa

API untuk **Admin Web Museum Mpu Purwa** (Express 5 + TypeScript + Prisma + MySQL/MariaDB). Repositori ini bagian dari tiga repositori: `museum-mpupurwa-frontend` (Next.js), **`museum-mpupurwa-backend`** (ini), dan `museum-mpupurwa-database` (padanan SQL murni).

## Teknologi

Node.js 20+, Express 5, TypeScript, **Prisma 6** + MySQL/MariaDB, bcryptjs, express-rate-limit, multer, **Vercel Blob** (penyimpanan foto dan PDF koleksi).

## Menjalankan

```bash
npm install
cp .env.example .env     # Windows PowerShell: copy .env.example .env
```

Isi `backend/.env` (lihat tabel di bawah), buat database kosong `museum_mpupurwa`, lalu:

```bash
npm run db:migrate       # menerapkan migrasi (membuat tabel)
npm run db:seed          # data awal
npm run dev              # http://localhost:5000
```

Akun demo setelah seed (semuanya kata sandi `admin123`): `admin@mpupurwa.test` (Super Admin), `dewi@mpupurwa.id` (admin), `hadi@mpupurwa.id` (nonaktif).

| Variabel | Isi |
|---|---|
| `PORT` | Port API (`5000`) |
| `NODE_ENV` | `development` atau `production` |
| `FRONTEND_URL` | Origin yang diizinkan CORS (`http://localhost:3000`) |
| `SESSION_SECRET` | Kunci penanda tangan cookie login (wajib di production) |
| `DATABASE_URL` | `mysql://USER:SANDI@localhost:3306/museum_mpupurwa` |
| `BLOB_READ_WRITE_TOKEN` | Token store Vercel Blob (akses **Public**) |

`.env` tidak boleh di-commit; hanya `.env.example`.

## Skrip

| Perintah | Fungsi |
|---|---|
| `npm run dev` | Jalankan dengan reload otomatis |
| `npm run build` / `npm start` | Build ke `dist/` lalu jalankan |
| `npm run db:migrate -- --name nama` | Buat dan terapkan migrasi baru |
| `npm run db:deploy` | Terapkan migrasi yang sudah ada (production) |
| `npm run db:seed` | Isi data awal |
| `npm run db:studio` | Prisma Studio |
| `npx tsc --noEmit` | Cek tipe |

Perubahan skema **hanya** lewat `prisma/schema.prisma` + migrasi (folder `prisma/migrations/` ikut di-commit), bukan lewat GUI.

## Struktur

Alur permintaan: **routes → controllers → services → models**.

```
src/
  app.ts            Express, CORS, JSON, pemasangan router
  server.ts         menjalankan server
  config/           env.ts (variabel environment) dan db.ts (koneksi Prisma)
  models/           satu file per tabel: SATU-SATUNYA yang menyentuh database
  services/         logika bisnis (memanggil models)
  controllers/      baca request, validasi, panggil service, kirim respons
  routes/           auth, dashboard, visitors, accounts, bookings, collections, uploads, public
  middleware/       authMiddleware, requireRole, requireVisitor, rateLimiter, requestLogger, errorHandler
  lib/              validate.ts, validatePassword.ts, logger.ts, checkinQr.ts, visitorToken.ts, prisma.ts
  types/            express.d.ts (req.user), tipe API
prisma/             schema.prisma, migrations/, seed.ts
```

Controller dan service tidak boleh menjalankan query; semua query ada di `models/`.

## API

Respons selalu `{ "success": true, "data": ... }` atau `{ "success": false, "message": "..." }` dengan status code yang sesuai (200, 201, 400, 401, 403, 404, 409, 429, 502).

| Endpoint | Akses | Keterangan |
|---|---|---|
| `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`, `POST /api/auth/forgot-password` | login dibatasi rate limiter | Sesi cookie `mpu_session` (httpOnly, ditandatangani) |
| `GET /api/dashboard`, `/visitor-trend`, `/booking-status` | login | Ringkasan dashboard |
| `GET /api/visitors/today` | login | Check-in hari ini |
| `GET /api/visitors/checkin-qr` | login | Isi QR check-in hari ini (ditampilkan di halaman Pengunjung Hari Ini) |
| `/api/accounts/visitors` | login | Daftar, ubah, hapus akun pengunjung |
| `/api/accounts/admins` | **Super Admin** | Daftar, tambah, ubah, hapus admin (email dobel 409) |
| `/api/bookings` | login | Daftar booking dan ubah status |
| `/api/collections` | login | CRUD koleksi (file lama di Vercel Blob ikut dihapus) |
| `POST /api/uploads` | login | multipart `file` + `kind` (`foto` JPG/PNG dan `dokumen` PDF, masing-masing maks 4 MB (batas request Vercel 4,5 MB)), mengembalikan URL Vercel Blob |

### API publik untuk aplikasi mobile (`/api/public`, tanpa cookie admin)

Kontrak lengkap, contoh, dan kode status ada di [`docs/KONTRAK-API-PUBLIK.md`](docs/KONTRAK-API-PUBLIK.md).

| Endpoint | Keterangan |
|---|---|
| `GET /api/public/collections`, `/collections/:id` | Koleksi |
| `POST /api/public/bookings`, `GET /api/public/bookings?ids=` | Ajukan dan baca status booking |
| `POST /api/public/visits` | Check-in; menerima `qrToken` (QR harian bertanda tangan) dan token login pengunjung |
| `POST /api/public/auth/register`, `/auth/login` | Daftar dan masuk dengan email, kode OTP 6 digit dikirim ke email |
| `POST /api/public/auth/verify-otp` | Verifikasi OTP, mengembalikan token `Authorization: Bearer` |
| `GET /api/public/auth/me` | Akun pengunjung yang login |

Variabel tambahan: `CHECKIN_QR_REQUIRED`, `CHECKIN_QR_SECRET`, `RESEND_API_KEY`, `MAIL_FROM`, `OTP_DEV_ECHO` (lihat `.env.example`).
