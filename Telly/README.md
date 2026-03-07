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
**Pertanyaan**: Alat apa yang digunakan penyerang untuk melakukan fuzz pada server web? (Format- sertakan versi misalnya, nmap@7.80)

**Analisis**: karena attacker terus menerus nanya ke server "ada folder/admin nggak?" maka ini disebut request. kita cuma mau liat obrolan yang pakai bahasa web (HTTP) sehingga filternya http.request <br>
**Bukti Log**: <img width="1057" height="582" alt="image" src="https://github.com/user-attachments/assets/8dea2410-2ded-4c2c-aca7-7d9c1c96242f" /> <br>
**Materi**: ffuf (fuzz faster u fool) adalah alat atau teknik otomatis untuk menemukan celah keamanan, kesalahan logika, atau direktori tersembunyi yang bisa ngetok pintu dalam satu detik. attacker meninggalkan jejak yang namanya user-agent dalam file rekaman jaringan(.pcap) setiap kali ngetok pintu akan ninggalin catatan kecil yang tertulis jelas nama alatnya. <br>
**Jawaban**: `ffuf@2.1.0`
