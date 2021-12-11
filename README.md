# Jarkom-Modul-5-D12-2021
Laporan Resmi Praktikum Jaringan Komputer 2021 - D12

- Nur Hidayati (05111940000028)
- Pramudityo Prabowo (05111940000210)
- Muhammad Rizky Widodo (05111940000216)

## A. Topologi Jaringan

## B. Subnetting VLSM

## C. Routing VLSM

## D. DHCP Server dan DHCP Relay 

## SOAL 1
Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi tidak menggunakan MASQUERADE.
### Solusi:
Syntax berikut diatur pada router FOOSHA:
```
iptables -t nat -A POSTROUTING -s 10.27.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.2
```
**Penjelasan:**

Kami menggunakan `-t nat` NAT Table pada `-A POSTROUTING` POSTROUTING chain untuk `-j SNAT` mengubah source address yang awalnya berupa private IPv4 address yang memiliki 16-bit blok dari private IP addresses yaitu `-s 10.27.0.0/21` 10.27.0.0/21 menjadi `--to-source 192.168.122.2` 192.168.122.2 karena FOOSHA adalah satu-satunya router yang terhubung ke cloud melalui eth0.

## SOAL 2
Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.
### Solusi:
Syntax pada DHCP Server JIPANGU dan DNS Server DORIKI:
```
iptables -A FORWARD -d 10.27.0.8/29 -i eth0 -p tcp --dport 80 -j DROP
```
**Penjelasan:**

Kami menggunakan `-A FORWARD` chain FORWARD untuk menyaring paket dengan `-p tcp` protokol tcp dari luar topologi menuju DHCP Server JIPANGU dan DNS Server DORIKI pada subnet yang sama yaitu A1 `-d 10.27.0.8/29` dimana akses SSH yang memiliki port 80 (HTTP) `--dport 80` yang masuk ke DHCP Server JIPANGU dan DNS Server DORIKI melalui `-i eth0` interfaces eth 0 paket masuk dari eth0 dari JIPANGU dan DORIKI agar `-j DROP` di DROP.

## SOAL 3
Karena kelompok kalian maksimal terdiri dari 3 orang. Kalian diminta untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.
### Solusi:
Syntax pada DHCP Server JIPANGU dan DNS Server DORIKI:
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```
**Penjelasan:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket dengan `-p icmp` protokol ICMP yang masuk agar dibatasi `-m connlimit --connlimit-above 3` hanya sebatas maksimal 3 koneksi saja `--connlimit-mask 0` darimana saja, sehingga selebihnya akan `-j DROP` di DROP.

## SOAL 4 & 5
Kemudian kalian diminta untuk membatasi akses ke Doriki yang berasal dari subnet Blueno, Cipher, Elena dan Fukuro dengan beraturan sebagai berikut:

4.	Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis.
5.	Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya.

Selain itu di reject
### Solusi Soal 4:
Syntax berikut diatur pada DNS Server DORIKI:
```
#Batas Akses Doriki Dari Blueno
iptables -A INPUT -s 10.27.0.128/25 -d 10.27.0.8/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 10.27.0.128/25 -j REJECT

#Batas Akses Doriki Dari Cipher
iptables -A INPUT -s 10.27.4.0/22 -d 10.27.0.8/29 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 10.27.4.0/22 -j REJECT
```
**Penjelasan:**

**Syntax 1:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.0.128/25` subnet BLUENO `-d 10.27.0.8/29` menuju alamat yaitu IP dari subnet DORIKI `-m time --timestart 07:00 --timestop 15:00` di waktu jam 07:00 sampai dengan jam 15:00 `--weekdays Mon,Tue,Wed,Thu` pada hari Senin, Selasa, Rabu, Kamis agar `-j ACCEPT` diterima.

**Syntax 2:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.0.128/25` subnet BLUENO di jam berapapun selain hari Senin, Selasa, Rabu dan Kamis agar `-j REJECT` ditolak.

**Syntax 3:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.4.0/22` subnet CIPHER `-d 10.27.0.8/29` menuju alamat yaitu IP dari subnet DORIKI `-m time --timestart 07:00 --timestop 15:00` di waktu jam 07:00 sampai dengan jam 15:00 `--weekdays Mon,Tue,Wed,Thu` pada hari Senin, Selasa, Rabu, Kamis agar `-j ACCEPT` diterima.

**Syntax 4:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.4.0/22` subnet CIPHER di jam berapapun selain hari Senin, Selasa, Rabu dan Kamis agar `-j REJECT` ditolak.

### Solusi Soal 5:
Syntax berikut diatur pada DNS Server DORIKI:
```
#Batas Akses Doriki Dari Elena
iptables -A INPUT -s 10.27.2.0/23 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
iptables -A INPUT -s 10.27.2.0/23 -j REJECT

#Batas Akses Doriki Dari Fukurou
iptables -A INPUT -s 10.27.1.0/24 -m time --timestart 15:01 --timestop 06:59 -j ACCEPT
iptables -A INPUT -s 10.27.1.0/24 -j REJECT
```
**Penjelasan:**

**Syntax 1:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.2.0/23` subnet ELENA `-m time --timestart 15:01 --timestop 06:59` di waktu jam 15:01 sampai dengan jam 06:59 setiap harinya agar `-j ACCEPT` diterima.

**Syntax 2:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.2.0/23` subnet ELENA di jam berapapun selain pukul 15:01 sampai 06:59 setiap harinya agar `-j REJECT` ditolak.

**Syntax 3:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.1.0/24` subnet FUKUROU `-m time --timestart 15:01 --timestop 06:59` di waktu jam 15:01 sampai dengan jam 06:59 setiap harinya agar `-j ACCEPT` diterima.

**Syntax 4:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket yang masuk dari `-s 10.27.1.0/24` subnet FUKUROU di jam berapapun selain pukul 15:01 sampai 06:59 setiap harinya agar `-j REJECT` ditolak.

## SOAL 6
Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate.
### Solusi:
Syntax pada GUANHAO diatur sebagai berikut:
```
iptables -t nat -A PREROUTING -p tcp -d 10.27.0.10 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.27.0.18:80
iptables -t nat -A PREROUTING -p tcp -d 10.27.0.10 -j DNAT --to-destination 10.27.0.19:80
iptables -t nat -A POSTROUTING -p tcp -d 10.27.0.18 --dport 80 -j SNAT --to-source 10.27.0.10
iptables -t nat -A POSTROUTING -p tcp -d 10.27.0.19 --dport 80 -j SNAT --to-source 10.27.0.10
```
**Penjelasan:**

**Syntax 1 dan 2:**

Kami menggunakan `-A PREROUTING` PREROUTING chain pada `-t nat` NAT table untuk mengubah destination IP yang awalnya menuju ke `10.27.0.10` DNS Server DORIKI menjadi ke `10.27.0.18:80` Web Server JORGE port 80 dan `10.27.0.19:80` Web Server MAINGATE port 80. Iptables ini juga menyertakan salah satu modul yang ada dalam aturan Load Balancing yaitu `-m statistic` modul statistik. Mode yang digunakan dari modul statistik untuk mengatasi soal 6 ini adalah `--mode nth` yang mengkonfigurasi agar aturan dilewati berdasarkan algoritma Round Robin. Aturan tersebut akan dievaluasi `--every 2` setiap 2 paket mulai dari `--packet 0` paket 0.

**Syntax 3:**

Kami menggunakan `-A POSTROUTING` POSTROUTING chain pada `-t nat` NAT table `-j SNAT` untuk mengubah source address yang awalnya bertujuan `-d 10.27.0.18` ke Web Server JORGE menjadi `--to-source 10.27.0.10` DNS Server DORIKI.

**Syntax 4:**

Kami menggunakan `-A POSTROUTING` POSTROUTING chain pada `-t nat` NAT table `-j SNAT` untuk mengubah source address yang awalnya bertujuan `-d 10.27.0.19` ke Web Server MAINGATE menjadi `--to-source 10.27.0.10` DNS Server DORIKI.

## Kendala Selama Pengerjaan
Adapun beberapa kendala yang kami alami selama pengerjaan soal yaitu sebagai berikut:
- Agak mengalami kesulitan saat menyusun konfigurasi iptables agar sesuai dengan perintah soal.
