# Belajar Prisma 6 dan ExpressJs
Kalau tujuan Anda **belajar konsep REST API + MariaDB + Prisma**, saya setuju bahwa **Prisma 6 lebih ramah untuk pemula** dibanding Prisma 7 saat ini.

Alasannya:

* Banyak tutorial dan dokumentasi komunitas masih menggunakan Prisma 6.
* Konfigurasi lebih sederhana.
* Tidak perlu memahami perubahan arsitektur Prisma 7 di awal.
* Fokus belajar bisa ke CRUD, relasi tabel, dan REST API, bukan konfigurasi tooling.

Node.js 24 tetap bisa digunakan dengan Prisma 6 selama menggunakan versi Prisma 6 yang masih kompatibel.

---

# 1. Install MariaDB di Ubuntu

Update package:

```bash
sudo apt update
```

Install MariaDB:

```bash
sudo apt install mariadb-server -y
```

Cek service:

```bash
sudo systemctl status mariadb
```

Masuk ke MariaDB:

```bash
sudo mysql
```

Buat database:

```sql
CREATE DATABASE belajar_prisma;
```

Buat user khusus (lebih baik daripada memakai root):

```sql
CREATE USER 'belajar'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON belajar_prisma.* TO 'belajar'@'localhost';
FLUSH PRIVILEGES;
```

Keluar:

```sql
EXIT;
```

---

# 2. Buat Project Baru

```bash
mkdir belajar-api
cd belajar-api
```

Inisialisasi npm:

```bash
npm init -y
```

---

# 3. Install Express

```bash
npm install express
```

Install nodemon:

```bash
npm install -D nodemon
```

Tambahkan script di `package.json`:

```json
{
  "scripts": {
    "dev": "nodemon src/server.js"
  }
}
```

---

# 4. Install Prisma 6

Untuk memastikan menggunakan Prisma 6:

```bash
npm install prisma@6 @prisma/client@6
```

Cek versi:

```bash
npx prisma -v
```

Contoh output:

```text
Prisma CLI : 6.x.x
@prisma/client : 6.x.x
```

---

# 5. Inisialisasi Prisma

Jalankan:

```bash
npx prisma init
```

Struktur yang dibuat:

```text
.
├── prisma
│   └── schema.prisma
├── .env
└── package.json
```

---

# 6. Konfigurasi Koneksi MariaDB

Edit `.env`

```env
DATABASE_URL="mysql://belajar:password123@localhost:3306/belajar_prisma"
```

Format umum:

```text
mysql://USER:PASSWORD@HOST:PORT/NAMA_DATABASE
```

---

# 7. Buat Schema Pertama

Buka:

```text
prisma/schema.prisma
```

Isi:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
}
```

---

# 8. Buat Tabel Database

Jalankan:

```bash
npx prisma migrate dev --name init
```

Prisma akan:

1. Membuat tabel User
2. Menyimpan migration
3. Generate Prisma Client

Struktur akan menjadi:

```text
prisma/
├── migrations/
└── schema.prisma
```

---

# 9. Lihat Database

Masuk ke MariaDB:

```bash
mysql -u belajar -p
```

Pilih database:

```sql
USE belajar_prisma;
```

Lihat tabel:

```sql
SHOW TABLES;
```

Lihat struktur:

```sql
DESCRIBE User;
```

---

# 10. Prisma Studio (Sangat Berguna untuk Pemula)

Prisma menyediakan GUI untuk melihat data.

Jalankan:

```bash
npx prisma studio
```

Biasanya terbuka di:

```text
http://localhost:5555
```

Ini seperti phpMyAdmin versi Prisma.

---

# 11. Membuat Express + Prisma

Struktur:

```text
src/
└── server.js
```

Isi `src/server.js`:

```javascript
const express = require("express");
const { PrismaClient } = require("@prisma/client");

const prisma = new PrismaClient();
const app = express();

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    message: "API berjalan"
  });
});

app.listen(3000, () => {
  console.log("Server berjalan di port 3000");
});
```

Jalankan:

```bash
npm run dev
```

---

# 12. CRUD Pertama dengan Prisma

### Create User

```javascript
app.post("/users", async (req, res) => {
  const { name, email } = req.body;

  const user = await prisma.user.create({
    data: {
      name,
      email
    }
  });

  res.status(201).json(user);
});
```

Test:

```bash
curl -X POST http://localhost:3000/users \
-H "Content-Type: application/json" \
-d '{"name":"Budi","email":"budi@gmail.com"}'
```

---

### Read Semua User

```javascript
app.get("/users", async (req, res) => {
  const users = await prisma.user.findMany();

  res.json(users);
});
```

---

### Read Berdasarkan ID

```javascript
app.get("/users/:id", async (req, res) => {
  const id = Number(req.params.id);

  const user = await prisma.user.findUnique({
    where: {
      id
    }
  });

  res.json(user);
});
```

---

### Update User

```javascript
app.put("/users/:id", async (req, res) => {
  const id = Number(req.params.id);

  const user = await prisma.user.update({
    where: {
      id
    },
    data: req.body
  });

  res.json(user);
});
```

---

### Delete User

```javascript
app.delete("/users/:id", async (req, res) => {
  const id = Number(req.params.id);

  await prisma.user.delete({
    where: {
      id
    }
  });

  res.json({
    message: "User dihapus"
  });
});
```

---

## Roadmap Belajar yang Saya Sarankan

Urutan yang akan membuat Anda cepat paham:

1. **Express dasar**

   * Routing
   * Middleware
   * Request & Response

2. **MariaDB dasar**

   * CREATE TABLE
   * INSERT
   * SELECT
   * UPDATE
   * DELETE

3. **Prisma dasar**

   * Schema
   * Migration
   * Prisma Studio
   * CRUD

4. **Relasi database**

   * User → Post (One-to-Many)
   * User → Profile (One-to-One)

5. **REST API profesional**

   * Controller
   * Route
   * Service
   * Validation
   * Error handling

6. **Authentication**

   * JWT
   * Login/Register

7. **Deployment**

Untuk tahap belajar, saya sarankan kita lanjut membuat proyek **Blog API sederhana** (User dan Post) karena di situ Anda akan belajar relasi Prisma, yang merupakan bagian paling penting setelah CRUD dasar.

## Pudin Saepudin
