# Install Express JS dan Nodemon
Buat Folder Project
```
mkdir belajar-expressjs
```
Masuk Kedalam folder tersebut
```
cd belajar-expressjs
```
Inisialisasi Project kita
```
npm init -y
```
Install Express JS
```
npm install express
```
Install Nodemon hanya kebutuhan developement
```
npm install --save-dev nodemon
```
Buat satu buah file : src/server.js
Masukan kode ini :
```
const express = require('express');
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```
Sesuaikan Package.json di bagian "scripts" dengan ini :
```
{
  "name": "belajar-express",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "nodemon src/server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "express": "^5.2.1"
  },
  "devDependencies": {
    "nodemon": "^3.1.14"
  }
}

```

Kita running kode tersebut
```
npm run dev
```

Aksess menggunakan browser
```
http://localhost:3000
```
## Dokumentasi
* [Nodemon](https://www.npmjs.com/package/nodemon)
* [Express JS](https://expressjs.com/en/)

## Pudin Saepudin
