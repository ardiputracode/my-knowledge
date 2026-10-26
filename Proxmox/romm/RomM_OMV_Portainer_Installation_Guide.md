# Panduan Instalasi RomM di OpenMediaVault + Portainer

**Versi dokumen:** 1.0  
**Tanggal:** 26 Oktober 2026  
**Target:** OpenMediaVault (OMV) dengan Docker dan Portainer  
**Layanan:** RomM + MariaDB; cache Valkey terintegrasi pada image RomM  
**Alamat akses:** `http://IP-OMV:8181`

> **Tujuan:** memasang RomM untuk mengelola library ROM melalui Portainer, dengan ROM berada pada disk yang dapat dibagikan lewat SMB dan data aplikasi pada folder Docker AppData. Semua jalur di panduan ini mengikuti lokasi disk OMV yang sudah ditentukan. Tidak ada password/API key asli yang ditulis di file.

## Daftar isi

1. Arsitektur dan lokasi penyimpanan
2. Prasyarat dan pemeriksaan awal OMV
3. Menyiapkan folder melalui SSH
4. Menyiapkan secret dan metadata provider
5. Docker Compose / Portainer Stack (YAML lengkap)
6. Men-deploy Stack di Portainer
7. Menyiapkan library dan akun admin
8. Mengelola ROM dari komputer Windows lewat SMB
9. Pemeriksaan container dan permission
10. Backup & pemulihan
11. Update dan pemeliharaan
12. Troubleshooting
13. Checklist instalasi
14. Referensi resmi

---

## 1. Arsitektur dan lokasi penyimpanan

| Komponen | Lokasi host OMV | Lokasi container / fungsi |
|---|---|---|
| Library game | `/srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8/SMB/romm/library` | `/romm/library` (bind mount) |
| Saves, state, upload | `/srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8/docker/appdata/romm/assets` | `/romm/assets` (bind mount) |
| Pengaturan tambahan | `/srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8/docker/appdata/romm/config` | `/romm/config` (bind mount) |
| Database MariaDB | Docker named volume `mysql_data` | `/var/lib/mysql` pada `romm-db` |
| Gambar metadata | Docker named volume `romm_resources` | `/romm/resources` |
| Cache/background tasks | Docker named volume `romm_redis_data` | `/redis-data` |

**Catatan penting:** ketiga *named volumes* dikelola Docker dan **tidak berada dalam tiga folder bind mount** di atas. Jika nama Stack adalah `romm`, nama volume fisiknya lazimnya `romm_mysql_data`, `romm_romm_resources`, dan `romm_romm_redis_data`, tetapi cek nama aktual dengan `docker volume ls`. Hapus/redeploy container biasa tidak otomatis menghapus volume; *menghapus stack beserta volume*, menghapus volume manual, atau memakai `docker compose down -v` dapat menghilangkan data.

**Port:** `8181` adalah port host OMV, dipetakan ke port `8080` di container. Port database tidak dipublikasikan ke LAN. Tidak diperlukan container Valkey terpisah pada konfigurasi referensi ini.

### Struktur library yang disarankan (Structure A)

```text
SMB/romm/library/
├── roms/
│   ├── psx/          # PlayStation 1; slug resmi = psx
│   ├── ps2/
│   ├── psp/
│   ├── gba/
│   └── snes/
└── bios/
    ├── psx/
    └── ps2/
```

Nama folder konsol memakai **slug platform RomM**. `ps` dikenali sebagai alias `psx` pada dokumentasi saat ini, tetapi `psx` lebih eksplisit. File game cukup ditempatkan di bawah `roms/<platform>/`; contoh `roms/gba/Nama Game.gba`. BIOS hanya diperlukan pada sistem tertentu; gunakan file yang dimiliki secara sah. RomM bisa mengindeks PS2, tetapi **tidak berarti semua sistem dapat dimainkan lewat browser**.

## 2. Prasyarat dan pemeriksaan awal OMV

Pastikan:

- Server OMV sudah memasang dan menjalankan Docker serta Portainer.
- Akun yang dipakai dapat mengakses Portainer dan SSH OMV (`sudo` bila diperlukan).
- Disk dengan UUID `87d634ab-40dc-437c-b70f-6785297a78d8` benar-benar terpasang dan memiliki ruang kosong.
- Port host `8181` belum digunakan layanan lain.
- Server bisa mengakses Docker registry dan penyedia metadata bila dipakai.

Cek via SSH:

```bash
docker version
docker ps
findmnt /srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8
df -h /srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8
sudo ss -ltnp | grep ':8181 ' || true
```

Jika `findmnt` tidak menampilkan mount disk yang dimaksud, **jangan mulai deploy**. Verifikasi mount di menu Storage OMV agar Docker tidak membuat folder di disk sistem akibat path disk data yang tidak terpasang.

## 3. Menyiapkan folder melalui SSH

Salin perintah berikut ke SSH OMV:

```bash
BASE="/srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8"

sudo mkdir -p "$BASE/SMB/romm/library/roms/psx"
sudo mkdir -p "$BASE/SMB/romm/library/roms/ps2"
sudo mkdir -p "$BASE/SMB/romm/library/roms/psp"
sudo mkdir -p "$BASE/SMB/romm/library/roms/gba"
sudo mkdir -p "$BASE/SMB/romm/library/roms/snes"
sudo mkdir -p "$BASE/SMB/romm/library/bios/psx"
sudo mkdir -p "$BASE/SMB/romm/library/bios/ps2"
sudo mkdir -p "$BASE/docker/appdata/romm/assets"
sudo mkdir -p "$BASE/docker/appdata/romm/config"
```

Periksa hasilnya:

```bash
sudo ls -ld "$BASE/SMB/romm/library" \
  "$BASE/docker/appdata/romm/assets" \
  "$BASE/docker/appdata/romm/config"
find "$BASE/SMB/romm/library" -maxdepth 2 -type d | sort
```

**Jangan gunakan `chmod -R 777` atau `chown -R` pada seluruh folder `SMB`**. Di OMV, izin pengguna SMB dan ACL perlu dipertahankan; jika muncul penolakan akses, lihat bagian troubleshooting terlebih dahulu.

## 4. Menyiapkan secret dan metadata provider

Di terminal OMV, jalankan **tiga kali** untuk menghasilkan nilai acak berbeda:

```bash
openssl rand -hex 32
```

Simpan ketiganya di password manager, dengan nama:

| Environment variable di Portainer | Kegunaan | Wajib? |
|---|---|---|
| `ROMM_DB_ROOT_PASSWORD` | Password root MariaDB | Ya |
| `ROMM_DB_PASSWORD` | Password akun database aplikasi | Ya |
| `ROMM_AUTH_SECRET_KEY` | Kunci tanda tangan autentikasi RomM | Ya |
| `SCREENSCRAPER_USER` | Username ScreenScraper | Opsional |
| `SCREENSCRAPER_PASSWORD` | Password ScreenScraper | Opsional |
| `IGDB_CLIENT_ID` | Client ID IGDB | Opsional |
| `IGDB_CLIENT_SECRET` | Client secret IGDB | Opsional |
| `STEAMGRIDDB_API_KEY` | API key gambar SteamGridDB | Opsional |
| `RETROACHIEVEMENTS_API_KEY` | API key RetroAchievements | Opsional |

Pastikan `ROMM_DB_PASSWORD` yang dipakai aplikasi sama persis dengan `MARIADB_PASSWORD` (YAML di bawah sudah menghubungkan keduanya). Jangan pakai password root database sebagai password aplikasi. **Jangan mengganti secret autentikasi sembarangan setelah instalasi**, karena dapat membuat sesi login/token tidak berlaku.

**Metadata:** file YAML mengaktifkan `HASHEOUS_API_ENABLED=true` agar tersedia metode pencocokan metadata tanpa API key. Untuk hasil yang lebih lengkap, sangat disarankan menambahkan ScreenScraper dan/atau IGDB sebelum scan pertama. Nilai provider lain dapat dibiarkan kosong di Portainer: YAML memakai default `:-` untuk mencegah error akibat variabel opsional yang tidak ditentukan.

> Keamanan: environment variables Portainer mengandung rahasia. Batasi hak akses administrator Portainer dan backup berkas Stack/secret secara aman. Jangan unggah YAML berisi password asli ke repositori publik.

## 5. Docker Compose / Portainer Stack (YAML lengkap)

YAML berikut sudah disesuaikan untuk path OpenMediaVault kamu, dengan MariaDB major version `11` dan port `8181`. Tempel utuh ke Portainer > Stacks > Add stack > Web editor.

```yaml
# RomM Stack for OpenMediaVault + Portainer
# Panduan lengkap: RomM_OMV_Portainer_Installation_Guide.md
# Buat tiga secret pada Portainer > Stacks > Environment variables
# ROMM_DB_ROOT_PASSWORD, ROMM_DB_PASSWORD, ROMM_AUTH_SECRET_KEY

services:
  romm:
    image: rommapp/romm:latest
    container_name: romm
    restart: unless-stopped
    ports:
      - "8181:8080"
    environment:
      TZ: Asia/Jakarta
      DB_HOST: romm-db
      DB_NAME: romm
      DB_USER: romm-user
      DB_PASSWD: "${ROMM_DB_PASSWORD:?Set ROMM_DB_PASSWORD in Portainer}"
      ROMM_AUTH_SECRET_KEY: "${ROMM_AUTH_SECRET_KEY:?Set ROMM_AUTH_SECRET_KEY in Portainer}"

      # Provider metadata opsional; boleh dikosongkan pada deployment awal
      SCREENSCRAPER_USER: "${SCREENSCRAPER_USER:-}"
      SCREENSCRAPER_PASSWORD: "${SCREENSCRAPER_PASSWORD:-}"
      IGDB_CLIENT_ID: "${IGDB_CLIENT_ID:-}"
      IGDB_CLIENT_SECRET: "${IGDB_CLIENT_SECRET:-}"
      STEAMGRIDDB_API_KEY: "${STEAMGRIDDB_API_KEY:-}"
      RETROACHIEVEMENTS_API_KEY: "${RETROACHIEVEMENTS_API_KEY:-}"
      HASHEOUS_API_ENABLED: "true"

    volumes:
      # Docker named volumes
      - romm_resources:/romm/resources
      - romm_redis_data:/redis-data

      # ROM library di folder yang dapat dibagikan melalui SMB OMV
      - /srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8/SMB/romm/library:/romm/library

      # Aplikasi / file upload di Docker AppData
      - /srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8/docker/appdata/romm/assets:/romm/assets
      - /srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8/docker/appdata/romm/config:/romm/config

    depends_on:
      romm-db:
        condition: service_healthy

  romm-db:
    image: mariadb:11
    container_name: romm-db
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: "${ROMM_DB_ROOT_PASSWORD:?Set ROMM_DB_ROOT_PASSWORD in Portainer}"
      MARIADB_DATABASE: romm
      MARIADB_USER: romm-user
      MARIADB_PASSWORD: "${ROMM_DB_PASSWORD:?Set ROMM_DB_PASSWORD in Portainer}"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      start_period: 30s
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql_data:
  romm_resources:
  romm_redis_data:
```

**Penting:** YAML ini memakai interpolasi `${VARIABLE}`. Maka sebelum klik Deploy, definisikan tiga variabel wajib di bagian **Environment variables** pada halaman Stack Portainer. Tanpa variabel wajib, deployment akan ditolak oleh Compose dan password tidak akan diganti otomatis.

## 6. Deploy Stack melalui Portainer

1. Akses Portainer melalui browser dan pilih environment Docker pada host OMV.
2. Buka menu **Stacks** > **Add stack**.
3. Isi **Name:** `romm`.
4. Pilih **Web editor**, lalu paste YAML di atas (atau upload file `romm-portainer-omv.yml` jika tersedia).
5. Pada bagian **Environment variables**, tambahkan tiga variabel wajib dari Bagian 4 beserta nilainya; API metadata opsional juga bisa ditambahkan pada langkah ini.
6. Periksa lagi ketiga bind mount untuk memastikan UUID disk sesuai dan port `8181` belum dipakai.
7. Klik **Deploy the stack**.
8. Masuk ke **Containers**; pastikan `romm-db` sehat (*healthy*) dan `romm` berjalan (*running*).

Untuk verifikasi via SSH:

```bash
docker ps --filter 'name=romm'
docker logs --tail 100 romm-db
docker logs --tail 150 romm
```

Log database saat pertama kali mulai bisa membutuhkan proses inisialisasi. Jangan menyalin password/API key ke tiket publik ketika membagikan log.

## 7. Menyiapkan library dan akun admin

### 7.1 Login awal

Buka di komputer yang satu LAN dengan OMV:

```text
http://IP-OMV:8181
```

Ganti `IP-OMV` dengan IP NAS/server OMV, misalnya `192.168.1.10` (contoh saja). Pada instalasi baru RomM akan menampilkan **Setup Wizard**. Buat username dan password administrator, kemudian login.

### 7.2 Masukkan ROM

Salin ROM ke folder platform yang sesuai di `SMB/romm/library/roms/`. Contoh:

```text
library/roms/gba/Contoh Game.gba
library/roms/psx/Contoh Game.chd
library/roms/snes/Contoh Game.sfc
```

Gunakan nama file yang informatif agar cocok dengan metadata. Untuk game multi-disc, baca aturan folder multi-file pada dokumentasi RomM sebelum memindahkan file.

### 7.3 Scan library

1. Login ke antarmuka web RomM.
2. Buka fungsi **Scan** pada UI (label/letaknya mungkin berubah antarversi).
3. Pilih provider metadata yang sudah diaktifkan.
4. Pada scan pertama, gunakan mode **Quick** jika tersedia.
5. Tunggu hasil scan, periksa game *matched* dan *unmatched*.
6. Jika judul tidak cocok, benahi nama file/platform dan lakukan scan ulang sesuai pilihan UI.

RomM dapat membaca ROM tanpa metadata API, tetapi hasil pencocokan/cover bisa terbatas. Jangan berasumsi semua game yang terindeks bisa dimainkan lewat browser; dukungan emulator bergantung platform dan format ROM.

## 8. Mengelola ROM dari Windows melalui SMB

Path `/srv/.../SMB/romm/library` adalah **path Linux lokal**; ini bukan alamat SMB Windows. Agar dapat diakses melalui File Explorer, pastikan di OMV folder tersebut sudah tercakup dalam *Shared Folder* dan layanan SMB/CIFS memang membagikannya, dengan hak akses yang sesuai.

Contoh pengaksesan dari Windows (nama share bergantung setting OMV):

```text
\\IP-OMV\NamaShare\romm\library\roms
```

Jika yang kamu bagikan di OMV adalah folder `SMB` sebagai `NamaShare`, urutan `romm\library\roms` di atas benar. Jika share menunjuk langsung folder `romm`, jalurnya menjadi `\\IP-OMV\NamaShare\library\roms`. Jangan membuat Samba share baru dengan akses write untuk `Everyone` tanpa pertimbangan keamanan.

**Alur kerja yang nyaman:** copy ROM dari PC Windows melalui SMB, lalu buka RomM dan jalankan Scan. Usahakan tidak melakukan scan saat file berukuran besar masih dalam proses transfer.

## 9. Pemeriksaan mount dan permission

### 9.1 Periksa folder yang dilihat container

```bash
docker exec romm ls -la /romm/library
docker exec romm ls -la /romm/library/roms
docker exec romm ls -la /romm/assets
docker exec romm ls -la /romm/config
```

Jika library benar, setidaknya direktori `roms` dan `bios` terlihat. Periksa identitas container dan hak akses host:

```bash
docker exec romm id
BASE="/srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8"
stat -c '%A %U:%G (%u:%g) %n' "$BASE/SMB/romm/library" \
  "$BASE/docker/appdata/romm/assets" \
  "$BASE/docker/appdata/romm/config"
namei -l "$BASE/SMB/romm/library"  # bila paket util-linux menyediakan namei
```

Untuk **baca/scan**, container perlu izin read dan traverse (`r`/`x`) pada library. Untuk **upload melalui RomM**, container juga butuh write pada folder ROM yang dituju. Untuk file `assets` dan `config`, container membutuhkan izin menulis. Bila ada masalah, sesuaikan ACL/user/group pada path **spesifik** memakai pengaturan OMV; jangan merusak ownership seluruh shared folder.

### 9.2 Periksa port

```bash
sudo ss -ltnp | grep ':8181 '
docker port romm
```

Jika web UI tidak bisa dibuka, cek firewall, alamat IP OMV, status container, dan apakah ada reverse proxy yang mengalihkan port.

## 10. Backup & pemulihan

### Apa yang wajib dicadangkan?

| Data | Prioritas | Catatan |
|---|---|---|
| MariaDB (`mysql_data`) | Kritis | Akun, koleksi, metadata, progres, referensi save |
| `assets` | Kritis | Saves, states, file upload |
| `config` | Kritis | Konfigurasi kustom |
| `library` | Kritis | ROM dan BIOS asli milikmu; backup terpisah |
| `romm_resources` | Disarankan | Cover/screenshots; sebagian dapat diunduh lagi |
| `romm_redis_data` | Rendah | Cache/antrian sementara |
| Secret Portainer | Kritis | Simpan aman di luar sistem yang sama |

### 10.1 Backup database (online/logical dump)

Perintah ini membaca password dari environment di dalam container sehingga tidak perlu menulis password di command SSH:

```bash
BACKUP_DIR="$HOME/romm-backups"
mkdir -p "$BACKUP_DIR"
chmod 700 "$BACKUP_DIR"
umask 077

docker exec romm-db sh -c \
  'exec mariadb-dump --user="$MARIADB_USER" --password="$MARIADB_PASSWORD" --single-transaction --databases "$MARIADB_DATABASE"' \
  > "$BACKUP_DIR/romm-db-$(date +%F).sql"

ls -lh "$BACKUP_DIR"/romm-db-*.sql
```

Perintah dump membutuhkan `romm-db` dalam kondisi running. Pastikan file output tidak kosong, cek log bila perintah gagal, dan lakukan uji restore di lingkungan terpisah sebelum mengandalkannya. Folder `~/romm-backups` hanya lokasi *sementara* untuk contoh; pindahkan hasil ke media cadangan berbeda/offsite. Hindari menyimpan satu-satunya backup pada disk yang sama dengan library.

### 10.2 Backup `assets`, `config`, dan library

Hentikan aktivitas tulis ke RomM saat backup file agar konsisten dengan database. Untuk contoh arsip:

```bash
BASE="/srv/dev-disk-by-uuid-87d634ab-40dc-437c-b70f-6785297a78d8"
BACKUP_DIR="$HOME/romm-backups"
mkdir -p "$BACKUP_DIR"
chmod 700 "$BACKUP_DIR"

tar -C "$BASE/docker/appdata/romm" -czf \
  "$BACKUP_DIR/romm-appdata-$(date +%F).tar.gz" assets config

# Library biasanya besar; salin secara terpisah ke disk lain/NAS cadangan.
# Misalnya gunakan rsync menuju path backup tujuan yang sudah disiapkan.
```

Untuk `romm_resources`, bila ingin pemulihan lebih cepat, backup named volume setelah stack dihentikan, **tanpa** menghapus volumenya. Nama yang tepat dapat dicari lewat:

```bash
docker volume ls | grep romm
```

> **Penting:** jangan memakai `docker compose down -v`, opsi Portainer yang menghapus volume, atau `docker volume prune` tanpa memahami dampaknya. Database dan koleksi pengguna dapat hilang.

### 10.3 Alur pemulihan secara aman

1. Siapkan disk/mount dan ketiga bind path yang sama.
2. Buat Stack yang sama dengan nilai database dan `ROMM_AUTH_SECRET_KEY` sesuai backup.
3. Jalankan MariaDB hingga sehat. Saat restore, hentikan container aplikasi `romm` **saja** agar tidak ada aktivitas tulis dan biarkan `romm-db` tetap berjalan.
4. Restore dump SQL dari backup yang diketahui baik; contoh menggunakan variabel root password yang sudah terpasang dalam container:

   ```bash
   docker exec -i romm-db sh -c \
     'exec mariadb --user=root --password="$MARIADB_ROOT_PASSWORD"' \
     < /path/ke/romm-db-YYYY-MM-DD.sql
   ```

5. Pulihkan direktori `assets/` dan `config/` (serta library ROM jika diperlukan) ke lokasi bind mount masing-masing; sesuaikan permission/ACL.
6. Nyalakan kembali container `romm`, login, cek daftar game, saves, dan metadata.
7. Jika ada indikasi konflik skema database antarversi, gunakan image RomM yang cocok dengan versi backup terlebih dahulu sebelum melakukan upgrade.

*Catatan:* pemulihan database ke instalasi yang sudah berisi data bisa menimpa atau bentrok; rute paling aman adalah **lingkungan kosong dengan cadangan terverifikasi**, bukan import sembarang ke instance produksi.

## 11. Update dan pemeliharaan

- **Sebelum update:** lakukan backup database + assets + config, catat tag image lama serta simpan file Stack dan secret.
- Di **Portainer > Stacks > romm > Editor**, verifikasi YAML dan variabel; kemudian lakukan **Update the stack** dengan opsi menarik ulang image (*re-pull*) jika tersedia. Alternatif: gunakan fitur *Recreate* container dengan image terbaru, sesuai versi Portainer.
- `rommapp/romm:latest` akan mengikuti rilis terkini saat image ditarik ulang; untuk sistem yang dipakai rutin, **pin tag versi RomM** agar upgrade terkendali.
- Image database sengaja dipin ke `mariadb:11` (major version 11), sehingga tidak meloncat ke major MariaDB berikutnya secara otomatis. Tetap backup sebelum mengubahnya.
- Setelah update, cek container, login, library, metadata, dan saves. Hindari update RomM dan MariaDB major sekaligus.
- Data named volume tetap ada saat *recreate* normal selama volume yang sama digunakan dan tidak dihapus.

### Keamanan jaringan

Gunakan `http://IP-OMV:8181` untuk LAN privat. Jika RomM perlu diakses lewat internet, prioritaskan VPN pribadi (misalnya WireGuard/Tailscale) atau reverse proxy HTTPS yang dikonfigurasi dengan benar. Jangan membuka port `8181` langsung ke internet tanpa pengamanan. Batasi akses Portainer dan share SMB ke pengguna yang memang perlu.

## 12. Troubleshooting

| Gejala | Penyebab umum | Langkah pemeriksaan |
|---|---|---|
| `Error: port is already allocated` | Port 8181 dipakai layanan lain | `sudo ss -ltnp \| grep ':8181 '`; ganti mapping mis. `8282:8080` |
| Database tidak `healthy` | Password/inisialisasi/mount volume bermasalah | `docker logs romm-db --tail 150`; periksa healthcheck dan volume |
| `romm` restart terus | DB belum siap, secret salah, konfigurasi invalid | `docker logs romm --tail 200`; cek semua env wajib |
| `Permission denied` ketika scan/unggah | ACL/owner/group OMV tidak sesuai | `docker exec romm id`, `stat`, `getfacl` jika terpasang; perbaiki ACL pada path tertentu |
| Library kosong di UI | Path salah, struktur salah, disk belum mount, scan belum dijalankan | `findmnt`, `docker exec romm ls -la /romm/library`, cek `roms/<platform>` |
| Game tidak terdeteksi | Slug platform/format ROM tidak sesuai | Cek daftar platform dan first-scan guide RomM |
| Cover game tidak muncul | Metadata provider belum dikonfigurasi atau pencocokan gagal | Isi provider, periksa log scan, ulangi metadata matching |
| SMB bisa baca tetapi gagal menyalin | Pengaturan share/ACL pada OMV | Periksa SMB/CIFS, Shared Folder dan ACL untuk pengguna Windows |
| Setelah Stack dibuat ulang akun hilang | Named volume DB berubah/terhapus | Cek `docker volume ls`; restore backup SQL, jangan inisialisasi ulang sembarangan |
| Web UI tidak bisa dibuka | IP/port salah, kontainer mati, firewall | `docker ps`, `docker port romm`, firewall/port OMV |

**Catatan error MariaDB:** mengganti `MARIADB_PASSWORD` pada YAML **tidak otomatis mengubah password user pada database yang sudah terinisialisasi** di named volume lama. Jika memakai volume yang sama, pertahankan credential lama atau ubah user/password secara sengaja di MariaDB dan samakan dengan `DB_PASSWD` RomM.

## 13. Checklist instalasi

- [ ] Disk UUID OMV ter-mount dan memiliki ruang cukup.
- [ ] Port `8181` tersedia.
- [ ] Folder `library/roms`, `library/bios`, `assets`, dan `config` dibuat.
- [ ] Path bind mount dalam YAML sudah tepat.
- [ ] Secret root DB, password aplikasi, dan auth key sudah dibuat dan disimpan aman.
- [ ] Portainer Stack `romm` berhasil di-deploy.
- [ ] `romm-db` healthy dan `romm` running.
- [ ] Web UI dapat diakses di `http://IP-OMV:8181`.
- [ ] Admin dibuat, ROM ada di direktori dengan slug platform benar.
- [ ] Scan berhasil dan minimal satu ROM terdeteksi.
- [ ] SMB Windows memiliki akses yang sesuai.
- [ ] Backup DB dan appdata berhasil dibuat, lalu disalin ke media cadangan berbeda.

## 14. Referensi resmi

1. **Repository RomM:** <https://github.com/rommapp/romm>
2. **Compose resmi:** <https://github.com/rommapp/romm/blob/master/examples/docker-compose.example.yml>
3. **RomM Quick Start:** <https://docs.romm.app/latest/getting-started/quick-start/>
4. **Folder Structure:** <https://docs.romm.app/latest/getting-started/folder-structure/>
5. **Supported Platforms & aliases:** <https://docs.romm.app/latest/platforms/supported-platforms/>
6. **Metadata Providers:** <https://docs.romm.app/latest/getting-started/metadata-providers/>
7. **Your First Scan:** <https://docs.romm.app/latest/getting-started/first-scan/>
8. **Environment Variables:** <https://docs.romm.app/latest/reference/environment-variables/>
9. **Backup & Restore:** <https://docs.romm.app/latest/install/backup-and-restore/>

---

*Dokumen ini disusun untuk penggunaan pribadi dengan konfigurasi path OpenMediaVault yang telah ditentukan. Tinjau kembali dokumentasi RomM apabila melakukan upgrade besar karena opsi dan perilaku RomM dapat berubah.*
