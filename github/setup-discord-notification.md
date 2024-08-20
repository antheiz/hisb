# Panduan Menambahkan Notifikasi Discord ke Workflow GitHub Actions

Panduan ini akan membantu Anda mengirim notifikasi ke channel Discord tertentu setelah status _release_ pada workflow GitHub Actions dijalankan. Notifikasi akan dikirim berdasarkan keberhasilan atau kegagalan _release_ tersebut.

## 1. Menyiapkan Discord Webhook

1. Membuat Webhook di Discord:

    - Masuk ke Discord dan pilih server yang ingin Anda gunakan.
    - Klik kanan pada channel di mana Anda ingin menerima notifikasi, lalu pilih **Edit Channel**.
    - Buka tab **Integrations** dan pilih **Webhook**.
    - Klik **New Webhook**, beri nama, pilih channel tujuan, dan simpan URL Webhook yang dihasilkan.

2. Tambahkan Webhook URL ke Repository GitHub:

    - Buka repository Anda di GitHub.
    - Pergi ke tab **Settings**, kemudian pilih **Secrets and Variables** > **Actions**.
    - Klik **New Repository Secret** dan tambahkan secret baru dengan nama `DISCORD_WEBHOOK_URL`, lalu masukkan URL webhook dari Discord.

## 2. Menambahkan Notifikasi ke File Workflow

Setelah webhook Discord siap, tambahkan notifikasi ke file workflow Anda dengan langkah berikut:

1. Buat atau modifikasi file workflow di `.github/workflows/` dengan nama, misalnya `deploy-server.yml`.
2. Masukkan konfigurasi berikut ke dalam file workflow:

```sh
# .github/workflows/deploy-server.yml

name: 🚀 Publish Website to Production Environment

on:
  release:
    types: [published]

jobs:
  web-deploy:
    name: 🚀 Deploy Website on Release
    runs-on: ubuntu-latest

    steps:
    - name: 🔧 Get short SHA
      id: get-short-sha
      run: echo "SHORT_SHA=${GITHUB_SHA::7}" >> $GITHUB_ENV

    # Notifikasi ke Discord jika sukses
    - name: 📣 Notify Discord on Success
      if: success()
      run: |
        curl -X POST -H "Content-Type: application/json" \
              -d '{"embeds": [{"author": {"name": "'${{ github.actor }}'", "icon_url": "https://github.com/${{ github.actor }}.png"}, "title": "[${{ github.repository }}:${{ github.ref_name }}] Deploy Succeeded", "url": "https://github.com/${{ github.repository }}/commit/${{ github.sha }}", "description": "`${{ env.SHORT_SHA }}` - Release: ${{ github.event.release.name }} - '${{ github.actor }}'", "color": 3066993}]}' \
              ${{ secrets.DISCORD_WEBHOOK_URL }}

    # Notifikasi ke Discord jika gagal
    - name: 📣 Notify Discord on Failure
      if: failure()
      run: |
        curl -X POST -H "Content-Type: application/json" \
              -d '{"embeds": [{"author": {"name": "'${{ github.actor }}'", "icon_url": "https://github.com/${{ github.actor }}.png"}, "title": "[${{ github.repository }}:${{ github.ref_name }}] Deploy Failed", "url": "https://github.com/${{ github.repository }}/commit/${{ github.sha }}", "description": "`${{ env.SHORT_SHA }}` - Release: ${{ github.event.release.name }} - '${{ github.actor }}'", "color": 15158332}]}' \
              ${{ secrets.DISCORD_WEBHOOK_URL }}
```

**Penjelasan:**

- Webhook Configuration:
    - **Get Short SHA**: Langkah ini mengekstrak 7 karakter pertama dari commit SHA dan menyimpannya sebagai variabel lingkungan (**SHORT_SHA**). Ini digunakan untuk menampilkan informasi versi kode pada notifikasi.

- Notifikasi Discord pada Keberhasilan _(Success)_:
    - Jika _release_ berhasil, maka workflow akan mengirimkan notifikasi ke channel Discord dengan pesan berwarna hijau (warna **#3066993**) yang menunjukkan keberhasilan.

    - Notifikasi Discord pada Kegagalan _(Failure)_:
    - Jika _release_ gagal, workflow akan mengirimkan notifikasi ke channel Discord dengan pesan berwarna merah (warna **#15158332**) yang menunjukkan kegagalan.


## 3. Menambahkan Webhook URL ke GitHub Repository

Setelah mendapatkan URL webhook dari Discord, Anda perlu menambahkannya sebagai webhook di repository GitHub untuk menerima notifikasi. Berikut adalah panduannya:

1. Buka Repository di GitHub:

    - Buka halaman repository yang akan digunakan di GitHub.

2. Masuk ke Pengaturan Webhook:

    - Klik tab Settings di bagian atas halaman repository.
    Pada panel sebelah kiri, pilih Webhooks.

3. Tambahkan Webhook Baru:
    
    - Klik tombol Add webhook.

4. Isi Form Webhook:

    - Pada form yang muncul, isi bagian berikut:
        - **Payload URL**: Paste URL webhook dari Discord yang telah Anda salin, kemudian tambahkan **/github** di akhir URL. Contoh:

            ```sh
            https://discord.com/api/webhooks/1234567890/TOKEN/github
            ```
        
        - **Content type**: Pilih **application/json** dari dropdown.
        - **Secret**: Kosongkan atau isi jika diperlukan (opsional).

5. Menentukan Event Trigger:

    - Pada bagian **Which events would you like to trigger this webhook?**, pilih **Let me select individual events**.
    - Centang opsi **Workflow** untuk memicu webhook saat ada workflow selesai dijalankan.
    - Anda juga dapat mencentang event lain jika diperlukan.

6. Simpan Webhook:

    - Klik **Add webhook** untuk menyimpan pengaturan.

## 4. Menjalankan Workflow

Setelah melakukan konfigurasi di atas, simpan file _workflow_. Proses deployment akan berjalan setiap kali _release_ baru diterbitkan, dan hasilnya akan dikirimkan sebagai notifikasi ke channel Discord sesuai dengan hasil eksekusi (_success_ atau _failure_).