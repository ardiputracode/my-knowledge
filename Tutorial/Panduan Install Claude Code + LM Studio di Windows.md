# Panduan Install Claude Code + LM Studio di Windows

Panduan ini ditujukan untuk **pemula** yang ingin menggunakan **Claude Code sebagai coding agent**, tetapi model AI dijalankan secara **lokal menggunakan LM Studio**.

Contoh setup:

* OS: Windows 10 / Windows 11
* Editor: VSCodium / VS Code
* Coding Agent: Claude Code
* AI Server: LM Studio
* Model: Gemma / Qwen / model lokal lainnya
* Project contoh: Electron

---

# 1. Gambaran Sistem

Sistem yang akan kita buat:

```text
VSCodium
    │
    │ Project
    ▼
Claude Code
    │
    │ Request AI
    ▼
http://localhost:1234
    │
    ▼
LM Studio
    │
    ▼
Model Lokal
Gemma / Qwen / dll.
```

Artinya:

* **VSCodium** digunakan untuk membuka dan mengedit project.
* **Claude Code** digunakan sebagai AI coding agent.
* **LM Studio** menjalankan AI server di komputer.
* **Model lokal** yang memberikan respons AI.

Dengan konfigurasi ini, Claude Code diarahkan ke LM Studio menggunakan:

```text
ANTHROPIC_BASE_URL=http://localhost:1234
```

---

# 2. Install Git for Windows

Claude Code membutuhkan beberapa fungsi command line yang disediakan oleh Git for Windows.

Download:

```text
https://git-scm.com/downloads/win
```

Install menggunakan pengaturan default.

Setelah selesai, buka PowerShell baru.

Cek instalasi:

```powershell
git --version
```

Jika berhasil akan muncul seperti:

```text
git version 2.x.x.windows.x
```

Berarti Git sudah siap.

---

# 3. Install LM Studio

Download LM Studio:

```text
https://lmstudio.ai/
```

Install seperti aplikasi Windows biasa.

Setelah selesai:

1. Buka LM Studio.
2. Cari model AI.
3. Download model.
4. Load model tersebut.

Contoh model:

```text
Gemma
Qwen
GPT-OSS
```

Untuk coding agent, sebaiknya gunakan model yang memiliki kemampuan:

* Coding
* Tool calling
* Instruction following
* Context yang cukup besar

Jika model Gemma yang sudah kamu gunakan berjalan dengan baik, model tersebut bisa tetap digunakan terlebih dahulu.

---

# 4. Install Claude Code

Buka **PowerShell**.

Tidak perlu menjalankan PowerShell sebagai Administrator.

Jalankan:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Tunggu proses instalasi selesai.

Setelah itu:

1. Tutup PowerShell.
2. Buka PowerShell baru.

Cek Claude Code:

```powershell
claude --version
```

Contoh hasil:

```text
2.1.278 (Claude Code)
```

Jika versi Claude Code muncul, instalasi berhasil.

---

# 5. Menjalankan Local Server LM Studio

Buka:

```text
LM Studio
```

Load model yang ingin digunakan.

Kemudian masuk ke bagian:

```text
Developer
```

Cari Local Server.

Klik:

```text
Start Server
```

Secara default LM Studio menggunakan:

```text
http://localhost:1234
```

Artinya AI server sekarang berjalan di komputer sendiri.

---

# 6. Menghubungkan Claude Code ke LM Studio

Sekarang Claude Code perlu diberi tahu bahwa AI yang digunakan bukan server Anthropic, tetapi LM Studio.

Buka PowerShell.

Jalankan:

```powershell
$env:ANTHROPIC_BASE_URL="http://localhost:1234"
$env:ANTHROPIC_AUTH_TOKEN="lmstudio"
$env:CLAUDE_CODE_ATTRIBUTION_HEADER="0"
```

Sekarang Claude Code akan diarahkan ke:

```text
Claude Code
      │
      ▼
localhost:1234
      │
      ▼
LM Studio
```

---

# 7. Masalah Environment Variable Hilang

Ada satu masalah dengan cara sebelumnya.

Perintah:

```powershell
$env:ANTHROPIC_BASE_URL="http://localhost:1234"
```

hanya berlaku untuk **PowerShell yang sedang terbuka**.

Jika PowerShell ditutup, konfigurasi tersebut hilang.

Akibatnya setiap membuka PowerShell kita harus memasukkannya lagi.

Supaya tidak perlu melakukan itu, kita akan menyimpannya secara permanen.

---

# 8. Menyimpan Konfigurasi Secara Permanen

Buka PowerShell.

Jalankan sekali:

```powershell
[Environment]::SetEnvironmentVariable(
    "ANTHROPIC_BASE_URL",
    "http://localhost:1234",
    "User"
)

[Environment]::SetEnvironmentVariable(
    "ANTHROPIC_AUTH_TOKEN",
    "lmstudio",
    "User"
)

[Environment]::SetEnvironmentVariable(
    "CLAUDE_CODE_ATTRIBUTION_HEADER",
    "0",
    "User"
)
```

Setelah selesai:

1. Tutup PowerShell.
2. Tutup CMD.
3. Tutup VSCodium.
4. Buka kembali PowerShell.

Hal ini diperlukan agar Windows membaca environment variable yang baru.

---

# 9. Mengecek Environment Variable

Buka PowerShell baru.

Jalankan:

```powershell
echo $env:ANTHROPIC_BASE_URL
```

Seharusnya:

```text
http://localhost:1234
```

Kemudian:

```powershell
echo $env:ANTHROPIC_AUTH_TOKEN
```

Seharusnya:

```text
lmstudio
```

Kemudian:

```powershell
echo $env:CLAUDE_CODE_ATTRIBUTION_HEADER
```

Seharusnya:

```text
0
```

Jika semuanya muncul, konfigurasi sudah tersimpan.

---

# 10. Mengecek dari CMD

Environment variable tersebut juga dapat digunakan dari CMD.

Buka CMD.

Jalankan:

```cmd
echo %ANTHROPIC_BASE_URL%
```

Hasil:

```text
http://localhost:1234
```

Kemudian:

```cmd
echo %ANTHROPIC_AUTH_TOKEN%
```

Hasil:

```text
lmstudio
```

Jadi kita tidak perlu mengatur ulang konfigurasi setiap membuka CMD atau PowerShell.

---

# 11. Menjalankan Claude Code

Pastikan LM Studio sudah:

```text
✓ Dibuka
✓ Model sudah di-load
✓ Local Server sudah ON
✓ Port menggunakan 1234
```

Kemudian buka project.

Contoh:

```powershell
cd "D:\side project\audioqa-worktoolkit"
```

Jalankan:

```powershell
claude
```

Claude Code akan terbuka.

---

# 12. Menjalankan dari VSCodium

Cara yang lebih praktis adalah menjalankannya langsung dari VSCodium.

Buka VSCodium.

Pilih:

```text
File
↓
Open Folder
```

Pilih project.

Contoh:

```text
D:\side project\audioqa-worktoolkit
```

Kemudian buka:

```text
Terminal
↓
New Terminal
```

Pastikan terminal berada di:

```text
D:\side project\audioqa-worktoolkit
```

Jalankan:

```powershell
claude
```

Claude Code sekarang bekerja pada project tersebut.

---

# 13. Tes Claude Code

Untuk percobaan pertama, jangan langsung meminta Claude Code mengubah project.

Gunakan prompt sederhana:

```text
Analyze this project structure.

Do not modify any files.

Explain what this project does.

Explain it in simple language for a beginner.
```

Claude Code akan membaca project dan menjelaskannya.

---

# 14. Prompt untuk Mempelajari Project Electron

Untuk project Electron yang sudah cukup besar, gunakan:

```text
Please analyze this Electron project and learn its current architecture and application flow.

For now, DO NOT modify any files.

Please:

1. Inspect the project folder structure.
2. Read package.json.
3. Read the Electron main process files.
4. Read preload scripts.
5. Read renderer HTML, CSS, and JavaScript modules.
6. Identify how data flows between Renderer -> Preload -> Main Process.
7. Identify all IPC handlers and API integrations.
8. Identify how external services are currently implemented.
9. Identify the application startup flow from npm start until the UI is displayed.
10. Identify the build and packaging flow.
11. Summarize the architecture in simple language suitable for a beginner.

Do not refactor, fix, delete, or create files yet.

The goal is only to understand the existing project before we continue development.
```

Ini lebih aman daripada langsung meminta AI memperbaiki seluruh project.

---

# 15. Workflow yang Disarankan

Jangan langsung memberikan task besar seperti:

```text
Fix my entire application.
```

Lebih baik gunakan alur:

```text
Analyze
   ↓
Explain
   ↓
Create Plan
   ↓
Edit
   ↓
Test
   ↓
Review
```

Contohnya:

### Langkah 1 — Analisis

```text
Analyze jira-api.js.

Do not modify anything yet.

Find potential bugs and security problems.
```

### Langkah 2 — Buat rencana

```text
Create a plan to fix these problems.

Do not modify the file yet.
```

### Langkah 3 — Edit

Setelah rencananya masuk akal:

```text
Implement the fixes.

Do not remove existing comments.

Only modify code related to these fixes.
```

### Langkah 4 — Test

```text
Check the modified code for:

- syntax errors
- runtime errors
- security problems
- regression risks
```

Cara ini lebih aman untuk model lokal.

---

# 16. Rutinitas Harian

Setelah semua konfigurasi selesai, kamu **tidak perlu melakukan proses instalasi lagi**.

Setiap ingin coding:

```text
1. Buka LM Studio

2. Load model

3. Start Local Server

4. Buka VSCodium

5. Buka project

6. Buka Terminal

7. Jalankan:

   claude

8. Berikan task
```

---

# 17. Arsitektur Akhir

Sistem akhirnya:

```text
┌──────────────────────────────┐
│          VSCodium            │
│                              │
│     Electron Project         │
│             │                │
│             ▼                │
│        Claude Code           │
└─────────────┬────────────────┘
              │
              │
              │ ANTHROPIC_BASE_URL
              │
              ▼
       localhost:1234
              │
              ▼
┌──────────────────────────────┐
│          LM Studio           │
│                              │
│    Anthropic-compatible      │
│        API Server            │
│                              │
│             │                │
│             ▼                │
│        Local Model           │
│                              │
│    Gemma / Qwen / dll.       │
└──────────────────────────────┘
```

---

# 18. Tentang Login Claude

Jika tujuan kita adalah menggunakan model lokal melalui LM Studio, Claude Code diarahkan ke:

```text
http://localhost:1234
```

melalui:

```text
ANTHROPIC_BASE_URL
```

Arsitektur yang diinginkan:

```text
Claude Code
      │
      ▼
LM Studio
      │
      ▼
Local Model
```

bukan:

```text
Claude Code
      │
      ▼
Anthropic Cloud
```

Karena itu pastikan konfigurasi LM Studio sudah benar terlebih dahulu jika Claude Code meminta login.

---

# 19. Jika Muncul "Not Logged In"

Contoh:

```text
Not logged in · Please run /login
```

Pertama periksa environment variable:

```powershell
echo $env:ANTHROPIC_BASE_URL
```

Harus:

```text
http://localhost:1234
```

Periksa token:

```powershell
echo $env:ANTHROPIC_AUTH_TOKEN
```

Harus:

```text
lmstudio
```

Kemudian pastikan:

```text
LM Studio        = terbuka
Model            = loaded
Local Server     = ON
Port             = 1234
```

Setelah itu tutup Claude Code:

```text
Ctrl + C
```

Kemudian jalankan lagi:

```powershell
claude
```

---

# 20. Jika Claude Code Tidak Bisa Menghubungi LM Studio

Periksa LM Studio.

Pastikan server menunjukkan alamat:

```text
http://localhost:1234
```

Kemudian cek:

```powershell
echo $env:ANTHROPIC_BASE_URL
```

Jika kosong, environment variable belum terbaca.

Tutup terminal dan buka terminal baru.

Jika masih kosong, simpan kembali:

```powershell
[Environment]::SetEnvironmentVariable(
    "ANTHROPIC_BASE_URL",
    "http://localhost:1234",
    "User"
)
```

Lalu restart terminal/VSCodium.

---

# 21. Jika Ingin Menghapus Konfigurasi LM Studio

Jika suatu hari ingin mengembalikan Claude Code agar tidak diarahkan ke LM Studio:

```powershell
[Environment]::SetEnvironmentVariable(
    "ANTHROPIC_BASE_URL",
    $null,
    "User"
)

[Environment]::SetEnvironmentVariable(
    "ANTHROPIC_AUTH_TOKEN",
    $null,
    "User"
)

[Environment]::SetEnvironmentVariable(
    "CLAUDE_CODE_ATTRIBUTION_HEADER",
    $null,
    "User"
)
```

Kemudian:

```text
Tutup PowerShell
Tutup CMD
Tutup VSCodium
```

Buka kembali terminal.

Konfigurasi LM Studio sudah dilepas.

---

# 22. Ringkasan

Yang hanya dilakukan **sekali**:

```text
Install Git
      ↓
Install LM Studio
      ↓
Download model
      ↓
Install Claude Code
      ↓
Set ANTHROPIC_BASE_URL
      ↓
Set ANTHROPIC_AUTH_TOKEN
```

Setelah itu penggunaan sehari-hari hanya:

```text
LM Studio
   ↓
Load Model
   ↓
Start Server
   ↓
VSCodium
   ↓
Open Project
   ↓
Terminal
   ↓
claude
   ↓
Mulai Coding
```

Dengan setup ini, **Claude Code berfungsi sebagai coding agent**, sementara proses AI dapat dijalankan oleh **model lokal melalui LM Studio**.
