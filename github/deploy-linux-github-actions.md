# _Deploying_ Aplikasi ke Server Linux dengan GitHub Actions

Melakukan penyebaran aplikasi ke server dengan GitHub Actions (GH), ikuti panduan berikut:

## Prasyarat

1. Server telah diinstall web server (mis: nginx) dan ssh
2. Konfigurasi web server telah dilakukan (termasuk mengatur server block)
3. Telah menginstall process manager seperti pm2 atau supervisor _(optional)_
4. Telah membuat ssh key di server untuk digunakan sebagai proses _deployment_
5. Telah membuat project repository di GitHub

## 1. Buat ssh-key di Server

Buat ssh-key di server dengan menjalankan perintah `ssh-keygen`, ketika perintah dijalankan, jika diminta beri nama untuk `key`-nya, silahkan lengkapi dengan nama yang diinginkan. kali ini sa beri nama `fighter`. Hasil dari perintah tersebut akan membuat dua file, yaitu _public key_ dan _private key_. public key dapat dikenal dengan nama yang berakhir dengan ekstensi `.pub`, misal `fighter.pub`.

Setelah itu, copy isi dari `public_key` ke file `authorized_keys`.

## 2. Buat Konfigurasi Github Actions

Buatkan file github actions pada directory `.github/workflows` file yang dibuat bisa diberi nama bebas, kali ini sa beri nama `deploy-server.yml`. Berikut contoh isi konfigurasi dari file tersebut:

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

Pada file tersebut terdapat beberapa konfigurasi yang telah diatur, secara umum tahapan tersebut menunjukan proses deploy aplikasi ke server linux dengan trigger actions berdasarkan release terbaru. dan mengakses `ssh-private-key` yang disimpan pada environment dari repository termasuk `server-host`, `server_username`, dan `server_port`. dan proses cloning private repository ke server linux dengan menggunakan `ssh-key` yang dibuat sebelumnya. Setelah proses tersebut ditampilkan daftar file yang disebarkan ke server termasuk merestart aplikasi yang dijalankan dengan pm2.

## 3. Buat Environment Variable di Repository

Karena pada file gh terdapat akses variabel pada environment repository, maka perlu dibuatkan pada halaman environtment di repository github. Ini dapat diakses setelah klik menu `settings` pada repository. setelah mengakses, buatkan setiap variable environment tersebut pada pada tab `secrets`, klik button New repository secrets.

## 4. Buatkan Deploy Keys di Repository

Selanjutnya, karena pada pipelin, repository yang diakses bersifat private, maka perlu untuk menambahkan `ssh public key` ke deploy keys dengan type read only.
Jadi buatkan sebuah key dan isi key tersebut adalah ssh public key, jika berdasarkan tahapan sebelumnya, file public key bernama `fighter.pub`.

## 5. Tambahkan GitHub sebagai Known Host di Server

Selanjutnya tambahkan github.com sebagai known hosts access di server dengan  menjalankan perintah berikut:

```sh
ssh-keyscan github.com >> ~/.ssh/known_hosts
```

## 6. Jalankan Actions

Setelah itu, simpan file konfigurasi github actions dan release project. setelah itu proses running workflow akan berjalan dan proses deployment ke server akan dilakukan. jika berhasil, maka project akan berhasil disebarkan ke server.