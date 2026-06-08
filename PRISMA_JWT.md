Kalau Anda membuat **Express + Prisma + JWT Authentication**, struktur folder yang sederhana dan mudah dipelajari bisa seperti ini:

```text
project/
│
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── src/
│   │
│   ├── config/
│   │   └── prisma.js
│   │
│   ├── controllers/
│   │   └── auth.controller.js
│   │
│   ├── services/
│   │   └── auth.service.js
│   │
│   ├── middlewares/
│   │   ├── auth.middleware.js
│   │   └── error.middleware.js
│   │
│   ├── routes/
│   │   └── auth.routes.js
│   │
│   ├── utils/
│   │   ├── jwt.js
│   │   └── response.js
│   │
│   ├── app.js
│   └── server.js
│
├── .env
├── package.json
└── package-lock.json
```

---

# Fungsi Tiap Folder

## prisma/

Berisi schema dan migration.

```text
prisma/
├── schema.prisma
└── migrations/
```

Contoh model User Anda:

```prisma
model User {
  id             Int    @id @default(autoincrement())
  nama_lengkap   String
  email          String @unique
  password       String?
  role           UserRole @default(guest)

  @@map("users")
}
```

---

## config/prisma.js

Satu koneksi Prisma untuk seluruh aplikasi.

```js id="8qsrch"
const { PrismaClient } = require("@prisma/client");

const prisma = new PrismaClient();

module.exports = prisma;
```

---

## routes/

Menentukan endpoint.

```text
routes/
└── auth.routes.js
```

Contoh:

```js id="m9c0eo"
const express = require("express");
const router = express.Router();

const authController = require("../controllers/auth.controller");

router.post("/register", authController.register);
router.post("/login", authController.login);

module.exports = router;
```

---

## controllers/

Menerima request dari user.

```text
controllers/
└── auth.controller.js
```

Contoh:

```js id="udn14n"
const authService = require("../services/auth.service");

exports.register = async (req, res) => {
  const result = await authService.register(req.body);

  res.json(result);
};
```

Controller sebaiknya tipis.

Jangan isi logika database di sini.

---

## services/

Semua logika bisnis.

```text
services/
└── auth.service.js
```

Contoh:

```js id="h6h8s5"
const prisma = require("../config/prisma");
const bcrypt = require("bcrypt");

exports.register = async (data) => {

  const hashedPassword = await bcrypt.hash(
    data.password,
    10
  );

  const user = await prisma.user.create({
    data: {
      nama_lengkap: data.nama_lengkap,
      email: data.email,
      password: hashedPassword
    }
  });

  return user;
};
```

Di sinilah Prisma biasanya digunakan.

---

## utils/jwt.js

Membuat token JWT.

```js id="eqf6ti"
const jwt = require("jsonwebtoken");

exports.generateToken = (payload) => {
  return jwt.sign(
    payload,
    process.env.JWT_SECRET,
    {
      expiresIn: "1d"
    }
  );
};
```

---

## Login JWT

Contoh service login:

```js id="2swwpr"
const prisma = require("../config/prisma");
const bcrypt = require("bcrypt");
const { generateToken } = require("../utils/jwt");

exports.login = async (data) => {

  const user = await prisma.user.findUnique({
    where: {
      email: data.email
    }
  });

  if (!user) {
    throw new Error("User tidak ditemukan");
  }

  const match = await bcrypt.compare(
    data.password,
    user.password
  );

  if (!match) {
    throw new Error("Password salah");
  }

  const token = generateToken({
    id: user.id,
    role: user.role
  });

  return {
    token
  };
};
```

---

## middlewares/auth.middleware.js

Melindungi route.

```js id="s0s4a4"
const jwt = require("jsonwebtoken");

module.exports = (req, res, next) => {

  const authHeader = req.headers.authorization;

  if (!authHeader) {
    return res.status(401).json({
      message: "Unauthorized"
    });
  }

  const token = authHeader.split(" ")[1];

  const decoded = jwt.verify(
    token,
    process.env.JWT_SECRET
  );

  req.user = decoded;

  next();
};
```

---

## Route yang Diproteksi

```js id="h6gfdx"
router.get(
  "/profile",
  authMiddleware,
  async (req, res) => {

    res.json({
      user: req.user
    });

  }
);
```

Request:

```http
GET /profile
Authorization: Bearer TOKEN_JWT
```

---

## app.js

Menampung semua route.

```js id="rv7ygt"
const express = require("express");

const authRoutes = require("./routes/auth.routes");

const app = express();

app.use(express.json());

app.use("/api/auth", authRoutes);

module.exports = app;
```

---

## server.js

Menjalankan server.

```js id="1n5pbg"
require("dotenv").config();

const app = require("./app");

app.listen(3000, () => {
  console.log("Server running");
});
```

---

# Alur Request

Misalnya user login:

```text
POST /api/auth/login
        │
        ▼
auth.routes.js
        │
        ▼
auth.controller.js
        │
        ▼
auth.service.js
        │
        ▼
Prisma
        │
        ▼
MySQL
```

Ini pola yang cukup umum dipakai di project Express yang sudah terstruktur.

---

# Package yang Biasanya Dipasang

```bash
npm install express
npm install prisma
npm install @prisma/client
npm install bcrypt
npm install jsonwebtoken
npm install dotenv
npm install cors
npm install morgan
```

Untuk development:

```bash
npm install -D nodemon
```

Script di `package.json`:

```json
{
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js"
  }
}
```

Untuk tahap awal, fokus dulu membuat 3 endpoint:

1. `POST /register`
2. `POST /login`
3. `GET /profile` (dengan JWT)

Setelah ketiganya berjalan, Anda sudah memahami sekitar 70–80% dasar penggunaan Prisma dalam sistem autentikasi Express.
