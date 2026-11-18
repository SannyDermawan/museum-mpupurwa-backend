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

```
src/
  app.ts            Express, CORS, JSON, pemasangan router
  server.ts         menjalankan server
  config.ts         membaca .env
  routes/           auth, dashboard, visitors, accounts, bookings, collections, uploads
  controllers/      baca request, validasi, panggil service, kirim respons
  services/         logika bisnis dan query Prisma
  middleware/       authMiddleware, requireRole, rateLimiter, errorHandler
  lib/              prisma.ts, validate.ts, validatePassword.ts
  types/            express.d.ts (req.user), tipe API
prisma/             schema.prisma, migrations/, seed.ts
```

Controller tidak boleh berisi query database; semua query ada di `services/`.

## API

Respons selalu `{ "success": true, "data": ... }` atau `{ "success": false, "message": "..." }` dengan status code yang sesuai (200, 201, 400, 401, 403, 404, 409, 429, 502).

| Endpoint | Akses | Keterangan |
|---|---|---|
| `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`, `POST /api/auth/forgot-password` | login dibatasi rate limiter | Sesi cookie `mpu_session` (httpOnly, ditandatangani) |
| `GET /api/dashboard`, `/visitor-trend`, `/booking-status` | login | Ringkasan dashboard |
| `GET /api/visitors/today` | login | Check-in hari ini |
| `/api/accounts/visitors` | login | Daftar, ubah, hapus akun pengunjung |
| `/api/accounts/admins` | **Super Admin** | Daftar, tambah, ubah, hapus admin (email dobel 409) |
| `/api/bookings` | login | Daftar booking dan ubah status |
| `/api/collections` | login | CRUD koleksi (file lama di Vercel Blob ikut dihapus) |
| `POST /api/uploads` | login | multipart `file` + `kind` (`foto` JPG/PNG maks 5 MB, `dokumen` PDF maks 10 MB), mengembalikan URL Vercel Blob |
