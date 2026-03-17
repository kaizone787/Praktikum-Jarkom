# Laporan Praktikum Jaringan Komputer
## Modul 3: HTTP

**Nama:**   Muhammad Chaesar Pratama 
**NIM:** 103072400119  
**Kelas:** IF-04-01  
**Minggu / Modul Ke-:** 3  

---

#### 3.1 Basic HTTP GET/response interaction
**Tujuan:** Mengunduh file HTML sederhana yang sangat pendek dan tidak berisi objek yang disematkan.

**Bukti *Screenshot* Wireshark:**
> *![Figure 3.1 Basic HTTP GET](images/Figure3.1.png)*

**Analisis:**
Saat browser meminta `file1.html`, terjadi pertukaran pesan yang sederhana: klien mengirimkan satu pesan HTTP `GET` ke server (`gaia.cs.umass.edu`), dan server merespons dengan pesan HTTP `200 OK`. Respons dari server ini langsung membawa isi teks dari file HTML tersebut di dalam paketnya. Proses ini menunjukkan interaksi dasar tanpa hambatan atau fragmentasi karena ukuran file yang sangat kecil.

#### 3.2 HTTP CONDITIONAL GET/response interaction
**Tujuan:** Melakukan GET bersyarat saat mengambil objek HTTP yang dipengaruhi oleh aktivitas *caching* pada *browser web*.

**Bukti *Screenshot* Wireshark:**
> *![Figure 3.2 HTTP Conditional GET](images/Figure3.2.png)*

**Analisis:**
Pada percobaan ini, karena browser sudah menyimpan *cache* dari akses sebelumnya, browser mengirimkan *request* `GET` bersyarat yang menyertakan header `If-Modified-Since`. Server mengecek apakah file tersebut telah diubah sejak waktu yang tertera. Jika tidak ada perubahan, server tidak akan membuang *bandwidth* dengan mengirim ulang isi file, melainkan hanya membalas dengan status `304 Not Modified`.

#### 3.3 Retrieving Long Documents
**Tujuan:** Mengamati perilaku HTTP saat mengunduh dokumen file HTML yang berukuran cukup panjang (sekitar 4500 byte), yang melebihi batas ukuran satu paket TCP.

**Bukti *Screenshot* Wireshark:**
> *![Figure 3.3 Retrieving Long Documents](images/Figure3.3.png)*

**Analisis:**
Pada percobaan ini, saat mengunduh file berukuran sekitar 4500 byte, terlihat keterangan `[4 Reassembled TCP Segments (4864 bytes)]` pada bagian paket HTTP `200 OK`. Hal ini terjadi karena ukuran file tersebut melebihi batas Maximum Transmission Unit (MTU) jaringan yang standarnya sekitar 1500 byte. Akibatnya, protokol TCP harus memecah (memfragmentasi) dokumen tersebut menjadi 4 segmen yang lebih kecil agar dapat ditransmisikan melintasi jaringan, sebelum akhirnya dirakit kembali (*reassembled*) di sisi *client*.

#### 3.4 HTML Documents dengan Embedded Objects
**Tujuan:** Menganalisis apa yang terjadi ketika *browser* mengunduh dokumen HTML yang menyertakan objek lain (seperti gambar) yang direferensikan melalui URL dan disimpan di server yang berbeda.

**Bukti *Screenshot* Wireshark:**
> *![Figure 3.4 Embedded Objects](images/Figure3.4.png)*

**Analisis:**
Saat mengakses halaman yang memiliki objek tersemat (seperti gambar logo pearson dan kurose), browser tidak mendapatkan semuanya sekaligus. Browser pertama-tama mengunduh file HTML dasar terlebih dahulu melalui *request* `GET`. Setelah mem-*parsing* file HTML tersebut dan menemukan referensi URL ke objek gambar, browser secara otomatis menginisiasi *request* HTTP `GET` baru secara spesifik untuk mengambil masing-masing gambar tersebut dari server.

#### 3.5 HTTP Authentication
**Tujuan:** Mengunjungi situs web yang dilindungi oleh kata sandi dan memeriksa urutan pesan HTTP yang dipertukarkan.

**Bukti *Screenshot* Wireshark:**
> *![Figure 3.5 HTTP Authentication](images/Figure3.5.png)*

**Analisis:**
Pada simulasi halaman web yang dilindungi sandi (`protected_pages`), percobaan akses pertama tanpa sandi langsung ditolak oleh server dengan status `401 Unauthorized`. Setelah kredensial (*username* dan *password*) dimasukkan pada *pop-up*, klien mengirimkan *request* `GET` kedua menggunakan header `Authorization: Basic`. Analisis pada *header* ini membuktikan bahwa metode tersebut tidak aman karena kredensial tidak dienkripsi, melainkan hanya dikodekan menggunakan format Base64 (terlihat sebagai string `d2lyZXNoYXJr...`). Siapa pun yang menyadap lalu lintas jaringan dapat dengan mudah mendekode string Base64 tersebut untuk melihat kata sandi aslinya.

### 4. Kesimpulan
Berdasarkan praktikum Modul 3 mengenai investigasi protokol HTTP menggunakan Wireshark, dapat ditarik beberapa kesimpulan sebagai berikut:

1. **Interaksi Dasar HTTP:** Komunikasi dasar antara *client* dan server terjadi melalui pertukaran pesan, di mana *client* mengirimkan *request* HTTP `GET` dan server membalas dengan *response* beserta kode status (seperti `200 OK`).
2. **Optimalisasi Caching:** Mekanisme *Conditional GET* (menggunakan *header* `If-Modified-Since`) memungkinkan *browser* memvalidasi *cache* lokal. Jika file tidak mengalami perubahan, server membalas dengan status `304 Not Modified` tanpa mengirim ulang isi file.
3. **Penanganan Dokumen Besar:** Dokumen yang ukurannya melebihi batas MTU (*Maximum Transmission Unit*) jaringan akan difragmentasi menjadi beberapa segmen TCP agar dapat ditransmisikan, lalu dirakit kembali (*reassembled*) setibanya di tujuan.
4. **Pengambilan Objek Tersemat:** Ketika *browser* memuat dokumen HTML yang memiliki referensi objek lain (seperti gambar), *browser* akan mengunduh file HTML utamanya terlebih dahulu, kemudian secara otomatis mengirimkan *request* HTTP `GET` baru untuk setiap objek gambar yang ditemukan.
5. **Celah Keamanan Autentikasi Dasar:** Penggunaan *HTTP Basic Authentication* rentan terhadap penyadapan. Kredensial pengguna hanya dikodekan menggunakan Base64 pada *header* `Authorization: Basic`, tanpa proses enkripsi, sehingga data dapat dengan mudah dicegat dan dibaca menggunakan *packet sniffer*.