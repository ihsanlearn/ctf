# Memory

Ini adalah teknik menganalisis _dump_ memori (RAM) dari komputer yang sedang menyala. Aset yang kamu dapat biasanya berekstensi `.mem`, `.dmp`, `.raw`, atau `.vmem`.

Tujuannya? Melihat apa yang sedang terjadi saat itu juga: program yang jalan, password yang diketik, koneksi jaringan aktif, atau isi clipboard (Copy-Paste).

***

## 🧠 Panduan Lengkap CTF Memory Forensics

### 1. Langkah Awal: "Quick Win" (Tanpa Volatility) ⚡

Sama seperti Network dan Stego, kadang flag itu tersimpan sebagai teks biasa di memori. Jangan langsung pakai tool berat, coba ini dulu:

*   Strings is King:

    Cari teks yang bisa dibaca manusia di dalam tumpukan binary memori.

    Bash

    ```
    strings file.mem | grep -i "flag"
    strings file.mem | grep -i "password"
    strings file.mem | grep -i "CTF{"
    ```

    _Tips: Kalau file-nya besar (GB), pakai `strings -n 10 file.mem` (minimal 10 karakter)._

***

### 2. The King: Volatility Framework 🌋

Di CTF, tool standar industri adalah Volatility.

Catatan: Panduan ini menggunakan syntax Volatility 2 (Python 2) karena masih paling sering muncul di CTF dan plugin-nya sangat lengkap.

#### A. Identifikasi Profile (Langkah Wajib!)

Kamu tidak bisa menganalisis kalau tidak tahu OS apa yang dipakai korban (Win7? Win10? Linux?).

*   Command:

    Bash

    ```
    volatility -f file.mem imageinfo
    ```
* Hasil: Lihat bagian `Suggested Profile(s)`.
  * _Contoh:_ `Win7SP1x64`.
  * Ingat profile ini, karena akan dipakai di SEMUA command selanjutnya (`--profile=Win7SP1x64`).

***

### 3. Analisis Proses & Command Line 💻

Mencari tahu program apa yang sedang dijalankan user.

*   Lihat Daftar Proses:

    Melihat semua aplikasi yang jalan (Chrome, Notepad, Malware, dll).

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 pslist
    ```
*   Lihat Command Line (SANGAT PENTING!):

    Melihat perintah apa yang diketikkan saat menjalankan program. Seringkali user mengetik flag atau password di sini.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 cmdline
    ```
*   Lihat History Console (CMD/PowerShell):

    Melihat apa yang diketik user di layar hitam (Command Prompt).

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 consoles
    ```
*   Environment Variables:

    Kadang flag disembunyikan di variabel lingkungan sistem.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 envars
    ```

***

### 4. Mencari File & Data Rahasia 📂

*   Scan Semua File di RAM:

    Mencari file yang ada di memori (seperti flag.txt, secret.pdf, pass.png).

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 filescan | grep "flag"
    volatility -f file.mem --profile=Win7SP1x64 filescan | grep "Desktop"
    ```

    _(Catat Offset memori file tersebut jika ketemu, misal: `0x000000007e60b190`)_
*   Dump (Ekstrak) File Tersebut:

    Setelah dapat offset-nya, ambil filenya keluar.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 dumpfiles -Q 0x[OFFSET] -D output_folder/
    ```
*   Notepad (Khusus Windows):

    Melihat teks apa yang sedang diketik di aplikasi Notepad tapi belum disave.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 notepad
    ```

***

### 5. User Activity (Apa yang User Lakukan?) 🕵️‍♂️

*   Clipboard (Copy-Paste):

    Apakah user baru saja meng-copy password atau flag? Cek clipboard-nya.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 clipboard
    ```
*   Screenshot:

    Mengambil gambar layar desktop user saat memori diambil.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 screenshot -D output_folder/
    ```
*   Paint (MS Paint):

    Kalau user menggambar sesuatu di Paint, kita bisa melihat gambarnya.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 mspaint -D output_folder/
    ```

***

### 6. Password & Credentials Hunting 🔐

*   Hashdump (Windows Passwords):

    Mengambil hash password user Windows (NTLM Hash). Hash ini bisa di-crack online atau pakai John The Ripper.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 hashdump
    ```
*   Mimikatz (The Golden Plugin):

    Plugin sakti untuk mengambil password PLAINTEXT (teks asli) dari memori tanpa perlu crack hash.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 mimikatz
    ```
*   LSA Secrets:

    Mengambil password default atau autologin yang tersimpan di registry.

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 lsadump
    ```

***

### 7. Network Connections 🌐

Mirip `netstat`. Melihat koneksi ke IP mana saja komputer ini terhubung. Berguna untuk melihat ke mana malware berkomunikasi.

Bash

```
volatility -f file.mem --profile=Win7SP1x64 netscan
```

***

### 8. Advanced / Malware Hunting 🦠

Jika soalnya berbau malware (bukan sekadar cari flag teks).

*   Malfind:

    Mencari injected code atau DLL tersembunyi yang mencurigakan (biasanya ditandai dengan header MZ tapi di lokasi yang tidak wajar).

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 malfind
    ```
*   Procdump:

    Mengambil file executable (EXE) dari proses yang berjalan untuk dianalisis lebih lanjut (Reverse Engineering).

    Bash

    ```
    volatility -f file.mem --profile=Win7SP1x64 procdump -p [PID] -D output_folder/
    ```

***

### 🌟 Tips

1. Grep adalah Sahabat: Output Volatility itu panjang banget. Selalu gunakan `| grep "keyword"` untuk mencari hal spesifik (misal: `flag`, `admin`, `pass`, `.txt`, `.png`).
2. Volatility 3: Saat ini sudah ada Volatility 3 (berbasis Python 3). Syntax-nya beda (tidak perlu `--profile`, dia deteksi otomatis).
   * _Contoh:_ `python3 vol.py -f file.mem windows.pslist`
   * Tapi untuk belajar dan CTF klasik, Volatility 2 masih lebih sering dipakai karena plugin-nya lebih banyak.
3. Browser History: Ada plugin khusus (seperti `chromehistory` atau `firefoxhistory`) tapi seringkali harus install plugin eksternal. Cara paling gampang? `strings file.mem | grep "http"` atau dump proses browsernya.
