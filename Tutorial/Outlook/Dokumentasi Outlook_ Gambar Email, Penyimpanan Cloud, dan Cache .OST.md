# Dokumentasi Outlook: Gambar Email, Penyimpanan Cloud, dan Cache `.OST`

## 1. Gambar pada Email Tidak Otomatis Muncul

Pada Microsoft Outlook, gambar dalam email terkadang tidak langsung ditampilkan dan muncul pilihan seperti **“Download pictures”**.

Hal ini umumnya terjadi karena gambar merupakan **external/remote image**, yaitu gambar yang berada di server pengirim dan perlu diambil melalui internet.

Outlook dapat memblokir gambar eksternal untuk:

- Menjaga privasi pengguna.
- Mencegah email mengetahui bahwa email sudah dibuka.
- Mengurangi risiko tracking pixel.
- Menghindari pengambilan konten dari server yang tidak tepercaya.

Jika ingin gambar otomatis muncul, pengaturan **Automatic Download** atau pengaturan gambar eksternal dapat disesuaikan, tergantung versi Outlook yang digunakan.

---

## 2. Cara Mengetahui Email Tersimpan di Cloud atau Lokal

Pada Outlook Windows, buka:

**File → Account Settings → Account Settings**

Kemudian periksa jenis akun pada tab **Email**.

### Microsoft 365 / Exchange

Jika akun menggunakan **Microsoft 365 atau Exchange**:

- Email utama tersimpan di **server/cloud**.
- Outlook membuat cache lokal agar email dapat digunakan dengan lebih cepat dan sebagian tetap dapat diakses ketika offline.
- Cache tersebut biasanya menggunakan file **`.ost`**.
- Perubahan seperti membaca, menghapus, atau mengirim email akan disinkronkan dengan server.

### IMAP

Untuk akun **IMAP**:

- Email utama tetap berada di server.
- Outlook menyimpan cache/salinan lokal.
- File yang digunakan juga dapat berupa **`.ost`**, tergantung konfigurasi dan versi Outlook.

### POP

Untuk akun **POP**:

- Email biasanya diunduh ke komputer.
- Penyimpanan dapat menggunakan file **`.pst`**.
- Apakah salinan tetap berada di server atau tidak tergantung konfigurasi akun.

---

## 3. Arti File `.OST`

Jika pada:

**File → Account Settings → Data Files**

terlihat file dengan ekstensi:

```text
.ost
```

maka biasanya Outlook sedang menggunakan **Offline Storage Table (OST)**.

`.ost` berfungsi sebagai **cache/salinan lokal** dari mailbox yang berada di server.

Contohnya:

```text
Cloud / Server
      ↕
   Sinkronisasi
      ↕
Laptop
  └── Outlook
       └── file .ost
```

Dengan demikian, keberadaan file `.ost` **tidak berarti email hanya tersimpan di laptop**.

Untuk akun Microsoft 365/Exchange, sumber utama mailbox tetap berada di server.

---

## 4. Berapa Lama Email Disimpan di Cache `.OST`?

Outlook dapat membatasi berapa lama email disimpan secara offline di komputer.

Pada Outlook Windows klasik, pengaturannya biasanya dapat ditemukan melalui:

**File → Account Settings → Account Settings → pilih akun → Change**

Kemudian cari:

**Mail to keep offline**

Pilihan yang tersedia dapat berbeda tergantung versi dan konfigurasi Outlook, misalnya:

- 1 month
- 3 months
- 6 months
- 1 year
- 2 years
- All

### Contoh

Misalnya pengaturannya:

```text
Mail to keep offline: 1 year
```

Artinya Outlook akan menyimpan sekitar **1 tahun email terakhir secara offline/cache di komputer**.

Email yang lebih lama **tidak otomatis dihapus dari cloud**.

Email tersebut tetap dapat berada di mailbox server dan dapat diakses melalui Outlook Web atau ketika Outlook mengambilnya dari server.

---

## 5. Jika Memilih “All”

Jika tersedia dan dipilih:

```text
Mail to keep offline: All
```

Outlook akan berusaha menyimpan seluruh mailbox yang tersedia dalam cache `.ost`.

Keuntungannya:

- Email lama lebih mudah diakses ketika offline.
- Pencarian email lokal dapat lebih lengkap.
- Outlook tidak perlu mengambil kembali banyak email dari server saat dibutuhkan.

Kekurangannya:

- Ukuran file `.ost` dapat menjadi sangat besar.
- Membutuhkan lebih banyak ruang penyimpanan di laptop.
- Sinkronisasi awal dapat membutuhkan waktu lebih lama.

---

## 6. `.OST` vs `.PST`

| File | Fungsi umum | Cloud/Server |
|---|---|---|
| `.ost` | Cache/salinan offline mailbox | Umumnya mailbox utama tetap di server |
| `.pst` | Penyimpanan data Outlook lokal/arsip | Dapat menjadi penyimpanan lokal |

**Catatan:** ekstensi file saja tidak selalu cukup untuk menentukan lokasi utama email. Jenis akun dan konfigurasi Outlook juga perlu diperiksa.

---

## 7. Cara Memastikan Email Benar-Benar Ada di Cloud

Cara paling aman adalah melakukan pengecekan melalui **Outlook Web**.

Jika email yang sama dapat ditemukan ketika login ke mailbox melalui browser, berarti email tersebut tersedia di server/cloud.

Secara sederhana:

```text
Email terlihat di Outlook Desktop
            ↓
Apakah terlihat juga di Outlook Web?
            ↓
          YA
            ↓
Email tersedia di server/cloud
```

Jika email hanya terlihat di Outlook Desktop tetapi tidak ada di Outlook Web, perlu diperiksa lebih lanjut apakah email tersebut berada di folder lokal, archive, `.pst`, atau konfigurasi lain.

---

## 8. Kesimpulan

Jika Outlook kamu menggunakan file **`.ost`**, kemungkinan besar:

1. **Email utama berada di server/cloud.**
2. **`.ost` adalah cache lokal Outlook.**
3. Cache digunakan agar Outlook dapat bekerja lebih cepat dan sebagian tetap dapat digunakan saat offline.
4. Lama email yang disimpan dalam cache ditentukan oleh **Mail to keep offline**.
5. Mengubah periode cache **tidak sama dengan menghapus email dari cloud**.
6. Untuk memastikan email benar-benar berada di cloud, periksa mailbox melalui **Outlook Web**.

### Ringkasan

```text
Microsoft 365 / Exchange
        │
        ▼
   Cloud / Server
        │
        │ Sinkronisasi
        ▼
 Outlook di Laptop
        │
        ▼
      .OST
   (cache lokal)
```