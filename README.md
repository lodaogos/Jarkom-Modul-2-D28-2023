# Jarkom-Modul-3-D28-2023

## No 1

Diminta untuk membuat topologi sesuai pembagian. Di sini kami mendapatkan topologi nomor 6.<br />
Kita bisa desain topologinya lalu edit config pada Pandudewanata sebagai berikut.
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.205.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.205.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.205.3.1
	netmask 255.255.255.0
```

Dan edit config pada switch-switch lain untuk menyesuaikan dengan config Pandudewanata.

## No 2



## No 3

## No 4

## No 5

## No 6

Diminta untuk membuat Werkudara menjadi dns slave dari Abimanyu. Kita bisa mengikuti langkah-langkah di modul untuk mengerjakan soal ini.

Pertama kita set sesuai konfigurasi dari modul zone abimanyu di Abimanyu.

```
echo 'zone "abimanyu.D28.com" {
    type master;
    notify yes;
    also-notify { 192.205.1.5; };
    allow-transfer { 192.205.1.5; };
    file "/etc/bind/abimanyu/abimanyu.D28.com";  # Update the file path as needed
};' >> /etc/bind/named.conf.local
```

Kemudian pada werkudara kita set zone dengan tipe slave dan set pada 3 webserver lain IP Werkudara di `/etc/resolv.conf`:

```
echo 'zone "abimanyu.D28.com" {
    type slave;
    masters { 192.205.1.4; };
    file "/var/lib/bind/abimanyu.D28.com";
};' >> /etc/bind/named.conf.local
```
```
echo 'nameserver 192.205.1.5' >> /etc/resolv.conf
```

Untuk cek kita bisa lakukan `service bind9 stop` pada yudhistira, kemudian ping lewat salah satu web server untuk ngecek apakah masih bisa terakses. Jika bisa berarti werkudara berhasil di set sebagai dns slave.

![no6_1](/src/No_6/1.jpg)

![no6_2](/src/No_6/2.jpg)



## No 7

Buat subdomain delegasi yaitu `baratayuda.abimanyu.d28.com`  di werkudara yang mengarah ke abimanyu. Untuk mengerjakan soal ini kita bisa mengikuti modul lagi.

Pertama kita echo ini ke dalam abimanyu.d28.com di Yudhistira (Ini dari script langsung jadi mungkin sudah ada beberapa soal yang dikerjakan sehigga command echo terlihat lebih banyak daripada biasanya)

Kemudian kita perlu jalankan 

```
echo 'options {
	directory "/var/cache/bind";
	allow-query { any; };
	auth-nxdomain no;
	listen-on-v6 { any; };
};' > /etc/bind/named.conf.options
```

Zone harus di edit menjadi seperti ini pada `/etc/bind/named.conf.local`

```
'zone "abimanyu.D28.com" {
    type master;
    file "/etc/bind/abimanyu/abimanyu.D28.com";  # Update the file path as needed
    allow-transfer { 192.205.1.5; };
};
```

Kemudian pada Werkudara kita jalankan ini 

```
echo 'options {
	directory "/var/cache/bind";
	allow-query { any; };
	auth-nxdomain no;
	listen-on-v6 { any; };
};' > /etc/bind/named.conf.options
```

Dan juga script ini

```
echo 'zone "baratayuda.abimanyu.D28.com" {
    type master;
    file "/etc/bind/delegasi/baratayuda.abimanyu.D28.com";  # Update the file path as needed
};' >> /etc/bind/named.conf.local

```

Hasil ping : 

![no7_1](/src/No_7/1.jpg)

![no7_2](/src/No_7/2.jpg)


## No 8

Untuk soal no 8 pada dasarnya sama cuman kali ini buatnya rjp.baratayuda.abimanyu.d28.com. Jadipertama kita masukin zone ke `/etc/bind/named.conf.local`

```
echo 'zone "rjp.baratayuda.abimanyu.D28.com" {
    type master;
    file "/etc/bind/delegasi/rjp.baratayuda.abimanyu.D28.com";  # Update the file path as needed
};' >> /etc/bind/named.conf.local
```

Kemudian buat directory `delegasi`

```
mkdir /etc/bind/delegasi
```

Kemudian jalankan script di bawah ini

```
echo '$TTL 604800
@ IN SOA rjp..abimanyu.D28.com. root.rjp.baratayuda.abimanyu.D28.com. (
    2022100601 ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ) ; Negative Cache TTL
;
@ IN NS rjp.baratayuda.abimanyu.D28.com.
@ IN A 192.205.3.3 ; IP Abimanyu
www IN CNAME rjp.baratayuda.abimanyu.D28.com.' > /etc/bind/delegasi/rjp.baratayuda.abimanyu.D28.com
```

Hasil ping : 

![no8](/src/No_8/1.jpg)


## No 9

Pada no 9 tinggal mengikuti modul dan lakukan hal ini pada setiap worker

```
mkdir /var/www/jarkom

echo ' <?php
echo "Halo, Kamu berada di Prabukusuma"; //diganti sesuai worker
?>' > /var/www/jarkom/index.php

echo 'server {

listen 80;

root /var/www/jarkom;

index index.php index.html index.htm;
server_name _;

location / {
    try_files $uri $uri/ /index.php?$query_string;
}

# pass PHP scripts to FastCGI server
location ~ \.php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
}

location ~ /\.ht {
    deny all;
}

error_log /var/log/nginx/jarkom_error.log;
access_log /var/log/nginx/jarkom_access.log;
}' > /etc/nginx/sites-available/jarkom

ln -s /etc/nginx/sites-available/jarkom /etc/nginx/sites-enabled

service nginx restart
```

## No 10

Pada nomor 10 kita mengikuti modul lagi, tapi pada dasarnya kita set load balancing di Arjuna

```
echo ' # Default menggunakan Round Robin
upstream myweb  {
server 192.205.3.4:8001; #IP Prabukusuma
server 192.205.3.3:8002; #IP Abimanyu
server 192.205.3.2:8003; #IP Wisanggeni
}

server {
listen 80;
server_name arjuna.D28.com;

location / {
proxy_pass http://myweb;
}
}' > /etc/nginx/sites-available/lb-jarkom

ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled
```

Hasil lynx ke arjuna.d28.com : 

![no10_1](/src/No_10/1.jpg)

![no10_2](/src/No_10/2.jpg)

![no10_3](/src/No_10/3.jpg)

Disarankan untuk setiap worker dijalankan ini di script untuk menghindari hal-hal yang tidak diinginkan :

![no10_4](/src/No_10/4.jpg)


## No 11

Untuk no 11 kita perlu install wget terlebih dahulu agar bisa download file dari gdrive dengan command :

```
wget --no-check-certificate "https://drive.google.com/u/0/uc?id=1a4V23hwK9S7hQEDEcv9FL14UkkrHc-Zc&export=download" -P ~/var/www/abimanyu.d28 -O abimanyu.zip
```

Setelah itu kita tinggal jalankan sesuai modul seperti script dibawah ini dengan sedikit modifikasi dimana kita mengunzip file yang di download tadi.

![no11_1](/src/No_11/1.jpg)

![no11_2](/src/No_11/2.jpg)

Hasil lynx ke www.abimanyu.d28.com : 

![no11_3](/src/No_11/3.jpg)


## No 12

Untuk mengubah www.abimanyu.d28.com/index.php/home menjadi www.abimanyu.d28.com/home, kita bisa mengubah .htaccess nya. SCript yang dijalankan adalah sebagai berikut.

```
a2enmod rewrite
service apache2 restart

echo'
RewriteEngine On
RewriteRule ^home$ /index.php/home [L]
' > /var/www/abimanyu.d28.com/.htaccess

```

Kemudian ubah config menjadi seperti ini 

```
<Directory /var/www/abimanyu.d28.com>
     Options +FollowSymLinks -Multiviews
     AllowOverride All
</Directory>
```

Hasil lynx www.abimanyu.d28.com/home


![no12](/src/No_12/1.jpg)


## No 13

Untuk no 13 kita perlu mengaktifkan parikesit.abimanyu.d28.com. Script yang dijalankan adalah sebagai berikut.

```
echo '
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName parikesit.abimanyu.d28.com
    DocumentRoot /var/www/parikesit.abimanyu.d28.com

    <Directory /var/www/parikesit.abimanyu.d28.com/error>
       Options +Indexes
    </Directory>
  
    <Directory /var/www/parikesit.abimanyu.d28.com/public>
       Options +Indexes
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/parikesit_error.log
    CustomLog ${APACHE_LOG_DIR}/parikesit_access.log combined
</VirtualHost>' > /etc/apache2/sites-available/parikesit.conf

mkdir /var/www/parikesit.abimanyu.d28.com

wget --no-check-certificate "https://drive.google.com/u/0/uc?id=1LdbYntiYVF_NVNgJis1GLCLPEGyIOreS&export=download" -O parikesit.zip

unzip  parikesit.zip -d /var/www

mv /var/www/parikesit.abimanyu.yyy.com/error /var/www/parikesit.abimanyu.d28.com 
mv /var/www/parikesit.abimanyu.yyy.com/public /var/www/parikesit.abimanyu.d28.com


a2ensite parikesit

```

Hasil lynx : 

![no13](/src/No_13/1.jpg)


## No 14

Kemudian pada directory /secret tidak dapat diakses (403 Forbidden).

Hal ini dapat dicapai dengan menambah konfig parikesit dengan line ini.

```
<Directory /var/www/parikesit.abimanyu.d28.com/secret>
        Options -Indexes
        AllowOverride None
        Require all denied
</Directory>
```

Kemudian kita perlu buat direktori secret dengan command `mkdir /var/www/parikesit.abimanyu.d28.com/secret`

Hasil lynx parikesit.abimanyu.d28.com/secret :

![no14_1](/src/No_14/1.jpg)

![no14_2](/src/No_14/2.jpg)

## No 15

Untuk kustomisasi yang dilakukan pada no 15 hanyalah mengubah string yang ditampilkan saja.

```
sed -i 's/ERROR gan, jangan dicoba lagi/404 Not Found/g' /var/www/parikesit.abimanyu.d28.com/error/404.html

sed -i 's/Tikus jangan masuk!/403 Forbidden/g' /var/www/parikesit.abimanyu.d28.com/error/403.html
```

Hasil perubahan :

![no15_1](/src/No_15/1.jpg)

![no15_2](/src/No_15/2.jpg)

## No 16

Agar bisa mengubah www.parikesit.abimanyu.d28.com/public/js menjadi www.parikesit.abimanyu.d28.com/js, kita perlu menambah dan mengubah file konfig sesuai yang berikut ini.

```
<Directory /var/www/www.parikesit.abimanyu.d28.com/public>
        Options +Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>

Alias “/js” “/var/www/parikesit.abimanyu.d28.com/public/js”
```

Hasil lynx : 

![no16](/src/No_16/1.jpg)


## No 17

Untuk yang ini perlu mengkonfigurasi untuk www.rjp.baratayuda.abimanyu.d28.com

Bisa ikuti script sebagai berikut ini (Tapi ingat untuk wget perlu diletakkan di bagian atas script karena jika tidak kita harus mengubah nameservernya):

```
echo '
<VirtualHost *:14000 *:14400>
    ServerAdmin webmaster@localhost
    ServerName rjp.baratayuda.abimanyu.d28.com
    ServerAlias www.rjp.baratayuda.abimanyu.d28.com
    DocumentRoot /var/www/rjp.baratayuda.abimanyu.d28.com
    ErrorLog ${APACHE_LOG_DIR}/baratayuda_error.log
    CustomLog ${APACHE_LOG_DIR}/baratayuda_access.log combined
</VirtualHost>
' > /etc/apache2/sites-available/baratayuda.conf
wget --no-check-certificate "https://drive.google.com/u/0/uc?id=1pPSP7yIR05JhSFG67RVzgkb-VcW9vQO6&export=download" -O baratayuda.zip


mkdir /var/www/rjp.baratayuda.abimanyu.d28.com

unzip -j baratayuda.zip -d /var/www/rjp.baratayuda.abimanyu.d28.com
echo '
Listen 80
Listen 14000
Listen 14400

<IfModule ssl_module>
    Listen 443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 443
</IfModule>
' > /etc/apache2/ports.conf

a2ensite baratayuda.conf
```

Hasil lynx : 

![no17](/src/No_17/1.jpg)

Karena saya sudah mengerjakan no 18, diperlukan username dan password agar bisa mengaksesnya.

## No 18

Untuk no ini kita perlu menambah username Wayang dengan password baratayudad28.

Kita bisa menjalankan command sebagai berikut `htpasswd -b -c /etc/apache2/.htpasswd Wayang baratayudad28` untuk set username dan passwordnya.

Kemudian di file config kita perlu set sebagai berikut.

```
<Directory /var/www/rjp.baratayuda.abimanyu.d28.com>
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```

Hasil lynx : 

![no18](/src/No_18/1.jpg)


## No 19

Untuk no 19 kita cukup tambahkan ini ke konfig agar setiap kali IP Abimanyu dicari, akan diarahkan ke www.abimanyu.d28.com

```
echo '
<VirtualHost *:80>
    ServerName 192.205.3.3
    Redirect permanent / http://www.abimanyu.d28.com/
</VirtualHost>' >> /etc/apache2/sites-available/000-default.conf
```

Hasil lynx 192.205.3.3 : 

![no19](/src/No_19/1.jpg)


## No 20

Untuk no 20 masih belum berhasil
