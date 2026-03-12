## 📝 Informasi Challenge
* **Nama Challenge**: Noxious
* **Kesulitan**: very easy
* **Kategori**: SOC
* **Author**: cyberjunkie

### 📖 Sherlock Scenario
The IDS device alerted us to a possible rogue device in the internal Active Directory network. The Intrusion Detection System also indicated signs of LLMNR traffic, which is unusual. It is suspected that an LLMNR poisoning attack occurred. The LLMNR traffic was directed towards Forela-WKstn002, which has the IP address 172.17.79.136. A limited packet capture from the surrounding time is provided to you, our Network Forensics expert. Since this occurred in the Active Directory VLAN, it is suggested that we perform network threat hunting with the Active Directory attack vector in mind, specifically focusing on LLMNR poisoning.

IDS menampilkan peringatan adanya perangkat nakal pada internal active directory jaringan. IDS mengindikasi adanya tanda LLMNR trafik yang tidak biasa.LLMNR berasal dari Forela-WKstn002, yang alamat ip nya 172.17.79.136. kejadian pada active directory VLAN.

---

## 🔍 Analisis Step-by-Step

### Task 1: IP Address Penyerang
**Pertanyaan**: alamat ip berbahaya yang digunakan?

* **Analisis**: menggunakan filter `llmnr && ip.dst==172.17.79.136`
* **Bukti Log**: <img width="1363" height="718" alt="image" src="https://github.com/user-attachments/assets/37daca3d-985b-49c7-85f2-06803dee8fee" />
* **Materi**: korban bertanya sehingga ip source adalah ip korban, bertanya ke semua orang sehingga ip dest adalah alamat multicast. karna ingin mencari attacker yang menjawab maka ip source adlah ip attacker dan ip dest adala ip korban.
* **Jawaban**: 172.17.79.135

---

### Task 2: Hostname rogue device
**Pertanyaan**: apa nama hostname perangkat yang berbahaya?

* **Analisis**: menggunakan filter `ip.addr == 172.17.79.135 && dhcp`
* **Bukti Log**: <img width="1360" height="719" alt="image" src="https://github.com/user-attachments/assets/4cca49d0-70bb-4c97-a245-c88c4e5e3d0a" />
* **Materi**: menggunakan DHCP karena perangkat jahat meminta alamat ip ke dalam jaringan dimana alamat ip itu sebelumnya dipakai oleh perangkat yang sah sehingga pada pcap. akan muncul keduanya sehingga pilih yang terakhir kemungkinan adalah jahat.
* **Jawaban**: kali

---

### Task 3: Hostname rogue device
**Pertanyaan**: apa nama hostname perangkat yang berbahaya?

* **Analisis**: menggunakan filter `ntlmssp` cari NTLMSSP_AUTH di packet details cari user name
* **Bukti Log**: <img width="1365" height="717" alt="image" src="https://github.com/user-attachments/assets/4d8c28ee-d31e-47ef-a56d-89faea2069e4" />
* **Materi**: Untuk mengonfirmasi apakah penyerang berhasil mencuri hash (identitas terenkripsi) dan siapa korbannya, kita harus mencari jejak protokol autentikasi Windows. Setelah penyerang "meracuni" jaringan dengan LLMNR/NBNS, komputer korban akan tertipu dan mencoba melakukan login ke mesin penyerang. Proses ini menggunakan protokol NTLMSSP. NTLMSSP untuk memunculkan semua proses pertukaran kunci/password
* **Jawaban**: john.deacon

---

### Task 4: Timestamp
**Pertanyaan**: pada trafik NTLM kredensial korban beberapa kali ke mesin penyerang, kapan pertama kali hash ditangkap ?

* **Analisis**: menggunakan filter `ntlmssp` cari NTLMSSP_AUTH
* **Bukti Log**: <img width="1365" height="720" alt="image" src="https://github.com/user-attachments/assets/f353d6f7-d049-4fae-a519-37784525acd4" />
* **Materi**: NTLM (NT LAN Manager) adalah protokol keamanan microsoft yang digunakan untuk membuktikan identitas (autentikasi) seseorang pengguna di dalam jaringan.
* cara kerja:
negotiate : komputer kita bilang ke server "saya mau masuk dan mendukung protokol NTLM"
challenge : server tidak langsung percaya, server kirim angka acak ke komputer
authenticate : komputer ambil angka acaknya campur dengan "hash" password kita, lalu dikirim lagi ke server
* **Jawaban**: 2024-06-24 11:18:30

---

### Task 6: NTLM challenge
**Pertanyaan**: berapa angka acak pada challenge?

* **Analisis**: menggunakan filter `ntlmssp` cari NTLMSSP_AUTH
* **Bukti Log**: <img width="1364" height="722" alt="image" src="https://github.com/user-attachments/assets/08e21e2f-5071-4081-bd95-68a3a90ed071" />
* **Materi**: karena challenge artinya nyari angka acaknya maka cari info paket yang NTLM challenge lalu cek di packet details angka tersebut.
* **Jawaban**: 601019d191f054f1

---

### Task 7: NTProofStr
**Pertanyaan**: cari value NTProofStr

* **Analisis**: menggunakan filter `ntlmssp` cari NTLMSSP_AUTH
* **Bukti Log**: <img width="1365" height="720" alt="image" src="https://github.com/user-attachments/assets/51071000-53e4-4415-8005-9d5083e32a79" />
* **Materi**: NTProofStr adalah nilai hasil kombinasi angka acak dengan hash password kita sehingga berada di tahap ketiga yaitu authenticate pada wireshark maka cari di info yang NTLM AUTH
* **Jawaban**: c0cc803a6d9fb5a9082253a04dbd4cd4

---

### Task 8: recover password
**Pertanyaan**: cari passwordnya

* **Analisis**: untuk memecahkan passwordnya bisa menggunakan hashcat atau john the ripper. perlu download dulu hashcat/john dan file rockyou.txt nya jika di windows untuk kali linux sudah ada
* **Bukti Log**: <img width="1358" height="659" alt="image" src="https://github.com/user-attachments/assets/438ce21e-856c-4891-8e26-e5f9acdc8507" />
* **Materi**: bikin file dulu misal namafile.txt isi dengan format seperti ini "User::Domain:ServerChallenge:NTProofStr:NTLMv2Response(without first 16 bytes)" di terminal ketik -> john --format=netntlmv2 --wordlist=rockyou.txt hashfile.txt
* **Jawaban**: NotMyPassword0K?

---

### Task 9: recover password
**Pertanyaan**: cari passwordnya

* **Analisis**: menggunakan filter `smb2`
* **Bukti Log**: <img width="1365" height="717" alt="image" src="https://github.com/user-attachments/assets/574df4a7-3f02-4a88-bc71-a4c39766cc9a" />
* **Materi**: SMB (server message block) protokol standar untuk berbagi file, printer, dan komunasi antar komputer dalam satu jaringan sehingga untuk memfilter kita menggunakan smb2. tree connect request yang dari karna korban melakukan pencarian tapi hindari path yang berakhiran IPC$, ADMIN$, C$ karna ini dilakukan otomatis oleh windows bukan manusia
* **Jawaban**: \\DC01\DC-Confidential

---
Secara keseluruhan, peristiwa ini diawali oleh kesalahan manusia yang sangat sederhana, yaitu kesalahan pengetikan nama server tujuan oleh korban, John Deacon. Saat ia bermaksud mengakses server DC01 namun mengetik DCC01, komputer korban mengirimkan sinyal ke seluruh jaringan untuk mencari alamat tersebut. Di sinilah penyerang yang telah siaga menggunakan alat bernama Responder melakukan teknik LLMNR Poisoning, yaitu dengan berpura-pura menjadi server palsu (DCC01) dan merayu komputer korban untuk terhubung kepadanya.  Karena komputer korban percaya, dimulailah proses autentikasi di mana komputer John mengirimkan Hash NTLMv2—sebuah bentuk sandi yang terenkripsi—melalui protokol SMB.

Setelah penyerang berhasil menangkap Hash tersebut dari lalu lintas jaringan, langkah berikutnya adalah melakukan serangan Offline Password Cracking menggunakan alat seperti Hashcat atau John the Ripper. Dengan mencocokkan Hash tersebut terhadap jutaan kata sandi umum dalam daftar rockyou.txt, penyerang akhirnya berhasil menemukan password asli John Deacon karena tingkat kompleksitasnya yang rendah.  Di saat yang sama, John yang tidak menyadari bahwa ia telah terhubung ke mesin penyerang tetap melanjutkan niatnya untuk mengakses data. Hal ini terlihat dari paket SMB2 Tree Connect Request, di mana John secara spesifik meminta akses ke folder non-default bernama resources, yang menandakan bahwa penyerang kini tidak hanya memegang kredensial John, tetapi juga mengetahui folder sensitif mana yang sedang dikerjakan oleh korban.
