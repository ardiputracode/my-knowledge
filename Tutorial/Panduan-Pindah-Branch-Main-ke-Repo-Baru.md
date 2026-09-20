# Panduan Menyalin Branch `main` ke Repository GitHub Baru

Panduan ini ditujukan untuk pemula yang ingin menyalin repository GitHub lama ke repository GitHub baru dengan kondisi:

- Repository lama memiliki branch `main` dan `sub`.
- Hanya branch `main` yang ingin dipindahkan.
- Seluruh commit history pada `main` harus tetap terbawa.
- Branch `sub` tidak perlu dipindahkan.
- Repository lama tetap aman dan tidak diubah.

---

## 1. Gambaran Proses

Misalnya repository lama:

```text
REPO-LAMA
├── main
│   ├── Commit A
│   ├── Commit B
│   ├── Commit C
│   └── Commit D
└── sub
    ├── Commit X
    └── Commit Y
```

Target repository baru:

```text
REPO-BARU
└── main
    ├── Commit A
    ├── Commit B
    ├── Commit C
    └── Commit D
```

Branch `sub` tidak akan dipindahkan.

---

## 2. Persiapan

Pastikan sudah memiliki:

- Git
- VSCodium
- Akun GitHub
- Repository sumber
- Repository tujuan

Cek apakah Git sudah terpasang:

```bash
git --version
```

Jika nomor versi muncul, Git sudah siap.

---

## 3. Buat Repository Baru di GitHub

Buat repository baru yang akan menjadi tujuan.

Contoh:

```text
Repo lama : QA-Toolkit
Repo baru : QA-Toolkit-Demo
```

Sebaiknya repository baru dibuat **kosong**.

Jangan aktifkan:

- Add a README file
- `.gitignore`
- License

Hal ini membantu menghindari konflik ketika melakukan push pertama.

---

## 4. Salin URL Repository Lama

Di GitHub:

1. Buka repository lama.
2. Klik **Code**.
3. Pilih **HTTPS**.
4. Copy URL repository.

Contoh:

```text
https://github.com/USERNAME/QA-Toolkit.git
```

---

## 5. Buka Terminal

Di VSCodium pilih:

```text
Terminal → New Terminal
```

Kemudian arahkan terminal ke folder tempat project ingin disimpan.

Contoh:

```bash
cd D:\Projects
```

Sesuaikan lokasi tersebut dengan komputer kamu.

---

## 6. Clone Hanya Branch `main`

Gunakan:

```bash
git clone --branch main --single-branch https://github.com/USERNAME/REPO-LAMA.git
```

Contoh:

```bash
git clone --branch main --single-branch https://github.com/USERNAME/QA-Toolkit.git
```

`--branch main` memilih branch `main`.

`--single-branch` membuat clone hanya mengambil history yang mengarah ke branch tersebut.

### Penting

Jangan tambahkan:

```bash
--depth 1
```

Karena `--depth 1` membuat shallow clone dan memotong history commit lama.

---

## 7. Masuk ke Folder Repository

```bash
cd REPO-LAMA
```

Contoh:

```bash
cd QA-Toolkit
```

---

## 8. Pastikan Branch Aktif adalah `main`

Jalankan:

```bash
git branch
```

Seharusnya:

```text
* main
```

Tanda `*` menunjukkan branch yang sedang aktif.

---

## 9. Cek Commit History

Jalankan:

```bash
git log --oneline
```

Contoh:

```text
a82bc11 Add mandatory update
72fd922 Fix Jira connection
18ca231 Add Report Builder
92ac731 Add settings page
9ab3211 Initial project
```

Jika commit lama muncul, history branch `main` sudah berhasil diambil.

Jika layar `git log` terbuka dalam mode khusus, tekan:

```text
q
```

untuk keluar.

---

## 10. Cek Remote Lama

Jalankan:

```bash
git remote -v
```

Contoh:

```text
origin  https://github.com/USERNAME/QA-Toolkit.git (fetch)
origin  https://github.com/USERNAME/QA-Toolkit.git (push)
```

Artinya repository lokal masih terhubung dengan repository lama.

---

## 11. Putuskan Hubungan dengan Repository Lama

Jalankan:

```bash
git remote remove origin
```

Perintah ini:

- Tidak menghapus repository GitHub lama.
- Tidak menghapus source code lokal.
- Tidak menghapus commit history.

Perintah tersebut hanya menghapus alamat remote `origin` lama dari repository lokal.

Cek:

```bash
git remote -v
```

Jika tidak ada output, remote lama sudah dilepas.

---

## 12. Salin URL Repository Baru

Buka repository baru di GitHub.

Klik:

```text
Code → HTTPS
```

Kemudian copy URL.

Contoh:

```text
https://github.com/USERNAME/QA-Toolkit-Demo.git
```

---

## 13. Hubungkan Repository Lokal ke Repository Baru

Jalankan:

```bash
git remote add origin https://github.com/USERNAME/REPO-BARU.git
```

Contoh:

```bash
git remote add origin https://github.com/USERNAME/QA-Toolkit-Demo.git
```

---

## 14. Pastikan Remote Sudah Benar

Jalankan:

```bash
git remote -v
```

Seharusnya:

```text
origin  https://github.com/USERNAME/QA-Toolkit-Demo.git (fetch)
origin  https://github.com/USERNAME/QA-Toolkit-Demo.git (push)
```

**Periksa langkah ini dengan teliti.**

Pastikan URL menunjuk ke **repository baru**, bukan repository lama.

---

## 15. Push Hanya Branch `main`

Jalankan:

```bash
git push -u origin main
```

Perintah ini akan mengirim branch `main` beserta commit history yang sudah ada di clone lokal ke repository baru.

Untuk skenario ini, jangan gunakan:

```bash
git push origin --all
```

Kita hanya ingin mengirim branch `main`.

---

## 16. Verifikasi di GitHub

Buka repository baru.

Pastikan:

1. Semua file dari `main` sudah muncul.
2. Buka bagian **Commits**.
3. Commit history lama masih muncul.
4. Repository baru hanya memiliki branch `main`.

Contoh hasil:

```text
REPO-LAMA
├── main
│   └── seluruh commit history
└── sub
    └── tidak dipindahkan

            ↓

REPO-BARU
└── main
    └── seluruh commit history main
```

---

## 17. Verifikasi dari Terminal

Jalankan:

```bash
git status
```

Contoh hasil:

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Cek remote:

```bash
git remote -v
```

Pastikan alamatnya adalah repository baru.

Terakhir, cek history:

```bash
git log --oneline
```

History lama seharusnya tetap tersedia.

---

# Ringkasan Perintah

Ganti `USERNAME`, `REPO-LAMA`, dan `REPO-BARU` sesuai repository kamu.

```bash
# 1. Clone hanya branch main beserta history-nya
git clone --branch main --single-branch https://github.com/USERNAME/REPO-LAMA.git

# 2. Masuk ke folder
cd REPO-LAMA

# 3. Pastikan branch aktif
git branch

# 4. Cek history
git log --oneline

# 5. Cek remote lama
git remote -v

# 6. Lepaskan remote lama
git remote remove origin

# 7. Pastikan remote sudah hilang
git remote -v

# 8. Tambahkan repository baru
git remote add origin https://github.com/USERNAME/REPO-BARU.git

# 9. Pastikan URL repository baru sudah benar
git remote -v

# 10. Push hanya main
git push -u origin main

# 11. Verifikasi
git status
git log --oneline
```

---

# Contoh Lengkap

Repository lama:

```text
https://github.com/USERNAME/QA-Toolkit.git
```

Repository baru:

```text
https://github.com/USERNAME/QA-Toolkit-Demo.git
```

Perintah:

```bash
git clone --branch main --single-branch https://github.com/USERNAME/QA-Toolkit.git

cd QA-Toolkit

git branch

git log --oneline

git remote -v

git remote remove origin

git remote -v

git remote add origin https://github.com/USERNAME/QA-Toolkit-Demo.git

git remote -v

git push -u origin main

git status
```

---

# Checklist Akhir

- [ ] Repository baru dibuat dalam keadaan kosong.
- [ ] Clone menggunakan `--branch main --single-branch`.
- [ ] Tidak menggunakan `--depth 1`.
- [ ] `git log --oneline` menampilkan history lama.
- [ ] Remote repository lama sudah dilepas.
- [ ] `origin` sudah menunjuk ke repository baru.
- [ ] Hanya menjalankan `git push -u origin main`.
- [ ] File sudah muncul di repository baru.
- [ ] Commit history lama muncul di repository baru.
- [ ] Repository baru hanya memiliki branch `main`.
- [ ] Repository lama tetap aman dan tidak berubah.

---

## Hasil Akhir

Repository sumber tetap memiliki:

```text
main
sub
```

Repository baru hanya memiliki:

```text
main
```

beserta seluruh commit history `main`.
