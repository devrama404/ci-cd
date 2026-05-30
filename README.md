Membuat Pipeline Ci-Cd Menggunakan Github actions
1. Buat folder .github/workflows terlebih dahulu
2. kemudian buat pipeline ci.yml
3. dan ketiga buat pipeline cd.yml

Secrets yang dibutuhkan

Repository → Settings → Secrets

EC2_HOST
EC2_PRIVATE_KEY

Fungsi-fungsi CI
1. Code Quality Check

Memastikan format kode sesuai standar.

2. Static Analysis

Mendeteksi kesalahan sebelum aplikasi dijalankan.
Menemukan bug lebih cepat
Mengurangi error saat runtime

3. Unit Testing

Menjalankan test otomatis.
Memastikan fitur lama tidak rusak
Mendeteksi regression

4. Build Verification

Memastikan aplikasi bisa dibangun (build).
Mengetahui error build sebelum deploy

5. Security Scanning (opsional)

Mendeteksi dependency yang rentan.

Fungsi-fungsi CD

1. Mengambil kode terbaru

Tujuannya agar server mendapatkan source code terbaru dari repository.

2. Install dependency

Tujuannya agar library yang dibutuhkan aplikasi terpasang.

3. Build aplikasi

Tujuannya menghasilkan artifact siap jalan di server.

4. Menyalin file ke server

Tujuannya memindahkan hasil build ke environment tujuan.

5. Restart service

Tujuannya agar aplikasi menggunakan versi terbaru.

6. Deploy Ke Server

Tujuannya agar aplikasi bisa diakses oleh user/pengguna
