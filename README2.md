# Panduan lengkap menjalankan nomor 2

**Praktikum Modul 3 — TCP Reno dan Cubic, prefix 10.83**

Panduan ini dipakai pada proyek **Praktikum_3** yang sudah diperbaiki. Semua perintah Linux dijalankan di konsol node yang disebutkan, bukan di PowerShell Windows. Ikon PC dan tata letak awal tetap dipertahankan.

## 1. Tujuan dan hasil yang harus dikumpulkan

Anda akan mengunduh tiga berkas dari Laterano menggunakan Columbia. Jalankan sekali dengan Cubic, kemudian ulangi dengan Reno. RhodesIsland memberi gangguan yang sama untuk seluruh pengujian.

| Komponen | Ketentuan |
|---|---|
| Gangguan pada RhodesIsland eth1 dan eth2 | Loss 1%, delay 100 ms, rate 2 Mbps |
| Web server | Laterano, 10.83.4.15, port 9999 |
| Client | Columbia, DHCP dari Victoria; pada proyek ini 10.83.3.100 |
| Berkas | tes1000.bin = 1.000 B; tes10000.bin = 10.000 B; tes5M.bin = 5.000.000 B |
| Algoritma pengirim | Cubic dan Reno, diatur pada Laterano |
| Capture tiap pengujian | eth0 Columbia dan eth0 Laterano |
| Jumlah pengujian | 3 berkas × 2 algoritma = 6 |
| Jumlah capture | 6 pengujian × 2 titik = 12 |

Simpan waktu transfer, ukuran yang diterima, throughput, RTT, retransmisi, dan penjelasan pola grafik. Jangan mengisi angka dari perkiraan atau menyalin angka acuan seolah-olah hasil pengujian baru.

## 2. Membuka dan menyalakan proyek

1. Nyalakan **GNS3 VM** melalui VMware Workstation. Tunggu sampai VM selesai boot.
2. Buka GNS3. Gunakan server yang sudah dikonfigurasi, yaitu **192.168.153.128:80**. Alamat server ini berbeda dari alamat node lab 10.83.x.x.
3. Buka proyek **Praktikum_3** yang ada. Bila memakai hasil ekspor, impor berkas `Praktikum-Modul3-10.83.gns3project` terlebih dahulu melalui menu impor proyek portable. Jangan membuat topologi kosong baru untuk menjalankan panduan ini.
4. Klik tombol hijau **Start/Resume all nodes**.
5. Tunggu sekitar 30–60 detik. Pada impor pertama di VM lain, pemasangan paket dapat membutuhkan waktu lebih lama dan akses internet.
6. Pastikan semua node berstatus berjalan. Node hijau menunjukkan prosesnya berjalan; koneksi jaringan tetap perlu dicek dengan langkah berikut.

Proyek ini sudah mempunyai startup IP, routing, DHCP, dan HTTP. Anda tidak perlu mengetik ulang seluruh konfigurasi jaringan setiap kali membuka proyek.

## 3. Membuka tiga konsol kerja

Siapkan tiga jendela agar tidak tertukar:

| Jendela | Cara membuka | Dipakai untuk |
|---|---|---|
| RhodesIsland | Klik kanan node → Console | Memasang dan memeriksa gangguan jaringan |
| Laterano | Klik kanan node → Console | Memeriksa server dan mengganti Cubic/Reno |
| Columbia | Klik kanan node → Console untuk desktop, lalu buka terminal/XTerm di desktop | Menjalankan curl |

Columbia mempunyai desktop melalui VNC. Jangan mengetik perintah Linux pada kolom alamat browser. Gunakan terminal di dalam desktop Columbia. Jika terminal tidak memakai root, gunakan `sudo -i` terlebih dahulu bila tersedia dan diperlukan.

## 4. Memastikan IP dan koneksi benar

### 4.1 Di Laterano

```sh
ip addr show eth0
ip route
```

Harus ada alamat **10.83.4.15/24** dan default route melalui **10.83.4.1**.

### 4.2 Di Columbia

```sh
ip addr show eth0
ip route
ping -c 3 10.83.4.15
```

Harus ada alamat **10.83.3.100/24** atau alamat valid lain dari pool Victoria **10.83.3.100–10.83.3.120**, serta default route melalui **10.83.3.1**. Pada proyek dan pengujian yang sudah dibuat, alamatnya 10.83.3.100.

Ping harus memperoleh balasan. Karena skenario memakai loss 1%, satu paket ping yang hilang tidak otomatis berarti routing rusak. Bila seluruh paket hilang atau muncul `Network is unreachable`, selesaikan masalah koneksi sebelum menguji transfer.

### 4.3 Periksa HTTP dari Columbia

```sh
curl --fail --max-time 10 -I http://10.83.4.15:9999/tes1000.bin
```

Hasil yang diharapkan: status **HTTP 200 OK** dan `Content-Length: 1000`. Jangan mulai enam pengujian jika server belum dapat diakses.

## 5. Memeriksa atau membuat tiga berkas uji — Laterano

Periksa dahulu:

```sh
ls -l /root/myweb/tes1000.bin /root/myweb/tes10000.bin /root/myweb/tes5M.bin
wc -c /root/myweb/tes1000.bin /root/myweb/tes10000.bin /root/myweb/tes5M.bin
```

Ukuran masing-masing harus tepat **1000**, **10000**, dan **5000000** byte. Angka 5 MB pada panduan ini memakai satuan desimal, bukan 5 MiB.

Bila berkas belum tersedia atau ukurannya salah, buat ulang di **Laterano**:

```sh
mkdir -p /root/myweb
python3 -c "from pathlib import Path; p=Path('/root/myweb'); (p/'tes1000.bin').write_bytes(bytes(1000)); (p/'tes10000.bin').write_bytes(bytes(10000)); (p/'tes5M.bin').write_bytes(bytes(5000000))"
wc -c /root/myweb/*.bin
sha256sum /root/myweb/tes1000.bin /root/myweb/tes10000.bin /root/myweb/tes5M.bin
```

Berkas tersebut berisi byte nol. HTTP server tidak mengompres respons, sehingga seluruh byte tetap dikirim melalui TCP.

## 6. Memastikan Python HTTP server berjalan — Laterano

Pada proyek hasil perbaikan, server sudah dimulai otomatis dari startup. Server menggunakan pustaka standar Python `http.server` dan menambahkan pencatatan cwnd socket.

Periksa:

```sh
ps aux | grep '[p]ython3'
tail -n 10 /root/http.log
```

Jika pemeriksaan HTTP pada langkah 4.3 sudah menghasilkan 200, gunakan server yang sedang berjalan. **Jangan menjalankan server kedua pada port 9999.**

Bila server tidak berjalan, mulai server yang sudah disediakan:

```sh
nohup python3 /root/http_lab.py > /root/http.log 2>&1 < /dev/null &
```

Kemudian ulangi pemeriksaan HTTP dari Columbia.

**Alternatif untuk server standar tanpa pencatatan cwnd:** bila skrip `/root/http_lab.py` tidak tersedia dan belum ada layanan pada port 9999, jalankan:

```sh
python3 -m http.server 9999 --directory /root/myweb
```

Biarkan terminal tersebut terbuka. Gunakan konsol Laterano kedua untuk mengganti algoritma. Capture Wireshark dan pengukuran curl tetap dapat dilakukan dengan server standar ini. Grafik cwnd langsung dari skrip tambahan tidak akan dibuat oleh server standar.

## 7. Memasang loss, delay, dan rate — RhodesIsland

Jalankan hanya pada **RhodesIsland**:

```sh
for dev in eth1 eth2; do
  tc qdisc replace dev "$dev" root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
  tc qdisc replace dev "$dev" parent 1:1 handle 10: netem delay 100ms loss 1%
done
tc -s qdisc show dev eth1
tc -s qdisc show dev eth2
```

Pada kedua interface harus terlihat:

- TBF dengan **rate 2Mbit**.
- Netem dengan **delay 100ms** dan **loss 1%**.

Gunakan konfigurasi yang sama untuk seluruh enam run. Jangan mengganti nilainya ketika berpindah dari Cubic ke Reno. Jangan memasang aturan tambahan pada Columbia atau Laterano.

Data dari server dan ACK dari client melewati arah egress yang berbeda pada RhodesIsland. Delay dasar round-trip mendekati **200 ms**. Parameter `latency 100ms` milik TBF adalah parameter antrean, bukan tambahan delay tetap 100 ms.

## 8. Menyiapkan folder hasil di komputer Windows

Buat satu folder, misalnya `Hasil-Nomor-2`. Di dalamnya buat:

```text
Hasil-Nomor-2/
  captures/
  screenshot/
  catatan-hasil.txt
```

Gunakan nama capture berikut agar pasangan kasus tidak tertukar:

| Run | Algoritma | Berkas | Capture Columbia | Capture Laterano |
|---|---|---|---|---|
| 1 | Cubic | tes1000.bin | cubic-tes1000-Columbia.pcapng | cubic-tes1000-Laterano.pcapng |
| 2 | Cubic | tes10000.bin | cubic-tes10000-Columbia.pcapng | cubic-tes10000-Laterano.pcapng |
| 3 | Cubic | tes5M.bin | cubic-tes5M-Columbia.pcapng | cubic-tes5M-Laterano.pcapng |
| 4 | Reno | tes1000.bin | reno-tes1000-Columbia.pcapng | reno-tes1000-Laterano.pcapng |
| 5 | Reno | tes10000.bin | reno-tes10000-Columbia.pcapng | reno-tes10000-Laterano.pcapng |
| 6 | Reno | tes5M.bin | reno-tes5M-Columbia.pcapng | reno-tes5M-Laterano.pcapng |

Format `.pcap` dari GNS3 juga boleh. Ekstensi `.pcapng` di atas sesuai penyimpanan standar Wireshark. Jangan sekadar mengganti ekstensi untuk mengonversi format.

## 9. Cara capture untuk SETIAP run

Lakukan urutan ini enam kali, satu kali untuk setiap baris tabel run.

1. Di GNS3, klik kanan **link Columbia–Switch1**, lalu pilih **Start capture**. Link ini terhubung ke **eth0 Columbia**.
2. Klik kanan **link Laterano–Hub1**, lalu pilih **Start capture**. Link ini terhubung ke **eth0 Laterano**.
3. Jika diminta memilih interface/titik capture, pastikan link dan interface tersebut benar.
4. Buka capture di Wireshark. Biasanya GNS3 membuka Wireshark otomatis sesuai pengaturan lokal. Bila muncul pemilihan aplikasi, pilih Wireshark.
5. Pastikan kedua capture sudah aktif sebelum menjalankan curl.
6. Jalankan **satu** perintah unduh sesuai run pada langkah 10 atau 11.
7. Tunggu sampai curl selesai dan prompt terminal kembali. Untuk 5 MB, tunggu prosesnya selesai; jangan menghentikannya hanya karena berjalan puluhan detik.
8. Tunggu lagi sekitar **2 detik** agar paket penutupan koneksi ikut tertangkap.
9. Di GNS3, klik kanan kedua link tadi dan pilih **Stop capture** untuk masing-masing link.
10. Simpan setiap capture dari Wireshark melalui **File → Save As**, memakai nama pada tabel run dan folder `captures`.
11. Pastikan ada dua berkas untuk run tersebut. Mulai capture baru untuk run berikutnya; jangan menumpuk keenam run ke satu capture.

Jika GNS3 masih memakai capture sebelumnya, hentikan capture lama sebelum membuat capture baru. Nama menu dapat sedikit berbeda menurut bahasa dan versi aplikasi, tetapi link yang ditangkap tetap dua link yang disebutkan di atas.

## 10. Tiga run Cubic

### 10.1 Atur algoritma pada Laterano

```sh
sysctl -w net.ipv4.tcp_congestion_control=cubic
cat /proc/sys/net/ipv4/tcp_congestion_control
```

Output pemeriksaan harus **cubic**. Algoritma diatur pada Laterano karena node inilah pengirim data berkas. Gunakan koneksi curl baru untuk setiap unduhan; tidak perlu mengganti algoritma pada Columbia.

### 10.2 Run 1 — Cubic, 1.000 byte

Mulai dua capture terlebih dahulu sesuai langkah 9. Lalu jalankan di **Columbia**:

```sh
mkdir -p /root/hasil-nomor2
curl --fail --max-time 180 -sS -o /root/hasil-nomor2/cubic-tes1000.bin -w 'HTTP=%{http_code} BYTES=%{size_download} SPEED_Bps=%{speed_download} TIME_s=%{time_total}\n' http://10.83.4.15:9999/tes1000.bin
sha256sum /root/hasil-nomor2/cubic-tes1000.bin
```

Catat output. Hentikan dan simpan kedua capture dengan nama **cubic-tes1000**.

### 10.3 Run 2 — Cubic, 10.000 byte

Mulai dua capture baru, kemudian di **Columbia**:

```sh
curl --fail --max-time 180 -sS -o /root/hasil-nomor2/cubic-tes10000.bin -w 'HTTP=%{http_code} BYTES=%{size_download} SPEED_Bps=%{speed_download} TIME_s=%{time_total}\n' http://10.83.4.15:9999/tes10000.bin
sha256sum /root/hasil-nomor2/cubic-tes10000.bin
```

Hentikan dan simpan kedua capture dengan nama **cubic-tes10000**.

### 10.4 Run 3 — Cubic, 5 MB

Mulai dua capture baru, kemudian di **Columbia**:

```sh
curl --fail --max-time 180 -sS -o /root/hasil-nomor2/cubic-tes5M.bin -w 'HTTP=%{http_code} BYTES=%{size_download} SPEED_Bps=%{speed_download} TIME_s=%{time_total}\n' http://10.83.4.15:9999/tes5M.bin
sha256sum /root/hasil-nomor2/cubic-tes5M.bin
```

Hentikan dan simpan kedua capture dengan nama **cubic-tes5M**.

## 11. Tiga run Reno

### 11.1 Ganti algoritma pada Laterano

```sh
sysctl -w net.ipv4.tcp_congestion_control=reno
cat /proc/sys/net/ipv4/tcp_congestion_control
```

Output harus **reno**. Perubahan berlaku pada koneksi baru. Semua curl berikut membuat koneksi baru. Konfigurasi loss/delay/rate tetap sama.

### 11.2 Run 4 — Reno, 1.000 byte

Mulai dua capture baru, kemudian di **Columbia**:

```sh
curl --fail --max-time 180 -sS -o /root/hasil-nomor2/reno-tes1000.bin -w 'HTTP=%{http_code} BYTES=%{size_download} SPEED_Bps=%{speed_download} TIME_s=%{time_total}\n' http://10.83.4.15:9999/tes1000.bin
sha256sum /root/hasil-nomor2/reno-tes1000.bin
```

Hentikan dan simpan kedua capture dengan nama **reno-tes1000**.

### 11.3 Run 5 — Reno, 10.000 byte

Mulai dua capture baru, kemudian di **Columbia**:

```sh
curl --fail --max-time 180 -sS -o /root/hasil-nomor2/reno-tes10000.bin -w 'HTTP=%{http_code} BYTES=%{size_download} SPEED_Bps=%{speed_download} TIME_s=%{time_total}\n' http://10.83.4.15:9999/tes10000.bin
sha256sum /root/hasil-nomor2/reno-tes10000.bin
```

Hentikan dan simpan kedua capture dengan nama **reno-tes10000**.

### 11.4 Run 6 — Reno, 5 MB

Mulai dua capture baru, kemudian di **Columbia**:

```sh
curl --fail --max-time 180 -sS -o /root/hasil-nomor2/reno-tes5M.bin -w 'HTTP=%{http_code} BYTES=%{size_download} SPEED_Bps=%{speed_download} TIME_s=%{time_total}\n' http://10.83.4.15:9999/tes5M.bin
sha256sum /root/hasil-nomor2/reno-tes5M.bin
```

Hentikan dan simpan kedua capture dengan nama **reno-tes5M**.

## 12. Membaca output curl dan memeriksa checksum

Contoh bentuk output:

```text
HTTP=200 BYTES=5000000 SPEED_Bps=116341 TIME_s=42.976749
```

- **HTTP=200:** server berhasil mengirim berkas.
- **BYTES:** harus sama dengan ukuran berkas yang diuji.
- **SPEED_Bps:** kecepatan rata-rata dari curl dalam byte per detik, bukan bit per detik.
- **TIME_s:** waktu total transfer dalam detik, termasuk setup koneksi.
- **Throughput tabel (kB/s):** `BYTES / TIME_s / 1000`. Ini goodput aplikasi, bukan total trafik jaringan beserta seluruh header dan retransmisi.

Jika memakai berkas byte nol yang dibuat pada langkah 5, checksum yang benar adalah:

| Berkas | SHA-256 |
|---|---|
| tes1000.bin | 541b3e9daa09b20bf85fa273e5cbd3e80185aa4ec298e765db87742b70138a53 |
| tes10000.bin | 95b532cc4381affdff0d956e12520a04129ed49d37e154228368fe5621f0b9a2 |
| tes5M.bin | b39781589c4403fb82174c9647a010464cff38bad976547d339899b00053a545 |

Jika isi berkas sengaja dibuat berbeda, bandingkan checksum Columbia dengan checksum berkas yang bersangkutan di Laterano, bukan dengan tabel byte nol di atas.

## 13. Membuka capture dan grafik di Wireshark

### 13.1 Memilih koneksi yang benar

1. Buka capture **Laterano** untuk satu run.
2. Isi display filter:

```text
tcp.port == 9999
```

3. Pilih paket respons dari **10.83.4.15** menuju Columbia.
4. Bila ada beberapa koneksi, lihat nilai **TCP Stream index** pada detail TCP, lalu filter koneksi yang dipilih dengan:

```text
tcp.stream == 0
```

Ganti angka 0 dengan stream index yang benar. Jangan berasumsi semua capture selalu memakai stream 0.

### 13.2 Time–Sequence (Stevens)

1. Dengan paket server pada koneksi uji dipilih, buka **Statistics → TCP Stream Graphs → Time Sequence (Stevens)**.
2. Pastikan arah grafik adalah **Laterano → Columbia**.
3. Amati kemajuan sequence number terhadap waktu. Bagian mendatar menunjukkan pengiriman tidak maju selama interval itu; perubahan kemiringan menunjukkan perubahan laju pengiriman.
4. Simpan screenshot grafik dengan nama run, misalnya `reno-tes5M-Stevens.png`.
5. Ulangi untuk masing-masing run yang ingin dibandingkan.

**Penting:** grafik Stevens bukan pengukuran cwnd langsung. Cwnd adalah state internal pengirim. Pada proyek ini, skrip server juga mencatatnya melalui TCP_INFO. Jangan menyamakan nilai advertised receive window pada header TCP dengan cwnd.

### 13.3 Retransmisi

Gunakan filter berikut pada capture Laterano. Tambahkan filter `tcp.stream == N` jika capture mempunyai lebih dari satu koneksi uji.

```text
tcp.srcport == 9999 && (tcp.analysis.retransmission || tcp.analysis.fast_retransmission || tcp.analysis.spurious_retransmission)
```

Catat jumlah **Displayed packets** di status Wireshark. OR pada filter membuat satu frame yang memenuhi lebih dari satu kondisi tetap dihitung satu kali. Definisi ini mencakup retransmisi control frame.

Jika ingin hanya retransmisi paket yang membawa data, tambahkan:

```text
tcp.len > 0
```

Gunakan definisi yang konsisten untuk keenam baris tabel, dan tuliskan definisinya pada laporan.

### 13.4 RTT

Untuk melihat sampel ACK RTT dari Columbia di capture Laterano:

```text
ip.src == 10.83.3.100 && tcp.port == 9999 && tcp.analysis.ack_rtt
```

Jika IP Columbia berbeda, ganti alamat pada filter. Pilih paket ACK, buka detail **Transmission Control Protocol → SEQ/ACK analysis**, lalu lihat waktu ACK yang tersedia. Nilai `tcp.analysis.ack_rtt` memakai satuan detik; kalikan 1.000 untuk milidetik.

Untuk mengikuti metode hasil yang sudah dibuat, rata-ratakan hanya ACK yang mengonfirmasi **frame data Laterano**, bukan SYN atau FIN. Field `tcp.analysis.acks_frame` menunjukkan frame yang di-ACK; periksa bahwa frame rujukannya memiliki `tcp.len > 0` dan berasal dari Laterano. Filter di atas adalah daftar kandidat, bukan otomatis seluruh sampel yang harus dimasukkan. Berkas kecil hanya memiliki sedikit sampel, sehingga satu kejadian dapat sangat memengaruhi rata-rata.

Jangan memakai hasil ping sebagai pengganti RTT TCP dari capture, karena soal meminta pengukuran berdasarkan capture Laterano.

## 14. Tabel acuan dari pengujian yang sudah dijalankan

Angka di bawah adalah hasil aktual run sebelumnya pada proyek ini. Hasil run baru dapat berbeda karena loss bersifat acak. Masukkan hasil baru Anda jika mengulang pengujian.

| Berkas | Algoritma | RTT rata-rata (ms) | Retransmisi | Throughput (kB/s) | Waktu (s) |
|---|---|---:|---:|---:|---:|
| tes1000.bin | Reno | 207,81 | 0 | 1,60 | 0,624 |
| tes1000.bin | Cubic | 201,91 | 0 | 1,63 | 0,615 |
| tes10000.bin | Reno | 206,94 | 0 | 16,52 | 0,605 |
| tes10000.bin | Cubic | 206,99 | 0 | 16,54 | 0,605 |
| tes5M.bin | Reno | 208,80 | 32 | 93,48 | 53,487 |
| tes5M.bin | Cubic | 204,99 | 24 | 116,34 | 42,977 |

Pada run tersebut seluruh enam unduhan menghasilkan HTTP 200, ukuran tepat, dan checksum cocok.

## 15. Cara menjelaskan hasil saat demo

1. Tunjukkan `tc -s qdisc` di RhodesIsland untuk membuktikan loss 1%, delay 100 ms, dan rate 2 Mbps ada pada kedua interface.
2. Sebelum tiap kelompok run, tunjukkan hasil pemeriksaan algoritma di Laterano: pertama Cubic, kemudian Reno.
3. Tunjukkan ukuran tiga berkas dan alamat HTTP server.
4. Jalankan satu unduhan dari Columbia dengan kedua capture sudah aktif.
5. Tunjukkan HTTP 200, jumlah byte, waktu, throughput, dan kecocokan checksum.
6. Buka capture Laterano dan grafik Stevens yang sesuai. Cocokkan dengan nama run agar data tidak tertukar.
7. Jelaskan bahwa transfer kecil masih didominasi overhead setup dan RTT, sehingga perbedaan algoritma belum terlihat jelas.
8. Jelaskan bahwa transfer besar melewati banyak RTT dan kejadian loss, sehingga pertumbuhan serta pemulihan cwnd lebih memengaruhi waktu total.

Pada hasil acuan, Cubic untuk 5 MB lebih cepat daripada Reno. Jangan menyimpulkan Cubic selalu menang pada semua jaringan. Satu run dengan random loss hanya menunjukkan hasil pada kondisi pengujian tersebut.

## 16. Mengatasi masalah yang sering muncul

| Gejala | Pemeriksaan dan tindakan |
|---|---|
| Columbia tidak memiliki IPv4 | Tunggu startup DHCP. Periksa Victoria dan koneksi Columbia–Switch1. Di konsol Victoria, lihat `tail -n 20 /root/kea.log`. |
| Laterano bukan 10.83.4.15 | Periksa Yan, log Kea, dan reservasi MAC Laterano pada `/root/kea-dhcp4.conf`. Jangan mengganti Laterano menjadi IP statis agar tampak berhasil; soal meminta client DHCP dengan reservasi. |
| `Network is unreachable` | Periksa IP dan default route Columbia. Jangan lanjutkan benchmark sebelum route tersedia. |
| `Connection refused` ke port 9999 | Periksa proses Python dan `/root/http.log` di Laterano. Jalankan hanya satu HTTP server pada port itu. |
| `Address already in use` | Sudah ada server di port 9999. Gunakan layanan yang sedang berjalan, jangan menumpuk server kedua. |
| HTTP 404 | Periksa nama berkas, huruf besar/kecil pada `tes5M.bin`, dan direktori `/root/myweb`. |
| `sysctl` gagal | Pastikan perintah dijalankan di Laterano dengan hak root, dan algoritma tersedia. Periksa `cat /proc/sys/net/ipv4/tcp_available_congestion_control`. |
| Capture tidak memuat transfer | Pastikan capture aktif sebelum curl, link yang dipilih benar, dan filter bukan milik stream lama. |
| Hanya ada satu capture per run | Ulangi run dengan capture pada kedua link, Columbia dan Laterano. |
| Tidak terlihat pola sawtooth pada 1.000 B | Hal itu wajar: transfer terlalu pendek untuk menunjukkan banyak siklus congestion avoidance. Jangan menggambar pola teoretis seolah-olah hasil aktual. |
| Waktu transfer baru berbeda dari tabel acuan | Loss acak dan kondisi VM dapat mengubah hasil. Catat hasil aktual baru; gunakan aturan dan metode yang sama. |
| Capture meminta aplikasi analisis yang keliru | Periksa pengaturan packet capture GNS3 dan pilih Wireshark sebagai aplikasi pembuka. Berkas capture tetap bisa dibuka manual di Wireshark. |

## 17. Setelah selesai

Sebelum menutup lab, pastikan sudah ada enam catatan hasil, 12 capture, dan grafik/penjelasan yang sesuai. Simpan proyek GNS3.

Untuk demo nomor 2, biarkan gangguan jaringan aktif sesuai soal. Pada proyek ini, restart juga menjalankan konfigurasi gangguan dari startup RhodesIsland.

Jika pengujian benar-benar sudah selesai dan Anda ingin mengembalikan jaringan normal untuk pekerjaan lain, jalankan **opsional** di RhodesIsland:

```sh
tc qdisc del dev eth1 root
tc qdisc del dev eth2 root
```

Jangan menjalankan pembersihan ini sebelum keenam pengujian dan pengambilan bukti selesai. Menghapus qdisc saat runtime tidak menghapus perintah startup, sehingga gangguan akan aktif kembali ketika node direstart.

## Ringkasan urutan kerja

**Nyalakan proyek → periksa IP dan HTTP → periksa ukuran berkas → pasang gangguan → set Cubic → tiga run dengan dua capture per run → set Reno → tiga run dengan dua capture per run → analisis capture Laterano → isi tabel → buat kesimpulan berdasarkan hasil aktual.**
