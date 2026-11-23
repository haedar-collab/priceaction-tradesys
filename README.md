# Tradesys Edge Price Action

Web app sistem trading crypto **Blueprint Price Action** (knowledge base `haedar-collab/blueprint-price-action-knowledge-base`) untuk **Binance Spot, hanya long**.
Satu file `index.html`, tanpa build, tanpa server, tanpa aplikasi berbayar. Data langsung dari API publik Binance di browser.

> Aplikasi ini hanya memberi **sinyal dan alert**. Tidak ada order yang dikirim ke exchange dan tidak butuh API key Binance.

---

## Fitur

| Tab | Isi |
|---|---|
| **Dashboard** | BTC, struktur TF besar semua pair (↑/↓), penjaga risiko, jumlah sinyal, pair **di lokasi**, sesi WIB, chart + **kartu Trading Plan** (5 pertanyaan pra-entry, grade A/B, cerita harga, frekuensi historis), papan sinyal, daftar "di lokasi: menunggu cerita berubah" |
| **Scanner** | Watchlist atau semua pair USDT. Status: **Sinyal · Di lokasi · Mendekati zona · Tengah chart · TF besar turun**, jarak ke zona (ATR), syarat yang kurang |
| **Chart** | Panel TF kecil / menengah / besar. Zona S/R & supply/demand, HH/HL/LH/LL, BOS/CHoCH, fake break (×), panel **irama candle** (body ÷ ATR), level plan, riwayat sinyal |
| **Backtest** | Satu pair, 3.000–10.000 candle, per setup & per grade, fee + slippage, penjaga risiko opsional |
| **Risiko & jurnal** | Kalkulator posisi (risk dulu → SL logis → lot mengikuti), jurnal R, status GO / Tunggu / Stop |
| **Alert** | Log sinyal, **peringatan intrabar realtime**, Telegram, auto-scan: Realtime WebSocket (default) / tiap close candle / 15 menit / 1 jam |
| **Pengaturan** | Setup, stop & target, risiko, biaya, endpoint, Telegram |

Bahasa EN/ID dan tema terang/gelap di header.

## Realtime (WebSocket)

Default auto-scan = **Realtime WebSocket**. Satu koneksi ke `wss://data-stream.binance.vision` (data pasar publik, tanpa API key) memuat stream kline semua pair hasil scan di ketiga timeframe. Badge **● Realtime** di header menunjukkan status koneksi.

- **Harga live** diperbarui setiap push Binance (±1–2 detik), tanpa polling REST.
- **Candle close:** bar baru ditambahkan, semua pair dievaluasi ulang ±2,5 detik setelah close (menunggu candle TF menengah/besar yang close bersamaan). Jika ada candle terlewat (koneksi putus), data pair itu diisi ulang dari REST.
- **Peringatan intrabar** (tab Alert, opsional ke Telegram): harga masuk zona support/demand, atau plan aktif menyentuh SL / TP1 / TP2. Sifatnya informasi; **sinyal entry tetap hanya dari candle yang sudah close** (tanpa repaint).
- Sambung ulang otomatis (1 → 30 detik), pindah ke `stream.binance.com:9443` jika endpoint utama gagal, dan watchdog jika 60 detik tidak ada data. Selama terputus, aplikasi kembali ke scan REST tiap close candle.

## Timeframe (maks 3, Bab 5)

| Mode | Besar (arah & area) | Menengah (lokasi) | Kecil (timing) |
|---|---|---|---|
| D1 · H4 · H1 | D1 | H4 | H1 |
| **H4 · H1 · M15** (default, sesuai roadmap buku) | H4 | H1 | M15 |
| H1 · M15 · M5 | H1 | M15 | M5 |

Hanya candle yang sudah close. Swing dihitung setelah sisi kanannya close (uji look-ahead: 981 titik, 0 selisih).

## Strategi

**Struktur (Bab 2):** swing high/low → HH-HL / LH-LL. BOS hanya jika **close** menembus swing; tembus pakai wick lalu close balik = **fake break**. Range = tidak ada BOS lama, atau high/low sejajar. TF besar downtrend → minggir.

**Lokasi (Bab 4, 6):** zona S/R = swing yang diikuti reaksi ≥ 1,5 ATR (area, bukan garis; mati jika close tembus, resistance tembus → berubah fungsi jadi support). Demand = base 1–3 candle kecil sebelum impuls ≥ 2 ATR.

**Perubahan cerita (Bab 9):** seller melambat → candle bullish ≥ 0,6 ATR close di atas high 2 candle sebelumnya, lebih besar dari candle merah sebelumnya → **candle berikutnya setuju** (atau koreksi kecil gagal lalu lanjut). Entry di close candle yang setuju, bukan candle pertama.

| | Pullback lanjutan (Bab 7) | Retest breakout (Bab 8) | Bawah range (Bab 2, 4) |
|---|---|---|---|
| Konteks | TF besar uptrend, TF menengah tidak patah turun | TF besar tidak downtrend | Range TF menengah: kedua batas ditolak ≥ 2×, lebar 3–12 ATR |
| Lokasi | Pullback **sehat** (pelan/overlap, tanpa candle bearish besar, tidak close tembus HL) ke support/demand | Close jelas di atas level + follow-through, harga pergi, lalu retest dan tidak close kembali ke dalam | Low sequence di 25% bawah range |
| SL | Di bawah awal shift (opsi: di bawah zona) | Di bawah low retest / level | Di bawah low range |
| TP1 / TP2 | Hambatan pertama (high sebelumnya / resistance / supply) / hambatan TF besar | Hambatan pertama / TF besar | Batas atas range / hambatan TF besar |

**Lima pertanyaan pra-entry (Bab 13):** struktur jelas, lokasi masuk akal, cerita berubah, tahu salah di mana, risk aman. Satu "enggak" = no trade.

**Grade (Bab 10):** 6 konfluensi (S/R + S/D sejalan, lokasi TF besar, karakter bersih, rejection, sequence kuat, area segar). ≥ 4 = **A** (risiko penuh), 2–3 = **B** (setengah), < 2 = skip.

**Manajemen:** TP1 ambil 50%, stop pindah ke higher low TF kecil terbaru (bukan asal BE), sisa keluar saat struktur kecil patah atau di TP2. Belum TP1 setelah 48 candle → keluar (tidak ada follow-through).

**Risiko:** 1% per trade (0,5% mode belajar), maks 1 trade per sesi (roadmap buku). Batas aplikasi (bukan dari buku): stop setelah 2 rugi beruntun, batas rugi harian 2%, maks 3 trade/hari, ruang ke hambatan pertama ≥ 1,2R, TP1 ≥ 3× biaya.

## Hasil uji dengan data Binance asli

10 pair (BTC, ETH, SOL, BNB, XRP, DOGE, ADA, AVAX, LINK, SUI), setting default, entry di open candle berikutnya, penjaga risiko aktif. Parameter **tidak** dioptimasi.

| Mode | Periode | Skenario | Trade | Win | Profit factor | Ekspektansi |
|---|---|---|---|---|---|---|
| H4 · H1 · M15 | 15 Jan – 23 Nov 2026 | Fee 0,1% + slip 0,02% | 105 | 38% | 0,86 | −0,09R (t −0,7) |
| H4 · H1 · M15 | sama | Tanpa biaya | 107 | 42% | 1,18 | +0,10R (t 0,7) |
| D1 · H4 · H1 | 27 Des 2025 – 23 Nov 2026 | Fee 0,1% + slip 0,02% | 31 | 29% | 0,62 | −0,28R |
| D1 · H4 · H1 | sama | Tanpa biaya | 31 | 32% | 0,77 | −0,14R |

Per setup (M15, dengan biaya): pullback 95 trade −0,07R · retest breakout 6 trade +0,03R · bawah range 4 trade −0,76R.

**Kesimpulan:** aturan buku yang diterjemahkan secara mekanis **belum menunjukkan edge** yang signifikan secara statistik. Tanpa biaya mendekati impas sedikit positif; setelah biaya negatif. Sinyal sangat jarang (±1 per 3 hari untuk 10 pair di M15), sebagian karena periode uji banyak downtrend dan sistem hanya long. Gunakan sebagai alat bantu membaca struktur–lokasi–cerita dan **paper/forward test** dulu, bukan sinyal otomatis dengan modal sungguhan.

## Cara upload (GitHub Pages)

1. Buat repo `priceaction-tradesys`, upload `index.html` dan `README.md` → **Commit changes**.
2. **Settings → Pages** → *Deploy from a branch*, branch `main`, folder `/ (root)` → **Save**.
3. Setelah 1–2 menit: `https://haedar-collab.github.io/priceaction-tradesys/`
4. HP: Chrome → ⋮ → **Add to Home screen**.

## Telegram

1. @BotFather → `/newbot` → salin token. Kirim pesan ke bot, buka `https://api.telegram.org/bot<TOKEN>/getUpdates`, salin `chat.id`.
2. **Pengaturan → Data dan Telegram**: isi token & chat ID, centang kirim sinyal live (dan peringatan intrabar jika mau), **Simpan**.
3. Tab **Alert**: *Kirim pesan tes* / *Kirim sinyal tes*.

## Batasan

- Auto-scan dan WebSocket hanya berjalan selama halaman terbuka. Di HP, browser bisa menghentikan koneksi saat layar mati atau tab di latar belakang; saat dibuka lagi aplikasi menyambung ulang dan mengisi data yang terlewat.
- Endpoint default `data-api.binance.vision`. Jika status "Data demo", ganti endpoint di Pengaturan.
- Tidak ada kalender berita; aturan buku "hindari news besar" dicek manual.
- Contoh di buku memakai forex; adaptasi ke crypto (ATR menggantikan pip, sesi WIB, hanya long) adalah keputusan aplikasi.

*Bukan nasihat keuangan.*
