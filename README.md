# cisco-email-server-simulation
Perancangan, konfigurasi, dan simulasi infrastruktur Email Server lintas domain (SMTP, POP3, DNS, dan TCP) menggunakan Cisco Packet Tracer beserta analisis pertukaran paket.
# 📧 Cross-Domain Email Server Configuration & Packet Analysis (Cisco Packet Tracer)

![Cisco Packet Tracer](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-1BA0D7?style=flat-square&logo=cisco)
![Protocols](https://img.shields.io/badge/Protocols-SMTP%20%7C%20POP3%20%7C%20DNS%20%7C%20TCP-success?style=flat-square)
![Architecture](https://img.shields.io/badge/Architecture-Cross--Domain%20Mail-orange?style=flat-square)

Proyek perancangan, konfigurasi, dan simulasi infrastruktur **Email Server** lintas domain (`compnet.com` dan `gooaya.com`) menggunakan **Cisco Packet Tracer**. Proyek ini berfokus pada integrasi layanan **SMTP**, **POP3**, dan **DNS** serta analisis mendalam terhadap alur pertukaran paket pada setiap lapisan protokol saat proses pengiriman dan penerimaan email berlangsung.

---

## 🌐 Topologi & Arsitektur Jaringan

Jaringan dirancang menggunakan topologi *star* yang berpusat pada satu switch (`2950-24 Switch0`) untuk menghubungkan seluruh perangkat klien dan infrastruktur server dalam satu segmen lokal:

```text
                       [CompNet Server]
                        (192.168.1.6)
                              |
      [PC0] -------+          |          +------- [First Server]
                   |          |          |
      [PC1] -------+--- [ Switch0 ] -----+------- [Second Server]
                   |     (2950-24)       |
      [PC2] -------+          |          +------- [DNS Server]
                              |
                       [Gooaya Server]
                        (192.168.1.5)
```

### Komponen Infrastruktur
* **End Devices (Clients):** `PC0`, `PC1`, dan `PC2`.
* **Network Switch:** `2950-24 Switch0`.
* **Servers:** `CompNet Server` (terlabel `ComNet Server`), `Gooaya Server`, `DNS Server`, `First Server`, dan `Second Server`.

---

## ⚙️ Spesifikasi & Konfigurasi

### 1. Konfigurasi Mail Server (`CompNet Server`)

| Parameter | Nilai Konfigurasi |
| :--- | :--- |
| **Mode IPv4** | Static |
| **IPv4 Address** | `192.168.1.6` |
| **Subnet Mask** | `255.255.255.0` |
| **Default Gateway** | `0.0.0.0` |
| **DNS Server** | `0.0.0.0` |
| **SMTP Service** | `ON` |
| **POP3 Service** | `ON` |
| **Domain Name** | `compnet.com` |
| **Akun Terdaftar** | User: `nicho` \| Password: `11111` |

### 2. Konfigurasi Email Client

| Parameter | Client Pengirim (`PC2`) | Client Penerima (`PC1`) |
| :--- | :--- | :--- |
| **Your Name** | `nicho` | `miau` |
| **Email Address** | `nicho@compnet.com` | `miau@gooaya.com` |
| **Incoming Mail Server** | `192.168.1.6` | `192.168.1.5` |
| **Outgoing Mail Server** | `192.168.1.6` | `192.168.1.5` |
| **User Name** | `nicho` | `miau` |

---

## 🛠️ Tahapan Implementasi

1. **Integrasi Perangkat ke Topologi**
   * Menambahkan node `Server-PT` baru sebagai **CompNet Server** dan menghubungkannya ke **Switch0** (`2950-24`).
2. **Pengalamatan IP Statis**
   * Mengatur antarmuka jaringan **CompNet Server** pada mode **Static** dengan alamat IPv4 `192.168.1.6` dan Subnet Mask `255.255.255.0`.
3. **Deployment Layanan Email & Manajemen Akun**
   * Mengaktifkan layanan **SMTP** dan **POP3** pada menu **Services → EMAIL** di **CompNet Server**.
   * Mendaftarkan domain `compnet.com` serta membuat kredensial pengguna `nicho` (`11111`).
4. **Provisioning Email Client**
   * Mengonfigurasi profil email pengirim pada **PC2** (`nicho@compnet.com` mengarah ke server `192.168.1.6`) dan memvalidasi profil penerima pada **PC1** (`miau@gooaya.com` mengarah ke server `192.168.1.5`).

---

## 🔬 Pengujian & Analisis Alur Paket

Pengujian dilakukan menggunakan **Simulation Mode** pada Cisco Packet Tracer untuk mengamati mekanisme kerja protokol secara *end-to-end*.

### Skenario 1: Pengiriman Email Lintas Domain (`PC2` → `PC1`)

* **Payload Pengujian:**
  * **To:** `miau@gooaya.com`
  * **Subject:** `Tes Simulation`
  * **Body:** `ini tes simulation kirim email`

* **Analisis Alur Paket (Detik `0.000` – `0.026`):**
  1. **Pembentukan Koneksi Awal TCP (`0.000` – `0.004` detik):** `PC2` menginisiasi sesi koneksi TCP menuju `CompNet Server` melalui `Switch0` untuk membuka jalur komunikasi pengiriman pesan.
  2. **Resolusi Nama Domain via DNS (`0.007` – `0.011` detik):** Karena alamat tujuan berada pada domain berbeda (`gooaya.com`), `CompNet Server` mengirimkan *DNS query* ke `DNS Server` melalui `Switch0` untuk memetakan nama domain tujuan ke alamat IP-nya (`192.168.1.5`).
  3. **Transmisi Pesan via SMTP (`0.004` – `0.020` detik):** Protokol **SMTP** (*Simple Mail Transfer Protocol*) berjalan di atas sesi TCP untuk mengirim pesan dari `PC2` ke `CompNet Server`, kemudian meneruskannya ke `Gooaya Server` sebagai server penerima.
  4. **Terminasi Sesi TCP (`0.020` – `0.026` detik):** Setelah seluruh data email berhasil diteruskan dan dikonfirmasi, kedua server dan klien saling bertukar paket TCP penutup untuk mengakhiri sesi dengan aman.

### Skenario 2: Pengambilan Email oleh Penerima (`PC1` ← `Gooaya Server`)

* **Analisis Alur Paket (Detik `0.000` – `0.015`):**
  1. **Pembentukan Koneksi TCP (`0.000` – `0.004` detik):** Ketika tombol **Receive** dijalankan pada `PC1`, klien melakukan *TCP handshake* dengan `Gooaya Server` melalui `Switch0`.
  2. **Sinkronisasi & Unduh Pesan via POP3 (`0.004` – `0.009` detik):** Protokol **POP3** (*Post Office Protocol version 3*) digunakan oleh `PC1` untuk mengautentikasi akun `miau` dan mengunduh pesan baru yang tersimpan pada `Gooaya Server`.
  3. **Konfirmasi & Penutupan Sesi TCP (`0.009` – `0.015` detik):** `PC1` mengirimkan paket TCP konfirmasi penerimaan (`ACK`) dan menutup koneksi POP3 setelah pesan tersimpan di kotak masuk lokal.

---

## ✅ Hasil Verifikasi

Simulasi berjalan dengan tingkat keberhasilan **100%**. Email dari `nicho@compnet.com` berhasil diterima pada *Mail Browser* klien penerima di **PC1**:

```text
From    : nicho@compnet.com
Subject : Tes Simulation
Sent    : Mon Sep 14 2026 16:37:17
Message : ini tes simulation kirim email
```

---

## 👩‍💻 Author

**Valencia Theodora**  
[LinkedIn](https://www.linkedin.com/in/valencia-theodora) | [GitHub](https://github.com/valenciatheodora07-tech)
