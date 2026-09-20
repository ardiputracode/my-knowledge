# Panduan Membuat Aplikasi Windows Portable dengan Electron.js

Dokumen ini menjelaskan cara membuat aplikasi desktop Windows sederhana menggunakan Electron.js, kemudian mengubahnya menjadi aplikasi `.exe` portable yang bisa langsung dijalankan tanpa proses instalasi.
Panduan ini dibuat untuk pemula.

## 1. Apa yang akan kita buat?

Di akhir panduan, kita akan memiliki aplikasi seperti ini:

```text
electron-test/
├── node_modules/
├── dist/
│   └── Electron Test 1.0.0.exe
├── index.html
├── main.js
├── package.json
└── package-lock.json
```

File:
```text
Electron Test 1.0.0.exe
```
adalah aplikasi Windows portable.

Artinya:
- Tidak perlu installer.
- Tidak perlu klik Next → Next → Finish.
- File `.exe` bisa langsung dijalankan.
- Bisa disalin ke folder lain.
- Bisa disimpan di flashdisk.
- Bisa dijalankan dari folder tertentu tanpa instalasi ke `Program Files`.

## 2. Apa itu Electron?

Electron adalah framework yang memungkinkan kita membuat aplikasi desktop menggunakan teknologi web:
- HTML
- CSS
- JavaScript

Secara sederhana:
```text
HTML + CSS + JavaScript
          ↓
       Electron
          ↓
     Windows .exe
```

Jadi kalau sudah terbiasa membuat website, kita bisa menggunakan kemampuan tersebut untuk membuat aplikasi desktop.

Electron menggunakan Chromium sebagai engine tampilan dan Node.js untuk kemampuan sistem seperti filesystem, proses, dan sebagainya.

## 3. Persiapan

Kita membutuhkan:
- Windows
- Node.js
- npm
- Electron
- electron-builder

## 4. Install Node.js

Buka website resmi Node.js:
https://nodejs.org/

Download versi LTS.

Setelah selesai install, buka:
- Command Prompt
atau:
- PowerShell

Kemudian jalankan:
```bash
node -v
```

Jika berhasil, akan muncul versi Node.js, misalnya:
```text
v22.x.x
```

Kemudian cek npm:
```bash
npm -v
```

Contohnya:
```text
10.x.x
```

Kalau kedua command tersebut menghasilkan nomor versi, berarti Node.js dan npm sudah siap.

## 5. Membuat folder project

Misalnya kita ingin menyimpan project di:
```text
D:\side project\electron test
```

Buka PowerShell dan jalankan:
```bash
cd "D:\side project"
mkdir "electron test"
cd "electron test"
```

Atau bisa juga membuat folder tersebut secara manual menggunakan Windows Explorer.

Kemudian pastikan terminal berada di folder project:
```text
PS D:\side project\electron test>
```

## 6. Membuat project Node.js

Jalankan:
```bash
npm init -y
```

Command tersebut akan membuat file:
```text
package.json
```

File ini berisi informasi dan konfigurasi project kita.

Strukturnya sekarang:
```text
electron test/
└── package.json
```

## 7. Install Electron

Install Electron sebagai dependency untuk development:
```bash
npm install --save-dev electron
```

Tunggu sampai proses selesai.

Kemudian cek apakah Electron sudah terinstall:
```bash
npx electron --version
```

Jika muncul misalnya:
```text
v37.x.x
```
berarti Electron sudah berhasil dipasang.

## 8. Membuat file `main.js`

Buat file:
```text
main.js
```

Isi dengan:
```javascript
const { app, BrowserWindow } = require("electron");

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  });

  win.loadFile("index.html");
}

app.whenReady().then(() => {
  createWindow();
});
```

Apa fungsi `main.js`?

`main.js` adalah bagian utama aplikasi Electron.
Tugasnya antara lain:
- membuat window aplikasi;
- membuka halaman HTML;
- mengatur lifecycle aplikasi;
- berkomunikasi dengan sistem operasi;
- menangani filesystem/database melalui Node.js.

Secara sederhana:
```text
main.js
   ↓
Electron
   ↓
Window aplikasi
```

## 9. Membuat file `index.html`

Buat file:
```text
index.html
```

Isi dengan:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Electron Test</title>
</head>
<body>
  <h1>Hello Electron!</h1>
  <p>
    Ini adalah aplikasi desktop pertama saya.
  </p>
</body>
</html>
```

Sekarang struktur project:
```text
electron test/
├── main.js
├── index.html
└── package.json
```

## 10. Mengatur `package.json`

### Apa itu `package.json`?
Bagi pemula, anggaplah `package.json` sebagai **"KTP" sekaligus "Buku Manual"** untuk aplikasi Anda. File ini adalah identitas utama project Node.js/Electron. Tanpa file ini, sistem tidak akan tahu nama aplikasi Anda, versi berapa yang sedang dipakai, atau bagaimana cara menjalankannya.

### Apa gunanya?
1.  **Identitas:** Menyimpan nama, versi, dan deskripsi aplikasi.
2.  **Titik Masuk (Entry Point):** Memberi tahu Electron file mana yang harus dijalankan pertama kali (dalam hal ini `main.js`).
3.  **Perintah Pintas (Scripts):** Memungkinkan Anda menjalankan perintah panjang hanya dengan ketikan singkat seperti `npm start`.
4.  **Daftar Belanjaan (Dependencies):** Mencatat library apa saja (seperti Electron) yang dibutuhkan agar aplikasi bisa berjalan.

### Penjelasan Parameter
Buka file `package.json` dan ubah isinya menjadi seperti berikut. Perhatikan penjelasan di bawah setiap baris:

```json
{
  "name": "electron-test",
  "version": "1.0.0",
  "description": "Aplikasi desktop Electron pertama",
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  },
  "devDependencies": {
    "electron": "^37.0.0"
  }
}
```

**Rincian Parameter:**
*   `"name"`: Nama unik proyek Anda (gunakan huruf kecil dan tanda hubung). Ini akan menjadi nama folder saat build nanti.
*   `"version"`: Versi aplikasi (format: major.minor.patch). Penting untuk manajemen update dan penamaan file `.exe` hasil build.
*   `"description"`: Penjelasan singkat tentang aplikasi Anda.
*   `"main"`: **(Sangat Penting)** File entry point aplikasi. Electron akan mencari dan menjalankan file ini saat aplikasi dimulai. Pastikan nilainya sama dengan nama file utama Anda (`main.js`).
*   `"scripts"`: Kumpulan perintah pintas.
    *   `"start": "electron ."` artinya ketika Anda mengetik `npm start` di terminal, sistem sebenarnya menjalankan perintah `electron .` (menjalankan Electron di folder saat ini).
*   `"devDependencies"`: Daftar library yang hanya dibutuhkan saat masa pengembangan (development), bukan saat aplikasi dipakai user akhir. Di sini tercatat `"electron"` beserta versinya.

> **Catatan:** Versi Electron tidak harus persis `37.0.0`. Gunakan versi yang sudah terinstall di project kamu (bisa dicek di `package-lock.json` atau output saat install).

## 11. Menjalankan aplikasi

Sekarang jalankan:
```bash
npm start
```

Electron akan membuka sebuah window.
Kurang lebih tampilannya:
```text
┌──────────────────────────────────────┐
│ Electron Test                    ×   │
├──────────────────────────────────────┤
│                                      │
│          Hello Electron!             │
│                                      │
│  Ini adalah aplikasi desktop         │
│  pertama saya.                       │
│                                      │
└──────────────────────────────────────┘
```

Selamat.
Pada tahap ini kita sudah mempunyai aplikasi desktop Electron.

## 12. Apa bedanya dengan website biasa?

Website biasa:
```text
HTML
CSS
JavaScript
   ↓
Chrome / Edge
   ↓
Website
```

Electron:
```text
HTML
CSS
JavaScript
   ↓
Electron + Chromium
   ↓
Windows Application
```

Jadi aplikasi Electron tidak harus dibuka melalui Chrome atau Edge.

## 13. Bagaimana dengan `localStorage`?

Electron tetap mendukung `localStorage`.

Contohnya:
```javascript
localStorage.setItem("username", "Budi");
const username = localStorage.getItem("username");
console.log(username);
```

Data tersebut akan disimpan oleh Chromium/Electron.

Yang penting:
```text
Chrome
└── localStorage Chrome

Electron
└── localStorage Electron
```

Keduanya terpisah.
Jadi data `localStorage` dari website yang dibuka di Chrome tidak otomatis tersedia di aplikasi Electron.
Sebaliknya, `localStorage` Electron juga tidak otomatis tersedia di Chrome.

## 14. Kapan menggunakan `localStorage`?

`localStorage` cocok untuk data sederhana seperti:
- theme = dark
- language = id
- sidebar = collapsed
- lastPage = dashboard

Contoh:
```javascript
localStorage.setItem("theme", "dark");
```

Tetapi untuk aplikasi yang memiliki data penting dalam jumlah banyak, sebaiknya jangan menjadikan `localStorage` sebagai database utama.

Misalnya aplikasi:
- Kasir
- Inventory
- Akuntansi
- CRM
- Manajemen pelanggan
- Transaksi

lebih cocok menggunakan database seperti SQLite.

Contoh arsitektur:
```text
Electron
   │
   ├── HTML/CSS/JS
   │
   └── Main Process
          │
          ↓
       SQLite
```

## 15. Membuat aplikasi menjadi `.exe`

Untuk membuat aplikasi Windows, kita membutuhkan tool packaging.
Kita akan menggunakan:
```text
electron-builder
```

Install dengan:
```bash
npm install --save-dev electron-builder
```

Setelah selesai, cek:
```bash
npx electron-builder --version
```

Jika muncul versi, berarti berhasil.

## 16. Masalah `approve-scripts` pada npm

Pada beberapa versi npm terbaru, npm dapat menampilkan pesan seperti:
```text
1 package has install scripts not yet covered by allowScripts:
electron-winstaller@5.4.0
```

Ini merupakan mekanisme keamanan npm untuk package yang mempunyai install script.

Jika package tersebut memang bagian dari dependency Electron yang kita gunakan, izinkan dengan:
```bash
npm approve-scripts electron-winstaller
```

Kemudian:
```bash
npm install
```

Setelah itu cek:
```bash
npx electron-builder --version
```

Jika berhasil, kita bisa lanjut.

## 17. Mengatur Electron Builder

Sekarang ubah `package.json`.

Contohnya:
```json
{
   "name": "electron-test",
   "version": "1.0.0",
   "description": "Aplikasi desktop Electron",
   "main": "main.js",
   "scripts": {
     "start": "electron .",
     "build": "electron-builder"
   },
   "devDependencies": {
     "electron": "^37.0.0",
     "electron-builder": "^26.0.0"
   },
   "build": {
     "appId": "com.example.electrontest",
     "productName": "Electron Test",
     "win": {
       "target": "portable"
     }
   }
 }
```

Bagian yang paling penting adalah:
```json
"win": {
  "target": "portable"
}
```

Ini memberitahu Electron Builder bahwa target kita adalah aplikasi Windows portable.

## 18. Build aplikasi

Sekarang jalankan:
```bash
npm run build
```

Electron Builder akan mulai melakukan proses packaging.
Tunggu sampai selesai.

Biasanya akan muncul folder:
```text
dist/
```

Struktur project menjadi:
```text
electron test/
├── dist/
│   └── Electron Test 1.0.0.exe
│
├── node_modules/
├── index.html
├── main.js
├── package.json
└── package-lock.json
```

File:
```text
Electron Test 1.0.0.exe
```
adalah aplikasi portable kita.

## 19. Menjalankan aplikasi portable

Masuk ke:
```text
dist
```

Kemudian double-click:
```text
Electron Test 1.0.0.exe
```

Aplikasi akan langsung terbuka.

Tidak perlu:
```text
Install
↓
Next
↓
Next
↓
Finish
```

Karena target kita adalah:
```text
portable
```

## 20. Mengganti Logo Aplikasi (`.exe`)

Secara default, jika kita tidak mengatur apa-apa, Electron Builder akan memakai logo bawaan Electron (gambar atom biru).

Kita bisa menggantinya dengan logo kita sendiri.

### Siapkan file logo

Untuk aplikasi Windows, format logo yang dipakai adalah:
```text
.ico
```

Ketentuan:
- Format: `.ico` (bukan `.png` atau `.jpg`)
- Ukuran: disarankan minimal 256x256 pixel
- Nama file: misalnya `icon.ico`

Kenapa harus `.ico`?
Karena Windows membaca ikon aplikasi dalam format `.ico`, bukan format gambar web biasa.

Kalau kamu punya gambar `.png` atau `.jpg`, convert dulu menjadi `.ico`.
Bisa pakai website converter gratis seperti:
- https://convertio.co/png-ico/
- https://icoconvert.com/

### Letakkan logo di folder project

Simpan `icon.ico` sejajar dengan `main.js` dan `package.json`.

Struktur project menjadi:
```text
electron test/
├── dist/
├── node_modules/
├── icon.ico
├── index.html
├── main.js
├── package.json
└── package-lock.json
```

### Daftarkan logo di `package.json`

Buka `package.json`, lalu tambahkan baris `"icon"` di dalam bagian `"build"`:

```json
"build": {
  "appId": "com.example.electrontest",
  "productName": "Electron Test",
  "icon": "icon.ico",
  "win": {
    "target": "portable"
  }
}
```

Arti parameternya:
*   `"icon"`: memberitahu Electron Builder file gambar mana yang akan dipakai sebagai ikon file `.exe`.

### (Opsional) Ganti logo di jendela aplikasi

Setelah build, ikon file `.exe` di Windows Explorer akan berubah.

Tetapi logo kecil di pojok kiri atas jendela aplikasi (title bar) diatur dari HTML.

Buka `index.html`, lalu tambahkan baris ini di dalam `<head>`:

```html
<link rel="icon" href="icon.ico">
```

### Build ulang

Jalankan lagi:
```bash
npm run build
```

Sekarang file:
```text
dist/
└── Electron Test 1.0.0.exe
```
sudah memakai logo kamu sendiri.

## 21. Portable bukan berarti data otomatis berada di samping `.exe`

Ini hal yang cukup penting.

Misalnya kita mempunyai:
```text
D:\Apps\Todo\
└── Todo.exe
```

Lalu aplikasi menggunakan:
```javascript
localStorage.setItem("username", "Budi");
```

Data `localStorage` tidak otomatis berarti:
```text
D:\Apps\Todo\
├── Todo.exe
└── localStorage.json
```

Electron mempunyai lokasi `userData` sendiri.
Kita bisa mengetahui lokasinya melalui:
```javascript
const { app } = require("electron");
console.log(app.getPath("userData"));
```

Di Windows, biasanya berada di area:
```text
%APPDATA%
```
untuk aplikasi tersebut.

Jadi konsep:
```text
Portable EXE
```
dan:
```text
Portable Data
```
adalah dua hal berbeda.

## 22. Kalau ingin database ikut dengan aplikasi

Misalnya kita ingin membuat aplikasi portable yang memiliki database:
```text
MyApp/
├── MyApp.exe
└── data/
    └── database.sqlite
```

Maka lokasi database perlu kita tentukan secara eksplisit.
Misalnya menggunakan Node.js dan SQLite dari Main Process.

Konsepnya:
```text
MyApp.exe
   │
   └── data/
       └── database.sqlite
```

Ini berguna jika aplikasi memang dimaksudkan untuk dijalankan dari:
- Flashdisk
- Hard disk eksternal
- Folder network tertentu
- Folder lokal tanpa instalasi

Namun untuk aplikasi production, lokasi database perlu dirancang hati-hati karena ada masalah permission, backup, concurrency, dan update aplikasi.

## 23. Struktur aplikasi Electron yang lebih realistis

Untuk aplikasi sederhana:
```text
Electron
│
├── main.js
│
└── index.html
```

Untuk aplikasi yang mulai serius:
```text
Electron App
│
├── main.js
│
├── preload.js
│
├── renderer/
│   ├── index.html
│   ├── app.js
│   └── style.css
│
└── database/
    └── database.sqlite
```

Konsep komunikasinya:
```text
Renderer
HTML / CSS / JS
      │
      │ IPC
      ▼
Preload
      │
      ▼
Main Process
      │
      ├── File System
      ├── SQLite
      ├── Printer
      └── OS
```

Untuk aplikasi Electron modern, sebaiknya gunakan Preload + `contextBridge` + IPC daripada memberikan akses Node.js secara langsung kepada halaman HTML.

## 24. Perintah yang perlu diingat

Untuk membuat project:
```bash
npm init -y
```

Install Electron:
```bash
npm install --save-dev electron
```

Menjalankan aplikasi:
```bash
npm start
```

Install Electron Builder:
```bash
npm install --save-dev electron-builder
```

Jika npm meminta approval:
```bash
npm approve-scripts electron-winstaller
```

Build:
```bash
npm run build
```

Hasilnya:
```text
dist/
└── Electron Test 1.0.0.exe
```

## 25. Alur keseluruhan

Kalau diringkas, prosesnya adalah:
```text
1. Install Node.js
         ↓
2. Buat folder project
         ↓
3. npm init -y
         ↓
4. Install Electron
         ↓
5. Buat main.js
         ↓
6. Buat index.html
         ↓
7. npm start
         ↓
8. Aplikasi Electron berjalan
         ↓
9. Install electron-builder
         ↓
10. Konfigurasi target portable
         ↓
11. npm run build
         ↓
12. dist/Electron Test 1.0.0.exe
```

## 26. Kesimpulan

Dengan Electron, kita bisa mengubah aplikasi berbasis:
```text
HTML
CSS
JavaScript
```
menjadi:
```text
Windows Desktop Application
        ↓
       .exe
```

Untuk membuat aplikasi portable, konfigurasi utamanya adalah:
```json
"win": {
  "target": "portable"
}
```

Kemudian:
```bash
npm run build
```

Hasil akhirnya adalah file `.exe` yang bisa langsung dijalankan.

Perlu diingat bahwa portable `.exe` tidak sama dengan portable database/data. Jika aplikasi membutuhkan database SQLite atau file data yang harus ikut berada di folder aplikasi, lokasi penyimpanannya perlu diatur secara khusus.

Untuk aplikasi yang lebih serius, struktur yang disarankan adalah:
```text
Renderer
   ↓
Preload / IPC
   ↓
Main Process
   ↓
SQLite / File System / OS
```

Dengan struktur tersebut, aplikasi Electron bisa berkembang dari aplikasi HTML sederhana menjadi aplikasi desktop lengkap seperti aplikasi kasir, inventory, administrasi, dashboard, dan sebagainya.