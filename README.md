# pemkab-simalungun-web-vuln-report
Vulnerability Assessment Report – Pemerintah Kabupaten Simalungun

> **Target**: https://www.simalungunkab.go.id  
> **Tanggal Asesmen**: 2026  
> **Jenis**: Gray-box Reconnaissance & Public-Facing Misconfigurations  
> **Penemu**: 061CYBERSEC

## 📌 Ringkasan Eksekutif

Situs resmi Pemkab Simalungun (**Laravel-based**) menunjukkan sejumlah **kerentanan kritis dan high-risk** yang berpotensi mengarah pada:
- **Exposure data sensitif**
- **RCE potensial via env dan konfigurasi**
- **Session hijacking / privilege escalation**

---

## ⚠️ Temuan Kritis

### 🔓 **Exposed Laravel Application Directories**
- `/config/` → Semua file konfigurasi Laravel dapat di-download  
  Contoh: [database.php](https://www.simalungunkab.go.id/config/database.php), [app.php](https://www.simalungunkab.go.id/config/app.php)
- `/storage/logs/laravel.log` → **Log aplikasi terbuka**, mengandung stack trace & error developer
- `/storage/framework/` → Cache dan compiled views terakses publik

> ⛔ Akibat: **Source code leakage**, **DB credential exposure**, **route enumeration**, **app key exposure**

### 🚫 **Missing Application Key (`APP_KEY`)**
- Banyak error menyatakan: `No application encryption key has been specified`
- Tanpa `APP_KEY`, enkripsi session & cookie **tidak akan bekerja**
- **Session bisa direkayasa / dipalsukan**

### 🧨 **Database Connection Failed (`root@localhost`)**
- Error SQL: `Access denied for user 'root'@'localhost' (using password: NO)`
- Menunjukkan aplikasi mencoba koneksi ke MySQL **tanpa password**, dengan user root → **Konfigurasi berbahaya**

### 🧩 **Broken Routes & Undefined Functions**
- `Call to undefined function site_url()` → Mengindikasikan **kode tidak lengkap / deploy gagal**
- `Target class [HomeController] does not exist` → **Route terdaftar, tapi controller hilang**

### 🔁 **Authentication Flow Broken**
- Error berulang: `Route [login] not defined`
- Middleware auth mencoba redirect ke `/login`, tapi route tersebut **tidak ada**

---


🛡️ Rekomendasi Mitigasi
1. **Lindungi direktori sensitif** via `.htaccess` / Nginx rules:
   ```nginx
   location ~ ^/(config|storage|vendor|\.env) { deny all; }
