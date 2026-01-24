# Tripay Ocestra

Project ini adalah **sistem terpusat** untuk mengelola dan mengoordinasikan beberapa aplikasi Tripay yang saling terintegrasi menggunakan Docker.

## 📦 Aplikasi yang Tersedia

Project ini menggabungkan 4 aplikasi dalam satu environment:

| Aplikasi         | Teknologi | Repository                                                                         | Branch            |
| ---------------- | --------- | ---------------------------------------------------------------------------------- | ----------------- |
| Dashboard Tripay | Nuxt.js   | [dashboard.tripay.co.id](https://github.com/trijayadigital/dashboard.tripay.co.id) | `dev`             |
| Trihariyanto     | Laravel   | [trihariyanto.com](https://github.com/trijayadigital/trihariyanto.com)             | `main`            |
| Tripay Payment   | Laravel   | [tripay-payment](https://github.com/trijayadigital/tripay-payment)                 | `feature/new-sso` |
| Tripay PPOB      | Laravel   | [tripay-ppob](https://github.com/trijayadigital/tripay-ppob)                       | `main`            |

---

## 🛠️ Instalasi

### Langkah 1: Clone Repository Utama

```bash
git clone https://github.com/trijayadigital/tripay-ocestra.git
cd tripay-ocestra
```

### Langkah 2: Clone Semua Aplikasi

```bash
# Clone dengan branch yang sesuai
git clone -b feature/new-sso https://github.com/trijayadigital/tripay-payment.git
git clone -b main https://github.com/trijayadigital/tripay-ppob.git
git clone -b dev https://github.com/trijayadigital/dashboard.tripay.co.id.git
git clone -b main https://github.com/trijayadigital/trihariyanto.com.git
```

### Langkah 3: Setup Database

> [!IMPORTANT]
> Jangan lupa membuat database di **local machine** (MySQL) untuk setiap aplikasi sebelum menjalankan Docker.

```sql
CREATE DATABASE tripay_payment;
CREATE DATABASE tripay_ppob;
```

### Langkah 4: Jalankan Docker

```bash
docker compose up --build
```

---

## 🌐 Cara Mengakses Aplikasi

Setelah Docker berjalan, buka browser dan akses URL berikut:

| Aplikasi         | URL Browser                         | Keterangan      |
| ---------------- | ----------------------------------- | --------------- |
| Dashboard Tripay | `http://dashboard.tripay.localhost` | Dashboard utama |
| Trihariyanto     | `http://trihariyanto.localhost`     | SSO & Auth      |
| Tripay Payment   | `http://tripay.localhost`           | Payment gateway |
| Tripay PPOB      | `http://tripay-ppob.localhost`      | PPOB service    |

### 🔐 Login ke Tripay Payment

> [!IMPORTANT]
> Halaman login Tripay Payment menggunakan **SSO yang di-hardcode**.

Untuk login, **HARUS** menggunakan URL:

```
http://tripay.localhost/member
```

❌ **Jangan** akses `http://tripay.localhost` langsung (tidak ada form login)  
✅ **Gunakan** `http://tripay.localhost/member` untuk SSO login

---

## ⚠️ Penting: Komunikasi Antar Backend

### 🖥️ Kapan Menggunakan Domain `.localhost`?

**Hanya untuk akses dari BROWSER** (Chrome, Firefox, Safari, dll)

Contoh yang **BENAR** dari browser:

- ✅ `http://trihariyanto.localhost`
- ✅ `http://tripay.localhost`
- ✅ `http://dashboard.tripay.localhost`

---

### 🔗 Komunikasi Backend-to-Backend

> [!WARNING]
> **JANGAN PERNAH** gunakan domain `.localhost` di dalam kode backend!

#### Kenapa?

Domain `.localhost` **hanya bisa diakses** oleh komputer Anda (host machine), **TIDAK** oleh container Docker.

Ketika backend di dalam container mencoba akses `trihariyanto.localhost`, Docker **tidak tahu** apa itu domain tersebut.

#### ❌ Contoh SALAH (Tidak Akan Bekerja)

```php
// Di dalam container Docker - AKAN ERROR!
$response = Http::get('http://trihariyanto.localhost/api/users');
curl('http://tripay.localhost/api/payment');
```

**Error yang muncul:**

```
Could not resolve host: trihariyanto.localhost
```

#### ✅ Contoh BENAR (Gunakan Service Name)

```php
// Gunakan nama service Docker - BEKERJA!
$response = Http::get('http://trihariyanto:8000/api/users');
curl('http://tripay-payment:8000/api/payment');
curl('http://dashboard:3000/api/data');
```

---

### 📋 Daftar Service Names untuk Backend

Gunakan tabel ini saat menulis kode backend yang perlu komunikasi antar service:

| Service Name     | Port | Aplikasi         | Contoh URL                            |
| ---------------- | ---- | ---------------- | ------------------------------------- |
| `trihariyanto`   | 8000 | trihariyanto.com | `http://trihariyanto:8000/endpoint`   |
| `tripay-payment` | 8000 | tripay-payment   | `http://tripay-payment:8000/endpoint` |
| `tripay-ppob`    | 8000 | tripay-ppob      | `http://tripay-ppob:8000/endpoint`    |

---

## 🛠️ Menjalankan Command (Artisan/Composer)

> [!WARNING]
> **JANGAN** menjalankan command `php artisan` atau `composer` langsung dari terminal di komputer Anda (macOS).
>
> Jika dijalankan dari luar Docker, PHP tidak akan bisa menemukan hostname `redis` atau database yang dikonfigurasi via Docker network.

### Cara yang Benar (Via Docker Exec)

Gunakan `docker exec` untuk menjalankan command di dalam container yang sedang berjalan:

**1. Menjalankan Artisan:**

```bash
docker exec -it <container_name> php artisan <command>
```

_Contoh:_

```bash
docker exec -it tripay-ppob php artisan migrate
docker exec -it tripay-payment php artisan tinker
```

**2. Menjalankan Composer:**

```bash
docker exec -it <container_name> composer <command>
```

_Contoh:_

```bash
docker exec -it tripay-trihariyanto composer install
```

### Nama Container yang Tersedia:

- `tripay-dashboard`
- `tripay-trihariyanto`
- `tripay-payment`
- `tripay-ppob`

---

### 💡 Ringkasan Cepat

| Skenario                 | URL yang Digunakan  | Contoh                           |
| ------------------------ | ------------------- | -------------------------------- |
| **Akses dari Browser**   | Domain `.localhost` | `http://tripay.localhost`        |
| **Backend → Backend**    | Service name Docker | `http://tripay-payment:8000`     |
| **Login Tripay Payment** | `/member` path      | `http://tripay.localhost/member` |
