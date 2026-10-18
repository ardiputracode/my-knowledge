# MikroTik hAP — Family Friendly + Anti-VPN (Guest 192.168.4.0/24)

> **Dokumentasi pribadi / panduan konfigurasi ulang**  
> **Versi dokumen:** 1.0 — 16 Oktober 2026  
> **Platform:** MikroTik RouterOS v7  
> **Target:** seluruh perangkat pada `192.168.4.0/24` melalui `wifi1-guest`  
> **Pilihan utama:** DNS Guest tetap memakai **Cloudflare Gateway custom milik sendiri** lewat DNS MikroTik  
> **Alternatif:** Cloudflare Families publik `1.1.1.3`  
> **FastTrack:** **tidak dimatikan**  
> **Status:** *runbook berdasarkan output konfigurasi yang dibagikan; pemasangan seluruh rule tambahan belum diverifikasi*. Jangan menganggap semua command di dokumen ini sudah aktif.

---

## Daftar isi

1. [Tujuan dan batasan](#1-tujuan-dan-batasan)
2. [Snapshot konfigurasi awal](#2-snapshot-konfigurasi-awal)
3. [Cara kerja DNS — hal terpenting](#3-cara-kerja-dns--hal-terpenting)
4. [Prasyarat, audit, backup](#4-prasyarat-audit-backup)
5. [Instalasi tahap demi tahap](#5-instalasi-tahap-demi-tahap)
6. [FastTrack tanpa dimatikan](#6-fasttrack-tanpa-dimatikan)
7. [Pengujian](#7-pengujian)
8. [Ganti DNS di kemudian hari](#8-ganti-dns-di-kemudian-hari)
9. [Rollback dan troubleshooting](#9-rollback-dan-troubleshooting)
10. [Quick reference](#10-quick-reference)
11. [Rujukan resmi](#11-rujukan-resmi)

---

## 1. Tujuan dan batasan

**Diterapkan ke:** `192.168.4.0/24` / `wifi1-guest`. LAN dan WLAN utama tidak mendapat rule pemblokiran baru.

Tujuan:

- Memaksa DNS port 53 agar tidak bebas dipilih pengguna Guest.
- Memblokir protokol/port VPN umum (bukan semua VPN di dunia).
- Menghambat DoT, DoQ, sejumlah DoH, dan QUIC/HTTP3.
- Mengisolasi perangkat Guest dari LAN, WireGuard, dan ZeroTier.
- Melarang manajemen router dari Guest sambil **tetap mengizinkan DHCP serta DNS router**.
- Menutup jalur IPv6 terusan dari Guest.
- Membiarkan FastTrack dan pengaturan DNS WAN/LAN utama sebagaimana semula.

**Tidak dijamin:** pemblokiran 100% VPN/proxy HTTPS melalui TCP 443, semua DoH, VPN via data seluler, konten yang tidak terklasifikasi, atau SafeSearch tanpa kebijakan yang benar di Cloudflare. Pemblokiran UDP 443 dapat mengganggu aplikasi/game/panggilan tertentu. Kebijakan perangkat (parental control, pembatasan aplikasi dan perubahan Private DNS) tetap diperlukan untuk perlindungan paling ketat.

---

## 2. Snapshot konfigurasi awal

Tabel berikut **terkonfirmasi dari output CLI sebelum konfigurasi baru diverifikasi**, bukan hasil audit perangkat terkini.

| Item | Hasil yang dilaporkan |
|---|---|
| LAN utama | `192.168.2.1/24` — `bridge-lan` |
| WLAN utama | `192.168.3.1/24` — `bridge-wlan` |
| Family / Guest | `192.168.4.1/24` — `wifi1-guest` |
| WireGuard server | `10.0.0.1/24` — `wg0` |
| ZeroTier | `192.168.193.1/24` — `zerotier1` |
| WAN/default route | `pppoe-out1` |
| Interface list WAN/LAN | `/interface list member print` kosong |
| FastTrack | `/ip firewall filter print where action=fasttrack-connection` kosong |
| DNS DHCP Guest | `192.168.4.1` |
| DNS router | Cloudflare Gateway DoH custom; `allow-remote-requests=yes` |
| DNS redirect lama | Ada rule NAT global `Force DNS UDP` dan `Force DNS TCP` (`action=redirect`, port 53) |
| IPv6 | Tidak terlihat IPv6 global pada `wifi1-guest`; ada entri invalid terkait pool di `bridge-lan` |

DNS router (endpoint yang tercatat):

```text
https://n3ldlhfgyq.cloudflare-gateway.com/dns-query
```

> **Privasi:** URL DoH ini adalah endpoint khusus lokasi/organisasi. Simpan file dokumentasi sebagai konfigurasi jaringan pribadi; jangan mengunggahnya ke repositori publik tanpa menyamarkan subdomain.

Pengaturan DNS router yang terlihat saat snapshot:

```text
use-doh-server: https://n3ldlhfgyq.cloudflare-gateway.com/dns-query
verify-doh-cert: no
allow-remote-requests: yes
```

`verify-doh-cert=no` artinya validasi sertifikat upstream DoH tidak dilakukan. **Jangan langsung mengubahnya ke `yes`** tanpa memastikan trust store/sertifikat dan resolusi bootstrap bekerja. Jadwalkan penguatan keamanan DNS router secara terpisah.

**PENTING untuk setup ulang setelah reset:** output `/ip dns print` menunjukkan `servers` dan `dynamic-servers` kosong. RouterOS tetap membutuhkan mekanisme *bootstrap* untuk mengubah nama host custom DoH menjadi IP saat cache kosong atau setelah reboot. Cek apakah sudah ada entri DNS static untuk endpoint:

```routeros
/ip dns static print detail
/ip dns cache print where name~"cloudflare-gateway.com"
```

Jika tidak ada static record maupun DNS bootstrap, **jangan mengandalkan DoH akan langsung pulih setelah restart**. Salah satu opsi umum adalah menambahkan resolver bootstrap ke `/ip dns servers`, misalnya `1.1.1.1,1.0.0.1`, *setelah mengecek efeknya pada konfigurasi yang ada*. RouterOS menjelaskan bahwa selama `use-doh-server` aktif, upstream utama tetap DoH (tidak otomatis fallback ke resolver biasa saat DoH gagal). Alamat bootstrap ini **bukan** pengganti kebijakan custom Gateway; dokumentasikan pengaturan DNS static dan bootstrap aktual setelah instal ulang.

**Rule lama yang perlu dipertahankan:** NAT masquerade ke `pppoe-out1`, server WireGuard UDP `51820` pada `chain=input`, penggunaan ZeroTier, serta aturan DNS redirect global bila ingin mempertahankan perilaku DNS jaringan utama. Dari snapshot, `ip-lokal` meliputi subnet Guest; karena itu akses manajemen router dari Guest perlu dibatasi dengan rule `input` khusus.

---

## 3. Cara kerja DNS — hal terpenting

### Mode A — Disarankan: Guest memakai Cloudflare Gateway custom

```text
Perangkat Guest (192.168.4.x)
  -> DNS DHCP 192.168.4.1 (atau DNS lain di port 53)
  -> NAT redirect port 53 pada MikroTik
  -> DNS resolver MikroTik
  -> Cloudflare Gateway DoH custom
  -> Kebijakan DNS Cloudflare Gateway
```

Dalam Mode A, **Guest dan LAN/WLAN utama memakai upstream DNS custom yang sama melalui router**. Jika ingin kebijakan *khusus Guest* (berbeda dari LAN utama), satu upstream DoH router ini tidak cukup untuk membedakan klien per subnet; perlu desain DNS resolver/policy terpisah. Pastikan kategori Adult, Malware, Phishing, dan bila tersedia Anonymizer/Proxy diblokir pada Cloudflare Gateway. Cloudflare Gateway juga mendukung aksi **Safe Search** dan **YouTube Restricted Mode** melalui DNS policies.

**Konsekuensi firewall yang sangat penting:** karena DNS port 53 dialihkan ke router, `chain=input` **harus mengizinkan UDP dan TCP port 53 dari `wifi1-guest` sebelum rule Drop Guest**. Tanpa pengecualian itu, Mode A akan putus DNS.

### Mode B — Alternatif: Guest memakai Cloudflare Families publik

```text
Perangkat Guest (192.168.4.x)
  -> NAT dst-nat port 53 khusus wifi1-guest
  -> Cloudflare Families 1.1.1.3
```

Mode B tidak menggunakan DNS Gateway custom untuk kueri DNS IPv4 standar Guest; filter malware+adult berbasis Cloudflare Families publik. LAN/WLAN utama tetap menggunakan custom Gateway router. URL DoH custom milikmu **bukan** IP pengganti untuk field `to-addresses` NAT.

**Pemilihan mode:** pilih **hanya satu mode sebagai sumber DNS utama**. NAT Mode B mesti berada **di atas** `Force DNS UDP/TCP` global. Aturan firewall Anti-VPN tetap sama untuk kedua mode.

---

## 4. Prasyarat, audit, backup

**Jalankan dari WinBox yang terhubung ke LAN/WLAN utama, bukan Guest.** Sebelum menempel command, pastikan nama interface belum berubah dan cek rule eksperimen dari panduan sebelumnya:

```routeros
/system resource print
/ip address print
/ip route print
/ip dns print
/ip dns static print detail
/ip dhcp-server network print
/ip firewall nat print detail
/ip firewall filter print detail
/ip firewall filter print where action=fasttrack-connection
/ip firewall address-list print where comment~"FAMILY-"
/ip firewall filter print where comment~"FAMILY-"
/ip firewall filter print where comment~"FMX-"
/ip firewall nat print where comment~"FAMILY-"
/ip firewall nat print where comment~"FMX-"
/ipv6 address print
/ipv6 nd print
/ipv6 firewall filter print
/interface wifi print detail where name="wifi1-guest"
```

Jika sudah ada aturan `FAMILY-`/`FMX-`, **jangan jalankan ulang `add` secara membabi buta**. Bandingkan rule lama dengan runbook ini dan sesuaikan terlebih dahulu; rule duplikat, terutama aturan `input`, dapat menyebabkan salah urutan atau terputusnya DNS.

**Backup dua jenis:**

```routeros
/system backup save name=before-family-protection
/export file=before-family-protection
```

- Backup `.backup`: pemulihan sistem dari perangkat/versi yang sesuai.
- Export `.rsc`: referensi konfigurasi teks yang lebih mudah diaudit; dapat berisi informasi sensitif.
- Unduh kedua file melalui WinBox **Files** dan simpan di tempat aman.

**Safe Mode:** tekan `Ctrl+X` di Terminal WinBox (muncul `<SAFE>`). `Ctrl+D` = batalkan perubahan Safe Mode; `Ctrl+X` lagi = keluar dan simpan. Jika sesi terputus abnormal, RouterOS dapat me-rollback perubahan Safe Mode. Kerjakan per tahap karena Safe Mode memiliki batas riwayat perubahan (~100 aksi), dan jangan bergantung padanya untuk perubahan yang membutuhkan reboot.

---

## 5. Instalasi tahap demi tahap

### Tahap 1 — Pilih DNS untuk Guest

**Rekomendasi: Mode A, Cloudflare Gateway custom.** Jika rule lama `Force DNS UDP/TCP` dan `/ip dns use-doh-server` masih ada, tidak perlu mengubah DNS router maupun DHCP Guest.

Jika pernah memasang `FAMILY-DNS-UDP`/`FAMILY-DNS-TCP` yang mengarahkan ke `1.1.1.3`, nonaktifkan keduanya agar rule redirect lama mengambil alih:

```routeros
/ip firewall nat
 disable [find where comment="FAMILY-DNS-UDP"]
 disable [find where comment="FAMILY-DNS-TCP"]
```

Cek bahwa rule lama masih aktif:

```routeros
/ip firewall nat print detail where comment~"Force DNS"
/ip dns print
/ip dhcp-server network print where address=192.168.4.0/24
```

Jika sedang **memasang dari kondisi reset** dan **belum ada** redirect port 53 global, buat redirect **khusus Guest** berikut sebagai pengganti (jangan tambah jika rule yang sama sudah ada):

```routeros
/ip firewall nat
add chain=dstnat in-interface=wifi1-guest protocol=udp dst-port=53 action=redirect to-ports=53 place-before=0 comment="FAMILY-DNS-ROUTER-UDP"
add chain=dstnat in-interface=wifi1-guest protocol=tcp dst-port=53 action=redirect to-ports=53 place-before=0 comment="FAMILY-DNS-ROUTER-TCP"
```

> Jika memilih **Mode B**, lewati perubahan Mode A ini dan gunakan instruksi [Ganti DNS](#8-ganti-dns-di-kemudian-hari) untuk membuat `dst-nat` ke `1.1.1.3` sebelum redirect umum.

### Tahap 2 — Address list DoH publik yang diketahui

```routeros
/ip firewall address-list
add list=FAMILY-DOH address=1.1.1.1 comment="FAMILY-DOH-CF"
add list=FAMILY-DOH address=1.0.0.1 comment="FAMILY-DOH-CF"
add list=FAMILY-DOH address=8.8.8.8 comment="FAMILY-DOH-GOOGLE"
add list=FAMILY-DOH address=8.8.4.4 comment="FAMILY-DOH-GOOGLE"
add list=FAMILY-DOH address=9.9.9.9 comment="FAMILY-DOH-QUAD9"
add list=FAMILY-DOH address=149.112.112.112 comment="FAMILY-DOH-QUAD9"
```

Daftar hanya contoh resolver publik. Tidak menutup semua DoH; beberapa IP memuat layanan lain yang juga mungkin terblokir.

### Tahap 3 — Bangun custom firewall chain, belum aktif

Nama chain: `FAMILY-FWD`. Semua rule dibangun sebelum menambahkan `jump` ke chain ini.

```routeros
/ip firewall filter
add chain=FAMILY-FWD src-address=!192.168.4.0/24 action=drop comment="FAMILY-SPOOF"
add chain=FAMILY-FWD protocol=udp dst-port=500,4500,1701,1194,51820 action=drop comment="FAMILY-VPN-UDP"
add chain=FAMILY-FWD protocol=tcp dst-port=1194,1723 action=drop comment="FAMILY-VPN-TCP"
add chain=FAMILY-FWD protocol=gre action=drop comment="FAMILY-VPN-GRE"
add chain=FAMILY-FWD protocol=ipsec-esp action=drop comment="FAMILY-VPN-IPSEC"
add chain=FAMILY-FWD protocol=tcp dst-port=853 action=drop comment="FAMILY-DOT"
add chain=FAMILY-FWD protocol=udp dst-port=853 action=drop comment="FAMILY-DOQ"
add chain=FAMILY-FWD protocol=udp dst-port=443 action=drop comment="FAMILY-QUIC"
add chain=FAMILY-FWD protocol=tcp dst-port=443 dst-address-list=FAMILY-DOH action=drop comment="FAMILY-DOH-IP"
add chain=FAMILY-FWD protocol=tcp dst-port=443 tls-host=dns.google action=drop comment="FAMILY-SNI-GOOGLE"
add chain=FAMILY-FWD protocol=tcp dst-port=443 tls-host=cloudflare-dns.com action=drop comment="FAMILY-SNI-CLOUDFLARE"
add chain=FAMILY-FWD protocol=tcp dst-port=443 tls-host=dns.quad9.net action=drop comment="FAMILY-SNI-QUAD9"
add chain=FAMILY-FWD protocol=tcp dst-port=443 tls-host=dns.nextdns.io action=drop comment="FAMILY-SNI-NEXTDNS"
add chain=FAMILY-FWD protocol=tcp dst-port=443 tls-host=dns.adguard-dns.com action=drop comment="FAMILY-SNI-ADGUARD"
add chain=FAMILY-FWD out-interface=!pppoe-out1 action=drop comment="FAMILY-ISOLATION"
add chain=FAMILY-FWD action=return comment="FAMILY-RETURN"
```

Penjelasan urutan:

- `FAMILY-SPOOF`: cegah klien Guest mengirim alamat sumber non-Guest melalui router.
- `FAMILY-VPN-*`: blokir beberapa port/protokol VPN standar; VPN bisa berpindah port, jadi **bukan anti-VPN mutlak**.
- `FAMILY-DOT/DOQ/QUIC`: blokir DNS terenkripsi standar dan UDP 443. QUIC juga dipakai aplikasi sah.
- `FAMILY-DOH-IP`/`FAMILY-SNI-*`: upaya memblokir DoH populer; SNI dapat disembunyikan/dipecah dan tidak selalu cocok.
- `FAMILY-ISOLATION`: dari Guest hanya boleh meneruskan paket via `pppoe-out1`. Melarang akses *routed* Guest ke `bridge-lan`, `bridge-wlan`, `wg0`, `zerotier1`, dan egress lain. Akses langsung antarklien satu WiFi perlu client isolation terpisah.
- `FAMILY-RETURN`: lalu lintas yang lolos masih harus melalui rule `forward` sebelumnya yang ada setelah `jump` (bukan otomatis di-accept).

**Aktifkan chain Guest sesudah semua rule di atas ada:**

```routeros
/ip firewall filter
add chain=forward in-interface=wifi1-guest action=jump jump-target=FAMILY-FWD place-before=0 comment="FAMILY-ENTRY"
```

**Mengapa bukan `out-interface-list=WAN`?** Pada snapshot, `/interface list member print` kosong. `out-interface=pppoe-out1` lebih cocok dengan rute default aktual. Jika jaringan WAN nanti berubah, sesuaikan `FAMILY-ISOLATION` dan pengujian.

### Tahap 4 — Amankan input router dari Guest, TANPA memutus DNS

Ini adalah **perbaikan penting dibanding script percobaan awal**. Masukkan aturan drop dulu, lalu pengecualian dengan `place-before=0`, sehingga hasil akhirnya berurutan DHCP → DNS UDP/TCP → ICMP → DROP.

```routeros
/ip firewall filter
add chain=input in-interface=wifi1-guest action=drop place-before=0 comment="FAMILY-INPUT-DROP"
add chain=input in-interface=wifi1-guest protocol=icmp action=accept place-before=0 comment="FAMILY-INPUT-PING"
add chain=input in-interface=wifi1-guest protocol=tcp dst-port=53 action=accept place-before=0 comment="FAMILY-INPUT-DNS-TCP"
add chain=input in-interface=wifi1-guest protocol=udp dst-port=53 action=accept place-before=0 comment="FAMILY-INPUT-DNS-UDP"
add chain=input in-interface=wifi1-guest protocol=udp dst-port=67 action=accept place-before=0 comment="FAMILY-INPUT-DHCP"
```

**Alasan:** DHCP Guest meminta layanan UDP 67 di router; DNS Guest Mode A juga masuk ke `chain=input` setelah `redirect`. **Jangan menaruh DROP di atas accept DHCP/DNS.** Rule ini membatasi WinBox TCP 7777, SSH, WebFig, dan servis IPv4 router lain dari Guest. Perangkat dari LAN utama tidak terpengaruh oleh rule berbasis `in-interface=wifi1-guest`.

> Jika perangkat Guest memerlukan NTP dari router, tambahkan pengecualian input UDP 123 **sebelum** `FAMILY-INPUT-DROP`; atau gunakan server waktu eksternal. Layanan router lain juga harus dipertimbangkan secara sengaja.

### Tahap 5 — Block IPv6 Guest tanpa mematikan IPv6 global

```routeros
/ipv6 firewall filter
add chain=forward in-interface=wifi1-guest action=drop place-before=0 comment="FAMILY-IPV6-FORWARD"
add chain=input in-interface=wifi1-guest action=drop place-before=0 comment="FAMILY-IPV6-INPUT"
```

Dua rule ini menolak IPv6 yang diteruskan dan masuk router dari Guest. Karena tidak ada IPv6 global Guest pada snapshot, rule berfungsi sebagai pengamanan antisipatif. Mereka **tidak** mematikan seluruh IPv6 router atau memperbaiki entri IPv6 invalid di `bridge-lan`. Jika nanti memakai IPv6 Guest secara resmi, rancang firewall dan DNS IPv6 khusus sebelum menghapus block ini.

### Tahap 6 — Opsional: isolasi sesama klien WiFi Guest

Cek dukungan dan kondisi asli:

```routeros
/interface wifi print detail where name="wifi1-guest"
```

Untuk interface WiFi RouterOS v7 yang mendukung properti ini:

```routeros
/interface wifi set wifi1-guest datapath.client-isolation=yes
```

Simpan nilai sebelumnya agar mudah dikembalikan. Jangan aktifkan bila perlu casting, printer, atau komunikasi langsung antarklien Guest. Perhatikan bahwa access-list WiFi dapat meng-override pengaturan client isolation.

### Tahap 7 — Opsional: blokir tujuan Cloudflare WARP yang diketahui

**Ini lebih agresif dan tidak dapat menjamin semua endpoint WARP masa depan.** IP Cloudflare dapat dipakai layanan lain. Gunakan hanya jika memang perlu dan uji aplikasi.

```routeros
/ip firewall address-list
add list=FAMILY-WARP address=162.159.192.0/24 comment="FAMILY-WARP-CONSUMER"
add list=FAMILY-WARP address=162.159.193.0/24 comment="FAMILY-WARP-WG"
add list=FAMILY-WARP address=162.159.197.0/24 comment="FAMILY-WARP-MASQUE"

/ip firewall filter
add chain=FAMILY-FWD dst-address-list=FAMILY-WARP action=drop place-before=[find where comment="FAMILY-RETURN"] comment="FAMILY-WARP-BLOCK"
```

Pastikan rule WARP berada **sebelum** `FAMILY-RETURN` dan **sebelum** `FAMILY-ISOLATION` tidak wajib, karena keduanya tidak menerima secara otomatis; yang penting bukan di bawah `RETURN`.

---

## 6. FastTrack tanpa dimatikan

Output awal menunjukkan **tidak ada rule FastTrack**:

```routeros
/ip firewall filter print where action=fasttrack-connection
```

Jadi **tidak ada perintah disable FastTrack yang perlu dijalankan**. Jika suatu hari ingin menambahkan rule FastTrack, jangan FastTrack koneksi yang salah satu ujungnya adalah `192.168.4.0/24`, karena paket yang di-FastTrack dapat melewati pemeriksaan firewall.

Prinsip yang ingin dipertahankan:

- Guest → inspeksi penuh pada firewall Guest.
- Subnet `192.168.2.0/24` dan `192.168.3.0/24` → kebijakan lama tetap; FastTrack boleh diterapkan secara khusus jika desain firewall mendukung.
- WireGuard/ZeroTier → tidak diubah melalui perintah dalam dokumen ini.

Jangan menambahkan rule FastTrack baru tanpa melihat urutan rule `forward` saat itu.

---

## 7. Pengujian

### A. Periksa aturan dan counter

```routeros
/ip firewall filter print stats where comment~"FAMILY-"
/ip firewall nat print stats where comment~"FAMILY-"
/ipv6 firewall filter print stats where comment~"FAMILY-"
/ip firewall filter print where action=fasttrack-connection
/ip dns print
```

Jika `FAMILY-ENTRY` bernilai nol ketika Guest aktif browsing, verifikasi `in-interface` yang sebenarnya dan apakah lalu lintas berjalan lewat router tersebut. NAT port 53 harus terlihat menerima query baru; koneksi lama dapat menggunakan keputusan NAT sebelumnya karena connection tracking.

### B. Uji dari perangkat yang benar-benar terhubung ke Guest

Di Windows:

```powershell
ipconfig /all
nslookup example.com 192.168.4.1
nslookup example.com 8.8.8.8
nslookup nudity.testcategory.com 192.168.4.1
```

- Mode A: Cloudflare Gateway custom harus menerima query; validasi melalui **Gateway DNS logs** dan domain yang benar-benar diblokir oleh **policy milikmu**. Domain uji Cloudflare Families tidak otomatis diblokir oleh Gateway custom tanpa policy yang sesuai.
- Mode B: domain tes resmi Cloudflare Families `nudity.testcategory.com` seharusnya mendapatkan jawaban blocked (umumnya `0.0.0.0`).
- Hasil `nslookup` yang menampilkan `Server: 8.8.8.8` **tidak membuktikan query keluar ke Google**: NAT bisa mengalihkan trafik di belakang layar.

### C. Checklist hasil

- [ ] Dari Guest, internet umum masih bisa diakses.
- [ ] Dari LAN/WLAN utama, internet dan DNS lama tetap berfungsi.
- [ ] DHCP Guest tetap memberikan IP, gateway `192.168.4.1`, DNS `192.168.4.1`.
- [ ] DNS port 53 dari Guest masuk ke resolver yang sesuai Mode A/B.
- [ ] DNS filter benar-benar memblokir kategori adult/malware (berdasarkan tes dan log).
- [ ] Guest tidak bisa mengakses WinBox/SSH/WebFig router.
- [ ] Guest tidak bisa mengakses IP LAN/WLAN utama melalui router.
- [ ] Guest tidak bisa routing ke WireGuard (`wg0`) dan ZeroTier (`zerotier1`).
- [ ] VPN WireGuard/OpenVPN pada port standar terblokir (VPN TCP 443 mungkin lolos).
- [ ] DoT/DoQ port 853 dan QUIC UDP 443 diuji.
- [ ] IPv6 Guest tidak menjadi jalur bypass.
- [ ] Uji aplikasi penting: browser, WhatsApp, panggilan video, streaming, game.
- [ ] Akses administrasi melalui LAN utama tetap tersedia.

**Catatan koneksi lama:** NAT dan FastTrack (jika ada di masa depan) mengikuti connection tracking. Setelah edit, reconnect WiFi Guest / tutup sesi aplikasi sebelum tes ulang. Hindari menghapus seluruh connection tracking router jika tidak diperlukan.

---

## 8. Ganti DNS di kemudian hari

### Dari Cloudflare Families (Mode B) kembali ke Gateway custom (Mode A)

Di **WinBox → IP → Firewall → NAT**, **disable** rule berkomentar `FAMILY-DNS-UDP` dan `FAMILY-DNS-TCP`. Jangan menghapus rule lama `Force DNS UDP` / `Force DNS TCP` jika hendak tetap memakai DNS router.

Terminal:

```routeros
/ip firewall nat
 disable [find where comment="FAMILY-DNS-UDP"]
 disable [find where comment="FAMILY-DNS-TCP"]
```

Guest akan memakai DNS MikroTik (`192.168.4.1`), yang upstream-nya tetap Cloudflare Gateway custom, **selama** rule redirect lama aktif dan `use-doh-server` benar.

### Dari Gateway custom (Mode A) ke Cloudflare Families publik (Mode B)

Jika rule `FAMILY-DNS-UDP/TCP` belum ada, tambahkan:

```routeros
/ip firewall nat
add chain=dstnat in-interface=wifi1-guest protocol=udp dst-port=53 action=dst-nat to-addresses=1.1.1.3 to-ports=53 place-before=0 comment="FAMILY-DNS-UDP"
add chain=dstnat in-interface=wifi1-guest protocol=tcp dst-port=53 action=dst-nat to-addresses=1.1.1.3 to-ports=53 place-before=0 comment="FAMILY-DNS-TCP"
```

Jika rule sudah ada dalam status disabled, **jangan tambahkan duplikat**, cukup ubah IP bila perlu lalu enable:

```routeros
/ip firewall nat
set [find where comment="FAMILY-DNS-UDP"] to-addresses=1.1.1.3
set [find where comment="FAMILY-DNS-TCP"] to-addresses=1.1.1.3
enable [find where comment="FAMILY-DNS-UDP"]
enable [find where comment="FAMILY-DNS-TCP"]
```

Pastikan kedua rule berada **di atas** NAT redirect port 53 lama (juga di atas `FAMILY-DNS-ROUTER-*` jika pernah dipasang). Untuk mengubah IP dari WinBox: **IP → Firewall → NAT → pilih FAMILY-DNS-UDP/TCP → tab Action → To Addresses**. Pasangan Cloudflare Families malware+adult: primary `1.1.1.3`, secondary `1.0.0.3`. Dokumen ini sengaja hanya mengarahkan ke primary agar aturan sederhana; perencanaaan failover terpisah bila dibutuhkan.

### Mengubah URL Cloudflare Gateway custom

Di WinBox **IP → DNS → Use DoH Server**, atau:

```routeros
/ip dns print
```

Jangan masukkan URL DoH ke field `to-addresses` NAT: field itu membutuhkan **alamat IP**, bukan URL. Jika URL Gateway berubah, konfigurasi upstream `/ip dns` dan DNS policies Cloudflare perlu diverifikasi bersama. Perubahan `/ip dns` berlaku ke **semua subnet** yang menggunakan resolver router, bukan hanya Guest.

---

## 9. Rollback dan troubleshooting

### A. Jika Safe Mode masih aktif

`Ctrl+D` untuk membatalkan tahap yang sedang dicoba. Jika koneksi administrator terputus tidak normal, Safe Mode akan berusaha memulihkan perubahan yang belum dikomit.

### B. Nonaktifkan hanya aturan Family Protection

Jika perubahan sudah disimpan dan ingin memulihkan akses sementara:

```routeros
/ip firewall filter disable [find where comment~"FAMILY-"]
/ip firewall nat disable [find where comment~"FAMILY-"]
/ipv6 firewall filter disable [find where comment~"FAMILY-"]
```

Konfigurasi Cloudflare Gateway pada `/ip dns`, DNS DHCP, NAT global lama, WireGuard, dan ZeroTier tidak disentuh.

### C. Hapus semua komponen Family Protection yang didokumentasikan

**Sebelum menjalankan, periksa nama komentar:** jika ada rule lain memakai prefix `FAMILY-` tanpa terkait runbook ini, hapus secara selektif, jangan jalankan perintah massal.

```routeros
/ip firewall filter remove [find where comment~"FAMILY-"]
/ip firewall nat remove [find where comment~"FAMILY-"]
/ipv6 firewall filter remove [find where comment~"FAMILY-"]
/ip firewall address-list remove [find where comment~"FAMILY-"]
```

Pengaturan WiFi client isolation tidak memiliki komentar `FAMILY-`; kembalikan secara manual ke nilai awal bila pernah diubah:

```routeros
/interface wifi print detail where name="wifi1-guest"
```

Rule `FMX-` dari percobaan sebelumnya **tidak termasuk** perintah rollback ini. Audit dan hapus/disable secara terpisah setelah dipastikan milik konfigurasi eksperimen.

### D. Masalah yang sering muncul

| Gejala | Periksa dan solusi |
|---|---|
| Guest mendapatkan IP tetapi website tidak bisa terbuka | Cek `FAMILY-INPUT-DNS-UDP/TCP` ditempatkan **di atas** `FAMILY-INPUT-DROP` (khusus Mode A); periksa koneksi DoH router dan DNS logs |
| DNS Guest masih memakai DNS custom padahal ingin `1.1.1.3` | Cek NAT `FAMILY-DNS-UDP/TCP` status enabled dan urutannya **lebih atas** dari `Force DNS`/`FAMILY-DNS-ROUTER-*` |
| Guest memakai `1.1.1.3` padahal ingin custom | Disable NAT `FAMILY-DNS-UDP/TCP`; pastikan redirect ke router serta custom DoH masih aktif |
| Browser lolos konten terblokir | Periksa Secure DNS browser, VPN/proxy, cache browser, kebijakan Cloudflare, dan rule DoH; jangan menganggap port 53 saja cukup |
| `FAMILY-ENTRY` tidak bertambah | Periksa bahwa Guest benar masuk dari `wifi1-guest`, bukan bridge/interface lain |
| WhatsApp/game/streaming terganggu | Uji sementara disable `FAMILY-QUIC` atau blokir port lain secara selektif; lakukan dengan Safe Mode dan catat perubahan |
| DNS seluruh router gagal | Cek `/ip dns print`, DoH endpoint custom, resolusi bootstrap, konektivitas TCP 443 dari router, serta validasi sertifikat |
| Akses LAN dari Guest gagal | Sesuai desain `FAMILY-ISOLATION`; buat pengecualian spesifik sebelum aturan itu jika memang diperlukan |
| Tidak bisa WinBox dari Guest | Sesuai desain. Admin harus memakai `bridge-lan`/`bridge-wlan` atau jalur administrasi yang aman |
| IPv6 Guest gagal | Sesuai desain, ada DROP pada `ipv6 firewall input/forward` |

**Catatan keamanan terpisah:** pada snapshot terdapat rule bernama `Drop Winbox from WAN` tetapi menggunakan `chain=forward`, bukan `chain=input`. Jangan mengandalkan nama komentarnya sebagai bukti WAN admin terlindungi; audit seluruh kebijakan administrasi router secara terpisah. Dokumen ini sengaja tidak mengubah firewall WAN secara global.

---

## 10. Quick reference

**Perintah status:**

```routeros
/ip address print
/ip route print
/ip dns print
/ip dhcp-server network print
/ip firewall nat print stats
/ip firewall filter print stats where comment~"FAMILY-"
/ipv6 firewall filter print stats where comment~"FAMILY-"
/ip firewall filter print where action=fasttrack-connection
```

**Konsep yang harus diingat:**

1. `192.168.4.0/24` = subnet Guest; `192.168.4.1` = gateway, bukan subnet.
2. `wifi1-guest` = interface spesifik Guest; `pppoe-out1` = WAN aktif pada snapshot.
3. **Mode A** = DNS Guest diarahkan ke DNS MikroTik → DoH custom Cloudflare Gateway. Ini pilihan yang direkomendasikan bila kebijakan filtering sudah disiapkan.
4. **Mode B** = NAT DNS Guest menuju `1.1.1.3`; tidak melewati Cloudflare Gateway custom untuk kueri port 53.
5. **Mode A perlu `input accept udp/tcp 53` sebelum `input drop` Guest**.
6. `FAMILY-FWD` = chain proteksi yang hanya dipanggil dari `wifi1-guest`, `FAMILY-RETURN` mengembalikan ke chain lama.
7. FastTrack **tidak perlu dinonaktifkan**; pada snapshot belum ada rule FastTrack. Jika nanti dibuat, kecualikan Guest dari FastTrack.
8. IPv6 Guest difilter terpisah; memblokir IPv4 saja belum cukup.
9. Semua rule dibedakan lewat prefix komentar `FAMILY-`; gunakan untuk audit dan rollback.
10. Simpan backup dan uji dari perangkat Guest serta LAN utama sebelum keluar dari Safe Mode.

---

## 11. Rujukan resmi

- MikroTik — [NAT dan perbedaan `dst-nat`/`redirect`](https://help.mikrotik.com/docs/spaces/ROS/pages/3211299/NAT)
- MikroTik — [Firewall chains: input, forward, jump, return](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/)
- MikroTik — [Connection tracking dan FastTrack](https://help.mikrotik.com/docs/spaces/ROS/pages/130220087/Connection%2Btracking)
- MikroTik — [Safe Mode / Console](https://help.mikrotik.com/docs/spaces/ROS/pages/8978498/Console)
- MikroTik — [DNS over HTTPS pada RouterOS](https://manual.mikrotik.com/docs/network-management/dns/)
- MikroTik — [WiFi client isolation](https://help.mikrotik.com/docs/spaces/ROS/pages/224559120/WiFi)
- Cloudflare — [Custom Gateway DoH endpoints](https://developers.cloudflare.com/cloudflare-one/traffic-controls/resolver-endpoints/dns-over-https/)
- Cloudflare — [DNS policies, Safe Search, YouTube Restricted](https://developers.cloudflare.com/cloudflare-one/traffic-controls/dns-policies/)
- Cloudflare — [Cloudflare 1.1.1.1 for Families](https://developers.cloudflare.com/1.1.1.1/setup/)

---

*Dokumen ini merekam desain dan perintah konfigurasi berdasarkan keadaan router yang dibagikan pada 16 Oktober 2026. Ia bukan ekspor running configuration aktual setelah instalasi. Selalu lakukan audit ulang interface, route, DNS, filter dan NAT sebelum memakainya pada router yang sudah di-reset atau telah berubah.*
