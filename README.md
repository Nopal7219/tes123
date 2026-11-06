# Jawaban nomor 2

Pengujian dijalankan ulang pada proyek **Praktikum_3**, prefix **10.83**. Ikon dan tata letak awal tidak diubah.

## Pelaksanaan a–f

**a. RhodesIsland:** eth1 dan eth2 masing-masing menggunakan loss 1%, delay 100 ms, dan rate 2 Mbps. Konfigurasi aktual TBF + netem tersimpan pada `qdisc.txt`.

```sh
for dev in eth1 eth2; do
  tc qdisc replace dev "$dev" root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
  tc qdisc replace dev "$dev" parent 1:1 handle 10: netem delay 100ms loss 1%
done
```

**b. Laterano:** TCP congestion control diatur ke Cubic sebelum tiga pengujian pertama. Nilai algoritma yang benar-benar digunakan socket diperiksa melalui TCP_CONGESTION.

```sh
sysctl -w net.ipv4.tcp_congestion_control=cubic
```

**c. Laterano:** tersedia `tes1000.bin` (1.000 byte), `tes10000.bin` (10.000 byte), dan `tes5M.bin` (5.000.000 byte) di `/root/myweb`. Python HTTP server tersedia pada `http://10.83.4.15:9999/`. Server memakai pustaka standar `http.server` dengan pengambilan sampel TCP_INFO untuk mencatat cwnd socket.

**d. Columbia:** setiap berkas diunduh melalui koneksi HTTP baru, dengan waktu dan ukuran diukur oleh curl. Berkas unduhan diverifikasi menggunakan SHA-256.

```sh
curl --fail --max-time 180 -s -o /root/received.bin \
  -w 'size=%{size_download} speed=%{speed_download} total=%{time_total} http=%{http_code}\n' \
  http://10.83.4.15:9999/tes5M.bin
sha256sum /root/received.bin
```

**e. Laterano dan Columbia:** algoritma diubah ke Reno, kemudian tiga unduhan yang sama diulangi memakai koneksi baru.

```sh
sysctl -w net.ipv4.tcp_congestion_control=reno
```

**f. Capture:** keenam kasus masing-masing mempunyai satu capture pada eth0 Columbia dan satu pada eth0 Laterano. Total **12 berkas PCAP** ada di folder `captures`. Nama berkas menyebut algoritma, ukuran berkas, dan titik capture. Seluruh enam transfer selesai dengan **HTTP 200, ukuran tepat, dan checksum cocok**.

## Tabel hasil aktual

| Berkas | Algoritma | RTT rata-rata (ms) | Retransmisi | Throughput (kB/s) | Waktu transfer (s) | Pola cwnd |
|---|---|---:|---:|---:|---:|---|
| tes1000.bin | Reno | 207.81 | 0 | 1.60 | 0.624 | cwnd 10–10 segmen. Transfer pendek; pola avoidance belum jelas. |
| tes1000.bin | Cubic | 201.91 | 0 | 1.63 | 0.615 | cwnd 10–10 segmen. Transfer pendek; pola avoidance belum jelas. |
| tes10000.bin | Reno | 206.94 | 0 | 16.52 | 0.605 | cwnd 10–14 segmen. Transfer pendek; pola avoidance belum jelas. |
| tes10000.bin | Cubic | 206.99 | 0 | 16.54 | 0.605 | cwnd 10–14 segmen. Transfer pendek; pola avoidance belum jelas. |
| tes5M.bin | Reno | 208.80 | 32 | 93.48 | 53.487 | cwnd 2–72 segmen. Kenaikan bertahap dan penurunan tajam berulang (sawtooth). |
| tes5M.bin | Cubic | 204.99 | 24 | 116.34 | 42.977 | cwnd 6–32 segmen. Pertumbuhan melengkung, plateau, dan penurunan saat recovery. |

RTT dihitung dari mean `tcp.analysis.ack_rtt` pada capture Laterano untuk ACK Columbia yang mengonfirmasi frame data server. Handshake dan FIN dikecualikan. Retransmisi adalah union frame server bertanda retransmission, fast_retransmission, atau spurious_retransmission; satu frame dihitung satu kali. Throughput di tabel adalah goodput aplikasi: ukuran byte berkas / waktu total curl / 1.000. Waktu curl mencakup setup koneksi. `summary.json` dan CSV memuat jumlah sampel RTT serta rincian retransmisi data.

## Grafik

![Time–Sequence Stevens](grafik/time-sequence.png)

![cwnd aktual TCP_INFO](grafik/cwnd.png)

Grafik Time–Sequence menunjukkan kemajuan sequence number paket terhadap waktu. Cwnd tidak dikirim di header TCP; grafik cwnd tambahan berasal dari TCP_INFO socket server, dengan interval target 20 ms.

## Kesimpulan

Pada berkas 1.000 B dan 10.000 B, kedua algoritma menyelesaikan transfer dalam waktu yang berdekatan. Ukuran tersebut masih dekat initial window sehingga biaya handshake, request, dan RTT lebih dominan daripada perbedaan congestion avoidance. Grafik belum membentuk pola sawtooth atau kubik yang jelas.

Pada 5 MB, Reno memerlukan **53.487 s**, goodput **93.48 kB/s**, dan **32 retransmisi**. Cubic memerlukan **42.977 s**, goodput **116.34 kB/s**, dan **24 retransmisi**. Dalam run ini **Cubic selesai 19.6% lebih singkat**. RTT rata-rata Reno adalah 208.80 ms dan Cubic 204.99 ms. Throughput yang lebih tinggi tidak harus disertai RTT lebih rendah karena antrean dapat bertambah saat pengiriman lebih agresif.

Transfer besar melewati lebih banyak RTT dan kejadian loss, sehingga cara algoritma memulihkan cwnd berpengaruh lebih lama. Reno memperlihatkan pertumbuhan bertahap disertai penurunan tajam; Cubic memperlihatkan pertumbuhan melengkung dan periode relatif datar di antara pemulihan. Bentuk aktual juga dipengaruhi slow start dan recovery kernel. Hasil satu run dengan random loss ini tidak menetapkan pemenang universal untuk semua kondisi jaringan.

Delay 100 ms diterapkan pada masing-masing arah egress yang dilalui data dan ACK, sehingga RTT dasar mendekati 200 ms. Batas 2 Mbps setara 250 kB/s sebelum overhead; goodput aktual lebih rendah karena RTT, loss, retransmisi, dan perilaku cwnd.

Dasar teori: [RFC 5681](https://www.rfc-editor.org/rfc/rfc5681.html) dan [RFC 9438](https://datatracker.ietf.org/doc/html/rfc9438).
