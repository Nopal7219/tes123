> [!NOTE]
> Jawaban diisi dari pengujian aktual dengan prefix **10.83**. Bukti yang sudah tersedia ditautkan langsung. Penanda **belum dilampirkan** menunjukkan screenshot yang masih perlu kamu ambil, bukan hasil yang sudah terbukti secara visual. Ikuti [Panduan Screenshot](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md) dan kirim hasilnya untuk diperiksa. Angka baru dari pengujian ulang harus mengganti angka lama beserta capture pasangannya.

| Name           | NRP        | Kelas     |
| --- | --- | --- |
| Muhammad Nawfal| 5025241185 | B         |

> [!IMPORTANT]
> Remember to use the IP prefix allocation provided earlier for this pre-lab assignment (pra-praktikan) and for future ones.
> Access it on [Linktree](https://linktr.ee/jarkom)

> [!TIP]
> **Due Date : `5th November 2026, 23.00 WIB`**

#### Put your GNS3 Project files here (gdrive or github upload)!

**Belum diunggah.** Salinan definisi topologi tersedia di [Pra_Praktikum3.gns3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Pra_Praktikum3.gns3). File ini bukan backup lengkap volume/image Docker; tautan Drive/GitHub perlu ditambahkan setelah proyek lengkap diunggah.

---

# Bagian A - Reliable Data Transfer pada Kondisi Normal

## A.3 - Persiapan Topologi
 
> Langkah kerja, topologi, dan persiapan konfigurasi network (Pre-Praktikum). Seluruh pengujian pada Bagian A - D menggunakan topologi yang sama.
>
> *Work steps, topology, and network configuration preparation (Pre-Practicum). All tests in Parts A - D use the same topology.*
 
#### Soal A.3.1 - Topologi GNS3
 
> Buat topologi sesuai gambar dan tabel pemetaan node.
>
> *Build the topology according to the figure and the node mapping table.*
 
**Answer:**

Topologi menggunakan NAT1, RhodesIsland, Lungmen, Ursus, Switch1, Hub1, Columbia, Victoria, Laterano, dan Yan. Prefix yang digunakan adalah **10.83**.

![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/topologi.jpg)

 
#### Soal A.3.2 - Konfigurasi IP / Network & Static Routing
 
> Tunjukkan konfigurasi `/etc/network/interfaces` tiap router (RhodesIsland, Lungmen, Ursus), command `ip route add` (dapat dikonfigurasikan otomatis melalui `init.sh`), hasil `ip route show` pada ketiga router, serta bukti ping antara Victoria dan Yan.
>
> *Show the `/etc/network/interfaces` config of each router (RhodesIsland, Lungmen, Ursus), the `ip route add` commands (can be automated through `init.sh`), the `ip route show` output on all three routers, and proof of ping between Victoria and Yan.*
 
**Answer:**

Ketiga router menggunakan IP statis pada jaringan internal dan IP forwarding. RhodesIsland eth0 mendapat DHCP dari NAT.

**RhodesIsland — /etc/network/interfaces**

```sh
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
    address 10.83.1.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet static
    address 10.83.2.1
    netmask 255.255.255.0
```

**/root/init.sh (routing dan startup)**

```sh
#!/bin/sh
ifup -a
sysctl -w net.ipv4.ip_forward=1
ip route replace 10.83.3.0/24 via 10.83.1.2 dev eth1
ip route replace 10.83.4.0/24 via 10.83.2.2 dev eth2
iptables -t nat -C POSTROUTING -s 10.83.0.0/16 -o eth0 -j MASQUERADE 2>/dev/null || iptables -t nat -A POSTROUTING -s 10.83.0.0/16 -o eth0 -j MASQUERADE
echo nameserver 8.8.8.8 > /etc/resolv.conf
```

**Lungmen — /etc/network/interfaces**

```sh
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 10.83.1.2
    netmask 255.255.255.0
    gateway 10.83.1.1

auto eth1
iface eth1 inet static
    address 10.83.3.1
    netmask 255.255.255.0
```

**/root/init.sh (routing dan startup)**

```sh
#!/bin/sh
ifup -a
sysctl -w net.ipv4.ip_forward=1
ip route replace 10.83.2.0/24 via 10.83.1.1
ip route replace 10.83.4.0/24 via 10.83.1.1
echo nameserver 8.8.8.8 > /etc/resolv.conf
```

**Ursus — /etc/network/interfaces**

```sh
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 10.83.2.2
    netmask 255.255.255.0
    gateway 10.83.2.1

auto eth1
iface eth1 inet static
    address 10.83.4.1
    netmask 255.255.255.0
```

**/root/init.sh (routing dan startup)**

```sh
#!/bin/sh
ifup -a
sysctl -w net.ipv4.ip_forward=1
ip route replace 10.83.1.0/24 via 10.83.2.1
ip route replace 10.83.3.0/24 via 10.83.2.1
echo nameserver 8.8.8.8 > /etc/resolv.conf
```

`ip route replace` dipakai sebagai padanan `ip route add` agar aman dijalankan ulang. Route yang terverifikasi:

```text
# RhodesIsland
10.83.1.0/24 dev eth1
10.83.2.0/24 dev eth2
10.83.3.0/24 via 10.83.1.2 dev eth1
10.83.4.0/24 via 10.83.2.2 dev eth2
# Lungmen
default via 10.83.1.1 dev eth0
10.83.1.0/24 dev eth0
10.83.2.0/24 via 10.83.1.1 dev eth0
10.83.3.0/24 dev eth1
10.83.4.0/24 via 10.83.1.1 dev eth0
# Ursus
default via 10.83.2.1 dev eth0
10.83.1.0/24 via 10.83.2.1 dev eth0
10.83.2.0/24 dev eth0
10.83.3.0/24 via 10.83.2.1 dev eth0
10.83.4.0/24 dev eth1
```

Ping Victoria `10.83.3.2` ↔ Yan `10.83.4.2` berhasil. Output lengkap terdapat pada [verifikasi Victoria](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Victoria-verifikasi.txt) dan [verifikasi Yan](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Yan-verifikasi.txt).

[Langkah screenshot A32](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a32) — **belum dilampirkan**

 
#### Soal A.3.3 - NAT dan DNS
 
> Tunjukkan aturan `iptables` MASQUERADE pada RhodesIsland (`-s [IP Prefix].0.0/16 -o eth0`), `nameserver 8.8.8.8` pada `/etc/resolv.conf` di semua node, dan bukti semua node (RhodesIsland, Lungmen, Ursus, Victoria, Yan) dapat `ping google.com`.
>
> *Show the `iptables` MASQUERADE rule on RhodesIsland (`-s [IP Prefix].0.0/16 -o eth0`), `nameserver 8.8.8.8` in `/etc/resolv.conf` on all nodes, and proof that all nodes (RhodesIsland, Lungmen, Ursus, Victoria, Yan) can `ping google.com`.*
 
**Answer:**

Aturan NAT pada RhodesIsland menggunakan sumber `10.83.0.0/16` dan interface keluar `eth0`:

```sh
iptables -t nat -A POSTROUTING -s 10.83.0.0/16 -o eth0 -j MASQUERADE
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

DNS 8.8.8.8 dikonfigurasi pada seluruh node. RhodesIsland, Lungmen, Ursus, Victoria, Yan, Laterano, dan Columbia berhasil ping google.com pada verifikasi akhir. Output ping tersimpan pada masing-masing file `[Node]-verifikasi.txt`.

[Langkah screenshot A33](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a33) — **belum dilampirkan**

 
#### Soal A.3.4 - DHCP Server (kea-dhcp4) pada Yan dan Victoria
 
> Tunjukkan `/etc/kea/kea-dhcp4.conf` pada Yan (termasuk reservasi Laterano `[IP Prefix].4.15`) dan Victoria menggunakan kea-dhcp4 (bukan udhcpd), bukti `kea-dhcp4` berjalan, serta bukti lease client (`udhcpc -i eth0` dan `ip -br addr`) pada Laterano dan Columbia.
>
> *Show `/etc/kea/kea-dhcp4.conf` on Yan (including the Laterano reservation `[IP Prefix].4.15`) and Victoria using kea-dhcp4 (not udhcpd), proof that `kea-dhcp4` is running, plus client lease evidence (`udhcpc -i eth0` and `ip -br addr`) on Laterano and Columbia.*
 
**Answer:**

DHCP menggunakan **kea-dhcp4**. Victoria melayani pool `10.83.3.100–10.83.3.120`, gateway `10.83.3.1`; Yan melayani pool `10.83.4.50–10.83.4.60`, gateway `10.83.4.1`.

**Yan — /etc/kea/kea-dhcp4.conf**

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [
        "eth0"
      ]
    },
    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },
    "valid-lifetime": 3600,
    "subnet4": [
      {
        "id": 4,
        "subnet": "10.83.4.0/24",
        "pools": [
          {
            "pool": "10.83.4.50 - 10.83.4.60"
          }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "10.83.4.1"
          },
          {
            "name": "domain-name-servers",
            "data": "8.8.8.8"
          }
        ],
        "reservations": [
          {
            "hw-address": "02:42:41:9f:ec:00",
            "ip-address": "10.83.4.15"
          }
        ]
      }
    ]
  }
}
```

**Victoria — /etc/kea/kea-dhcp4.conf**

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [
        "eth0"
      ]
    },
    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },
    "valid-lifetime": 3600,
    "subnet4": [
      {
        "id": 3,
        "subnet": "10.83.3.0/24",
        "pools": [
          {
            "pool": "10.83.3.100 - 10.83.3.120"
          }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "10.83.3.1"
          },
          {
            "name": "domain-name-servers",
            "data": "8.8.8.8"
          }
        ]
      }
    ]
  }
}
```

Log startup menunjukkan `DHCP4_STARTED`. Laterano memperoleh **10.83.4.15** melalui reservasi MAC `02:42:41:9f:ec:00`; Columbia memperoleh **10.83.3.100**. Lease serta konfigurasi dapat ditelusuri pada [Yan](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Yan-verifikasi.txt) dan [Victoria](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Victoria-verifikasi.txt).

[Langkah screenshot A34](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a34) — **belum dilampirkan**

 
#### Soal A.3.5 - HTTP Server pada Laterano
 
> Pastikan Yan dan Laterano terhubung ke internet dan python3 sudah terpasang di Laterano (seperti modul 2). Tunjukkan `/root/myscripts/start_http.sh` yang dijalankan di tmux (split window) serta pembuatan berkas uji dengan `dd` (`tes1000.bin`, `tes10000.bin`, `tes50000.bin`, `tes5M.bin`).
>
> *Make sure Yan and Laterano can access the internet and python3 is installed on Laterano (as in module 2). Show `/root/myscripts/start_http.sh` running in tmux (split window) and the creation of the test files with `dd` (`tes1000.bin`, `tes10000.bin`, `tes50000.bin`, `tes5M.bin`).*
 
**Answer:**

Yan dan Laterano berhasil mengakses internet. Python3 tersedia di Laterano. Isi `/root/myscripts/start_http.sh`:

```sh
#!/bin/sh
mkdir -p /root/myscripts/myweb
exec python3 -m http.server 9999 -d /root/myscripts/myweb
```

Berkas uji dibuat di `/root/myscripts/myweb`:

```sh
cd /root/myscripts/myweb
dd if=/dev/urandom of=tes1000.bin bs=1000 count=1
dd if=/dev/urandom of=tes10000.bin bs=10000 count=1
dd if=/dev/urandom of=tes50000.bin bs=50000 count=1
dd if=/dev/urandom of=tes5M.bin bs=5000 count=1000
```

Ukuran aktual: 1.000 B, 10.000 B, 50.000 B, dan 5.000.000 B. Digunakan `/dev/urandom` untuk menghasilkan data acak. **Pada pelaksanaan awal server berjalan melalui nohup, belum melalui tmux split window.** Langkah memindahkannya ke tmux dan mengambil bukti ada pada panduan; jangan menjalankan ulang `dd` hanya untuk screenshot karena isi/checksum berkas akan berubah.

[Langkah screenshot A35](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a35) — **belum dilampirkan**

 
#### Soal A.3.6 - Verifikasi Awal dari Columbia
 
> Dari Columbia (VNC/xterm), tunjukkan lease DHCP dari Victoria, `ping` ke `[IP Prefix].4.15`, dan Firefox ke `http://[IP Prefix].4.15:9999` yang menampilkan seluruh berkas uji pada listing.
>
> *From Columbia (VNC/xterm), show the DHCP lease from Victoria, `ping` to `[IP Prefix].4.15`, and Firefox at `http://[IP Prefix].4.15:9999` listing all the test files.*
 
**Answer:**

Columbia memperoleh lease **10.83.3.100/24** dari Victoria, dengan gateway **10.83.3.1**. Ping menuju **10.83.4.15** berhasil. HTTP listing telah diverifikasi menggunakan curl dan memuat `tes1000.bin`, `tes10000.bin`, `tes50000.bin`, dan `tes5M.bin`. **Bukti Firefox dari VNC/xterm masih perlu diambil**; verifikasi curl tidak diklaim sebagai bukti browser.

[Langkah screenshot A36](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a36) — **belum dilampirkan**


## Problem A.4 - Three-Way Handshake

> [!NOTE]
> Skenario 1: tanpa gangguan `tc/netem` pada eth1 dan eth2 RhodesIsland. Download `tes10000.bin` dari Columbia ke Laterano (port 9999). Capture Wireshark di **Columbia** dan **Laterano** dengan filter `tcp.port == 9999`.
>
> *Scenario 1: no `tc/netem` disruption on eth1 and eth2 of RhodesIsland. Download `tes10000.bin` from Columbia to Laterano (port 9999). Capture with Wireshark on **Columbia** and **Laterano** using the filter `tcp.port == 9999`.*

 
#### Soal A.4.1 - Persiapan & Prediksi
 
> Pastikan tidak ada qdisc gangguan (`tc qdisc show`/`tc qdisc del`). Tuliskan **prediksi** ISN client pada paket SYN sebelum melihat capture.
>
> *Make sure no disruptive qdisc is active. Write your **prediction** of the client's ISN in the SYN packet before looking at the capture.*
 
**Answer:**

Prediksi kerja yang dicatat sebelum pengujian: ISN aktual client merupakan nilai 32-bit yang ditentukan stack TCP dan tidak harus nol. Wireshark dapat menampilkan SYN sebagai SEQ relatif 0 karena nomor urut dinormalisasi terhadap ISN.

Sebelum capture, qdisc gangguan pada eth1 dan eth2 RhodesIsland dihapus; qdisc yang teramati adalah **fq_codel tanpa netem**. [Transkrip qdisc](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/tc-normal.txt).

[Langkah screenshot A41](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a41) — **belum dilampirkan**

 
#### Soal A.4.2 - Tabel Observasi Three-Way Handshake

> [!NOTE]
> Untuk parameter 6-8 gunakan capture **Laterano**; untuk 9-10 gunakan capture **Columbia**.
>
> *For items 6-8, use the **Laterano** capture; for 9-10, use the **Columbia** capture.*
 
> Isi tabel berikut berdasarkan capture Wireshark.
>
> *Fill in the following table based on the Wireshark capture.*
 
| No | Parameter/Aspek | Field Wireshark | Instruksi / *Instruction* | Jawaban & Screenshot |
| --- | --- | --- | --- | --- |
| 1 | IP sumber dan tujuan<br>*Source and destination IP* | `ip.src`, `ip.dst` | Pastikan koneksi terjadi antara client (pengirim request) dan server (penerima request).<br>*Make sure the connection is between the client (request sender) and the server (request receiver).* | Client `10.83.3.100:57256` → server `10.83.4.15:9999`. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/A-handshake-seq-fin.jpg) |
| 2 | Proses handshake (3 paket awal)<br>*Handshake process (first 3 packets)* | `tcp.flags.syn`, `tcp.flags.ack` | Amati tiga paket pertama: SYN → SYN-ACK → ACK.<br>*Observe the first three packets: SYN → SYN-ACK → ACK.* | Frame 1 SYN → frame 2 SYN+ACK → frame 3 ACK. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/A-handshake-seq-fin.jpg) |
| 3 | Prediksi vs aktual: sequence number awal client<br>*Predicted vs actual: client's initial sequence number* | `tcp.seq` (SYN) | Bandingkan prediksi sebelum capture dengan nilai aktual, jelaskan mengapa ISN acak, bukan nol.<br>*Compare your prediction before the capture with the actual value, and explain why the ISN is random, not zero.* | ISN raw client **1622372703**, SEQ relatif **0**. Sesuai prediksi bahwa ISN aktual tidak harus nol; tampilan relatif menormalkan ISN. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/A-isn-aktual.jpg) |
| 4 | Sequence number awal server<br>*Server's initial sequence number* | `tcp.seq` (SYN-ACK) | Tunjukkan byte awal dari sisi server.<br>*Show the initial byte on the server side.* | Frame 2: ISN raw server **1002860090**, SEQ relatif **0**. [Langkah screenshot A42](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a42) — **belum dilampirkan** |
| 5 | ACK setelah handshake<br>*ACK after the handshake* | `tcp.ack` | Tunjukkan bahwa client mengonfirmasi SYN-ACK dari server.<br>*Show that the client acknowledges the server's SYN-ACK.* | Frame 3: ACK relatif **1**, raw **1002860091**, mengonfirmasi SYN server. [Langkah screenshot A42](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a42) — **belum dilampirkan** |
| 6 | Verifikasi SEQ segmen berikutnya = SEQ + panjang payload<br>*Verify next segment's SEQ = SEQ + payload length* | `tcp.seq`, `tcp.len` | Pilih satu segmen data di tengah stream (bukan segmen pertama atau terakhir), lalu buktikan secara manual bahwa segmen berikutnya memiliki `tcp.seq` = `tcp.seq` + `tcp.len` segmen ini.<br>*Choose a data segment in the middle of the stream (not the first or last), then manually prove that the next segment has `tcp.seq` = this segment's `tcp.seq` + `tcp.len`.* | Frame 9: SEQ **1653**, len **1448**. Frame 10: SEQ **3101**. Perhitungan **1653 + 1448 = 3101**. [Langkah screenshot A42](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a42) — **belum dilampirkan** |
| 7 | Pola ACK penerima<br>*Receiver's ACK pattern* | `tcp.ack` | Tunjukkan bahwa ACK bertambah kumulatif, menandakan data diterima secara berurutan.<br>*Show that ACKs increase cumulatively, indicating that data is received in order.* | Contoh frame 16 ACK **1653**, frame 17 ACK **3101**; ACK maju kumulatif hingga **10206** setelah data dan FIN. [Langkah screenshot A42](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a42) — **belum dilampirkan** |
| 8 | RTT rata-rata dan implikasinya terhadap timer<br>*Average RTT and its effect on the timer* | `tcp.analysis.ack_rtt` | Catat RTT pada salah satu ACK (di Laterano), lalu diskusikan mengapa timer yang terlalu pendek maupun terlalu panjang sama-sama bermasalah bagi kinerja transfer.<br>*Record the RTT from one ACK (on Laterano), then discuss why a timer that is too short and one that is too long both cause transfer performance problems.* | Rata-rata seluruh sampel ACK RTT stream Laterano **3,944 ms**; contoh frame 16 **2,873 ms**. Timer terlalu pendek dapat memicu retransmisi tidak perlu; terlalu panjang memperlambat pemulihan. [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** |
| 9 | Status retransmisi<br>*Retransmission status* | `tcp.analysis.retransmission` | Semestinya tidak ada retransmisi, menandakan link sehat.<br>*There should be no retransmissions, indicating a healthy link.* | Capture Columbia: **0 retransmisi** pada tes10000.bin. Filter `tcp.analysis.retransmission` menampilkan 0 paket. [Langkah screenshot A42](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a42) — **belum dilampirkan** |
| 10 | Pola penutupan koneksi<br>*Connection closing pattern* | `tcp.flags.fin`, `tcp.flags.ack` | Tunjukkan bahwa TCP menutup koneksi melalui four-way handshake (FIN/ACK).<br>*Show that TCP closes the connection through a four-way handshake (FIN/ACK).* | Capture Columbia: FIN+ACK server frame **15**, ACK client frame **22**, FIN+ACK client frame **23**, ACK server frame **24**. [Langkah screenshot A42](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#a42) — **belum dilampirkan** |

#### A.4.2 - Pertanyaan Analisis / Refleksi

> 1. Apa fungsi utama sequence number dan acknowledgment number dalam TCP?
>    *What are the main functions of sequence numbers and acknowledgment numbers in TCP?*
>
> 2. Mengapa ACK pada TCP bersifat kumulatif, bukan per-paket?
>    *Why are TCP ACKs cumulative rather than per-packet?*
>
> 3. Apa yang akan terjadi jika salah satu segmen data hilang di tengah pengiriman?
>    *What will happen if a data segment is lost in the middle of a transfer?*
>
> 4. Apa bukti dari hasil Wireshark bahwa TCP berhasil melakukan transfer data secara reliable?
>    *What evidence in the Wireshark results shows that TCP successfully transfers data reliably?*

**Answer:**

1. Sequence number menunjukkan posisi byte dalam stream TCP; acknowledgment number menunjukkan byte berikutnya yang diharapkan penerima.
2. ACK kumulatif mengonfirmasi seluruh byte berurutan sampai sebelum nomor ACK. Ini efisien dan ACK berikutnya tetap dapat mengonfirmasi data bila ACK sebelumnya hilang. SACK dapat melengkapi informasi blok yang diterima di luar urutan.
3. Jika segmen hilang, ACK kumulatif dapat tertahan pada byte yang hilang sementara paket berikutnya memicu duplicate ACK. Pengirim memulihkan data melalui retransmisi berdasarkan indikasi ACK/SACK atau timer.
4. Dalam uji tes10000.bin, handshake berhasil, SEQ/ACK maju sesuai payload, seluruh 10.000 B diterima oleh curl, tidak ada retransmisi, dan koneksi ditutup dengan FIN/ACK. Ini mendukung keberhasilan transfer pada percobaan tersebut.


---

# Bagian B - Retransmission & Duplicate ACK

> [!NOTE]
> Skenario 2: packet loss 1% pada eth1 dan eth2 RhodesIsland. Download `tes5M.bin`. Untuk parameter 2 dan 5 gunakan capture **Columbia**; selain itu gunakan capture **Laterano**.
>
> *Scenario 2: 1% packet loss on eth1 and eth2 of RhodesIsland. Download `tes5M.bin`. For items 2 and 5, use the **Columbia** capture; for the others, use the **Laterano** capture.*

## Problem B.1 - B.3
 
#### Soal B.1 - Persiapan Gangguan & Prediksi
 
> Terapkan `tc qdisc add dev eth1/eth2 root netem loss 1%`. Tuliskan prediksi reaksi penerima terhadap loss sebelum melihat capture.
>
> *Apply `tc qdisc add dev eth1/eth2 root netem loss 1%`. Write your prediction of the receiver's reaction to loss before looking at the capture.*
 
**Answer:**

Prediksi kerja sebelum capture: ketika ada rentang byte yang hilang, penerima mengulang ACK untuk byte berikut yang diharapkan, bukan mengirim NAK TCP. Pengirim dapat mengirim ulang melalui fast retransmit atau mekanisme berbasis timer.

Pada eth1 dan eth2 RhodesIsland diterapkan:

```sh
tc qdisc del dev eth1 root 2>/dev/null || true
tc qdisc del dev eth2 root 2>/dev/null || true
tc qdisc add dev eth1 root netem loss 1%
tc qdisc add dev eth2 root netem loss 1%
tc qdisc show dev eth1
tc qdisc show dev eth2
```

[Output konfigurasi aktual](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/tc-loss1.txt). Probabilitas 1% berlaku pada setiap qdisc egress, bukan jaminan tepat 1% pada setiap unduhan.

[Langkah screenshot B1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b1) — **belum dilampirkan**

 
#### Soal B.3 - Tabel Observasi Retransmisi & Duplicate ACK
 
> Isi tabel berikut berdasarkan capture Wireshark dan output `curl`.
>
> *Fill in the following table based on the Wireshark capture and the `curl` output.*
 
| No | Parameter/Aspek | Field Wireshark | Instruksi / *Instruction* | Jawaban & Screenshot |
| --- | --- | --- | --- | --- |
| 1 | Persentase loss yang disimulasikan<br>*Simulated loss percentage* | `tc qdisc show` | Tunjukkan nilai setup packet loss (1%) pada `tc netem`.<br>*Show the packet loss setting (1%) in `tc netem`.* | Loss **1%** pada eth1 dan eth2. [Langkah screenshot B1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b1) — **belum dilampirkan** |
| 2 | Prediksi vs aktual: reaksi penerima terhadap loss (Columbia)<br>*Predicted vs actual: receiver's response to loss (Columbia)* | `tcp.analysis.duplicate_ack` | Bandingkan prediksi dengan pola dupACK aktual (ACK bernomor sama berulang), bukan NAK.<br>*Compare your prediction with the actual dupACK pattern (repeated ACKs with the same number), not NAK.* | ACK berulang untuk byte hilang sesuai prediksi; contoh ACK **217407** berulang, bukan NAK. Gunakan capture Columbia. [Langkah screenshot B3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b3) — **belum dilampirkan** |
| 3 | Jumlah retransmisi<br>*Number of retransmissions* | `tcp.analysis.retransmission` | Tunjukkan paket yang dikirim ulang akibat tidak menerima ACK yang diharapkan.<br>*Show the packets sent again because the expected ACK was not received.* | **32 retransmisi** pada Laterano. [Langkah screenshot B3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b3) — **belum dilampirkan** |
| 4 | Verifikasi SEQ segmen retransmisi = SEQ segmen asli yang hilang<br>*Verify retransmitted segment's SEQ = original lost segment's SEQ* | `tcp.seq`, `tcp.len` | Bandingkan paket `tcp.analysis.retransmission` dengan segmen aslinya, buktikan bahwa retransmisi mengirim ulang byte yang identik, bukan data baru.<br>*Compare a `tcp.analysis.retransmission` packet with its original segment, and prove that the retransmission sends the same bytes again, not new data.* | Laterano frame **215**: SEQ **217407**, len **1448**; frame **244**: SEQ **217407**, len **1448**. Payload kedua paket identik. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/B-loss1-retransmisi-dupACK.jpg) [Langkah screenshot B3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b3) — **belum dilampirkan** |
| 5 | Jumlah duplicate ACK (Columbia)<br>*Number of duplicate ACKs (Columbia)* | `tcp.analysis.duplicate_ack` | Tunjukkan bahwa sedikitnya 3 dupACK berturut-turut memicu fast retransmit.<br>*Show that at least 3 consecutive dupACKs trigger fast retransmit.* | Columbia: **183 duplicate ACK**. Periksa kelompok ACK yang sama dan label Dup ACK #1/#2/#3 pada capture Columbia. [Langkah screenshot B3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b3) — **belum dilampirkan** |
| 6 | RTT sebelum dan sesudah loss<br>*RTT before and after loss* | `tcp.analysis.ack_rtt` | Tunjukkan bahwa RTT cenderung meningkat karena waktu recovery.<br>*Show that RTT tends to increase because of recovery time.* | Rata-rata Laterano normal 5 MB **5,902 ms**, loss1 **1,555 ms**; maksimum loss1 **41,215 ms**. Rata-rata tidak meningkat pada hasil ini; tidak dipaksakan sesuai dugaan. [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** |
| 7 | Fast retransmit terdeteksi beserta selisih waktunya<br>*Detected fast retransmit and its time difference* | `tcp.analysis.fast_retransmission`, `frame.time_relative` | Ukur selisih waktu antara dupACK ke-3 dan retransmisi yang menyusul, bandingkan dengan potensi timeout.<br>*Measure the time between the third dupACK and the retransmission that follows, and compare it with a possible timeout.* | Laterano: dupACK ke-3 frame **239** pada **0,025192 s**, fast retransmit frame **244** pada **0,025758 s**; selisih **0,566 ms**. Ada **25** paket berlabel fast retransmission. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/B-loss1-retransmisi-dupACK.jpg) |
| 8 | Timeout retransmission<br>*Timeout retransmission* | `frame.time_relative` | Terjadi bila tidak ada dupACK yang memicu fast retransmit. Jika tidak muncul, jelaskan apa yang diamati beserta alasannya.<br>*Occurs when there are no dupACKs to trigger fast retransmit. If it does not appear, describe what was observed and why.* | Terdapat **7** retransmisi tanpa label fast; pemicu timeout **belum dapat dipastikan** dari label ini saja. Interval paket dan `tcp.analysis.rto` bukan pembacaan timer kernel. [Langkah screenshot B3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#b3) — **belum dilampirkan** |
| 9 | Throughput rata-rata<br>*Average throughput* | `curl -w` (`speed_download`) | Bandingkan dengan throughput pada Skenario 1 (tanpa loss).<br>*Compare with the throughput in Scenario 1 (without loss).* | Normal 5 MB **12.560.604 B/s**; loss1 **7.464.725 B/s**. [Langkah screenshot LOG](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#log) — **belum dilampirkan** |
| 10 | Lama transfer total<br>*Total transfer duration* | `curl -w` (`time_total`) | Tunjukkan bahwa waktu transfer meningkat dibanding Skenario 1.<br>*Show that the transfer time increases compared with Scenario 1.* | Normal 5 MB **0,398070 s**; loss1 **0,669817 s**. [Langkah screenshot LOG](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#log) — **belum dilampirkan** |
| 11 | Pola congestion window (perkiraan)<br>*Congestion window pattern (estimate)* | `tcp.analysis.bytes_in_flight` | Tunjukkan bahwa cwnd menurun setelah loss, indikasi congestion control aktif.<br>*Show that cwnd decreases after loss, indicating that congestion control is active.* | `bytes_in_flight` menunjukkan data belum di-ACK; perubahan di sekitar loss dapat dipakai sebagai indikator, tetapi **cwnd langsung tidak diukur**. Penurunan cwnd spesifik belum dibuktikan dari field ini saja. [Langkah screenshot FLIGHT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#flight) — **belum dilampirkan** |

#### B.3 - Pertanyaan Analisis / Refleksi

> 1. Bagaimana TCP mengetahui bahwa sebuah segmen hilang?
>    *How does TCP know that a segment is lost?*
>
> 2. Jelaskan perbedaan antara timeout retransmission dan fast retransmit.
>    *Explain the difference between timeout retransmission and fast retransmit.*
>
> 3. Mengapa throughput menurun saat terjadi packet loss?
>    *Why does throughput decrease when packet loss occurs?*
>
> 4. Bagaimana Wireshark menunjukkan bahwa TCP tetap reliable meskipun terjadi packet loss?
>    *How does Wireshark show that TCP remains reliable even when packet loss occurs?*

**Answer:**

1. TCP mendeteksi indikasi kehilangan dari ACK kumulatif yang tidak maju, duplicate ACK, informasi SACK, dan berakhirnya timer retransmisi.
2. Fast retransmit memulihkan data sebelum menunggu RTO berdasarkan indikasi kehilangan dari ACK. Timeout retransmission terjadi saat timer berakhir tanpa konfirmasi yang diperlukan. Label non-fast pada Wireshark saja tidak membuktikan bahwa timer kernel telah habis.
3. Throughput turun karena bandwidth/waktu dipakai untuk retransmisi dan congestion control dapat mengurangi jumlah data yang boleh dikirim. Pada uji 5 MB ini throughput turun dari 12.560.604 menjadi 7.464.725 B/s.
4. Capture menunjukkan retransmisi dengan SEQ dan payload yang sama, ACK kemudian maju, dan curl tetap menerima 5.000.000 B. Ini membuktikan pemulihan kehilangan pada transfer yang diamati.


---

# Bagian C - TCP Congestion Control (CUBIC vs Reno)

> [!NOTE]
> Skenario 3: loss 1%, delay 100 ms, rate limit 2 Mbps (tbf + netem) pada eth1 dan eth2 RhodesIsland. Algoritma diatur di **Laterano** (sender). Download `tes5M.bin`. Untuk parameter 4, 5, dan 9 gunakan capture **Columbia**; selain itu gunakan capture **Laterano**. Jangan klaim mengukur cwnd secara langsung dari grafik Time-Sequence / `bytes_in_flight`.
>
> *Scenario 3: 1% loss, 100 ms delay, 2 Mbps rate limit (tbf + netem) on eth1 and eth2 of RhodesIsland. The algorithm is set on **Laterano** (sender). Download `tes5M.bin`. For items 4, 5, and 9 use the **Columbia** capture; for the others use the **Laterano** capture. Do not claim to measure cwnd directly from the Time-Sequence graph / `bytes_in_flight`.*

## Problem C.1 - C.3

#### Soal C.1 - Setup Gangguan & Prediksi Kurva cwnd

> Tunjukkan konfigurasi `tc` (tbf + netem). Tuliskan prediksi bentuk kurva cwnd untuk Reno dan CUBIC sebelum melihat grafik.
>
> *Show the `tc` configuration (tbf + netem). Write your predicted cwnd curve shape for Reno and CUBIC before looking at the actual graphs.*

**Answer:**

Prediksi Reno: setelah loss, cwnd berkurang dan pada congestion avoidance tumbuh kira-kira linear per RTT. Prediksi CUBIC: setelah pengurangan cwnd, pertumbuhan mengikuti fungsi kubik terhadap waktu di sekitar maksimum sebelumnya. Ini prediksi kerja yang dicatat sebelum eksperimen, bukan pengukuran cwnd.

Konfigurasi dijalankan pada **kedua interface** RhodesIsland:

```sh
for dev in eth1 eth2; do
  tc qdisc del dev "$dev" root 2>/dev/null || true
  tc qdisc add dev "$dev" root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
  tc qdisc add dev "$dev" parent 1:1 handle 10: netem delay 100ms loss 1%
  tc qdisc show dev "$dev"
done
```

[Output aktual](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/tc-limited.txt). Delay data dan ACK masing-masing sekitar 100 ms sehingga RTT dasar mendekati 200 ms. Parameter latency TBF mengatur antrean dan bukan delay tetap tambahan.

[Langkah screenshot C1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#c1) — **belum dilampirkan**


#### Soal C.2 - Konfigurasi Algoritma Congestion Control
 
> Atur algoritma di Laterano (sender) lalu jalankan download `tes5M.bin` dari Columbia untuk masing-masing algoritma.
>
> *Set the algorithm on Laterano (sender), then download `tes5M.bin` from Columbia for each algorithm.*
 
| Langkah / *Step* | CUBIC | Reno |
| --- | --- | --- |
| Command di Laterano<br>*Command on Laterano* | `ip tcp_metrics flush`<br>`sysctl -w net.ipv4.tcp_no_metrics_save=1`<br>`sysctl -w net.ipv4.tcp_congestion_control=cubic`<br>`sysctl net.ipv4.tcp_congestion_control` | `sysctl -w net.ipv4.tcp_congestion_control=reno`<br>`ip tcp_metrics flush`<br>`sysctl net.ipv4.tcp_congestion_control` |
| Command di Columbia<br>*Command on Columbia* | `curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.83.4.15:9999/tes5M.bin` | Perintah curl sama; hasil **89.455 B/s**, **55,893794 s**. |
| Screenshot | [Langkah screenshot C2](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#c2) — **belum dilampirkan** | [Langkah screenshot C2](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#c2) — **belum dilampirkan** |
 
#### Soal C.3 - Tabel Perbandingan CUBIC vs Reno
 
> Isi tabel berikut dengan hasil aktual untuk CUBIC dan Reno (nilai + screenshot), lalu tulis analisis singkat pada kolom catatan.
> Setiap sel kosong untuk CUBIC dan RENO pada tabel di bawah **wajib** berisi **nilai aktual** dan **screenshot** sebagai bukti, kecuali parameter nomor 1.
>
> *Fill in the following table with actual results for CUBIC and Reno (value + screenshot), then write a short analysis in the notes column.*
> *Every cell for CUBIC and RENO column in the table below **must** contain the **actual value** and a **screenshot** as evidence, except for parameter 1.*
 
| No | Parameter/Aspek | Field Wireshark / Command | CUBIC | Reno | Catatan analisis / *Analysis notes* |
| --- | --- | --- | --- | --- | --- |
| 1 | Prediksi vs aktual: bentuk kurva cwnd<br>*Predicted vs actual: cwnd curve shape* | Time - Sequence Graph | Prediksi pertumbuhan kubik; SEQ aktual naik dengan kemiringan berubah. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/C-cubic-Wireshark-Stevens.jpg) | Prediksi pertumbuhan linear pada congestion avoidance; SEQ aktual naik dengan beberapa bagian landai. [Langkah screenshot GRAPH](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#graph) — **belum dilampirkan** | Time–Sequence bukan cwnd; jangan menyamakan sumbu grafik. |
| 2 | RTT rata-rata (ms)<br>*Average RTT (ms)* | `tcp.analysis.ack_rtt` | 207,059 ms [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | 206,871 ms [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | Rata-rata seluruh sampel ACK RTT stream pada Laterano. |
| 3 | RTT maksimum (ms)<br>*Maximum RTT (ms)* | `tcp.analysis.ack_rtt` | 348,229 ms [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | 606,655 ms [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | Maksimum field ACK RTT, bukan maksimum dari metode sampling lain pada grafik RTT. |
| 4 | Throughput (B/s) (Columbia)<br>*Throughput (B/s) (Columbia)* | `curl -w` (`speed_download`) | 96.871 B/s [Langkah screenshot LOG](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#log) — **belum dilampirkan** | 89.455 B/s [Langkah screenshot LOG](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#log) — **belum dilampirkan** | Goodput aplikasi curl; satu percobaan tiap algoritma. |
| 5 | Waktu transfer total (s) (Columbia)<br>*Total transfer time (s) (Columbia)* | `curl -w` (`time_total`) | 51,614753 s [Langkah screenshot LOG](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#log) — **belum dilampirkan** | 55,893794 s [Langkah screenshot LOG](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#log) — **belum dilampirkan** | CUBIC selesai lebih cepat pada percobaan ini. |
| 6 | Jumlah retransmisi<br>*Number of retransmissions* | `tcp.analysis.retransmission` | 29 [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | 36 [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | Jumlah filter retransmission di Laterano. |
| 7 | Jumlah duplicate ACK<br>*Number of duplicate ACKs* | `tcp.analysis.duplicate_ack` | 380 [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | 482 [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | Jumlah filter duplicate_ack di Laterano. |
| 8 | Pola congestion window (awal - akhir)<br>*Congestion window pattern (start - end)* | `tcp.analysis.bytes_in_flight` | Awal payload server 206 → 1654 → 3102 B in flight; selanjutnya berubah selama transfer. [Langkah screenshot FLIGHT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#flight) — **belum dilampirkan** | Awal payload server 206 → 1654 → 3102 B in flight; selanjutnya berubah selama transfer. [Langkah screenshot FLIGHT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#flight) — **belum dilampirkan** | Ini indikator outstanding data, bukan nilai cwnd langsung; lihat [Langkah screenshot FLIGHT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#flight) — **belum dilampirkan** |
| 9 | Nilai receive window rata-rata (Columbia)<br>*Average receive window value (Columbia)* | `tcp.window_size_value` | 4.231,899 (raw) [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | 19.004,884 (raw) [Langkah screenshot STAT](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#stat) — **belum dilampirkan** | Rata-rata raw window dari paket Columbia, termasuk SYN; perhatikan window scaling. |
| 10 | Stabilitas throughput (grafik)<br>*Throughput stability (graph)* | Throughput Graph | Berfluktuasi; laju meningkat sekitar detik 28–35. ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/C-cubic-Wireshark-Throughput.jpg) | Berfluktuasi; laju awal dan akhir lebih tinggi. [Langkah screenshot GRAPH](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#graph) — **belum dilampirkan** | Grafik olahan kedua capture: ![Bukti](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/bukti/C-perbandingan-grafik.png) |


#### C.3 - Pertanyaan Analisis / Refleksi

> 1. Algoritma mana yang memberikan throughput lebih tinggi pada kondisi jaringan ini? Mengapa demikian?
>    *Which algorithm provides higher throughput under these network conditions? Why?*
>
> 2. Apakah ada hubungan antara jumlah retransmisi dan jenis algoritma congestion control yang digunakan?
>    *Is there a relationship between the number of retransmissions and the congestion control algorithm used?*
>
> 3. Apa perbedaan bentuk kurva congestion window antara Reno dan Cubic pada grafik Time-Sequence (Stevens)?
>    *How do the congestion window curves of Reno and Cubic differ in the Time-Sequence (Stevens) graph?*
>
> 4. Algoritma mana yang paling sensitif terhadap RTT tinggi?
>    *Which algorithm is most sensitive to high RTT?*
>
> 5. Algoritma mana yang lebih efisien pada jaringan dengan latency tinggi seperti pada skenario ini? Jelaskan.
>    *Which algorithm is more efficient on a network with high latency, as in this scenario? Explain.*

**Answer:**

1. CUBIC menghasilkan throughput **96.871 B/s**, lebih tinggi daripada Reno **89.455 B/s** pada satu percobaan ini, dengan durasi 51,614753 s dibanding 55,893794 s. Perbedaan pemulihan laju dapat berpengaruh, tetapi realisasi loss acak juga berbeda; hasil ini tidak membuktikan CUBIC selalu unggul.
2. Keduanya memakai sinyal kehilangan dalam pengendalian kemacetan. Teramati 29 retransmisi CUBIC dan 36 Reno; jumlah juga bergantung pada loss aktual, antrean, dan paket yang terkirim.
3. Prediksi cwnd Reno dan CUBIC berbeda pada pola pertumbuhannya. Namun grafik Time–Sequence menunjukkan kemajuan nomor urut byte terhadap waktu, **bukan cwnd langsung**. Grafik aktual memperlihatkan perubahan kemiringan; pola sawtooth cwnd tidak harus tampak sebagai garis SEQ yang turun.
4. Pertumbuhan aditif Reno lebih terikat pada jumlah RTT sehingga RTT tinggi dapat memperlambat kenaikan jendela per detik. CUBIC memakai pertumbuhan berbasis waktu dengan penyesuaian Reno-friendly.
5. Dalam uji ini CUBIC lebih efisien menurut throughput dan waktu transfer. RTT rata-rata keduanya sekitar 207 ms. Kesimpulan dibatasi pada satu pengujian per algoritma dengan loss acak.


---

# Bagian D - Verifikasi Perilaku TCP vs UDP

> [!NOTE]
> Packet loss 5% pada eth1 dan eth2 RhodesIsland. `iperf3 -s` di Laterano; client di Columbia.

## Problem D.1 - D.3

#### Soal D.1 - Setup Gangguan (`tc qdisc show`)

> Loss 5% diterapkan pada kedua interface RhodesIsland.

**Answer:**

Aturan sebelumnya dihapus sehingga uji ini hanya memakai loss 5%, tanpa pembatasan 2 Mbit/s dan tanpa delay netem 100 ms dari bagian C:

```sh
for dev in eth1 eth2; do
  tc qdisc del dev "$dev" root 2>/dev/null || true
  tc qdisc add dev "$dev" root netem loss 5%
  tc qdisc show dev "$dev"
done
```

[Output aktual](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/tc-loss5.txt). Laterano menjalankan iperf3 server pada port 5201; client adalah Columbia.

[Langkah screenshot D1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d1) — **belum dilampirkan**


#### Soal D.3 - Tabel Hasil Pengujian TCP vs UDP

> Jalankan pengujian TCP lalu UDP, catat hasil sender/receiver (nilai + screenshot), lalu bandingkan. Kesimpulan harus didasarkan pada hasil pengukuran yang setara.
> Setiap sel kosong pada tabel di bawah **wajib** berisi **nilai aktual** dan **screenshot** sebagai bukti.
>
> *Run the TCP test and then the UDP test, record the sender/receiver results (value + screenshot), then compare them. The conclusion must be based on comparable measurements.*
> *Every cell in the table below **must** contain the **actual value** and a **screenshot** as evidence.*
 
| Parameter | TCP | UDP |
| --- | --- | --- |
| Command di Columbia<br>*Command on Columbia* | `iperf3 -c 10.83.4.15 -t 10` | `iperf3 -c 10.83.4.15 -u -b 5M -t 10` |
| Bitrate/throughput sender | 6,395822 Mbit/s [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** | 5,000198 Mbit/s [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** |
| Bitrate/throughput receiver | 6,289562 Mbit/s [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** | 4,723704 Mbit/s [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** |
| Retransmission yang dilaporkan<br>*Reported retransmissions* | 289 [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** | Tidak ada retransmisi bawaan UDP [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** |
| Lost / Total Datagrams | Tidak tersedia dalam format TCP [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** | 238 / 4317 (5,513088%) [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** |
| Jitter (jika tercantum)<br>*Jitter (if shown)* | Tidak dilaporkan [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** | 0,122897 ms [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** |
| Ringkasan respons terhadap loss<br>*Summary of the response to loss* | Menyesuaikan laju dan mengirim ulang data [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** | Sender mengejar target 5 Mbit/s; receiver berkurang karena datagram hilang [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan** |
| Screenshot output iperf3 | [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan**  | [Langkah screenshot D3](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#d3) — **belum dilampirkan**  |


Output asli memakai tambahan `-J` untuk JSON: [TCP](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/data/D-TCP.json), [UDP](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/data/D-UDP.json). Angka di atas berasal dari file tersebut.

#### D.3 - Pertanyaan Analisis / Refleksi

> 1. Mengapa hasil iperf3 mode UDP menampilkan persentase datagram loss, sedangkan hasil mode TCP tidak?
>    *Why do the iperf3 UDP results show the percentage of datagram loss, while the TCP results do not?*
>
> 2. Mengapa throughput UDP tetap mendekati nilai yang diminta (`-b`) meskipun terjadi packet loss, sedangkan throughput TCP menurun?
>    *Why does UDP throughput stay close to the requested value (-b) despite packet loss, while TCP throughput decreases?*
>
> 3. Mengapa throughput TCP bisa turun pada kondisi loss, sedangkan UDP tetap mencoba mengirim sesuai target bitrate?
>    *Why can TCP throughput decrease under loss, while UDP keeps trying to send at the target bitrate?*
>
> 4. Untuk jenis aplikasi seperti apa pilihan UDP lebih tepat dibanding TCP, berdasarkan hasil pengamatan ini?
>    *Based on these observations, for what types of applications is UDP a better choice than TCP?*

**Answer:**

1. iperf3 UDP menghitung datagram hilang menggunakan nomor urut datagram. TCP memulihkan kehilangan dan menyajikan stream byte sehingga tidak melaporkan lost/total dalam format datagram UDP.
2. Bitrate **sender** UDP mendekati target 5 Mbit/s, tetapi throughput **receiver** hanya 4,723704 Mbit/s akibat kehilangan. Jadi laju penerima UDP tetap terpengaruh loss; yang terus mendekati target adalah laju pengirim.
3. TCP melakukan retransmisi serta menyesuaikan cwnd/laju setelah indikasi kemacetan. UDP iperf3 tetap mencoba mengirim sesuai target bitrate tanpa retransmisi bawaan. TCP dalam uji ini mencapai receiver 6,289562 Mbit/s, lebih tinggi daripada UDP yang ditargetkan 5 Mbit/s; karena batas laju aplikasi berbeda, ini bukan perbandingan kapasitas maksimum yang sepenuhnya setara.
4. UDP cocok untuk aplikasi waktu nyata yang memprioritaskan ketepatan waktu dan dapat menoleransi/menangani kehilangan, misalnya percakapan suara dan video interaktif. Keandalan tambahan dapat dibuat oleh aplikasi.


---

# Bagian E - Tugas Mandiri

> [!IMPORTANT]
> Gunakan topologi dari pengerjaan Bagian A - D sebelumnya. Isi tabel dengan **hasil aktual**, bukan angka perkiraan.
>
> *Use the topology from your earlier work in Parts A - D. Fill in the table with **actual results**, not estimated numbers.*

#### Soal E.1 - Ulangi Skenario 1 untuk Tiga Ukuran Berkas

> Ulangi Skenario 1 (tanpa gangguan) untuk `tes1000.bin`, `tes10000.bin`, dan `tes5M.bin`, lalu lengkapi tabel berikut berdasarkan capture Wireshark di Laterano dan Columbia serta output `curl`.
> Setiap sel kosong pada tabel di bawah **wajib** berisi **nilai aktual** dan **screenshot** sebagai bukti.
>
> *Repeat Scenario 1 for the three file sizes and complete the table using the Wireshark captures from Laterano and Columbia and the `curl` output.*
> *Every cell in the table below **must** contain the **actual value** and a **screenshot** as evidence.*
 
**Tabel Observasi Kondisi Normal**
 
| No | Parameter/Aspek | Field Wireshark | tes1000.bin | tes10000.bin | tes5M.bin |
| --- | --- | --- | --- | --- | --- |
| 1 | Ukuran berkas (B)<br>*File size (B)* | - | 1000 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 10000 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 5000000 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 2 | Jumlah total segmen TCP<br>*Total number of TCP segments* | Statistics Conversations TCP | 13 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 24 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 4275 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 3 | RTT rata-rata (Laterano)<br>*Average RTT (Laterano)* | `tcp.analysis.ack_rtt` | 1.271 ms [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 3.944 ms [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 5.902 ms [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 4 | Throughput (curl) | `speed_download` | 215053 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 1185114 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 12560604 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 5 | Waktu total transfer (curl)<br>*Total transfer time (curl)* | `time_total` | 0,004650 s [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 0,008438 s [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 0,398070 s [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 6 | Sequence number awal client<br>*Client's initial sequence number* | `tcp.seq` (SYN) | 1373903765 (raw); relatif 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 1622372703 (raw); relatif 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 2382143879 (raw); relatif 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 7 | Sequence number awal server<br>*Server's initial sequence number* | `tcp.seq` (SYN-ACK) | 3042906599 (raw); relatif 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 1002860090 (raw); relatif 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 942977871 (raw); relatif 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 8 | Jumlah ACK yang dikirim client<br>*Number of ACKs sent by the client* | Statistics Endpoints TCP | 6 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 11 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 816 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 9 | Status retransmisi (Laterano)<br>*Retransmission status (Laterano)* | `tcp.analysis.retransmission` | 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 0 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** | 9 [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan** |
| 10 | Screenshot | Wireshark / curl | [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan**  | [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan**  | [Langkah screenshot E1](C:/Users/Lenovo/Documents/Codex/2026-11-05/install-this-skill-4/outputs/Panduan-Screenshot.md#e1) — **belum dilampirkan**  |


RTT = rata-rata seluruh sampel `tcp.analysis.ack_rtt` pada capture Laterano untuk stream HTTP. Jumlah ACK client menghitung paket client dengan flag ACK, termasuk GET dan FIN+ACK; bukan sekadar jumlah paket dalam Endpoints. Throughput memakai B/s.

#### E.2 - Pertanyaan Kesimpulan

> 1. Apakah jumlah segmen TCP, RTT, dan throughput berubah secara proporsional terhadap ukuran berkas?
>    *Do the number of TCP segments, RTT, and throughput change in proportion to file size?*
>
> 2. Parameter mana yang relatif tetap tidak bergantung pada ukuran berkas, dan mengapa demikian?
>    *Which parameters stay relatively constant regardless of file size, and why?*
>
> 3. Apa perbedaan antara total waktu download yang diukur curl dengan RTT paket TCP?
>    *What is the difference between the total download time measured by curl and TCP packet RTT?*
>

**Answer:**

1. Tidak semuanya berubah proporsional. Jumlah segmen meningkat dari **13 → 24 → 4275**, tetapi ada overhead handshake, HTTP, ACK dan retransmisi. RTT rata-rata **1,271 → 3,944 → 5,902 ms** serta throughput **215.053 → 1.185.114 → 12.560.604 B/s** juga tidak sebanding langsung dengan ukuran. Transfer kecil lebih dipengaruhi overhead awal; antrean dan beban VM ikut memengaruhi hasil.
2. IP endpoint, port layanan 9999, mekanisme handshake, serta aturan byte SEQ/ACK relatif tetap. RTT dasar terutama dipengaruhi jalur, meskipun sampelnya dapat berubah karena antrean. ISN aktual berubah per koneksi dan bukan fungsi ukuran berkas.
3. Waktu total curl adalah waktu penyelesaian proses unduhan, termasuk pembentukan koneksi dan penerimaan seluruh respons. RTT adalah waktu satu pengiriman sampai ACK-nya teramati. Karena itu waktu download tidak sama dengan satu RTT.

Pada transfer normal 5 MB tetap teramati **9 retransmisi** meskipun tidak ada netem. Hasil ini dilaporkan apa adanya; antrean, reordering atau beban lingkungan virtual merupakan kemungkinan yang belum dipastikan penyebabnya. Seluruh 5.000.000 B diterima. Checksum unduhan verifikasi tambahan juga cocok dengan server.

