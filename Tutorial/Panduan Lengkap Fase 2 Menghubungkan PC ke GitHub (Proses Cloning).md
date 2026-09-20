# 🚀 Panduan Lengkap Fase 2: Menghubungkan PC ke GitHub (Proses Cloning)

Setelah memiliki akun, repository, dan menginstal Git, langkah selanjutnya adalah "menurunkan" atau menyalin repository dari GitHub ke dalam PC-mu. Proses ini disebut **Cloning** (mengkloning). 

---

## 📂 Langkah 1: Menyiapkan Lokasi Folder (Contoh)
Kita perlu tempat yang rapi untuk menyimpan proyek ini. 
- **Contoh Lokasi**: Kita akan membuat folder khusus bernama `ProyekWeb` di drive D (atau di folder Documents jika kamu tidak punya drive D).
- **Tujuan Akhir**: Folder proyek kita akan berada di `D:\ProyekWeb\nama-repository-kamu`.

*Catatan: Jika kamu menggunakan folder Documents, jalannya akan seperti `C:\Users\NamaUser\Documents\ProyekWeb`.*

---

## 💻 Langkah 2: Membuka Command Prompt dan Pindah Lokasi
Kita harus memberi tahu Command Prompt (CMD) untuk bekerja di dalam folder yang baru kita siapkan. Kita menggunakan perintah `cd` (*Change Directory* / Ganti Direktori).

1. Buka **Command Prompt**.
2. Ketik perintah berikut untuk pindah ke drive D (jika kamu memakai contoh drive D), lalu tekan **Enter**:
   ```bash
   D:
   ```
3. Buat folder baru bernama `ProyekWeb` dengan perintah:
   ```bash
   mkdir ProyekWeb
   ```
4. Masuk ke dalam folder tersebut dengan perintah:
   ```bash
   cd ProyekWeb
   ```
   *(Sekarang, tulisan di CMD-mu seharusnya berubah menjadi `D:\ProyekWeb>`, yang menandakan kamu sudah berada di lokasi yang tepat).*

---

## 📥 Langkah 3: Menjalankan Perintah Clone
Ini adalah inti dari Fase 2. Kita akan menggunakan alamat URL yang sudah kamu salin di Fase 1.

1. Pastikan kamu masih berada di dalam folder `D:\ProyekWeb>`.
2. Ketik perintah `git clone` diikuti dengan spasi, lalu *paste* (tempel) alamat URL repository-mu. Bentuknya akan seperti ini:
   ```bash
   git clone https://github.com/username-kamu/nama-repository-kamu.git
   ```
3. Tekan **Enter**.
4. **Apa yang terjadi?**: Git akan menghubungi GitHub, mengunduh seluruh isi repository (termasuk file `README.md` yang kita buat tadi), dan membuat folder baru dengan nama repository-mu secara otomatis di dalam folder `ProyekWeb`.

---

## ✅ Langkah 4: Memastikan File Sudah Berhasil Diunduh
Kita perlu memverifikasi bahwa proses cloning berhasil.

1. Masih di Command Prompt, ketik perintah untuk melihat isi folder saat ini:
   ```bash
   dir
   ```
   *(Perintah `dir` akan menampilkan daftar folder dan file. Kamu seharusnya melihat nama repository-mu muncul di daftar).*
2. Masuk ke dalam folder repository tersebut (ganti `nama-repository-kamu` dengan nama aslimu):
   ```bash
   cd nama-repository-kamu
   ```
3. Ketik `dir` sekali lagi. 
4. **Hasil yang diharapkan**: Kamu akan melihat file `README.md` di dalam daftar. 

**Selamat!** 🎉 PC Windows 11-mu sekarang sudah resmi terhubung dan memiliki salinan lokal dari repository GitHub-mu.