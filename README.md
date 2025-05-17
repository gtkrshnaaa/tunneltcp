# **tunneltcp: Static Port TCP Tunnel over VPS**

## Deskripsi

**TunnelTCP** adalah sistem tunneling TCP sederhana yang memungkinkan aplikasi yang berjalan secara lokal (misalnya di laptop atau komputer pribadi) untuk diakses secara publik melalui VPS. Sistem ini menggunakan pendekatan **one tunnel = one TCP connection** (**satu koneksi per tunnel**), di mana setiap aplikasi lokal akan dihubungkan ke port tertentu di VPS melalui satu koneksi TCP tetap.

Tidak seperti solusi seperti Ngrok yang dinamis, TunnelTCP menetapkan port yang konsisten, memungkinkan kontrol penuh terhadap manajemen akses, keamanan, serta integrasi dengan proxy eksternal seperti Nginx.

---

## Tujuan

Proyek ini dibuat untuk:

* Menyediakan solusi tunneling ringan dan stabil menggunakan port tetap.
* Mendukung penggunaan banyak tunnel secara bersamaan (skala kecil hingga menengah).
* Memfasilitasi remote access terhadap aplikasi lokal tanpa perlu IP publik.
* Memberikan kontrol penuh atas konfigurasi koneksi tanpa ketergantungan ke layanan pihak ketiga.

---

## Arsitektur Sistem

### Komponen Utama

* **Client Tunnel**
  Program CLI yang berjalan di komputer lokal, bertugas membuat koneksi ke VPS dan menghubungkan ke aplikasi lokal.

* **Server Tunnel**
  Program yang berjalan di VPS, mendengarkan port-port TCP tertentu, menerima koneksi dari user/public, dan meneruskan ke client tunnel.

* **Aplikasi Lokal**
  Web server atau service lain yang hanya berjalan di `localhost` (misal: port 8000, 8001, dst).

* **User/Public**
  Klien yang mengakses aplikasi melalui `vps_ip:port`.

---

## Alur Koneksi

1. Server Tunnel di VPS membuka banyak port publik (misal 60001 ke atas).
2. Client Tunnel menginisiasi koneksi TCP ke VPS di masing-masing port.
3. Client Tunnel juga membuka koneksi ke aplikasi lokal (misalnya `localhost:8000`).
4. Ketika user/public mengakses `vps_ip:60001`, server menerima koneksi tersebut.
5. Server Tunnel meneruskan data ke koneksi Client Tunnel yang sudah ada.
6. Client Tunnel menghubungkan ke aplikasi lokal dan meneruskan data dua arah.

---

## Port Mapping

| VPS Port | Local Port | Keterangan              |
| -------- | ---------- | ----------------------- |
| 60001    | 8000       | Untuk aplikasi lokal #1 |
| 60002    | 8001       | Untuk aplikasi lokal #2 |
| 60003    | 8002       | Untuk aplikasi lokal #3 |

Setiap port publik hanya digunakan untuk satu aplikasi, dan setiap tunnel menggunakan **satu koneksi TCP tetap**.

---

## Struktur Direktori Proyek

```
tunneltcp/
├── client/
│   ├── main.c
│   ├── tunnel.c
│   ├── tunnel.h
│   ├── config.json
│   └── utils.c
├── server/
│   ├── main.c
│   ├── listener.c
│   ├── listener.h
│   └── utils.c
├── common/
│   ├── log.c
│   └── log.h
├── Makefile
└── README.md
```

---

## Konfigurasi

Client Tunnel membaca file konfigurasi seperti berikut:

```json
[
  { "local_port": 8000, "remote_port": 60001 },
  { "local_port": 8001, "remote_port": 60002 }
]
```

Konfigurasi ini menentukan hubungan antara port aplikasi lokal dan port publik di VPS.

---

## Fitur Sistem

* Koneksi TCP tetap antara client dan server untuk setiap tunnel.
* Port publik yang konsisten dan mudah dikontrol.
* Logging status koneksi.
* Reconnect otomatis saat koneksi client ke VPS terputus.
* Graceful shutdown.
* Bisa dijalankan dalam banyak instance secara paralel.
* Logging ke file dengan sistem rotasi.

---

## Rencana Pengembangan

* **TLS Support:** enkripsi koneksi antar client dan server.
* **Authentication Token:** validasi client saat terhubung ke server.
* **Monitoring Panel:** dashboard ringan untuk melihat status tunnel yang aktif.
* **Proxy Integrasi:** integrasi dengan Nginx untuk dukungan domain & SSL.
* **Manajemen Dinamis:** endpoint untuk menambahkan/menghapus tunnel secara real-time.

---

## Evaluasi & Pengujian

Pengujian dilakukan pada beberapa skenario:

### 1. Skenario Normal

* 3 aplikasi lokal diakses melalui 3 tunnel paralel.
* Sistem stabil selama >2 jam tanpa reconnect.

### 2. Skalabilitas

* 50 tunnel aktif secara paralel.
* Konsumsi memori dan CPU tetap dalam batas normal.
* Socket tidak bocor.

### 3. Kegagalan

* Simulasi putus koneksi internet.
* Koneksi client auto-reconnect dalam waktu kurang dari 5 detik.
* Server menangani koneksi baru tanpa restart.

### 4. Latency

* Rata-rata latency: \~15–40ms tergantung jarak ke VPS.
* Throughput maksimal: 100+ req/s per tunnel (tergantung aplikasi lokal).

---

## Kebutuhan Sistem

* **VPS:**

  * Minimal 1 core, 512MB RAM.
  * Akses root untuk membuka port TCP.
  * IP publik wajib tersedia.

* **Firewall:**

  * Port 60001–60100 (atau sesuai kebutuhan) harus dibuka di VPS.

---

## Lisensi

Open source, lisensi MIT. Bebas digunakan, dimodifikasi, dan disebarluaskan.

---

## Kontribusi

Saat ini repository belum sepenuhnya terbuka untuk kontribusi eksternal. Pull request dan kolaborasi dari komunitas **mungkin akan dibuka di masa mendatang**, terutama setelah fase awal pengembangan selesai.

Silakan pantau perubahan pada bagian ini atau file `CONTRIBUTING.md` untuk informasi lebih lanjut. Jika bagian ini sudah diperbarui, berarti kontribusi eksternal sudah mulai diterima.

---
