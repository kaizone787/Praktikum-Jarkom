# Laporan Praktikum Jaringan Komputer
## Modul 6: TCP

**Nama:** Muhammad Chaesar Pratama  
**NIM:** 103072400119  
**Kelas:** IF-04-01  
**Minggu / Modul Ke-:** 6  

---

### 6.2 Menangkap Transfer TCP dalam Jumlah Besar

**Tujuan:** Mempersiapkan file teks berukuran cukup besar dan menangkap lalu lintas jaringan saat file tersebut diunggah ke *remote server* menggunakan protokol HTTP POST.

**Langkah Persiapan:**
1. Mengunduh salinan ASCII dari naskah *Alice in Wonderland*.
![Isi File alice.txt](assets/image/Screenshot%202026-06-22%20192825.png)

2. Mengakses halaman unggah lab Wireshark TCP dan memilih file yang akan dikirim.
![Halaman Upload](assets/image/Screenshot%202026-06-22%20192849.png)

3. Berhasil mengunggah file `alice.txt` ke server `gaia.cs.umass.edu`.
![Upload Berhasil](assets/image/Screenshot%202026-06-22%20192838.png)

---

### 6.3 Tampilan Awal pada Captured Trace

**Tujuan:** Menganalisis alamat IP dan *port* yang digunakan dalam transaksi pengiriman pesan HTTP melalui protokol TCP.

**Bukti *Screenshot* Wireshark:**
![Initial Trace TCP](assets/image/Screenshot%202026-06-22%20194237.png) 

**Analisis / Jawaban:**
1. Berdasarkan *trace* jaringan, alamat IP komputer klien (sumber) yang mentransfer file ke `gaia.cs.umass.edu` adalah **192.168.1.102**, dan menggunakan nomor port TCP **1161**.
2. Alamat IP dari server `gaia.cs.umass.edu` adalah **128.119.245.12**. Server ini mengirim dan menerima segmen TCP untuk koneksi tersebut melalui port **80**.

---

### 6.4 Dasar TCP

**Tujuan:** Menginvestigasi tahapan *three-way handshake*, nomor urut (*sequence number*), *acknowledgement*, ukuran segmen, serta parameter koneksi TCP lainnya.

**Bukti *Screenshot* Wireshark (Awal Koneksi / Handshake):**
![Three-Way Handshake](assets/image/Screenshot%202026-06-22%20194415.png)

**Bukti *Screenshot* Wireshark (Perintah HTTP POST):**
![HTTP POST Command](assets/image/Screenshot%202026-06-22%20194514.png)

**Analisis / Jawaban:**
1. Nomor urut (*sequence number*) segmen TCP SYN yang digunakan untuk memulai sambungan adalah **0** (*relative*). Segmen tersebut teridentifikasi sebagai segmen SYN karena pada bagian `Flags` di *header* TCP, *bit* **SYN bernilai 1 (Set)**.
2. Nomor urut segmen SYNACK yang dikirim oleh server sebagai balasan adalah **0** (*relative*). Nilai dari *field* `Acknowledgement` adalah **1**. Server menentukan nilai ini dengan menambahkan 1 pada *sequence number* inisial klien. Segmen ini dikenali sebagai SYNACK karena *bit* **SYN bernilai 1** dan *bit* **ACK bernilai 1**.
3. Segmen TCP yang berisi perintah HTTP POST memiliki *sequence number* **1**.
4. Menganggap segmen berisi HTTP POST sebagai segmen pertama, berikut adalah nomor urut dari enam segmen pertama:
   * **Segmen 1:** Seq = **1**
   * **Segmen 2:** Seq = **566**
   * **Segmen 3:** Seq = **2026**
   * **Segmen 4:** Seq = **3486**
   * **Segmen 5:** Seq = **4946**
   * **Segmen 6:** Seq = **6406**
5. Panjang (*length*) dari enam segmen TCP pertama bervariasi bergantung pada ukuran muatan (seperti 565 byte pada POST), namun data utamanya memiliki panjang maksimum segmen (MSS) sebesar **1460** byte.
6. Jumlah minimum ruang *buffer* yang tersedia yang disarankan kepada penerima adalah **5840** bytes. Dalam *trace* ini, pengiriman tidak terhambat oleh kurangnya ruang *buffer* penerima.
7. Pada *trace* ini, **TIDAK ADA** segmen yang ditransmisikan ulang. Hal ini dipastikan dengan memeriksa *field sequence number* yang terus maju tanpa ada indikasi `[TCP Retransmission]` dari Wireshark.
8. Penerima biasanya mengakui data yang diterima dalam ACK secara kumulatif. Terdapat kasus di awal koneksi di mana penerima melakukan ACK untuk setiap segmen yang diterima secara individual.
9. *Throughput* untuk sambungan TCP ini dapat diestimasi dengan mengkalkulasi keseluruhan *payload* yang berhasil ditransfer lalu membaginya dengan total durasi sesi pengiriman.

---

### 6.5 Congestion Control pada TCP

**Tujuan:** Mengamati algoritma *congestion control* TCP (termasuk fase *slow start* dan *congestion avoidance*) dengan memanfaatkan grafik *Time-Sequence*.

**Bukti Grafik Time-Sequence (Stevens):**
![Time-Sequence Graph](assets/image/Screenshot%202026-06-22%20195023.png)

**Analisis / Jawaban:**
1. Berdasarkan alat *plotting Time-Sequence-Graph (Stevens)*, fase ***slow start*** TCP terlihat pada awal grafik di mana kurva naik secara eksponensial (jarak vertikal antar titik melebar dengan cepat). Algoritma ***congestion avoidance*** mengambil alih ketika kurva berubah menjadi pertumbuhan linier yang lebih landai. Data yang diukur di lapangan seringkali menunjukkan sedikit fluktuasi karena variasi latensi (*delay jitter*) dan proses di OS, berbeda dengan kurva teori TCP yang mulus dan ideal.

---

### Kesimpulan

Praktikum Modul 6 mengenai protokol TCP menggunakan Wireshark menghasilkan beberapa temuan utama:
1. **Inisiasi Koneksi:** Transmisi TCP selalu didahului oleh proses *three-way handshake* (SYN, SYN-ACK, ACK) untuk sinkronisasi parameter awal antara *client* dan server.
2. **Keandalan Transmisi:** TCP menggunakan kombinasi *Sequence Number* dan *Acknowledgement Number* untuk melacak pengiriman paket dan memastikan tidak ada data yang hilang atau rusak.
3. **Mekanisme *Flow Control*:** TCP mencegah *buffer overflow* di sisi penerima dengan secara dinamis mengiklankan sisa kapasitas *buffer* melalui parameter *Window Size*.
4. **Mekanisme *Congestion Control*:** Visualisasi grafik transmisi mengonfirmasi keberadaan mekanisme pencegahan kemacetan jaringan, yang diawali dengan peningkatan laju pengiriman secara agresif (*slow start*), lalu beralih ke peningkatan linier yang berhati-hati (*congestion avoidance*) setelah mendekati batas kapasitas jaringan.