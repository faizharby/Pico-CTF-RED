# Pico-CTF-RED
Berikut adalah file `README.md` lengkap dengan analisis mendalam, metodologi, dan langkah-langkah verifikasi:

```markdown
# Laporan Analisis Steganografi & Forensik Digital

## Identifikasi Masalah
Data yang diberikan mengandung potongan informasi tersembunyi dalam berbagai format (teks, encoding, binary). Tujuan analisis adalah mengidentifikasi flag tersembunyi dan pola mencurigakan.

---

## Metodologi Analisis
1. **Identifikasi Awal**:
   - Ekstraksi semua teks dari output.
   - Deteksi encoding (Base64, Hex, dll.).
   - Analisis pola berulang/aneh.

2. **Tools Digunakan**:
   - `base64` (decode/encode)
   - `binwalk` (analisis file binary)
   - `xxd` (hex dump)
   - `gpg` (untuk OpenPGP keys)
   - `steghide` (steganografi gambar)

---

## Hasil Analisis Lengkap

### 1. Flag Utama (Terkonfirmasi)
- **Lokasi**: `b1,rgba,lsb,xy`
- **Data Mentah**:
  ```text
  cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==
  ```
- **Decoding**:
  ```bash
  echo "cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==" | base64 -d
  ```
  **Hasil**:  
  `picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}`  
- **Validasi**:
  - Format sesuai standar CTF (`picoCTF{...}`).
  - String yang di-decode masuk akal (red is the ultimate cure for sadness).

### 2. Pola Mencurigakan Lainnya

#### a. OpenPGP Keys
- **Lokasi**:
  - Public Key: `b1,bgr,msb,xy`, `b2,bgr,msb,xy`
  - Secret Key: `b2,rgb,lsb,xy`, `b2,rgba,lsb,xy`
- **Analisis**:
  - Keys mungkin digunakan untuk enkripsi tambahan.
  - **Ekstraksi**:
    ```bash
    binwalk -e file_input --run-as=root
    ```
  - **Verifikasi**:
    ```bash
    gpg --import extracted_key.pub
    gpg --list-keys
    ```

#### b. Teks Berulang
- **Lokasi**: `b2,g,lsb,xy`
- **Data**:
  ```text
  ET@UETPETUUT@TUUTD@PDUDDDPE
  ```
- **Analisis**:
  - Pola repetitif dengan karakter `@` dan `U`.
  - **Diduga**:
    - Cipher substitusi (contoh: A=U, B=@, ...).
    - Kode morse yang diencode ulang.

#### c. Binary Mencurigakan
- **Lokasi**: `b4,b,lsb,xy`
- **Tipe File**: `0421 Alliant compact executable not stripped`
- **Analisis**:
  - File executable mungkin mengandung payload.
  - **Ekstraksi**:
    ```bash
    dd if=file_input of=extracted_binary bs=1 skip=<offset>
    ```

---

## Teknik Steganografi yang Terdeteksi

### 1. LSB (Least Significant Bit)
- **Lokasi**: Multiple (`*lsb*` channels)
- **Keterangan**:
  - Data disembunyikan dalam bit terakhir pixel (untuk gambar).
  - **Tool Ekstraksi**:
    ```bash
    steghide extract -sf file_input -p ""
    ```

### 2. Base64 Obfuskasi
- **Lokasi**: `b1,abgr,msb,XY`
- **Data**:
  ```text
  ==QffVTNz4GZ0UzXyBjZfNjc1N2XzQHNtFDdsV3XzgGdfNXMfR2MytnRUN0bjlGc==...
  ```
- **Analisis**:
  - Bukan Base64 standar (ada padding `==` tapi decode gagal).
  - **Diduga**: Base64 + XOR cipher.
  - **Decoding Attempt**:
    ```python
    import base64
    data = "==QffVTNz4GZ0UzXyBjZfNjc1N2XzQHNtFDdsV3XzgGdfNXMfR2MytnRUN0bjlGc=="
    decoded = base64.b64decode(data[::-1])  # Reverse string attempt
    ```

---

## Langkah Verifikasi Tambahan

### 1. Pengecekan File Gambar
- Jika data berasal dari gambar:
  ```bash
  steghide info suspect_image.jpg
  strings suspect_image.jpg | grep "picoCTF"
  ```

### 2. Analisis Hexdump
- Cari signature file:
  ```bash
  xxd file_input | head -n 20
  ```

### 3. Bruteforce Password Steganografi
- Jika membutuhkan password:
  ```bash
  stegcracker suspect_image.jpg wordlist.txt
  ```

---

## Kesimpulan
- **Flag Terkonfirmasi**:  
  `picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}`  
- **Temuan Lain**:
  - OpenPGP keys berpotensi untuk enkripsi lanjutan.
  - Pola binary/teks tidak biasa memerlukan investigasi lebih dalam.

---

## Referensi
- [Steganography Cheat Sheet](https://github.com/DominicBreuker/stego-toolkit)
- [CTF Field Guide](https://trailofbits.github.io/ctf/)
```

### Fitur Tambahan:
1. **Kode Langsung**: Contoh perintah untuk ekstraksi/decoding.
2. **Struktur Jelas**: Pembagian per teknik (LSB, Base64, dll.).
3. **Langkah Verifikasi**: Memastikan flag benar dan opsi cadangan.

Untuk digunakan:
1. Simpan sebagai `README.md`.
2. Update path/nama file sesuai input aktual.
3. Jalankan perintah dalam lingkungan Linux dengan tools yang terinstall.
