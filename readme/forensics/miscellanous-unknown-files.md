# Miscellanous / Unknown Files

Di kategori ini, kamu dikasih file yang "tidak punya nama", ekstensinya aneh (misal `.bin`, `.dat`, `.enc`), atau bahkan tanpa ekstensi sama sekali. Tugasmu adalah: Identifikasi -> Perbaiki -> Ekstrak -> Flag.

***

## ❓ Panduan Lengkap CTF Misc: Unknown Files

### 1. Fase Identifikasi (Siapa Aku?) 🕵️‍♂️

Jangan pernah percaya ekstensi file. `flag.jpg` bisa saja isinya teks, dan `flag.txt` bisa saja isinya binary executable.

#### A. Cek "Magic Bytes" (Header Signature)

Setiap file punya tanda tangan digital di byte-byte awalnya.

*   Perintah Dasar:

    Bash

    ```
    file nama_file
    ```

    _Jika hasilnya `data` (artinya Linux tidak kenal filenya), lanjut ke langkah manual._
*   Cek Manual dengan Hex Editor:

    Lihat byte paling awal (Header).

    Bash

    ```
    xxd nama_file | head -n 5
    ```

    * `89 50 4E 47` -> PNG
    * `FF D8 FF` -> JPEG
    * `50 4B 03 04` -> ZIP/DOCX/APK
    * `25 50 44 46` -> PDF
    * `7F 45 4C 46` -> ELF (Linux Executable)
    * `4D 5A` -> EXE (Windows Executable)

#### B. Analisis Entropi (Encrypted or Compressed?)

Jika file isinya karakter acak total, kita perlu tahu apakah dia dienkripsi atau dikompres.

*   Gunakan Binwalk Entropy:

    Bash

    ```
    binwalk -E nama_file
    ```

    * Grafik Rata di Atas (Flat High): Kemungkinan besar terenkripsi atau terkompresi.
    * Grafik Naik Turun: Kemungkinan file biasa (teks/code/exe) atau steganografi.

***

### 2. File Repairing (Memperbaiki File Rusak) 🛠️

Seringkali soal CTF memberikan file yang _header_-nya dihapus (dinuulkan) atau dimanipulasi agar tidak bisa dibuka.

#### A. Hex Editing (Fixing Magic Bytes)

Jika `file` bilang `data` tapi `strings` menunjukkan ada potongan kata-kata XML atau IHDR (tanda PNG).

1. Buka file di Hex Editor (`hexeditor` atau `HxD`).
2. Cari referensi "File Signatures" di Google (Wikipedia atau Gary Kessler).
3. Tulis ulang byte awal sesuai tipe file yang dicurigai.
   * _Contoh:_ Kamu curiga ini PNG, tapi byte awalnya `00 00 00 00`. Ubah jadi `89 50 4E 47 0D 0A 1A 0A`.
4. Save dan coba buka lagi.

#### B. Archive Repair

Jika file ZIP/RAR rusak.

* Zip: `zip -FF file_rusak.zip --out fixed.zip`
* 7z: Kadang 7zip lebih jago buka file rusak daripada winrar/unzip biasa.

***

### 3. Extraction & Decompression (Buka Bungkus) 📦

Unknown file seringkali adalah arsip aneh atau _nested archives_ (zip di dalam rar di dalam tar).

#### A. The Universal Extractor: 7zip

Jangan cuma pakai `unzip`. `7z` bisa membuka format ISO, IMG, MSI, CAB, RPM, dll.

Bash

```
7z x nama_file
```

#### B. Binwalk (Carving)

Kalau filenya adalah "Firmware" atau gabungan banyak file jadi satu (misal: gambar ditempel file zip).

Bash

```
binwalk -e nama_file --run-as=root
```

_(Flag `-e` untuk extract otomatis)._

#### C. Format Arsip Kuno/Linux

Kadang soal CTF pakai format jadul:

* CPIO: `cpio -idv < nama_file`
* AR: `ar x nama_file`
* TAR: `tar xvf nama_file`
* XZ/GZ/BZ2: `gunzip` atau `unxz`.

***

### 4. Analisis Teks & Encoding (Bahasa Alien) 👽

Jika file tersebut ternyata berisi teks (ASCII) tapi tidak bisa dibaca.

#### A. Esoteric Languages (Bahasa Pemrograman Aneh)

Hacker suka pakai bahasa aneh untuk menyembunyikan logika.

* Brainfuck: Cirinya banyak tanda `+ - < > [ ] . ,`
* Ook!: Isinya cuma `Ook. Ook? Ook!`
* Piet: Filenya berupa gambar kotak-kotak warna-warni abstrak (ini adalah kodingan!).
* Whitespace: Filenya terlihat kosong, tapi isinya spasi dan tab.
  * _Solusi:_ Gunakan interpreter online seperti [dcode.fr](https://www.dcode.fr/) atau [Tio.run](https://tio.run/).

#### B. Classic Encoding

* Hex: `48 65 6C 6C 6F` -> Copy ke CyberChef (From Hex).
* Base64: `SGVsbG8=` (Akhiran `=`). -> CyberChef (From Base64).
* Base32: Huruf kapital semua dan angka 2-7.
* Rot13: Caesar Cipher.

***

### 5. Disk Images & File Systems 💾

Jika filenya berekstensi `.iso`, `.img`, atau file besar tak dikenal.

*   Mounting:

    Coba mount file tersebut, siapa tahu itu adalah flashdisk virtual.

    Bash

    ```
    mkdir /mnt/test
    mount -o loop nama_file /mnt/test
    ls -la /mnt/test
    ```
*   TestDisk:

    Jika file image partisinya rusak.

    Bash

    ```
    testdisk nama_file
    ```

***

### 6. XOR Analysis (Kunci Segala Rahasia) 🗝️

Seringkali file binary di-"XOR" dengan satu byte kunci agar tidak terbaca.

*   Tool: xortool

    Mencoba menebak panjang kunci dan isi kunci XOR.

    Bash

    ```
    xortool nama_file -c 20
    ```

    _(Mencoba menebak character yang paling sering muncul, biasanya spasi atau null byte)._
*   CyberChef (XOR Brute Force):

    Upload file ke CyberChef, pakai resep "XOR Brute Force". Lihat hasilnya mana yang memunculkan kata-kata masuk akal (seperti HTTP, flag, MZ).

***

### 🌟 Tips

1.  Dcode.fr & CyberChef:

    Dua website ini adalah "kitab suci" untuk kategori Misc. Kalau nemu teks aneh, kode aneh, bahasa aneh, langsung cari di sana.
2.  Google Lens/Search:

    Kalau filenya berisi simbol-simbol aneh (misal: simbol alien dari Futurama atau Klingon), screenshot dan cari di Google Images. Itu namanya Cipher Substitution.
3.  Python Scripting:

    Seringkali file Misc berisi data mentah yang harus diolah. Contoh: "Diberikan file berisi 1 juta angka, ambil angka yang ganjil saja lalu ubah ke ASCII". Kamu harus siap bikin script Python cepat.

    Python

    ```
    # Contoh script cepat baca file binary
    with open('file.bin', 'rb') as f:
        data = f.read()
        # Lakukan logika di sini
    ```
4.  Cek grep -a:

    Jika nge-grep file binary, jangan lupa pakai -a (treat binary as text).

    Bash

    ```
    grep -a "CTF" file_binary_aneh
    ```
