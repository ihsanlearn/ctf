# Log Analysis

Seringkali kita dikasih file `server.log` ukurannya 500MB, isinya teks berantakan, campur aduk, dan kita disuruh cari satu baris "serangan" atau flag di antaranya. Kuncinya di sini bukan membaca satu-satu, tapi Filtering dan Anomaly Detection.

***

## 📜 Panduan Lengkap CTF Log Analysis

### 1. Fase Triage (Bedah Cepat) 🚑

_Jangan langsung buka file 1GB pakai Notepad atau VSCode! Laptopmu bisa hang._

*   Cek Ukuran & Jenis:

    Bash

    ```
    ls -lh access.log      # Cek ukuran (Human readable)
    file access.log        # Cek tipe file (ASCII? UTF-8? Gzip?)
    ```
*   Intip Isinya (Head & Tail):

    Lihat struktur awal dan akhir log untuk memahami formatnya (timestamp, IP, method).

    Bash

    ```
    head -n 20 access.log  # Lihat 20 baris pertama
    tail -n 20 access.log  # Lihat 20 baris terakhir
    ```
*   Baca Tanpa Hang (less):

    Gunakan less untuk membaca file besar. Tekan q untuk keluar.

    Bash

    ```
    less access.log
    # Di dalam less: ketik /keyword untuk mencari kata.
    ```

***

### 2. The Holy Trinity: Sort, Uniq, Count 📊

Ini adalah Teknik Paling Ampuh untuk log yang tidak jelas strukturnya. Kita mencari anomali (sesuatu yang terlalu sering muncul atau terlalu jarang muncul).

*   Mencari "Top Hits":

    Misal, siapa IP yang paling sering ngebom server? Atau endpoint apa yang paling sering diakses?

    Bash

    ```
    # Contoh: Ambil kolom IP (misal kolom ke-1), urutkan, hitung kemunculannya
    awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 10
    ```

    * `awk '{print $1}'`: Ambil kolom pertama (biasanya IP).
    * `sort`: Urutkan biar yang sama jadi sebelahan.
    * `uniq -c`: Hapus duplikat dan hitung jumlahnya.
    * `sort -nr`: Urutkan berdasarkan angka (jumlah) dari terbesar ke terkecil.
*   Mencari "The Rare One":

    Kadang serangan itu cuma terjadi 1 kali (misal upload shell).

    Bash

    ```
    # Lihat yang munculnya paling sedikit (bottom 10)
    awk '{print $1}' access.log | sort | uniq -c | sort -n | head -n 10
    ```

***

### 3. Cleaning & Parsing (Membersihkan Sampah) 🧹

Kalau log-nya berantakan (tidak terstruktur rapi), kita harus memilahnya.

*   Cut (Potong per Karakter/Delimiter):

    Jika pemisahnya bukan spasi, tapi koma (CSV) atau pipa |.

    Bash

    ```
    cut -d ',' -f 3 log_kacau.txt  # Ambil kolom ke-3, pemisahnya koma
    ```
*   Awk (The Surgeon):

    Tool paling sakti buat log.

    Bash

    ```
    # Print kolom 1, 4, dan 7 saja
    awk '{print $1, $4, $7}' access.log

    # Cari yang status codenya 200 (OK)
    awk '$9 == 200 {print $0}' access.log

    # Panjang baris (Mencari payload buffer overflow yang panjang banget)
    awk 'length($0) > 100' access.log
    ```
*   Sed (Search & Replace):

    Berguna buat decode log yang di-encode URL (%20, %27).

    (Tips: Gunakan tool urldecode atau CyberChef lebih mudah, tapi sed bisa buat cleaning dasar).

***

### 4. Filtering Strategy (Membuang Noise) 🗑️

Log biasanya 90% sampah (noise). Kamu harus membuang sampah itu biar flag/serangannya kelihatan.

*   Grep -v (Invert Match):

    Ini lebih penting daripada grep biasa. Buang apa yang kamu tahu itu aman.

    Bash

    ```
    # Tampilkan log TAPI buang yang ada kata "Googlebot", buang file ".css", buang file ".png"
    cat access.log | grep -v "Googlebot" | grep -v ".css" | grep -v ".png" | grep -v ".js"
    ```

    _Sisanya biasanya adalah serangan atau akses file aneh._
*   Grep -E (Regex):

    Mencari pola spesifik.

    Bash

    ```
    # Cari format email
    grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}" log.txt

    # Cari pola flag standar (CTF{...})
    grep -oE "CTF\{.*?\}" log.txt
    ```

***

### 5. Tools Otomatis & Visualisasi 🛠️

Jika manual terlalu capek, gunakan tools ini:

#### A. lnav (Log File Navigator) - _Highly Recommended_

Ini tool favorit buat CTF. Dia mewarnai log secara otomatis, bisa menumpuk beberapa log file, dan punya fitur filter SQL-like.

* Install: `sudo apt install lnav`
* Cara pakai: `lnav access.log`
* Fitur: Tekan `i` untuk pindah ke mode histogram (lihat lonjakan traffic).

#### B. jq (Untuk JSON Logs)

Kalau log-nya format JSON (kurung kurawal `{}`), jangan baca manual!

*   Pretty Print (Biar rapi):

    Bash

    ```
    cat log.json | jq .
    ```
*   Filter Field Tertentu:

    Bash

    ```
    cat log.json | jq -r '.user_agent'
    ```

#### C. GoAccess (Web Log Analyzer)

Mengubah `access.log` web server menjadi tampilan dashboard HTML yang keren dan mudah dibaca.

* Command: `goaccess access.log -o report.html --log-format=COMBINED`
* Buka `report.html` di browser, kamu bisa lihat grafik serangan dengan jelas.

***

### 6. Skenario Serangan pada Log (Cheat Sheet) 🏴‍☠️

Apa yang harus kamu cari di dalam log?

#### A. Web Server Attacks (Apache/Nginx)

* SQL Injection: Cari kata kunci `UNION`, `SELECT`, `OR 1=1`, `%27` (tanda kutip).
* LFI (Local File Inclusion): Cari `../../../../etc/passwd` atau `boot.ini`.
* XSS (Cross Site Scripting): Cari `<script>`, `alert(`, `onerror=`.
* Command Injection: Cari `whoami`, `cat`, `ls`, `wget`, `curl` di dalam URL.
* Web Shell: Cari akses ke file ekstensi aneh `.php`, `.php5`, `.phtml` dengan metode `POST`.

#### B. Brute Force Attacks (Auth Logs)

* Cari pesan `Failed password` yang berulang-ulang dari IP yang sama dalam waktu singkat.
* Cari pesan `Invalid user`.

#### C. User Agent Aneh

* User Agent sering dipakai hacker untuk menyelipkan serangan (Shellshock) atau tool scanning.
* Cari: `sqlmap`, `nikto`, `gobuster`, `curl`, `python-requests`.

***

### 🌟 Tips

1.  Base64 Decoding:

    Seringkali payload serangan di-encode base64. Kalau nemu string acak panjang (misal: ZXhpdCgpOw==), coba decode.

    Bash

    ```
    echo "ZXhpdCgpOw==" | base64 -d
    ```
2.  User Agent is a liar:

    Jangan percaya 100% pada log. Hacker bisa memalsukan User Agent atau IP (lewat header X-Forwarded-For).
3.  Timeline is Key:

    Jika kamu menemukan satu request berbahaya (misal shell upload berhasil), catat jam/waktu-nya. Lalu filter log di sekitar waktu tersebut untuk melihat apa yang dilakukan hacker setelah berhasil masuk.
