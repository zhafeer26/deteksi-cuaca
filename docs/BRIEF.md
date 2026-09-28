# BRIEF — Aplikasi "Kapan Bisa Jemur?"

Ringkasan 3 menit. Dokumen lengkap: `docs/PRD.md`.

---

## Masalah

Owner laundry di **Ngampel, Mojoroto, Kota Kediri** harus memutuskan setiap hari: jemur cucian sekarang atau tidak. Saat ini keputusan itu bergantung pada tebakan dan 2–3 aplikasi cuaca yang datanya sering bertolak belakang. Salah tebak = cucian setengah kering kena hujan → dicuci ulang → kehilangan waktu dan jadwal antar.

## Solusi

Satu **file HTML** yang dibuka di HP, menampilkan kondisi cuaca khusus jemur:

- **Status per jam** (hijau/kuning/merah) — aman, waspada, atau jangan jemur.
- **Kartu 7 hari** lengkap dengan **"Jam terbaik menjemur"** per hari (mis. `09.00–15.00`).
- **Simpan lokasi** sekali, tidak perlu diatur ulang.
- **Berfungsi offline** setelah pernah dibuka (data terakhir tersimpan di HP).

## Cara kerjanya

Dua sumber data gratis diambil langsung oleh browser, tanpa server dan tanpa biaya:

| Sumber | Peran | Detail |
|---|---|---|
| **BMKG** (resmi, gratis, tanpa batasan komersial) | Verifikasi + cadangan | Prakiraan **level kelurahan** (yang paling akurat untuk kami), 3 hari, per 3 jam |
| **Open-Meteo** (gratis) | Tulang punggung per jam | Per jam hingga 7 hari, termasuk peluang hujan & jam matahari |

Setiap status wajib menunjukkan **alasannya** (contoh: "Merah karena peluang hujan 75% + BMKG: Hujan Sedang"). Hijau/merah tanpa alasan dilarang — supaya tidak ada kepercayaan buta pada ramalan.

## Keputusan yang sudah diambil

1. **Format: 1 file HTML**, bukan aplikasi dan bukan website ber-server. Nol biaya, bisa dibagikan lewat WhatsApp.
2. **BMKG sebagai sumber aman utama.** Open-Meteo bisa **dimatikan** dengan satu toggle — untuk alasan lisensi (lihat Risiko).
3. **Notifikasi ditiadakan di versi pertama.** Bukan lupa: dari file lokal (`file://`), ponsel tidak mengizinkan notifikasi saat HP dalam kondisi terkunci/halaman ditutup. Ini batasan platform. Bisa ditambahkan di v2 bila file di-hosting di HTTPS gratis.
4. **Terdapat baris "Jam terbaik menjemur"** di setiap kartu hari.

## Risiko utama (singkat)

1. **Hujan lokal** — ramalan bisa benar untuk radius 1–3 km tapi salah di lokasi Anda. Karena itu setiap warna harus menunjukkan alasan, dan Anda tetap harus melihat langit. Ini bukan bug, ini sifat ramalan cuaca.
2. **Lisensi Open-Meteo** — free tier resminya untuk penggunaan **non-komersial**, sedangkan laundry adalah usaha. Dampak praktis kecil (pemakaian sangat sedikit), tapi keputusan memakainya ada di Anda → bisa dimatikan permanen kapan saja, aplikasi tetap berjalan penuh dengan BMKG.
3. **Data BMKG tidak per jam** — diperbarui 2×/hari per 3 jam. Aplikasi ini untuk **merencanakan**, bukan ramalan menit-ke-menit.

## Data teknis yang sudah diverifikasi (tidak perlu diubah)

- Kode wilayah: `35.71.01.1010` — desa **Ngampel**, kecamatan **Mojoroto**, Kab./Kota **Kota Kediri**, Jawa Timur.
- Koordinat: `-7.7933, 111.9955` (dikonfirmasi langsung oleh BMKG & OpenStreetMap).
- Kedua API mengizinkan akses dari browser (CORS) — file HTML statis bisa berjalan seperti ini.

## Batasan yang jujur

- Bukan ramalan menit-ke-menit.
- Tanpa notifikasi (hingga dipindah ke hosting HTTPS).
- Tergantung ketersediaan/kestabilan API gratis milik pihak ketiga.

## Langkah berikutnya

1. Anda menyetujui PRD ini.
2. Saya bangun `index.html` (1 file, mobile-first, tanpa build step) sesuai PRD §7, §11.
3. Anda buka di HP, cek selama ~2 minggu, lalu kita evaluasi akurasi & apakah Open-Meteo tetap dipakai.
4. Bila perlu, v2: notifikasi (butuh hosting gratis HTTPS + service worker).

---

*Dokumen pendamping ini merangkum `docs/PRD.md`. Jika ada perbedaan, PRD yang menjadi acuan.*