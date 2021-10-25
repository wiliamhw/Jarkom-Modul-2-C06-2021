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
3. Pada **EniesLobby** dan **Water7**, install dan aktifkan **bind9** dengan perintah:
    ```
    apt-get install bind9 -y
    service bind9 start
    ```
4. Pada **Skypiea**, install **apache2** dan **php7.0** serta jalankan **apache2** dengan perintah berikut:
    ```
    apt-get install apache2
    service apache2 start
    apt-get install php
    apt-get install libapache2-mod-php7.0
    ```
