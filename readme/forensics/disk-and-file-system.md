# Disk & File System

Di kategori ini, kamu akan diberikan _image_ dari sebuah harddisk, flashdisk, atau partisi sistem operasi. File-nya biasanya berekstensi `.dd`, `.img`, `.E01`, `.iso`, atau `.vmdk`.

Tujuannya? Mengembalikan file yang dihapus, mencari log aktivitas user, membongkar registry, atau menemukan file rahasia di folder sistem.

***

## 💾 Panduan Lengkap CTF Disk Forensics

### 1. Fase Reconnaissance (Bedah Luar) 🔍

Sebelum "membedah" isinya, cek dulu jenis disk image-nya.

*   Identifikasi File:

    Bash

    ```
    file evidence.dd
    ```
*   Cek Partisi (Partition Table):

    Kamu perlu tahu struktur partisinya (di mana mulai, di mana berakhir).

    Bash

    ```
    fdisk -l evidence.dd
    ```

    _Atau gunakan tool dari The Sleuth Kit (TSK):_

    Bash

    ```
    mmls evidence.dd
    ```

    _(Catat Start Sector/Offset dari partisi yang ukurannya paling besar/masuk akal)._

***

### 2. The Sleuth Kit (TSK): Bedah Tanpa Mounting 🕵️‍♂️

Ini adalah Skill Wajib di Linux. Kamu bisa melihat isi file image _tanpa_ perlu menempelkannya (mount) ke sistem operasi kamu.

#### A. Listing File (`fls`)

Melihat daftar file (termasuk yang dihapus).

*   Command:

    Bash

    ```
    fls -o [offset_partisi] evidence.dd
    ```

    _Contoh:_ `fls -o 2048 evidence.dd`
*   Tanda File Dihapus:

    Perhatikan file yang ada tanda bintang \* atau tulisan (deleted). Itu biasanya tempat flag bersembunyi!
*   Cari Folder Spesifik:

    Bash

    ```
    fls -r -o 2048 evidence.dd | grep "Desktop"
    ```

#### B. Extract File (`icat`)

Mengambil file keluar dari image berdasarkan Inode Number (angka di sebelah kiri nama file hasil `fls`).

*   Command:

    Bash

    ```
    icat -o [offset_partisi] evidence.dd [inode_number] > file_rahasia.txt
    ```

***

### 3. Mounting (Menempelkan Disk) 🔧

Kalau kamu ingin menelusuri folder-folder layaknya buka File Explorer biasa.

*   Mounting Image Raw (.dd/.img):

    Gunakan mount dengan opsi loop.

    Bash

    ```
    # Buat folder mounting point dulu
    mkdir /mnt/analisis

    # Mount (Gunakan offset bytes: Sector Start * 512)
    # Misal start sector 2048, maka offset = 2048 * 512 = 1048576
    sudo mount -o ro,loop,offset=1048576 evidence.dd /mnt/analisis
    ```

    _Note: Opsi `ro` (Read-Only) SANGAT PENTING agar kamu tidak merusak bukti/flag secara tidak sengaja._

***

### 4. File Carving (Recover Data Hilang) ⛏️

Jika file sudah dihapus dan sistem file-nya rusak (tidak terbaca di `fls`), gunakan teknik _Carving_. Teknik ini mencari "header" file (misal header JPG, PDF) di tumpukan data mentah.

#### A. Foremost

Tool klasik, cepat, dan efektif.

*   Command:

    Bash

    ```
    foremost -i evidence.dd -o hasil_recovery/
    ```

    _Cek folder `hasil_recovery/`. File akan dikelompokkan berdasarkan jenis (jpg, pdf, zip)._

#### B. Photorec (Bagian dari TestDisk)

Lebih powerful dari Foremost, mengenali lebih banyak jenis file.

* Command: `photorec evidence.dd`
* Ikuti UI terminal-nya -> Pilih partisi -> Pilih \[Whole] (scan seluruh disk) -> Tunggu proses selesai.

***

### 5. Artifact Hunting (Apa yang dicari?) 🎯

Tergantung OS target (Windows/Linux), ini lokasi-lokasi "emas"-nya:

#### A. Jika Targetnya LINUX 🐧

1. History Perintah:
   * `/home/user/.bash_history` (Apa yang diketik user? Sering ada password/flag di sini).
2. SSH Keys:
   * `/home/user/.ssh/id_rsa` (Kunci rahasia untuk masuk server lain).
3. Log System:
   * `/var/log/auth.log` (Siapa yang login, sudo, dll).
   * `/var/log/syslog`
4. Shadow File (Hash Password):
   * `/etc/shadow` (Crack hash-nya pakai Hashcat/John).

#### B. Jika Targetnya WINDOWS 🪟

1. Registry Hives: (Lokasi: `C:\Windows\System32\config\`)
   * SAM: Berisi hash password user.
   * SOFTWARE: Daftar aplikasi terinstall.
   * SYSTEM: Info sistem & USB yang pernah colok.
   * _Tools Analisis:_ Gunakan RegRipper (`rip.pl`) untuk membaca file registry ini di Linux.
2. NTFS Artifacts ($MFT):
   * Master File Table berisi metadata semua file. Analisis pakai tool `analyzeMFT.py`.
3. Recycle Bin:
   * Folder `$Recycle.Bin`. Cek isinya, mungkin user menghapus flag.
4. Prefetch:
   * `C:\Windows\Prefetch`. Melihat aplikasi apa yang baru saja dijalankan.

***

### 6. The "Heavy" GUI: Autopsy 🐕

Kalau soalnya sangat kompleks dan butuh analisis mendalam (keyword search, timeline analysis), gunakan Autopsy.

* Di Kali Linux: `sudo autopsy`
* Buka browser di `http://localhost:9999/autopsy`.
* Buat Case baru -> Add Image -> Tunggu proses _ingest_ selesai.
* Fitur andalan: Timeline Analysis (Melihat apa yang terjadi menit demi menit) dan Keyword Search.

***

### 🌟 Tips

1.  Grep Strings (Selalu!):

    Sebelum pakai tools ribet, coba keberuntungan dulu.

    Bash

    ```
    strings evidence.dd | grep -i "CTF{"
    strings evidence.dd | grep -t d | grep "http" (Cari URL)
    ```
2.  BitLocker/LUKS (Disk Terenkripsi):

    Kalau pas di-mount minta password, berarti disk dienkripsi.

    * Cari file "recovery key" atau file gambar berisi password di sekitar folder user.
3.  Mounting File E01 (Expert Witness Format):

    Format .E01 adalah format standar forensik profesional. Untuk mount di Linux, butuh tools tambahan:

    Bash

    ```
    ewfmount evidence.E01 /mnt/ewf
    # Setelah command ini, akan muncul file raw image di /mnt/ewf yang bisa kamu mount biasa.
    ```
