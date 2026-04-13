# **LAPORAN PRAKTIKUM JARINGAN KOMPUTER - MODUL 4**
## **Domain Name System (DNS)**

### **Identitas Mahasiswa**
**Nama:** Muhammad Chaesar Pratama  
**NIM:** 103072400119
**Kelas:** IF - 04 - 01

---

## A. Tujuan Praktikum
1. Melakukan investigasi komprehensif mengenai mekanisme kerja DNS dengan memanfaatkan perangkat lunak analisis jaringan Wireshark.

---

## B. Pengantar
Dalam infrastruktur internet, DNS memiliki fungsi krusial sebagai sistem resolusi yang menerjemahkan nama host (domain) menjadi alamat IP yang dapat dipahami oleh mesin. Modul praktikum ini difokuskan pada analisis operasional DNS dari perspektif klien. Secara fundamental, proses yang terjadi melibatkan pengiriman *query* (permintaan) dari klien menuju *local DNS server* untuk kemudian mendapatkan respons hasil resolusi. Klien itu sendiri tidak perlu memproses alur komunikasi kompleks di latar belakang—seperti komunikasi rekursif maupun iteratif antar server DNS hierarkis—karena proses tersebut ditangani sepenuhnya oleh server.

---

## C. Hasil dan Pembahasan

### 1. Penggunaan Perintah Nslookup
Modul ini mengimplementasikan penggunaan perintah `nslookup`. Pada sistem operasi Windows, utilitas ini dapat dieksekusi melalui *Command Prompt* dengan mengetikkan `nslookup`. Alat ini memfasilitasi *host* untuk mengajukan *query* dan mengekstraksi record data dari server DNS. Permintaan ini dapat ditujukan ke berbagai lapisan hierarki server, mulai dari *root*, *top-level domain* (TLD), server otoritatif, hingga *resolver* perantara. Mekanisme kerjanya dimulai saat perintah dikirimkan ke server DNS yang terkonfigurasi, klien kemudian menunggu respons, dan hasilnya akan dicetak langsung pada layar.

#### a. Resolusi Standar (`nslookup www.mit.edu`)
![nslookup www.mit.edu](assets/image/Screenshot%202026-03-31%20080955.png)
Keluaran dari eksekusi perintah ini menyajikan dua blok informasi utama:
- Identifikasi server DNS lokal yang merespons permintaan, mencakup nama (*tusbind.ac.id*) dan alamat IP-nya (*10.217.7.77*).
- Hasil resolusi *query*, yang menampilkan deretan alamat IP terkait domain *www.mit.edu*.

#### b. Pencarian *Name Server* (`nslookup -type=NS mit.edu`)
![nslookup -type=NS mit.edu](assets/image/Screenshot%202026-03-31%20080955.png)
- Argumen `-type=NS` secara spesifik menginstruksikan server DNS lokal untuk melacak *hostname* dari server DNS otoritatif yang mengelola domain MIT.
- Tampilan awal menunjukkan identitas server DNS lokal yang bertugas mengeksekusi pencarian.
- Munculnya status *"Non-authoritative answer"* mengindikasikan bahwa data balasan diambil dari *cache* milik server perantara, bukan hasil respons langsung dari server utama MIT.
- Sebagai tambahan informasi, server lokal secara otomatis menyertakan alamat IP dari masing-masing server otoritatif tersebut tanpa adanya instruksi eksplisit.

#### c. *Query* Langsung ke Server DNS Spesifik
![nslookup www.mit.edu 8.8.8.8](assets/image/Screenshot%202026-03-31%20081029.png)
Menjalankan utilitas dengan format `nslookup www.mit.edu 8.8.8.8` dirancang untuk me-rutekan permintaan secara langsung ke server DNS eksternal yang ditentukan.
- **Tujuan:** Mengidentifikasi alamat IP untuk host *www.mit.edu*.
- **Pengalihan Server:** Kehadiran IP `8.8.8.8` di akhir perintah memaksa klien untuk melakukan *bypass* terhadap server lokal bawaan, dan mengirimkan *query* langsung ke Google Public DNS.
- **Analisis:** Baris *Server* menunjukkan bahwa IP `8.8.8.8` menangani balasan tersebut. Karena Google bertindak sebagai *resolver* publik yang mengandalkan *cache*, respons yang dilampirkan tetap dilabeli sebagai *"non-authoritative answer"*.

#### d. Investigasi IP Server Web Regional Asia
![nslookup en.snu.ac.kr](assets/image/Screenshot%202026-03-31%20081029.png)
Analisis Output:
- Bagian *Address* teratas merupakan identitas IP server DNS lokal yang beroperasi melayani permintaan klien.
- Terdapat parameter `new.snu.ac.kr`, yang menunjukkan *hostname* definitif dari server yang meng-hosting halaman situs web tersebut.
- Dua format pengalamatan IP dikembalikan: IPv6 (misalnya `64:ff9b::932e:a81` dari record AAAA) serta IPv4 (`147.46.10.129` dari record A).

#### e. Pencarian Server Otoritatif (NS Record)
![nslookup -type=NS ox.ac.uk](assets/image/Screenshot%202026-03-31%20081045.png)
Analisis Output:
- Eksekusi ini dirancang untuk mendeteksi data *Name Server* (NS) bagi *top-level domain* universitas `ox.ac.uk`.
- Proses pencarian dikelola oleh *local DNS server* dengan IP `10.55.39.218`.
- Ditemukan sejumlah server DNS otoritatif yang memegang kendali atas domain tersebut, di antaranya memuat *hostname* seperti `auth5.dns.ox.ac.uk`, `dns0.ox.ac.uk`, hingga `auth6.dns.ox.ac.uk`.

#### f. Pelacakan Infrastruktur Email (MX Record)
![nslookup -type=MX ox.ac.uk](assets/image/Screenshot%202026-03-31%20081045.png)
Analisis Output:
- Penambahan parameter `-type=MX` mendikte utilitas untuk melakukan penyaringan khusus pada record *Mail Exchanger* (MX). Analogi praktisnya: "Ke alamat server manakah rute pengiriman surat elektronik dengan tujuan domain `ox.ac.uk` harus diarahkan pertama kali?".
- Ditampilkan atribut `MX preference = 4`, yang merepresentasikan matriks prioritas perutean pada infrastruktur *mail server*.
- Dalam skenario keberadaan ganda record MX, agen pengiriman email akan berupaya menjalin koneksi dengan *mail server* yang memiliki digit *preference* paling rendah terlebih dahulu.

---

### 2. Utilitas Jaringan Ipconfig
Perintah `ipconfig` merupakan utilitas dasar yang berfungsi untuk mengekstraksi dan menyajikan parameter konfigurasi TCP/IP sistem. Data yang ditampilkan mencakup rincian IP mesin, rujukan server DNS, detail adaptor antarmuka jaringan (NIC), dan informasi *leasing* DHCP.

#### a. Informasi Detail dengan `ipconfig /all`
![ipconfig /all](assets/image/Screenshot%202026-03-31%20081723.png)
- Parameter `/all` mengekspos profil jaringan *host* secara menyeluruh.
- **IPv4 Address:** Konfigurasi alamat jaringan lokal saat ini yang digunakan klien (contoh: `10.55.39.11`).
- **Subnet Mask:** Indikator pembatas rentang blok IP jaringan lokal.
- **Default Gateway:** Menandakan titik keluar masuknya rute jaringan, umumnya mengarah pada alamat router (*Gateway IPv4* `10.55.39.218`).
- **DNS Servers:** Entitas *server* yang dihubungi pertama kali untuk tugas penerjemahan nama domain (tercatat di IP `10.55.39.218`).
- **Physical Address:** Deretan heksadesimal yang merepresentasikan MAC *Address* milik *hardware* adaptor.

#### b. Pemantauan Cache dengan `ipconfig /displaydns`
![ipconfig /displaydns](assets/image/Screenshot%202026-03-31%20081807.png)
- Perintah ini dimanfaatkan untuk menampilkan entri DNS yang riwayatnya tersimpan secara internal di memori klien (*local cache*).
- Tabel yang dihasilkan mendemonstrasikan detail spesifik dari setiap record berserta rentang *Time To Live* (TTL) aktual yang masih tersisa, diukur dalam satuan detik.

---

### 3. Analisis Lalu Lintas DNS Menggunakan Wireshark

#### a. Tracing Komunikasi DNS Reguler (Tanpa Utilitas *nslookup*)
![Permintaan DNS Frame 79](assets/image/Screenshot%202026-03-31%20083911.png)
![Detail Query Frame 79](assets/image/Screenshot%202026-03-31%20084028.png)
![Balasan DNS Frame 93](assets/image/Screenshot%202026-03-31%20084734.png)

**Pertanyaan dan Evaluasi Analisis:**

1. **Apakah pesan tersebut dikirimkan melalui UDP atau TCP?**
   > Berdasarkan pemantauan paket, baik permintaan maupun balasan resolusi DNS dieksekusi secara eksklusif menggunakan protokol transmisi tanpa koneksi, yakni **UDP**. Verifikasi temuan ini terlihat secara eksplisit pada hierarki *User Datagram Protocol* dalam panel detail Wireshark.

2. **Apa port tujuan pada pesan permintaan DNS? Apa port sumber pada pesan balasannya?**
   > - *Port* spesifik yang dituju (*Destination Port*) saat klien memancarkan permintaan DNS (teramati pada *Frame* 79) adalah layanan bernomor **53**.
   > - Berkebalikan dengan itu, *port* sumber (*Source Port*) yang memprakarsai pengiriman paket balasan dari pihak server (misalnya pada *Frame* 93 atau 81) juga identik di nomor **53**.

3. **Pada pesan permintaan DNS, apa alamat IP tujuannya? Apa alamat IP server DNS lokal Anda? Apakah kedua alamat IP tersebut sama?**
   > - Saat mengamati lalu lintas paket permintaan DNS (contoh di *Frame* 79), IP yang menjadi tujuannya tercatat pada rentang **10.55.39.218**.
   > - Merujuk pada diagnostik `ipconfig`, telah dibuktikan bahwa server DNS lokal *default* yang terpasang di jaringan juga memiliki IP **10.55.39.218**.
   > - Dengan demikian, **Ya**, kedua identitas numerik IP tersebut sama. Komputer *host* yang bernomor IP asal **10.55.39.11** menyalurkan *query* resolusinya langsung kepada perangkat DNS server lokal tersebut.

4. **Apa “jenis” atau ”type” dari pesan tersebut? Apakah pesan permintaan tersebut mengandung ”jawaban” atau ”answers”?**
   > - Tinjauan paket menunjukkan pesan permintaan bervariasi; *Frame* 79 dikategorikan sebagai *type* **AAAA** (klausul pencarian record IPv6), sedangkan *Frame* 80 bertindak sebagai *type* **A** (klausul pencarian record IPv4 reguler).
   > - Secara fungsional, paket transmisi awal (*query*) ini didesain mutlak **tanpa mengandung blok jawaban**. Status ini tervalidasi oleh nilai *Answer RRs: 0* pada inspektor paket.

5. **Berapa banyak ”jawaban” atau ”answers” yang terdapat di dalamnya? Apa saja isi yang terkandung dalam setiap jawaban tersebut?**
   > - Pada segmentasi paket balasan untuk *Type* A (*Frame* 93), deteksi mengonfirmasi adanya keberadaan **2 blok jawaban** (*Answer RRs: 2*). Data mentah (*payload*) balasan tersebut memuat pemetaan sepasang IP server aktual: **104.16.45.99** serta **104.16.44.99**.
   > - Kondisi serupa juga teramati pada paket balasan *Type* AAAA (*Frame* 81), yang secara konsisten mengembalikan **2 blok jawaban** berisi struktur jaringan IPv6.

6. **Apakah alamat IP pada paket TCP SYN sesuai dengan alamat IP yang tertera pada pesan balasan DNS?**
   > **Ya**. Protokol sesi inisiasi TCP SYN secara presisi mencantumkan dan memanggil alamat IP *destination* yang sejalan dengan referensi yang sebelumnya diberikan melalui siklus balasan resolusi DNS (sebagai contoh IP 104.16.45.99).

7. **Apakah host Anda perlu mengirimkan pesan permintaan DNS baru setiap kali ingin mengakses suatu gambar?**
   > **Tidak diperlukan pengiriman permintaan berulang**. Selama seluruh sumber daya grafis (gambar) berada di *root domain* yang sama, alamat IP tujuan akan diambil langsung dari *local cache storage* OS klien, bergantung penuh pada validitas sisa waktu *Time To Live* (TTL) entri tersebut.

#### b. Tracing DNS Interaktif Menggunakan *nslookup*
![Tracing DNS dengan nslookup](assets/image/Screenshot%202026-04-14%20001649.png)

**Pertanyaan dan Evaluasi Analisis:**
1. **Apa port tujuan pada pesan permintaan DNS? Apa port sumber pada pesan balasan DNS?**
   > - Permintaan awal (*query*) diarahkan tepat ke nomor *port* spesifik, yaitu **53**, seperti tertera pada baris *Dst Port: 53* di panel detail (paket *Frame* 21).
   > - Balikan informasinya dikirimkan dengan identifikasi nomor *Source Port* yang sama, yakni **53** (paket *Frame* 20 atau 22).

2. **Ke alamat IP manakah pesan permintaan DNS dikirimkan? Apakah alamat IP tersebut merupakan default alamat IP server DNS lokal Anda?**
   > Proses *routing* DNS terdeteksi menyalurkan *query* ke arah target alamat ber-IP **10.243.77.244**. Alamat spesifik ini merupakan IP operasional utama dari infrastruktur *local server* jaringan pada saat itu.

3. **Periksa pesan permintaan DNS. Apa "jenis" atau "type" dari pesan tersebut? Apakah pesan tersebut mengandung "jawaban" atau "answers"?**
   > - Karakteristik *query* pada blok pesan transmisi ini dikategorikan di bawah *Type* **AAAA** pada Frame 21 (mengacu pada pencarian parameter IPv6) dan *Type* **A** pada Frame 19 (pencarian IPv4).
   > - Seperti protokol standarnya, paket inisiasi ini murni bersifat permintaan dan **tidak melampirkan komponen jawaban** (*Answer RRs: 0*).

4. **Periksa pesan balasan DNS. Berapa banyak "jawaban" atau "answers" yang terdapat di dalamnya? Apa saja isi yang terkandung dalam setiap jawaban tersebut?**
   > Observasi terhadap respons *Frame* 20 membuktikan eksistensi dari **3 blok jawaban utuh**. Kandungan muatan dari ketiga respons ini dijabarkan berurutan:
   > - **Respons ke-1 (CNAME):** Menetapkan indikator alias, di mana struktur alamat *www.mit.edu* secara kanonis dilimpahkan pemetaannya ke alamat sekunder *www.mit.edu.edgekey.net*.
   > - **Respons ke-2 (CNAME):** Melakukan perpanjangan rantai rujukan alias dari titik *www.mit.edu.edgekey.net* menuju alamat server akhir bernama *e9566.dscb.akamaiedge.net*.
   > - **Respons ke-3 (*Type* A):** Mengekstrak dan menyuplai alamat fisik IP definitif `23.217.163.122` sebagai destinasi pamungkas untuk dihubungi oleh klien jaringan.

#### c. Resolusi Manual ke DNS Server Spesifik
![Query ke DNS Server Spesifik](assets/image/Screenshot%202026-04-14%20001849.png)

**Pertanyaan dan Evaluasi Analisis:**
1. **Ke alamat IP manakah pesan permintaan DNS dikirimkan? Apakah alamat IP tersebut merupakan default alamat IP server DNS lokal Anda?**
   > - Lintasan pemantauan mencatat pengiriman paket permintaan resolusi (Frame 96) membidik blok *IP Address* **8.8.8.8**.
   > - Angka identifikasi alamat ini bertentangan dengan profil DNS *default* lokal; melainkan, IP ini dikenal publik sebagai identitas dari armada server rekursif *Google Public DNS*. Skenario jaringan ini tercipta dikarenakan parameter baris perintah telah sengaja mengesampingkan server internal lewat eksekusi manual format `nslookup www.aiit.or.kr 8.8.8.8`.

2. **Periksa pesan permintaan DNS. Apa "jenis" atau "type" dari pesan tersebut? Apakah pesan tersebut mengandung "jawaban" atau "answers"?**
   > - Berdasarkan konstruksi *header* paketnya pada Frame 96, format *query* tercatat sebagai tipe data pencarian **A** (*Host Address*).
   > - Paket keberangkatan ini **tidak berisi resolusi jawaban** apa pun di dalamnya (*Answer RRs: 0*).

3. **Periksa pesan balasan DNS. Berapa banyak "jawaban" atau "answers" yang terdapat di dalamnya? Apa saja isi yang terkandung dalam setiap jawaban tersebut?**
   > Meninjau pada struktur paket umpan balik (*Frame* 98), terkonfirmasi jumlah penemuan **2 hasil jawaban independen** untuk *Type* A. Perincian data dari tanggapan tersebut adalah:
   > - Baris balasan pertama secara eksplisit merujuk ke rute pengalamatan IPv4 **172.67.152.120**.
   > - Baris balasan kedua menunjuk pada cadangan rute IPv4 yakni **104.21.74.8**.

---

## D. Kesimpulan
Sebagai landasan sistem penamaan, modul ini menguraikan esensi operasional *Domain Name System* (DNS) sebagai arsitektur *resolver* krusial yang menerjemahkan bahasa domain tekstual ke format alamat IP numerik. Adopsi perintah konsol *nslookup* memberdayakan kapabilitas *host* klien guna memformulasikan ragam *query* khusus atas blok-blok data DNS, termasuk tipe A, hierarki NS, serta prioritas rute email MX. Secara paralel, utilitas konfigurasi bawaan *ipconfig* dimaksimalkan untuk membedah topologi konfigurasi *network layer* dan mengelola sirkulasi *database cache* milik entitas lokal. Terakhir, pelacakan empiris lewat sistem perangkat lunak *Wireshark* menghasilkan bukti visual definitif mengenai alur transfer informasi protokol DNS yang mentransmisikan *payload*-nya secara efisien melalui pondasi *datagram* UDP, mendokumentasikan proses utuh dari keberangkatan parameter *query* inisial, sampai dengan penerimaan balik blok-blok data penyelesaian resolusi rekursif berbasis alias sistem seperti CNAME.