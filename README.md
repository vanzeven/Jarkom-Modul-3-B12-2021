# Jarkom-Modul-3-B12-2021

Anggota: Faisal Reza M (05111940000009)

Spesifikasi: harus ada screenshot, cara pengerjaan, kendala

### Network Configuration tiap node
**Loguetown, Alabasta, dan TottoLand**
```
auto eth0
iface eth0 inet dhcp
```

**Skypie**
```
auto eth0
iface eth0 inet dhcp
hwaddress ether 1e:59:f1:4f:80:46
```
Sama seperti client lainnya, hanya saja kita men-define hwaddressnya agar tidak berubah2 tiap restart

**Foosha**
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.13.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.13.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.13.3.1
	netmask 255.255.255.0
```

**EniesLobby**
```
auto eth0
iface eth0 inet static
	address 10.13.2.2
	netmask 255.255.255.0
	gateway 10.13.2.1
```

**Water7**
```
auto eth0
iface eth0 inet static
	address 10.13.2.3
	netmask 255.255.255.0
	gateway 10.13.2.1
```

**Jipangu**
```
auto eth0
iface eth0 inet static
	address 10.13.2.4
	netmask 255.255.255.0
	gateway 10.13.2.1
```

### Nomor 1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

Pertama, buat topologi seperti di bawah (maaf jika bentuknya tidak persis seperti yang disuruh, karena saya harus memastikan agar semua node bisa cukup dalam layar meskipun window restore-down dan tidak perlu scroll), dan pastikan bahwa semua yang ada di https://github.com/arsitektur-jaringan-komputer/Modul-Jarkom/blob/master/Modul-3/prerequisite.md#inget-ini-yaa- sudah dilakukan

![image](https://user-images.githubusercontent.com/11045113/141646743-808ef280-002e-456d-b0e2-70198812fd2b.png)

isi script.sh di Jipangu
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt update
apt install isc-dhcp-server -y
dhcpd --version
bash copy.sh
```
install isc-dhcp-server, lalu panggil copy.sh

isi copy.sh
```
cp isc-dhcp-server /etc/default/
cp dhcpd.conf /etc/dhcp/
service isc-dhcp-server restart
```

isi isc-dhcp-server
```
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES="eth0"
```

isi dhcpd.conf
```
subnet 10.13.2.0 netmask 255.255.255.0 {
}

subnet 10.13.1.0 netmask 255.255.255.0 {
  range 10.13.1.20 10.13.1.99;
  range 10.13.1.150 10.13.1.169;
  option routers 10.13.1.1;
  option broadcast-address 10.13.1.255;
  option domain-name-servers 10.13.2.4, 10.13.2.2;
  default-lease-time 360;
  max-lease-time 7200;
}

subnet 10.13.3.0 netmask 255.255.255.0 {
  range 10.13.3.30 10.13.3.50;
  option routers 10.13.3.1;
  option broadcast-address 10.13.3.255;
  option domain-name-servers 10.13.2.4, 10.13.2.2;
  default-lease-time 720;
  max-lease-time 7200;
}

host Skypie {
    hardware ethernet 1e:59:f1:4f:80:46;
    fixed-address 10.13.3.69;
}
```

- subnet 10.13.2.0: karena DHCP server kita berada di bawah switch 2
- untuk detail konfigurasi lain akan dijelaskan di nomor yang sesuai
- untuk detail node Enies dan Water7 akan dijelaskan di nomor yang sesuai

### Nomor 2
Foosha sebagai DHCP Relay

isi script.sh di Foosha
```
apt update
apt install isc-dhcp-relay -y
echo 'net.ipv4.ip_forward=1'>/etc/sysctl.conf
```
Saat kita awal2 menginstall isc-dhsc-relay, installer akan menanyakan konfigurasi yang akan kita gunakan, kita harus menginputkan ip DHCP Server (Jipangu): 10.13.2.4

Lalu, installer akan menanyakan interface mana yang ingin kita layani, kita ketikkan: `eth1 eth2 eth3`, karena client dan dhcp server kita ada di interface tersebut

Untuk pertanyaan terakhir, tekan enter saja karena kita tidak membutuhkan parameter khusus

Lalu baris terakhir akan menyalakan kemampuan ip_forward di sysctl.conf

### Nomor 3
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

kita gunakan network config berikut pada client agar mereka dapat ip dari dhcp server
```
auto eth0
iface eth0 inet dhcp
```
lalu kita definisikan 2 buah range pada subnet 10.13.1.0
```
subnet 10.13.1.0 netmask 255.255.255.0 {
  range 10.13.1.20 10.13.1.99;
  range 10.13.1.150 10.13.1.169;
..}
```
![image](https://user-images.githubusercontent.com/11045113/141646838-a18970e3-03e8-498f-b1c4-7c33a37d76e8.png)

### Nomor 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

hal yang sama juga kita lakukan untuk subnet 10.13.3.0
```
subnet 10.13.3.0 netmask 255.255.255.0 {
  range 10.13.3.30 10.13.3.50;
..}
```

### Nomor 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

kita definisikan option dns pada subnet 1.0 dan 3.0
```
subnet .. {
	..
	option domain-name-servers 10.13.2.2;
	..
}
```

pastikan juga bahwa dns forwarder sudah di-set di Enies (samakan dengan foosha), di /etc/bind/named.conf.options
```
options {
        directory "/var/cache/bind";
        forwarders {
                192.168.122.1;
        };
...}
```

### Nomor 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit

- 6 menit = 360 detik
- 12 menit = 720 detik
- 120 menit = 7200 detik

kita letakkan angka tersebut pada config di jipangu
```
subnet 10.13.1.0 netmask 255.255.255.0 {
  ..
  default-lease-time 360;
  max-lease-time 7200;
}

subnet 10.13.3.0 netmask 255.255.255.0 {
  ..
  default-lease-time 720;
  max-lease-time 7200;
}
```

### Nomor 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

kita buat block host untuk skypie di dhcpd.conf (samakan hweth dengan yang telah kita set sebelumnya di network config)
```
host Skypie {
    hardware ethernet 1e:59:f1:4f:80:46;
    fixed-address 10.13.3.69;
}
```
![image](https://user-images.githubusercontent.com/11045113/141646902-1dc33896-bfeb-4e30-a638-b217930fb53c.png)

### Nomor 8-13
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000 (8). Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy (9). Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00) (10).

Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie (11).

Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps (12). Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya (13).

EniesLobby: membuat jualbelikapal.b12.com script.sh
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt update
apt install bind9 -y
mkdir /etc/bind/kaizoku
bash copy.sh
```

isi copy.sh
```
cp named.conf.local /etc/bind
cp named.conf.options /etc/bind
cp franky.b12.com /etc/bind/kaizoku
cp jualbelikapal.b12.com /etc/bind/kaizoku
service bind9 restart
```

isi jualbelikapal.b12.com
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     jualbelikapal.b12.com. root.jualbelikapal.b12.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      jualbelikapal.b12.com.
@       IN      A       10.13.2.3       ;ip water7
www     IN      CNAME   jualbelikapal.b12.com.
```

Water7: isi script.sh
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt update
apt install apache2-utils squid -y
bash copy.sh
```
squid adalah proxy server dan apache2-utils digunakan untuk htpassword

isi copy.sh
```
cp squid.conf /etc/squid/
cp acl.conf /etc/squid/
cp acl-bandwidth.conf /etc/squid/
htpasswd -cmb /etc/squid/passwd luffybelikapalb12 luffy_b12
htpasswd -mb /etc/squid/passwd zorobelikapalb12 zoro_b12
htpasswd -mb /etc/squid/passwd a a
service squid restart
```
- kita gunakan htpasswd dengan param -c untuk membuat file baru
- param m untuk menggunakan md5
- param b agar kita dapat sekaligus mengetikkan username dan password

isi squid.conf
```
include /etc/squid/acl.conf
include /etc/squid/acl-bandwidth.conf

http_port 5000
visible_hostname jualbelikapal.b12.com
#^NO8
dns_nameservers 10.13.2.2
#agar tidak kena mercusuar

acl redir dstdomain google.com
http_access deny redir
deny_info http://super.franky.b12.com redir
#^NO11

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm Proxy
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
#^NO9
http_access allow AVAILABLE_WORKING USERS
#no10
```

isi acl.conf
```
acl AVAILABLE_WORKING time MTWH 07:00-11:00
acl AVAILABLE_WORKING time TWHF 17:00-23:59
acl AVAILABLE_WORKING time WHFA 00:00-03:00
```

isi acl-bandwidth.conf
```
acl download url_regex -i \.jpg$ \.png$

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd

acl luffy proxy_auth luffybelikapalb12
acl zoro proxy_auth zorobelikapalb12

delay_pools 2
delay_class 1 1
delay_parameters 1 1250/1250
delay_access 1 deny zoro
delay_access 1 allow download
delay_access 1 deny all

delay_class 2 1
delay_parameters 2 -1/-1
delay_access 2 allow zoro
delay_access 2 deny luffy
delay_access 2 deny all
```
untuk setup server skypie, kurang lebih sama dengan https://github.com/vanzeven/Jarkom-Modul-2-B12-2021#8-setelah-melakukan-konfigurasi-server-maka-dilakukan-konfigurasi-webserver-pertama-dengan-webserver-wwwfrankyyyycom-pertama-luffy-membutuhkan-webserver-dengan-documentroot-pada-varwwwfrankyyyycom ,hanya saja isi folder diganti, download dengan `wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip`

aktifkan proxy, lalu coba akses google menggunakan lynx di loguetown

![image](https://user-images.githubusercontent.com/11045113/141646950-265a0fca-af6d-46d6-b0de-ecc2f1b7f967.png)

akan terredirect ke super franky

![image](https://user-images.githubusercontent.com/11045113/141647088-ff2520e8-12f2-4e2c-a089-5804c836d4a0.png)

mencoba menggunakan user luffy

![image](https://user-images.githubusercontent.com/11045113/141647107-9bf6094c-9e1b-4db8-ac78-57f0d57cae58.png)

jika mendownload gambar, maka kecepatannya akan di-throttle

![image](https://user-images.githubusercontent.com/11045113/141647114-5d2af2a7-3a99-4bbc-a04a-b8cd7f0892ee.png)

namun, jika menggunakan user zoro, mendownload gambar sebesar 1.8M pun akan lancar

![image](https://user-images.githubusercontent.com/11045113/141647141-e1d06484-6494-47cf-90a8-07cad4101ce4.png)

### Kendala
- Loguetown saya tiba2 tidak bisa mengakeses ubuntu.com, sehingga saya menghabiskan waktu dan energi cukup lama untuk men-debugnya
- Saya kesusahan dalam mengerjakan no 1&2 karena tidak ada di modul (jadi tidak sempat tanya asisten) dan error log nya juga tidak ada jadi debugnya susah
- Karena nomor tersebut nomor awal, waktu dan energi saya terkuras cukup banyak untuk mengerjakan sebelum akhirnya saya memutuskan untuk mengerjakan nomor 3 terlebih dahulu
