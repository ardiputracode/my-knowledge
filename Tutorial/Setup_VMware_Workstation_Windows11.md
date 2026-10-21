# Panduan Setup VMware Workstation di Windows 11

Panduan ini ditujukan untuk menjalankan **VMware Workstation sebagai
platform virtualisasi utama** di Windows 11.

Target konfigurasi:

``` text
Hardware
   ↓
AMD-V / Intel VT-x
   ↓
VMware Workstation
   ↓
Virtual Machine
```

Bukan:

``` text
Hardware
   ↓
Windows Hypervisor / Hyper-V
   ↓
VMware Workstation
   ↓
Virtual Machine
```

> **Catatan:** Mematikan Hyper-V/VBS dapat menonaktifkan fitur Windows
> tertentu dan mengurangi sebagian perlindungan keamanan. Jika VMware
> sudah bekerja dengan baik saat Hyper-V aktif, konfigurasi VMware-only
> sebenarnya tidak wajib.

------------------------------------------------------------------------

# 1. Persiapan

Sebelum melakukan perubahan, pastikan:

-   Windows 11 sudah diperbarui.
-   Virtualization AMD-V atau Intel VT-x aktif di BIOS/UEFI.
-   Semua pekerjaan sudah disimpan karena komputer akan direstart.
-   VMware Workstation sudah atau akan diinstal.

Untuk memeriksa virtualization:

1.  Tekan `Ctrl + Shift + Esc`.
2.  Buka **Task Manager**.
3.  Pilih **Performance**.
4.  Pilih **CPU**.
5.  Cari `Virtualization: Enabled`.

Jika tertulis `Virtualization: Disabled`, aktifkan AMD-V/SVM atau Intel
Virtualization Technology melalui BIOS/UEFI terlebih dahulu.

------------------------------------------------------------------------

# 2. Nonaktifkan Fitur Hyper-V

Tekan `Win + R`, masukkan:

``` text
optionalfeatures
```

Nonaktifkan fitur berikut jika tersedia:

-   [ ] Hyper-V
-   [ ] Virtual Machine Platform
-   [ ] Windows Hypervisor Platform
-   [ ] Windows Sandbox

Jika **WSL/WSL2 tidak digunakan**, kamu juga dapat menonaktifkan:

-   [ ] Windows Subsystem for Linux

> Jika kamu masih membutuhkan WSL2 atau Windows Sandbox, pertimbangkan
> untuk tidak menggunakan konfigurasi VMware-only karena fitur tersebut
> bergantung pada teknologi virtualisasi Windows.

------------------------------------------------------------------------

# 3. Nonaktifkan Memory Integrity

Buka:

**Windows Security → Device Security → Core isolation → Core isolation
details**

Ubah **Memory integrity** menjadi **OFF**.

> **PERHATIAN:** Memory Integrity merupakan fitur keamanan Windows.
> Mematikannya dapat mengurangi perlindungan terhadap serangan tertentu
> yang menargetkan kernel/driver. Jangan mematikannya kecuali memang
> diperlukan untuk konfigurasi VMware-only.

------------------------------------------------------------------------

# 4. Nonaktifkan Windows Hypervisor Saat Boot

Buka **Command Prompt** atau **Windows Terminal** menggunakan **Run as
administrator**.

Jalankan:

``` bat
bcdedit /set hypervisorlaunchtype off
```

Jika berhasil, Windows akan menampilkan:

``` text
The operation completed successfully.
```

------------------------------------------------------------------------

# 5. Restart Windows

Restart komputer agar perubahan hypervisor berlaku sepenuhnya.

------------------------------------------------------------------------

# 6. Install VMware Workstation

Download VMware Workstation dari sumber resmi VMware/Broadcom.

Jalankan installer sebagai administrator. Untuk penggunaan normal, opsi
default installer biasanya sudah cukup.

Restart Windows jika diminta.

------------------------------------------------------------------------

# 7. Buat Virtual Machine

Buka **VMware Workstation** lalu pilih:

**Create a New Virtual Machine → Typical**

Pilih ISO sistem operasi yang ingin digunakan, misalnya Windows 11 ISO.

------------------------------------------------------------------------

# 8. Rekomendasi Resource VM

## CPU

Mulai dari:

``` text
Processors: 1
Cores per processor: 2
```

Total: **2 vCPU**.

Jangan langsung memberikan semua core CPU laptop kepada VM.

## RAM

Untuk Windows 11:

-   Minimum praktis: **4 GB**
-   Lebih nyaman: **6 GB**

Jika laptop mempunyai RAM 16 GB, 4--6 GB untuk VM merupakan titik awal
yang masuk akal.

------------------------------------------------------------------------

# 9. Storage VM

Untuk Windows 11, gunakan sekitar **64--80 GB**.

Contoh:

``` text
Maximum disk size: 80 GB
```

Dengan virtual disk yang berkembang sesuai penggunaan, VMware tidak
harus langsung menggunakan seluruh kapasitas maksimum tersebut.

------------------------------------------------------------------------

# 10. Install VMware Tools

Setelah Windows di dalam VM selesai diinstal, install **VMware Tools**.

VMware Tools membantu meningkatkan:

-   Resolusi layar
-   Mouse integration
-   Clipboard
-   Copy/paste
-   Shared folders
-   Driver VM
-   Integrasi host dan guest

Restart Guest Windows setelah instalasi.

------------------------------------------------------------------------

# 11. Update Windows Guest

Di dalam VM buka:

**Settings → Windows Update → Check for updates**

Install update yang diperlukan dan restart VM.

Tujuannya adalah membuat satu kondisi Windows bersih yang siap dijadikan
baseline.

------------------------------------------------------------------------

# 12. Buat Snapshot Windows Bersih

Setelah Windows Guest siap dan belum dipenuhi aplikasi testing, buat
snapshot bernama:

``` text
Clean Windows 11
```

Workflow:

``` text
Clean Windows
      ↓
Install aplikasi
      ↓
Testing
      ↓
Selesai
      ↓
Restore Snapshot
      ↓
Clean Windows lagi
```

Tidak perlu install ulang Windows setiap kali melakukan testing.

------------------------------------------------------------------------

# 13. Contoh Workflow Testing Aplikasi Electron

``` text
Development PC
      │
      ├── npm run build
      │
      ↓
Setup.exe
      │
      ↓
VMware Windows 11
      │
      ├── Install Setup.exe
      ├── Test aplikasi
      ├── Test shortcut
      ├── Test Start Menu
      ├── Test icon
      ├── Test auto-update
      ├── Test mandatory update
      └── Test uninstall
      │
      ↓
Restore Snapshot
      │
      ↓
Clean Windows 11
```

Ini membuat testing lebih konsisten karena setiap release dapat diuji
dari kondisi Windows yang sama.

------------------------------------------------------------------------

# 14. Shared Folder

Jika sering memindahkan installer dari Host ke VM:

**VM Settings → Options → Shared Folders → Always enabled**

Tambahkan folder build, misalnya:

``` text
D:\side project\builds
```

------------------------------------------------------------------------

# 15. Network

Untuk testing aplikasi biasa gunakan **NAT**.

``` text
Internet
   ↓
Windows Host
   ↓
VMware NAT
   ↓
Windows Guest
```

Mode ini cocok untuk download update, test API, test auto-update, GitHub
release, web API, dan Jira API.

------------------------------------------------------------------------

# 16. Optimasi untuk Laptop

Rekomendasi awal:

``` text
CPU     : 2 vCPU
RAM     : 4–6 GB
Storage : 64–80 GB
Network : NAT
```

Jika performa kurang, naikkan resource secara bertahap.

Jangan langsung memberikan terlalu banyak CPU dan RAM karena dapat
membuat Windows Host kekurangan resource.

------------------------------------------------------------------------

# 17. Saat Menggunakan Baterai

VM membutuhkan CPU dan RAM tambahan sehingga konsumsi daya laptop
meningkat.

Jika menggunakan baterai:

-   Suspend VM jika tidak digunakan.
-   Jangan biarkan VM berjalan di background.
-   Gunakan 2 vCPU.
-   Gunakan RAM secukupnya.
-   Hindari menjalankan update Windows Guest saat tidak diperlukan.

------------------------------------------------------------------------

# 18. Mengaktifkan Hypervisor Windows Kembali

Jika ingin menggunakan Hyper-V/WSL2/Sandbox kembali, buka Terminal
sebagai Administrator dan jalankan:

``` bat
bcdedit /set hypervisorlaunchtype auto
```

Aktifkan kembali fitur yang diperlukan melalui:

``` text
Win + R
→ optionalfeatures
```

Jika sebelumnya Memory Integrity dimatikan, pertimbangkan
mengaktifkannya kembali melalui:

**Windows Security → Device Security → Core isolation → Memory integrity
→ ON**

Kemudian restart Windows.

------------------------------------------------------------------------

# 19. Konfigurasi Akhir yang Direkomendasikan

``` text
BIOS / UEFI
│
├── AMD-V / Intel VT-x
│       ON
│
▼
Windows 11
│
├── Hyper-V
│       OFF
│
├── Windows Hypervisor Platform
│       OFF
│
├── Virtual Machine Platform
│       OFF jika tidak dibutuhkan
│
├── Windows Sandbox
│       OFF jika tidak dibutuhkan
│
├── Memory Integrity
│       OFF hanya jika diperlukan
│
└── hypervisorlaunchtype
        OFF
        │
        ▼
VMware Workstation
        │
        ▼
Windows 11 Guest
```

------------------------------------------------------------------------

# 20. Setup VM yang Disarankan untuk Software Testing

``` text
Guest OS    : Windows 11
CPU         : 2 vCPU
RAM         : 4–6 GB
Disk        : 80 GB
Network     : NAT
VMware Tools: Installed
Snapshot    : Clean Windows 11
```

Contoh snapshot:

``` text
01 - Clean Windows 11
02 - Windows Updated
03 - Testing Environment
```

Hindari terlalu banyak snapshot karena dapat menggunakan storage
tambahan dan membuat pengelolaan VM lebih rumit.

------------------------------------------------------------------------

# Hasil Akhir

``` text
Windows 11 Host
      │
      ↓
VMware Workstation
      │
      ↓
Windows 11 Test VM
      │
      ├── Clean Snapshot
      │
      ├── Install Build
      │
      ├── Test
      │
      └── Restore
```

Konfigurasi ini cocok untuk development dan software QA karena
environment testing dapat dikembalikan ke kondisi bersih dengan cepat
tanpa harus melakukan install ulang Windows setiap kali melakukan
pengujian.
