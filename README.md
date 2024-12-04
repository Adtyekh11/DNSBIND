# Instalasi dan konfigurasi DNS BIND Ubuntu server
### Kelompok A2 (DNS BIND)
- Muhamad Rizky
- Aditya Eka Heriyawan
- Rafi Asmaul Rizal
## 1. Instalasi

Persiapan
Pastikan sistem Anda sudah diperbarui:
> sudo apt update && sudo apt upgrade -y

  

## 2. Install BIND9
Instal paket BIND9:
> sudo apt install bind9 bind9utils bind9-doc -y

## 3. Konfigurasi DNS Server
### a. Konfigurasi File Utama
File utama konfigurasi BIND terletak di /etc/bind/named.conf. Namun, Anda biasanya hanya perlu mengedit file berikut:
- /etc/bind/named.conf.options – Untuk pengaturan umum.
- /etc/bind/named.conf.local – Untuk zona khusus Anda.
##### Edit /etc/bind/named.conf.options:
Buka file:
> sudo nano /etc/bind/named.conf.options

### Tambahkan atau sesuaikan konfigurasi berikut:
    options {
    directory "/var/cache/bind";
    recursion yes;       // Aktifkan recursive query jika menjadi DNS Resolver.
    allow-query { any; };// Izinkan query dari semua IP.
    forwarders {
        8.8.8.8;         // Google DNS
        8.8.4.4;
    };

    dnssec-validation auto; // Gunakan DNSSEC.

    };


Simpan dan keluar (Ctrl+O, Enter, Ctrl+X).

### b. Tambahkan Zona DNS
Edit file /etc/bind/named.conf.local untuk menambahkan zona domain Anda:

> sudo nano /etc/bind/named.conf.local
 Contoh konfigurasi untuk domain lokal example.com:
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
};
### c. Buat File Zona DNS
Salin template dari db.local:

> sudo cp /etc/bind/db.local /etc/bind/db.example.com
> sudo cp /etc/bind/db.127 /etc/bind/db.192.168.1

### Edit file /etc/bind/db.example.com untuk menyesuaikan dengan domain Anda:

> sudo nano /etc/bind/db.example.com
Isi dengan contoh berikut:

    $TTL    604800
    @       IN      SOA     ns1.example.com. admin.example.com. (
                        1         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

    ; Nameservers
    @       IN      NS      ns1.example.com.

    ; A Records
    ns1     IN      A       192.168.1.10
    www     IN      A       192.168.1.10



Edit file /etc/bind/db.192.168.1 untuk zona reverse:

> sudo nano /etc/bind/db.192.168.1
Isi dengan contoh berikut:

   
    @       IN      SOA     ns1.example.com. admin.example.com. (
                        1         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

    ; Nameservers
    @       IN      NS      ns1.example.com.

    ; PTR Records
    10      IN      PTR     ns1.example.com.



## 4. Periksa Konfigurasi
Gunakan perintah berikut untuk memeriksa konfigurasi:

> sudo named-checkconf
> sudo named-checkzone example.com /etc/bind/db.example.com
> sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1
Jika tidak ada error, lanjutkan.

## 5. Restart dan Aktifkan BIND
Restart layanan BIND:
> sudo systemctl restart bind9

Pastikan BIND aktif saat boot:
> sudo systemctl enable bind9

## 6. Uji DNS
### a. Uji Query
Gunakan perintah dig untuk menguji:

> dig @192.168.1.10 example.com
> dig @192.168.1.10 www.example.com

### b. Uji Reverse DNS

> dig -x 192.168.1.10 @192.168.1.10
Jika hasilnya benar, maka konfigurasi Anda berhasil.

## 7. Firewall (Opsional)
Pastikan port 53 (DNS) diizinkan:
> sudo ufw allow 53
> sudo ufw reload
DNS BIND sekarang seharusnya sudah berjalan dengan baik! Jika ada masalah, beri tahu saya untuk bantuan lebih lanjut.
