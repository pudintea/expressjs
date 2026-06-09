# PRISMA 7 dan Express JS dan MySQL/MariaDB

Karena kita menggunakan **Prisma versi 7** (versi terbaru di tahun 2026), kita akan mengikuti standar arsitektur terbarunya secara ketat dari awal. Mari kita bangun bersama langkah demi langkah.

---

## Langkah 1: Inisialisasi Folder Proyek Baru

Buka terminal Anda, buat folder baru yang benar-benar bersih, dan masuk ke dalamnya:

```bash
mkdir express-prisma7-mysql
cd express-prisma7-mysql

# Inisialisasi proyek Node.js secara default
npm init -y

```

---

## Langkah 2: Konfigurasi ESM (ECMAScript Modules)

Prisma 7 mewajibkan penggunaan fitur modern JavaScript (`import` / `export`). Buka file `package.json` yang baru saja terbentuk, lalu tambahkan properti `"type": "module"` dan *script* `"dev"` untuk menjalankan server:

```json
{
  "name": "express-prisma7-mysql",
  "version": "1.0.0",
  "type": "module",
  "main": "index.js",
  "scripts": {
    "dev": "nodemon index.js"
  }
}

```

---

## Langkah 3: Instalasi Dependencies Resmi Prisma 7

Sekarang, ketik perintah ini di terminal Anda untuk menginstal paket-paket yang dibutuhkan:

```bash
# 1. Instal dependencies utama (Express, Prisma Client, Driver Adapter, dan Dotenv)
npm install express @prisma/client @prisma/adapter-mariadb dotenv

# 2. Instal dependencies development (Prisma CLI dan Nodemon)
npm install prisma nodemon --save-dev

```

---

## Langkah 4: Inisialisasi Prisma ORM v7

Jalankan perintah khusus Prisma 7 ini untuk membuat struktur folder database:

```bash
npx prisma init --datasource-provider mysql --output ../generated/prisma

```

Perintah di atas akan otomatis menghasilkan beberapa file penting secara bersamaan di dalam proyek Anda:

1. Folder `prisma` berisi file `schema.prisma`.
2. File konfigurasi baru bernama `prisma.config.ts`.
3. File `.env` di luar folder.

---

## Langkah 5: Atur Kredensial Database di File `.env`

Buka file `.env` yang berada di root folder proyek Anda. Masukkan informasi koneksi ke MySQL/MariaDB lokal Anda secara detail seperti berikut (sesuaikan username dan password Anda):

```env
DATABASE_URL="mysql://root:password_kamu@localhost:3306/db_express_prisma7"
DATABASE_USER="root"
DATABASE_PASSWORD="password_kamu"
DATABASE_NAME="db_express_prisma7"
DATABASE_HOST="localhost"
DATABASE_PORT=3306

```

---

## Langkah 6: Definisikan Model User di `prisma/schema.prisma`

Buka file `prisma/schema.prisma`. Pastikan isinya sudah sesuai dengan standar Prisma 7, lalu tambahkan model `User`:

```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "mysql"
}

// Model data User kita
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
}

```

---

## Langkah 7: Sinkronisasi dan Generate Client

Sekarang jalankan perintah ini di terminal agar Prisma membuat tabel di MySQL dan membuat fungsi-fungsi *query* otomatis di folder `generated`:

```bash
# 1. Buat tabel ke dalam MySQL
npx prisma migrate dev --name init_users

# 2. Buat Prisma Client internal
npx prisma generate

```

---

## Langkah 8: Buat Instance Koneksi Adapter (`lib/prisma.js`)

Buat folder baru bernama `lib` di dalam proyek Anda, lalu buat file bernama `prisma.js` di dalam folder tersebut. File ini bertugas merakit koneksi menggunakan adapter resmi Prisma 7:

```javascript
// lib/prisma.js
import "dotenv/config";
import { PrismaMariaDb } from "@prisma/adapter-mariadb";
//import { PrismaClient } from "../generated/prisma/client.js"; // Memanggil dari folder custom generated
import { PrismaClient } from "../generated/prisma/client.ts";

// Konfigurasi adapter database lokal sesuai aturan Prisma 7
const adapter = new PrismaMariaDb({
  host: process.env.DATABASE_HOST,
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  connectionLimit: 5,
});

// Masukkan adapter ke dalam konstruktor PrismaClient
const prisma = new PrismaClient({ adapter });

export { prisma };

```

---

## Langkah 9: Bangun REST API Express di File `index.js`

Buat file utama bernama `index.js` di root folder proyek Anda (sejajar dengan `package.json`), lalu isi dengan kode REST API berikut:

```javascript
// index.js
import express from 'express';
import { prisma } from './lib/prisma.js'; // Impor instance prisma yang aman dari lib

const app = express();
app.use(express.json());

const PORT = 3000;

// 1. POST: Membuat User Baru
app.post('/users', async (req, res) => {
  const { name, email } = req.body;
  try {
    const newUser = await prisma.user.create({
      data: { name, email },
    });
    res.status(201).json(newUser);
  } catch (error) {
    res.status(400).json({ error: "Gagal membuat user. Email mungkin sudah terdaftar." });
  }
});

// 2. GET: Mengambil Semua Data User
app.get('/users', async (req, res) => {
  try {
    const users = await prisma.user.findMany();
    res.status(200).json(users);
  } catch (error) {
    res.status(500).json({ error: "Gagal mengambil data dari database." });
  }
});

// 3. DELETE: Menghapus User Berdasarkan ID
app.delete('/users/:id', async (req, res) => {
  const { id } = req.params;
  try {
    await prisma.user.delete({
      where: { id: parseInt(id) },
    });
    res.status(200).json({ message: `User dengan ID ${id} berhasil dihapus.` });
  } catch (error) {
    res.status(400).json({ error: "User tidak ditemukan atau gagal dihapus." });
  }
});

// Menghidupkan Server Express
app.listen(PORT, () => {
  console.log(`🚀 Server Express & Prisma 7 berjalan sukses di http://localhost:${PORT}`);
});

```

---

## Langkah 10: Jalankan Aplikasi Anda!

Sekarang, saatnya melihat hasilnya. Jalankan perintah ini di terminal:

```bash
npm run dev

```

Selamat! Struktur proyek Anda sekarang sudah 100% menggunakan arsitektur paling modern dan standar resmi dari Prisma 7. Server Anda akan menyala dengan mulus dan siap menerima request CRUD Anda tanpa ada lagi pesan error modul maupun datasource. Silakan dicoba!

## Pudin Saepudin
