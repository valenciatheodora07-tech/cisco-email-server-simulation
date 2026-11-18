# cisco-email-server-simulation
Perancangan, konfigurasi, dan simulasi infrastruktur Email Server lintas domain (SMTP, POP3, DNS, dan TCP) menggunakan Cisco Packet Tracer beserta analisis pertukaran paket.
# 📧 Cross-Domain Email Server Simulation — Cisco Packet Tracer

Proyek ini mendokumentasikan perancangan, konfigurasi, serta analisis pertukaran paket tingkat protokol pada infrastruktur **Email Server lintas domain** menggunakan **Cisco Packet Tracer**. Simulasi ini membedah secara mendalam bagaimana klien dan server berkomunikasi menggunakan protokol **TCP**, **DNS**, **SMTP**, dan **POP3** dalam skenario pengiriman serta penerimaan pesan elektronik antar-domain (`compnet.com` ke `gooaya.com`).

---

## 📋 Daftar Isi
- [Tujuan Proyek](#-tujuan-proyek)
- [Arsitektur & Topologi Jaringan](#-arsitektur--topologi-jaringan)
- [Tabel Pengalamatan & Konfigurasi Layanan](#-tabel-pengalamatan--konfigurasi-layanan)
- [Analisis Alur Paket (Simulation Mode)](#-analisis-alur-paket-simulation-mode)
  - [1. Alur Pengiriman Email (SMTP & DNS)](#1-alur-pengiriman-email-smtp--dns)
  - [2. Alur Penerimaan Email (POP3)](#2-alur-penerimaan-email-pop3)
- [Hasil Verifikasi](#-hasil-verifikasi)

---

## 🎯 Tujuan Proyek
1. Membangun dan mengintegrasikan node Email Server baru (`ComNet Server`) ke dalam jaringan lokal berbasis *switch* dengan topologi *star*.
2. Mengonfigurasi layanan **SMTP** (*Simple Mail Transfer Protocol*) dan **POP3** (*Post Office Protocol v3*) beserta manajemen akun pengguna pada server.
3. Menganalisis tahapan pertukaran paket (*packet flow*) secara *real-time* menggunakan **Simulation Mode**, mulai dari *TCP handshake*, resolusi domain via **DNS**, *mail relay* antar-server, hingga pengunduhan email oleh klien penerima.

---

## 🌐 Arsitektur & Topologi Jaringan

Jaringan lokal dibangun menggunakan topologi *star* yang terpusat pada **Cisco Catalyst 2950-24 (`Switch0`)** sebagai penghubung seluruh node klien dan server:

* **End Devices (Clients):** `PC0`, `PC1` (klien penerima), dan `PC2` (klien pengirim).
* **Dedicated Servers:**
  * `ComNet Server`: Email Server utama untuk domain `compnet.com`.
  * `Gooaya Server`: Email Server tujuan untuk domain `gooaya.com`.
  * `DNS Server`: Server resolusi nama domain untuk menerjemahkan alamat domain tujuan menjadi alamat IP.
  * `First Server` & `Second Server`: Server pendukung infrastruktur jaringan lokal.

---

## ⚙️ Tabel Pengalamatan & Konfigurasi Layanan

### 1. Konfigurasi Server
| Nama Perangkat | Peran Layanan | IPv4 Address | Subnet Mask | Domain / Service Aktif |
| :--- | :--- | :---: | :---: | :--- |
| **ComNet Server** | Email Server (Asal) | `192.168.1.6` | `255.255.255.0` | Domain: `compnet.com` (`SMTP`: ON, `POP3`: ON) |
| **Gooaya Server** | Email Server (Tujuan) | `192.168.1.5` | `255.255.255.0` | Domain: `gooaya.com` (`SMTP`: ON, `POP3`: ON) |
| **DNS Server** | Name Resolution | — | — | `DNS`: ON (Pemetaan record `compnet.com` & `gooaya.com`) |

### 2. Konfigurasi Mail Client
| Perangkat | Nama Pengguna | Alamat Email | Incoming Mail Server | Outgoing Mail Server |
| :--- | :---: | :--- | :---: | :---: |
| **PC2 (Pengirim)** | `nicho` | `nicho@compnet.com` | `192.168.1.6` | `192.168.1.6` |
| **PC1 (Penerima)** | `miau` | `miau@gooaya.com` | `192.168.1.5` | `192.168.1.5` |

---

## 🔍 Analisis Alur Paket (Simulation Mode)

Pengujian dilakukan dengan mengirimkan email dari `nicho@compnet.com` (`PC2`) kepada `miau@gooaya.com` (`PC1`) dengan subjek **`Tes Simulation`**.

### 1. Alur Pengiriman Email (SMTP & DNS)
Proses pengiriman pesan dari `PC2` menuju `Gooaya Server` melalui `ComNet Server` tercatat berlangsung selama **`0.000s` hingga `0.026s`**:

1. **Pembentukan Koneksi Awal / TCP Handshake (`0.000s – 0.004s`):**
   * `PC2` menginisiasi sesi komunikasi dengan mengirimkan paket `TCP` melalui `Switch0`.
   * `ComNet Server` merespons paket tersebut untuk menyelesaikan proses *3-way handshake*.
2. **Resolusi Nama Domain via DNS (`0.007s – 0.011s`):**
   * Karena alamat tujuan berada pada domain berbeda (`gooaya.com`), `ComNet Server` mengirimkan *DNS query* melalui `Switch0` ke `DNS Server`.
   * `DNS Server` membalas dengan alamat IP tujuan (`192.168.1.5`).
3. **Transfer & Relay Pesan via SMTP (`0.004s – 0.020s`):**
   * `PC2` mengirimkan data email ke `ComNet Server` menggunakan protokol `SMTP`.
   * `ComNet Server` bertindak sebagai *Mail Transfer Agent (MTA)* yang meneruskan pesan `SMTP` tersebut ke `Gooaya Server` melalui `Switch0`.
4. **Penutupan Koneksi TCP (`0.020s – 0.026s`):**
   * Setelah transaksi pesan selesai, `ComNet Server`, `Gooaya Server`, dan `PC2` saling bertukar paket `TCP` penutup untuk mengakhiri sesi dengan aman.

### 2. Alur Penerimaan Email (POP3)
Setelah daftar peristiwa (*Event List*) dibersihkan, pengujian pengambilan pesan dilakukan dari `PC1` melalui tombol **Receive**, yang berlangsung selama **`0.000s` hingga `0.015s`**:

1. **Pembentukan Koneksi TCP (`0.000s – 0.004s`):**
   * `PC1` mengirimkan permintaan koneksi `TCP` via `Switch0` ke `Gooaya Server` (`192.168.1.5`), yang kemudian direspons balik oleh server.
2. **Pengambilan Pesan Menggunakan POP3 (`0.004s – 0.009s`):**
   * `PC1` menjalankan protokol **POP3** untuk meminta daftar dan isi pesan baru dari `Gooaya Server`.
   * `Gooaya Server` memproses permintaan autentikasi dan mengirimkan paket data email kembali ke `PC1` melalui `Switch0`.
3. **Konfirmasi & Penutupan Sesi (`0.009s – 0.015s`):**
   * `PC1` dan `Gooaya Server` bertukar paket konfirmasi `TCP` (`ACK`) sebelum menutup sesi komunikasi pengunduhan pesan.

---

## ✅ Hasil Verifikasi

| Skenario Pengujian | Parameter Uji | Hasil Observasi |
| :--- | :--- | :--- |
| **Registrasi Akun Mail** | User `nicho` pada `ComNet Server` (`compnet.com`) | Akun tersimpan aktif dan dapat digunakan untuk autentikasi klien `PC2`. |
| **Pengiriman Lintas Domain** | `nicho@compnet.com` ➔ `miau@gooaya.com` | Resolusi DNS berhasil dan pesan diteruskan via SMTP tanpa *packet loss*. |
| **Penerimaan Pesan Klien** | Klik `Receive` pada `Mail Browser` `PC1` | Email dengan subjek **`Tes Simulation`** dan isi **`ini tes simulation kirim email`** tampil di kotak masuk `PC1`. |
