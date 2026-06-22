# Prisma 6 Blog API Sederhana
Bagus. Memisahkan **server**, **app**, **routes**, dan **controllers** memang praktik yang lebih rapi dan mendekati struktur proyek profesional.

Untuk belajar, kita akan membuat **Blog API sederhana** dengan:

* User
* Post
* Relasi One-to-Many (1 User punya banyak Post)

Contoh:

```text
Budi
├── Belajar Prisma
├── Belajar Express

Andi
├── REST API Pemula
```

---

# Struktur Project

Saya sarankan seperti ini:

```text
belajar-api/
│
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── src/
│   ├── app.js
│   ├── server.js
│
│   ├── routes/
│   │   ├── userRoutes.js
│   │   └── postRoutes.js
│
│   ├── controllers/
│   │   ├── userController.js
│   │   └── postController.js
│
│   └── lib/
│       └── prisma.js
│
├── .env
└── package.json
```

---

# Langkah 1: Schema Prisma

Edit `prisma/schema.prisma`

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

  posts Post[]
}

model Post {
  id      Int    @id @default(autoincrement())
  title   String
  content String?

  authorId Int

  author User @relation(fields: [authorId], references: [id])
}
```

---

# Langkah 2: Jalankan Migration

```bash
npx prisma migrate dev --name add_blog
```

Lalu buka Prisma Studio:

```bash
npx prisma studio
```

Untuk melihat tabel yang dibuat.

---

# Langkah 3: Buat Prisma Client Terpusat

File:

```text
src/lib/prisma.js
```

Isi:

```javascript
const { PrismaClient } = require("@prisma/client");

const prisma = new PrismaClient();

module.exports = prisma;
```

Tujuannya agar semua controller memakai instance yang sama.

---

# Langkah 4: Membuat App

File:

```text
src/app.js
```

Isi:

```javascript
const express = require("express");

const userRoutes = require("./routes/userRoutes");
const postRoutes = require("./routes/postRoutes");

const app = express();

app.use(express.json());

app.use("/users", userRoutes);
app.use("/posts", postRoutes);

module.exports = app;
```

---

# Langkah 5: Membuat Server

File:

```text
src/server.js
```

Isi:

```javascript
const app = require("./app");

const PORT = 3000;

app.listen(PORT, () => {
  console.log(`Server berjalan di port ${PORT}`);
});
```

Perbedaannya:

* `app.js` → konfigurasi Express
* `server.js` → menjalankan server

---

# Langkah 6: Membuat Controller User

File:

```text
src/controllers/userController.js
```

Isi:

```javascript
const prisma = require("../lib/prisma");

exports.getUsers = async (req, res) => {
  const users = await prisma.user.findMany({
    include: {
      posts: true
    }
  });

  res.json(users);
};

exports.createUser = async (req, res) => {
  const { name, email } = req.body;

  const user = await prisma.user.create({
    data: {
      name,
      email
    }
  });

  res.status(201).json(user);
};
```

---

# Langkah 7: Membuat User Routes

File:

```text
src/routes/userRoutes.js
```

Isi:

```javascript
const express = require("express");

const {
  getUsers,
  createUser
} = require("../controllers/userController");

const router = express.Router();

router.get("/", getUsers);
router.post("/", createUser);

module.exports = router;
```

---

# Langkah 8: Membuat Post Controller

File:

```text
src/controllers/postController.js
```

Isi:

```javascript
const prisma = require("../lib/prisma");

exports.getPosts = async (req, res) => {
  const posts = await prisma.post.findMany({
    include: {
      author: true
    }
  });

  res.json(posts);
};

exports.createPost = async (req, res) => {
  const {
    title,
    content,
    authorId
  } = req.body;

  const post = await prisma.post.create({
    data: {
      title,
      content,
      authorId
    }
  });

  res.status(201).json(post);
};
```

---

# Langkah 9: Membuat Post Routes

File:

```text
src/routes/postRoutes.js
```

Isi:

```javascript
const express = require("express");

const {
  getPosts,
  createPost
} = require("../controllers/postController");

const router = express.Router();

router.get("/", getPosts);
router.post("/", createPost);

module.exports = router;
```

---

# Langkah 10: Menjalankan

```bash
npm run dev
```

---

# Membuat User

Request:

```http
POST /users
```

Body:

```json
{
  "name": "Budi",
  "email": "budi@gmail.com"
}
```

Response:

```json
{
  "id": 1,
  "name": "Budi",
  "email": "budi@gmail.com"
}
```

---

# Membuat Post

Request:

```http
POST /posts
```

Body:

```json
{
  "title": "Belajar Prisma",
  "content": "Artikel pertama",
  "authorId": 1
}
```

---

# Melihat Semua Post

Request:

```http
GET /posts
```

Response:

```json
[
  {
    "id": 1,
    "title": "Belajar Prisma",
    "content": "Artikel pertama",
    "authorId": 1,
    "author": {
      "id": 1,
      "name": "Budi",
      "email": "budi@gmail.com"
    }
  }
]
```

---

## Memahami `include`

Ini salah satu fitur Prisma yang sangat penting.

Tanpa include:

```javascript
const posts = await prisma.post.findMany();
```

Hasil:

```json
[
  {
    "id": 1,
    "title": "Belajar Prisma",
    "authorId": 1
  }
]
```

Dengan include:

```javascript
const posts = await prisma.post.findMany({
  include: {
    author: true
  }
});
```

Prisma otomatis melakukan join relasi dan mengembalikan data author.

---

## Langkah Berikutnya

Setelah proyek ini berjalan, saya sarankan menambahkan:

1. Error handling global
2. Validasi request (misalnya email wajib valid)
3. Update dan delete user/post
4. Pagination (`?page=1&limit=10`)
5. JWT login/register
6. Hash password dengan bcrypt

Itu akan membawa proyek ini dari level "belajar CRUD" ke struktur yang cukup dekat dengan API produksi.
