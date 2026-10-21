# Panduan Microsoft Hyper-V untuk Pemula

Panduan ini ditujukan untuk pengguna yang baru pertama kali menggunakan **Microsoft Hyper-V**.

Hyper-V memungkinkan kita membuat **komputer virtual (Virtual Machine / VM)** di dalam Windows.

Contohnya, kita bisa menjalankan:

- Windows 11 kedua di dalam laptop
- Linux
- Environment testing aplikasi
- Testing installer
- Testing update aplikasi
- Sistem operasi percobaan tanpa mengganggu Windows utama

---

# 1. Apa Itu Hyper-V? 💻

Hyper-V adalah fitur virtualisasi bawaan Microsoft Windows.

Secara sederhana:

```text
Laptop / PC Utama
│
├── Windows utama
│
└── Hyper-V
    │
    ├── Windows 11 Virtual
    ├── Ubuntu Virtual
    └── VM lainnya
```

Windows atau Linux yang berjalan di Hyper-V disebut:

```text
Virtual Machine
```

atau biasa disingkat:

```text
VM
```

VM bekerja seperti komputer terpisah, tetapi sebenarnya menggunakan CPU, RAM, SSD, dan jaringan dari komputer utama.

---

# 2. Contoh Kegunaan Hyper-V

Hyper-V sangat berguna untuk:

- Mencoba Windows tanpa install ulang PC
- Mencoba Linux
- Testing aplikasi
- Testing installer `.exe`
- Testing uninstall aplikasi
- Testing auto-update aplikasi
- Membuat environment Windows yang bersih
- Belajar server atau networking
- Mencoba software tanpa langsung mengganggu Windows utama

Untuk developer, Hyper-V sangat berguna karena kita dapat melakukan testing di Windows yang masih bersih.

---

# 3. Persyaratan Hyper-V

Sebelum mulai, pastikan Windows yang digunakan mendukung Hyper-V.

Umumnya Hyper-V tersedia pada:

```text
Windows 11 Pro
Windows 11 Enterprise
Windows 11 Education
```

Hyper-V versi penuh biasanya tidak tersedia secara resmi sebagai fitur bawaan pada:

```text
Windows 11 Home
```

Selain itu, processor harus mendukung virtualisasi.

Processor modern Intel dan AMD biasanya sudah mendukungnya.

---

# 4. Mengecek Dukungan Virtualisasi

Tekan:

```text
Win + R
```

Ketik:

```text
msinfo32
```

Kemudian tekan:

```text
Enter
```

Akan terbuka:

```text
System Information
```

Pada halaman **System Summary**, scroll ke bagian bawah.

Cari informasi seperti:

```text
Hyper-V - VM Monitor Mode Extensions: Yes
Hyper-V - Second Level Address Translation Extensions: Yes
Hyper-V - Virtualization Enabled in Firmware: Yes
Hyper-V - Data Execution Prevention Available: Yes
```

Idealnya semuanya menunjukkan:

```text
Yes
```

---

# 5. Jika Virtualization Masih "No"

Jika muncul:

```text
Virtualization Enabled in Firmware: No
```

kemungkinan virtualisasi belum diaktifkan di BIOS / UEFI.

Masuk ke BIOS laptop atau PC.

Nama pengaturannya bisa berbeda-beda, misalnya:

```text
Intel Virtualization Technology
Intel VT-x
AMD SVM
AMD-V
Virtualization Technology
```

Ubah menjadi:

```text
Enabled
```

Kemudian:

```text
Save & Exit
```

Komputer akan restart.

> ⚠️ Jangan mengubah pengaturan BIOS lain jika tidak tahu fungsinya.

---

# 6. Mengaktifkan Hyper-V

Setelah virtualisasi aktif, kita bisa mengaktifkan Hyper-V.

Tekan:

```text
Win + R
```

Ketik:

```text
optionalfeatures
```

Tekan:

```text
Enter
```

Akan muncul jendela:

```text
Windows Features
```

Cari:

```text
Hyper-V
```

Centang Hyper-V.

Pastikan bagian berikut juga aktif:

```text
Hyper-V
├── Hyper-V Management Tools
└── Hyper-V Platform
```

Disarankan juga mengaktifkan:

```text
Virtual Machine Platform
Windows Hypervisor Platform
```

Kemudian klik:

```text
OK
```

Windows akan memasang komponen yang diperlukan.

Setelah selesai, restart komputer.

---

# 7. Membuka Hyper-V Manager

Setelah komputer restart:

Klik:

```text
Start
```

Cari:

```text
Hyper-V Manager
```

Kemudian buka aplikasinya.

Di bagian kiri biasanya akan muncul nama komputer.

Contoh:

```text
Hyper-V Manager
└── DESKTOP-ABC123
```

Klik nama komputer tersebut.

Jika tidak muncul error, Hyper-V sudah siap digunakan. ✅

---

# 8. Siapkan File ISO Sistem Operasi

Untuk membuat VM, kita biasanya membutuhkan file installer sistem operasi dalam format:

```text
.iso
```

Contohnya:

```text
Windows11.iso
Ubuntu.iso
```

Untuk Windows 11, gunakan ISO resmi dari Microsoft.

Agar rapi, bisa membuat folder seperti:

```text
D:\Hyper-V\
```

Kemudian:

```text
D:\Hyper-V\
│
├── ISO\
├── VM\
└── Export\
```

Contoh penyimpanan ISO:

```text
D:\Hyper-V\ISO\Windows11.iso
```

---

# 9. Membuat Virtual Machine Baru 🖥️

Buka:

```text
Hyper-V Manager
```

Pada menu sebelah kanan pilih:

```text
New
> Virtual Machine
```

Akan muncul wizard.

Klik:

```text
Next
```

---

# 10. Memberikan Nama VM

Pada bagian nama, isi misalnya:

```text
Windows 11 Test
```

Aktifkan:

```text
Store the virtual machine in a different location
```

Lokasinya bisa menggunakan:

```text
D:\Hyper-V\VM\
```

Kemudian klik:

```text
Next
```

---

# 11. Memilih Generation

Hyper-V akan memberikan pilihan:

```text
Generation 1
Generation 2
```

Untuk sistem operasi modern seperti:

```text
Windows 10
Windows 11
Linux modern
```

pilih:

```text
Generation 2
```

Generation 2 mendukung fitur modern seperti:

```text
UEFI
Secure Boot
Virtual TPM
```

Klik:

```text
Next
```

> ⚠️ Generation VM tidak bisa diubah setelah VM dibuat.

---

# 12. Mengatur RAM

Selanjutnya tentukan RAM untuk VM.

Contoh jika komputer memiliki RAM 16 GB:

```text
Startup Memory:
4096 MB
```

atau:

```text
8192 MB
```

Untuk penggunaan ringan:

```text
4 GB
```

biasanya cukup.

Untuk penggunaan yang lebih nyaman:

```text
8 GB
```

Aktifkan:

```text
Use Dynamic Memory for this virtual machine
```

Dynamic Memory memungkinkan Hyper-V menyesuaikan penggunaan RAM sesuai kebutuhan VM.

Klik:

```text
Next
```

---

# 13. Mengatur Internet VM 🌐

Pada bagian:

```text
Configure Networking
```

biasanya tersedia:

```text
Default Switch
```

Pilih:

```text
Default Switch
```

Default Switch adalah pilihan paling mudah agar VM bisa mendapatkan koneksi internet.

Klik:

```text
Next
```

---

# 14. Membuat Virtual Hard Disk

Selanjutnya buat hard disk virtual.

File hard disk Hyper-V biasanya menggunakan format:

```text
.vhdx
```

Contoh:

```text
Windows 11 Test.vhdx
```

Untuk Windows 11 bisa menggunakan ukuran:

```text
80 GB
```

atau:

```text
100 GB
```

Jangan khawatir.

Jika disk menggunakan sistem dynamic, file VHDX tidak langsung memakan seluruh kapasitas yang ditentukan.

Misalnya kita memilih:

```text
100 GB
```

tetapi Windows baru menggunakan sekitar 25 GB, ukuran file fisik VHDX juga akan berkembang mengikuti penggunaan data.

Klik:

```text
Next
```

---

# 15. Memilih File ISO

Pilih:

```text
Install an operating system from a bootable image file
```

Klik:

```text
Browse
```

Pilih ISO.

Contoh:

```text
D:\Hyper-V\ISO\Windows11.iso
```

Klik:

```text
Next
```

Kemudian:

```text
Finish
```

VM sekarang sudah dibuat. 🎉

---

# 16. Mengatur CPU VM

Sebelum menjalankan VM, sebaiknya atur jumlah CPU.

Klik kanan VM.

Contoh:

```text
Windows 11 Test
```

Pilih:

```text
Settings
```

Masuk ke:

```text
Processor
```

Hyper-V kadang memberikan default:

```text
1 virtual processor
```

Untuk penggunaan normal bisa menggunakan:

```text
2 CPU
```

atau:

```text
4 CPU
```

Untuk laptop modern, konfigurasi:

```text
4 virtual processors
```

biasanya sudah nyaman.

> Jangan memberikan seluruh core CPU komputer utama ke VM.

---

# 17. Mengaktifkan Secure Boot dan TPM

Windows 11 membutuhkan fitur keamanan modern.

Masih di:

```text
Settings
```

pilih:

```text
Security
```

Pastikan:

```text
Enable Secure Boot
```

aktif.

Gunakan template:

```text
Microsoft Windows
```

Kemudian aktifkan:

```text
Enable Trusted Platform Module
```

Ini adalah Virtual TPM untuk VM.

Klik:

```text
Apply
```

kemudian:

```text
OK
```

---

# 18. Menjalankan VM

Klik kanan VM.

Pilih:

```text
Connect
```

Akan muncul jendela Virtual Machine Connection.

Klik:

```text
Start
```

Jika muncul tulisan:

```text
Press any key to boot from CD or DVD...
```

segera tekan:

```text
Enter
```

atau:

```text
Space
```

Installer Windows akan dimulai.

---

# 19. Install Windows

Proses install hampir sama seperti install Windows biasa.

Urutannya kurang lebih:

```text
Language
↓
Install Now
↓
Pilih Windows Edition
↓
Accept License
↓
Custom Install
```

Kemudian biasanya muncul:

```text
Drive 0 Unallocated Space
```

Pilih drive tersebut.

Klik:

```text
Next
```

Windows akan melakukan instalasi.

Yang penting:

```text
Windows VM
```

dipasang ke:

```text
file .vhdx
```

bukan langsung ke disk Windows utama.

---

# 20. Setelah Windows Selesai Diinstall

Setelah masuk ke Windows VM, komputer virtual dapat digunakan seperti komputer biasa.

Contohnya:

```text
Install browser
Install VSCodium
Install aplikasi
Testing installer
Testing update
Testing uninstall
```

VM bisa diperlakukan seperti sebuah PC terpisah.

---

# 21. Fitur Penting: Checkpoint 📸

Salah satu fitur terbaik Hyper-V adalah:

```text
Checkpoint
```

Checkpoint seperti membuat titik pemulihan kondisi VM.

Contohnya kita memiliki:

```text
Windows 11 yang masih bersih
```

Sebelum install aplikasi:

Klik kanan VM.

Pilih:

```text
Checkpoint
```

Beri nama:

```text
00 - Clean Windows
```

Kemudian lakukan testing.

Jika terjadi masalah, kita bisa kembali ke kondisi tersebut.

Klik kanan checkpoint.

Pilih:

```text
Apply
```

VM akan kembali ke kondisi sebelumnya.

---

# 22. Contoh Penggunaan Checkpoint

Contoh workflow:

```text
Clean Windows
       ↓
Checkpoint
       ↓
Install aplikasi
       ↓
Testing
       ↓
Testing update
       ↓
Testing uninstall
       ↓
Restore checkpoint
       ↓
Windows kembali bersih
```

Ini sangat berguna untuk testing aplikasi.

---

# 23. Nama Checkpoint yang Disarankan

Agar mudah dipahami, gunakan nama yang jelas.

Contoh:

```text
00 - Clean Windows
01 - Before App Install
02 - App Installed
03 - Before Auto Update Test
```

Dengan nama seperti ini, kita lebih mudah mengetahui kondisi setiap checkpoint.

---

# 24. Mengaktifkan Enhanced Session Mode

Enhanced Session Mode membuat penggunaan VM Windows lebih nyaman.

Fitur ini dapat membantu untuk:

- Clipboard
- Copy-paste
- Sharing drive
- Resolusi layar
- Audio
- Perangkat lokal tertentu

Di Hyper-V Manager pilih:

```text
Hyper-V Settings
```

Masuk ke:

```text
Enhanced Session Mode Policy
```

Aktifkan:

```text
Allow enhanced session mode
```

Kemudian buka:

```text
Enhanced Session Mode
```

Aktifkan:

```text
Use enhanced session mode
```

Klik:

```text
OK
```

---

# 25. Sharing Drive dari PC Utama ke VM

Saat membuka koneksi ke VM, pilih:

```text
Show Options
```

Kemudian:

```text
Local Resources
```

Pilih:

```text
More...
```

Centang drive yang ingin dibagikan.

Contoh:

```text
D:
```

Setelah masuk ke Windows VM, drive dari komputer utama dapat muncul sebagai drive atau resource yang terhubung.

Cara ini memudahkan memindahkan:

```text
Installer
File testing
Build aplikasi
Dokumen
```

dari host ke VM.

---

# 26. Cara Mematikan VM dengan Benar

Sebaiknya shutdown VM seperti komputer biasa.

Dari Windows VM:

```text
Start
> Power
> Shut down
```

Atau melalui Hyper-V:

```text
Shut Down
```

Perbedaan beberapa opsi:

```text
Shut Down
= Mematikan Windows dengan normal

Save
= Menyimpan kondisi VM seperti hibernate

Turn Off
= Mematikan paksa seperti mencabut listrik
```

Sebisa mungkin hindari:

```text
Turn Off
```

kecuali VM benar-benar hang atau tidak bisa dimatikan normal.

---

# 27. Setting VM yang Disarankan

Untuk komputer dengan RAM sekitar 16 GB dan kebutuhan testing ringan:

```text
Generation:
2

RAM:
4-8 GB

Dynamic Memory:
ON

CPU:
4 virtual processors

Virtual Disk:
80-100 GB

Disk Type:
Dynamic VHDX

Network:
Default Switch

Secure Boot:
ON

Virtual TPM:
ON

Checkpoint:
ON
```

Untuk sekadar testing installer, 4 GB RAM sering kali sudah cukup.

Jika aplikasi atau Windows terasa berat, bisa dinaikkan ke 6 GB atau 8 GB.

---

# 28. Struktur Folder yang Disarankan 📁

Agar file VM tidak berantakan, gunakan struktur seperti:

```text
D:\Hyper-V\
│
├── ISO\
│   ├── Windows11.iso
│   └── Ubuntu.iso
│
├── VM\
│   ├── Windows11-Test\
│   └── Ubuntu-Test\
│
└── Export\
```

Fungsinya:

```text
ISO
= Menyimpan installer sistem operasi

VM
= Menyimpan virtual machine

Export
= Menyimpan backup VM
```

---

# 29. Backup Virtual Machine

Hyper-V menyediakan fitur Export.

Sebelum export, sebaiknya VM dalam kondisi mati.

Di Hyper-V Manager:

Klik kanan VM.

Pilih:

```text
Export
```

Pilih folder tujuan.

Contoh:

```text
D:\Hyper-V\Export\
```

Hyper-V akan menyimpan data penting VM seperti:

```text
Configuration
Virtual Hard Disk
Checkpoint
Virtual Machine State
```

---

# 30. Mengembalikan VM dari Backup

Jika ingin menggunakan kembali VM hasil export:

Buka:

```text
Hyper-V Manager
```

Pilih:

```text
Import Virtual Machine
```

Kemudian pilih folder hasil export.

Ikuti wizard sampai selesai.

VM akan muncul kembali di Hyper-V Manager.

---

# 31. Workflow untuk Testing Installer Aplikasi

Hyper-V sangat cocok digunakan untuk testing installer aplikasi.

Contoh:

```text
Windows Host
│
├── Source Code
│
├── VSCodium
│
├── Node.js
│
└── Build Tools
│
└── Hyper-V
    │
    └── Windows 11 Test
        │
        ├── Clean Windows
        ├── Install aplikasi
        ├── Test aplikasi
        ├── Test update
        ├── Test uninstall
        └── Restore checkpoint
```

Keuntungannya adalah aplikasi diuji pada Windows yang lebih bersih.

---

# 32. Contoh Workflow Testing Auto Update

Misalnya kita memiliki:

```text
Versi lama:
0.5.5

Versi baru:
0.5.6
```

Workflow testing:

```text
1. Restore checkpoint "Clean Windows"

2. Install aplikasi versi 0.5.5

3. Jalankan aplikasi

4. Pastikan versi 0.5.5 berjalan normal

5. Publish versi 0.5.6

6. Jalankan pengecekan update

7. Pastikan aplikasi menemukan versi 0.5.6

8. Jalankan proses update

9. Restart aplikasi jika diperlukan

10. Pastikan versi berubah menjadi 0.5.6
```

Dengan cara ini kita bisa memastikan update benar-benar berjalan di komputer yang bersih.

---

# 33. Contoh Workflow Testing Installer

Buat checkpoint:

```text
00 - Clean Windows
```

Kemudian:

```text
Install aplikasi
↓
Cek shortcut
↓
Cek Start Menu
↓
Jalankan aplikasi
↓
Cek fitur utama
↓
Restart Windows
↓
Jalankan aplikasi kembali
↓
Uninstall aplikasi
↓
Cek apakah uninstall bersih
```

Jika selesai:

```text
Restore "00 - Clean Windows"
```

VM kembali seperti sebelum aplikasi diinstall.

---

# 34. Kenapa Checkpoint Sangat Berguna?

Tanpa VM:

```text
Install
↓
Test
↓
Uninstall
↓
Masih mungkin ada cache / registry / file tersisa
```

Dengan VM dan checkpoint:

```text
Clean Windows
↓
Install
↓
Test
↓
Restore Checkpoint
↓
100% kembali ke kondisi checkpoint
```

Inilah salah satu alasan Hyper-V sangat cocok untuk QA dan testing aplikasi.

---

# 35. Beberapa Hal yang Perlu Dihindari ⚠️

Jangan terlalu banyak memberikan resource ke VM.

Contoh jika PC hanya memiliki:

```text
16 GB RAM
```

jangan memberikan:

```text
14 GB
```

ke VM.

Windows utama juga membutuhkan RAM.

Lebih aman menggunakan:

```text
4-8 GB
```

untuk VM Windows biasa.

---

# 36. Jangan Menyimpan Terlalu Banyak Checkpoint

Checkpoint menggunakan ruang disk.

Semakin banyak perubahan VM setelah checkpoint dibuat, semakin besar penggunaan storage.

Hindari membuat puluhan checkpoint tanpa kebutuhan.

Contoh yang cukup:

```text
00 - Clean Windows
01 - Basic Setup Complete
02 - Before Important Test
```

Checkpoint testing sementara bisa dihapus jika sudah tidak digunakan.

---

# 37. Pastikan SSD Memiliki Ruang Kosong

VM dapat menggunakan storage cukup besar.

Contoh:

```text
Windows VM:
30-60 GB

ISO:
5-7 GB

Checkpoint:
Bisa bertambah beberapa GB
```

Jadi sebaiknya sediakan minimal:

```text
80-120 GB
```

ruang kosong jika akan menggunakan Windows VM secara rutin.

---

# 38. Apakah VM Aman untuk Windows Utama?

Secara umum, VM terisolasi dari Windows utama.

Windows VM menggunakan virtual disk:

```text
.vhdx
```

bukan langsung menggunakan partisi Windows utama.

Namun VM tetap bisa terhubung dengan komputer utama jika kita secara sengaja memberikan akses seperti:

```text
Shared Drive
Network Share
Clipboard
Folder Sharing
```

Jadi tetap berhati-hati saat menjalankan software yang benar-benar tidak dipercaya.

---

# 39. Hyper-V vs Install Windows Langsung

Tanpa Hyper-V:

```text
Install Windows
↓
Partisi disk
↓
Restart
↓
Dual boot
↓
Risiko salah partisi
```

Dengan Hyper-V:

```text
Buat VM
↓
Pilih ISO
↓
Install
↓
Windows berjalan di dalam window
```

Untuk belajar dan testing, Hyper-V jauh lebih praktis.

---

# 40. Ringkasan Langkah dari Awal Sampai Selesai 🚀

Urutan lengkap:

```text
1. Cek Windows mendukung Hyper-V

2. Cek virtualization

3. Aktifkan virtualization di BIOS jika diperlukan

4. Aktifkan Hyper-V melalui Windows Features

5. Restart Windows

6. Buka Hyper-V Manager

7. Download ISO

8. Buat Virtual Machine

9. Pilih Generation 2

10. Tentukan RAM

11. Gunakan Default Switch

12. Buat VHDX

13. Pilih ISO

14. Atur CPU

15. Aktifkan Secure Boot

16. Aktifkan Virtual TPM

17. Start VM

18. Install Windows

19. Selesaikan setup Windows

20. Buat checkpoint "Clean Windows"

21. Mulai testing aplikasi
```

---

# 41. Konfigurasi Sederhana yang Direkomendasikan

Jika bingung memilih setting, gunakan ini:

```text
VM Name:
Windows 11 Test

Generation:
Generation 2

Startup RAM:
4096 MB

Dynamic Memory:
Enabled

Processor:
4

Network:
Default Switch

Virtual Disk:
100 GB Dynamic VHDX

Secure Boot:
Enabled

TPM:
Enabled

Checkpoint:
Enabled
```

Setting ini cukup untuk belajar dan melakukan testing aplikasi Windows ringan hingga menengah.

---

# 42. Kesimpulan

Microsoft Hyper-V memungkinkan kita mempunyai komputer virtual tanpa harus membeli komputer tambahan.

Untuk pemula, manfaat terbesar Hyper-V adalah:

- Aman untuk belajar
- Mudah membuat Windows testing
- Bisa menggunakan checkpoint
- Tidak perlu install ulang PC utama
- Cocok untuk testing installer
- Cocok untuk testing update aplikasi
- Bisa kembali ke kondisi Windows bersih dengan cepat

Konsep terpenting yang perlu diingat:

```text
Host
= Komputer / Windows utama

VM
= Komputer virtual

VHDX
= Hard disk virtual

ISO
= Installer sistem operasi

Checkpoint
= Titik pemulihan kondisi VM

Default Switch
= Koneksi jaringan VM
```

Jika baru mulai menggunakan Hyper-V, cukup fokus dahulu pada:

```text
Buat VM
↓
Install Windows
↓
Buat Checkpoint
↓
Lakukan Testing
↓
Restore Checkpoint
```

Itu sudah cukup untuk mendapatkan manfaat utama Hyper-V. ✅
