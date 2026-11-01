# FreshTrack Dashboard

Dashboard web real-time untuk memantau **sisa umur simpan produk** pada sistem cold chain. Dashboard ini adalah pasangan dari project simulasi ESP32 di Wokwi (**Dynamic Shelf-Life Tracker + MQTT**): perangkat mengirim telemetri ke broker HiveMQ, dan dashboard ini menampilkannya langsung di browser serta dapat mengirim perintah balik ke perangkat.

Dashboard berupa satu file `index.html` tanpa proses build dan tanpa backend.

## Fitur

- Tampilan sisa umur simpan (hari dan jam) beserta persentase dan *progress bar*
- Status produk berwarna: Aman, Waspada, Jual segera, Kedaluwarsa
- Kartu suhu, kelembapan, dan laju susut
- Grafik garis suhu dan sisa umur simpan (90 titik terakhir, sekitar 3 menit)
- Indikator koneksi broker dan status perangkat (online, offline, atau tidak ada data baru)
- Tombol kontrol jarak jauh: **Batch baru** dan **Ganti produk**
- Mode terang dan gelap, mengikuti pengaturan sistem dan tersimpan di browser
- Tampilan responsif untuk layar ponsel

## Cara Kerja

```
ESP32 (Wokwi)  --MQTT/TCP 1883-->  broker.hivemq.com  --MQTT/WSS 8884-->  Dashboard (browser)
      ^                                                                          |
      +------------------------ topik <BASE>/cmd <------------------------------+
```

Dashboard terhubung ke broker publik HiveMQ melalui WebSocket aman (`wss://broker.hivemq.com:8884/mqtt`) menggunakan library [MQTT.js](https://github.com/mqttjs/MQTT.js) v5.10.1 yang dimuat dari CDN.

### Topik MQTT

| Topik | Arah (dari sisi dashboard) | Isi |
|-------|:--------------------------:|-----|
| `<BASE>/data` | Subscribe | JSON telemetri, dikirim perangkat tiap 2 detik |
| `<BASE>/status` | Subscribe | `online` atau `offline` (retained, via Last Will) |
| `<BASE>/cmd` | Publish | Perintah `reset` (batch baru) atau `ganti` (ganti produk) |

### Format Data

```json
{
  "produk": "Susu Segar",
  "suhu": 8.5,
  "lembap": 60.0,
  "laju": 1.37,
  "sisa_jam": 150.2,
  "persen": 89.4,
  "status": "AMAN"
}
```

### Status Produk

| Status di data | Label di dashboard | Kondisi (sisa umur) |
|----------------|--------------------|:-------------------:|
| `AMAN` | Aman | 50% atau lebih |
| `WASPADA` | Waspada | 20% sampai 50% |
| `JUAL` | Jual segera | kurang dari 20% |
| `EXP` | Kedaluwarsa | habis |

## Konfigurasi

Konfigurasi ada di bagian `<script>` pada `index.html`:

```js
const TOPIC_BASE = "scm/freshtrack/kelompokX";
const BROKER_URL = "wss://broker.hivemq.com:8884/mqtt";
const MAX_POINTS = 90;      // jumlah titik pada grafik
const STALE_MS   = 7000;    // batas waktu dianggap tidak ada data baru (ms)
```

> **Penting:** nilai `TOPIC_BASE` harus **sama persis** dengan `TOPIC_BASE` di `sketch.ino` pada project Wokwi. Broker HiveMQ bersifat publik, jadi ganti `kelompokX` dengan nama unik kelompokmu agar data tidak bercampur dengan pengguna lain.

## Cara Menjalankan

### 1. Jalankan simulasi Wokwi

Buka project Wokwi, pastikan `TOPIC_BASE` sudah diganti, lalu klik **Start Simulation**. Tunggu sampai Serial Monitor menampilkan `berhasil` pada koneksi MQTT.

### 2. Jalankan dashboard

Pilih salah satu cara:

**Lokal dengan npm** (butuh Node.js):

```bash
npm run dev
```

Perintah ini menjalankan `npx serve .`. Buka alamat yang ditampilkan di terminal, biasanya `http://localhost:3000`.

**Langsung dari berkas:** buka `index.html` di browser.

**GitHub Pages:** aktifkan Pages pada repositori (Settings, Pages, pilih branch `main` dan folder root). Karena Pages memakai HTTPS, koneksi `wss://` ke broker tetap berjalan normal.

### 3. Pantau dan kontrol

Setelah data pertama diterima, dashboard menampilkan produk, sisa umur simpan, dan grafik. Ubah suhu pada sensor DHT22 di Wokwi untuk melihat laju susut dan status berubah. Tombol **Batch baru** dan **Ganti produk** pada dashboard memiliki fungsi yang sama dengan tombol fisik di rangkaian.

## Struktur Repositori

```
.
├── index.html          # Seluruh dashboard (HTML, CSS, JavaScript)
├── package.json        # Skrip npm untuk server lokal
├── package-lock.json
└── README.md
```

## Teknologi

- HTML, CSS, dan JavaScript murni (tanpa framework)
- [MQTT.js](https://github.com/mqttjs/MQTT.js) untuk koneksi MQTT over WebSocket
- Grafik dibuat dengan SVG inline
- Font Inter dari Google Fonts

## Pemecahan Masalah

| Gejala | Kemungkinan penyebab |
|--------|----------------------|
| Indikator broker tidak menjadi "tersambung" | Tidak ada internet, atau jaringan memblokir port 8884 |
| Broker tersambung tetapi data tidak muncul | Simulasi Wokwi belum berjalan, atau `TOPIC_BASE` berbeda antara dashboard dan `sketch.ino` |
| Status perangkat "offline" | Simulasi dihentikan, atau ESP32 belum berhasil terhubung ke MQTT |
| Status "Tidak ada data baru" | Perangkat online tetapi tidak mengirim data lebih dari 7 detik |
| Tombol kontrol tidak berpengaruh | Broker belum tersambung, atau topik `cmd` tidak cocok dengan perangkat |

## Project Terkait

Kode ESP32 dan rangkaian simulasi: [Dynamic Shelf-Life Tracker di Wokwi](https://wokwi.com/projects/476959827586961409).

## Tim

Dikerjakan oleh **Kelompok SCM** sebagai mini project mata kuliah Supply Chain Management.

## Lisensi

Project ini dibuat untuk keperluan pembelajaran. Tambahkan lisensi (misalnya MIT) sesuai kebutuhan.
