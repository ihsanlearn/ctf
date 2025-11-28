# Network

***

## 🦈 Panduan Lengkap CTF Network Forensics

### 1. Fase Reconnaissance (Analisis Mentah) 🔍

_Sebelum buka Wireshark, cek dulu isi kasarnya._

*   Strings (Lagi-lagi Strings!):

    Sama seperti stego, kadang flag ada di clear text.

    Bash

    ```
    strings capture.pcap | grep -i "flag"
    strings capture.pcap | grep -i "password"
    strings capture.pcap | grep -i "User-Agent"
    ```
*   Capinfos:

    Cek statistik file pcap (durasi capture, jumlah paket, dll) untuk memahami seberapa besar datanya.

    Bash

    ```
    capinfos capture.pcap
    ```

***

### 2. The King: Wireshark 🦈

Ini adalah senjata utama. Jangan cuma dibuka, tapi kuasai fitur-fitur ini:

#### A. Filtering (Menyaring Sampah)

Jangan pusing dengan ribuan paket. Gunakan filter bar di atas:

* Hanya HTTP: `http` (Cari web traffic).
* Cari Login (POST req): `http.request.method == "POST"` (Biasanya berisi username/password).
* Cari String Tertentu: `frame contains "flag"` atau `tcp contains "ctf"`.
* Filter IP: `ip.addr == 192.168.1.5`

#### B. Follow TCP/HTTP Stream

Jangan baca paket satu-satu!

* Caranya: Klik kanan pada satu paket yang menarik -> Follow -> TCP Stream (atau HTTP Stream).
* Hasilnya: Kamu akan melihat percakapan utuh (request & response) dalam format teks yang mudah dibaca. Password sering ketemu di sini!

#### C. Export Objects (Mengambil File)

Jika user di dalam pcap pernah mendownload gambar, PDF, atau ZIP, kamu bisa mengambilnya kembali.

* Caranya: Klik menu File -> Export Objects -> Pilih HTTP (atau SMB/FTP).
* Action: Save All. Lalu cek file hasil download-nya satu per satu.

***

### 3. The "Lazy" Tool: NetworkMiner ⛏️

_Tools favorit buat yang malas manual di Wireshark._

*   Kenapa pakai ini?

    NetworkMiner otomatis menyusun ulang file, gambar, dan kredensial yang ada di dalam pcap tanpa kamu perlu klik kanan satu-satu.
* Fitur Utama:
  * Tab Files: Langsung memunculkan semua gambar/dokumen yang lewat di jaringan.
  * Tab Credentials: Kadang langsung memunculkan username & password (FTP, HTTP Basic Auth, dll).
  * Tab DNS: Melihat domain apa saja yang diakses.
*   Note: NetworkMiner berjalan native di Windows, tapi bisa jalan di Kali Linux pakai `mono`.

    Bash

    ```
    sudo apt install networkminer
    networkminer
    ```

***

### 4. Analisis Protokol Spesifik 📡

#### A. HTTP & FTP (Clear Text Protocols)

* Ini sasaran empuk. Password dan isi file tidak dienkripsi.
* Cari request `POST` untuk login.
* Cari command `RETR` (di FTP) yang menandakan transfer file, lalu extract datanya.

#### B. DNS (Domain Name System)

* Kasus: _DNS Exfiltration_. Hacker mencuri data dengan menyisipkannya ke dalam nama subdomain.
  * _Contoh:_ `bagian1flag.attacker.com`, `bagian2flag.attacker.com`.
* Analisis: Filter `dns` di Wireshark. Lihat _Query Name_-nya. Gabungkan potongan-potongan subdomain tersebut (biasanya encoded hex atau base64).

#### C. USB Traffic (Keyboard/Mouse Sniffing)

* Kasus: Kamu dapat pcap isi traffic USB. Hacker menyadap keyboard korban.
* Analisis:
  * Cari data `USB HID` (Human Interface Device).
  * Gunakan `tshark` untuk mengambil _Leftover Capture Data_.
  * Gunakan script python (banyak di GitHub, keyword: "ctf usb keyboard parser") untuk menerjemahkan kode hex USB menjadi huruf yang kita baca.

***

### 5. Command Line Power: TShark ⚡

Versi terminal dari Wireshark. Berguna kalau file pcap-nya besar sekali (GB) dan bikin Wireshark _crash_, atau untuk scripting.

*   Contoh Command:

    Extract field tertentu (misal isi DNS query):

    Bash

    ```
    tshark -r capture.pcap -T fields -e dns.qry.name
    ```

***

### 6. Kasus Enkripsi (HTTPS/TLS) 🔒

_Kasus: Semua traffic isinya "Application Data" (Terbaca acak/kacau) karena HTTPS._

Kamu TIDAK BISA membaca ini kecuali kamu punya kuncinya.

1. Cari SSL Key Log File: Kadang di soal CTF, diberikan file tambahan bernama `ssl.log` atau `master.key`.
2. Decrypt di Wireshark:
   * Buka Wireshark -> Edit -> Preferences.
   * Buka Protocols -> TLS (atau SSL).
   * Masukkan file key tadi di kolom (Pre)-Master-Secret log filename.
   * _Boom!_ Traffic HTTPS akan berubah warna jadi hijau (decrypted) dan bisa dibaca sebagai HTTP biasa.

***

### 🌟 Tips

1.  Scapy (Python Library): Kalau tools di atas gak mempan, kamu harus coding pakai Python menggunakan library `scapy` untuk membedah paket secara kustom.

    Python

    ```
    from scapy.all import *
    packets = rdpcap('file.pcap')
    # Lakukan loop dan logic sendiri
    ```
2. Jangan Lupa `grep`: Di Linux, kamu bisa gabungin `tshark` sama `grep`.
   * `tshark -r file.pcap -Y "http.request" -T fields -e http.host | sort | uniq` (Melihat daftar semua website yang dikunjungi).
