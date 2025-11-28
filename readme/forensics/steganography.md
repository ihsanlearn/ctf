# Steganography

***

## 🚩 Cheatsheet Lengkap: CTF Steganography

### 1. Fase Reconnaissance (Analisis Awal) 🔍

_Lakukan ini segera setelah kamu mengunduh file, sebelum menggunakan tools berat._

#### Kasus: Mendapatkan File Misterius (Tanpa Ekstensi atau Ekstensi Mencurigakan)

Terkadang ekstensi file menipu (misal `flag.jpg` padahal isinya text).

*   Cek Identitas File:

    Gunakan command ini untuk melihat Magic Bytes header file.

    Bash

    ```
    file nama_file
    ```
*   Cek String Terbaca:

    Seringkali pembuat soal lupa (atau sengaja) menaruh flag atau password dalam bentuk teks biasa di dalam file binary.

    Bash

    ```
    strings nama_file | grep -i "flag"
    strings nama_file | head -n 20
    ```
*   Cek Metadata (EXIF):

    Informasi tersembunyi bisa ada di Comment, Author, atau GPS Coordinates.

    Bash

    ```
    exiftool nama_file
    ```

***

### 2. Image Steganography (Gambar) 🖼️

#### A. Analisis Visual & LSB (Least Significant Bit)

_Teknik ini memanipulasi bit warna terkecil pada pixel sehingga tidak terlihat mata telanjang._

* Tools:
  1. StegSolve (Java Tool):
     * _Cara pakai:_ Buka gambar di StegSolve -> Tekan panah kiri/kanan.
     * _Cari apa:_ Lihat layer "Red/Green/Blue plane 0". Jika ada noise aneh atau QR Code/Teks muncul, itu flag-nya.
  2. zsteg (Khusus PNG/BMP):
     * Tool ini sangat _powerful_ untuk mendeteksi LSB secara otomatis.
     *   _Command:_

         Bash

         ```
         zsteg -a nama_file.png
         ```

#### B. File Extraction (Ada File di Dalam File)

_Kasus: Gambar terlihat normal, tapi ukurannya tidak wajar (terlalu besar)._

* Tools:
  1. Binwalk:
     * Menganalisis hex signature untuk melihat apakah ada file lain (misal: ZIP, PDF) yang "ditempel" di belakang gambar.
     *   _Command Extract:_

         Bash

         ```
         binwalk -e nama_file.jpg
         ```
  2. Foremost:
     * Alternatif binwalk untuk teknik _carving_ (mengukir/mengambil data).
     * _Command:_ `foremost -i nama_file.jpg`

#### C. Embedding dengan Password

_Kasus: Kamu menemukan password/passphrase dari tahap Recon (strings) atau deskripsi soal._

* Tools:
  1. Steghide (JPG & WAV):
     * Tools klasik untuk menyembunyikan pesan teks/file dalam gambar JPEG atau audio WAV.
     *   _Command Extract:_

         Bash

         ```
         steghide extract -sf gambar.jpg
         ```

         _(Akan diminta password. Jika tidak punya, tekan Enter (kosong) atau lakukan cracking)._
  2. Stegseek (Bruteforce Steghide):
     * Jika kamu tidak punya password, gunakan ini (jauh lebih cepat dari stegcracker). Gunakan wordlist `rockyou.txt`.
     *   _Command:_

         Bash

         ```
         stegseek gambar.jpg /usr/share/wordlists/rockyou.txt
         ```

#### D. Perbaikan Header (Corrupt Image)

_Kasus: Gambar tidak bisa dibuka di viewer biasa._

* Buka file menggunakan Hex Editor (seperti `HxD`, `Bless`, atau `ghex`).
* Cek byte awal (Magic Bytes).
  * PNG harus diawali: `89 50 4E 47 0D 0A 1A 0A`
  * JPG harus diawali: `FF D8 FF`
* Jika byte awalnya salah, perbaiki manual di Hex Editor agar gambar bisa terbuka.

***

### 3. Audio Steganography 🎵

#### A. Analisis Spektrogram

_Kasus: Mendengar suara aneh (noise, glitch) di tengah lagu/rekaman._

* Tools: Audacity atau Sonic Visualizer.
* Teknik:
  1. Buka file audio.
  2. Ubah tampilan dari "Waveform" menjadi "Spectrogram".
  3. Lihat secara visual. Seringkali flag "digambar" menggunakan frekuensi suara sehingga akan muncul tulisan di layarmu.

#### B. Hidden Messages

* Morse Code: Jika terdengar bunyi "tiit-tuit" (panjang pendek), decode menggunakan [Morse Decoder Online](https://morsecode.world/international/decoder/audio-decoder-adaptive.html).
* DTMF: Jika terdengar seperti suara tombol telepon, gunakan DTMF decoder.

***

### 4. Text & Whitespace 📄

*   Stegsnow:

    Menyembunyikan pesan menggunakan spasi dan tab (whitespace) di akhir baris file teks ASCII.

    * _Command:_ `stegsnow -C file.txt`
*   Zero Width Character:

    Karakter yang tidak memiliki lebar (tidak terlihat) tapi ada datanya. Copy paste teks ke tool online seperti Diff Checker atau \[Unicode Inspector].

***

### 🌟 Tips

1. Polyglot Files: Hati-hati dengan file yang valid sebagai gambar TAPI juga valid sebagai file ZIP atau script PHP. Cek dengan `binwalk`.
2. CyberChef: Simpan link [CyberChef](https://gchq.github.io/CyberChef/). Ini adalah "Swiss Army Knife" untuk decoding (Base64, Hex, ROT13, XOR) secara cepat di browser.
3. Google Lens: Jika flagnya berupa gambar QR Code yang rusak atau potongan gambar aneh, coba scan atau cari referensinya.
