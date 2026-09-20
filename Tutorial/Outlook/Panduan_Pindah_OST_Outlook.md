# Panduan Pemindahan Lokasi File Data Outlook Classic (.ost) ke Drive D

Dokumen ini berisi panduan teknis langkah demi langkah untuk memindahkan lokasi file cache offline Microsoft Outlook (`.ost`) dari drive sistem (`C:`) ke drive penyimpanan sekundernya (`D:`). Metode ini menggunakan fitur **Directory Junction (`mklink /J`)** bawaan Windows untuk menghemat kapasitas SSD/Drive C tanpa merusak konfigurasi profil email.

---

## 📋 Informasi Dasar

* **Tipe File:** `.ost` (Offline Outlook Data File)
* **Lokasi Default:** `%localappdata%\Microsoft\Outlook\`
* **Metode:** Windows Directory Junction (`mklink /J`)
* **Sifat Konfigurasi:** Permanen (Dapat dibatalkan/di-unlink kapan saja tanpa kehilangan data)

---

## 🚀 Bagian 1: Panduan Link (Memindahkan OST ke Drive D)

Prosedur ini dilakukan untuk memindahkan fisik data Outlook ke Drive D dan membuat jalan pintas (*junction link*) dari lokasi lama di Drive C.

### Langkah 1: Tutup Aplikasi Outlook
Pastikan Microsoft Outlook sudah ditutup sepenuhnya agar file `.ost` tidak terkunci oleh sistem.

### Langkah 2: Pindahkan Folder Data ke Drive D
1. Tekan kombinasi tombol **`Windows + R`** pada keyboard untuk membuka jendela *Run*.
2. Ketik `%localappdata%\Microsoft` lalu tekan **Enter**.
3. Cari folder bernama **`Outlook`**.
4. Klik kanan folder **`Outlook`** tersebut, lalu pilih **Cut** (atau tekan **`Ctrl + X`**).
5. Buka **Drive D:** pada Windows Explorer.
6. Buat folder baru (misalnya dengan nama `OutlookData`), lalu buka folder tersebut.
7. Tempelkan (**Paste** / **`Ctrl + V`**) folder `Outlook` di dalam lokasi `D:\OutlookData`.
   > **Hasil Akhir Lokasi Baru:** `D:\OutlookData`

### Langkah 3: Membuat Symbolic Link (Junction)
1. Buka Start Menu, ketik `cmd`.
2. Klik kanan pada **Command Prompt**, lalu pilih **Run as administrator**.
3. Ketik atau *copy-paste* perintah berikut ke dalam jendela Command Prompt, lalu tekan **Enter**:

   ```cmd
   mklink /J "%localappdata%\Microsoft\Outlook" "D:\OutlookData"
   ```

4. Jika berhasil, Command Prompt akan menampilkan pesan konfirmasi:
   `Junction created for C:\Users\...\AppData\Local\Microsoft\Outlook <<===>> D:\OutlookData`

### Langkah 4: Verifikasi
Buka kembali aplikasi Microsoft Outlook. Seluruh data offline (`.ost`) sekarang dibaca dan ditulis secara fisik di **Drive D**, sehingga Drive C tidak akan cepat penuh.

---

## 🔄 Bagian 2: Panduan Unlink (Mengembalikan Storage ke Drive C)

Jika di kemudian hari Anda ingin mengembalikan penyimpanan data Outlook ke lokasi semula di Drive C, ikuti langkah-langkah berikut:

### Langkah 1: Tutup Aplikasi Outlook
Pastikan Microsoft Outlook tidak sedang berjalan.

### Langkah 2: Hapus Junction Link di Drive C
1. Tekan **`Windows + R`**, ketik `%localappdata%\Microsoft` lalu tekan **Enter**.
2. Cari folder bernama **`Outlook`** (yang memiliki ikon panah kecil seperti *shortcut*).
3. **Hapus (Delete)** folder *shortcut* **`Outlook`** tersebut.
   > ℹ️ **Catatan Keamanan:** Menghapus folder shortcut ini **TIDAK akan menghapus** data email fisik yang ada di Drive D.

### Langkah 3: Pindahkan Kembali Folder Data ke Drive C
1. Buka folder lokasi penyimpanan di **`D:\OutlookData`**.
2. **Cut** (`Ctrl + X`) folder/isi data dari Drive D.
3. Buka lokasi `%localappdata%\Microsoft\`.
4. **Paste** (`Ctrl + V`) data tersebut kembali ke folder `%localappdata%\Microsoft\` dan pastikan nama foldernya kembali menjadi **`Outlook`**.

### Langkah 4: Verifikasi
Buka kembali Microsoft Outlook. Penyimpanan akan beroperasi normal menggunakan lokasi awal di Drive C.

---

## 📌 Hal Penting untuk Diperhatikan

1. **Integritas Data:** Metode Junction ini aman karena data tidak diubah, hanya lokasinya yang dipindahkan.
2. **Perubahan Drive Letter:** Pastikan huruf Drive D tidak berubah (misalnya tergeser menjadi E:). Selama Drive D adalah harddisk/SSD internal yang terpasang permanen, koneksi tidak akan terputus.
3. **Akun Server:** Karena file `.ost` adalah cache dari server (Exchange / Microsoft 365 / IMAP), apabila terjadi kendala ekstrem pada file lokal, email fisik Anda tetap tersimpan aman di server email.
