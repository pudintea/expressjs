Tentu. Karena Anda masih awam, saya akan ajarkan dari dasar sampai paham hubungan antara:

1. SQL Database (MySQL)
2. Prisma Schema
3. Migration
4. Query Data (CRUD)

Kita gunakan tabel `users` yang Anda berikan sebagai contoh nyata.

---

# 1. Apa itu Prisma?

Prisma adalah ORM untuk Node.js.

Tanpa Prisma:

```js
const [rows] = await connection.query(
  "SELECT * FROM users WHERE email = ?",
  [email]
);
```

Dengan Prisma:

```js
const user = await prisma.user.findUnique({
  where: {
    email: email
  }
});
```

Lebih mudah dibaca dan lebih aman.

---

# 2. Install Prisma

Misalnya project Express.js.

Install:

```bash
npm install prisma --save-dev
npm install @prisma/client
```

Lalu inisialisasi:

```bash
npx prisma init
```

Akan muncul folder:

```text
prisma/
 └─ schema.prisma

.env
```

---

# 3. Koneksi Database

Misalnya database MySQL:

```env
DATABASE_URL="mysql://root:password@localhost:3306/db_kampus"
```

File `.env`

---

# 4. Ubah SQL Menjadi Prisma Schema

SQL Anda:

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nama_lengkap VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    no_hp VARCHAR(20),
    password VARCHAR(255) NULL,
    google_id VARCHAR(100) NULL UNIQUE,
    login_provider ENUM('local', 'google') NOT NULL DEFAULT 'local',
    avatar_url VARCHAR(255) NULL,
    role ENUM('guest', 'admin') NOT NULL DEFAULT 'guest',
    token TEXT NULL,
    active ENUM('active', 'inactive') NOT NULL DEFAULT 'inactive',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

Di Prisma menjadi:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

enum LoginProvider {
  local
  google
}

enum UserRole {
  guest
  admin
}

enum UserStatus {
  active
  inactive
}

model User {
  id             Int             @id @default(autoincrement())
  nama_lengkap   String          @db.VarChar(100)
  email          String          @unique @db.VarChar(100)
  no_hp          String?         @db.VarChar(20)

  password       String?         @db.VarChar(255)

  google_id      String?         @unique @db.VarChar(100)

  login_provider LoginProvider   @default(local)

  avatar_url     String?         @db.VarChar(255)

  role           UserRole        @default(guest)

  token          String?         @db.Text

  active         UserStatus      @default(inactive)

  created_at     DateTime        @default(now())
  updated_at     DateTime        @updatedAt

  @@map("users")
}
```

---

# 5. Apa Itu Migration?

Migration adalah cara Prisma membuat atau mengubah tabel otomatis.

Misalnya:

Awalnya:

```prisma
model User {
  id    Int @id @default(autoincrement())
  email String
}
```

Lalu Anda tambahkan:

```prisma
nama_lengkap String
```

Prisma akan membuat SQL otomatis:

```sql
ALTER TABLE users
ADD nama_lengkap VARCHAR(191);
```

Anda tidak perlu menulis SQL manual.

---

# 6. Membuat Migration Pertama

Setelah schema selesai:

```bash
npx prisma migrate dev --name init_users
```

Contoh:

```bash
npx prisma migrate dev --name init_users
```

Output:

```text
prisma/
 ├─ migrations/
 │   └─ 20260608120000_init_users
 │       └─ migration.sql
```

Prisma akan:

✅ Membuat tabel

✅ Menyimpan history migration

✅ Update database

---

# 7. Generate Prisma Client

Biasanya otomatis, tapi bisa manual:

```bash
npx prisma generate
```

---

# 8. Membuat File Prisma Client

Misalnya:

```js
// lib/prisma.js

const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

module.exports = prisma;
```

---

# 9. Insert Data

Tambah user:

```js
const prisma = require('./lib/prisma');

await prisma.user.create({
  data: {
    nama_lengkap: "Budi Santoso",
    email: "budi@gmail.com",
    password: "123456"
  }
});
```

SQL yang terjadi:

```sql
INSERT INTO users (...)
VALUES (...);
```

---

# 10. Ambil Semua User

```js
const users = await prisma.user.findMany();

console.log(users);
```

Sama dengan:

```sql
SELECT * FROM users;
```

---

# 11. Cari Berdasarkan Email

```js
const user = await prisma.user.findUnique({
  where: {
    email: "budi@gmail.com"
  }
});
```

Sama dengan:

```sql
SELECT *
FROM users
WHERE email='budi@gmail.com';
```

---

# 12. Update User

```js
await prisma.user.update({
  where: {
    id: 1
  },
  data: {
    nama_lengkap: "Budi Update"
  }
});
```

Sama dengan:

```sql
UPDATE users
SET nama_lengkap='Budi Update'
WHERE id=1;
```

---

# 13. Hapus User

```js
await prisma.user.delete({
  where: {
    id: 1
  }
});
```

Sama dengan:

```sql
DELETE FROM users
WHERE id=1;
```

---

# 14. Kalau Sudah Punya Database?

Misalnya tabel sudah ada dan dibuat manual menggunakan SQL Anda.

Jangan pakai migrate dulu.

Gunakan:

```bash
npx prisma db pull
```

Prisma akan membaca database dan membuat schema otomatis.

Contoh:

```bash
npx prisma db pull
```

Output:

```prisma
model users {
 ...
}
```

Lalu generate client:

```bash
npx prisma generate
```

Ini disebut **introspection**.

---

# 15. Kapan Pakai `db pull` dan Kapan Pakai `migrate`?

### Kasus A — Database sudah ada

Gunakan:

```bash
npx prisma db pull
```

karena Prisma mengikuti database.

---

### Kasus B — Prisma yang mengelola database

Gunakan:

```bash
npx prisma migrate dev
```

karena database mengikuti Prisma.

---

# Alur yang Saya Sarankan Untuk Pemula

Jika Anda baru belajar:

### Langkah 1

Buat database kosong:

```sql
CREATE DATABASE belajar_prisma;
```

### Langkah 2

Buat schema Prisma dari contoh di atas.

### Langkah 3

Jalankan:

```bash
npx prisma migrate dev --name init_users
```

### Langkah 4

Jalankan:

```bash
npx prisma studio
```

Prisma Studio adalah GUI untuk melihat data seperti phpMyAdmin.

### Langkah 5

Coba:

```js
create()
findMany()
update()
delete()
```

sampai paham hubungan antara Prisma dan SQL.

---

Saran saya: sebelum masuk ke relasi (One-to-Many, Many-to-Many), pastikan Anda benar-benar paham **migration**, **schema.prisma**, dan **CRUD dasar** pada tabel `users`. Setelah itu saya bisa ajarkan tahap berikutnya: membuat sistem login Google + JWT menggunakan Prisma berdasarkan struktur tabel yang Anda punya.
