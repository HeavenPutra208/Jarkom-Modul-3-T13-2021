# Jarkom-Modul-3-T13-2021

## Anggota Kelompok
| No. | Nama | NRP |
| ------ | ------ | ------ |
| 1. | Heaven Happyna Putra F. | (05311940000026) |
| 2. | Gavin Bagus Kenzie Narain | (05311940000028) |
| 3. | Tera Nurwahyu Pratama | (05311940000039) |

## Daftar Isi
* [Soal 1](#soal-1)
* [Soal 2](#soal-2)
* [Soal 3](#soal-3)
* [Soal 4](#soal-4)
* [Soal 5](#soal-5)
* [Soal 6](#soal-6)
* [Soal 7](#soal-7)
* [Soal 8](#soal-8)
* [Soal 9](#soal-9)
* [Soal 10](#soal-10)
* [Soal 11](#soal-11)
* [Soal 12](#soal-12)
* [Soal 13](#soal-13)

## Soal 1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.
### Penyelesaian
Untuk no. 1, kami membuat topologi seperti modul GNS3 hingga bisa terkoneksi dengan internet, lalu konfigurasi sebagai berikut:

<br>
<img width="500" src="https://github.com/HeavenPutra208/Jarkom-Modul-3-T13-2021/blob/main/img/1.png" />
<br>

Untuk router **Foosha** pada /etc/network/interfaces, konfigurasi nodenya seperti berikut: 

```bash
auto eth0
iface eth0 inet dhcp

#client

auto eth1
iface eth1 inet static
        address 10.48.1.1
        netmask 255.255.255.0

auto eth3
iface eth3 inet static
        address 10.48.3.1
        netmask 255.255.255.0

#server2
auto eth2
iface eth2 inet static
        address 10.48.2.1
        netmask 255.255.255.0
```

Pada Router **Foosha** ini perlu dilakukan setting dulu di bagian ```net.ipv4.ip_forward``` pada ```/etc/sysctl.conf``` lalu diaktifkan dengan command ```sysctl -p```.

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

Untuk **EnniesLobby** sebagai DNS server, dikonfigurasikan node nya sebagai berikut:

```bash
apt-get update
apt-get install nano -y
apt-get install bind9
```

Pada etc/network/interfaces, dikonfigurasikan sebagai berikut:

```bash
auto eth0
iface eth0 inet static
        address 10.48.2.2
        netmask 255.255.255.0
        gateway 10.48.2.1
```

Untuk **Water7** sebagai proxy server, dikonfigurasikan node nya sebagai berikut:

```bash
apt-get update
apt-get install nano -y
apt-get install squid
```

Pada etc/network/interfaces, dikonfigurasikan sebagai berikut:

```bash
auto eth0
iface eth0 inet static
        address 10.48.2.3
        netmask 255.255.255.0
        gateway 10.48.2.1
```

Untuk **Jipangu** sebagai DHCP Sever, diperlukan instalasi ISC-DHCP-Server terlebih dahulu dengan menggunakan ```install dhcp server apt-get install isc-dhcp-server```. Kemudian, update package lists di router Foosha dengan perintah ```apt-get update Install isc-dhcp-server``` di router Foosha ```apt-get install isc-dhcp-server```. Pastikan **isc-dhcp-server** telah terinstall dengan perintah ```dhcpd --version```.

Lalu serring interfacesnya di etc/network/interfaces dengan konfigurasi sebagai berikut:

```bash
auto eth0
iface eth0 inet static
        address 10.48.2.4
        netmask 255.255.255.0
        gateway 10.48.2.1
```

Pada client (Loguetown, Alabasta, TottoLand, Skypie) di bagian /etc/network/interfaces, dimasukkan konfigurasi sebagai berikut:

```bash
auto eth0
iface eth0 inet dhcp
```

## Soal 2
dan Foosha sebagai DHCP Relay.
### Penyelesaian
Karena foosha berfungsi sebagai DHCP Relay maka perlu dilakukan instalasi package:
```bash
apt-get install isc-dhcp-relay
echo 'SERVERS=10.48.2.4
INTERFACES="eth1 eth2 eth3"' > /etc/default/isc-dhcp-relay
```

Kemudian dilakukan Restart service isc-dhcp-server dengan perintah dengan perintah:
```bash
restart service isc-dhcp-relay restart
```

## Soal 3
Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:
1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169
### Penyelesaian
Karena client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server, juga client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169, maka dilakukan setting interfaces dan deklarasi subnet pada Jipangu (DHCP Server).

Setting INTERFACES pada /etc/default/isc-dhcp-server:

```bash
# Defaults for isc-dhcp-server initscript
# sourced by /etc/init.d/isc-dhcp-server
# installed at /etc/default/isc-dhcp-server by the maintainer scripts

#
# This is a POSIX shell fragment
#

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPD_CONF=/etc/dhcp/dhcpd.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPD_PID=/var/run/dhcpd.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES="eth0"
```

Kemudian dilakukan deklarasi sbunet dengan pada ```/etc/dhcp/dhcpd.conf``` ditambahkan:
```bash
echo "subnet 10.48.3.0 netmask 255.255.255.0 {
}

subnet 10.48.1.0 netmask 255.255.255.0 {
    **range 10.48.1.20 10.48.1.99;
    range 10.48.1.150 10.48.1.169;**
    option routers 10.48.1.1;
    option broadcast-address 10.48.1.255;
    option domain-name-servers 10.48.2.2;
    default-lease-time 360;
    max-lease-time 7200;
}
" >> /etc/dhcp/dhcpd.conf
```

## Soal 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50 
### Penyelesaian
Kurang lebih hampir sama seperti no. 3, dilakukan konfigurasi pada Jipangu (DHCP Server) bagian ```/etc/dhcp/dhcpd.conf```, dengan sebagai berikut:
```bash
echo "subnet 10.48.3.0 netmask 255.255.255.0 {
    **range 10.48.3.30 10.48.3.50;**
    option routers 10.48.3.1;
    option broadcast-address 10.48.3.255;
    option domain-name-servers 10.48.2.2;
    default-lease-time 720;
    max-lease-time 7200;
}
" >> /etc/dhcp/dhcpd.conf
```

## Soal 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.
### Penyelesaian
Dilakukan setting konfigurasi DNS server di EniesLobby bagian ```/etc/bind/named.conf.options```:
```bash
options {
        directory "/var/cache/bind";
        forwarders {
        192.168.122.1;
        };
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

Diketahui bahwa ip EniesLobby ```10.48.2.2```, maka mengubahnya di settingan seperti no 3 dan 4

<br>
<img width="500" src="https://github.com/HeavenPutra208/Jarkom-Modul-3-T13-2021/blob/main/Img/2.png" />
<br>

## Soal 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.
### Penyelesaian
Karena:

6 menit = 360 detik

12 menit = 720 detik

120 menit = 7200 detik,

Maka dilakukan pengubahan settingan seperti no 3 dan 4

**Jipangu (DHCP Sever) pada /etc/dhcp/dhcpd.conf:**

```bash
subnet 10.48.1.0 netmask 255.255.255.0 {
    range 10.48.1.20 10.48.1.99;
    range 10.48.1.150 10.48.1.169;
    option routers 10.48.1.1;
    option broadcast-address 10.48.1.255;
    option domain-name-servers 10.48.2.2;
    ***default-lease-time 360;***
    **max-lease-time 7200;**

subnet 10.48.3.0 netmask 255.255.255.0 {
    range 10.48.3.30 10.48.3.50;
    option routers 10.48.3.1;
    option broadcast-address 10.48.3.255;
    option domain-name-servers 10.48.2.2;
    **default-lease-time 720;
    max-lease-time 7200;**
```

## Soal 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69
### Penyelesaian
Pada **Skypie (Client)**, ditambahkan hardware address sebagai berikut:


Kemudian dijalankan perintah echo:
```bash
echo -e "\nhwaddress ether d2:4c:32:45:27:1f" >> /etc/network/interfaces
```

Pada **Skypie (Client)**, tambahkan line baru pada ```/etc/dhcp/dhcpd.conf```:
```bash
host Skypie {
    hardware ethernet d2:4c:32:45:27:1f;
    fixed-address 10.48.3.69;
}
```

### Screenshot hasil konfigurasi dhcp

Alabasta (switch 1)

<br>
<img width="500" src="https://github.com/HeavenPutra208/Jarkom-Modul-3-T13-2021/blob/main/Img/3.png" />
<br>

Loguetown (switch 1)

<br>
<img width="500" src="https://github.com/HeavenPutra208/Jarkom-Modul-3-T13-2021/blob/main/Img/4.png" />
<br>

Skypie (switch 3)

<br>
<img width="500" src="https://github.com/HeavenPutra208/Jarkom-Modul-3-T13-2021/blob/main/Img/5.png" />
<br>

TottoLand (switch 3)

<br>
<img width="500" src="https://github.com/HeavenPutra208/Jarkom-Modul-3-T13-2021/blob/main/Img/6.png" />
<br>

****

## Soal 8
Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.

Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000
### Penyelesaian
Dilakukan instalasi squid pada node Water7. Kemudian, pada **Loguetown (Proxy client)**, dijalankan perintah:
```bash
export http_proxy="http://10.48.2.3:5000"
lynx google.com
```

Pada ```Water7 (Proxy Server)```, dijalankan juga perintah:
```bash
apt-get update
apt-get install nano -y
apt-get install squid
```

Kemudian dilakukan konfigurasi pada ```/etc/squid/squid.conf```:
```bash
http_port 5000
dns_nameservers 10.48.2.2
visible_hostname jualbelikapal.t13.com
http_access allow all
```

## Soal 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy.
### Penyelesaian
Pada **Water7 (Proxy Server)**, dilakukan instalasi ```apache2-utils``` dengan melakukan perintah:
```bash
apt-get install apache2-utils -y
```

dan untuk autentikasi usernya:

```bash
htpasswd -m -c /etc/squid/passwd luffybelikapalt13
htpasswd -m /etc/squid/passwd zorobelikapalt13
```

## Soal 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)
### Penyelesaian
Pada **Water7 (Proxy Server)** di ```/etc/squid/acl.conf```, ditambahkan konfigurasi sebagai berikut:
```bash
acl surfing_time_1 time MTWH 07:00-11:00
acl surfing_time_2 time TWHF 17:00-24:00
acl surfing_time_3 time WHFA 00:00-03:00
```

dan pada ```/etc/squid/squid.conf```, ditambahkan konfigurasi sebagai berikut:
```bash
http_port 5000
**include /etc/squid/acl.conf**
visible_hostname jualbelikapal.t13.com
dns_nameservers 10.48.2.2
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
**http_access allow USERS AVAILABLE_WORKING
http_access deny all**
```

Kemudian di-restart dan dicoba di Loguetown dengan mencoba command ```lynx google.com```:


## Soal 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie
### Penyelesaian
Pada **EnniesLobby**, ditambahkan buat domain super franky pada /etc/bind/named.conf.local, sehingga ditambahkan konfigurasi sebagai berikut:
```bash
zone "super.franky.t13.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.t13.com";
};
```

Kemudian buat file conf, arahkan A ke ip skypie:
```bash
mkdir /etc/bind/kaizoku
cp /etc/bind/db.local /etc/bind/kaizoku/super.franky.t13.com
```

Pada /etc/bind/kaizoku/super.franky.t13.com, dikonfigurasikan sebagai berikut:
```bash
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     super.franky.t13.com. root.super.franky.t13.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.t13.com.
@       IN      A       10.48.3.69
```

Sedangkan pada **Skypie**, dibuat webserver dengan command berikut:

```bash
apt-get install apache2
apt-get install php
apt-get install libapache2-mod-php7.0
apt-get install nano -y
apt-get install wget -y
apt-get install unzip -y
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/super.franky.t13.com.conf
wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip -P /var/www/
unzip /var/www/super.franky.zip -d /var/www/
mv /var/www/super.franky /var/www/super.franky.t13.com
a2ensite super.franky.t13.com
service apache2 restart
```

Dan pada /etc/apache2/sites-available/super.franky.t13.com.conf, dilakukan konfigurasi virtualhost sebagai berikut:
```bash
<VirtualHost *:80>
        ServerName super.franky.t13.com
        ServerAlias www.super.franky.t13.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.t13.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Lalu dibuat directnya.

Pada **Water7 (Proxy Server)**, di bagian ```acl banned_sites dstdomain google.com```, dijalankan command berikut:

```bash
acl banned_sites dstdomain google.com
```

Dan pada file squid ```/etc/squid/squid.conf```, dilakukan konfigurasi sebagai berikut:
```bash
#include /etc/squid/acl-bandwidth.conf
http_port 5000
include /etc/squid/acl.conf

# tambahkan
**include /etc/squid/acl-ban.conf
deny_info http://super.franky.t13.com/ banned_sites
deny_info ERR_ACCESS_DENIED all
http_reply_access deny banned_sites**

dns_nameservers 10.48.2.2
visible_hostname jualbelikapal.t13.com
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS AVAILABLE_WORKING
http_access deny all
```

## Soal 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps.
### Penyelesaian
Pada Water7 (Proxy Server), dilakukan beberapa konfigurasi berikut:

1. Pada ```/etc/squid/acl-file-access.conf```, dilakukan konfigurasi sebagai berikut:
```bash
acl images urlpath_regex -i \.png$ \.jpg$
acl otherfiles urlpath_regex -i \.html$ \.css$ \.99689$ \.jpg234$ \.cursed$ \.listingbro$ \.woi$
```

2. Pada ```/etc/squid/acl-auth.conf```, dilakukan konfigurasi sebagai berikut:
```bash
acl luffy_auth proxy_auth luffybelikapalt13
acl zoro_auth proxy_auth zorobelikapalt13
```

3. Pada ```/etc/squid/acl-bandwidth.conf```, dilakukan konfigurasi sebagai berikut:
```bash
delay_pools 2

delay_class 1 1
delay_access 1 allow images
delay_parameters 1 10000/10000
```

4. Pada ```/etc/squid/squid.conf```, dilakukan konfigurasi sebagai berikut:
```bash
#include /etc/squid/acl-bandwidth.conf
http_port 5000
include /etc/squid/acl.conf

include /etc/squid/acl-ban.conf
deny_info http://super.franky.t13.com/ banned_sites
deny_info ERR_ACCESS_DENIED all
http_reply_access deny banned_sites

dns_nameservers 10.48.2.2
visible_hostname jualbelikapal.t13.com
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED

#tambahkan
**include /etc/squid/acl-file-access.conf
include /etc/squid/acl-auth.conf
include /etc/squid/acl-bandwidth.conf

http_access deny otherfiles luffy_auth
http_access deny images zoro_auth**

http_access allow USERS AVAILABLE_WORKING
http_access deny all
```

## Soal 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya.
### Penyelesaian
Ditambahkan lagi konfigurasi bandwidth untuk zorobelikapalt13. Pada **Water7 (Proxy Server)** bagian ```/etc/squid/acl-bandwidth.conf```, ditambahkan script sebagai berikut:
```bash
delay_pools 2

delay_class 1 1
delay_access 1 allow images
delay_parameters 1 10000/10000

#tambahkan
**delay_class 2 1
delay_access 2 allow otherfiles
delay_parameters 2 none**
```

## Kendala yang Dialami
Saat pengerjaan, kebanyakan materi soal tidak ada di modul, contohnya DHCP Relay, tidak disebutkan dalam modul satupun. Kami sempat bingung sehingga harus menggali lebih dalam lewat Google. Untuk kendala lainnya tidak ada, semua soal sudah kami kerjakan dengan benar.
