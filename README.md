# mooc-app (versi ter-hardening)

Frontend statis (SPA JSON-driven, pola sama dengan `cms-app`) untuk
MOOC IPWIJA. Lihat `SECURITY.md` untuk daftar lengkap perubahan
dibanding versi sebelumnya, dan `../mooc-api-fixed/` untuk backendnya.

## Menjalankan lokal

Situs ini statis murni — cukup file server apa pun (mis.
`npx serve .` atau ekstensi Live Server) dari folder ini. **Tidak bisa
dibuka langsung lewat `file://`** (fetch ke API akan diblokir CORS).

Pastikan origin lokal Anda (mis. `http://localhost:8080`) ada di daftar
`ALLOWED_ORIGINS` sisi `mooc-api` (lihat `wrangler.toml` di sana —
`http://localhost:8080` & `http://127.0.0.1:8080` sudah termasuk
default).

## Konfigurasi

Satu-satunya hal yang perlu disesuaikan ke deployment Anda: `API_ORIGIN`
di `db.js` (baris pertama) — harus sama persis dengan domain Worker
`mooc-api` Anda, dan `preconnect`/`connect-src` CSP di `index.html`
harus ikut disamakan.

## Akun demo

Sama dengan `mooc-api/README.md`: `admin/admin123`, `dosen/dosen123`,
`peserta/peserta123` — ganti/hapus sebelum produksi.
