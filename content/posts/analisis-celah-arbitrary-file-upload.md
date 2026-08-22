---
title: "Celah Arbitrary File Upload: Mengubah Fitur Upload Menjadi Remote Code Execution"
date: 2026-08-21
draft: false
description: "Edukasi mengenai cara kerja celah keamanan upload file, teknik bypass yang biasa digunakan peretas, dan langkah pencegahannya."
tags: ["web-security", "rce", "penetration-testing", "file-upload", "bug-bounty"]
categories: ["Cybersecurity", "Web Security"]
author: "Ravxy1337"
ShowToc: true
TocOpen: true
---

Fitur upload file pada aplikasi web sering kali menjadi target utama para peretas. Jika sistem keamanan server tidak menyaring file yang masuk dengan benar, peretas dapat mengunggah file script berbahaya. Dampak paling fatal dari celah ini adalah Remote Code Execution atau yang biasa disebut dengan RCE. RCE membuat peretas bisa menjalankan perintah apa saja langsung di dalam server target.

Celah ini dinamakan Arbitrary File Upload. Celah terjadi ketika website membiarkan pengguna mengunggah jenis file apa saja tanpa ada batasan yang ketat. Sebagai contoh, jika form upload foto profil menerima file berformat PHP atau ASP, peretas dapat mengunggah script backdoor. Setelah file berhasil terunggah, peretas hanya perlu mengakses link file tersebut untuk mulai mengendalikan server.

Banyak pengembang website mencoba mengamankan fitur ini dengan cara yang kurang tepat. Peretas biasanya mempunyai trik untuk melewati pengamanan tersebut. Berikut adalah beberapa teknik bypass yang sering ditemukan di dunia nyata.

1. **Mengubah Ekstensi File**
   Ketika server memblokir file berekstensi `.php`, peretas akan mencoba alternatif lain yang masih bisa dieksekusi oleh server. Contohnya adalah ekstensi `.php5`, `.phtml`, atau `.phar`.
2. **Memanipulasi Content Type**
   Server terkadang hanya memeriksa header tipe file saat diunggah. Peretas dapat mengubah header ini menjadi tipe gambar yang aman seperti `image/jpeg` padahal isi file sebenarnya adalah script php jahat.
3. **Menggunakan Double Extension**
   Peretas sering menggunakan nama file seperti `gambar.jpg.php` untuk mengelabui filter yang hanya membaca kata di tengah nama file.

Untuk mencegah celah berbahaya ini, sistem keamanan harus diterapkan pada beberapa lapisan. Cara terbaik adalah memproses ulang setiap file yang diunggah. Server harus mengubah nama file asli menjadi string acak dan menyimpan file tersebut di luar folder utama website. Selain itu, pastikan folder tempat menyimpan file unggahan tidak memiliki izin untuk mengeksekusi script.
