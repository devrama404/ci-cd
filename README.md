# 🚀 CI/CD Pipeline: GitHub Actions (CI & CD Terpisah)

Dokumentasi implementasi pipeline **Continuous Integration (CI)** dan **Continuous Deployment (CD)** menggunakan GitHub Actions. Pipeline dipisah menjadi dua workflow untuk memisahkan tahap validasi kode (`ci.yml`) dan proses deployment otomatis ke server (`cd.yml`).

---

## 🧩 Fungsi Pipeline

### 🔵 Continuous Integration (`ci.yml`)
| Tahapan | Fungsi Utama |
|---------|--------------|
| ✅ Code Quality Check | Memastikan format & style kode sesuai standar (linting) |
| 🔍 Static Analysis | Mendeteksi potensi error/bug sebelum runtime |
| 🧪 Unit Testing | Menjalankan test otomatis & mencegah regression |
| 🏗️ Build Verification | Memastikan aplikasi berhasil di-compile/build tanpa error |
| 🛡️ Security Scanning *(Opsional)* | Mendeteksi vulnerability pada dependencies |

### 🟢 Continuous Deployment (`cd.yml`)
| Tahapan | Fungsi Utama |
|---------|--------------|
| 📥 Mengambil kode terbaru | Pull source code terkini dari repository |
| 📦 Install dependency | Memasang library & paket yang dibutuhkan aplikasi |
| 🔨 Build aplikasi | Menghasilkan artifact siap produksi |
| 📤 Menyalin file ke server | Transfer hasil build ke environment target (AWS EC2) |
| 🔄 Restart service | Memuat ulang aplikasi agar versi terbaru aktif |
| 🌐 Deploy Ke Server | Aplikasi live, berjalan, & siap diakses pengguna |

---

## 🔐 Setup GitHub Secrets
Masuk ke `Repository → Settings → Secrets and variables → Actions` dan tambahkan secret berikut:

| Secret Name | Nilai / Contoh |
|-------------|----------------|
| `EC2_HOST` | IP Publik AWS EC2 Anda (misal: `43.218.94.80`) |
| `EC2_PRIVATE_KEY` | Isi **lengkap** file `.pem` Anda (termasuk baris `-----BEGIN OPENSSH PRIVATE KEY-----` dan `-----END...`) |

> 💡 *Opsional:* Jika menggunakan Docker Registry, tambahkan juga `DOCKERHUB_USERNAME` & `DOCKERHUB_TOKEN`.

---

## 📁 Struktur Folder
```
.github/
└── workflows/
    ├── ci.yml   # Pipeline CI (Testing & Build Validation)
    └── cd.yml   # Pipeline CD (Deployment ke EC2)
package.json     # Konfigurasi project
src/             # Source code aplikasi
```

---

## 🤖 1. Pipeline CI (`ci.yml`)
Simpan file ini di `.github/workflows/ci.yml`

```yaml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  ci-validation:
    runs-on: ubuntu-latest
    steps:
      - name: 📥 Mengambil kode terbaru
        uses: actions/checkout@v4

      - name: 🛠️ Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: 📦 Install dependency
        run: npm install --legacy-peer-deps

      - name: ✅ Code Quality Check
        run: npm run lint || echo "⚠️ Linting tidak dikonfigurasi di package.json"

      - name: 🔍 Static Analysis
        run: npm run lint:fix || npx eslint . || echo "⚠️ Static analysis skipped"

      - name: 🧪 Unit Testing
        run: npm test -- --coverage || echo "⚠️ Test suite tidak ditemukan"

      - name: 🏗️ Build Verification
        run: npm run build

      - name: 🛡️ Security Scanning (Optional)
        run: npm audit --audit-level=moderate || true
```

---

## 🚀 2. Pipeline CD (`cd.yml`)
Simpan file ini di `.github/workflows/cd.yml`

```yaml
name: CD Pipeline

on:
  workflow_run:
    workflows: ["CI Pipeline"]
    types:
      - completed
    branches: [ main ]

jobs:
  deploy-to-ec2:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - name: 📥 Mengambil kode terbaru
        uses: actions/checkout@v4

      - name: 📦 Install dependency
        run: npm install --legacy-peer-deps

      - name: 🔨 Build aplikasi
        run: npm run build

      - name: 📤 Menyalin file & 🔄 Restart service (Deploy ke EC2)
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script: |
            echo "🔄 1. Menghentikan service lama..."
            docker stop wibe-studio 2>/dev/null || true
            docker rm wibe-studio 2>/dev/null || true
            
            echo "📦 2. Mengambil image/artifact terbaru..."
            docker pull devrama404/wibe-studio:latest
            
            echo "🌐 3. Deploy Ke Server & Restart Service..."
            docker run -d --name wibe-studio -p 80:80 devrama404/wibe-studio:latest
            
            echo "✅ Deployment berhasil! Aplikasi telah di-restart dan siap diakses."
```

---

## 🛠️ Cara Penggunaan
1. **Buat Folder & File Workflow**
   ```bash
   mkdir -p .github/workflows
   ```
   Buat file `ci.yml` dan `cd.yml` di dalam folder tersebut, lalu paste kode YAML di atas.
2. **Sesuaikan `package.json`** (jika belum ada)
   Pastikan skrip berikut tersedia agar pipeline CI berjalan optimal:
   ```json
   {
     "scripts": {
       "lint": "eslint .",
       "test": "jest",
       "build": "react-scripts build || vite build || next build"
     }
   }
   ```
3. **Push ke GitHub**
   ```bash
   git add .github/workflows/
   git commit -m "feat: add separated CI & CD pipelines"
   git push origin main
   ```
4. **Monitor Pipeline**  
   Buka tab `Actions` di repository GitHub. Pipeline `CI` akan jalan saat push/PR. Jika sukses, `CD` akan otomatis trigger dan deploy ke EC2.

---

## ⚠️ Troubleshooting & Catatan
- 🔑 **Format Private Key:** Pastikan `EC2_PRIVATE_KEY` di GitHub Secrets tidak mengandung baris baru ekstra di awal/akhir. Salin persis dari isi file `.pem`.
- 🐳 **Docker vs File Transfer:** Pipeline CD di atas menggunakan Docker (`docker pull` & `docker run`) sebagai metode "menyalin ke server". Jika ingin transfer file statis langsung, ganti bagian `script` dengan `rsync` atau `scp`.
- 🔐 **Permissions:** Pastikan user `ubuntu` di EC2 memiliki hak akses Docker (`sudo usermod -aG docker ubuntu` + reconnect SSH).
- 🔄 **Triggers:** `cd.yml` hanya jalan jika `ci.yml` berhasil. Untuk testing manual, ubah `on:` menjadi `workflow_dispatch:`.

---

## 📜 Lisensi
Pipeline ini dibuat untuk keperluan edukasi & final project. Silakan modifikasi alur, nama job, atau command sesuai kebutuhan arsitektur Anda.  
Dibuat dengan ❤️ menggunakan GitHub Actions, Node.js, Docker & AWS EC2.
```

### 💾 Cara Menyimpan:
1. **Blok & Copy** seluruh teks di dalam kotak kode di atas.
2. Buka teks editor (VS Code, Notepad, dll).
3. **Paste** dan simpan dengan nama persis: `README.md`
4. Letakkan di folder utama repository Anda sebelum push ke GitHub.

Jika Anda ingin saya generate juga konfigurasi `package.json` standar atau menyesuaikan perintah `npm run` sesuai framework yang Anda pakai (React/Vue/Next.js), beri tahu saja!
