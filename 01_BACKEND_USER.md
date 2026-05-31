Bagus. Saya akan buatkan versi yang **siap dipakai** untuk CRUD User berdasarkan kode awal Anda, dengan arsitektur:

```text
Route
 ↓
Validator
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Database
```

---

# Struktur Folder

```text
src/
│
├── modules/
│   │
│   ├── users/
│   │   ├── user.controller.js
│   │   ├── user.service.js
│   │   ├── user.repository.js
│   │   ├── user.validator.js
│   │   ├── user.model.js
│   │   └── user.routes.js
│
├── config/
│   ├── db.js
│   └── env.js
│
├── middlewares/
│   ├── validate.middleware.js
│   ├── error.middleware.js
│   └── notfound.middleware.js
│
├── utils/
│   ├── response.js
│   └── bcrypt.js
│
├── exceptions/
│   ├── AppError.js
│   ├── NotFoundError.js
│   ├── BadRequestError.js
│   └── UnauthorizedError.js
│
├── routes.js
├── app.js
└── server.js
```

---

# config/env.js

```js
require("dotenv").config();

module.exports = {
  PORT: process.env.PORT || 3000,

  DB_HOST: process.env.DB_HOST,
  DB_USER: process.env.DB_USER,
  DB_PASSWORD: process.env.DB_PASSWORD,
  DB_NAME: process.env.DB_NAME,
};
```

---

# config/db.js

```js
const mysql = require("mysql2/promise");
const env = require("./env");

const pool = mysql.createPool({
  host: env.DB_HOST,
  user: env.DB_USER,
  password: env.DB_PASSWORD,
  database: env.DB_NAME,
});

module.exports = pool;
```

---

# exceptions/AppError.js

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);

    this.statusCode = statusCode;
  }
}

module.exports = AppError;
```

---

# exceptions/NotFoundError.js

```js
const AppError = require("./AppError");

class NotFoundError extends AppError {
  constructor(message = "Data tidak ditemukan") {
    super(message, 404);
  }
}

module.exports = NotFoundError;
```

---

# exceptions/BadRequestError.js

```js
const AppError = require("./AppError");

class BadRequestError extends AppError {
  constructor(message = "Bad Request") {
    super(message, 400);
  }
}

module.exports = BadRequestError;
```

---

# exceptions/UnauthorizedError.js

```js
const AppError = require("./AppError");

class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") {
    super(message, 401);
  }
}

module.exports = UnauthorizedError;
```

---

# utils/bcrypt.js

```js
const bcrypt = require("bcrypt");

exports.hash = async (password) => {
  return await bcrypt.hash(password, 10);
};

exports.compare = async (
  password,
  hashPassword
) => {
  return await bcrypt.compare(
    password,
    hashPassword
  );
};
```

---

# utils/response.js

```js
exports.success = (
  res,
  data = null,
  message = "Success",
  status = 200
) => {
  return res.status(status).json({
    success: true,
    message,
    data,
  });
};

exports.error = (
  res,
  message,
  status = 500
) => {
  return res.status(status).json({
    success: false,
    message,
  });
};
```

---

# middlewares/error.middleware.js

```js
module.exports = (
  err,
  req,
  res,
  next
) => {
  console.error(err);

  return res.status(
    err.statusCode || 500
  ).json({
    success: false,
    message:
      err.message ||
      "Internal Server Error",
  });
};
```

---

# middlewares/notfound.middleware.js

```js
module.exports = (
  req,
  res
) => {
  res.status(404).json({
    success: false,
    message: "Route tidak ditemukan",
  });
};
```

---

# middlewares/validate.middleware.js

```js
const {
  validationResult,
} = require("express-validator");

module.exports = (
  req,
  res,
  next
) => {
  const errors =
    validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(422).json({
      success: false,
      errors: errors.array(),
    });
  }

  next();
};
```

---

# modules/users/user.model.js

```js
class User {
  constructor(data) {
    this.id = data.id;
    this.nama_lengkap =
      data.nama_lengkap;
    this.email = data.email;
    this.no_hp = data.no_hp;
    this.created_at =
      data.created_at;
  }
}

module.exports = User;
```

---

# modules/users/user.repository.js

```js
const db = require(
  "../../config/db"
);

class UserRepository {
  async findAll() {
    const [rows] = await db.query(
      `
      SELECT
      id,
      nama_lengkap,
      email,
      no_hp,
      created_at
      FROM users
    `
    );

    return rows;
  }

  async findById(id) {
    const [rows] = await db.query(
      `
      SELECT *
      FROM users
      WHERE id=?
    `,
      [id]
    );

    return rows[0];
  }

  async findByEmail(email) {
    const [rows] = await db.query(
      `
      SELECT *
      FROM users
      WHERE email=?
    `,
      [email]
    );

    return rows[0];
  }

  async create(user) {
    return await db.query(
      `
      INSERT INTO users
      (
        nama_lengkap,
        email,
        no_hp,
        password
      )
      VALUES
      (?,?,?,?)
    `,
      [
        user.nama_lengkap,
        user.email,
        user.no_hp,
        user.password,
      ]
    );
  }

  async update(id, user) {
    return await db.query(
      `
      UPDATE users
      SET
      nama_lengkap=?,
      email=?,
      no_hp=?
      WHERE id=?
    `,
      [
        user.nama_lengkap,
        user.email,
        user.no_hp,
        id,
      ]
    );
  }

  async delete(id) {
    return await db.query(
      `
      DELETE FROM users
      WHERE id=?
    `,
      [id]
    );
  }
}

module.exports =
  new UserRepository();
```

---

# modules/users/user.service.js

```js
const User = require("./user.model");

const repository = require(
  "./user.repository"
);

const bcryptUtil = require(
  "../../utils/bcrypt"
);

const NotFoundError = require(
  "../../exceptions/NotFoundError"
);

const BadRequestError = require(
  "../../exceptions/BadRequestError"
);

class UserService {
  async getAll() {
    return await repository.findAll();
  }

  async getById(id) {
    const user =
      await repository.findById(id);

    if (!user) {
      throw new NotFoundError(
        "User tidak ditemukan"
      );
    }

    return new User(user);
  }

  async create(payload) {
    const existing =
      await repository.findByEmail(
        payload.email
      );

    if (existing) {
      throw new BadRequestError(
        "Email sudah digunakan"
      );
    }

    const password =
      await bcryptUtil.hash(
        payload.password
      );

    await repository.create({
      ...payload,
      password,
    });
  }

  async update(id, payload) {
    await this.getById(id);

    await repository.update(
      id,
      payload
    );
  }

  async delete(id) {
    await this.getById(id);

    await repository.delete(id);
  }
}

module.exports =
  new UserService();
```

---

# modules/users/user.controller.js

```js
const service = require(
  "./user.service"
);

const response = require(
  "../../utils/response"
);

class UserController {
  async index(
    req,
    res,
    next
  ) {
    try {
      const users =
        await service.getAll();

      return response.success(
        res,
        users
      );
    } catch (err) {
      next(err);
    }
  }

  async show(
    req,
    res,
    next
  ) {
    try {
      const user =
        await service.getById(
          req.params.id
        );

      return response.success(
        res,
        user
      );
    } catch (err) {
      next(err);
    }
  }

  async store(
    req,
    res,
    next
  ) {
    try {
      await service.create(
        req.body
      );

      return response.success(
        res,
        null,
        "User berhasil dibuat",
        201
      );
    } catch (err) {
      next(err);
    }
  }

  async update(
    req,
    res,
    next
  ) {
    try {
      await service.update(
        req.params.id,
        req.body
      );

      return response.success(
        res,
        null,
        "User berhasil diupdate"
      );
    } catch (err) {
      next(err);
    }
  }

  async destroy(
    req,
    res,
    next
  ) {
    try {
      await service.delete(
        req.params.id
      );

      return response.success(
        res,
        null,
        "User berhasil dihapus"
      );
    } catch (err) {
      next(err);
    }
  }
}

module.exports =
  new UserController();
```

---

# modules/users/user.validator.js

```js
const { body } = require(
  "express-validator"
);

exports.createUserValidator = [
  body("nama_lengkap")
    .notEmpty()
    .withMessage(
      "Nama lengkap wajib diisi"
    ),

  body("email")
    .isEmail()
    .withMessage(
      "Email tidak valid"
    ),

  body("password")
    .isLength({ min: 6 })
    .withMessage(
      "Password minimal 6 karakter"
    ),
];
```

---

# modules/users/user.routes.js

```js
const express = require(
  "express"
);

const router =
  express.Router();

const controller = require(
  "./user.controller"
);

const {
  createUserValidator,
} = require("./user.validator");

const validate = require(
  "../../middlewares/validate.middleware"
);

router.get(
  "/",
  controller.index
);

router.get(
  "/:id",
  controller.show
);

router.post(
  "/",
  createUserValidator,
  validate,
  controller.store
);

router.put(
  "/:id",
  controller.update
);

router.delete(
  "/:id",
  controller.destroy
);

module.exports = router;
```

---

# routes.js

```js
const express = require(
  "express"
);

const router =
  express.Router();

router.use(
  "/users",
  require(
    "./modules/users/user.routes"
  )
);

module.exports = router;
```

---

# app.js

```js
const express = require(
  "express"
);

const routes = require(
  "./routes"
);

const errorMiddleware = require(
  "./middlewares/error.middleware"
);

const notFoundMiddleware = require(
  "./middlewares/notfound.middleware"
);

const app = express();

app.use(express.json());

app.use("/api", routes);

app.use(notFoundMiddleware);

app.use(errorMiddleware);

module.exports = app;
```

---

# server.js

```js
const app = require("./app");

const env = require(
  "./config/env"
);

app.listen(
  env.PORT,
  () => {
    console.log(
      `Server running on port ${env.PORT}`
    );
  }
);
```

---

# .env

```env
PORT=3000

DB_HOST=localhost
DB_USER=root
DB_PASSWORD=
DB_NAME=express_crud
```

Dengan struktur ini, ketika nanti Anda membuat module lain seperti `auth`, `products`, atau `orders`, Anda tinggal menyalin pola folder `users` dan mengganti logic bisnisnya tanpa mengubah arsitektur aplikasi. Ini membuat codebase tetap konsisten dan mudah dirawat seiring pertumbuhan project.
