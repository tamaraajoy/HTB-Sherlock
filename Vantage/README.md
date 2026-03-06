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
---

## 💡 Kesimpulan Investigasi
Serangan dimulai dengan **Brute Force** terhadap berbagai layanan user. Setelah mendapatkan akses **root**, penyerang melakukan **Persistence** dengan membuat user **cyberjunkie** dan mengunduh toolkit audit/persistensi menggunakan **curl**.
