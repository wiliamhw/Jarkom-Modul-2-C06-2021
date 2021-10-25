# Jarkom-Modul-2-C06-2021

## Anggota Kelompok : 
- 05111940000004 | Nizar Mayraldo
- 05111940000087 | William Handi Wijaya
- 05111940000202 | Muhammad Naufaldillah

## Jawaban Praktikum
### No.1
EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet

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

#### Skypie GNS Interfaces (sebagai Web Server)
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
    1. install dan aktifkan **bind9** dengan perintah:
        ```
        apt-get install bind9 -y
        service bind9 start
        ```
    2. Pada `/etc/resolv.conf`, tuliskan `nameserver 192.168.122.1`
4. Pada **Skypiea**, install **apache2** dan **php7.0** serta jalankan **apache2** dengan perintah berikut:
    ```
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
Buat subdomain **super.franky.yyy.com** dengan alias **www.super.franky.yyy.com** yang diatur DNS nya di **EniesLobby** dan mengarah ke **Skypie**.

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



