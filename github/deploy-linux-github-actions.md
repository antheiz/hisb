# Panduan Deployment Aplikasi ke Server Linux dengan GitHub Actions

Panduan ini menjelaskan langkah-langkah untuk melakukan _deployment_ aplikasi ke server Linux menggunakan GitHub Actions. Pastikan semua prasyarat telah dipenuhi sebelum memulai.

## Prasyarat

1. Server
    - Web server (misalnya, Nginx) telah diinstal.
    - SSH telah diaktifkan.
2. Konfigurasi Web Server
    - Konfigurasi server block telah selesai.
3. Process Manager
    - Telah menginstal process manager seperti PM2 atau Supervisor _(opsional)_.
4. SSH Key
    - SSH key telah dibuat di server untuk keperluan _deployment_.
5. GitHub Repository
    - Repository proyek telah dibuat di GitHub.

## 1. Membuat SSH Key di Server

Untuk membuat SSH key di server, jalankan perintah berikut:

```sh
ssh-keygen
```

Jika diminta untuk memberi nama pada key, Anda dapat menggunakan nama apa pun. Dalam contoh ini, nama yang digunakan adalah `fighter`. Perintah ini akan menghasilkan dua file: _public key_ dan _private key_. File _public key_ biasanya memiliki ekstensi `.pub`, misalnya `fighter.pub`.

Selanjutnya, salin isi dari _public key_ (**fighter.pub**) ke file **authorized_keys** untuk memberikan akses SSH:

```sh
cat fighter.pub >> ~/.ssh/authorized_keys
```

## 2. Membuat Konfigurasi GitHub Actions

Buat file konfigurasi GitHub Actions pada direktori `.github/workflows/`. Anda bisa memberi nama file secara bebas, misalnya `deploy-server.yml`. Berikut contoh isi dari file tersebut:

```sh
# ./github/workflows/deploy-server.yml

name: 🚀 Publish Website to Production Environment

on:
  release:
    types: [published]

jobs:
  web-deploy:
    name: 🚀 Deploy Website on Release
    runs-on: ubuntu-latest

    steps:
    # deploy server config
    - name: 🔐 Install SSH Key
      uses: shimataro/ssh-key-action@v2
      with:
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        known_hosts: github.com

    - name: 📂 Pull Latest Code on Server   
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        port: ${{ secrets.SERVER_PORT }}
        script: |
          # Menjalankan ssh-agent untuk mengelola kunci SSH
          # source: https://github.com/appleboy/ssh-action/issues/232#issuecomment-1849260489
          eval `ssh-agent -s`
          ssh-add  ~/.ssh/fighter

          # Pindah ke direktori proyek
          cd /var/www/

          # Clone repositori privat
          if [ ! -d "project-backend" ]; then
            git clone git@github.com:antheiz/project-backend.git project-backend
          else
            cd project-backend
            # Fetch latest changes
            git fetch origin main
            # Reset local changes and ensure latest code is applied
            git reset --hard origin/main
          fi

    - name: 🧹 List Files on Server
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        port: ${{ secrets.SERVER_PORT }}
        script: |
          echo "Listing files in /var/www/project-backend/"
          ls -la /var/www/project-backend/

          # Tambahkan pm2 ke PATH
          export PATH=$PATH:/usr/bin/pm2

          # Restart App
          /usr/bin/pm2 restart all
```

**Penjelasan:**

1. **SSH Key**: Menginstal SSH key yang disimpan sebagai secret di repository GitHub.
2. **Code Pulling**: Menggunakan SSH untuk menarik kode terbaru dari repository GitHub ke server.
3. **Listing & Restart**: Menampilkan daftar file yang telah disebarkan dan merestart aplikasi menggunakan PM2.

## 3. Menambahkan Environment Variables di Repository

Pada file GitHub Actions, terdapat variabel environment yang diperlukan, seperti **SSH_PRIVATE_KEY**, **SERVER_HOST**, dan lainnya. Anda harus menambahkan variabel ini melalui pengaturan repository di GitHub:

1. Buka tab **Settings** di repository.
2. Pilih **Secrets and Variables**.
3. Klik **New Repository Secret** dan tambahkan variabel yang dibutuhkan.

## 4. Menambahkan Deploy Keys di Repository

Karena repository yang diakses bersifat privat, Anda harus menambahkan _public key_ ke _Deploy Keys_ di GitHub dengan tipe _read-only_. Langkah-langkahnya:

1. Buka tab **Settings** di repository.
2. Pilih **Deploy Keys**.
3. Klik **Add Deploy Key** dan masukkan isi dari file `fighter.pub`.

## 5. Menambahkan GitHub sebagai Known Host di Server

Untuk menghindari peringatan host key saat melakukan koneksi ke GitHub, tambahkan GitHub sebagai _known host_ di server dengan perintah berikut:

```sh
ssh-keyscan github.com >> ~/.ssh/known_hosts
```

## 6. Menjalankan GitHub Actions

Setelah semua konfigurasi selesai:

1. Simpan file konfigurasi GitHub Actions.
2. Buat release baru pada repository GitHub Anda.

Proses _workflow_ akan berjalan secara otomatis, dan aplikasi akan disebarkan ke server. Jika berhasil, aplikasi Anda akan siap digunakan di server.