## 📝 Informasi Challenge
* **Nama Challenge**: MangoBleed
* **Kesulitan**: very easy
* **Kategori**: Digital Forensics & Incident Response (DFIR)
* **Author**: cyberjunkie

### 📖 Sherlock Scenario
Anda dihubungi pagi ini untuk menangani insiden prioritas tinggi terkait dugaan peretasan server. Host tersebut, mongodbsync, merupakan server cadangan MongoDB. Menurut administrator, pemeliharaan dilakukan sebulan sekali, dan mereka baru saja menyadari adanya kerentanan bernama MongoBleed. Sebagai langkah pencegahan, administrator telah memberikan akses tingkat root untuk memfasilitasi investigasi Anda.

Anda telah mengumpulkan akuisisi triage dari server menggunakan UAC. Lakukan analisis triage cepat terhadap artefak yang dikumpulkan untuk menentukan apakah sistem telah terkompromi, identifikasi aktivitas penyerang (initial access, persistence, privilege escalation, lateral movement, atau data exfiltration), serta buat ringkasan temuan berupa penilaian awal insiden dan rekomendasi langkah selanjutnya.

---

## 🔍 Analisis Step-by-Step

### Task 1: IP Address Penyerang
**Pertanyaan**: Berapa alamat IP yang digunakan penyerang untuk melakukan serangan brute force?

**Analisis**: Dalam `auth.log`, terlihat ribuan percobaan login gagal dalam waktu yang sangat singkat (beberapa percobaan per detik) dari IP yang sama.<br>
**Bukti Log**: `sshd[2325]: Invalid user admin from 65.2.161.68 port 46380`.<br>
**Materi**: Serangan *Brute Force* SSH ditandai dengan log `Invalid user` atau `Failed password` yang berulang dari satu sumber IP.<br>
**Jawaban**: `65.2.161.68`

---

### Task 2: Versi mongodb
**Pertanyaan**: Berapa versi mongodb yang menjadi korban?

**Analisis**: analisis mongodb /var/log/mongodb/
**Bukti Log**: <img width="1364" height="688" alt="image" src="https://github.com/user-attachments/assets/ac0a72fc-3f2f-49d1-80fd-3bcf82e2ecd6" />
**Jawaban**: 8.0.16

---

### Task 3: remote ip address attacker
**Pertanyaan**: Berapa alamat IP yang digunakan secara remote oleh attacker untuk melakukan menyerangan ?

**Analisis**: analisis mongodb /var/log/mongodb/
**Bukti Log**: <img width="1364" height="685" alt="image" src="https://github.com/user-attachments/assets/355a0758-5b61-4bd0-9cbe-8ce752d978aa" />
**Jawaban**: 8.0.16

---

### Task 5: Mongod logs
**Pertanyaan**: Berapa total log yang diinisiasi oleh attacker?

**Analisis**: analisis mongodb /var/log/mongodb/
**Bukti Log**: <img width="1115" height="628" alt="image" src="https://github.com/user-attachments/assets/0894ac63-7af6-4683-a26d-abe9eb205bf2" />
** Materi**: pada windows masuk ke folder tempat file dimana /c singkatan dari count
**Jawaban**: 75260

---

### Task 6: Mongod logs
**Pertanyaan**: Berapa total log yang diinisiasi oleh attacker?

**Analisis**: analisis mongodb /var/log/mongodb/
**Bukti Log**: <img width="1115" height="628" alt="image" src="https://github.com/user-attachments/assets/0894ac63-7af6-4683-a26d-abe9eb205bf2" />
** Materi**: pada windows masuk ke folder tempat file dimana /c singkatan dari count
**Jawaban**: 75260
