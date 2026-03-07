## 📝 Informasi Challenge
* **Nama Challenge**: Telly
* **Kesulitan**: very easy
* **Kategori**: SOC
* **Author**: Cyberjunkie

### 📖 Sherlock Scenario
You are a Junior DFIR Analyst at an MSSP that provides continuous monitoring and DFIR services to SMBs. Your supervisor has tasked you with analyzing network telemetry from a compromised backup server. A DLP solution flagged a possible data exfiltration attempt from this server. According to the IT team, this server wasn't very busy and was sometimes used to store backups.

kamu adalah seorang DFIR analyst junior di perusahaan MSSP yang menyediakan layanan monitoring untuk UKM. supervisor kamu meminta kamu untuk menganalisis jaringan di server backup. DLP menunjukkan flag bahwa adanya kemungkinan percobaan data dari server. menurut tim IT nya, server tidak sibuk dan kadang digunakan untuk nyimpen backup.

MSSP adalah perusahaan pihak ketiga yang disewa perusahaan lain untuk mengelola dan memantau keamana mereka secara penuh. SMB (small and medium-sized business) istilah lain untuk UKM (usaha kecil dan menengah). DLP (data loss prevention) adalah software untuk solusi dalam mendeteksi dan mencegah kebocoran data keluar dari jaringan perusahaan.

---

## 🔍 Analisis Step-by-Step

### Task 1: Tool Penyerang
**Pertanyaan**: What CVE is associated with the vulnerability exploited in the Telnet protocol?

* **Analisis**: filter dengan telnet, cek di paket awal follow TCP stream jika ada -f root maka akan mencurigakan.
* **Bukti Log**: <img width="1356" height="659" alt="image" src="https://github.com/user-attachments/assets/6c5b8d34-fd34-4f3f-b66f-d22e384ca68a" />
* **Materi**: CVE (Common Vulnerabilities and Exposures) adalah daftar kerentanan keamanan siber yang dipublikasikan secara publik agar para ahli keamanan bisa mengidentifikasi dan memperbaikinya. Telnet (Teletype Network) adalah protokol komunikasi jaringan yang digunakan untuk mengakses dan mengelola komputer atau perangkat lain secara jarak jauh (remote) melalui jaringan lokal maupun internet. Follow TCP Stream adalah fitur yang digunakan untuk menyatukan kembali potongan-potongan paket menjadi satu naskah percakapan yang utuh dan berurutan dari awal sampai akhir. Normalnya, protokol Telnet akan menanyakan identitas pengguna melalui variabel lingkungan bernama USER misal user=admin. nilai -f root adalah sebuah parameter sistem, -f berarti "force", server telnet tidak memeriksa input ini dengan benar (ini kelemahannya) sehingga server salah mengartikan membuat attacker bisa masuk ke sistem secara paksa sebagai user root tanpa perlu cek password.
* **Jawaban**: CVE-2026-24061

---

### Task 8: Database credit card
**Pertanyaan**: temukan nomor credit card atas nama Quinn Harris.

* **Analisis**:karena file adalah database maka menggunakan export object untuk mendapatkannya.
* **Bukti Log**:<img width="1360" height="627" alt="image" src="https://github.com/user-attachments/assets/cc8dcbaf-35a5-4ae5-9354-047a7bc6bf2c" />
* **Materi**: file database adalah file biner. follow HTTP stream dirancang untuk menampilkan teks percakatan seperti kode HTML, JSON atau teks biasa ketika file yang dikirm adalah file biner, gambar, atau dokumen wireshark tidak bisa menampilkannya sehingga perlu pake fitur export objects.
* **Jawaban**: 5312269047781209
