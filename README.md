# Jarkom-Modul-3-B06-2021

## Anggota Kelompok : 
- 05111940000048 | Bagaskoro Kuncoro Ardi 
- 05111940000096 | Stefanus Albert Kosim 

#

### Soal 8

Node Loguetown digunakan sebagai client proxy yang diakses dengan nama ***jualbelikapal.B06.com*** dan port yang digunakan adalah 5000

### Jawab

Agar Node Loguetown dapat mengakses proxy dengan nama ***jualbelikapal.B06.com***, maka terlebih dahulu dibuat domain tersebut di *Node **EniesLobby*** yang mengarah ke ***IP Proxy Server*** yaitu ***Water7***. Domain dibuat mirip seperti modul 2 yaitu dengan membuat direktori **kaizoku** di `/etc/bind/`, lalu menyalin file `/etc/bind/db.local` ke `/etc/bind/kaizoku/jualbelikapal.B06.com`, lalu isi file tersebut diubah.

```
mkdir /etc/bind/kaizoku

cp /etc/bind/db.local /etc/bind/kaizoku/jualbelikapal.B06.com

echo ";
; BIND data file for local loopback interface
\$TTL    604800
@       IN      SOA     jualbelikapal.B06.com. root.jualbelikapal.B06.com. (
                        2021110701      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      jualbelikapal.B06.com.
@               IN      A       10.10.2.3
" > /etc/bind/kaizoku/jualbelikapal.B06.com
```

![8_Konfig DNS](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/8-1.png)

Setelah diubah, zone pada `/etc/bind/named.conf.local` juga ditambahi domain yang telah dibuat tadi 

```
zone "jualbelikapal.B06.com" {
		type master;
		notify yes;
		file "/etc/bind/kaizoku/jualbelikapal.B06.com";
};
```

![8_Konfig zone](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/8-2.png)

Untuk mengetahui apakah domain telah berfungsi dengan baik, dilakukan ping dari client

![8_Ping](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/8-3.png)

Selanjutnya adalah mengatur konfigurasi proxy di **Water7**. Sebelum melakukan konfigurasi, lakukan backup pada file `/etc/squid/squid.conf` yang lama, lalu buat file ***squid.conf*** yang baru

```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
touch /etc/squid/squid.conf
```

Agar dapat diakses melalui port 5000 dan visible_hostname adalah ***jualbelikapal.B06.com***, maka tambahi konfigurasi pada file ***squid.conf*** yang baru saja dibuat.

```
http_port 5000
visible_hostname jualbelikapal.B06.com
```

![8_Squid.conf](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/8-4.png)

Kemudian pada node ***Loguetown***, proxy diaktifkan dengan menggunakan command

```
export http_proxy="http://jualbelikapal.B06.com:5000"
```

![8_http_proxy](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/8-5.png)

#

### Soal 9

Digunakan 2 autentikasi user proxy dengan enkripsi MD5 yaitu ***luffybelikapalB06*** dengan password ***luffy_B06*** dan ***zorobelikapalB06*** dengan password ***zoro_B06***.

### Jawab

Untuk membuat autentikasi dengan enkripsi MD5, digunakan ***htpasswd*** yang merupakan *utility* dari apache2 sehingga terlebih dahulu perlu menginstal `apache2-utils`. 

```
apt-get update
apt-get install apache2-utils -y
```

Selanjutnya adalah membuat file ***htpasswd***, lalu mengisi file tersebut dengan username dan password pada soal menggunakan ***htpasswd***

```
touch /etc/squid/passwd
htpasswd -nb luffybelikapalB06 luffy_B06 >> /etc/squid/passwd
htpasswd -nb zorrobelikapalB06 zorro_B06 >> /etc/squid/passwd
```

Kemudian pada file `/etc/squid/squid.conf` ditambahkan konfigurasi berikut

```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```

![9_squid](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/9-1.png)

Untuk membuktikan bahwa autentikasi telah berhasil, dicoba untuk mengakses web random

![9_hasil](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/9-2.gif)

#

### Soal 10

Transaksi hanya bisa dilakukan pada hari **Senin-Kamis pukul 07:00-11:00**, pada hari **Selasa-Jumat pukul 17:00-23:59** dan dilanjut hari **Rabu-Sabtu pukul 00:00-03:00**

### Jawab

Agar transaksi hanya dapat diakses pada waktu-waktu tertentu, maka dibuat konfigurasi dengan ACL dengan terlebih dahulu membuat file `/etc/squid/acl.conf` lalu diisi seperti konfigurasi dibawah

```
touch /etc/squid/acl.conf

acl WORKING1 time MTWH 07:00-11:00
acl WORKING2 time TWHF 17:00-23:59
acl WORKING3 time WHFA 00:00-03:00
```

![10_acl](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/10-1.png)

Selanjutnya adalah mengimport file acl tadi ke `/etc/squid/squid.conf` lalu membuat hak akses hanya diberikan pada **waktu sesuai konfigurasi ACL** **dan** **user yang telah dibuat** dengan menambah konfigurasi berikut ke ***squid.conf***

```
include /etc/squid/acl.conf

http_access allow WORKING1 USERS
http_access allow WORKING2 USERS
http_access allow WORKING3 USERS
http_access deny all
```

![10_squid](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/10-2.png)

![10_hasil](https://github.com/yanzkosim/Jarkom-Modul-3-B06-2021/blob/main/Screenshot/10-3.gif)

#



Keterangan :

http_access allow WORKING1 USERS berarti akses diberikan jika yang mengakses web tersebut adalah **USERS** pada autentikasi nomor 9 **dan** diakses pada waktu sesuai dengan konfigurasi **WORKING1** di **acl.conf**




## Catatan

Selalu jalankan command berikut mulai dari nomor 8 hingga nomor terakhir apabila telah dilakukan konfigurasi

```
service squid restart
```