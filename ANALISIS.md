# Analisis Project — Theatron (UI Aplikasi Tiket Event)

**Lokasi:** `c:/Users/User/Downloads/UI-design`
**Cakupan:** 13 halaman HTML, ±128 KB, tanpa file CSS/JS bersama
**Metode:** pembacaan seluruh source + penelusuran alur nyata di browser (Beranda → Cari → Detail → Masuk → Pilih Tiket → Data Pemesan → Cek Harga → Metode Bayar)

---

## 1. Ringkasan Eksekutif

Ini adalah **prototype UI statis** untuk aplikasi pembelian tiket event bernama **Theatron**. Kualitas visualnya sudah tinggi dan terasa seperti produk sungguhan: palet "paper/warm cream" dengan aksen terracotta, frame 390 px yang konsisten, state fokus yang benar, dukungan `prefers-reduced-motion`, dan microcopy Bahasa Indonesia yang rapi.

Masalahnya bukan pada tampilan, tapi pada **struktur dan sambungan antar halaman**:

| Aspek | Nilai | Catatan singkat |
|---|---|---|
| Kualitas visual | **8 / 10** | Konsisten, hangat, hierarki jelas |
| Konsistensi implementasi | **4 / 10** | Setiap halaman menyalin ulang CSS & markup |
| Aksesibilitas | **6 / 10** | Fondasi bagus (ARIA tab/radio, focus ring), tapi tanpa `<main>`, `<h1>`, dan state ARIA yang bocor |
| Arsitektur kode | **2 / 10** | Nol file bersama, ±40 KB CSS terduplikasi, styling via inline + selector substring |
| Kelengkapan alur | **5 / 10** | Happy path putus, fitur antrean yatim, logika harga bertabrakan |

**Tiga hal yang paling mendesak:**

1. **Alur pembayaran tidak pernah berhasil.** `ui-bayar.html` → `ui-belum.html` (gagal). Halaman sukses `ui-berhasil.html` hanya bisa dicapai dari layar gagal. Semua transaksi berakhir di "Pembayaran belum selesai".
2. **`ui-tunggu.html` (Ruang Tunggu) tidak terhubung ke mana pun.** Padahal `ui-masuk.html` menulis "Masuk sebelum antre supaya posisi antreanmu tidak terbuang" — antrean adalah fitur inti tiket konser, dan di alur nyata ia dilewati sepenuhnya.
3. **Tidak ada satu pun file CSS/JS bersama.** 13 halaman × ~3 KB CSS identik. Mengubah satu warna berarti mengedit 13 file, dan sudah ada bukti drift: `.page-title` yang hilang di `ui-cari.html`, progress bar yang menjorok 40 px di checkout, dan dua nilai merah (`#BF4527` vs `#D2431B`) yang saling menimpa.

---

## 2. Inventaris File

| File | Ukuran | Peran dalam alur | Status |
|---|---|---|---|
| `ui-beranda.html` | 14.7 KB | Beranda / katalog | ✅ |
| `ui-cari.html` | 14.6 KB | Hasil pencarian + filter | ⚠️ filter palsu |
| `ui-detail.html` | 8.7 KB | Detail event (dijual) | ✅ |
| `ui-detail-belum.html` | 10.3 KB | Detail event (belum dibuka) + countdown | ⚠️ pengingat tidak tersimpan |
| `ui-masuk.html` | 6.8 KB | Login | ⚠️ tanpa daftar/lupa sandi |
| `ui-tunggu.html` | 7.8 KB | Ruang tunggu / antrean | ❌ **yatim, tidak ada yang menautkan** |
| `ui-pilih.html` | 9.3 KB | Pilih jumlah tiket (step 1/4) | ✅ |
| `ui-data.html` | 7.3 KB | Data pemesan (step 2/4) | ✅ |
| `ui-harga.html` | 8.2 KB | Cek harga final (step 3/4) | ⚠️ kontradiksi biaya layanan |
| `ui-bayar.html` | 9.3 KB | Metode pembayaran (step 4/4) | ⚠️ tombol kembali salah target |
| `ui-belum.html` | 5.8 KB | Pembayaran belum selesai | ✅ |
| `ui-berhasil.html` | 7.0 KB | Pembayaran berhasil | ⚠️ sulit dicapai |
| `ui-tiket.html` | 15.2 KB | Tiket saya (4 tab + QR) | ✅ tab ARIA paling matang |

**Di folder induk** (`Downloads/`) ditemukan:
- `ui-belum_1.html`, `ui-data_1.html`, `ui-harga_1.html`, `ui-masuk_1.html`, `ui-pilih_1.html`, `ui-berhasil_1.html` — **duplikat identik** (hash MD5 sama persis). Versi di dalam folder `UI-design` adalah versi yang lebih baru/lebih lengkap.
- `Wireframe dan UI Aplikasi Tiket.html` (1.2 MB) — export bundler (kemungkinan sumber desain awal).
- `index.html`, `Mockup TEMANPORSI.html` (0 byte), dan file tidak terkait (tugas kuliah, dll.).

Risiko: tidak jelas mana versi kanonik. Duplikat `_1` sebaiknya dihapus dari folder induk.

---

## 3. Peta Alur Navigasi

```
                    ┌──────────────────────────────────────────┐
                    │                                          │
              ┌─────▼─────┐   Lihat semua    ┌──────────┐      │
              │ ui-beranda│─────────────────▶│  ui-cari │◀─────┼──┐
              │  (Beranda)│◀─────────────────│  (Cari)  │      │  │
              └─────┬─────┘   bottom nav     └────┬─────┘      │  │
                    │ klik kartu                  │ kartu      │  │
                    ▼                             ▼            │  │
              ┌───────────┐    kartu "BELUM   ┌────────────────┐│  │
              │ ui-detail │    DIBUKA"        │ui-detail-belum ││  │
              │  (dijual) │                    │  (pengingat)   ││  │
              └─────┬─────┘                    └────────────────┘│  │
                    │ Beli Tiket                                  │  │
                    ▼                                             │  │
              ┌───────────┐                                       │  │
              │ ui-masuk  │  ("Masuk sebelum antre…")             │  │
              └─────┬─────┘                                       │  │
                    │ submit / Google                             │  │
                    ▼                                             │  │
              ┌───────────┐          ┌──────────────┐             │  │
              │ ui-pilih  │   ???    │  ui-tunggu   │  ❌ YATIM   │  │
              │  1/4      │╌╌╌╌╌╌╌╌╌▶│  (antrean)   │             │  │
              └─────┬─────┘  tidak    └──────────────┘             │  │
                    │        tersambung                            │  │
                    ▼                                             │  │
              ┌───────────┐                                       │  │
              │ ui-data   │ 2/4                                   │  │
              └─────┬─────┘                                       │  │
                    ▼                                             │  │
              ┌───────────┐                                       │  │
              │ ui-harga  │ 3/4                                   │  │
              └─────┬─────┘                                       │  │
                    ▼                                             │  │
              ┌───────────┐  Bayar Sekarang                       │  │
              │ ui-bayar  │ 4/4 ──────────┐                      │  │
              └───────────┘               ▼                      │  │
                 ▲ back                 ┌──────────┐  Cek Status  │  │
                 └────────── ??? ───────│ ui-belum │─────────────▶│  │
                   (menunjuk ui-tunggu) │  (gagal) │              │  │
                                        └────┬─────┘              │  │
                                             │ Ganti Metode Bayar │  │
                                             ▼                    │  │
                                        ui-bayar                  │  │
                                             │                    │  │
                                        ┌────▼─────────┐          │  │
                                        │ ui-berhasil  │          │  │
                                        │  (sukses)    │──────────┘  │
                                        └────┬─────────┘  Tiket Saya │
                                             ▼                      │
                                        ┌──────────┐                │
                                        │ ui-tiket │◀───────────────┘
                                        │ 4 tab QR │
                                        └──────────┘
```

**Semua tautan `href` sudah menunjuk file yang benar** (diverifikasi otomatis: tidak ada link putus). Masalahnya ada di **semantik rute**, bukan integritas link.

### Cacat alur yang terbukti saat ditelusuri di browser

| # | Temuan | Bukti |
|---|---|---|
| A1 | Happy path tidak pernah sampai `ui-berhasil.html`. "Bayar Sekarang" langsung ke layar gagal. | `ui-bayar.html`: `href="ui-belum.html"` |
| A2 | `ui-tunggu.html` yatim — nol inbound link. Copy login menyebut antrean, tapi submit langsung ke `ui-pilih.html`. | pencarian `ui-tunggu` di semua `href`: hanya `ui-bayar` yang menautkannya (sebagai tombol *kembali*, itu pun salah) |
| A3 | Tombol Kembali di `ui-bayar.html` menuju `ui-tunggu.html`, bukan `ui-harga.html`. | terverifikasi di browser: `link "Kembali" /url: ui-tunggu.html` |
| A4 | Konteks event hilang setelah login: `ui-detail → ui-masuk → ui-pilih`, tidak kembali ke event yang sedang dibeli. | markup |
| A5 | "Ingatkan Saya" di `ui-detail-belum.html` hanya mengubah state tombol — tidak disimpan, tab "Pengingat" di `ui-tiket.html` selalu kosong. | `#ingat` handler + panel `panel-pengingat` statis |
| A6 | `ui-tiket` tab "Menunggu" → `ui-belum` → "Lihat di Tiket Saya" → `ui-tiket` tab **Aktif** (bukan Menunggu). Loop tanpa akhir. | markup |
| A7 | Tidak ada state "kedaluwarsa / gagal permanen". Countdown `14 : 32` dan `09 : 45` statis, tidak berjalan. | `ui-belum.html`, `ui-pilih.html` |
| A8 | Setelah selesai bayar tidak ada pengosongan keranjang — `localStorage` tetap menyimpan 2 tiket. | `ui-berhasil.html` tidak menghapus key |
---

## 4. Design System — Yang Sudah Bagus, dan Yang Bocor

### 4.1 Token yang berhasil didefinisikan

Setiap halaman mendeklarasikan `:root` yang identik (10 dari 13 halaman, sama persis):

```css
--ink:#202521  --ink-weak:#1D1B1A  --muted:#565D57  --muted-soft:#5C5A55
--paper:#FBF8F1  --card:#F7F5F0  --canvas:#E9E3DA
--line:#DDD5C8  --line-soft:#E4DFD5
--accent:#BF4527  --accent-solid:#D2431B  --accent-hover:#A63314
--accent-soft-bg:#FBE9E3  --accent-soft-text:#8F2A0A
--success-bg:#E1F2EE  --success-text:#0B6E5B
--warning-bg:#FBF1CF  --warning-text:#6B4E00
--radius-sm/md/lg/frame  --shadow  --shadow-card  --focus-ring  --focus-offset
```

Ini fondasi yang benar — hanya saja **fisiknya digandakan 13 kali**, bukan diimpor.

### 4.2 Tipografi

| Aspek | Temuan |
|---|---|
| Font | `"Plus Jakarta Sans"` dideklarasikan di `.frame` / `body>div` **tapi tidak pernah dimuat**. Tidak ada `<link>` Google Fonts, tidak ada `@font-face`. Seluruh teks jatuh ke `system-ui`/`Segoe UI`. **Identitas tipografi yang dirancang tidak pernah tampil.** |
| Skala | 11 / 12 / 12.5 / 13 / 13.5 / 14 / 14.5 / 15 / 16 / 17 / 19 / 20 / 24 / 28 / 40 / 56 px — 16 nilai berbeda, inkremental 0.5 px. Tidak ada skala tetap. |
| Akses | 11 px (bottom nav, badge) ≈ 8.25 pt — di bawah ambang nyaman. |

Memuat font saja akan langsung menaikkan persepsi kualitas seluruh aplikasi.

### 4.3 Radius — tidak konsisten untuk peran yang sama

| Peran | Nilai yang dipakai |
|---|---|
| Kartu konten | `16px` (var `--radius-lg`), `18px` (override `[style*="border: 1px solid #E4DFD5"]`), `20px` (inline di `ui-tiket`, `ui-harga`) |
| Tombol | `14px` (`--radius-md`), `12px` (banyak tombol inline), `22px` (`.btn-icon-circle`) |
| Chip | `19px` (setengah tinggi) |

### 4.4 Warna — dua merah yang saling menimpa

`ui-detail.html`, `ui-detail-belum.html`, `ui-tiket.html` memuat aturan ini:

```css
body>div a[style*="background: #D2431B"],
body>div button[style*="background: #D2431B"] {
  background: var(--accent) !important;   /* #BF4527 */
}
```

Akibatnya tombol utama "Beli Tiket" yang ditulis `#D2431B` dirender **`#BF4527`**, sementara lencana/aksen lain tetap `#D2431B`. Dua merah berbeda di satu layar — tidak kentara di mata, tapi ini gejala: *style ditulis inline, lalu ditimpa `!important` untuk memaksanya konsisten.*

### 4.5 Jarak (spacing)

Padding horizontal halaman memakai 20 px secara konsisten (baik). Yang bocor adalah **indentasi progress bar checkout**:

```html
<!-- ui-harga.html & ui-bayar.html -->
<div style="padding:0 20px">
  <div class="progress">…</div>
</div>
```
```css
.progress { padding: 0 20px; }
```

→ 20 + 20 = **40 px**, sementara kartu konten di `margin: 0 20px` = **20 px**. Terverifikasi di browser: bar progres mulai ±12 px lebih kanan daripada kartu di bawahnya. `ui-data.html` dan `ui-pilih.html` tidak terkena (strukturnya berbeda) — jadi dua halaman checkout sejajar, dua lainnya tidak.

---

## 5. Temuan Visual & Konsistensi

| # | Tingkat | Temuan | Lokasi |
|---|---|---|---|
| B1 | **P1** | Font "Plus Jakarta Sans" tidak pernah dimuat → fallback sistem. Lihat §4.2 | semua halaman |
| B2 | **P1** | `.page-title` dipakai tapi **tidak punya aturan CSS** di file itu → judul header "Cari" tampil 16 px *regular*, sedangkan di halaman lain 16 px **bold**. Terverifikasi di browser. | `ui-cari.html:52` |
| B3 | **P1** | Progress bar menjorok 40 px vs konten 20 px. Terverifikasi di browser. | `ui-harga.html`, `ui-bayar.html` |
| B4 | **P1** | Struktur root tidak seragam: `.frame` (10 halaman) vs `body>div` (3 halaman) vs inline `width:390px` (2 halaman). Selector global `body>div …` hanya bekerja di halaman yang memakai `div` telanjang. | `ui-tiket`, `ui-detail`, `ui-detail-belum` |
| B5 | **P2** | Layar status tidak berpola: `ui-belum`/`ui-berhasil` tanpa header (pakai `.empty-state`), `ui-tunggu` dengan header. | 3 halaman |
| B6 | **P2** | Dua token merah saling menimpa (§4.4). | 3 halaman |
| B7 | **P2** | Ikon tombol hapus pencarian memakai **chevron kiri (‹)** dengan label "Hapus pencarian" — arah dan makna ikon salah. Terlihat jelas di browser. | `ui-cari.html` |
| B8 | **P2** | `<title>` campur: `"Theatron \| Beranda"` (bagus) vs `"UI Cari"`, `"UI Detail Event"`, `"UI Pilih Tiket"` (sisa nama file generator, bocor ke tab browser). | 12 halaman |
| B9 | **P2** | Chip kategori beranda tidak punya "Kuliner", padahal ada kartu `data-category="kuliner"` → filter apa pun menyembunyikan kartu tanpa penjelasan. | `ui-beranda.html` |
| B10 | **P2** | CTA bawah (`margin-top:auto` di frame `min-height:844px`) berada di luar viewport pada layar < 844 px dan tidak sticky → di desktop preview tombol utama tidak terlihat tanpa scroll. | semua halaman checkout |
| B11 | **P3** | `<style>` ganda dalam satu file: `ui-tiket` **4 blok**, `ui-beranda` 2 blok, `ui-detail-belum` 2 blok. | 3 halaman |
| B12 | **P3** | Tidak ada favicon, `meta description`, `theme-color`, atau OG tag. Link yang dibagikan tampil kosong. | semua |

---

## 6. Aksesibilitas

### Yang sudah benar (jangan diubah)
- `lang="id"` di semua halaman.
- `:focus-visible` global 3 px outline dengan offset — konsisten dan terlihat.
- `prefers-reduced-motion: reduce` menonaktifkan transisi di semua halaman.
- `aria-hidden="true"` konsisten pada semua ikon SVG dekoratif.
- `ui-tiket.html`: **pola tab ARIA terlengkap** — `role="tablist"/"tab"/"tabpanel"`, `aria-controls`, `aria-selected`, roving `tabindex`, navigasi ←/→/Home/End. Ini referensi untuk halaman lain.
- `ui-bayar.html`: radio memakai `<input type="radio">` asli di dalam `<label>` — state, keyboard, dan screen reader otomatis benar.
- Target sentuh mayoritas 44–54 px.

### Yang perlu diperbaiki

| # | Tingkat | Temuan | Lokasi |
|---|---|---|---|
| C1 | **P1** | Tidak ada landmark `<main>` di halaman mana pun. Screen reader tidak bisa melompat ke konten. | semua |
| C2 | **P1** | Tidak ada `<h1>` di beranda/cari/detail — heading mulai dari `<h2>`. Struktur dokumen tidak punya puncak. | 6 halaman |
| C3 | **P1** | Chip filter: markup awal `class="chip active"` tetapi `aria-pressed="false"` → state visual dan ARIA bertentangan sebelum JS jalan. | `ui-cari.html:58` |
| C4 | **P1** | Label chip diubah JS menjadi `"Kategori ✓"` — karakter centang masuk ke *accessible name*; pembaca layar membaca "Kategori centang" alih-alih mengandalkan `aria-pressed`. Cukup andalkan `aria-pressed`, jangan ubah teks. | `ui-cari.html` `render()` |
| C5 | **P1** | Status hasil pencarian ditulis lewat `summary.lastChild.textContent` — manipulasi DOM rapuh; teks "untuk konser" tidak dibungkus elemen sendiri dan mudah rusak bila markup berubah. | `ui-cari.html` |
| C6 | **P2** | Tombol notifikasi memakai class `.btn-back` (semantik kode salah) dan titik merah status tanpa teks alternatif — "ada notifikasi" tidak terbaca. | `ui-beranda.html:64` |
| C7 | **P2** | Countdown statis tanpa `role="timer"` / `aria-live`: `ui-belum` (`14 : 32`) dan `ui-pilih` (`09 : 45`). `ui-detail-belum` sudah benar (`role="timer"` + `aria-label` dinamis) — tiru pola itu. | 2 halaman |
| C8 | **P2** | Tombol "−" tidak `disabled` saat jumlah 0 → pengguna menekan tombol yang tidak bereaksi apa pun, tanpa umpan balik. | `ui-pilih.html` |
| C9 | **P2** | `.chip` tinggi 38 px < 44 px minimum target sentuh. | `ui-beranda`, `ui-cari` |
| C10 | **P3** | Pesan validasi form memakai bahasa browser (Inggris bila locale en) karena tidak ada validasi kustom. Terverifikasi: muncul "Please fill out this field" pada form login ber-`lang="id"`. | `ui-masuk`, `ui-data` |
| C11 | **P3** | `progressbar` di `ui-tunggu` tidak pernah di-update (`aria-valuenow="38"` statis); `#1.234` dan "± 8 menit" juga hardcode dan tidak saling konsisten. | `ui-tunggu.html` |

---

## 7. Arsitektur & Kualitas Kode

### 7.1 Masalah inti: duplikasi total

Tidak ada satu pun file `.css` atau `.js` di folder. Konkretnya:

- **±40 KB CSS identik** disalin ke 13 file (blok `:root`, reset, `.frame`, `.btn-back`, prose media query, `.btn-primary`, `.btn-secondary`, `.card`, `.bottom-bar`, dsb.).
- **Blok CSS "penambal" global yang sama persis** (satu baris sangat panjang berisi `:where(a[href],button,…):focus-visible`, `.card-event:hover`, dll.) ditempel di **ke-13 halaman**, termasuk di halaman yang tidak punya `.card-event` sama sekali.
- **Bottom nav ditulis ulang secara manual** di `ui-beranda`, `ui-cari`, `ui-tiket` — satu di antaranya (`ui-tiket`) memakai inline style, dua lainnya memakai class. Empat ikon SVG identik disalin 3×.
- **Pola "baca `localStorage` → format Rupiah"** diduplikasi di `ui-harga`, `ui-bayar`, `ui-berhasil`.
- **Data event hardcode** di 7 file. "Konser Musik Senja" muncul dengan format tanggal berbeda ("Sab, 12 Des 2026" vs "Sabtu, 12 Desember 2026") dan harga `150000` ditulis literal di JS `ui-harga`.

### 7.2 Pola paling rapuh: styling berbasis substring atribut

```css
body>div div[style*="padding: 14px 20px 0"][style*="overflow: hidden"] { … }
body>div div[style*="background: #FFFFFF"][style*="border: 1px solid #E4DFD5"] { … }
body>div a[style*="background: #D2431B"] { … }
```

Ada **8 selector seperti ini**. Artinya: satu spasi berubah di atribut `style` → aturan CSS mati diam-diam. Ini bukan styling, ini menebak-nebak berdasarkan string. Semua aturan ini seharusnya menjadi class biasa.

### 7.3 Perang spesifisitas

```css
.frame { width:390px!important; height:auto!important; overflow:visible!important; background:var(--card)!important; … }
```

`!important` dipakai untuk menimpa inline style pada elemen yang sama. Rantainya: *inline style → `!important` → `[style*=…]` → `!important` lagi*. Setiap perubahan kecil butuh `!important` baru.

### 7.4 Logika bisnis yang bertabrakan

| Klaim | Lokasi |
|---|---|
| "Harga akhir, sudah termasuk biaya layanan" | `ui-detail.html`, `ui-detail-belum.html`, `ui-pilih.html` |
| "Biaya layanan **Rp 10.000**" (ditambahkan) | `ui-harga.html`, JS `+ 10000` |
| "Harga sudah final. **Tidak ada biaya tambahan** setelah ini." | `ui-harga.html` |

Ketiganya tampil di alur yang sama. Pengguna melihat "sudah termasuk" di halaman detail, lalu dikenai biaya tambahan Rp 10.000 di halaman harga, disertai jaminan "tidak ada biaya tambahan". **Ini keputusan produk yang harus diambil dulu**, baru diperbaiki — bukan sekadar bug CSS.

Tambahan: `ui-harga` dan `ui-bayar` menghitung `selection.total + 10000` tanpa cek apakah `total > 0`; jika `localStorage` kosong (incognito / dibuka langsung) halaman tetap menampilkan 2 tiket dan Rp 310.000 sebagai default palsu.

### 7.5 State management

Satu key `localStorage` (`eventTicketSelection`) berisi `{regular, vip, total}`, tanpa versi, tanpa validasi, dan setiap pembacaan dibungkus `catch {}` kosong yang menelan semua error (termasuk `SyntaxError` bila data korup → halaman menampilkan angka default yang salah tanpa peringatan).

### 7.6 Yang tidak ada (dan seharusnya ada sebagai bagian dari UI kit)

- `README.md` — cara menjalankan, peta alur, daftar halaman.
- `package.json` / tooling — tidak perlu framework, tapi `live-server` + `prettier` akan menolong.
- Tidak ada halaman **Akun/Profil** padahal bottom nav punya item "Akun" yang selalu mengarah ke halaman login.
- Tidak ada halaman **daftar** dan **lupa kata sandi** meski judulnya "Masuk atau Daftar".
- Tidak ada **state kosong/error** untuk pencarian tanpa hasil di beranda, gagal jaringan, atau tiket kedaluwarsa (`ui-cari` sudah punya `#search-empty` — jadikan pola).
---

## 8. Rencana Perbaikan (berurutan)

### P0 — Perbaikan yang mengubah kesimpulan jadi "aplikasi bisa dipakai" (perkiraan 0.5–1 hari)

1. **Buat `assets/css/tokens.css` + `base.css` + `components.css`.** Pindahkan seluruh `:root`, reset, `.frame`, `.btn-*`, `.card`, `.chip`, `.bottom-bar`, `.theatron-header`, media query, dan `prefers-reduced-motion` ke sana. Semua halaman cukup `<link rel="stylesheet" href="assets/css/app.css">`. Ini langsung memotong ±40 KB duplikasi dan menghentikan drift.
2. **Muat font sungguhan.** Tambahkan `<link>` Google Fonts untuk Plus Jakarta Sans (400/600/700) + preconnect, atau self-host `.woff2`. Tanpa ini, seluruh keputusan tipografi tidak pernah terlihat.
3. **Perbaiki alur sukses.** `ui-bayar.html` "Bayar Sekarang" → halaman proses singkat → `ui-berhasil.html`. `ui-belum.html` tetap ada tapi hanya dicapai dari state gagal/timeout, bukan dari jalur utama.
4. **Sambungkan `ui-tunggu.html`.** `ui-masuk.html` → `ui-tunggu.html` → `ui-pilih.html`. Hapus tautan balik yang salah di `ui-bayar.html` (ganti ke `ui-harga.html`).
5. **Ganti semua `style="…"` menjadi class**, lalu hapus 8 selector `[style*="…"]` dan seluruh `!important` yang tersisa.

### P1 — Konsistensi & kebenaran (perkiraan 1–2 hari)

6. Putuskan model harga: inklusif atau eksklusif biaya layanan, lalu samakan teks di `ui-detail`, `ui-pilih`, `ui-harga`, `ui-bayar`, `ui-berhasil`.
7. Buat `data/events.js` sebagai satu-satunya sumber data event (id, nama, tanggal ISO, kota, harga, kategori, status). Semua kartu dan halaman detail me-render dari sana; `ui-detail.html?id=...`.
8. Tambahkan aturan `.page-title` (16 px / 700) ke stylesheet bersama → menutup B2 sekaligus B4.
9. Perbaiki indentasi progress bar: pindahkan `.progress` keluar dari wrapper ber-padding, atau hapus padding dari `.progress`.
10. Ganti ikon tombol hapus pencarian dengan "×", bukan chevron.
11. Seragamkan `<title>` menjadi `Theatron | <Nama Halaman>` di 13 file.
12. Samakan struktur root: semua halaman memakai `<div class="frame">`, hapus selector `body>div > *`.
13. Buat `assets/js/store.js` (baca/tulis localStorage dengan validasi skema + versi) dan `assets/js/money.js` (`formatRupiah`). Hapus 3 salinan logika yang ada.

### P2 — Aksesibilitas & kelengkapan (perkiraan 1–2 hari)

14. Bungkus konten setiap halaman dengan `<main id="konten">`; tambahkan `<h1>` (bisa visually-hidden pada halaman yang desainnya tidak punya judul besar).
15. Sinkronkan state ARIA: `aria-pressed` awal harus sama dengan class `active`; jangan ubah teks chip jadi "Kategori ✓" (cukup `aria-pressed`).
16. Bungkus teks hasil pencarian dalam elemen sendiri (`<span id="query-label">`) dan hilangkan `lastChild.textContent`.
17. Tambahkan `role="timer"` + `aria-live="off"` pada countdown statis, atau jadikan benar-benar berjalan dengan `setInterval` + `aria-live="polite"` yang hanya membacakan perubahan per menit.
18. `disabled` pada tombol "−" saat jumlah 0; naikkan tinggi `.chip` ke 44 px.
19. Buat halaman **Akun** dan **Daftar**; hubungkan item bottom nav "Akun" ke sana.
20. Tambahkan state kosong/error yang konsisten untuk: pencarian tanpa hasil, pembayaran gagal, tiket kedaluwarsa, koneksi terputus.
21. Tambahkan favicon, `meta name="description"`, `theme-color`, OG tag.

### P3 — Kebersihan (opsional)

22. Hapus `ui-*_1.html` duplikat dari folder induk; pindahkan file tidak terkait keluar dari folder project.
23. Tambahkan `README.md` berisi peta alur dan daftar halaman.
24. Buat `src/pages/*.html` bila ingin menata 13 file dalam satu folder datar yang lebih rapi.

---

## 9. Struktur Folder yang Diusulkan

```
UI-design/
├─ index.html                  → redirect/beranda
├─ README.md
├─ pages/
│  ├─ beranda.html
│  ├─ cari.html
│  ├─ detail.html              (?id=konser-senja)
│  ├─ detail-belum.html
│  ├─ masuk.html
│  ├─ daftar.html              (baru)
│  ├─ tunggu.html
│  ├─ akun.html                (baru)
│  ├─ tiket.html
│  ├─ checkout-pilih.html
│  ├─ checkout-data.html
│  ├─ checkout-harga.html
│  ├─ checkout-bayar.html
│  ├─ belum.html
│  └─ berhasil.html
├─ assets/
│  ├─ css/
│  │  ├─ tokens.css            → :root, warna, radius, shadow, z-index
│  │  ├─ base.css              → reset, body, .frame, tipografi, focus ring
│  │  ├─ components.css        → .btn-*, .card, .chip, .bottom-bar, .theatron-header, .progress
│  │  └─ pages.css             → aturan spesifik per halaman
│  ├─ js/
│  │  ├─ store.js              → eventTicketSelection: read/write/validate/clear
│  │  ├─ money.js              → formatRupiah()
│  │  ├─ nav.js                → render bottom nav + tandai item aktif
│  │  ├─ search.js
│  │  ├─ checkout.js
│  │  └─ tabs.js               → pola tab ARIA dari ui-tiket, dipakai ulang
│  ├─ img/                     → QR, ilustrasi kartu (saat ini masih div gradient)
│  └─ fonts/                   → Plus Jakarta Sans .woff2 (opsional, self-host)
└─ data/
   └─ events.js                → satu sumber data event
```

---

## 10. Contoh Perbaikan Konkret

**Token bersama (`assets/css/tokens.css`)** — dipakai oleh semua halaman, tidak lagi disalin:

```css
:root {
  --ink: #202521;
  --ink-weak: #1D1B1A;
  --muted: #565D57;
  --muted-soft: #5C5A55;
  --paper: #FBF8F1;
  --card: #F7F5F0;
  --canvas: #E9E3DA;
  --line: #DDD5C8;
  --line-soft: #E4DFD5;
  --accent: #BF4527;
  --accent-solid: #D2431B;
  --accent-hover: #A63314;
  --accent-soft-bg: #FBE9E3;
  --accent-soft-text: #8F2A0A;
  --success-bg: #E1F2EE;
  --success-text: #0B6E5B;
  --warning-bg: #FBF1CF;
  --warning-text: #6B4E00;
  --radius-sm: 10px;
  --radius-md: 14px;
  --radius-lg: 16px;
  --radius-frame: 28px;
  --shadow: 0 22px 60px rgba(38, 31, 23, 0.12);
  --shadow-card: 0 5px 16px rgba(38, 31, 23, 0.045);
  --focus-ring: 3px solid var(--ink);
  --focus-offset: 3px;
  --font-sans: "Plus Jakarta Sans", system-ui, "Segoe UI", sans-serif;
}
```

> Catatan: `--accent` dan `--accent-solid` bernilai beda. Pilih **satu** sebagai warna aksi utama dan turunkan yang lain (mis. `--accent` untuk teks/ikon di atas latar terang, `--accent-solid` untuk isian tombol). Jangan biarkan keduanya bertukar peran seperti sekarang.

**Kartu event menjadi komponen** — hilangkan inline style dan selector substring:

```css
.card-event {
  display: flex;
  gap: 12px;
  align-items: center;
  padding: 10px;
  background: #fff;
  border: 1px solid var(--line-soft);
  border-radius: var(--radius-lg);
  text-decoration: none;
  color: inherit;
}
.card-event__media {
  position: relative;
  overflow: hidden;
  width: 72px;
  height: 72px;
  border-radius: 12px;
  flex: none;
}
```

**Store dengan validasi** — menggantikan tiga salinan `try { JSON.parse(localStorage…) } catch {}`:

```js
const STORAGE_KEY = 'eventTicketSelection';
const STORAGE_VERSION = 1;

export function saveSelection(selection) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify({ version: STORAGE_VERSION, ...selection }));
}

export function loadSelection() {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return null;
  try {
    const parsed = JSON.parse(raw);
    if (parsed.version !== STORAGE_VERSION) return null;
    if (!Number.isInteger(parsed.regular) || !Number.isInteger(parsed.vip)) return null;
    return parsed;
  } catch {
    return null;
  }
}

export function clearSelection() {
  localStorage.removeItem(STORAGE_KEY);
}
```

---

## 11. Checklist QA Sebelum Prototype Dianggap Selesai

- [ ] Semua 13 halaman memuat font Plus Jakarta Sans (cek di DevTools → Fonts)
- [ ] Setiap halaman punya tepat satu `<h1>` dan satu `<main>`
- [ ] Nol error/konsol peringatan di semua halaman
- [ ] Nol atribut `style=` yang mengatur layout/warna (hanya bila benar-benar dinamis)
- [ ] Nol selector `[style*="…"]`
- [ ] Alur penuh berjalan: Beranda → Cari → Detail → Masuk → Antrean → Pilih → Data → Harga → Bayar → **Berhasil** → Tiket Saya
- [ ] `ui-tunggu.html` tercapai dari alur nyata
- [ ] Tombol kembali di setiap halaman menuju halaman sebelumnya yang benar
- [ ] Navigasi keyboard penuh: Tab melewati semua kontrol, `Enter`/`Space` mengaktifkan, `←`/`→` bekerja di tab tiket
- [ ] Fokus terlihat di semua elemen interaktif (outline 3 px tidak terpotong)
- [ ] Layar 320 px, 360 px, 390 px, 430 px: tidak ada scroll horizontal, tidak ada teks terpotong
- [ ] `prefers-reduced-motion: reduce` mematikan semua transisi
- [ ] Harga konsisten dari detail → pilih → harga → bayar → berhasil (angka yang sama di 5 layar)
- [ ] Teks biaya layanan tidak saling bertentangan antar halaman
- [ ] `localStorage` kosong (incognito) tidak menampilkan angka pesanan palsu
- [ ] Kontras teks minimal 4.5:1 (cek `--muted-soft` di atas `--paper` dan `--accent-soft-text` di atas `--accent-soft-bg`)

---

## 12. Kesimpulan

Ini bukan pekerjaan yang perlu dibuang — sebaliknya, ini **desain yang sudah matang dengan implementasi yang belum dirapikan**. Lapisan visual (warna, ritme, ikon, microcopy, state hover/focus) sudah level produksi. Yang hilang adalah lapisan rekayasa:

- **satu sumber kebenaran untuk style** (tiga file CSS),
- **satu sumber kebenaran untuk data** (`events.js`),
- **satu sumber kebenaran untuk state** (`store.js`),
- dan **alur yang benar-benar tersambung** (antrean + pembayaran sukses).

Kerjakan P0 lebih dulu (CSS bersama + font + alur sukses + antrean). Itu sekitar satu hari kerja dan langsung mengubah prototype ini dari "kumpulan layar bagus" menjadi "aplikasi yang bisa didemokan ujung ke ujung" — sekaligus menghapus sebagian besar temuan di dokumen ini secara struktural, bukan tambal-sulam.
