# Forensics

***

## 🕵️‍♂️ DIGITAL FORENSICS: THE MASTER GUIDE

### 📑 BAB 1: Mindset & Dasar (Fundamentals)

Definisi: Seni membedah file, log, atau rekaman jaringan untuk menemukan jejak digital (Flag) yang disembunyikan, dihapus, atau disamarkan.

3 Hukum Utama:

1. Ekstensi File itu BOHONG: Jangan percaya `.txt`, `.jpg`, dll. Percaya hanya pada Magic Bytes.
2. Metadata is Key: Data tentang data seringkali lebih penting daripada isinya.
3. Visible vs Invisible: Apa yang terlihat di layar (viewer) beda dengan apa yang dilihat komputer (hex/binary).

***

### 🛠️ BAB 2: File Analysis (Static)

Analisis file tanpa menjalankannya. Ini langkah pertama saat menerima soal.

#### 1. Identifikasi File (Magic Bytes)

Header file (byte awal) adalah identitas asli file.

| **Format** | **Magic Bytes (Hex)**     | **ASCII** |
| ---------- | ------------------------- | --------- |
| JPEG       | `FF D8 FF`                | ÿØÿ       |
| PNG        | `89 50 4E 47 0D 0A 1A 0A` | .PNG....  |
| GIF        | `47 49 46 38`             | GIF8      |
| ZIP        | `50 4B 03 04`             | PK..      |
| PDF        | `25 50 44 46`             | %PDF      |
| WAV        | `52 49 46 46`             | RIFF      |

* Tools: `file`, `hexeditor`, `HxD`.
* Teknik: Jika file tidak bisa dibuka, cek header-nya di Hex Editor. Perbaiki jika tidak sesuai standar di atas.

#### 2. Strings Analysis

Mencari teks manusia di dalam tumpukan kode biner.

* Command:
  * `strings namafile` (Basic).
  * `strings -n 10 namafile` (Cari string minimal 10 karakter).
  * `strings namafile | grep -i "CTF"` (Filter flag).

#### 3. Metadata Extraction

Melihat informasi tersembunyi seperti GPS, Device info, Author, Comment.

* Tools: `exiftool`.
* Command: `exiftool namafile`.

#### 4. File Carving (Matryoshka Files)

Mengekstrak file yang disembunyikan/ditempel di dalam file lain (misal: ZIP di dalam PNG).

* Tools: `binwalk`, `foremost`.
* Command: `binwalk -e namafile` (Otomatis ekstrak semua yang ditemukan).

***

### 🖼️ BAB 3: Image Steganography (Stego)

Menyembunyikan pesan di dalam media visual.

#### 1. LSB (Least Significant Bit)

Menyisipkan data pada bit warna terkecil yang tidak mengubah tampilan gambar secara kasat mata.

* Target: PNG, BMP.
* Tools:
  * `zsteg -a file.png` (Paling ampuh buat Linux/Ruby).
  * `StegSolve.jar` (GUI Java, geser panah kiri/kanan untuk lihat layer warna).

#### 2. Passphrase Protected Stego

Data dienkripsi dan disisipkan, butuh password untuk ekstrak.

* Target: JPG, WAV (Audio).
* Tools: `steghide`.
* Command:
  * Cek info: `steghide info file.jpg`.
  * Ekstrak: `steghide extract -sf file.jpg`.
* Jika tidak punya password? Brute force pakai `stegseek` (cepat) atau `stegcracker`.

#### 3. Visual & Physical Manipulation

* Crop/Height Manipulation: Gambar dipotong (height diperkecil) untuk menyembunyikan bagian bawah.
  * _Cek:_ `pngcheck -v file.png` (Kalau error CRC, berarti ukuran dimanipulasi).
  * _Fix:_ Edit hex value untuk `Height` menjadi lebih besar.
* Colors: Teks ditulis dengan warna `#000001` di atas background `#000000`. Gunakan `StegSolve` atau `GIMP` (mainkan level/contrast).

***

### 🎵 BAB 4: Audio Forensics

#### 1. Visualisasi Gelombang (Spectrogram)

Buka file audio, ubah tampilan dari Waveform ke Spectrogram. Seringkali flag "digambar" di frekuensi suara.

* Tools: `Audacity`, `Sonic Visualiser`.

#### 2. Morse Code & DTMF

* Morse: Bunyi "tit tit tuuut". Decode manual atau pakai audio decoder online.
* DTMF: Bunyi tombol telepon. Gunakan tool online DTMF decoder.

***

### 🌐 BAB 5: Network Forensics (PCAP Analysis)

Menganalisis rekaman lalu lintas jaringan (`.pcap` atau `.pcapng`). Kamu pasti jago ini karena background bug hunting!

#### 1. Wireshark Essentials

* Protocol Hierarchy: Menu `Statistics` -> `Protocol Hierarchy`. Lihat protokol apa yang dominan (HTTP, FTP, SMB?).
* Follow Stream: Klik kanan paket -> `Follow` -> `TCP Stream` / `HTTP Stream`. Ini menyatukan paket-paket terpecah menjadi percakapan utuh.
* Export Objects: Menu `File` -> `Export Objects` -> `HTTP`. Mengambil file (gambar/exe/zip) yang pernah didownload lewat jaringan itu.

#### 2. Keyword Search

`Ctrl+F` -> Pilih "Packet Bytes" -> Cari string "CTF", "Password", "Login", "Authorization".

***

### 🧠 BAB 6: Advanced Forensics (Memory & Disk)

Level sulit. Menganalisis dump RAM atau Harddisk image.

#### 1. Memory Forensics (RAM)

Analisis file `.vmem` atau `.raw` (dump memori komputer hidup).

* Tool: `Volatility` (versi 2.6 atau 3).
* Perintah Wajib (Volatility 2 syntax):
  * `imageinfo`: Cek profil OS (misal Win7SP1x64).
  * `pslist`: Lihat proses/aplikasi apa yang sedang jalan.
  * `cmdscan` / `consoles`: Lihat apa yang diketik di Command Prompt (history).
  * `clipboard`: Lihat apa yang di-copy paste user (sering ada password di sini!).
  * `notepad`: Dump teks yang sedang dibuka di Notepad.
  * `filescan | grep flag`: Cari file bernama flag di memori.

#### 2. Disk Forensics

Mengembalikan file yang dihapus dari image harddisk (`.dd`, `.img`).

* Tools:
  * `Autopsy` (GUI, lengkap banget).
  * `TestDisk` (CLI, ampuh buat recover partisi).
  * `PhotoRec` (Temannya TestDisk, spesialis recover file media).

***

### 📝 BAB 7: Scripting & Decoding (The "Coder" Advantage)

Karena kamu bisa Python, ini super power kamu!

#### 1. Decoding

Seringkali data disamarkan dengan encoding.

* Base64: `ZmxhZ...` (akhiran `=`).
* Hex: `46 6C 61 67` (Angka 0-9, Huruf A-F).
* Rot13/Caesar: Huruf digeser.
* Tool: CyberChef (Website wajib bookmark! Bisa decode apa aja).

#### 2. Python for Forensics

Kadang kamu perlu bikin script kecil untuk:

* XOR decoding (operasi matematika bitwise).
* Parsing binary file custom.
* Brute force sederhana.

Python

```
# Contoh script XOR sederhana
data = open("file.bin", "rb").read()
key = 0x55
output = bytearray([b ^ key for b in data])
open("result.txt", "wb").write(output)
```

***

### 🎒 Checklist Persiapan (Install Ini di Kali Linux)

Pastikan tools ini siap panggil di terminal:

1. Analisis Dasar: `file`, `strings`, `grep`, `xxd`.
2. Gambar: `exiftool`, `binwalk`, `zsteg` (Ruby gem), `pngcheck`, `steghide`, `stegseek`.
3. Jaringan: `wireshark`, `tshark`.
4. OCR: `tesseract`.
5. Memori: `volatility` (biasanya perlu install manual atau docker).
6. Browser: Bookmark "CyberChef" dan "Decoder.fr".

***
