---
title: "Membongkar Sisi Gelap Keamanan Web Kampus: Dari Google Dork hingga RCE"
date: 2026-06-30T17:00:00+07:00
author: "RavxyTech"
tags: ["web-security", "pentesting", "kampus", "sqli", "lfi", "rce", "cybersecurity", "bug-bounty"]
categories: ["Research", "Cybersecurity", "Edukasi"]
description: "Analisis komprehensif tentang celah keamanan web kampus, mulai dari target subdomain terlantar, exploit CMS (OJS, SLiMS, Balitbang), kerentanan file manager, hingga eksekusi RCE."
draft: false
image: "/images/kampus.png"

---
{{< figure src="/images/kampus.png" alt="kampus exploit" >}}

Sering banget kan denger berita database kampus bocor, atau desas-desus mahasiswa yang tiba-tiba nilainya berubah jadi A semua? Kalau sebelumnya kita bahas kerentanan umum kayak SQLi dan XSS, realitanya di lapangan... *attack vector*-nya jauh lebih liar dan sistematis dari itu wkwkwk.

Artikel ini bakal ngebahas secara mendalam dan lebih "niat" (siapin kopi sama rokok dulu wkwk) tentang kenapa infrastruktur digital pendidikan sering kali jadi sasaran empuk para peretas. Kita akan bedah anatomi serangannya, mulai dari teknik *reconnaissance* di subdomain terlantar, eksploitasi CMS kampus, kerentanan file manager, hingga masuk ke ranah *Remote Code Execution* (RCE). 

---

## Akar Masalah: Subdomain Terlantar (The Forgotten Assets)

Kenapa kampus sering kebobolan? Jawabannya simpel: **Subdomain yang tidak terurus**.

Sebuah kampus biasanya punya domain utama (misal: `kampus.ac.id`) yang dijaga dengan sangat ketat. Tapi mereka juga punya puluhan hingga ratusan subdomain untuk berbagai keperluan: `jurnal.kampus.ac.id`, `perpus.kampus.ac.id`, `pmb2018.kampus.ac.id`, `alumni.kampus.ac.id`, dan sebagainya.

Nah, **seorang hacker itu jarang menyerang pintu depan (domain utama)**. Mereka biasanya *scanning* dan mencari celah satu per satu di subdomain ini. Kenapa? Karena admin IT kampus sering kali lupa, kekurangan personel, atau malas mengecek dan memelihara web-web di subdomain yang mungkin umurnya sudah belasan tahun. Subdomain lawas ini ibarat jendela belakang yang dibiarkan terbuka di sebuah rumah yang pagarnya tinggi.

---

## Senjata Utama Attacker: Google Dorking & Reconnaissance

Untuk mencari "jendela yang terbuka" ini, attacker nggak selalu perlu nge-*bruteforce* server dengan alat berat. Sering kali cukup modal *Google Dorking*! 

*Google Dorking* adalah teknik menggunakan operator pencarian tingkat lanjut di Google (seperti parameter `site:`, `inurl:`, `intitle:`, atau `filetype:`) untuk mencari halaman web spesifik yang terekspos ke publik namun seharusnya disembunyikan.

**Apa yang biasa dicari menggunakan Google Dork di domain kampus?**
- Halaman panel login administrator CMS tertentu yang tersembunyi.
- Direktori *file* yang terbuka bebas dan bisa di-browse (*Directory Listing / Index of*).
- File *backup* database (`.sql`, `.bak`, `.zip`) atau file log konfigurasi.
- Mengidentifikasi modul pihak ketiga atau CMS rentan yang digunakan oleh pihak kampus.

Hanya bermodalkan pencarian cerdas di mesin pencari, berbagai informasi sensitif bisa terangkut semua tanpa membunyikan alarm *Firewall* atau *Intrusion Detection System* (IDS) di server kampus sama sekali.

---

## "Menu Langganan" Kerentanan di Web Kampus

Setelah target subdomain terlantar ditemukan, ini dia beberapa *vulnerability* legendaris yang paling sering dieksploitasi untuk membobol sistem:

### 1. Default Credential & Bypass Admin
Pernah kepikiran nggak, web jurnal atau perpus kampus jebol gara-gara password adminnya masih pakai bawaan pabrik seperti `admin` dan *username*-nya `admin`? Percaya atau nggak, ini kasus yang sangat, sangat umum! 

Selain *Default Credential*, kerentanan *Bypass Admin* juga sering terjadi akibat kesalahan logika kode. Misalnya, sistem validasi otorisasi sesi (*session*) yang lemah, di mana attacker cukup memodifikasi *cookie* secara lokal di browser (misal mengubah parameter dari `role=mahasiswa` menjadi `role=admin`), dan *boom!* Langsung dapet akses *dashboard* administrator tanpa perlu menebak password.

### 2. File Manager Pihak Ketiga yang Bolong
Aplikasi web sering kali butuh fitur manajemen konten untuk meng-*upload* gambar atau dokumen pengumuman. Banyak sistem kampus menggunakan modul pihak ketiga (*open-source*) lawas yang diintegrasikan ke *dashboard* mereka, seperti:
- **Kcfinder**
- **Fckeditor** atau **CKEditor** versi kuno
- **Elfinder**
- **Kindeditor**
- Plugin **Com_media** di beberapa CMS

Masalahnya, modul-modul *editor* lawas ini sering kali memiliki celah keamanan yang sangat fatal, yaitu **Arbitrary File Upload**. Sistem gagal menyaring ekstensi file dengan benar. Alih-alih meng-*upload* file gambar (`.jpg` atau `.png`), attacker malah bisa menyisipkan file bereksistensi `.php` (yang isinya kode *Webshell* atau *Backdoor*). Begitu file PHP itu dipanggil lewat URL, attacker otomatis memiliki kendali penuh ke server!

### 3. Eksploitasi CMS Kampus yang Usang (OJS, SLiMS, Balitbang, Drupal)
Website kampus sangat bergantung pada berbagai *Content Management System* (CMS) spesifik untuk kebutuhan akademik. Namun, CMS ini sangat rawan jika dibiarkan *outdated*:
- **OJS (Open Journal Systems)**: Sangat populer untuk publikasi jurnal dosen. Versi lawas OJS terkenal memiliki rentetan celah *Privilege Escalation* dan kelemahan di fitur unggah dokumennya.
- **SLiMS (Senayan Library Management System)**: Tulang punggung sistem perpustakaan di banyak universitas. Jika versinya tidak diperbarui, beberapa rilis lama memiliki *vulnerability* injeksi SQL.
- **CMS Balitbang**: Ini legenda banget di ekosistem web pendidikan Indonesia. Versi-versi tuanya memiliki celah yang dijuluki **"SQL Balitbang"** yang memungkinkan penyerang men-dump isi *database* dengan eksploitasi sederhana.
- **Drupal (Versi Jadul)**: CMS *powerful* namun di masa lalu punya sejarah kelam dengan eksploit bernama *Drupalgeddon*, yang memungkinkan attacker melakukan *Remote Code Execution* tanpa perlu login sama sekali.

### 4. Dari LFI (Local File Inclusion) Menuju RCE
*Local File Inclusion* (LFI) terjadi saat web server secara tidak sengaja mengizinkan pengguna dari luar untuk menginklusi dan membaca file-file sensitif di dalam *server local* (misalnya membaca file `/etc/passwd`).

Namun, di tangan attacker yang jago, LFI bukanlah tujuan akhir. Mereka bisa melakukan *chaining* (menggabungkan beberapa teknik celah) dari LFI menjadi **RCE (Remote Code Execution)**. Salah satu teknik terkenalnya adalah *Log Poisoning*: attacker menyisipkan *payload* PHP berbahaya ke dalam *file log* akses milik server apache/nginx, lalu memanggil *file log* tersebut melalui celah LFI. Saat log itu terinklusi, kode PHP di dalamnya akan tereksekusi.

**Apa bahayanya RCE?** 
*Remote Code Execution* adalah mimpi buruk terburuk bagi Admin Server. Attacker mendapatkan *Shell* atau koneksi terminal langsung (*Reverse Shell*) ke dalam server dan bisa mengeksekusi perintah sistem operasi selayaknya admin. Dari akses *user* web biasa, mereka tinggal mencari celah eskalasi hak istimewa (*Privilege Escalation*) untuk naik menjadi *Root* (seperti yang kita bahas di artikel Root Server sebelumnya wkwk).

---

## Strategi Pertahanan (Defense in Depth)

Buat teman-teman admin server kampus, tim IT *Support*, atau mahasiswa tingkat akhir yang mau bantu kampus berbenah, berikut adalah mitigasi yang terstruktur:

| Vektor Ancaman | Solusi & Mitigasi Aktif (Remediasi) |
| :--- | :--- |
| **Subdomain Terlantar** | Lakukan **Asset Inventory** rutin. Data semua subdomain. Segera matikan atau *takedown* aplikasi lawas yang sudah tidak dipakai (*Decommissioning*). |
| **Google Dorking** | Konfigurasi file `robots.txt` dengan benar. Lindungi direktori sensitif. Admin juga wajib melakukan *Google Dorking* ke domain kampusnya sendiri untuk deteksi dini. |
| **Default Credential** | Hapus akun bawaan setelah instalasi. Terapkan kebijakan *password* yang *complex* dan wajibkan aktivasi **Multi-Factor Authentication (MFA)** untuk panel admin. |
| **Celah Arbitrary Upload**| **Isolasi direktori upload!** Pastikan direktori tempat menyimpan *file* yang diunggah pengguna tidak bisa mengeksekusi script PHP. Contoh, gunakan `.htaccess` (`php_flag engine off`). |
| **CMS & Plugin Lawas** | Tetapkan jadwal *Patch Management* bulanan. Segera mutakhirkan (update) OJS, Drupal, dan SLiMS ke rilis terbaru. Jangan gunakan *file manager* usang yang sudah tidak diperbarui. |
| **LFI & RCE** | Konfigurasi file `php.ini` dengan prinsip *Hardening*. Nonaktifkan fungsi sistem berbahaya seperti `allow_url_include`, `system()`, `exec()`, dan `shell_exec()` jika tidak dibutuhkan aplikasi. |

Mengelola infrastruktur IT skala besar seperti di universitas memang kompleks. Tapi dengan memahami cara berpikir dan *offensive mindset* penyerang, institusi bisa menyusun prioritas perbaikan yang tepat sasaran demi melindungi data sivitas akademika.

---

> **Disclaimer**
> 
> Seluruh materi, konsep kerentanan, metodologi, dan penjelasan teoretis dalam artikel ini disusun **murni untuk tujuan edukasi dan peningkatan kesadaran keamanan siber (cybersecurity awareness)**. 
>
> Penulis sama sekali tidak mengajarkan, memberikan *payload* eksploitasi yang fungsional, apalagi menyarankan Anda untuk mencoba meretas website kampus mana pun. Melakukan aksi pencarian celah, *Google Dorking* dengan niat penyerangan, atau pengujian *vulnerability* pada sistem jaringan institusi **TANPA IZIN tertulis resmi (*Authorization* / *Rules of Engagement*) adalah TINDAKAN ILEGAL**. Pelakunya dapat dituntut sesuai dengan hukum pidana UU ITE. 
> 
> Gunakanlah ilmu keamanan siber untuk melindungi aset digital. Jadilah *Security Researcher* atau *Ethical Hacker* yang bertanggung jawab dan beretika!
