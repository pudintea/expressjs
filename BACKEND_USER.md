# REST API BACKEND USER Feature-Based / Modular Architecture

Struktur Folder Final
```
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
│   │
│   ├── auth/
│   │   ├── auth.controller.js
│   │   ├── auth.service.js
│   │   ├── auth.repository.js
│   │   ├── auth.validator.js
│   │   └── auth.routes.js
│   │
│   ├── products/
│   ├── orders/
│   └── customers/
│
├── config/
│   ├── db.js
│   └── env.js
│
├── middlewares/
│   ├── auth.middleware.js
│   ├── validate.middleware.js
│   ├── error.middleware.js
│   └── notfound.middleware.js
│
├── utils/
│   ├── response.js
│   ├── bcrypt.js
│   ├── jwt.js
│   └── logger.js
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

## Pudin Saepudin
