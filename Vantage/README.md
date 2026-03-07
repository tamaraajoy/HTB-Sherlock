## 📝 Informasi Challenge
* **Nama Challenge**: Vantage
* **Kesulitan**: very easy
* **Kategori**: Digital Forensics & Incident Response (DFIR)
* **Author**: xenophon32

### 📖 Sherlock Scenario
Sebuah perusahaan kecil baru saja memindahkan beberapa aset digital mereka ke instalasi private cloud. Namun, tim pengembang (developers) melakukan kesalahan fatal dengan meninggalkan halaman pengalihan (redirect) menuju dashboard admin yang masih aktif di server web publik mereka.

Tim keamanan kemudian menerima email ancaman dari seseorang yang mengaku sebagai penyerang. Penyerang tersebut mengklaim bahwa mereka telah berhasil menemukan dashboard tersebut dan membocorkan data pengguna perusahaan

---

## 🔍 Analisis Step-by-Step

### Task 1: Tool Penyerang
**Pertanyaan**: Alat apa yang digunakan penyerang untuk melakukan fuzz pada server web? (Format- sertakan versi misalnya, nmap@7.80)

**Analisis**: karena attacker terus menerus nanya ke server "ada folder/admin nggak?" maka ini disebut request. kita cuma mau liat obrolan yang pakai bahasa web (HTTP) sehingga filternya http.request <br>
**Bukti Log**: <img width="1057" height="582" alt="image" src="https://github.com/user-attachments/assets/8dea2410-2ded-4c2c-aca7-7d9c1c96242f" /> <br>
**Materi**: ffuf (fuzz faster u fool) adalah alat atau teknik otomatis untuk menemukan celah keamanan, kesalahan logika, atau direktori tersembunyi yang bisa ngetok pintu dalam satu detik. attacker meninggalkan jejak yang namanya user-agent dalam file rekaman jaringan(.pcap) setiap kali ngetok pintu akan ninggalin catatan kecil yang tertulis jelas nama alatnya. <br>
**Jawaban**: `ffuf@2.1.0`

---

### Task 2: Subdomain
**Pertanyaan**: Subdomain mana yang ditemukan penyerang?

**Analisis**: Kita perlu analisis nama host dan pada gambar kita menemukan alat yang sebelumnya ditemukan <br>
**Bukti Log**: <img width="1366" height="768" alt="Screenshot 2026-03-06 144331" src="https://github.com/user-attachments/assets/bf55ab6a-feef-4b83-abba-e37e90a7d829" /><br>
**Materi**:  developer sering menggunakan subdomain untuk menaruh aset penting karena menganggap orang luar tidak akan tau namanya sehingga di kasus ini kemungkinan subdomain diserang.<br>
**Jawaban**: `cloud.vantage.tech`

---

### Task 3: Percobaan attacker login
**Pertanyaan**: berapa banyak percobaan login yang dilakukan attacker sebelum akhirnya berhasil masuk ke dashboard?

**Analisis**: lihat kolom dengan info POST <br>
**Bukti Log**: <img width="1289" height="625" alt="image" src="https://github.com/user-attachments/assets/fad9126f-4f3f-4eea-bb1c-50a9422eb7a0" /><br>
**Materi**: `http.request.method == "POST" `gunakan filter berikut untuk percobaan login. ip 10.116.0.3 adalah ip internal atau sebuah load balancer/proxy seringkali perusahaan besar tidak membiarkan server web mereka berhadapan langsung dengan internet, mereka memasang 'penjaga gerbang' sehingga ketika penyerang ngirim data diterima dulu oleh penjaga gerbang makanya jumlahnya sama<br>
**Jawaban**: 3

---

### Task 4: waktu download
**Pertanyaan**: Kapan penyerang download openstack API remote access config file?(UTC)

**Analisis**: cari yang infonya GET lalu identifikasi yang mengarah ke ip penyerang.<br>
**Bukti Log**: <img width="1291" height="654" alt="image" src="https://github.com/user-attachments/assets/49df66a4-d5c7-479e-b6e1-7a0cf7d7af3b" /><br>
**Materi**: file konfigurasi akses jarak jauh untuk openstack biasanya bernama openrc atau clouds.yaml. menggunakan filter GET karena mendownload. sehingga menggunakan filter http.request.uri contains "openrc" <br>
**Jawaban**: 2025-07-01 09:40:29

---

### Task 5: Waktu interaksi dengan API
**Pertanyaan**: Kapan waktu pertama kali penyerang berinteraksi dengan API nya?

**Analisis**: Setelah penyerang mencuri file konfigurasi maka penyerang akan mencoba berinteraksi dengan controller sehingga perlu difilter ip source penyerang dan HTTP<br>
**Bukti Log**: <img width="1286" height="619" alt="image" src="https://github.com/user-attachments/assets/0373afa6-74e4-4842-bc3c-7cc37430e107" /><br>
**Materi**: file controller mencatat komunikasi langsung antara penyerang dengan sistem pusat (API) dengan file konfigurasi yang mereka curi, file web-server mencatat interaksi di browser. Openstack API berkomunikasi dengan website melalui protokol HTTP. kenapa tidak memakai protokol lain karena teknologi cloud modern hampir semuanya berbasis REST API yang dirancang untuk berjalan di atas HTTP karena HTTP lebih mudah melewati firewall karena dianggap sebagai lalu lintas web biasa sehingga penyerang lebih mudah menyusup tanpa dicurigai.<br>
**Jawaban**: 2025-07-01 09:41:44

---

### Task 6: project ID openstack
**Pertanyaan**: Berapa project ID yang diakses penyerang?

**Analisis**: Berdasarkan dokumentasi MITRE, pembuatan akun lokal masuk dalam teknik *Create Account*.<br>
**Bukti Log**: <img width="1299" height="630" alt="image" src="https://github.com/user-attachments/assets/1355a0e4-03bb-41f2-a4de-d71401df46b0" /><br>
**Materi**: project id dalam openstack adalah kode identifikasi unik berbentuk alfanumerik yang digunakan untuk membedakan satu kelompok sumber daya dengan kelompok lainnya. project id penting karena cloud bekerja dalam sebuah 'kamar' , menjadi identitas permanen, tiap kali penyerang mengirim perintak ke controller melalui API maka perlu menyertakan project id agar server tau perintah di jalankan dimana. setelah penyerang mengirimkan kredensial (POST) maka server ngirim balasan berisi data JSON di dalam JSON tersebut ada token -> project -> id<br>
* **Jawaban**: 9fb84977ff7c4a0baf0d5dbb57e235c7

---

### Task 7: Layanan openstack
**Pertanyaan**: Layanan OpenStack mana yang menyediakan autentikasi dan otorisasi untuk API OpenStack?

**Materi**: layanan openstack yang memberikan fitur autentikasi dan otorisasi adalah keystone. key stone berfungsi untuk menyimpan daftar semua pengguna, grup, dan project id. ketika penyerang mengrimkan kredensial, keystone memeriksa apakah datanya benar. jika benar keystone akan memberikan token.<br>
**Jawaban**: keystone
---

### Task 8: Endpoint URL
**Pertanyaan**: apa Endpoint URL dari layanan Swift?

**Analisis**: filter dengan `http contains "project" && http contains "id"`<br>
**Bukti Log**: <img width="1361" height="664" alt="image" src="https://github.com/user-attachments/assets/4d82f1fe-9a54-4bfb-8193-f60b6b5317d4" /><br>
**Materi**: Endpoint URL Swift adalah alamat web khusus (API) yang digunakan untuk berinteraksi langsung dengan layanan Object Storage (penyimpanan data) seperti mengunggah, melihat, mencuri file rahasia dari cloud di sistem cloud OpenStack. penyerang tidak lagi menggunakan dashboard web tapi bisa langsung ngirim perintah langsung ke alamat URL ini. <br>
* **Jawaban**: [9fb84977ff7c4a0baf0d5dbb57e235c7](http://134.209.71.220:8080/v1/AUTH_9fb84977ff7c4a0baf0d5dbb57e235c7)

---

### Task 9 : Containers
**Pertanyaan**: Berapa containers yang diambil attacker?

**Analisis**: filter dengan `tcp.port == 8080` klik kanan pada salah satu paket follow -> HTTP stream <br>
**Bukti Log**: user-data <br> <img width="910" height="619" alt="Screenshot 2026-03-07 123051" src="https://github.com/user-attachments/assets/1a06bd4a-005a-481c-af63-6e6cc56c8f61" /> <br> employee-data <br> <img width="908" height="626" alt="image" src="https://github.com/user-attachments/assets/32d5ecce-f159-4362-93af-4b809e3abcc7" /> <br> json <img width="984" height="624" alt="Screenshot 2026-03-07 125854" src="https://github.com/user-attachments/assets/4286cf32-fcc3-41cb-bf19-cddf1761fece" /> <br>
**Materi**: container adalah folder tingkat pertama yang digunakan untuk mengelompokkan file-file digital. containter biasanya hanya satu tingkat saja tidak seperti folder di komputer yang bisa subfolder berlapis-lapis. <br>
* **Jawaban**: 3

---

### Task 10 : Download user-data
**Pertanyaan**: Kapan attacker download user-data?

**Analisis**: filter dengan `http.request.method == "GET" && tcp.port == 8080` klik kanan pada salah satu paket follow -> HTTP stream <br>
**Bukti Log**: <img width="1363" height="623" alt="image" src="https://github.com/user-attachments/assets/977d4df8-a616-4d5f-9192-76175b508382" /> <br>
**Materi**: bedanya ada yang `user-data?format=json` dan `user-data/user-details.csv` adalah .csv yang di download attacker sedangkan format=json adalah saat attacker nanya ke server di dalam folder user-data ada file apa aja dan dijawab server daftar nama filenya hanya teks kode JSON <br>
* **Jawaban**: 3

---

### Task 11 & 12 : Username & Password
**Pertanyaan**: Apa username yang attacker buat baru dan passwordnya?

**Analisis**: filter dengan `http.request.method == "POST" && http contains "users"` buka di packet detailsnya <br>
**Bukti Log**: <img width="1355" height="530" alt="Screenshot 2026-03-07 132253" src="https://github.com/user-attachments/assets/1350824d-3d55-4d10-bc51-eed3511c8b1a" /> <br>
* **Jawaban**: jellibean & P@$$word

---

### Task 10 : MITRE tactic id
**Pertanyaan**: apa MITRE tactic id yang digunakan attacker pada task 11 & 12 ?

**Analisis**: https://attack.mitre.org/tactics/enterprise/ <br>
**Materi**: attacker membuat akun baru dan passwordnya berarti dia melakukan persistence , pada kategori persistence pada link itu hal utama yang dilakukannya adalah create acoount (T1136) karena membuat akun pada cloud sehingga (.003) <br>
* **Jawaban**: T1136.003

---

## 💡 Kesimpulan Investigasi
Berdasarkan analisis forensik terhadap log jaringan yang tersedia, berikut adalah ringkasan kronologi serangan pada infrastruktur Vantage:

Reconnaissance & Initial Access: Penyerang memulai serangan dengan melakukan fuzzing menggunakan alat ffuf@2.1.0 untuk menemukan direktori tersembunyi. Mereka berhasil menemukan subdomain cloud.vantage.tech dan melakukan serangan brute force pada formulir login admin, yang akhirnya berhasil masuk pada percobaan ke-4 (setelah 3 kali gagal).

Credential Theft: Setelah mendapatkan akses ke dashboard admin, penyerang segera mengunduh file konfigurasi openrc yang berisi kredensial akses API jarak jauh untuk lingkungan OpenStack perusahaan.

Discovery & API Interaction: Menggunakan kredensial tersebut, penyerang mulai berinteraksi langsung dengan Keystone API (Identity Service) untuk mendapatkan Project ID (9fb84977ff7c4a0baf0d5dbb57e235c7). Penyerang kemudian memetakan layanan Swift dan menemukan 3 kontainer penyimpanan: images, user-data, dan employee-data.

Data Exfiltration: Penyerang melakukan pencurian data (exfiltration) dengan mengunduh file sensitif user-details.csv dan employee-details.csv. Data yang bocor mencakup informasi pribadi (PII) pelanggan dan karyawan, termasuk nama, email, departemen, dan nomor telepon.

Persistence: Untuk memastikan mereka tetap memiliki akses meskipun password admin diubah, penyerang membuat akun pengguna baru bernama jellibean dengan kata sandi P@$$word dan memberikan hak akses administratif. Aktivitas ini dikategorikan dalam teknik MITRE ATT&CK T1136.003 (Create Account: Cloud Account).
