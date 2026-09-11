---
title: "Mau Belajar Cyber Security Tapi Bingung Mulai Dari Mana? Baca Ini Dulu"
date: 2026-09-12
author: "RavxyTech"
tags: ["cyber-security", "roadmap", "red-team", "blue-team", "pemula", "belajar", "pentesting", "soc", "sertifikasi", "career"]
categories: ["Edukasi", "Cybersecurity"]
description: "Panduan lengkap buat kamu yang mau belajar cyber security tapi masih bingung harus mulai dari mana, mau ambil jalur red team atau blue team, dan harus pakai sumber belajar apa. Semua dibahas dari nol sampai level lanjutan."
cover:
  image: "/images/roadmap.png"
  alt: "Roadmap Belajar Cyber Security"
  caption: "Panduan belajar cyber security dari nol untuk red team dan blue team."
  relative: false
draft: false
ShowToc: true
TocOpen: true
---

Siapin dulu kopi dan rokoknya. Kalau bukan perokok ya gapapa, yang penting siapin minuman anget karena artikel ini bakal panjang.

Artikel ini saya tulis karena pertanyaan "mau belajar cyber security, harus mulai dari mana?" itu terus muncul. Jadi sekalian saya buat panduan yang lengkap, dari nol sampai ke spesialisasi meskipun aku bukan spesialis wkwkwk, supaya bisa jadi pegangan buat siapa aja yang serius mau masuk ke bidang ini.

Langsung aja ya.

---

## Red Team dan Blue Team

Di cyber security ada dua kubu yaitu: **Red Team** dan **Blue Team**.

**Red Team** itu tim penyerang. Bukan menyerang dalam artian merusak, tapi mensimulasikan serangan nyata ke infrastruktur atau aplikasi sebuah organisasi. Tujuannya supaya celah keamanan ditemukan duluan sebelum penyerang asli yang nemuin. Orang-orang di jalur ini biasa disebut penetration tester atau ethical hacker.

**Blue Team** kebalikannya. Mereka yang bertahan, mendeteksi, dan merespons serangan. Kalau red team cari celah, blue team yang memastikan celah itu tertutup. Peran yang masuk sini antara lain SOC analyst, incident responder, threat hunter, dan digital forensics analyst.

Gampangnya, red team itu penguji gembok, blue team yang bikin dan jaga gemboknya. Dua-duanya harus ada supaya keamanan beneran teruji.

Kalian nggak perlu langsung tentuin mau ambil jalur yang mana sekarang. Baca dulu semuanya, baru tentukan belakangan.

---

## Fondasi Dasar yang Wajib Dikuasai Dulu

Mau pilih red team atau blue team, ada beberapa fondasi yang harus dikuasai lebih dulu. Banyak orang gagal bukan karena materinya susah, tapi karena langsung loncat ke topik lanjutan tanpa pondasi yang kuat.

### Jaringan Komputer (Networking Dasar)

Hampir semua hal di cyber security berhubungan dengan jaringan. Serangan terjadi lewat jaringan, deteksi juga lewat jaringan. Kalau belum paham bagaimana data berpindah dari satu komputer ke komputer lain, pasti akan kesulitan di topik-topik berikutnya.

Yang perlu dipelajari di tahap ini: model OSI dan TCP/IP, cara kerja protokol umum seperti HTTP, HTTPS, DNS, DHCP, FTP, SSH, dan TCP/UDP. Lalu pahami juga IP addressing dan subnetting, karena ini fondasi untuk mengerti bagaimana jaringan dibagi dan diatur.

Untuk belajar, kursus gratis dari **Cisco Networking Academy** lewat program "Introduction to Networks" sudah sangat solid. Kalau lebih suka video, channel YouTube **NetworkChuck** menjelaskan networking dengan cara yang ringan. Buku **"Computer Networking: A Top-Down Approach"** karya Kurose dan Ross juga referensi standar yang dipakai banyak kampus di seluruh dunia.

Sertifikasi **CompTIA Network+** cocok kalau mau memvalidasi pemahaman networking dasar. Materi persiapannya sekaligus bisa jadi bahan belajar yang terstruktur.

### Sistem Operasi dan Linux

Linux itu akan dipakai setiap hari, mau jadi red team maupun blue team. Kebanyakan server berjalan di atas Linux, tools pentesting mayoritas dikembangkan untuk Linux, dan tools monitoring juga banyak yang native di Linux.

Yang perlu dikuasai bukan cuma bisa buka terminal, tapi benar-benar paham navigasi filesystem, permission dan ownership, cara kerja proses, manajemen user, memanipulasi teks di terminal (grep, awk, sed, cat, less), dan mengelola software lewat package manager.

Cara paling efektif ya langsung pakai. Install distribusi Linux di virtual machine pakai VirtualBox atau VMware, terus gunakan sehari-hari. Ubuntu atau Debian cocok untuk pemula. Jangan langsung pakai Kali Linux kalau belum terbiasa dengan Linux secara umum, Kali itu dirancang sebagai tools pentesting, bukan sebagai OS belajar.

**OverTheWire** punya seri challenge namanya "Bandit" yang bagus banget untuk mengasah command line Linux dari nol. Di **TryHackMe** juga ada room "Linux Fundamentals" yang cocok untuk pemula. Buku **"The Linux Command Line"** karya William Shotts tersedia gratis di website-nya dan isinya lengkap.

### Dasar Pemrograman dan Scripting

Nggak harus jadi developer handal, tapi harus bisa menulis dan membaca kode. Di red team, kita perlu menulis script untuk automasi, modifikasi exploit, atau bikin tools sederhana. Di blue team, kita perlu script untuk parsing log, automasi analisis, atau integrasi antar tools.

Bahasa yang paling direkomendasikan pertama kali adalah **Python**. Sintaksnya mudah, komunitasnya besar, library-nya banyak termasuk yang spesifik untuk security, dan hampir semua tools cyber security punya komponen Python. Selain Python, pelajari juga **Bash scripting** karena ini bahasa native untuk automasi di Linux.

Untuk Python dari nol, **"Automate the Boring Stuff with Python"** karya Al Sweigart tersedia gratis online dan pendekatannya sangat praktikal. Kalau sudah nyaman dengan dasarnya, lanjut pelajari library yang sering dipakai di security seperti `requests`, `socket`, `scapy`, dan `subprocess`.

### Konsep Dasar Keamanan Informasi

Sebelum masuk ke hal teknis yang lebih dalam, kita perlu paham dulu konsep dasar keamanan informasi secara umum.

Yang paling fundamental adalah **CIA Triad**: Confidentiality (kerahasiaan), Integrity (integritas), dan Availability (ketersediaan). Setiap ancaman keamanan pada dasarnya menyerang salah satu atau lebih dari tiga aspek ini. Pelajari juga jenis ancaman umum seperti malware, phishing, social engineering, dan denial of service. Pahami juga dasar kriptografi, minimal perbedaan enkripsi simetris dan asimetris, hashing, dan digital signature. Nggak perlu jadi ahli matematika, tapi perlu tahu prinsip kerjanya.

**CompTIA Security+** sangat cocok untuk tahap ini. Materinya mencakup hampir semua konsep dasar keamanan yang perlu diketahui, dan sertifikasi ini sering jadi syarat minimum untuk posisi entry-level di industri. Buku **"CompTIA Security+ Get Certified Get Ahead"** karya Darril Gibson salah satu bahan persiapan yang paling direkomendasikan. Video dari **Professor Messer** di YouTube juga gratis dan mengikuti silabus Security+ secara lengkap.

---

## Roadmap Jalur Red Team

Buat yang tertarik sisi ofensif, suka cari celah dan mecahin puzzle, jalur ini kemungkinan cocok. Roadmap ini disusun bertahap dari paling dasar sampai lanjutan.

### Tahap 1: Pengenalan Penetration Testing dan Metodologi

Sebelum langsung terjun ke tools, kita perlu paham dulu apa itu penetration testing, bagaimana prosesnya, dan apa aturan mainnya. Pentesting bukan cuma "coba-coba bobol", ada metodologi yang harus diikuti.

Tahapan standarnya: reconnaissance (pengumpulan informasi), scanning dan enumeration, exploitation, post-exploitation (apa yang dilakukan setelah berhasil masuk), dan reporting (pembuatan laporan). Setiap tahapan punya tujuan spesifik dan saling terkait.

Pahami juga perbedaan black box testing (penguji nggak dikasih info apapun), white box testing (penguji dikasih akses penuh ke source code), dan grey box testing (kombinasi keduanya). Dan yang penting, pahami rules of engagement dan kenapa izin tertulis sebelum pengujian itu krusial.

TryHackMe punya learning path **"Jr Penetration Tester"** yang dirancang untuk pemula. Bisa juga mulai baca standar industri seperti **OWASP Testing Guide** dan **PTES (Penetration Testing Execution Standard)**.

### Tahap 2: Reconnaissance dan Information Gathering

Tahap ini sering diremehkan padahal jadi fondasi setiap engagement pentesting. Semakin banyak informasi yang dikumpulkan tentang target, semakin besar peluang nemuin celah. Penyerang di dunia nyata menghabiskan sebagian besar waktunya di tahap ini.

Ada dua jenis. **Passive reconnaissance** itu pengumpulan informasi tanpa interaksi langsung dengan target, misalnya WHOIS lookup, DNS enumeration, Google dorking, analisis metadata dokumen, dan OSINT. **Active reconnaissance** melibatkan interaksi langsung, seperti port scanning pakai Nmap, banner grabbing, dan service enumeration.

Tools yang perlu dipelajari: **Nmap** untuk network scanning (ini tools yang bakal terus dipakai sepanjang karir), **theHarvester** untuk ngumpulin email dan subdomain, **Shodan** untuk cari perangkat yang terekspos di internet, dan **Recon-ng** sebagai framework OSINT.

### Tahap 3: Web Application Security

Area ini luas banget dan jadi salah satu spesialisasi paling banyak diminati di pentesting. Alasannya simpel: hampir semua organisasi punya aplikasi web, dan hampir semua aplikasi web punya celah.

Topik yang perlu dikuasai bertahap: SQL Injection (sudah pernah dibahas detail di artikel terpisah di blog ini), Cross-Site Scripting (XSS), CSRF, SSRF, IDOR, broken authentication, file upload vulnerabilities, command injection, dan LFI/RFI. Daftarnya panjang, tapi nggak perlu dikuasai semuanya sekaligus. Mulai dari yang paling umum dulu.

**PortSwigger Web Security Academy** adalah sumber belajar gratis terbaik untuk topik ini. Setiap topik ada teorinya, lalu disertai lab interaktif yang bisa dikerjakan langsung di browser. Kalau serius di web security, habiskan waktu yang cukup di sini. Buku **"The Web Application Hacker's Handbook"** karya Dafydd Stuttard dan Marcus Pinto (penulis Burp Suite) juga referensi klasik yang wajib dibaca. Tools utamanya **Burp Suite**, proxy interceptor yang jadi standar industri untuk web app testing.

Untuk latihan, **HackTheBox** dan **TryHackMe** punya banyak machine yang fokus di web vulnerabilities. OWASP juga punya **Juice Shop** dan **DVWA** yang bisa di-install sendiri di lab lokal.

### Tahap 4: Network Penetration Testing

Setelah cukup nyaman dengan web app security, saatnya masuk ke infrastruktur jaringan. Di dunia nyata, pentesting nggak cuma dilakukan terhadap aplikasi web, tapi juga terhadap server, router, switch, dan layanan jaringan lainnya.

Di tahap ini kita belajar service exploitation, yaitu mengeksploitasi layanan yang jalan di server seperti SMB, FTP, SSH, RDP, dan sejenisnya. Pelajari juga password attacks baik online (brute force terhadap layanan) maupun offline (cracking hash yang sudah diperoleh). Dan pelajari dasar **Metasploit Framework**, salah satu tools paling populer untuk exploitation.

**HackTheBox** jadi platform utama untuk latihan. Mulai dari machine rating "Easy", kerjakan satu per satu, dan biasakan menulis writeup setiap kali berhasil menyelesaikan sebuah machine. Menulis writeup bukan cuma dokumentasi, tapi memaksa kita benar-benar paham setiap langkahnya.

Sertifikasi **eJPT (eLearnSecurity Junior Penetration Tester)** dari INE Security bagus banget sebagai sertifikasi pentesting pertama. Ujiannya berbasis praktik, bukan cuma pilihan ganda, dan materinya sudah termasuk dalam paket INE. Cocok untuk yang sudah selesai fondasi dasar dan mau validasi kemampuan pentesting level awal.

### Tahap 5: Privilege Escalation

Ketika berhasil dapat akses awal ke sebuah sistem, biasanya masuknya sebagai user dengan hak akses terbatas. Privilege escalation itu proses menaikkan hak akses dari user biasa jadi administrator (root di Linux, SYSTEM di Windows). Skill ini krusial karena nilai penetration test sering diukur dari seberapa dalam kita bisa masuk.

Untuk **Linux privilege escalation**: exploiting SUID binaries, cron job abuse, kernel exploits, sudo misconfigurations, dan PATH hijacking. Untuk **Windows**: token impersonation, service exploitation, unquoted service paths, DLL hijacking, dan SeImpersonatePrivilege abuse.

Tools yang perlu dikenal: **LinPEAS** dan **WinPEAS** untuk automasi enumeration, **GTFOBins** (referensi cara exploit binary Linux yang punya permission lebih), **PowerUp** dan **SharpUp** untuk enumeration di Windows. Kursus Tib3rius di Udemy tentang privilege escalation sangat praktikal dan langsung bisa diterapkan.

### Tahap 6: Active Directory Attacks

Kalau mau serius di pentesting, Active Directory (AD) nggak bisa dihindari. Mayoritas perusahaan menengah dan besar menggunakan AD untuk mengelola user, komputer, dan resource di jaringan internal. Karena itu, sebagian besar engagement pentesting di enterprise pasti melibatkan AD.

Yang dipelajari: cara kerja AD secara fundamental termasuk domain, forest, trust, Group Policy, Kerberos authentication, dan LDAP. Lalu teknik serangan seperti AS-REP Roasting, Kerberoasting, Pass-the-Hash, Pass-the-Ticket, DCSync, Golden Ticket, Silver Ticket, dan lateral movement.

Tools standarnya: **Impacket** (kumpulan Python script untuk interaksi dengan protokol Windows), **BloodHound** (pemetaan jalur serangan di AD), **Rubeus** (serangan Kerberos), dan **CrackMapExec** (Swiss army knife pentesting AD).

Kursus **Practical Ethical Hacking** dari TCM Security (Heath Adams) salah satu yang paling direkomendasikan untuk AD attack dari nol. TryHackMe punya room "Active Directory Basics" dan seri "Attacking Active Directory". HackTheBox juga banyak machine bertema AD, terutama tingkat Medium ke atas.

Sertifikasi **PNPT (Practical Network Penetration Tester)** dari TCM Security sangat relevan di tahap ini. Ujiannya skenario nyata di mana kita harus melakukan full pentest termasuk AD exploitation, dan di akhir harus nulis laporan profesional.

### Tahap 7: Level Lanjutan dan Spesialisasi

Kalau sudah melewati semua tahap di atas, fondasinya sudah kuat. Dari sini bisa pilih mau mendalami spesialisasi tertentu atau naik level.

**Exploit development** salah satu jalur lanjutan paling menantang. Belajar buffer overflow, format string vulnerabilities, return-oriented programming (ROP), dan bikin exploit dari nol. Butuh pemahaman mendalam tentang assembly language, arsitektur CPU, dan cara kerja memori.

**Mobile app pentesting** juga makin diminati, menganalisis dan mengeksploitasi aplikasi Android dan iOS. **Cloud security pentesting** makin relevan karena banyak infrastruktur pindah ke AWS, Azure, dan GCP.

Untuk sertifikasi lanjutan, **OSCP (Offensive Security Certified Professional)** adalah standar emas di industri pentesting. Ujiannya 24 jam, harus exploit beberapa mesin lalu nulis laporan detail. Bukan sertifikasi yang gampang, tapi kalau fondasi dari tahap sebelumnya solid, modalnya sudah cukup. Persiapan yang umum dilakukan: machine di HackTheBox dan **Proving Grounds** dari Offensive Security, serta kursus **PEN-200**.

Setelah OSCP ada **OSWE** untuk web app security mendalam, **OSEP** untuk advanced exploitation, dan **CRTO (Certified Red Team Operator)** dari Zero-Point Security untuk red team operations.

---

## Roadmap Jalur Blue Team

Buat yang lebih tertarik sisi pertahanan, suka analisis data dan pecahin misteri dari log, jalur blue team bisa jadi pilihan. Roadmap ini juga disusun bertahap.

### Tahap 1: Memahami Ancaman dan Konsep Pertahanan

Sebelum bisa bertahan, kita perlu tahu dulu dari apa kita bertahan.

Pelajari jenis serangan yang paling umum: phishing, malware, ransomware, denial of service, sampai supply chain attack. Pahami **Cyber Kill Chain** dari Lockheed Martin yang menggambarkan tahapan serangan dari reconnaissance sampai exfiltration. Framework lain yang penting adalah **MITRE ATT&CK**, knowledge base yang mendokumentasikan teknik dan taktik penyerang di dunia nyata. Framework ini akan terus dipakai sepanjang karir di blue team.

Pahami juga konsep defense-in-depth (pertahanan berlapis), prinsip least privilege, dan network segmentation.

TryHackMe punya learning path **"SOC Level 1"** yang jadi titik awal yang baik. **LetsDefend** juga punya modul pengenalan bagus untuk paham workflow blue team analyst.

### Tahap 2: Log Analysis dan Network Traffic Analysis

Ini skill inti setiap blue team analyst. Semua aktivitas di jaringan dan sistem meninggalkan jejak dalam bentuk log, dan tugas kita membaca jejak itu untuk mendeteksi aktivitas mencurigakan.

Mulai dengan memahami jenis log: log OS (Windows Event Logs, syslog di Linux), log aplikasi (web server, database), log jaringan (firewall, proxy), dan log keamanan (IDS/IPS). Pelajari cara membacanya, apa yang dicari, dan bagaimana identifikasi pola serangan.

Untuk network traffic analysis, **Wireshark** wajib dikuasai. Wireshark memungkinkan kita capture dan analisis paket jaringan secara detail. Perlu bisa baca file PCAP, identifikasi protokol, filter traffic, dan kenali pola anomali. Pelajari juga **tcpdump** untuk capture di command line karena di banyak server nggak akan ada GUI.

Tools lain: **Zeek** (sebelumnya Bro), network security monitor yang bisa analisis traffic real-time. Buku **"The Practice of Network Security Monitoring"** karya Richard Bejtlich referensi klasik untuk topik ini.

**CyberDefenders** punya challenge yang fokus di log analysis dan traffic analysis, setiap challenge menyediakan dataset yang harus dianalisis untuk menjawab pertanyaan investigasi.

### Tahap 3: SIEM (Security Information and Event Management)

Di dunia nyata, SOC analyst nggak baca log manual satu per satu. Volume log organisasi besar bisa jutaan event per hari. SIEM itu platform yang ngumpulin log dari berbagai sumber, normalisasi datanya, lalu menyediakan kemampuan pencarian, korelasi, dan alerting.

SIEM yang paling banyak dipakai: **Splunk** dan **Microsoft Sentinel** (untuk Azure). Splunk jadi standar de facto di banyak SOC, dan kemampuan pakai **SPL (Search Processing Language)** sangat dicari. Ada juga **Elastic SIEM** (Elasticsearch, Logstash, Kibana) yang open-source dan makin banyak dipakai.

Yang dipelajari bukan cuma cara pakai tools-nya, tapi juga konsepnya: bagaimana log dikumpulkan dan dinormalisasi, cara nulis detection rules yang efektif, mengelola alert supaya nggak kebanyakan false positive, dan cara investigasi pakai SIEM waktu ada alert.

Splunk punya platform belajar gratis **Splunk Education** dan sertifikasi **Splunk Core Certified User**. Di TryHackMe ada room Splunk dan ELK Stack. **LetsDefend** juga punya simulasi SOC environment lengkap dengan SIEM, bisa latihan nangani alert seperti SOC analyst sungguhan.

### Tahap 4: Incident Response

Ketika serangan terdeteksi atau bahkan sudah terjadi dan baru disadari belakangan, seseorang harus merespons. Itu tugas incident responder. Bukan cuma "matiin servernya", tapi bagaimana merespons secara terstruktur supaya dampak diminimalkan dan bukti diamankan.

Pelajari framework standar seperti **NIST SP 800-61** (Computer Security Incident Handling Guide). Framework ini membagi proses jadi empat fase: preparation, detection and analysis, containment/eradication/recovery, dan post-incident activity.

Perlu juga paham triage, yaitu menilai seberapa serius insiden dan tentukan prioritas. Harus bisa bedakan false positive dan insiden nyata, lalu ambil tindakan yang tepat.

Skill teknis yang dibutuhkan: forensik dasar pada sistem yang terkompromi (ambil memory dump, disk image, analisis artefak), chain of custody untuk preservasi bukti, dan menulis incident report yang jelas.

Buku **"Incident Response & Computer Forensics"** karya Jason Luttgens, Matthew Pepe, dan Kevin Mandia (pendiri Mandiant) referensi standar. **"Blue Team Handbook: Incident Response Edition"** karya Don Murdoch formatnya lebih ringkas dan cocok untuk referensi cepat.

### Tahap 5: Digital Forensics

Digital forensics itu proses mengumpulkan, menganalisis, dan menyajikan bukti digital dari sistem yang terkompromi. Kalau incident response fokus pada respons secepat mungkin, forensics lebih ke investigasi mendalam untuk paham apa yang sebenarnya terjadi.

Yang dipelajari: disk forensics (analisis hard drive, file yang dihapus, timeline aktivitas), memory forensics (analisis RAM dump, proses berbahaya, koneksi mencurigakan), dan network forensics (analisis traffic capture untuk rekonstruksi kronologi serangan).

Tools-nya: **Autopsy** dan **FTK Imager** untuk disk forensics, **Volatility** untuk memory forensics (standar industri, open-source), dan **Wireshark** untuk network forensics.

**CyberDefenders** dan **Blue Team Labs Online (BTLO)** punya challenge forensics yang bagus. BTLO khususnya menyediakan skenario investigasi yang cukup realistis.

Sertifikasi **BTL1 (Blue Team Level 1)** dari Security Blue Team bagus sebagai sertifikasi blue team pertama. Ujiannya berbasis skenario praktikal, harus investigasi insiden dan susun laporan. **CompTIA CySA+ (Cybersecurity Analyst+)** juga relevan, fokusnya lebih luas mencakup threat detection, analysis, dan response.

### Tahap 6: Threat Hunting

SOC analyst bekerja reaktif (respons alert yang masuk), threat hunter bekerja proaktif. Threat hunting itu proses cari ancaman yang mungkin sudah ada di jaringan tapi belum terdeteksi tools keamanan. Ini skill lanjutan yang butuh kombinasi semua kemampuan dari tahap sebelumnya.

Threat hunter harus paham mendalam tentang TTP (taktik, teknik, prosedur) penyerang. Di sini **MITRE ATT&CK** jadi sangat penting. Harus bisa pakai framework ini untuk bentuk hipotesis tentang teknik penyerang, lalu cari buktinya di log.

Pelajari juga **threat intelligence**, bagaimana kumpulkan dan pakai informasi tentang ancaman (IOC, TTPs, campaign, actor profiling) untuk perkuat deteksi. Platform seperti **MISP** dan **OpenCTI** dipakai untuk kelola dan berbagi threat intelligence.

Buku **"Crafting the InfoSec Playbook"** karya Jeff Bollinger, Brandon Enright, dan Matthew Valites bagus untuk paham cara bangun kapabilitas deteksi dan hunting.

### Tahap 7: Level Lanjutan dan Spesialisasi

Seperti red team, blue team juga punya spesialisasi lanjutan.

**Malware analysis** salah satu yang paling menantang. Analisis malware secara statis (baca kode tanpa jalankan) dan dinamis (jalankan di environment terkontrol, amati perilaku). Butuh pemahaman assembly, reverse engineering, dan tools seperti **Ghidra** (gratis dari NSA), **IDA Pro**, dan **x64dbg**. Buku **"Practical Malware Analysis"** karya Michael Sikorski dan Andrew Honig masih sangat relevan.

**Security engineering dan architecture** untuk yang mau fokus merancang infrastruktur keamanan, desain jaringan aman, hardening sistem, dan evaluasi arsitektur.

**Cloud security** makin penting di blue team. Monitor, deteksi, dan respons insiden di AWS, Azure, GCP.

Sertifikasi lanjutan: **BTL2** dari Security Blue Team, **GCIH (GIAC Certified Incident Handler)** dan **GCFA (GIAC Certified Forensic Analyst)** dari SANS. Sertifikasi SANS kualitasnya tinggi meskipun biayanya juga tinggi.

---

## Tips Tambahan

### Gabung Komunitas

Belajar sendirian bisa, tapi belajar dalam komunitas jauh lebih efektif. Gabung server Discord atau forum yang fokus di cyber security. Di Indonesia ada beberapa komunitas yang aktif, dan di TryHackMe maupun HackTheBox juga ada forum diskusi. Jangan malu tanya, dan kalau sudah bisa sesuatu, jangan pelit berbagi, ntah cari di forum lain juga boleh biasa nya di kampus2 gitu ada, + kalo mau roadmap yang jelas ke web **roadmap.sh** ya abang ku.

### Bikin Catatan Belajar

Kedengarannya sepele tapi dampaknya besar. Catat semua yang dipelajari, entah di Notion, Obsidian, atau blog pribadi. Menulis ulang apa yang sudah dipelajari dengan bahasa sendiri bikin pemahaman jauh lebih dalam dibanding cuma baca dan nonton. Catatan ini juga jadi referensi pribadi yang berguna banget di kemudian hari.

### Latihan Rutin

Lebih baik latihan satu jam tiap hari daripada sepuluh jam sekali seminggu. Konsistensi ngalahin intensitas. Kerjakan satu room di TryHackMe tiap hari, atau satu challenge di CyberDefenders tiap minggu. Ikut CTF (Capture The Flag) kalau ada kesempatan, cara yang menyenangkan untuk asah skill sambil kompetisi. **CTFtime.org** bisa bantu cari kompetisi CTF yang sedang atau akan berlangsung.

### Sabar

Ini tips paling penting. Banyak yang mulai semangat tapi berhenti di tengah jalan karena kewalahan. Lihat orang lain sudah rooting machine, sementara masih struggle paham subnetting. Itu wajar. Tiap orang punya pace belajar masing-masing. Yang penting bukan seberapa cepat sampai, tapi terus jalan.

Jangan bandingin progress sama orang lain. Orang yang keliatan jago di Twitter atau LinkedIn juga pernah ada di posisi yang sama. Mereka cuma sudah mulai lebih dulu.

---

> **Disclaimer**
>
> Artikel ini ditulis untuk tujuan edukasi. Seluruh referensi, platform, dan sertifikasi yang disebutkan adalah informasi publik yang tersedia secara umum. Belajar dan berlatih cyber security hanya boleh dilakukan secara legal, pada sistem yang dimiliki sendiri atau yang telah memberikan izin eksplisit.
>
> [ravxytech.site](https://ravxytech.site) hadir untuk berbagi pengetahuan seputar teknologi dan cyber security secara bertanggung jawab.
