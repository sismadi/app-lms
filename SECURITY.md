# Keamanan & Perubahan `mooc-app`

Pelengkap `mooc-api/SECURITY.md` — dokumen itu menjelaskan sisi server,
ini menjelaskan yang berubah di frontend statis. Pola mengikuti
`cms-app` (piawai).

## Temuan sisi frontend, dari yang paling parah

| # | Temuan | Perbaikan |
|---|---|---|
| KRITIS | `db.js` lama membuat auth.login() mengunduh SELURUH tabel `users` (termasuk password plaintext semua akun) ke browser, hanya untuk mencocokkan 1 baris (`db.find('users', u => ...)`). | `db.js` baru murni pemanggil API (`apiPublic`/`apiAuth`); login mengirim `{username,password}` ke server, menerima token — tidak pernah mengunduh tabel user. |
| KRITIS | Kunci jawaban kuis (`quiz.questions[i].ans`) dikirim ke peserta dan dicocokkan di `quiz.js` versi lama (`web.evaluateQuiz`, `btoa(selected)===q.ans`, semua di browser). | `quiz.js` baru: soal diambil TANPA `ans` (`db.quizPublic`), jawaban dikirim ke server untuk dinilai (`db.quizSubmit`) — lihat `mooc-api/SECURITY.md` KRITIS 3. |
| SEDANG | **Tidak ada fungsi escape HTML sama sekali** di `script.js` lama — `genericForm`, `renderTable`, `titleHero`, `hero`, komponen `certificate` (nama peserta pada sertifikat!), dll menyisipkan data pengguna langsung ke `innerHTML`. | Ditambahkan `escHtml()`/`safeUrl()` (lihat `script.js`), diterapkan di **setiap** titik yang menyisipkan data pengguna: form generik, tabel (dengan `rawKeys` untuk kolom "Aksi" yang memang HTML), judul kursus/materi, nama & skor di sertifikat, embed gambar/PDF/video. |
| SEDANG | Sertifikat (`components.certificate`) menampilkan `d.name` (nama profil PESERTA, bisa diedit bebas) & `d.exam` (judul kuis, bisa diedit DOSEN) tanpa escape, di halaman **publik** `/?cert/<id>` yang bisa dibuka siapa saja tanpa login. | Paling berisiko dari semua celah XSS di sini — peserta mana pun bisa mendaftar, mengisi nama profil dengan payload, lulus 1 kuis, lalu membagikan tautan sertifikatnya. Diperbaiki dengan `escHtml()` di komponen `certificate`. |
| RENDAH | `admin.js` lama memanggil beberapa fungsi `db.*`/`courseSvc.*` secara sinkron padahal sudah async (bug, bukan celah keamanan langsung, tapi indikasi migrasi setengah jalan). | `admin.js` baru konsisten `async/await`, 1 panggilan teragregasi (`db.adminStats()`). |
| RENDAH | `preconnect`/domain API tersebar di beberapa tempat tanpa jaminan konsisten (pola yang sama pernah ditemukan sebagai bug di `cms-app`: preconnect menunjuk domain yang salah). | `API_ORIGIN` di `db.js` adalah SATU-SATUNYA sumber kebenaran; `index.html` `preconnect`/`dns-prefetch` disalin persis dari nilai itu. |

## Perubahan struktural: "semua form dalam drawer"

Sebelumnya HANYA form dosen/admin/kuis (dibangun lewat
`web.openFormFromPage`) yang tampil di drawer kanan
(`#formDrawerPanel`); form akun (masuk, daftar, lupa password, reset
password, pengaturan profil) dirender **inline** di badan halaman.

Sekarang **semua form** — termasuk seluruh alur akun — dibuka lewat
drawer yang sama, lewat mekanisme baru `web.AUTO_DRAWER_ROUTES` +
`web.openFormFromPage(pageData, {silent:true})` yang dipanggil otomatis
di `web.navigate()` (lihat `script.js`) untuk rute `login`, `daftar`,
`lupa-password`, `reset-password`, dan `settings`. Resolver-nya
(`auth.js`) TIDAK diduplikasi — bentuk `{ rightCol: { fields, ... } }`
yang sama dipakai baik untuk latar halaman maupun isi drawer, jadi
tidak ada 2 sumber kebenaran untuk 1 form.

## Perubahan lain yang perlu diketahui (bukan celah keamanan)

- **Kartu "Modul Terakhir Dilihat" di Dashboard dihapus.** Versi lama
  menyimpannya di `localStorage` lintas-kursus; sekarang `lastId`
  tersimpan **per kursus** di server (`progress.lastId`) — cukup untuk
  menandai posisi terakhir di tiap kursus, tapi tidak lagi ada satu
  nilai "modul terakhir secara global" untuk ditampilkan sebagai 1
  kartu ringkas tanpa endpoint tambahan. Trade-off yang disengaja demi
  kesederhanaan; bisa ditambahkan lagi lewat endpoint baru bila
  dibutuhkan.
- **Form edit soal kuis tidak lagi pra-isi jawaban benar.** Dosen tetap
  BERHAK melihatnya (lewat `?view=quiz-admin`), tapi form edit meminta
  mengetik ulang jawaban benar saat mengedit — lebih sederhana &
  konsisten dengan form tambah soal, daripada mem-`atob()` lalu
  menaruhnya di atribut `value` HTML.
- **Halaman lupa-password tidak lagi pernah menampilkan tautan reset
  mentah sebagai fallback.** Server sekarang SELALU membalas
  `{ok:true}` yang sama baik email ditemukan atau tidak (lihat
  `mooc-api/SECURITY.md`) — menampilkan tautan mentah di UI akan
  membocorkan status "email ini terdaftar/tidak" lewat cara lain.

## File yang dihapus

- `config.js` — kosong (0 byte), tidak dipakai di mana pun; dihapus
  beserta tag `<script>`-nya di `index.html`.
