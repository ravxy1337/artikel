---
title: "Memahami SQL Injection dan Cara Mencegahnya"
date: 2026-09-08
author: "RavxyTech"
tags: ["sql-injection", "web-security", "pemula", "backend", "database", "php", "keamanan-web"]
categories: ["Web Security", "Edukasi"]
description: "Penjelasan SQL Injection dari nol untuk pemula, mulai dari konsep dasar, kenapa bisa terjadi, apa dampaknya, sampai cara mencegahnya dengan kode yang benar."
cover:
  image: "/images/sqli.png"
  alt: "Ilustrasi SQL Injection"
  caption: "SQL Injection adalah teknik serangan di mana penyerang menyisipkan atau \"menyuntikkan\" perintah SQL berbahaya ke dalam input yang diterima aplikasi."
  relative: false
draft: false
---

SQL Injection adalah salah satu kerentanan web yang paling lama dikenal, dan ironisnya masih menjadi salah satu yang paling sering ditemukan sampai sekarang. OWASP secara konsisten menempatkannya dalam daftar sepuluh risiko keamanan aplikasi web teratas selama bertahun-tahun.

Bagi yang baru belajar pengembangan backend, memahami celah ini sejak awal bukan pilihan, melainkan keharusan. Satu baris kode yang ditulis tanpa mempertimbangkan keamanan bisa membuka jalan bagi penyerang untuk mengakses seluruh isi database.

---

## Apa Itu SQL Injection

SQL Injection adalah teknik serangan di mana penyerang menyisipkan atau "menyuntikkan" perintah SQL berbahaya ke dalam input yang diterima aplikasi. Ketika aplikasi menggabungkan input tersebut langsung ke dalam query database tanpa validasi, perintah yang disisipkan ikut dieksekusi oleh database engine.

Intinya, penyerang berhasil mengubah struktur query SQL yang seharusnya sudah didefinisikan developer menjadi perintah yang berbeda sesuai keinginannya. Database tidak bisa membedakan mana perintah yang berasal dari developer dan mana yang berasal dari penyerang, karena keduanya diterima sebagai teks yang dieksekusi secara langsung.

---

## Kenapa SQL Injection Bisa Terjadi

Penyebabnya selalu sama di hampir setiap kasus: input dari pengguna digabungkan langsung ke dalam string query SQL tanpa melewati proses validasi atau sanitasi.

Ini bukan kelemahan dari sisi database engine seperti MySQL, PostgreSQL, atau SQLite. Database hanya mengeksekusi apa yang dikirimkan kepadanya. Masalahnya ada di sisi aplikasi, tepatnya pada cara developer membangun query tersebut.

Selama ada input dari pengguna yang langsung masuk ke query tanpa diproses terlebih dahulu, celah ini akan selalu ada.

---

## Contoh Kode yang Rentan

Berikut contoh implementasi form login menggunakan PHP yang rentan terhadap SQL Injection.

```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = mysqli_query($conn, $query);

if (mysqli_num_rows($result) > 0) {
    echo "Login berhasil!";
} else {
    echo "Username atau password salah.";
}
?>
```

Baris yang menjadi sumber masalah adalah:

```php
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

Nilai `$username` dan `$password` diambil langsung dari request POST dan ditempelkan ke dalam string query tanpa proses apapun. Apa yang pengguna kirimkan akan menjadi bagian dari perintah SQL yang dieksekusi database secara harfiah.

---

## Bagaimana Query Bisa Dimanipulasi

Anggap seorang penyerang mengisi kolom username dengan nilai berikut:

```
' OR '1'='1
```

Maka query yang terbentuk di sisi server akan menjadi:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = ''
```

Kondisi `'1'='1'` selalu bernilai benar. Akibatnya, database mengembalikan hasil meskipun tidak ada username maupun password yang cocok di tabel. Logika autentikasi yang dibangun developer berhasil dibypass hanya dengan memanfaatkan cara query itu dikonstruksi.

Pada skenario yang lebih serius, penyerang bisa menggunakan teknik seperti `UNION SELECT` untuk mengekstrak data dari tabel lain, atau menggunakan komentar SQL (`--`) untuk memotong bagian query yang tidak diinginkan.

---

## Dampak yang Bisa Ditimbulkan

Tingkat kerusakan yang bisa terjadi bergantung pada seberapa luas akses yang berhasil diperoleh penyerang.

Pada level paling dasar, data pengguna bisa diekstrak secara massal. Alamat email, nama lengkap, nomor telepon, bahkan password yang tidak di-hash dengan benar semuanya bisa dibaca. Pada level yang lebih dalam, penyerang bisa melakukan bypass autentikasi dan masuk sebagai pengguna mana saja termasuk akun administrator, mengubah atau menghapus data di database, sampai di beberapa konfigurasi server yang tidak dikonfigurasi dengan benar, membaca file dari sistem operasi lewat fungsi seperti `LOAD_FILE()`.

---

## Cara Mencegahnya

**Prepared Statement**

Ini adalah solusi utama dan yang paling penting diterapkan. Dengan prepared statement, struktur query didefinisikan terlebih dahulu menggunakan placeholder, dan data dari pengguna dimasukkan secara terpisah. Database memproses keduanya dalam konteks yang berbeda, sehingga data tidak bisa mengubah struktur perintah SQL.

Berikut perbaikan dari kode rentan di atas menggunakan prepared statement:

```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

// Query dibuat dengan placeholder, struktur sudah terkunci
$stmt = $conn->prepare("SELECT * FROM users WHERE username = ? AND password = ?");

// Data dimasukkan terpisah, tidak bisa mengubah struktur query
$stmt->bind_param("ss", $username, $password);
$stmt->execute();

$result = $stmt->get_result();

if ($result->num_rows > 0) {
    echo "Login berhasil!";
} else {
    echo "Username atau password salah.";
}
?>
```

Dengan pendekatan ini, input `' OR '1'='1` akan diperlakukan sepenuhnya sebagai nilai string biasa, bukan sebagai bagian dari perintah SQL. Serangan tidak akan berhasil karena struktur query sudah dikunci sejak awal.

**Validasi Input**

Prepared statement menangani SQL Injection, tapi validasi input tetap perlu dilakukan sebagai lapisan perlindungan tambahan. Pastikan tipe dan format data yang diterima sesuai dengan yang diharapkan. Angka harus benar-benar angka, email harus memiliki format yang valid, dan panjang input perlu dibatasi sesuai kebutuhan.

**ORM**

Framework modern seperti Laravel (Eloquent) atau Django (ORM bawaan) sudah menggunakan prepared statement secara otomatis di balik layar. Menggunakan ORM mengurangi risiko menulis query yang rentan secara signifikan. Meski begitu, jika ada kebutuhan untuk menulis raw query, kewaspadaan yang sama tetap perlu diterapkan.

**Prinsip Least Privilege**

Akun database yang digunakan aplikasi sebaiknya hanya memiliki izin yang benar-benar dibutuhkan. Jika aplikasi hanya perlu membaca dan menulis data, tidak perlu memberikan izin `DROP`, `ALTER`, atau akses ke database sistem. Dengan pembatasan ini, dampak dari eksploitasi yang berhasil pun bisa diminimalkan.

---

SQL Injection bukan kerentanan yang sulit dicegah. Seluruh solusinya sudah tersedia dan sudah didukung di hampir semua bahasa pemrograman dan framework. Yang membuatnya masih relevan sampai sekarang adalah banyak kode yang ditulis tanpa mempertimbangkan aspek keamanan sejak awal.

Kebiasaan menulis prepared statement, memvalidasi input, dan menerapkan prinsip least privilege perlu dibentuk dari awal proses belajar, bukan ditambahkan belakangan setelah aplikasi sudah selesai dibangun.

---

> **Disclaimer**
>
> Artikel ini ditulis untuk tujuan edukasi. Seluruh contoh dan penjelasan ditujukan agar pembaca bisa menulis kode yang lebih aman, bukan untuk panduan mengeksploitasi sistem orang lain. Uji keamanan hanya boleh dilakukan pada sistem yang dimiliki sendiri atau yang telah memberikan izin eksplisit.
>
> [ravxytech.site](https://ravxytech.site) hadir untuk berbagi pengetahuan seputar teknologi dan cyber security secara bertanggung jawab.
