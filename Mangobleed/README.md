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
