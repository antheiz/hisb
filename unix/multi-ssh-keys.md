# Pengaturan SSH Multi-Akun Github

Untuk melakukan pengaturan SSH untuk multi-akun Github, ikuti panduan berikut:

## 1. Buat Kunci SSH baru

- Buka terminal dan jalankan perintah berikut untuk membuat kunci SSH baru. Gantikan `email@example.com` dengan email yang terkait dengan setiap akun Github anda.

    ```sh
    ssh-keygen -t rsa -b 4096 -C "email@example.com"
    ```

    > Angka 4096 dalam perintah diatas menunjukkan panjang bit dari kunci yang dihasilkan.

- Saat diminta untuk memasukkan nama file di mana kunci akan disimpan, masukkan path dan nama file yang unik, misal:

    ```sh
    /home/username/.ssh/id_rsa_username
    ```

## 2. Tambahkan Kunci SSH ke ssh-agent

> `ssh-agent` adalah program yang digunakan untuk menyimpan kunci privat SSH secara pribadi di memori.  Ini sangat berguna dalam memudahkan manajemen kunci SSH.

- Pastikan ssh-agent sedang berjalan dengan perintah:

    ```sh
    eval "$(ssh-agent -s)"
    ```

- Tambahkan kunci SSH ke ssh-agent:

    ```sh
    ssh-add ~/.ssh/id_rsa_username
    ```

## 3. Tambahkan Kunci SSH ke Akun Github

- Salin isi kunci SSH publik yang baru saja Anda buat.

    ```sh
    cat ~/.ssh/id_rsa_username.pub | pbcopy
    ```

- Buka pengaturan SSH dan GPG di Github, klik "New SSH key", beri judul, dan tempelkan kunci publik yang telah disalin ke dalam kotak teks.

## 4. Konfigurasi File ~/.ssh/config

- Buat atau modifikasi file konfigurasi SSH untuk menambahkan host baru untuk setiap akun Github.

    ```sh
    nano ~/.ssh/config
    ```

- Tambahkan konfigurasi berikut:

    ```sh
    # Akun Github pertama
    Host first.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_username

    # Akun Github kedua
    Host second.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_username2
    ```

    > Ganti `id_rsa_username` dengan nama file kunci pribadi yang benar untuk kedua kunci ssh.


## 5. Gunakan Host Baru dalam Repository

Ketika ingin bekerja dengan repository yang terkait dengan setiap akun, clone repository berbasis SSH. Lalu ganti url remote menggunakan host baru:

```sh
git clone git@first.github.com:username/repo.git
```

> Ganti `username` dengan nama pengguna Github Anda dan `repo` dengan nama repository.