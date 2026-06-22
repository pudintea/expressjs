# Response API Helper
Ini pertanyaan yang sangat bagus. Banyak tutorial CRUD berhenti di tahap "data berhasil dikirim", padahal di dunia kerja **konsistensi response API** sangat penting.

Kalau setiap endpoint mengembalikan format berbeda-beda, frontend akan sulit menggunakannya.

Contoh yang **kurang baik**:

`GET /users`

```json
[
  {
    "id": 1,
    "name": "Budi"
  }
]
```

`POST /users`

```json
{
  "id": 1,
  "name": "Budi"
}
```

`DELETE /users/1`

```json
{
  "message": "User dihapus"
}
```

Bentuknya tidak konsisten.

---

# Standar Response yang Saya Sarankan

Untuk proyek Express + Prisma belajar hingga profesional, gunakan format:

```json
{
  "success": true,
  "message": "Pesan operasi",
  "data": {}
}
```

atau

```json
{
  "success": true,
  "message": "Data berhasil diambil",
  "data": []
}
```

Error:

```json
{
  "success": false,
  "message": "User tidak ditemukan",
  "errors": null
}
```

Validation error:

```json
{
  "success": false,
  "message": "Validasi gagal",
  "errors": {
    "email": "Email wajib diisi"
  }
}
```

---

# Membuat Helper Response

Buat file:

```text
src/utils/apiResponse.js
```

```javascript
exports.success = (
  res,
  message,
  data = null,
  statusCode = 200
) => {
  return res.status(statusCode).json({
    success: true,
    message,
    data
  });
};

exports.error = (
  res,
  message,
  errors = null,
  statusCode = 400
) => {
  return res.status(statusCode).json({
    success: false,
    message,
    errors
  });
};
```

---

# Menggunakan di Controller

Sebelum:

```javascript
res.json(users);
```

Sesudah:

```javascript
const response = require("../utils/apiResponse");

response.success(
  res,
  "Data user berhasil diambil",
  users
);
```

Response:

```json
{
  "success": true,
  "message": "Data user berhasil diambil",
  "data": [
    {
      "id": 1,
      "name": "Budi",
      "email": "budi@gmail.com"
    }
  ]
}
```

---

# Contoh Create User

Controller:

```javascript
exports.createUser = async (req, res) => {
  const { name, email } = req.body;

  const user = await prisma.user.create({
    data: {
      name,
      email
    }
  });

  response.success(
    res,
    "User berhasil dibuat",
    user,
    201
  );
};
```

Response:

```json
{
  "success": true,
  "message": "User berhasil dibuat",
  "data": {
    "id": 1,
    "name": "Budi",
    "email": "budi@gmail.com"
  }
}
```

---

# Menangani Data Tidak Ditemukan

```javascript
const user = await prisma.user.findUnique({
  where: {
    id
  }
});

if (!user) {
  return response.error(
    res,
    "User tidak ditemukan",
    null,
    404
  );
}
```

Response:

```json
{
  "success": false,
  "message": "User tidak ditemukan",
  "errors": null
}
```

---

# Tambahkan Meta untuk Pagination

Nanti ketika API berkembang:

```json
{
  "success": true,
  "message": "Data post berhasil diambil",
  "data": [...],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 50,
    "totalPages": 5
  }
}
```

Karena itu saya biasanya membuat helper lebih fleksibel:

```javascript
exports.success = (
  res,
  message,
  data = null,
  statusCode = 200,
  meta = null
) => {
  return res.status(statusCode).json({
    success: true,
    message,
    data,
    meta
  });
};
```

---

# Struktur Project yang Lebih Profesional

```text
src/
│
├── controllers/
├── routes/
├── services/
├── middlewares/
├── utils/
│   └── apiResponse.js
│
├── lib/
│   └── prisma.js
│
├── app.js
└── server.js
```

---

# Tambahkan Global Error Handler

Daripada setiap controller berisi banyak `try-catch`, buat middleware khusus.

`src/middlewares/errorHandler.js`

```javascript
module.exports = (err, req, res, next) => {
  console.error(err);

  return res.status(500).json({
    success: false,
    message: "Internal Server Error",
    errors: process.env.NODE_ENV === "development"
      ? err.message
      : null
  });
};
```

Di `app.js`:

```javascript
const errorHandler = require("./middlewares/errorHandler");

app.use(errorHandler);
```

Controller:

```javascript
exports.getUsers = async (req, res, next) => {
  try {
    const users = await prisma.user.findMany();

    response.success(
      res,
      "Data user berhasil diambil",
      users
    );
  } catch (err) {
    next(err);
  }
};
```

---

# Format yang Umum Dipakai di Dunia Kerja

Response sukses:

```json
{
  "success": true,
  "message": "Data user berhasil diambil",
  "data": [...]
}
```

Response gagal:

```json
{
  "success": false,
  "message": "User tidak ditemukan",
  "errors": null
}
```

Response validasi:

```json
{
  "success": false,
  "message": "Validasi gagal",
  "errors": {
    "email": [
      "Email wajib diisi"
    ]
  }
}
```

Dengan pola ini, frontend cukup selalu memeriksa:

```javascript
if (response.success) {
  // gunakan response.data
} else {
  // tampilkan response.message
}
```

dan seluruh API Anda akan konsisten dari endpoint pertama sampai ratusan endpoint berikutnya.

## Pudin Saepudin
