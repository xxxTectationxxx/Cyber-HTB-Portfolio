# 🚩 HackTheBox: Cap - Write-up

**Date:** 15 January 2026  
**Machine:** Cap  
**Difficulty:** Easy  
**Platform:** Linux  
**IP Address:** 10.10.10.245

---

## 📖 Executive Summary
Cap adalah mesin Linux tingkat mudah yang mengeksploitasi kerentanan pada dashboard pemantauan jaringan. Kerentanan utama yang ditemukan adalah **IDOR** (Insecure Direct Object Reference) yang memungkinkan akses ke data sensitif user lain, kebocoran kredensial melalui analisis **PCAP**, dan eskalasi hak akses menggunakan **Linux Capabilities**.

## ⭐ SET UP MECHINE
Pertama Pastikan Mechine sudah terkoneksi menggunakan OpenVPN samapai mendapatkan IP Address target  : 
<img width="1400" height="202" alt="image" src="https://github.com/user-attachments/assets/95bbe744-9691-4c06-8843-ee143b588a8c" />


## <br/> 🔎 Phase 1: Reconnaissance (Information Gathering)
### Mulai dari soal Pertama : 
#### 1.1 How many TCP ports are open?
Gunakan Tools Untuk Mendeteksi Service yang Berjalan : 
<br/><img width="614" height="143" alt="image" src="https://github.com/user-attachments/assets/eae248f1-80dc-4513-9fce-13aa6f485d89" />
<br/>Penjelasan : 
<br/>Disini saya menggunakan tools NMAP dengan command <ins>nmap 10.10.10.245<ins/> Untuk mengetahui jumlah port yang terbuka dari IP Address tersebut dan didapatkan JAWABAN = 3

### Lanjut Ke Soal Kedua : 
#### 1.2 After running a "Security Snapshot", the browser is redirected to a path of the format /[something]/[id], where [id] represents the id number of the scan. What is the [something]?
Akses Website HTTP pada IP Address 10.10.10.245, Kemudian Akses bagian NavBar Kemudian Security Snapshot
<br/><img width="942" height="654" alt="image" src="https://github.com/user-attachments/assets/92d36cd4-1ba8-4938-89eb-fa28b4e04b13" />
Penjelasan : 
Langkah ertama untuk meelakuakan information gathering yaitu mengetahu semua service yang ada di website sehingga bisa mengetahui kerentanan yang mungkin terjadi dari website tersebut dan dapat dari Serive Snapshot JAWABAN KEDUA = data

### Soal Ketiga : 
#### 1.3 Are you able to get to other users' scans? yes/no
Akses Halaman Snapshot : 
<img width="928" height="696" alt="image" src="https://github.com/user-attachments/assets/3be8a0b3-d79e-45ad-a094-8d66cc8e91a3" />
<br/>Penjelasan : 
<br/>Coba Lakukan Pergantian ID dari /data/<id> dan lihat output nya bisa melihat Snapshot user lain sehingga JAWABAN = yes

## <br/> 💥 Phase 2: IDOR Attack 
### Soal Keempat : 
#### 2.1 What is the ID of the PCAP file that contains sensative data?
Ganti User ID menjadi 0 : 
<br/><img width="930" height="694" alt="image" src="https://github.com/user-attachments/assets/ba0f5a83-2bdb-434c-8b6d-3574d4b2130f" />

### Soal Kelima : 
#### 2.2 Which application layer protocol in the pcap file can the sensetive data be found in?
Download PCAP Dan Buka menggunakan WireShark
<br/><img width="926" height="104" alt="image" src="https://github.com/user-attachments/assets/df51cc76-7afa-4f97-b1c3-d7a5461ac2ce" />
<br/>Penjelasan : 
<br/>Dengan bisa mengganti user ID menunjukkan bahwa target ini rentan dengan IDOR, untuk menemukan data sensitif anda dapat mengakses user ID = 0 dimana didalamnya terdapat username dan password dari FTP dengan username == nathan dan password = Buck3tH4TF0RM3! sehingga ini bisa menjawab Soal Keempat dan Kelima yaitu JAWABAN = 0 dan ftp

### Soal Keenam : 
#### 2.3 We've managed to collect nathan's FTP password. On what other service does this password work?
Coba Akses SSH 
<br/><img width="1061" height="617" alt="image" src="https://github.com/user-attachments/assets/4afc4162-b66c-4c4f-bf8b-90d9bdfe71d8" />
<br/>Penjelasan : 
<br/>Terkadang pemilik server menggunakan password yang sama walaupun servicenya berbeda "all door from 1 key" jadi JAWABANYA = ssh

### Soal Ketujuh : 
#### 2.4 Submit the flag located in the nathan user's home directory.
Akses folder /home/nathan dan baca file user.txt
<br/><img width="300" height="68" alt="image" src="https://github.com/user-attachments/assets/67ad92c3-bc75-4b48-aa34-3f70d5647f49" />
<br/>Jadi User Flag yaitu JAWABAN = 57de4594d0e5c56acd63ca2781964947

## <br/> 🔥 Phase 3 : PRIVILLAGE ESCALATION 
### Soal Kedelapan : 
#### 3.1 What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?
Cara Automation Menggunakan LinPeash : 
<br/><img width="977" height="114" alt="image" src="https://github.com/user-attachments/assets/9c2c37d9-6783-4ff9-824e-61106294ed1a" />
<br/><br/>Cara Manual : 
<br/><img width="971" height="112" alt="image" src="https://github.com/user-attachments/assets/c10abaa8-875c-499f-a249-4e228bf24ac4" />
<br/>Penjelasan : 
<br/>Jalankan Linpeash untuk mendapatkan ringkasan potensi jalur untuk privillage escalation, disini muncul path dimana kemungkinan 95% berhasil sehingga JAWABANNYA = /usr/bin/python3.8 atau kalau mau menggunakan cara manual bisa menggunakan command getcap -r / 2>/dev/null hasilnya sama saja


### Soal Kesembilan : 
#### 3.2 Submit the flag located in root's home directory.
Naikkan Privilage dengan Command /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")' dan Akses Flag pada /root/root.txt
<br/><img width="782" height="106" alt="image" src="https://github.com/user-attachments/assets/ff94dc8f-0adb-4c5c-8850-eca4c38f1c8c" />
<br/>Penjelasan : 
<br/>Dari temuan sebelumnya adanya Indikasi akses root muncul karena /usr/bin/python3.8 memiliki capability cap_setuid, yang secara teknis mengizinkan interpreter tersebut untuk memanipulasi User ID (UID) prosesnya sendiri. Dalam kondisi normal, hanya akun root yang bisa mengubah identitas proses menjadi user lain, namun karena flag +eip (Effective, Inheritable, Permitted) aktif pada Python, user biasa dapat menyalahgunakan fungsi ini dengan menjalankan perintah os.setuid(0) di dalam skrip Python untuk mengubah identitasnya menjadi root secara instan. Dari proses tersebut dapat JAWABAN = 046e6901b7455c3cfbe0b557157a9776


### <br/> 📔 Pelajaran Yang Didapat : 
Dari Lab ini saya belajar proses Information Gathering sederhana menggunakan NMAP kemudian Exploitasi menggunakan IDOR lalu menganalisa menggunakan Wireshark sampai pada Alur Proses Privillage Escalation dari User biasa sampai dapet Privillage Root
