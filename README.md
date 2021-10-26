# Jarkom-Modul-2-C06-2021

## Anggota Kelompok : 
- 05111940000004 | Nizar Mayraldo
- 05111940000087 | William Handi Wijaya
- 05111940000202 | Muhammad Naufaldillah

## Jawaban Praktikum
### No.1
EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypiea akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet

### Jawaban
#### Topologi GNS
![Topologi GNS](https://user-images.githubusercontent.com/52129348/138724448-efea0761-f7ee-4cbb-a147-abc2961b71d1.png)

#### Foosha GNS Interfaces (sebagai Router)
```
# DHCP config for eth0
auto eth0
iface eth0 inet dhcp

# Static config for eth1
auto eth1
iface eth1 inet static
	address 10.17.1.1
	netmask 255.255.255.0

# Static config for eth2
auto eth2
iface eth2 inet static
	address 10.17.2.1
	netmask 255.255.255.0
```

#### Loguetown GNS Interfaces (sebagai klien)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.1.2
	netmask 255.255.255.0
	gateway 10.17.1.1
```

#### Alabasta GNS Interfaces (sebagai klien)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.1.3
	netmask 255.255.255.0
	gateway 10.17.1.
```

#### EniesLobby GNS Interfaces (sebagai DNS Master Server)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.2.2
	netmask 255.255.255.0
	gateway 10.17.2.1
```

#### Water7 GNS Interfaces (sebagai DNS Slave Server)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.2.3
	netmask 255.255.255.0
	gateway 10.17.2.1
```

#### Skypiea GNS Interfaces (sebagai Web Server)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.2.4
	netmask 255.255.255.0
	gateway 10.17.2.1
```

#### Konfigurasi node agar dapat mengakses internet dan DNS master.
1. Pada **Foosha**, jalankan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.17.0.0/16`.
2. Pada node **LogueTown** dan **Alabasta**,
    1. Pada `/etc/resolv.conf`, tuliskan:
      ```
      nameserver 10.17.2.2
      nameserver 192.168.122.1
      ```
    2. Install **lynx** dengan perintah `apt get install lynx`.
    3. Install **dnsutils** dengan perintah `apt-get install dnsutils`.
3. Pada **EniesLobby** dan **Water7**, 
    1. Pada `/etc/resolv.conf`, tuliskan `nameserver 192.168.122.1`
    2. Install dan aktifkan **bind9** dengan perintah:
        ```
        apt-get install bind9 -y
        service bind9 start
        ```
4. Pada **Skypiea**:
	1. Pada `/etc/resolv.conf`, tuliskan: `nameserver 192.168.122.1`.
	2. Install **apache2** dan **php7.0** serta jalankan **apache2** dengan perintah berikut:
		```
        apt-get install wget
        apt-get install unzip
		apt-get install apache2
		service apache2 start
		apt-get install php
		apt-get install libapache2-mod-php7.0
		```
Setelah itu, konfigurasi ini akan disimpan pada `/root/.bashrc`.


### No.2
Pada node **EniesLobby** dan di dalam folder **kaizoku**, buat wesite utama dengan URL **franky.C06.com** dan alias **www.franky.C06.com**!

### Jawaban
1. Tuliskan kode berikut pada `/root/2.sh` di node **EniesLobby**:
```
#!/bin/bash

# Mendaftarkan domain name
echo "
zone \"franky.C06.com\" {
        type master;
        file \"/etc/bind/kaizoku/franky.C06.com\";
};
" > /etc/bind/named.conf.local

# Buat folder kaizoku dan file yang menyimpan konfigurasi DNS
[ ! -d "/etc/bind/kaizoku" ] && `mkdir /etc/bind/kaizoku`
`cp /etc/bind/db.local /etc/bind/kaizoku/franky.C06.com`

# Buat konfigurasi DNS
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     franky.C06.com. root.franky.C06.com. (
                     2021100401         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.C06.com.
@       IN      A       10.17.2.4
www     IN      CNAME   franky.C06.com.
" > /etc/bind/kaizoku/franky.C06.com

# Restart bind9
`service bind9 restart`
```
2. Tuliskan `bash /root/2.sh` pada `/root/.bashrc` di node **EniesLobby**.

#### Hasil
![Hasil no.2](https://user-images.githubusercontent.com/52129348/138732437-fca89a63-2e96-4478-a4ec-43388a433967.png)


### No.3
Buat subdomain **super.franky.yyy.com** dengan alias **www.super.franky.yyy.com** yang diatur DNS nya di **EniesLobby** dan mengarah ke **Skypiea**.

### Jawaban
1. Tuliskan kode berikut pada `/root/3.sh` di node **EniesLobby**:
	```
    #!/bin/bash

    # Tambahkan subdomain beserta aliasnya
    echo "
    super           IN      A       10.17.2.4
    www.super       IN      CNAME   super.franky.C06.com
    " >> /etc/bind/kaizoku/franky.C06.com

    # Restart bind9
    `service bind9 restart`
    ```
2. Tuliskan `bash /root/3.sh` pada `/root/.bashrc` di node **EniesLobby**. 

#### Hasil
![Hasil no.3](https://user-images.githubusercontent.com/52129348/138736381-6e9686a2-ea74-45e3-b579-7fbeaf88d308.png)


### No.4
Buat reverse domain untuk domain utama.

### Jawaban
1. Tuliskan kode berikut pada `/root/4.sh` di node **EniesLobby**:
	```
    #!/bin/bash

    # Daftarkan domain name baru
    echo "
    zone \"2.17.10.in-addr.arpa\" {
            type master;
            file \"/etc/bind/kaizoku/2.17.10.in-addr.arpa\";
    };
    " >> /etc/bind/named.conf.local

    # Buat folder kaizoku dan file yang menyimpan konfigurasi DNS
    [ ! -d "/etc/bind/kaizoku" ] && `mkdir /etc/bind/kaizoku`
    `cp /etc/bind/db.local /etc/bind/kaizoku/2.17.10.in-addr.arpa`

    # Buat konfigurasi DNS
    echo "
    ;
    ; BIND data file for local loopback interface
    ;
    \$TTL    604800
    @       IN      SOA     franky.C06.com. root.franky.C06.com. (
                         2021100401         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    2.17.10.in-addr.arpa.      IN      NS      franky.C06.com.
    4                          IN      PTR     franky.C06.com. ;BYTE kE-4 IP Skypiea
    " > /etc/bind/kaizoku/2.17.10.in-addr.arpa

    # Restart bind9
    `service bind9 restart`
    ```
2. Tuliskan `bash /root/4.sh` pada `/root/.bashrc` di node **EniesLobby**. 

#### Hasil
![Hasil no.4](https://user-images.githubusercontent.com/52129348/138739394-a5f12cb8-2a17-4e25-95f2-9c94039eb5bd.png)


### No.5
Supaya tetap bisa menghubungi Franky jika server **EniesLobby** rusak, maka buat **Water7** sebagai DNS Slave untuk domain utama.

### Jawaban
1. Edit domain name (zone) `franky.C06.com` pada `/etc/bind/named.conf.local` di node **EniesLobby** menjadi:
    ```
    zone "franky.C06.com" {
        type master;
        notify yes;
        also-notify { 10.17.2.3; };
        allow-transfer { 10.17.2.3; };
        file "/etc/bind/kaizoku/franky.C06.com";
    };
    ```
2. Tuliskan kode untuk menambah domain name di bawah ini pada `/etc/bind/named.conf.local` di node **Water7**:
    ```
    zone "franky.C06.com" {
        type slave;
        masters { 10.17.2.2; };
        file "/var/lib/bind/franky.C06.com";
    };
    ```

#### Hasil
![Screenshot from 2021-10-26 00-42-59](https://user-images.githubusercontent.com/52129348/138744230-a22ad90a-b5ba-4cac-8c3f-2a3f41b89e3b.png)


### No.6
Setelah itu terdapat subdomain `mecha.franky.yyy.com` dengan alias `www.mecha.franky.yyy.com` yang didelegasikan dari **EniesLobby** ke **Water7** dengan IP menuju ke **Skypiea** dalam folder `sunnygo`

### Jawaban
#### Konfigurasi EniesLobby
1. Tuliskan kode berikut pada `/root/6.sh`.
    ```
    #!/bin/bash

    # Tambahkan nameserver menuju Water7
    echo "
    ns2     IN      A       10.17.2.3 ;IP Water7
    mecha   IN      NS      ns2
    " >> /etc/bind/kaizoku/franky.C06.com
    
    # Restart bind9
    `service bind9 restart`
    ```
2. Tuliskan `bash /root/6.sh` pada `/root/.bashrc.
3. Edit file `/etc/bind/named.conf.options` dengan mengkomen `dnssec-validation auto;` dan menambahkan `allow-query{any;};`.
4. Pastikan `allow-transfer { 10.17.2.3; };` sudah tertulis pada zone `franky.C06.com` di file `/etc/bind/named.conf.local`. (merupakan jawaban no.5)

#### Konfigurasi Water7
1. Tuliskan kode berikut pada `/root/6.sh`.
    ```
    #!/bin/bash

    # Daftarkan domain name baru
    echo "
    zone \"mecha.franky.C06.com\" {
            type master;
            file \"/etc/bind/sunnygo/mecha.franky.C06.com\";
    };
    " >> /etc/bind/named.conf.local

    # Buat folder sunnygo dan file yang menyimpan konfigurasi DNS
    [ ! -d "/etc/bind/sunnygo" ] && `mkdir /etc/bind/sunnygo`
    `cp /etc/bind/db.local /etc/bind/sunnygo/mecha.franky.C06.com`

    # Buat konfigurasi DNS
    echo "
    ;
    ; BIND data file for local loopback interface
    ;
    \$TTL    604800
    @       IN      SOA     mecha.franky.C06.com. root.mecha.franky.C06.com. (
                         2021100401         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      mecha.franky.C06.com.
    @       IN      A       10.17.2.4       ; IP Skypiea
    www     IN      CNAME   mecha.franky.C06.com.
    " > /etc/bind/sunnygo/mecha.franky.C06.com

    # Restart bind9
    `service bind9 restart`
    ```
3. Tuliskan `bash /root/6.sh` pada `/root/.bashrc.
4. Edit file `/etc/bind/named.conf.options` dengan mengkomen `dnssec-validation auto;` dan menambahkan `allow-query{any;};`.

#### Hasil
![Hasil no.6](https://user-images.githubusercontent.com/52129348/138835481-251f39a7-5ff0-479f-83f5-ce69e79f9418.png)

### No.7
Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui **Water7** dengan nama `general.mecha.franky.C06.com` dengan alias `www.general.mecha.franky.C06.com` yang mengarah ke **Skypiea**.

#### Jawaban
Pada **Water7**:
1. Tuliskan kode berikut pada `/root/7.sh`.
    ```
    #!/bin/bash

    # Daftarkan domain name baru
    echo "
    zone \"general.mecha.franky.C06.com\" {
            type master;
            file \"/etc/bind/sunnygo/general.mecha.franky.C06.com\";
    };
    " >> /etc/bind/named.conf.local

    # Buat folder sunnygo dan file yang menyimpan konfigurasi DNS
    [ ! -d "/etc/bind/sunnygo" ] && `mkdir /etc/bind/sunnygo`
    `cp /etc/bind/db.local /etc/bind/sunnygo/general.mecha.franky.C06.com`

    # Buat konfigurasi DNS
    echo "
    ;
    ; BIND data file for local loopback interface
    ;
    \$TTL    604800
    @       IN      SOA     general.mecha.franky.C06.com. root.general.mecha.franky.C06.com. (
                         2021100401         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      general.mecha.franky.C06.com.
    @       IN      A       10.17.2.4       ; IP Skypiea
    www     IN      CNAME   general.mecha.franky.C06.com.
    " > /etc/bind/sunnygo/general.mecha.franky.C06.com

    # Restart bind9
    `service bind9 restart`
    ```
2. Tuliskan `bash /root/7.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil no.7](https://user-images.githubusercontent.com/52129348/138836907-e70d6bab-f007-43f4-8911-b2b1c0949571.png)


### No.8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver `www.franky.C06.com`. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada `/var/www/franky.C06.com`.

#### Jawaban
Pada **Skypiea**:
1. Pastikan `wget`, `unzip`, `apache2`, `php`, dan `libapache2-mod-php7.0` sudah terinstall. (instalasi  ini sudah dilakukan saat konfigurasi awal di no.1)
2. Tuliskan kode berikut pada `/root/8.sh`:
    ```
    #!/bin/bash

    # Download franky.zip dan unzip file tersebut
    [ ! -f "franky.zip" ] && `wget -O franky.zip https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/franky.zip`
    [ ! -d "franky" ] && `unzip franky.zip`

    # Buat folder yang menjadi document root dan dapatkan konten web dari franky.zip
    [ ! -d "/var/www/franky.C06.com" ] && `mkdir /var/www/franky.C06.com`
    `cp -r franky /var/www/franky.C06.com`

    # Buat file yang menyimpan konfigurasi DNS
    `cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/franky.C06.com.conf`

    # Buat konfigurasi server apache2
    echo "
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost

        DocumentRoot /var/www/franky.C06.com
        ServerName franky.C06.com
        ServerAlias www.franky.C06.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
    </VirtualHost>
    " > /etc/apache2/sites-available/franky.C06.com.conf

    # Aktifkan document root tersebut
    `a2ensite franky.C06.com`
    `service apache2 restart`
    ```
3. Tuliskan `bash /root/8.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil no.8](https://user-images.githubusercontent.com/52129348/138851685-3e532aab-a767-45d7-add5-a248d375caa7.png)


### No.9
Setelah itu, Luffy juga membutuhkan agar url `www.franky.yyy.com/index.php/home` dapat menjadi menjadi `www.franky.yyy.com/home.`

#### Jawaban
Pada **Skypiea**:
1. Tuliskan kode berikut pada `/root/9.sh`:
    ```
    #!/bin/bash

    a2enmod rewrite
    service apache2 restart

    # Tulis .htaccess
    echo "
    RewriteEngine On
    RewriteRule ^home$ index.php/home
    " > /var/www/franky.C06.com/.htaccess

    # Tambahkan konfigurasi pada franky.C06.com.conf
    echo "
    <Directory /var/www/franky.C06.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
    " >> /etc/apache2/sites-available/franky.C06.com.conf
    ```
2. Tuliskan `bash /root/9.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil no.9a](https://user-images.githubusercontent.com/52129348/138854956-81c8ad94-31ac-4b7d-9c66-eb8b930ab640.png)  
![Hasil no.9b](https://user-images.githubusercontent.com/52129348/138855014-ea5f9db5-f12e-49b2-a483-3a8db4923728.png)


### No.10
Setelah itu, pada subdomain `www.super.franky.C06.com`, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada `/var/www/super.franky.C06.com`.

#### Jawaban
Pada **Skypiea**:
1. Tuliskan kode berikut pada `/root/10.sh`:
	```
    #!/bin/bash

    # Download super.franky.zip dan unzip file tersebut
    [ ! -f "super.franky.zip" ] && `wget -O super.franky.zip https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip`
    [ ! -d "super.franky" ] && `unzip super.franky.zip`

    # Buat folder yang menjadi document root dan dapatkan konten web dari super.franky.zip
    `cp -r super.franky/. /var/www/super.franky.C06.com`

    # Buat file yang menyimpan konfigurasi DNS
    `cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/super.franky.C06.com.conf`

    # Buat konfigurasi server apache2
    echo "
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost

        DocumentRoot /var/www/super.franky.C06.com
        ServerName super.franky.C06.com
        ServerAlias www.super.franky.C06.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
    </VirtualHost>
    " > /etc/apache2/sites-available/super.franky.C06.com.conf

    # Aktifkan document root tersebut
    `a2ensite super.franky.C06.com`
    `service apache2 restart`
    ```
2. Tuliskan `bash /root/10.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil 10a](https://user-images.githubusercontent.com/52129348/138857832-c5749e44-4b2d-467c-95d9-790163a7530b.png)  
![Hasil 10b](https://user-images.githubusercontent.com/52129348/138896544-a677b116-ff5f-4edc-be95-b1361b2511f6.png)


### No.11
Akan tetapi, pada folder `/public`, Luffy ingin hanya dapat melakukan directory listing saja.

#### Jawaban
Pada **Skypiea**:
1. Tuliskan kode berikut pada `/root/11.sh`:
    ```
    #!/bin/bash

    # Tambahkan konfigurasi pada super.franky.C06.com.conf
    echo "
    <Directory /var/www/super.franky.C06.com>
        Options +Indexes
    </Directory>
    " >> /etc/apache2/sites-available/super.franky.C06.com.conf
    ```
2. Tuliskan `bash /root/11.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil 11a](https://user-images.githubusercontent.com/52129348/138859376-783acd7b-2570-4668-ae94-85634572db8b.png)
![Hasil 11b](https://user-images.githubusercontent.com/52129348/138859402-753ec0e2-b92b-4c8b-9320-9f33ee6dd686.png)


### No.12
Tidak hanya itu, Luffy juga menyiapkan error file `404.html` pada folder `/errors` untuk mengganti error kode pada apache.

#### Jawaban
Pada **Skypiea**:
1. Tuliskan kode berikut pada `/root/12.sh`:
    ```
    #!/bin/bash

    # Tambahkan konfigurasi pada super.franky.C06.com.conf
    echo "
    <Directory /var/www/super.franky.C06.com>
        ErrorDocument 404 /error/404.html
    </Directory>
    " >> /etc/apache2/sites-available/super.franky.C06.com.conf
    
    # Aktifkan konfigurasi tersebut
    `service apache2 restart`
    ```
2. Tuliskan `bash /root/12.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil 12a](https://user-images.githubusercontent.com/52129348/138882850-25c0160f-f9eb-4050-831c-24f532b2d867.png)
![Hasil 12b](https://user-images.githubusercontent.com/52129348/138882886-0c86404c-1381-4ca9-add3-694310ec9531.png)


### No.13
Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset `www.super.franky.yyy.com/public/js` menjadi `www.super.franky.yyy.com/js`.

#### Jawaban
Pada **Skypiea**:
1. Tuliskan kode berikut pada `/root/13.sh`:
    ```
    #!/bin/bash

    # Tambahkan konfigurasi pada super.franky.C06.com.conf
    echo "Alias \"/js\" \"/var/www/super.franky.C06.com/public/js\"
    " >> /etc/apache2/sites-available/super.franky.C06.com.conf

    # Aktifkan konfigurasi tersebut
    `service apache2 restart`
    
    ```
2. Tuliskan `bash /root/13.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil 13a](https://user-images.githubusercontent.com/52129348/138885101-0bfa3267-8fc8-40a3-9f37-ac7f565be6b9.png)  
![Hasil 13b](https://user-images.githubusercontent.com/52129348/138885125-6835cc22-b7c2-45de-8dd1-b085c69ffaff.png)


### No.14
Dan Luffy meminta untuk web `www.general.mecha.franky.C06.com` hanya bisa diakses dengan port 15000 dan port 15500.

#### Jawaban:
Pada **client**, pastikan IP **Water 7** berada di atas IP **EniesLobby**.
Pada **Skypiea**:
1. Tuliskan kode berikut pada `/root/14.sh`:
    ```
    #!/bin/bash

    # Download general.mecha.franky.zip dan unzip file tersebut
    [ ! -f "general.mecha.franky.zip" ] && `wget -O general.mecha.franky.zip https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/general.mecha.franky.zip`
    [ ! -d "general.mecha.franky" ] && `unzip general.mecha.franky.zip`

    # Buat folder yang menjadi document root dan dapatkan konten web dari general.mecha.franky.zip
    `cp -r /general.mecha.franky/. /var/www/general.mecha.franky.C06.com`

    # Buat file yang menyimpan konfigurasi DNS
    `cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/general.mecha.franky.C06.com.conf`

    # Buat konfigurasi server apache2
    echo "
    <VirtualHost *:15000 *:15500>
        ServerAdmin webmaster@localhost

        DocumentRoot /var/www/general.mecha.franky.C06.com
        ServerName general.mecha.franky.C06.com
        ServerAlias www.general.mecha.franky.C06.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
    </VirtualHost>
    " > /etc/apache2/sites-available/general.mecha.franky.C06.com.conf

    # Buka port 15000 dan 15500 pada Apache
    echo "
    Listen 15000
    Listen 15500
    " >> /etc/apache2/ports.conf

    # Aktifkan document root tersebut
    `a2ensite general.mecha.franky.C06.com`
    `service apache2 restart`
    
    ```
2. Tuliskan `bash /root/14.sh` pada `/root/.bashrc`.

#### Hasil
![Hasil 14a](https://user-images.githubusercontent.com/52129348/138893936-258a2656-0ece-47b2-ade2-26227aa6eda3.png)
![Hasil 14b](https://user-images.githubusercontent.com/52129348/138893997-867cf614-9692-46e1-9c5f-f8470df8f513.png)


### No.15
No.14 hanya bisa diakses dengan dengan authentikasi username luffy dan password onepiece dan file di `/var/www/general.mecha.franky.C06`.

###Kendala yang dialami selama pengerjaan
* VM Error
* Tidak bisa mengakses ip di gns
* Tidak bisa import project


