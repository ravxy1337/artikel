---
title: "Git Exposed: Ketika Folder .git Terbuka untuk Umum"
date: 2026-06-25
author: "RavxyTech"
tags: ["git", "web-security", "pentesting", "recon", "disclosure", "red-team"]
categories: ["Research", "Web Security"]
description: "Folder .git yang tidak sengaja terbuka di server bisa mengekspos source code, kredensial database, API key, sampai riwayat seluruh perubahan kode. Artikel ini membahas cara menemukannya dan apa yang bisa didapat dari sana."
draft: false
---

Banyak developer yang tidak sadar bahwa proses deployment mereka meninggalkan sesuatu yang tidak seharusnya ada di server publik: folder `.git`. Folder ini adalah jantung dari version control Git, berisi riwayat lengkap semua perubahan kode, pesan commit, branch, bahkan file konfigurasi yang mungkin menyimpan hal-hal sensitif seperti kredensial database, API key, atau password admin.

Kalau folder ini bisa diakses siapa saja lewat browser, maka semua isi repositori bisa diunduh dan dipelajari oleh siapapun, termasuk yang tidak punya hak akses sama sekali.

---

## Kenapa Ini Bisa Terjadi

Git bekerja dengan menyimpan seluruh riwayat repositori di dalam satu folder bernama `.git` yang ada di root project. Saat developer melakukan deployment ke server web, kadang mereka langsung menyalin seluruh isi folder project termasuk folder `.git` itu sendiri ke dalam direktori publik web server.

Hasilnya, folder tersebut bisa diakses lewat URL seperti `https://target.com/.git/` dan siapapun bisa membaca isinya. Ini bukan kerentanan pada Git-nya sendiri, melainkan kesalahan konfigurasi pada proses deployment yang membiarkan direktori internal ikut terpublish.

Yang membuat temuan ini lebih dari sekadar "bocor source code" adalah isi dari commit history dan file konfigurasi yang ada di dalamnya. Developer sering menyimpan kredensial langsung di file konfigurasi, lalu kemudian menggantinya di commit berikutnya. Tapi riwayat commit tetap menyimpan versi lama yang berisi kredensial tersebut, dan semuanya bisa dibaca.

---

## Cara Menemukan Git yang Terekspos

Ada beberapa cara untuk menemukan `.git` yang terbuka, dari yang paling manual sampai yang paling otomatis.

**Akses Manual**

Cara paling sederhana adalah langsung membuka `/.git/` di browser. Kalau server menampilkan directory listing atau file `HEAD` dan `config` bisa dibaca, maka `.git` tersebut terekspos.

```
https://target.com/.git/
https://target.com/.git/HEAD
https://target.com/.git/config
```

**DotGit Browser Extension**

Ada ekstensi browser bernama DotGit yang tersedia untuk Firefox dan Chrome. Cara kerjanya sederhana: setiap kali mengunjungi sebuah website, ekstensi ini otomatis mengecek apakah `.git` di domain tersebut bisa diakses publik. Kalau terdeteksi, akan muncul notifikasi beserta opsi untuk langsung mengunduh repositorinya.

**Fuzzing dengan Feroxbuster**

Untuk pendekatan yang lebih sistematis, bisa menggunakan tools seperti `feroxbuster` yang melakukan brute-force direktori dan file berdasarkan wordlist. Feroxbuster ditulis dalam Rust sehingga cepat dan efisien untuk scanning rekursif.

```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/dirb/common.txt
```

Kalau `.git` muncul di hasil scan dengan status code 200 atau 403, itu sudah cukup jadi indikasi awal bahwa folder tersebut ada dan perlu diperiksa lebih lanjut.

---

## Mengunduh Repositori dengan Wget

Setelah memastikan bahwa `.git` memang bisa diakses, langkah berikutnya adalah mengunduh seluruh isinya. Perintah wget dengan opsi mirror bekerja dengan baik untuk ini karena secara rekursif mengunduh semua file yang ada di dalam folder tersebut.

```bash
wget --mirror -I .git https://target.com/.git/
```

Penjelasan singkat dari masing-masing opsi yang dipakai: `--mirror` membuat wget mengunduh secara rekursif semua konten yang ada, sementara `-I .git` membatasi path yang diunduh hanya pada direktori `.git` saja sehingga tidak mengunduh halaman web lainnya.

{{< figure src="/images/git-wget-download.png" alt="Proses download .git via wget" caption="wget mengunduh seluruh isi folder .git dari target secara rekursif." >}}

Setelah proses unduhan selesai, wget akan membuat folder dengan nama domain target di direktori lokal. Masuk ke folder tersebut untuk melanjutkan proses analisis.

---

## Memulihkan File dari Git

Setelah folder `.git` berhasil diunduh, langkah selanjutnya adalah memulihkan file-file yang tersimpan di dalam repositori tersebut. Pertama, cek dulu status repositori untuk melihat file apa saja yang tercatat.

```bash
cd nama-target.com
git status
```

{{< figure src="/images/git-status.png" alt="Output git status menampilkan file yang terhapus" caption="git status menampilkan daftar file yang tercatat di repositori tapi belum ada di direktori lokal." >}}

Biasanya akan terlihat daftar panjang file dengan status `deleted`. Artinya file-file tersebut tercatat di repositori Git tapi belum ada di direktori lokal karena yang diunduh hanya folder `.git`-nya saja, bukan working directory lengkapnya. Untuk memulihkan semua file tersebut, gunakan perintah berikut.

```bash
git restore .
```

Setelah perintah ini dijalankan, semua file yang tercatat di repositori akan dipulihkan ke direktori lokal.

{{< figure src="/images/git-restore-ls.png" alt="Hasil ls setelah git restore" caption="Seluruh file berhasil dipulihkan. Beberapa file konfigurasi langsung terlihat menarik untuk diperiksa." >}}

Dari hasil `ls -la`, struktur lengkap aplikasi sudah bisa dilihat. Ada folder `application`, `assets`, `libs`, file `index.php`, `captcha.php`, dan berbagai file lainnya. Untuk aplikasi yang menggunakan framework seperti CodeIgniter, folder `application/config/` adalah tempat pertama yang perlu diperiksa karena di sanalah file konfigurasi database biasanya disimpan.

---

## Membaca File Konfigurasi

File konfigurasi adalah tujuan utama dari proses pemulihan ini. Di aplikasi berbasis CodeIgniter misalnya, kredensial database disimpan di `application/config/database.php`.

{{< figure src="/images/git-database-config.png" alt="Isi file database.php yang berisi kredensial" caption="File konfigurasi database berisi hostname, username, password, dan nama database secara lengkap." >}}

Dari file ini bisa langsung terbaca `hostname`, `username`, `password`, dan nama `database` yang digunakan aplikasi. Informasi ini sudah cukup untuk mencoba koneksi langsung ke database jika port MySQL terbuka ke publik.

---

## Membaca Riwayat Commit

Selain file konfigurasi yang ada sekarang, Git juga menyimpan seluruh riwayat perubahan. Ini sangat berguna karena developer kadang menyimpan kredensial di commit lama sebelum akhirnya menggantinya. Untuk melihat daftar semua commit yang ada di repositori, gunakan perintah berikut.

```bash
git log
```

Output dari `git log` akan menampilkan daftar commit lengkap dengan hash, nama author, tanggal, dan pesan commit. Setiap pesan commit bisa memberikan petunjuk tentang perubahan apa yang dilakukan, misalnya "changed database password" atau "removed hardcoded credentials".

Untuk membaca detail perubahan di commit tertentu, gunakan hash commit yang didapat dari `git log`.

```bash
git show a1b2c3d4e5f6
```

Perintah ini akan menampilkan diff lengkap dari commit tersebut, termasuk baris-baris yang dihapus (ditandai dengan `-`) dan baris yang ditambahkan (ditandai dengan `+`). Dari sini bisa dilihat misalnya password lama yang sudah diganti, atau konfigurasi yang pernah ada di versi sebelumnya.

---

## Akses ke Database

Dengan kredensial yang sudah didapat dari file konfigurasi, langkah selanjutnya adalah mencoba koneksi ke database. Perlu dicatat bahwa file konfigurasi biasanya menggunakan `localhost` sebagai hostname karena memang diasumsikan diakses dari server itu sendiri.

Namun di beberapa kasus, ada file lain di repositori seperti `Envoy.blade.php` (untuk aplikasi Laravel) yang mendefinisikan server eksternal lengkap dengan IP-nya. Dari file seperti ini bisa didapat IP server yang sebenarnya.

Untuk mencoba koneksi ke database menggunakan kredensial yang ditemukan:

```bash
mysql -h IP_SERVER -u username -p
```

Kalau koneksi berhasil, maka seluruh database target sudah bisa diakses. Dari sini bisa dilihat tabel apa saja yang ada, termasuk tabel user yang mungkin menyimpan akun administrator aplikasi.

---

## Mitigasi

Ada dua pendekatan untuk mengatasi masalah ini. Yang pertama dan paling bersih adalah tidak menyertakan folder `.git` sama sekali saat deployment ke server produksi. Gunakan pipeline CI/CD yang hanya mendeploy file yang diperlukan, bukan seluruh isi repositori.

Yang kedua adalah memblokir akses ke folder `.git` di level konfigurasi web server. Berikut konfigurasi untuk beberapa web server yang umum digunakan.

Untuk **Apache 2.4**, tambahkan di `httpd.conf` atau file `.htaccess`:

```apache
<DirectoryMatch "^/.*/\.git/">
    Require all denied
</DirectoryMatch>
```

Untuk **Apache 2.2**, konfigurasinya sedikit berbeda:

```apache
<DirectoryMatch "^/.*/\.git/">
    Order deny,allow
    Deny from all
</DirectoryMatch>
```

Untuk **Nginx**, tambahkan di dalam blok `server` di `nginx.conf`:

```nginx
location ~ /\.git {
    deny all;
}
```

Untuk **Lighttpd**, tambahkan modul access di konfigurasi:

```
server.modules += ( "mod_access" )
```

Selain memblokir akses, penting juga untuk melakukan audit pada repositori itu sendiri. Kalau pernah ada kredensial yang dicommit, meskipun sudah dihapus di commit berikutnya, commit lama tetap menyimpannya. Gunakan tools seperti `git-secrets` atau `trufflehog` untuk memindai riwayat commit dan mendeteksi apakah ada data sensitif yang pernah dicommit.

```bash
# Scan repositori dengan trufflehog
trufflehog git file://./
```

---

Temuan ini mungkin terlihat sederhana, tapi dampaknya bisa sangat serius. Dari satu folder `.git` yang terbuka, seorang penyerang bisa mendapatkan source code lengkap, riwayat seluruh perubahan, kredensial database, API key, dan berbagai informasi sensitif lainnya tanpa perlu mengeksploitasi satu vulnerability pun. Semua data itu memang sudah ada di sana, dan yang dibutuhkan hanya sebuah URL yang tepat.

---

> **Peringatan**
>
> Seluruh teknik yang dijelaskan dalam artikel ini hanya boleh dipraktikkan pada sistem yang kamu miliki sendiri, lingkungan lab yang sudah disiapkan untuk pengujian, atau dalam program bug bounty dengan scope yang jelas dan izin tertulis dari pemilik sistem. Mengakses sistem orang lain tanpa izin adalah tindakan ilegal dan bisa dikenai sanksi hukum.
>
> Artikel ini dibuat murni untuk keperluan edukasi keamanan siber di [ravxytech.site](https://ravxytech.site). Pahami risikonya, lakukan dengan bertanggung jawab.