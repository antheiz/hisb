# Onlinekan Localhost

Kalo mau localhost diakses oleh komputer lain dengan aman dan cepat bisa pake
[cloudflared tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/).
Cloudflare Tunnel atau dikenal sebagai `cloudflared tunnel` adalah sebuah layanan dari Cloudflare yang memungkinkan
untuk berbagi sumber daya di jaringan internal, seperti server web lokal, ke internet.

**Cara Pake Cloudflare Tunnel**

1. Buat akun Cloudflare

    Sebelum pake layanan ini, harus punya akun yang terdaftar di cloudflare. Jadi buat akun dulu dengan
    klik [disini](https://dash.cloudflare.com/sign-up)

2. Instalasi `cloudflared`

    Unduh dan install software `cloudflared` dari situs web Cloudflare [disini](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/). Ini tersedia untuk berbagai sistem operasi.
    
3. Otentikasi `cloudflared`

    Selanjutnya perlu login ke cloudflare dengan menjalankan perintah `cloudflared tunnel login` di Terminal/Command Prompt. Ini berfungsi untuk menghubungkan software `cloudflared` yang telah di-install dengan akun Cloudflare.
    
4. Jalankan Tunnel

    Yep! pada tahap ini saatnya jalankan tunnel untuk mengonlinekan sumberdaya di localhost.
    Jalankan perintah `cloudflared tunnel --url localhost:8000` di Terminal dan Ini akan memulai tunnel dan mengarahkan lalu lintas dari domainnya cloudflare ke localhost:8000.
    
    >  port `:8000` bisa diganti dengan port lain yang jalankan di localhost
    

Selanjutnya silahkan pelajari lebih lengkap terkait berbagai perintah dan konfigurasi `cloudflared tunnel` di dokumentasinya
yang bisa diakses melalui [link ini](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/)
