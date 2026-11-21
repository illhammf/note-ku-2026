<div align="center">

# 🚀 note-ku

### Laravel + Filament + Docker · Project 2026

Built with ❤️ using Laravel, Filament, Docker & WSL

![Laravel](https://img.shields.io/badge/Laravel-12-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-8.3-777BB4?style=for-the-badge&logo=php&logoColor=white)
![Filament](https://img.shields.io/badge/Filament-v3-F59E0B?style=for-the-badge)
![MariaDB](https://img.shields.io/badge/MariaDB-10.11-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)

[![GitHub](https://img.shields.io/badge/Open-GitHub-black?style=for-the-badge&logo=github)](https://github.com/illhammf/note-ku-2026)

</div>

> ℹ️ README ini dibuat otomatis oleh `start.sh` (Ilham Boilerplate) pada 21-11-2026 22:31.
> Silakan diedit sesuai kebutuhan proyek.

---

## 📑 Daftar Isi

- [Tentang Project](#-tentang-project)
- [URL Aplikasi](#-url-aplikasi)
- [Setup untuk Anggota Tim](#-setup-untuk-anggota-tim-setelah-clone)
- [Perintah Sehari-hari](#-perintah-sehari-hari)
- [Alur Membuat Fitur CRUD](#-alur-membuat-fitur-crud)
- [Docker Services & Database](#-docker-services--database)
- [Struktur Project](#-struktur-project)
- [Troubleshooting](#-troubleshooting)
- [Checklist Pengembangan](#-checklist-pengembangan)

---

## 📖 Tentang Project

**note-ku** adalah aplikasi web berbasis Laravel 12 dengan panel admin Filament 3,
dijalankan sepenuhnya di Docker (Nginx HTTPS + PHP 8.3-FPM + MariaDB 10.11).

| Fitur Bawaan | Keterangan |
|---|---|
| ✅ Laravel 12 + Filament 3 | Panel admin siap pakai di `/admin` |
| ✅ Filament Shield & Logger | Manajemen role/permission dan jejak aktivitas |
| ✅ HTTPS lokal | Domain `https://note-ku.test` via mkcert |
| ✅ Setup otomatis | `src/.env`, `APP_KEY`, dan migrate dibuat oleh entrypoint |
| ✅ Alias terminal | `dca`, `dcm`, `dcf`, `dcp`, dan lainnya |

---

## 🌐 URL Aplikasi

| Halaman | URL |
|---|---|
| Website | https://note-ku.test |
| Admin Panel | https://note-ku.test/admin |
| Repository | https://github.com/illhammf/note-ku-2026 |

---

## 👥 Setup untuk Anggota Tim (setelah clone)

Jalankan dari **WSL Ubuntu**. Pastikan Docker, mkcert, dan git sudah terpasang.

```bash
# 1. Clone dan masuk ke folder proyek
git clone https://github.com/illhammf/note-ku-2026.git
cd note-ku-2026

# 2. Buat file .env untuk Docker Compose
cp .env.example .env

# 3. Buat sertifikat SSL lokal
mkdir -p nginx/ssl
mkcert -install
mkcert -cert-file nginx/ssl/note-ku.test.crt -key-file nginx/ssl/note-ku.test.key note-ku.test localhost 127.0.0.1

# 4. Daftarkan domain ke hosts WSL
echo "127.0.0.1 note-ku.test" | sudo tee -a /etc/hosts

# 5. Build image lalu pasang dependensi Composer (progres terlihat langsung)
docker compose build
docker compose run --rm --no-deps -w /var/www/html --entrypoint composer php install

# 6. Jalankan kontainer (src/.env, APP_KEY, dan migrate dibuat otomatis)
docker compose up -d
docker compose logs -f php   # tunggu sampai muncul "Ready to serve!", lalu Ctrl+C
```

Tambahkan juga baris `127.0.0.1 note-ku.test` ke file hosts Windows
(`C:\Windows\System32\drivers\etc\hosts`, buka Notepad sebagai Administrator).

> ⚠️ Jangan menjalankan `docker compose restart php` saat setup pertama masih berjalan.

---

## ⚡ Perintah Sehari-hari

| Command | Deskripsi |
|---|---|
| `dcu` | Jalankan kontainer (`docker compose up -d`) |
| `dcd` | Matikan kontainer proyek ini |
| `dca <perintah>` | Perintah Artisan di kontainer PHP (contoh: `dca migrate`) |
| `dci` | Jalankan `php artisan project:init` |
| `dcp "pesan"` | Git add, commit, pull --rebase, lalu push |
| `dcm ModelName` | Buat model + migration + seeder + controller |
| `dcf ModelName` | Buat Filament resource dari kolom tabel (setelah `dca migrate`) |
| `dcr ModelName` | Hapus model, migration, seeder, controller, policy, dan resource |
| `docker compose exec php bash` | Masuk ke kontainer PHP |
| `docker compose logs -f php` | Lihat log kontainer PHP |

---

## 🧩 Alur Membuat Fitur CRUD

```text
dcm Nama  →  isi file migration  →  dca migrate  →  dcf Nama
```

Jika nama tabel tidak mengikuti aturan jamak bahasa Inggris, set di model:
`protected $table = 'nama_tabel';`

---

## 🐳 Docker Services & Database

| Service | Port |
|---|---|
| Nginx | 80 / 443 |
| PHP-FPM | Internal (9000) |
| MariaDB | 13306 (host) |

Koneksi Navicat/DBeaver: host `127.0.0.1`, port `13306`, user `root`, password `p455w0rd`,
database `note-ku`.

---

## 📂 Struktur Project

```text
.
├── db/              # Konfigurasi & data MariaDB (data tidak di-commit)
├── nginx/           # Konfigurasi Nginx & sertifikat SSL lokal (ssl/ tidak di-commit)
├── php/             # Dockerfile, entrypoint, konfigurasi PHP
├── src/             # Aplikasi Laravel
├── .env.example     # Salin menjadi .env (dibaca Docker Compose)
└── docker-compose.yml
```

---

## 🩺 Troubleshooting

| Gejala | Solusi |
|---|---|
| `502 Bad Gateway` | PHP belum siap. Cek `docker compose logs -f php`, tunggu "Ready to serve!" |
| `vendor/autoload.php` tidak ada | `docker compose stop php` → `rm -rf src/vendor` → perintah composer di langkah 5 → `docker compose up -d` |
| `APP_KEY` kosong | Pastikan vendor lengkap dulu, lalu `docker compose restart php` |
| Domain tidak bisa dibuka | Cek baris `127.0.0.1 note-ku.test` di hosts WSL **dan** Windows |
| Port 80/443/13306 dipakai | Matikan proyek lain dulu dengan `dcd` di folder proyek tersebut |
| `git push` lambat/timeout | DNS WSL lambat; ulangi beberapa saat lagi |

---

## 📋 Checklist Pengembangan

- [ ] Setup database & migration
- [ ] Setup role & permission (Filament Shield)
- [ ] Membuat resource admin
- [ ] Membuat halaman pengguna
- [ ] Seeder data awal
- [ ] Pengujian
- [ ] Deploy

---

<div align="center">

👨‍💻 Developer: **illhammf** · Generated with **Ilham Boilerplate 2026**

> "Konsistensi Mengalahkan Bakat Saat Bakat Tidak Muncul."

🔥 Happy Coding Yaa!!!

</div>
