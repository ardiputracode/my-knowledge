# Remote Desktop (RDP) ke PC Rumah Lewat Browser dengan Cloudflare Tunnel

Panduan ini ditulis supaya kamu bisa mengulang semuanya dari nol di kemudian hari, walaupun sudah lupa. Semua langkah diurutkan dan dijelaskan alasannya.

---

## 1. Hasil akhir yang ingin dicapai

Dari mana pun (kantor, kafe, HP, laptop orang lain), kamu cukup membuka browser, login pakai email + kode OTP, lalu desktop PC rumah muncul di tab browser. **Tidak perlu install apa-apa** di perangkat yang kamu pakai, dan **tidak perlu membuka port di router rumah**.

Cara kerjanya:

```
Browser kamu
    |
    v
Cloudflare Access (cek: apakah kamu orang yang boleh masuk?)
    |
    v
Cloudflare Tunnel  <---- container "cloudflared" di jaringan rumah
    |                      (koneksi keluar dari rumah, jadi tidak perlu buka port)
    v
PC rumah (192.168.2.254, port 3389)
```

---

## 2. Data yang dipakai (isi sesuai kondisi kamu)

| Hal | Nilai kamu |
|---|---|
| IP PC rumah | `192.168.2.254` |
| Port RDP | `3389` |
| Domain di Cloudflare | `putraserver.biz.id` |
| Subdomain untuk RDP | `rdp.putraserver.biz.id` (contoh, boleh nama lain) |
| Nama team Zero Trust | `<nama-team>` (lihat di dashboard Zero Trust) |
| Nama target | `pc-rumah` (contoh, boleh nama lain, tanpa spasi) |

Simpan tabel ini. Kalau kamu mengganti nama, ganti juga di semua langkah di bawah.

---

## 3. Istilah yang perlu diketahui

- **Cloudflare Tunnel / cloudflared**: program kecil yang berjalan di jaringan rumah dan membuat "terowongan" keluar ke Cloudflare. Dengan itu PC rumah bisa dijangkau tanpa membuka port.
- **Zero Trust**: bagian dari Cloudflare yang mengatur siapa boleh mengakses apa.
- **Access application**: "pintu" dengan penjaga. Hanya orang yang lolos aturan (policy) yang bisa lewat.
- **Policy**: aturan penjaga. Contoh: "Allow, kalau emailnya `aku@gmail.com`".
- **Target**: daftar komputer tujuan (di sini PC rumah), dikenali lewat IP.
- **Route (Private CIDR)**: petunjuk untuk tunnel bahwa IP tertentu bisa dijangkau lewat tunnel itu. `/32` artinya hanya satu IP saja.
- **App Launcher**: halaman daftar aplikasi yang boleh kamu akses, tampil sebagai kotak-kotak (tile).

---

## 4. Syarat sebelum mulai

1. Akun Cloudflare dengan **domain yang aktif** (di sini `putraserver.biz.id`). Tanpa domain, cara browser ini tidak bisa dipakai.
2. Akun **Zero Trust** dengan paket **Free** (gratis sampai 50 pengguna).
3. Satu mesin di jaringan rumah yang selalu menyala untuk menjalankan `cloudflared` (di sini container Docker). Mesin ini harus bisa menjangkau `192.168.2.254:3389`.
4. PC rumah memakai **Windows Pro atau Enterprise** (Windows Home tidak mendukung).
5. Akun Windows di PC rumah **punya password** (bukan hanya PIN).

---

## 5. Langkah-langkah setup

### Langkah A. Siapkan Windows di PC rumah

1. Aktifkan Remote Desktop: *Settings → System → Remote Desktop → On*.
2. Pastikan akun yang akan dipakai login ada di daftar yang boleh RDP (akun administrator biasanya otomatis boleh).
3. Pastikan akun itu punya password.
4. Atur **security layer** ke Negotiate atau SSL (kalau tidak, koneksi gagal):
   - Tekan `Win + R`, ketik `gpedit.msc`, Enter.
   - Buka: *Computer Configuration → Administrative Templates → Windows Components → Remote Desktop Services → Remote Desktop Session Host → Security*.
   - Buka **Require use of specific security layer for remote (RDP) connections** → pilih **Enabled** → Security Layer: **Negotiate** (atau SSL) → OK.
   - Buka Command Prompt sebagai admin, jalankan `gpupdate /force`.
5. Firewall Windows harus mengizinkan RDP (biasanya otomatis saat RDP diaktifkan).

### Langkah B. Buat Tunnel dan jalankan cloudflared

1. Login ke dashboard Zero Trust: https://one.dash.cloudflare.com
2. Buka *Networks → Tunnels → Create a tunnel → Cloudflared*.
3. Beri nama, lalu simpan.
4. Cloudflare menampilkan perintah instalasi lengkap dengan token. Pilih **Docker**, salin perintahnya, dan jalankan di mesin rumah. Bentuknya kira-kira:

   ```
   docker run -d --name cloudflared --restart unless-stopped cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <TOKEN-KAMU>
   ```
   (Pakai perintah yang diberikan dashboard, jangan ketik manual tokennya. Token itu rahasia.)

5. Tunggu sampai status tunnel di dashboard menjadi **Healthy**.

### Langkah C. Tambah route ke PC rumah

1. Buka tunnel tadi → tab **Routes** (atau **Add a route**).
2. Pilih **Private CIDR** (di dokumentasi disebut juga *Tunnel CIDR*).
3. Isi: `192.168.2.254/32` lalu simpan.

> Pilih **Private CIDR**, bukan "Published application". Published application untuk web biasa (HTTP), sedangkan RDP bukan HTTP.

### Langkah D. Buat Target

1. Buka *Access controls → Targets → Add a target*.
2. **Target hostname**: `pc-rumah` (tanpa spasi).
3. **IP addresses**: `192.168.2.254`, pilih dari dropdown (virtual network: default).
4. Simpan.

Kalau IP tidak muncul di dropdown, berarti route di Langkah C belum benar.

### Langkah E. Buat DNS record

1. Buka dashboard Cloudflare (bukan Zero Trust) → pilih domain `putraserver.biz.id` → **DNS → Records → Add record**.
2. Isi:
   - **Type**: `A`
   - **Name**: `rdp`
   - **IPv4 address**: `240.0.0.0`
   - **Proxy status**: **Proxied** (awan oranye, harus nyala)
3. Simpan.

Angka `240.0.0.0` hanya pengisi supaya record sah. Lalu lintas sebenarnya diatur Cloudflare.

### Langkah F. Buat Access application

1. Di Zero Trust: *Access controls → Applications → Create new application*.
2. Pilih tab **Self-hosted and private**, lalu sub-tab **Public DNS** (bukan *Private destinations*, itu untuk cara WARP). Klik **Continue with Self-hosted and private**.
3. Isi:
   - **Add public hostname**: pilih domain `putraserver.biz.id`, subdomain `rdp`.
   - Nyalakan **Allow access through browser-based RDP, SSH, or VNC sessions**, pilih **RDP**.
   - **Target criteria**: pilih target `pc-rumah`, **Port** `3389`.
   - **Policy**: buat policy dengan aksi **Allow**, aturan **Include → Emails** = emailmu. (Hanya Allow atau Block yang didukung.)
   - **Authenticate with Cloudflare One Client**: biarkan **mati**.
   - **Show application in App Launcher**: biarkan **nyala**.
4. Klik **Create**.

### Langkah G. (Disarankan) Policy Gateway

1. Buka *Traffic policies → Firewall policies → Network → Add a policy*.
2. Kondisi: **Access Infrastructure Target** *is* **Present**, aksi **Allow**.
3. Pastikan **Enforce Cloudflare One Client session duration** dalam keadaan **mati**, kalau tidak akses bisa terblokir.

### Langkah H. (Opsional) Aktifkan App Launcher

Kalau saat membuka `https://<nama-team>.cloudflareaccess.com` muncul pesan *"contact your administrator to enable the Access App Launcher"*:

1. Buka *Access controls → Access settings → App Launcher* → **Manage** → nyalakan.
2. Pastikan login method **One-time PIN** aktif.
3. Buat **App Launcher policy**: **Allow**, aturan **Emails** = emailmu.
4. Simpan.

Kalau kamu tidak mau repot, lewati langkah ini dan pakai **URL langsung** (lihat bagian 6).

---

## 6. Cara mengakses (setiap kali mau RDP)

### Cara 1: URL langsung (paling andal, bisa di-bookmark)

Buka di browser:

```
https://rdp.putraserver.biz.id/rdp/<vnet-id>/192.168.2.254/3389
```

- `<vnet-id>` = ID virtual network tempat route berada. Kalau tidak pernah membuat virtual network khusus, ini virtual network **default**. ID-nya bisa dilihat di dashboard (*Networking → Virtual networks*). Setelah ketemu, tulis di sini supaya tidak lupa: `____________________`

Langkahnya:
1. Buka URL di atas.
2. Masukkan **email** yang ada di policy, lalu masukkan **kode OTP** yang dikirim ke email itu.
3. Muncul layar *"Sign in to your remote desktop"*.
4. Isi **username dan password akun Windows PC rumah** (bukan password Cloudflare).
5. Desktop PC rumah muncul di browser.

### Cara 2: Lewat App Launcher

1. Buka `https://<nama-team>.cloudflareaccess.com`.
2. Login dengan email + OTP.
3. Klik tile `pc-rumah`.
4. Login Windows seperti langkah 4 di atas.

Cara 2 hanya bekerja kalau Langkah H sudah dilakukan dan **Show application in App Launcher** menyala.

### Isi kolom username Windows

- Akun lokal: `namauser` atau `.\namauser`
- Akun Microsoft: email akunnya, dan pakai **password akun Microsoft**, bukan PIN
- Akun domain (jaringan perusahaan): `DOMAIN\namauser`
- Lupa nama akun? Di PC itu, buka Command Prompt dan ketik `whoami`. Hasilnya `NAMAPC\namauser`, ambil bagian `namauser`.

---

## 7. Kalau ada masalah

| Gejala | Penyebab umum | Solusi |
|---|---|---|
| *"Please contact your administrator to enable the Access App Launcher"* | App Launcher belum diaktifkan | Lakukan Langkah H, atau pakai URL langsung |
| *"You do not have permission to connect to any applications"* | Email login tidak cocok dengan policy Allow, atau aplikasi tidak ditampilkan di App Launcher | Samakan email persis dengan di policy; cek **Show application in App Launcher**; cek policy App Launcher; login ulang lewat jendela incognito |
| Halaman Access menolak akses | Policy aplikasi RDP salah atau tidak terpasang | Buka aplikasi → tab Policies → pastikan ada **Allow** dengan emailmu |
| Target `192.168.2.254` tidak muncul di dropdown | Route tunnel belum ada | Ulangi Langkah C |
| Login Windows gagal | Username/password salah, akun tanpa password, atau akun belum boleh RDP | Cek `whoami`, pakai password (bukan PIN), cek daftar Remote Desktop users |
| Koneksi gagal / layar hitam setelah login | Security layer RDP di Windows tidak cocok, RDP belum aktif, atau Windows Home | Ulangi Langkah A |
| Tidak bisa terhubung ke PC sama sekali | Container `cloudflared` tidak jalan atau tidak bisa menjangkau `192.168.2.254:3389` | `docker ps` untuk cek container, cek status tunnel Healthy, cek dari mesin container: `nc -zv 192.168.2.254 3389` |
| Tunnel status Down | Container mati atau tidak ada internet di mesin rumah | `docker logs cloudflared`, restart container |

Catatan: kalau `cloudflared` berjalan di Docker Desktop di PC target itu sendiri, IP `192.168.2.254` kadang tidak bisa dijangkau dari dalam container. Solusinya jalankan `cloudflared` di mesin lain di jaringan yang sama, atau install sebagai service Windows.

---

## 8. Biaya

- **Zero Trust**: paket **Free** (nama di dashboard: *Teams Free Base*) gratis sampai 50 pengguna.
- **Cloudflare Workers** dan **paket domain** di Cloudflare: Free.
- Yang bisa berbayar: **perpanjangan domain** `putraserver.biz.id` di registrar tempat membelinya, bukan di Zero Trust.
- Hindari: tombol **Upgrade** (Workers Paid $5/bulan), dan add-on berbayar di Zero Trust.
- Cek berkala di dashboard Cloudflare bagian **Billing → Subscriptions** bahwa semuanya masih Free.

---

## 9. Keterbatasan RDP lewat browser

- **Audio (suara/mic) tidak jalan.**
- Ukuran clipboard (copy-paste) maksimal sekitar 500 KB, dan copy-paste otomatis hanya di browser berbasis Chromium (Chrome, Edge, Brave).
- Kurang cocok untuk pemakaian visual berat. Untuk itu pakai Parsec atau klien RDP biasa.

---

## 10. Alternatif tanpa domain: WARP

Kalau tidak punya domain, atau mau pengalaman RDP seperti di jaringan lokal:

1. Lakukan Langkah B dan C (tunnel dan route `192.168.2.254/32`).
2. Di Zero Trust: *Settings → WARP Client → Device settings → Split Tunnels*. Pastikan `192.168.2.254` tidak ikut dikecualikan (mode Exclude), atau tambahkan (mode Include).
3. Install aplikasi **Cloudflare WARP** di perangkat yang dipakai, login ke Zero Trust organisasi kamu, aktifkan.
4. Buka aplikasi Remote Desktop dan konek ke `192.168.2.254`.

Kekurangannya: harus install WARP di setiap perangkat. Kalau jaringan tempat kamu mengakses juga memakai `192.168.2.x`, alamatnya bentrok dan perlu salah satu subnet diganti.

---

## 11. Checklist cepat saat setup ulang

- [ ] Windows: RDP aktif, akun punya password, security layer Negotiate/SSL, edisi Pro
- [ ] Tunnel dibuat, container `cloudflared` jalan, status **Healthy**
- [ ] Route **Private CIDR** `192.168.2.254/32`
- [ ] Target `pc-rumah` dengan IP `192.168.2.254`
- [ ] DNS record `rdp` tipe A, `240.0.0.0`, **Proxied**
- [ ] Access application: Public DNS, RDP browser aktif, target + port 3389, policy **Allow** emailmu
- [ ] (Disarankan) Gateway policy *Access Infrastructure Target is Present → Allow*
- [ ] (Opsional) App Launcher diaktifkan + policy-nya
- [ ] Tes: buka URL langsung → login OTP → login Windows
- [ ] Cek **Billing**: semua paket Free
