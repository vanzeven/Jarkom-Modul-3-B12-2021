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

Pertama, buat topologi seperti di bawah (maaf jika bentuknya tidak persis seperti yang disuruh, karena saya harus memastikan agar semua node bisa cukup dalam layar meskipun window di-minimize dan tidak perlu scroll), dan pastikan bahwa semua yang ada di https://github.com/arsitektur-jaringan-komputer/Modul-Jarkom/blob/master/Modul-3/prerequisite.md sudah dilakukan

!gambar topologi dengan window minimized



### Nomor 2
Foosha sebagai DHCP Relay

isi script.sh di Foosha
```
apt update
apt install isc-dhcp-relay -y
```
Saat kita awal2 menginstall isc-dhsc-relay, installer akan menanyakan konfigurasi yang akan kita gunakan, kita harus menginputkan ip DHCP Server (Jipangu): 10.13.2.4

Untuk 2 pertanyaan lainnya, enter enter saja agar relay mendeteksi otomatis config yang dibutuhkan

### Nomor 3
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

### Nomor 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

### Nomor 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### Nomor 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit

### Nomor 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

### Nomor 8
Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

### Nomor 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy

### Nomor 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

### Nomor 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

### Nomor 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

### Nomor 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

### Kendala
- Loguetown saya tiba2 tidak bisa mengakeses ubuntu.com, sehingga saya menghabiskan waktu dan energi cukup lama untuk men-debugnya
- Saya kesusahan dalam mengerjakan no 1&2 karena tidak ada di modul (jadi tidak sempat tanya asisten) dan error log nya juga tidak ada jadi debugnya susah
- Karena nomor tersebut nomor awal, waktu dan energi saya terkuras cukup banyak untuk mengerjakan sebelum akhirnya saya memutuskan untuk mengerjakan nomor 3 terlebih dahulu