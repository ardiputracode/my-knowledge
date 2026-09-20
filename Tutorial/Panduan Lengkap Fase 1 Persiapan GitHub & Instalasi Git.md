# 🚀 Panduan Lengkap Fase 1: Persiapan GitHub & Instalasi Git

Panduan ini dirancang khusus untuk pemula yang menggunakan Windows 11 dan ingin menghubungkan PC mereka ke GitHub untuk proyek web. Simpan teks ini sebagai file `panduan-github.md`.

---

## 🌐 Langkah 1: Membuat Akun GitHub
1. Buka browser di PC-mu, lalu kunjungi **[github.com](https://github.com)**.
2. Klik tombol **"Sign up"** di pojok kanan atas layar.
3. GitHub akan memandumu melalui beberapa layar:
   - Masukkan alamat email yang aktif.
   - Buat kata sandi (*password*) yang kuat.
   - Pilih *username* (nama pengguna). Ini akan menjadi bagian dari alamat web proyekmu nanti (contoh: `github.com/namapenggunamu`).
4. Ikuti proses verifikasi sederhana (biasanya berupa *puzzle* atau kode yang dikirim ke email).
5. Pilih paket **"Free"** (Gratis) saat ditanya tentang rencana penggunaan. Ini sudah lebih dari cukup untuk belajar dan proyek web pribadi.

---

## 📁 Langkah 2: Membuat Repository (Proyek Web) Pertama
Setelah akun jadi dan kamu sudah *login*, ikuti langkah ini untuk membuat "rumah" bagi kode web-mu:
1. Di halaman utama GitHub (Dashboard), cari tombol **"+"** di pojok kanan atas layar, lalu klik **"New repository"**.
2. **Repository name**: Beri nama proyekmu. 
   - *Tips*: Gunakan huruf kecil, angka, atau tanda strip (-). Hindari spasi. Contoh: `website-portofolio` atau `belajar-web-dasar`.
3. **Description (Opsional)**: Isi dengan kalimat singkat tentang proyek ini. Contoh: "Website pertama yang sedang saya pelajari".
4. **Public atau Private?**: 
   - *Public*: Siapa saja di internet bisa melihat kode ini (bagus untuk portofolio).
   - *Private*: Hanya kamu (dan orang yang kamu undang) yang bisa melihatnya. 
   - *Saran*: Untuk pemula, *Public* sering kali lebih mudah karena banyak tutorial yang mengasumsikan repo bersifat publik.
5. ⚠️ **LANGKAH SANGAT PENTING**: Cari dan **centang** kotak bertuliskan **"Add a README file"**.
   - *Mengapa?* File `README.md` ini ibarat "papan nama" di depan rumahmu. Dengan mencentang ini, GitHub akan langsung membuat satu file awal. Ini akan sangat memudahkan proses penyambungan ke PC-mu nanti, karena repository-mu tidak akan "kosong melompong" (yang bisa menyebabkan *error* saat perintah tarik data pertama kali).
6. Klik tombol hijau **"Create repository"** di bagian bawah.

---

## 🔗 Langkah 3: Mencatat "Alamat" Repository
Setelah repository berhasil dibuat, kamu akan diarahkan ke halaman proyek barumu.
1. Cari tombol hijau bertuliskan **"Code"** di bagian tengah atas daftar file.
2. Klik tombol tersebut, dan akan muncul sebuah *pop-up*.
3. Pastikan tab yang terpilih adalah **"HTTPS"**.
4. Salin (*copy*) alamat yang muncul di sana. Bentuknya akan persis seperti ini: 
   `https://github.com/username-kamu/nama-repository-kamu.git`
5. Simpan alamat ini di Notepad atau catat di kertas. Ini adalah "alamat rumah digital" yang nanti akan kita hubungkan ke PC Windows 11-mu.

---

## ⚙️ Langkah 4: Menginstal Git di Windows 11
Git adalah perangkat lunak "mesin penerjemah" yang harus diinstal di PC-mu agar bisa berkomunikasi dengan GitHub.
1. Kunjungi **[git-scm.com/downloads/win](https://git-scm.com/downloads/win)**.
2. Unduh (*download*) versi untuk Windows (klik tautan *Click here to download* untuk installer 64-bit).
3. Jalankan file `.exe` yang sudah diunduh.
4. **Proses Instalasi**: Kamu cukup klik **Next** terus hingga proses instalasi selesai. Pengaturan bawaan (*default*) sudah sangat cukup dan aman untuk kebutuhan kita saat ini.

---

## 💻 Langkah 5: Menguji Perintah Pertama
Kita perlu memastikan Git sudah terinstal dengan benar dan bisa dikenali oleh Windows.
1. Buka **Command Prompt** di PC-mu (Tekan tombol `Start` di keyboard, ketik `cmd`, lalu buka aplikasi "Command Prompt").
2. Ketik perintah berikut, lalu tekan **Enter**:
   ```bash
   git --version
   ```
3. **Hasil yang diharapkan**: Layar akan menampilkan versi Git yang terinstal, contohnya: `git version 2.4x.x.windows.1`. 
   - *Jika muncul pesan seperti ini, berarti Git sudah siap digunakan!*
   - *Jika muncul pesan "git is not recognized...", berarti instalasi belum berhasil atau kamu perlu me-restart PC terlebih dahulu.*

---

## 🆔 Langkah 6: Konfigurasi Identitas Git
Agar GitHub tahu siapa yang mengirim kode, kita harus mendaftarkan nama dan email yang sama dengan akun GitHub-mu.
1. Di Command Prompt, ketik dua perintah ini satu per satu (ganti dengan data aslimu), lalu tekan **Enter** setelah setiap baris:
   ```bash
   git config --global user.name "Nama Lengkapmu"
   git config --global user.email "emailmu@contoh.com"
   ```
2. Untuk memastikan berhasil, kamu bisa mengetik: `git config --list`