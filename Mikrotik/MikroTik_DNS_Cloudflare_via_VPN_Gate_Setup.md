# MikroTik — Cloudflare DNS via VPN Gate (OpenVPN)

> **Dokumentasi konfigurasi dan panduan instal ulang**  
> Tanggal dokumentasi: **17 Oktober 2026**  
> Perangkat saat pengujian: **MikroTik hAP ac³**  
> RouterOS saat pengujian: **7.26beta1** (ARM)  
> Status: **OpenVPN aktif, DNS utama `1.1.1.1`, dan static route via VPN terverifikasi.** Blackhole cadangan telah dipandu tetapi hasil `route print` final belum tercatat dalam percakapan.

---

## 1. Tujuan dan batasan

Menggunakan **Cloudflare DNS `1.1.1.1`** sebagai upstream resolver MikroTik, tetapi **jalur IP ke `1.1.1.1` melewati OpenVPN VPN Gate**. Trafik internet lainnya tetap lewat ISP, sehingga tidak semua koneksi perangkat masuk ke VPN.

```text
Perangkat LAN (DNS = IP router)
    │
    ▼
DNS resolver MikroTik
    │  query ke 1.1.1.1 (UDP/TCP 53)
    ▼
Static route 1.1.1.1/32
    │
    ▼
Interface OpenVPN VPN Gate (TCP 443)
    │
    ▼
Server VPN Gate → Cloudflare 1.1.1.1

Trafik web normal perangkat ───────────► ISP (default route biasa)
```

**Batasan penting:**

- Ini merupakan **DNS-only routing**, **bukan full-tunnel VPN**. Pemblokiran berdasarkan alamat IP/SNI atau metode non-DNS masih mungkin terjadi.
- Route `1.1.1.1/32` sebenarnya berlaku untuk **semua trafik IPv4** menuju alamat tersebut (bukan hanya port 53).
- Perangkat yang menggunakan DNS lain, DNS-over-HTTPS sendiri, DNS-over-TLS, atau IPv6 **tidak otomatis** memakai jalur ini.
- Trafik DNS terenkripsi **di dalam tunnel OpenVPN** sampai server VPN; setelah keluar dari server VPN menuju Cloudflare, DNS biasa pada port 53 tidak otomatis terenkripsi. Operator VPN Gate publik tetap menjadi pihak yang harus dipercaya.
- Konfigurasi DNS Guest **belum dibuktikan terpisah**. DHCP Guest menggunakan IP MikroTik, dan tanpa kebijakan tambahan akan memakai resolver upstream MikroTik yang sama.

---

## 2. Data konfigurasi yang berhasil diuji

| Parameter | Nilai |
|---|---|
| Model router | MikroTik hAP ac³ |
| RouterOS | 7.26beta1 |
| RAM | 256 MiB |
| Server VPN Gate saat pengujian | `219.100.37.206` |
| Nama interface hasil import | `ovpn-import1792209651` |
| Protokol transport VPN | TCP port `443` |
| Mode | OpenVPN TUN (`mode=ip`) |
| Cipher / auth | `aes128-cbc` / `sha1` |
| Pengguna VPN Gate | `vpn` (ikuti instruksi server untuk instal ulang) |
| `use-peer-dns` | `no` |
| `add-default-route` | `no` |
| `route-nopull` | `yes` |
| DNS utama RouterOS | `1.1.1.1` |
| DNS-over-HTTPS pada RouterOS | Tidak digunakan (`use-doh-server=""`) |
| DHCP DNS untuk client | IP gateway/router di subnet masing-masing |
| Static route | `1.1.1.1/32` via interface OpenVPN, `distance=1` |
| Blackhole fail-closed | `1.1.1.1/32`, `distance=254` (cek hasil print aktual) |

**Hasil pengujian saat setup:**

- `/interface ovpn-client print detail` menunjukkan flag **`R` (RUNNING)** dan **`H` (HW-CRYPTO)**.
- `/ip route print detail where dst-address=1.1.1.1/32` menunjukkan route OpenVPN **aktif (`As`)**, `distance=1`.
- Ping ke `1.1.1.1`: **5/5 reply, packet loss 0%, rata-rata ±321 ms**.
- Traceroute menampilkan hop `10.245.254.254` → `219.100.37.253` → jaringan transit → `1.1.1.1`, yang mendukung bahwa jalur menuju tujuan melewati VPN Gate.

> **Catatan:** `219.100.37.206` dan nama interface merupakan **contoh dari konfigurasi yang berhasil saat itu**, bukan alamat/nama permanen. Server publik VPN Gate dapat berubah atau tidak tersedia.

---

## 3. Persiapan sebelum instal ulang

1. Login MikroTik menggunakan **WinBox** atau Terminal.
2. Pastikan akses administrator dan internet router normal.
3. Simpan export konfigurasi sebelum mengubah jaringan:

   ```routeros
   /export file=before-vpngate
   ```

4. Unduh file konfigurasi `.ovpn` **yang baru** dari [VPN Gate](https://www.vpngate.net/en/) — pilih server dengan **OpenVPN TCP 443** dan mode TUN yang kompatibel. Situs VPN Gate dapat memberikan nama file/server/IP yang berbeda.
5. Upload file, misalnya `vpngate.ovpn`, ke **WinBox → Files**.

**Keamanan backup:** file `.rsc` dapat memuat informasi jaringan yang sensitif. Simpan backup secara privat dan jangan kirim export dengan data kredensial ke publik.

---

## 4. Impor profil OpenVPN

Cek nama file yang sudah diupload:

```routeros
/file print
```

Contoh impor profil yang digunakan pada setup ini:

```routeros
/interface ovpn-client import-ovpn-configuration file-name=vpngate.ovpn ovpn-user=vpn ovpn-password=vpn skip-cert-import=no
```

> Username/password `vpn` merupakan contoh dari layanan VPN Gate yang dipakai saat pengujian. Pastikan kembali nilai yang diminta layanan ketika melakukan instal ulang.

Temukan interface hasil impor:

```routeros
/interface ovpn-client print detail
```

Pada konfigurasi lama namanya `ovpn-import1792209651`. **Gunakan nama terbaru yang benar-benar muncul setelah impor.**

Agar VPN tidak mengambil alih default route dan DNS:

```routeros
/interface ovpn-client set [find name="ovpn-import1792209651"] add-default-route=no route-nopull=yes use-peer-dns=no
```

Jika interface belum aktif:

```routeros
/interface ovpn-client enable [find name="ovpn-import1792209651"]
```

Periksa koneksi:

```routeros
/interface ovpn-client print detail
/log print where topics~"ovpn"
```

**Indikator berhasil:** terdapat flag `R`/`RUNNING` pada interface tersebut. Jangan lanjut mengalihkan DNS sebelum tunnel berhasil.

---

## 5. Buat route khusus Cloudflare melalui VPN

Periksa terlebih dahulu supaya tidak membuat route duplikat:

```routeros
/ip route print detail where dst-address=1.1.1.1/32
```

Jika belum ada route VPN untuk tujuan tersebut, buat:

```routeros
/ip route add dst-address=1.1.1.1/32 gateway=ovpn-import1792209651 distance=1 comment="Cloudflare DNS via VPN Gate"
```

> Ganti `ovpn-import1792209651` sesuai nama interface aktual. Route ini tetap menggunakan `routing-table=main` pada konfigurasi yang diuji.

Verifikasi:

```routeros
/ip route print detail where dst-address=1.1.1.1/32
```

Hasil yang diharapkan: route `1.1.1.1/32` aktif (`A`) via interface VPN dengan `distance=1`.

---

## 6. Atur DNS upstream MikroTik

Konfigurasi yang dipakai:

```routeros
/ip dns set servers=1.1.1.1 use-doh-server="" allow-remote-requests=yes
```

Verifikasi:

```routeros
/ip dns print
```

Nilai kunci:

```text
servers:                 1.1.1.1
use-doh-server:          [kosong]
allow-remote-requests:   yes
```

`verify-doh-cert=yes` boleh tetap terlihat pada output, tetapi **tidak digunakan** karena `use-doh-server` kosong.

**Penting:** perintah di atas **menggantikan DoH upstream yang sebelumnya diatur**, termasuk jika sebelumnya memakai Cloudflare Gateway khusus. Dokumentasikan URL konfigurasi DoH yang lama sebelum menggantinya, jika suatu saat ingin kembali.

**Firewall:** ketika `allow-remote-requests=yes`, pastikan permintaan DNS **dari WAN** ke TCP/UDP port 53 diblokir oleh firewall. Jangan sampai router menjadi *open resolver* publik. Periksa aturan input yang sudah ada sebelum menambah rule baru.

---

## 7. DNS fail-closed: jangan fallback ke ISP saat VPN terputus

Tujuan: apabila route OpenVPN ke `1.1.1.1` tidak lagi aktif, tujuan IP tersebut dibuang (*blackhole*) alih-alih mengikuti default route ISP.

Cek lebih dahulu apakah blackhole sudah ada:

```routeros
/ip route print detail where dst-address=1.1.1.1/32
```

Jika belum ada, tambahkan **dengan sintaks switch RouterOS v7** (satu baris):

```routeros
/ip route add dst-address=1.1.1.1/32 blackhole distance=254 comment="DNS VPN fail-closed"
```

**Jangan gunakan** dua sintaks yang sempat menimbulkan error:

```text
SALAH: type=blackhole
SALAH: blackhole=yes
BENAR: blackhole
```

Verifikasi:

```routeros
/ip route print detail where dst-address=1.1.1.1/32
```

Idealnya terdapat dua route untuk tujuan yang sama:

| Kondisi | Route terpilih | Distance | Dampak |
|---|---|---:|---|
| VPN tersambung, route tersedia | Via OpenVPN | 1 | Query upstream dapat lewat VPN |
| Route VPN tidak tersedia | Blackhole | 254 | Tujuan `1.1.1.1` dibuang, tidak dialihkan ke default route ISP |

> Blackhole ini hanya menangani trafik dalam routing table terkait ke alamat **`1.1.1.1/32`**. Ini bukan perlindungan universal untuk resolver lain, IPv6, DNS yang tidak melalui router, atau kebocoran di luar konfigurasi ini. Jika VPN putus, query DNS yang tidak ada di cache berpotensi gagal sampai VPN pulih.

---

## 8. Verifikasi setelah setup

Jalankan berurutan:

**A. VPN RUNNING**

```routeros
/interface ovpn-client print detail
```

**B. Route Cloudflare**

```routeros
/ip route print detail where dst-address=1.1.1.1/32
```

**C. DNS upstream**

```routeros
/ip dns print
```

**D. Ping dan traceroute ke `1.1.1.1`**

```routeros
/ping 1.1.1.1 count=5
/tool traceroute 1.1.1.1
```

Hop menuju jaringan VPN Gate pada traceroute mengindikasikan route VPN dipakai. Angka ping/traceroute bukan jaminan seluruh query client menggunakan DNS router.

**E. Uji resolver MikroTik**

```routeros
:put [:resolve "example.com"]
/ip dns cache print where name~"example.com"
```

**F. Uji dari komputer LAN** (contoh Windows):

```powershell
nslookup example.com 192.168.1.1
```

Saat mengetes lewat Torch, ingat query bisa dijawab dari cache, sehingga trafik ke upstream tidak selalu muncul:

```routeros
/tool torch interface=ovpn-import1792209651
```

Untuk lebih yakin, gunakan nama domain baru yang belum tercache dan verifikasi paket menuju `1.1.1.1` di interface VPN. Hindari menghapus cache sembarangan saat jaringan aktif jika tidak diperlukan.

---

## 9. DHCP DNS per subnet (hasil inspeksi)

| Subnet | Gateway DHCP | DNS yang diterima client |
|---|---|---|
| `192.168.1.0/24` | `192.168.1.1` | `192.168.1.1` |
| `192.168.2.0/24` | `192.168.2.1` | `192.168.2.1` |
| `192.168.3.0/24` | `192.168.3.1` | `192.168.3.1` |
| `192.168.4.0/24` (**Guest**) | `192.168.4.1` | `192.168.4.1` |

Semua alamat tersebut merupakan alamat resolver pada MikroTik. **Alamat berbeda pada DHCP bukan berarti upstream DNS berbeda.** Tanpa kebijakan tambahan, query dari seluruh subnet yang masuk ke resolver yang sama akan menggunakan DNS upstream `1.1.1.1` ini.

Jika ingin Guest tetap memakai Cloudflare Families / filtering terpisah, **perlu audit** NAT, firewall, DNS forwarders, atau mekanisme lain yang tersedia pada instalasi aktual. Tidak ada hasil verifikasi akhir di percakapan yang membuktikan Guest masih menggunakan DNS filtering khusus.

Perintah audit:

```routeros
/ip dhcp-server network print detail
/ip firewall nat print detail
/ip firewall mangle print detail
/ip dns forwarders print detail
```

Jangan mengasumsikan redirect DNS Guest aktif hanya dari DHCP.

---

## 10. Troubleshooting cepat

| Gejala | Kemungkinan | Pemeriksaan |
|---|---|---|
| Import `.ovpn` gagal | Opsi enkripsi/TLS atau sertifikat profil tidak kompatibel | `/log print where topics~"ovpn"`; cek profil baru |
| Interface tidak `R` | Server publik mati, akses TCP 443 bermasalah, autentikasi gagal | `/interface ovpn-client print detail`; `/log print` |
| `1.1.1.1` tidak lewat VPN | Route tidak ada/tidak aktif/nama interface berubah | `/ip route print detail where dst-address=1.1.1.1/32` |
| DNS gagal setelah VPN putus | Efek fail-closed yang memang diinginkan | Periksa status VPN/route dan pulihkan VPN |
| DNS berbeda dari harapan | DoH masih terpasang atau client memakai DNS sendiri | `/ip dns print`; `nslookup` dari client |
| Latensi DNS tinggi | Rute/server VPN Gate publik lambat atau tidak stabil | Uji ping, traceroute, dan waktu lookup |
| Error `bad parameter type` | Menulis `type=blackhole` | Gunakan switch `blackhole` |
| Error `expected end of command` | Menulis `blackhole=yes` | Gunakan `blackhole` tanpa `=yes` |
| Baris `distance=254` error setelahnya | Perintah multiline sebelumnya terhenti akibat sintaks salah | Gunakan contoh perintah blackhole **satu baris** |

**Peringatan keamanan:** konfigurasi hasil pengujian menunjukkan `verify-server-certificate=no`. Ini berarti identitas server OpenVPN **tidak diverifikasi** sebagaimana semestinya. Server relay VPN Gate bersifat publik, cipher/auth profil ini tergolong lama, dan server dapat berganti kapan saja. Jangan jadikan konfigurasi ini jalur tepercaya untuk aktivitas atau data sensitif. Pertimbangkan layanan VPN dengan sertifikat server tervalidasi atau Cloudflare DoH langsung jika tujuan utamanya hanya integritas/privasi DNS.

**RouterOS beta:** karena saat pengujian menggunakan `7.26beta1`, perilaku atau sintaks dapat berubah di rilis mendatang. Saat instal ulang, cek dokumentasi RouterOS sesuai versi aktual.

---

## 11. Rollback / hapus konfigurasi VPN DNS

Jika ingin mengembalikan koneksi DNS ke ISP atau setup DoH sebelumnya, **siapkan DNS pengganti terlebih dahulu** agar internet tidak kehilangan resolusi nama. Contoh penggantian upstream DNS langsung (tanpa VPN):

```routeros
/ip dns set servers=1.1.1.1 use-doh-server="" allow-remote-requests=yes
```

**Perhatian:** sebelum route VPN dan blackhole dihapus, query ke `1.1.1.1` mungkin masih mengikuti route yang ada. Jika ingin kembali menggunakan DoH lama, ganti `use-doh-server` dengan URL DoH milikmu dari backup konfigurasi; jangan menebak URL-nya.

Hapus hanya route yang **berkomentar persis** seperti setup ini:

```routeros
/ip route remove [find where comment="Cloudflare DNS via VPN Gate"]
/ip route remove [find where comment="DNS VPN fail-closed"]
```

Lalu, jika sudah tidak dipakai layanan lain, nonaktifkan interface VPN:

```routeros
/interface ovpn-client disable [find name="ovpn-import1792209651"]
```

Periksa kondisi akhir:

```routeros
/ip dns print
/ip route print detail where dst-address=1.1.1.1/32
/interface ovpn-client print detail
```

**Catatan:** perintah rollback di atas menghapus perlindungan khusus `1.1.1.1` dan membuat DNS langsung ke upstream melalui jalur standar. Untuk mempertahankan privasi DNS tanpa VPN publik, konfigurasi DoH dengan verifikasi sertifikat dapat menjadi alternatif. Jangan hapus sertifikat yang terpasang sebelum memastikan tidak digunakan interface/aplikasi lain.

---

## 12. Checklist konfigurasi berhasil

- [ ] RouterOS dan model router dicatat.
- [ ] Backup/export konfigurasi disimpan.
- [ ] File `.ovpn` diunduh dari server tepercaya dan diupload.
- [ ] Interface OVPN tampil dengan flag `R`.
- [ ] `use-peer-dns=no`, `route-nopull=yes`, `add-default-route=no`.
- [ ] Route `1.1.1.1/32` via VPN aktif dengan `distance=1`.
- [ ] Resolver MikroTik memakai `servers=1.1.1.1`, `use-doh-server` kosong.
- [ ] Blackhole route `1.1.1.1/32 distance=254` ada dan diverifikasi.
- [ ] Traceroute ke `1.1.1.1` melewati tunnel VPN.
- [ ] Lookup DNS dari MikroTik dan client LAN berhasil.
- [ ] DNS dari WAN tidak dibuka untuk publik.
- [ ] Konfigurasi khusus Guest dan DNS lain/IPv6 diaudit jika diperlukan.
- [ ] Risiko `verify-server-certificate=no` dievaluasi sebelum penggunaan permanen.

---

## 13. Referensi resmi

- [MikroTik RouterOS — OpenVPN](https://help.mikrotik.com/docs/spaces/ROS/pages/2031655/OpenVPN)
- [MikroTik RouterOS 7.26 — Route CLI Reference](https://manual.mikrotik.com/docs/cli-reference/ip/route/)
- [MikroTik RouterOS 7.26 — DNS CLI Reference](https://manual.mikrotik.com/docs/cli-reference/ip/dns/)
- [MikroTik RouterOS — DNS](https://manual.mikrotik.com/docs/network-management/dns/)
- [VPN Gate — daftar server dan download OpenVPN](https://www.vpngate.net/en/)

---

*Dokumentasi dibuat berdasarkan konfigurasi RouterOS dan hasil pengujian yang dibagikan pada 17 Oktober 2026. Selalu verifikasi nama interface, alamat server VPN, dan kebijakan DNS Guest sebelum menerapkan ulang.*
