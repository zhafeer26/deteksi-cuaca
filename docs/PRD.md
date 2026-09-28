# PRD — Aplikasi "Kapan Bisa Jemur?"

Produk pencari tahu cuaca khusus laundry. Menjawab satu pertanyaan: **"Kapan saya bisa jemur cucian di luar hari ini, dan kapan tidak?"**

| Bidang | Nilai |
|---|---|
| Status | v1 (draf untuk persetujuan) |
| Tanggal | 28 September 2026 |
| Penulis | Owner laundry — Ngampel, Mojoroto, Kota Kediri |
| Sumber data | BMKG (utama) + Open-Meteo (pelengkap opsional) |
| Bentuk produk | 1 file `index.html`, 100% client-side, nol backend |
| Biaya | Rp 0 (hanya waktu & data internet HP) |

---

## 1. Ringkasan Eksekutif

Owner laundry di Kelurahan Ngampel perlu memutuskan setiap hari: **cucian jadi dijemur di luar sekarang, atau tidak.** Keputusan ini sekarang bergantung pada tebakan, membuka 2–3 aplikasi cuaca yang datanya saling bertolak belakang, dan inspeksi manual langit — yang sering gagal menangkap hujan lokal khas Kediri. Akibat salah tebak, cucian setengah kering kena hujan dan harus dicuci ulang: kehilangan waktu, energi, air, deterjen, dan jadwal antar.

Produk ini menggabungkan prakiraan dari **BMKG** (resmi, gratis, akurat untuk level kelurahan) dan **Open-Meteo** (per jam, 7 hari) menjadi satu tampilan ringkas di HP: **hijau/kuning/merah per jam, ditambah jam terbaik menjemur per hari.** Tanpa akun, tanpa server, tanpa biaya — cukup buka satu file HTML.

Keputusan utama yang sudah diambil:

| Keputusan | Pilihan | Alasan |
|---|---|---|
| Sumber data | BMKG + Open-Meteo | BMKG resmi tanpa batasan komersial; Open-Meteo memperhalus jadi per jam & menambah peluang hujan |
| Format | 1 file HTML statis | Nol biaya, sederhana, bisa dibagikan |
| Notifikasi | **TIDAK termasuk v1** | Butuh HTTPS + Service Worker; `file://` tidak mendukung. Batasan platform, bukan kelalaian |
| "Jam terbaik menjemur" | YA, 1 baris per kartu hari | Langsung menjawab pertanyaan inti user |

---

## 2. Masalah & Persona

### 2.1 Persona

- **Nama**: Owner laundry kecil-menengah
- **Lokasi operasional**: Kelurahan Ngampel, Kecamatan Mojoroto, Kota Kediri (kode wilayah BMKG `35.71.01.1010`, koordinat `-7.7933, 111.9955`, WIB `+0700`)
- **Perangkat**: HP Android, koneksi internet seluler biasa (kadang lemah), tidak ada alat pendeteksi cuaca (ANEMOMETER/weather station tidak dimiliki)
- **Konteks**: Jemur berlangsung di halaman terbuka. Kapasitas jemur terbatas dan sequential — cucian batch berikutnya baru bisa dijemur setelah batch sebelumnya kering. Satu hari jemur gagal = bottleneck antrian pickup.

### 2.2 Masalah spesifik (bukan "cuaca umum")

1. **Tebakan vs fakta**: keputusan jemur bertumpu pada perkiraan dan insting belaka; dalam jangka panjang merugikan konsistensi layanan.
2. **Kontradiksi antar-aplikasi**: aplikasi cuaca umum menampilkan suhu kota, bukan kondisi mikro kelurahan; dua aplikasi bisa menyebut "cerah" dan "hujan" bersamaan untuk lokasi yang sama.
3. **Tidak ada "jam optimal"**: user tahu hujan sering datang sore hari, tapi tidak tahu jendela 9.00–14.00 adalah jendela kering optimal — karena tidak ada aplikasi yang menyajikannya untuk keputusan jemur.
4. **Absennya instrumen**: BMKG/Open-Meteo adalah pengganti sensor yang paling murah dan andal yang tersedia; masalahnya bukan akses data, tapi **interpretasi** yang buruk. Produk ini menyelesaikan interpretasinya, bukan datanya.

### 2.3 Nilai yang dijanjikan

- Keputusan jemur dalam **< 10 detik** dari buka aplikasi.
- **Satu rujukan** untuk memutuskan, bukan 3 aplikasi yang saling bertikai.
- Pengurangan cucian yang kena hujan (klaim kualitas: "lebih sedikit cucian gagal").
- Nol biaya bulanan. Data terakhir tersimpan di HP (work offline setelah dibuka sekali).

---

## 3. Lingkup v1

### 3.1 Fitur masuk

| # | Fitur | Bidang yang diselesaikan |
|---|---|---|
| F1 | **Kartu "Sekarang"** — status besar + suhu + peluang hujan + alasan + ringkasan 3 jam ke depan | Keputusan saat ini, readable sekilas |
| F2 | **Strip 24 jam** — chip warna per jam (hijau/kuning/merah/abu), gulir horizontal, berisi jam lokal hari ini | "Jam berapa aman?" |
| F3 | **Kartu 7 hari** — nama hari, ikon, min/max suhu, peluang hujan max, total hujan, **jam terbaik menjemur** | Perencanaan dan jadwal antar |
| F4 | **Simpan lokasi** — sekali atur, tersimpan di HP (localStorage); tidak perlu pilih ulang | Pengalaman penggunaan harian |
| F5 | **Toggle Open-Meteo** — bisa dimatikan per lokasi | Mengakomodasi keputusan lisensi; degradasi otomatis |
| F6 | **Explainability** — setiap warna/status wajib menampilkan bukti angka yang mendukung (lihat §8) | Melawan risiko over-trust pada ramalan |
| F7 | **Mode offline/degradasi** — cache data terakhir + banner status | Kelangsungan penggunaan saat sinyal lemah |

### 3.2 Fitur yang DIBUANG dari v1 (dengan alasan)

| Fitur | Alasan pembuangan |
|---|---|
| Notifikasi/push | Butuh HTTPS + Service Worker + (untuk push penuh) server. Tidak jalan dari `file://`. Ditandai sebagai calon v2 |
| Rekomendasi "skor jemur 0–100" | Overkill; user memilih status sederhana. Diganti baris "jam terbaik" yang ringan |
| Riwayat cucian / tracking | Butuh persistensi antar-sesi & data berulang; menambah kompleksitas tanpa value inti |
| Multi-lokasi / cabang | Struktur `localStorage` sudah mengizinkan 1 lokasi aktif; multi-lokasi ditunda v2 |
| Autentikasi / akun | Tidak perlu untuk 1 user |
| IoT / sensor | Di luar lingkup — API cuaca adalah pengganti sensor |

---

## 4. Non-Goal (batas eksplisit v1)

- **TIDAK** ada notifikasi dan push v1.
- **TIDAK** ada janji akurasi menit-ke-menit. Ramalan adalah probabilitas; aplikasi wajib menampilkan ketidakpastian.
- **TIDAK** ada server, database, atau akun.
- **TIDAK** ada jaminan uptime dari pihak ketiga (BMKG/Open-Meteo adalah gratis dan tanpa SLA).
- **TIDAK** menggunakan ikon BMKG dari URL eksternal (demi 1 file + offline); ikon dipakai SVG inline.

---

## 5. Arsitektur

### 5.1 Alur data (semua di browser, nol backend)

```
┌──────────────────────────────────────────────────────────┐
│  index.html (1 file, self-contained)                     │
│                                                          │
│  baca config lokasi dari localStorage                     │
│        │                                                  │
│        ▼                                                  │
│  fetch BMKG (api.bmkg.go.id, adm4)  ────────┐             │
│  fetch Open-Meteo (lat/lon, 7 hari)  ───────┤ Promise     │
│        │                                    │ allSettled  │
│        ▼                                    ▼             │
│  normalisasi → 1 timeline per jam zona WIB                │
│        │                                                  │
│        ▼                                                  │
│  mesin status → per jam: HIJAU/KUNING/MERAH/ABU + alasan   │
│        │                                                  │
│        ▼                                                  │
│  hitung "jam terbaik" per hari (jendela cerah terpanjang) │
│        │                                                  │
│        ▼                                                  │
│  render mobile-first + simpan cache di localStorage       │
└──────────────────────────────────────────────────────────┘
```

- `Promise.allSettled`: satu API gagal tidak mematikan yang lain.
- Normalisasi waktu: Open-Meteo diberi `timezone=Asia/Jakarta` sehingga sudah WIB. BMKG punya field `local_datetime` (WIB). **Tidak ada hitung manual jam UTC** di kode.
- Layer API diisolasi dalam 2 modul pemetakan (BMKG & Open-Meteo) → kalau salah satu format JSON berubah, hanya modul itu yang di-patch.

### 5.2 Prinsip desain

1. **Explainable by default**: setiap chip warna menyertakan minimal satu bukti (angka/hari/jam sumber).
2. **Gratis secara default**: dua API gratis; cache agresif agar kuota tidak terbuang.
3. **Satu file**: tanpa build step, tanpa dependensi eksternal (semua CSS/JS inline).
4. **Degradasi elegan**: dari detail per jam → per 3 jam → cache lama → pesan error yang jelas; tidak pernah halaman putih.

---

## 6. Kontrak Data & Sumber Daya

### 6.1 BMKG — sumber utama (resmi pemerintah, lisensi bebas untuk komersial, wajib atribusi)

Akses (tanpa API key, batas 60 req/menit/IP):

```
GET https://api.bmkg.go.id/publik/prakiraan-cuaca?adm4=35.71.01.1010
```

Struktur dan field yang dipakai (sudah diverifikasi tanggapan nyata tanggal 28 Sept 2026):

```
lokasi      → adm4, provinsi, kotkab, kecamatan, desa, lat, lon, timezone
data[].cuaca[][]   → hari → slot per 3 jam
  local_datetime   → waktu lokal WIB (untuk timeline)
  t                → suhu °C
  hu               → kelembapan %
  tp               → peluang hujan %
  tcc              → tutupan awan %
  ws               → kecepatan angin km/jam
  wd               → arah angin (kode + teks)
  weather_desc     → kondisi cuaca bahasa Indonesia (dipakai untuk verifikasi silang)
  weather_desc_en  → kondisi cuaca Inggris
  time_index       → rentang jam yang dicakup slot (mis. "2-3" = jam 02–05 WIB)
  analysis_date    → waktu produksi model (menentukan kesegaran data)
```

Karakteristik: prakiraan **3 hari ke depan**, **per 3 jam**, diperbarui **2×/hari**. TERCATAT: data tidak selalu "sekarang" — bisa berumur hingga 3 jam atau setengah hari. Aplikasi harus menampilkan `analysis_date` sebagai label kesegaran.

### 6.2 Open-Meteo — pelengkap opsional (per jam, 7 hari)

Akses (tanpa API key, ToS: gratis untuk non-komersial — lihat §13.2; CC BY 4.0, wajib atribusi):

```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=-7.7933&longitude=111.9955
  &timezone=Asia%2FJakarta
  &forecast_days=7
  &hourly=temperature_2m,relative_humidity_2m,dew_point_2m,precipitation,precipitation_probability,rain,weather_code,cloud_cover,wind_speed_10m,wind_gusts_10m,uv_index,apparent_temperature
  &daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_sum,precipitation_probability_max,wind_speed_10m_max,uv_index_max,sunrise,sunset
```

Field yang dipakai:

```
hourly.*  → 168 titik (7 hari × 24 jam), sudah dalam WIB
  temperature_2m, relative_humidity_2m, dew_point_2m
  precipitation (mm/jam), rain (mm/jam)
  precipitation_probability (%)   ← dasar status merah/kuning
  weather_code (standar WMO)      ← status; sering jadi faktor penentu
  cloud_cover (%), wind_speed_10m, wind_gusts_10m
  uv_index (informasi perawatan)
daily.*   → 7 hari
  sunrise, sunset                  ← jendela siang untuk status "malam"
  temperature_2m_max/min, precipitation_sum, precipitation_probability_max
  uv_index_max
```

### 6.3 Strategi gabungan & urutan prioritas

1. Jam siang / per jam → **Open-Meteo** (karena per jam + peluang hujan).
2. Verifikasi silang cuaca → **BMKG** `weather_desc`. Bila keduanya bertentangan tentang hujan (Open-Meteo "cerah" vs BMKG "hujan"), jam itu di-mark **KUNING + catatan "data sumber berbeda"** — dengan alasan ditampilkan (lihat §8).
3. Open-Meteo mati / toggle OFF → degradasi ke **BMKG-only**, granularitas 3 jam (nilai per jam = nilai slot 3 jam yang mencakup jam tersebut), dengan banner peringatan (§9).
4. Keduanya gagal → **cache terakhir** berlabel waktu + banner error (§9).

### 6.4 Dasar cuaca yang tidak dipakai (dan alasannya)

| Variabel | Pemakaian |
|---|---|
| `dew_point_2m` | Info saja (kenyamanan pengeringan) — bukan penentu status |
| `apparent_temperature` | Info saja |
| `wind_speed_10m` | DITAMPILKAN sebagai info (angin mempercepat kering), tapi bukan penentu status karena di Indonesia faktor penentu kering adalah matahari & kelembapan, bukan angin |
| `uv_index` | Info "bahaya jemur lama" untuk perawatan pakaian |

---

## 7. Mesin Status (otak aplikasi)

### 7.1 Kelas status per jam

| Kelas | Warna | Makna operasional untuk laundry |
|---|---|---|
| CERAH | Hijau | Aman dijemur: matahari cukup, kelembapan rendah |
| MENDUNG | Kuning | Berisiko: langit tertutup / udara lembap / kemungkinan hujan menengah — jemur dengan pengawasan |
| HUJAN | Merah | Tidak layak: hujan aktif atau peluang tinggi — angkat cucian |
| MALAM | Abu-abu | Di luar jam matahari (tanpa matahari, proses kering hampir berhenti); netral |

### 7.2 Aturan klasifikasi (ambang default; dapat disetel via konfigurasi)

**HUJAN (Merah)** — salah satu dari:
- `precipitation > 0,3 mm/jam`, ATAU
- `precipitation_probability >= 60%`, ATAU
- kode WMO ∈ {51,53,55,56,57,61,63,65,66,67,80,81,82,95,96,99} (gerimis/hujan/badai/hujan es), ATAU
- BMKG `tp >= 50%`, ATAU
- BMKG `weather_desc` mengandung kata «hujan» (hujan, gerimis, badai, petir)

**MENDUNG (Kuning)** — salah satu dari:
- `cloud_cover >= 65%`, ATAU
- `relative_humidity >= 82%` **pada jam siang** (pagi/malam tidak), ATAU
- `precipitation_probability` 20–59%, ATAU
- kode WMO ∈ {1,2,3,45,48} (berawan/kabut), ATAU
- **Konflik dua sumber** tentang hujan (§6.3 poin 2)

**CERAH (Hijau)** — sisanya, DAN jam masuk rentang `sunrise`–`sunset` (dari Open-Meteo).

**MALAM (Abu-abu)** — jam di luar `sunrise`–`sunset`, tanpa kondisi HUJAN aktif (hujan malam tetap merah, karena cucian terpapar).

Prioritas evaluasi: **MALAM(malam hujan → tetap merah)** > **HUJAN** > **MENDUNG** > **CERAH**.

### 7.3 Pemetaan kode WMO → bahasa Indonesia (untuk tampilan)

| Kode | Deskripsi tampilan |
|---|---|
| 0 | Cerah |
| 1 | Cerah berawan |
| 2 | Berawan sebagian |
| 3 | Berawan / mendung |
| 45, 48 | Kabut |
| 51, 53, 55 | Gerimis |
| 61, 63, 65 | Hujan ringan / sedang / lebat |
| 80, 81, 82 | Hujan deras (shower) |
| 95 | Badai guntur |
| 96, 99 | Badai + hujan es |
| lainnya (salju dll) | Langka di Ngampel; ditampilkan apa adanya |

### 7.4 "Jam Terbaik Menjemur" (F3, per kartu hari)

Algoritma sederhana, tanpa skor bobot:

1. Kumpulkan jam-jam dalam rentang `sunrise`–`sunset` (per hari).
2. Cari **jendela kontinu terlama** berstatus CERAH.
3. Jika panjang jendela ≥ 2 jam → tampilkan `"Jam terbaik: 09.00–15.00"`.
4. Jika tidak ada jendela CERAH ≥ 2 jam → tampilkan `"Hari ini tidak ada jendela layak jemur"` (atau jendela terbaik terpendek disertai peringatan).
5. Tampilkan juga `"3 jam ke depan: C/R/M"` singkat di kartu Sekarang.

Contoh pengembangan: bila mapping dapat disetel user (mis. butuh ≥ 4 jam untuk bed lin.), simpan preferensi di config lokal — di v1 ambang ini dikunci di 2 jam.

### 7.5 Aturan explainability (WAJIB, F6)

Setiap chip/kartu **harus** menampilkan minimal satu bukti pendukung, format: `[kondisi] karena [bukti]`.

Contoh yang benar:
- 🟢 Cerah — awan 12%, peluang hujan 5%
- 🟡 Mendung — kelembapan 86%, awan 92% (2 sumber data berbeda)
- 🔴 Hujan — peluang hujan 75% (Open-Meteo) & BMKG: "Hujan Sedang"

Contoh yang TIDAK boleh:
- 🟢 Cerah (tanpa keterangan angka)

---

## 8. (Digabung ke §7.5) Explainability — lihat di atas

---

## 9. Cache, Offline & Degradasi

### 9.1 Cache di localStorage

| Cache | TTL | Tujuan |
|---|---|---|
| `jemur.cache.fused` (hasil gabungan siap-tampil) | 30 menit | Hemat kuota API & kecepatan render |
| `jemur.cache.bmkg` | 1 jam | Backup degradasi |
| `jemur.cache.openmeteo` | 30 menit | Backup degradasi |
| `jemur.config` | permanen | Lokasi & preferensi (tanpa TTL) |

### 9.2 Tiga tingkat degradasi

| Tingkat | Kondisi | Yang ditampilkan | Banner |
|---|---|---|---|
| L2 — Penuh | Kedua API 200 | Detail per jam + 7 hari, jam terbaik | tidak ada (footer kecil "Data: [analysis_date & waktu fetch]") |
| L1 — BMKG saja | Open-Meteo gagal / toggle OFF | Detail per 3 jam, 7 hari terpotong jadi 3 hari (sesuai BMKG) | `"Akurasi per jam berkurang — data BMKG per 3 jam"` |
| L0 — Cache | Kedua API gagal | Data cache terakhir, label jelas | `"Data tidak diperbarui sejak [tanggal]"` merah + tombol "Coba lagi" |

### 9.3 Offline penuh

Karena seluruh aset (CSS/JS/ikon SVG) inline di 1 file, halaman tetap bisa TAMPIL offline setelah pernah dimuat — hanya datanya saja yang basi. Ini perbaikan terbaik yang bisa dicapai tanpa Service Worker (Service Worker butuh HTTPS, tidak tersedia di `file://`).

---

## 10. Model Data Lokal (localStorage)

Skema tunggal — tanpa PII, tanpa akun:

```jsonc
// jemur.config
{
  "adm4": "35.71.01.1010",
  "lat": -7.7933,
  "lon": 111.9955,
  "label": "Ngampel, Mojoroto, Kediri",
  "useOpenMeteo": true,
  "updatedAt": "2026-09-28T02:00:00+07:00"
}

// jemur.cache.fused
{
  "payload": { "...": "hasil yang sudah dinormalisasi + status + alasan" },
  "fetchedAt": "2026-09-28T02:00:00+07:00",
  "analysisDate": "2026-09-28T00:00:00+07:00"   // dari BMKG
}
```

Keamanan/privacy: tidak ada data sensitif. Yang tersimpan hanya lokasi & preferensi. Data mentah API tidak disimpan lebih dari TTL.

---

## 11. Desain UI (mobile-first)

### 11.1 Struktur layar (satu halaman, gulir vertikal)

```
┌────────────────────────────┐
│  Header: "Kapan Bisa Jemur?"│
│  <lokasi> · <kesegaran data>│◄ header sticky
├────────────────────────────┤
│  KARTU SEKARANG            │
│   [ikon besar] CERAH       │
│   28°C · RH 46% · angin 15│
│   Peluang hujan 5%         │
│   Alasan: awan 12%, pop 5% │
│   3 jam ke depan: C·C·M    │
├────────────────────────────┤
│  STRIP 24 JAM  (gulir hor.)│
│  07|08|09|10|11|12|...     │
│  🟢🟢🟢🟢🟢🟡🟡...      │
├────────────────────────────┤
│  7 KARTU HARI              │
│  Sen · ☀️ 34/24 · 12% ·    │
│  Jam terbaik: 09.00–15.00  │
│  Sel · 🌧 32/24 · 75% ·    │
│  Tidak ada jendela layak   │
├────────────────────────────┤
│  PENGATURAN (collapsed)    │
│  Lokasi: [Ngampel...▼]     │
│  Open-Meteo: [ON]          │
│  Segarkan · Hapus data     │
├────────────────────────────┤
│  Footer atribusi           │
└────────────────────────────┘
```

### 11.2 Prinsip tampilan

- Warna sebagai **lapisan pertama**, angka sebagai **lapisan pendukung** (colorblind-friendly: warna + ikon + label teks, bukan warna saja).
- Kartu hari: 5 baris maks (hari, ikon+cuaca, suhu, peluang hujan, jam terbaik).
- Tombol besar untuk area sentuh HP.
- Dark/light otomatis mengikuti OS (nice-to-have; ditunda bila effort tidak sebanding nilainya pada v1).

### 11.3 Pilihan ikon

SVG inline (emoji atau path sederhana), **tidak** menyandang URL ikon BMKG — demi persyaratan "1 file + offline penuh". Trade-off dicatat: ikon official BMKG tidak dipakai.

---

## 12. Kewajiban Atribusi (HARD REQUIREMENT)

Footer **wajib** menampilkan dan akan ditegakkan pada evaluasi:

1. **BMKG** — "Data prakiraan cuaca bersumber dari BMKG (Badan Meteorologi, Klimatologi, dan Geofisika)". Kewajiban resmi dari BMKG: mencantumkan BMKG sebagai sumber data di aplikasi.
2. **Open-Meteo** — "Weather data by Open-Meteo.com" + link, sesuai lisensi CC BY 4.0.

Format footer v1:
```
Data cuaca: BMKG — Badan Meteorologi, Klimatologi, dan Geofisika a · b
Prakiraan tambahan: Open-Meteo.com (CC BY 4.0) c
```
(dengan `a`, `b`, `c` = link ke `data.bmkg.go.id`, `bmkg.go.id`, `open-meteo.com`)

---

## 13. Risiko & Mitigasi

Secara sengaja dipisah: yang **benar-benar penting** vs yang **teoretis**, karena pemahaman user dulu baru mitigasi.

### 13.1 Tinggi (harus diperlakukan serius)

**R1 — Hujan lokal & over-trust.** Ramalan untuk radius 1–3 km; di Ngampel hujan lokal sering beda satu kelurahan dengan tetangganya. Bahaya terbesar: user percaya 100% pada warna hijau lalu tidak cek langit.
- Mitigasi: *explainability wajib* (§7.5); banner kecil "Ramalan ≠ jaminan. Cek langit sebelum jemur panjang."; analisis komparasi harian opsional.

**R2 — Lisensi Open-Meteo (non-komersial).** ToS free tier: penggunaan non-komersial, < 10.000 request/hari, tanpa SLA. Usaha laundry termasuk komersial; secara tertulis pemakaian ini melanggar ToS, risiko dampak praktis kecil (Volume ratusan kali di bawah limit; tidak ada enforcement otomatis yang diketahui) tapi **ini keputusan user**, bukan asumsi saya.
- Mitigasi (sudah dikunci): Open-Meteo = **toggle opsional, default ON tapi bisa OFF satu-tap**; BMKG = sumber aman tanpa batasan komersial; dokumentasi risiko di BRIEF; bila user ragu → OFF total tanpa ubah kode.

**R3 — Coarseness BMKG (2×/hari, per 3 jam).** Data "jam sekarang" bisa berumur hingga 3 jam. Tidak cocok untuk keputusan menit-ke-menit.
- Mitigasi: menampilkan `analysis_date` sebagai label kesegaran; Open-Meteo menutup gap per jam; ekspektasi user dibangun lewat §9.2 dan banner.

### 13.2 Sedang — perlu desain yang aman

| Risiko | Dampak | Mitigasi |
|---|---|---|
| HP offline / sinyal mati | Tidak ada data | Cache local (§9.1); TTL panjang; banner tidak-menyembunyikan |
| API berubah format JSON | Halaman berhenti terbaca | Isolasi layer pemetaan per API (§5.1); risiko ini dilaporkan, bukan disembunyikan |
| CORS ditutup pihak penyedia | Halaman mati total, mustahil diperbaiki tanpa server | Dilaporkan sebagai keterbatasan desain; fallback: host 1 file di HTTPS gratis (v2) |

### 13.3 Rendah / teoretis

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Rate limit BMKG (60/menit/IP) | Throttling | Volume pemakaian << limit; cache 1 jam |
| Rate limit Open-Meteo (10k/hari) | Throttling | Volume pemakaian « limit; cache 30 menit |
| Perangkat lama / browser tua | Render buruk / localStorage penuh | Menargetkan browser modern Android; hapus-cache tombol; total data ~< 100 KB |

---

## 14. Kriteria Penerimaan (acceptance criteria — dapat diuji)

| ID | Kriteria | Cara uji |
|---|---|---|
| AC-1 | Antara buka file sampai kartu "Sekarang" render: ≤ 10 detik (tanpa cache, WiFi stabil) | Stopwatch, pertama buka |
| AC-2 | Setiap chip jam menampilkan ≥ 1 bukti angka pendukung warna | Inspeksi visual strip 24 jam |
| AC-3 | Strip 24 jam berisi 24 jam berturut-turut mulai jam lokal sekarang, scroll horizontal | Scroll & hitung |
| AC-4 | 7 kartu hari menampilkan: nama hari, ikon, min/max suhu, peluang hujan max, total hujan, dan "jam terbaik" | Inspeksi |
| AC-5 | Saat Open-Meteo OFF (toggle) → halaman tetap berfungsi, granularitas 3 jam, banner muncul | Matikan toggle, refresh |
| AC-6 | Saat kedua API gagal (uji dengan batasi jaringan / matikan internet lalu buka) → cache terakhir ditampilkan berlabel waktu + banner + tombol coba | Uji offline |
| AC-7 | Config lokasi bertahan setelah halaman ditutup & dibuka lagi | Ubah lokasi, tutup, buka |
| AC-8 | Halaman adalah 1 file HTML tanpa request eksternal selain ke-2 API cuaca | DevTools → Network tab, filter non-API |
| AC-9 | Footer menampilkan atribusi BMKG + link Open-Meteo | Inspeksi |
| AC-10 | Halaman tetap tampil bila data cache berumur 2 hari (dengan banner) | Uji offline lama |

---

## 15. Biaya & Pemanfaatan Kuota

- **Biaya riil**: Rp 0. Hosting: tidak ada (file lokal). Domain: tidak ada.
- **Pemanfaatan API**: setiap buka halaman ≈ 2–6 request (gabungan + cache). Limit BMKG 60/menit; Open-Meteo 10.000/hari. Untuk pemakaian 1 HP, konsumsi < 1% limit.
- **Biaya tersembunyi**: 1–2 MB data seluler per hari (JSON cuaca sangat kecil; sebagian besar adalah file HTML ~200–300 KB yang dimuat sekali, disimpan browser cache).

---

## 16. Iterasi & Rencana v2

Saat v1 stabil dipakai 2 minggu, evaluasi:

- Akurasi nyata vs pengalaman jemur harian (log informal: berapa kali app "salah").
- Bila Open-Meteo sering menentang BMKG → putuskan permanen: OFF toggle standar, atau bertahan.
- Fitur kandidat v2 (dengan catatan teknologinya): notifikasi (butuh hosting HTTPS + Service Worker), multi-batch laundry tracking, "cegah hujan tiba-tiba" dengan autocheck yang menaikkan banner saat data berubah.

---

## 17. Lampiran — Ringkasan keputusan yang sudah dikunci

| Keputusan | Nilai | Justifikasi cepat |
|---|---|---|
| Lokasi v1 | `35.71.01.1010` · Ngampel, Mojoroto, Kediri · `-7.7933, 111.9955` | Verifikasi langsung ke BMKG + OSM |
| Sumber data | BMKG utama + Open-Meteo toggleable | Akurasi + legalitas + kesederhanaan |
| Bentuk produk | 1 file `index.html`, client-side murni | Nol biaya, portable, offline |
| Notifikasi | Ditolak v1 | Batasan platform `file://` (butuh HTTPS+SW) |
| Jam terbaik | Termasuk, 1 baris per kartu | Menjawab pertanyaan inti |
| Explainability | Wajib, bukan opsional | Melawan kesalahan kepercayaan ramalan |
| Ikon | SVG inline (tidak pakai URL BMKG) | Syarat 1 file + offline |
| Atribusi | BMKG + Open-Meteo (CC BY 4.0), hard requirement | Wajib hukum/lisensi |