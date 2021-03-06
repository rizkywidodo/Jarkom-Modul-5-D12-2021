# Jarkom-Modul-5-D12-2021
Laporan Resmi Praktikum Jaringan Komputer 2021 - D12

- Nur Hidayati (05111940000028)
- Pramudityo Prabowo (05111940000210)
- Muhammad Rizky Widodo (05111940000216)

## A. Topologi Jaringan
Pertama, membuat topologi jaringan sesuai rancangan yaitu sebagai berikut:

![](./image/a.1.jpg)

Keterangan:
- Doriki sebagai DNS Server
- Jipangu sebagai DHCP Server
- Maingate dan Jorge sebagai Web Server
- Jumlah Host pada Blueno adalah 100 host
- Jumlah Host pada Cipher adalah 700 host
- Jumlah Host pada Elena adalah 300 host
- Jumlah Host pada Fukurou adalah 200 host

## B. Subnetting VLSM
Kelompok kami menggunakan teknik VLSM (Variable Length Subnet Masking) dengan langkah-langkah sebagai berikut:

**Langkah 1** - Melakukan pembagian subnet berdasarkan topologi yang terdapat pada soal.

![](./image/b.1.jpg)

Dari hasil pembagian subnet, kita mendapatkan delapan subnet yang terdiri atas dua subnet untuk router-router (A4, A5), empat subnet untuk router-client (A2, A3, A6, A7) dan dua subnet untuk router-server (A1 dan A8).

**Langkah 2** - Menentukan jumlah alamat IP yang dibutuhkan oleh tiap subnet dan lakukan labelling netmask berdasarkan jumlah IP yang dibutuhkan.

| Subnet       | Jumlah IP     | Netmask       |
| ------------ | ------------- | ------------- |
| A1           | 3             | /29           |
| A2           | 101           | /25           |
| A3           | 701           | /22           |
| A4           | 2             | /30           |
| A5           | 2             | /30           |
| A6           | 301           | /23           |
| A7           | 201           | /24           |
| A8           | 3             | /29           |
| Total        | 1314          | /21           |

**Langkah 3** - Subnet besar yang dibentuk memiliki NID ``10.27.0.0`` dengan netmask ``/21``. Menghitung pembagian IP berdasarkan NID dan netmask tersebut menggunakan tree seperti gambar di bawah.

![](./image/b.2.jpg)

Dari tree tersebut akan mendapat pembagian IP sebagai berikut:

![](./image/b.3.jpg)

Selanjutnya, hasil dari perhitungan subnet tersebut dapat diimplementasikan ke dalam GNS3. Pada file `/etc/sysctl.conf ` di router yang meliputi **FOOSHA**, **WATER7**, dan **GUANHAO** diedit dengan melakukan uncomment pada command `net.ipv4.ip forward=1` tujuannya agar dapat meneruskan route nantinya. Kemudian untuk mengaktifkan perubahan baru megetikkan command ` sysctl -p `.

Lalu, karena setiap subnet sudah mendapatkan pembagian IP, perlu dilakukan setting pada file `/etc/network/interfaces` masing-masing sebagai berikut:

**FOOSHA (Router)**

![](./image/b.4.jpg)

**WATER7 (Router)**

![](./image/b.5.jpg)

**GUANHAO (Router)**

![](./image/b.6.jpg)

**DORIKI (DNS Server)**

![](./image/b.7.jpg)

**JIPANGU (DHCP Server)**

![](./image/b.8.jpg)

**JORGE (Web Server)**

![](./image/b.9.jpg)

**MAINGATE (Web Server)**

![](./image/b.10.jpg)

Client mendapatkan pembagian IP bukan secara static melainkan dari DHCP, maka akan dibahas pada subbab D nantinya. 

## C. Routing VLSM
Untuk routing, diberikan static route pada semua router yang ada dengan route sebagai berikut untuk setiap router:

![](./image/c.png)

Selanjutnya diimlementasikan ke GNS3 pada masing-masing router, sebagai berikut:

### FOOSHA
FOOSHA diberikan route ke arah subnet A1, A2, A3, A6, A7, dan A8.

![](./image/c.1.png)

### WATER7
WATER7 diberikan default route.

![](./image/c.2.png)

### GUANHAO
GUANHAO diberikan default route.

![](./image/c.3.png)

Agar routing tidak perlu dilakukan berulang-ulang, disimpan pada file bash dengan nama `route.sh` kemudian untuk menjalankannya dengan mengetikkan command `bash route.sh`.

## D. DHCP Server dan DHCP Relay 
Agar IP Dinamis diberikan kepada client BLUENO, CIPHER, ELENA, dan FUKUROU dapat berjalan, kita perlu melakukan setting konfigurasi DHCP Server di JIPANGU dengan langkah-langkah sebagai berikut:

**Langkah 1** ??? Menginstall DHCP Server pada JIPANGU dengan command berikut:
```
apt-get update
apt-get install isc-dhcp-server -y
```
**Langkah 2** ??? Melakukan setting interfaces yang akan diberikan layanan DHCP oleh DHCP Server JIPANGU pada file `/etc/default/isc-dhcp-server `.

![](./image/d.1.png)

Interfaces dari server JIPANGU hanya ada satu interfaces yaitu `eth0`, dimana interfaces ini terhubung ke router WATER7.

**Langkah 3** ??? Agar DHCP Server JIPANGU dapat berjalan dengan lancar, maka kita perlu untuk melakukan deklarasi subnet yang terkoneksi pada JIPANGU yakni subnet A1, A2, A3, A6, dan A7 pada file `/etc/dhcp/dhcpd.conf`.

![](./image/d.2.jpg)

**Langkah 4** ??? Pada FOOSHA, WATER7, dan GUANHAO yang akan menjadi DHCP Relay, maka perlu menginstallnya dengan command
```
apt-get update
apt-get install isc-dhcp-relay
```
Setelah proses penginstallan berhasil, kita mulai untuk melakukan setting server dan setting interfaces yang membantu DHCP Request dapat diteruskan dengan baik ke DHCP Server pada file `/etc/default/isc-dhcp-relay`

**DHCP Relay pada FOOSHA**

![](./image/d.3.png)

Keterangan:

- `SERVERS="10.27.0.11"`: **JIPANGU** diminta oleh DHCP Relay **FOOSHA** untuk meneruskan DHCP Request, sehingga diisi dengan IP dari DHCP Server **JIPANGU**.
- `INTERFACES="eth1 eth2"`: DHCP Relay **FOOSHA** akan meneruskan DHCP Request dari subnet A4 **(WATER7)** dan subnet A5 **(GUANHAO)** dari network interfaces eth1 eth2.

**DHCP Relay pada WATER7**

![](./image/d.4.png)

Keterangan:

- `SERVERS="10.27.0.11"`: **JIPANGU** diminta oleh DHCP Relay **WATER7** untuk meneruskan DHCP Request, sehingga diisi dengan IP dari DHCP Server **JIPANGU**.
- `INTERFACES="eth0 eth1 eth2 eth3"`: DHCP Relay **WATER7** akan meneruskan DHCP Request dari subnet A2 **(BLUENO)** dan subnet A3 **(CIPHER)**.

**DHCP Relay pada GUANHAO**

![](./image/d.5.png)

Keterangan:

- `SERVERS="10.27.0.11"`: *JIPANGU* diminta oleh DHCP Relay *GUANHAO* untuk meneruskan DHCP Request, sehingga diisi dengan IP dari DHCP Server *JIPANGU*.
- `INTERFACES="eth0 eth1 eth3"`: DHCP Relay *GUANHAO* akan meneruskan DHCP Request dari subnet A6 *(ELENA)* dan subnet A7 *(FUKUROU)*.

**Langkah 5** ??? Pada masing-masing client yaitu **BLUENO, CIPHER, ELENA,** dan **FUKUROU** melakukan edit pada file `/etc/network/interfaces` sehingga menjadi sebagai berikut:
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

## SOAL 1
Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi tidak menggunakan MASQUERADE.
### Solusi:
Syntax berikut diatur pada router FOOSHA:
```
iptables -t nat -A POSTROUTING -s 10.27.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.2
```
**Penjelasan:**

Kami menggunakan `-t nat` NAT Table pada `-A POSTROUTING` POSTROUTING chain untuk `-j SNAT` mengubah source address yang awalnya berupa private IPv4 address yang memiliki 16-bit blok dari private IP addresses yaitu `-s 10.27.0.0/21` 10.27.0.0/21 menjadi `--to-source 192.168.122.2` 192.168.122.2 karena FOOSHA adalah satu-satunya router yang terhubung ke cloud melalui eth0.

Testing berhasil apabila semua node bisa melakukan `ping google.com`.

## SOAL 2
Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.
### Solusi:
Syntax pada DHCP Server JIPANGU dan DNS Server DORIKI:
```
iptables -A FORWARD -d 10.27.0.8/29 -i eth0 -p tcp --dport 80 -j DROP
```
**Penjelasan:**

Kami menggunakan `-A FORWARD` chain FORWARD untuk menyaring paket dengan `-p tcp` protokol tcp dari luar topologi menuju DHCP Server JIPANGU dan DNS Server DORIKI pada subnet yang sama yaitu A1 `-d 10.27.0.8/29` dimana akses SSH yang memiliki port 80 (HTTP) `--dport 80` yang masuk ke DHCP Server JIPANGU dan DNS Server DORIKI melalui `-i eth0` interfaces eth 0 paket masuk dari eth0 dari JIPANGU dan DORIKI agar `-j DROP` di DROP.

**Testing:**
1. Menginstall netcat pada JIPANGU dan DORIKI dengan command:
```
apt-get update
apt-get install netcat -y
```
2. Pada JIPANGU dan DORIKI ketikkan command `nc -l -p 80`.
3. Pada FOOSHA ketikkan command `nmap -p 80 10.27.0.10` (IP DORIKI) atau `nmap -p 80 10.27.0.11` (IP JIPANGU). Maka hasilnya sebagai berikut:

![](./image/img_2.jpg)

## SOAL 3
Karena kelompok kalian maksimal terdiri dari 3 orang. Kalian diminta untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.
### Solusi:
Syntax pada DHCP Server JIPANGU dan DNS Server DORIKI:
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```
**Penjelasan:**

Kami menggunakan `-A INPUT` INPUT chain untuk menyaring paket dengan `-p icmp` protokol ICMP yang masuk agar dibatasi `-m connlimit --connlimit-above 3` hanya sebatas maksimal 3 koneksi saja `--connlimit-mask 0` darimana saja, sehingga selebihnya akan `-j DROP` di DROP.

**Testing:**
1. Masuk ke empat node berbeda.
2. Melakukan ping ke arah JIPANGU dengan command `ping 10.27.0.11`. Hasilnya sebagai berikut:

![](./image/img_3.1.jpg)

![](./image/img_3.2.jpg)

![](./image/img_3.3.jpg)

![](./image/img_3.4.jpg)

Dapat dilihat pada gambar, untuk node BLUENO, CIPHER, dan ELENA berhasil mengakses JIPANGU, tetapi untuk node FUKUROU tidak dapat mengakses JIPANGU.

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

**Testing:**

1. Pada DORIKI, date diatur supaya menjadi 9 Nov pukul 10:00 dengan command `date -s "9 nov 2021 10:00:00"`.
2. Pada ke empat client melakukan ping ke arah DORIKI dengan command `ping 10.27.0.10`.

![](./image/img_4.1.jpg)

![](./image/img_4.2.jpg)

![](./image/img_4.3.jpg)

![](./image/img_4.4.jpg)

Dari hasil tersebut, dapat dilihat bahwa BLUENO dan CIPHER dapat mengakses DORIKI, sedangkan ELENA dan FUKUROU tidak dapat mengakses DORIKI.

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

**Testing:**

1. Pada DORIKI, date diatur supaya menjadi 9 Nov pukul 17:00 dengan command `date -s "9 nov 2021 17:00:00"`.
2. Pada ke empat client melakukan ping ke arah DORIKI dengan command `ping 10.27.0.10`.

![](./image/img_5.1.jpg)

![](./image/img_5.2.jpg)

![](./image/img_5.3.jpg)

![](./image/img_5.4.jpg)

Dari hasil tersebut, dapat dilihat bahwa BLUENO dan CIPHER tidak dapat mengakses DORIKI, sedangkan ELENA dan FUKUROU dapat mengakses DORIKI.

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

**Testing:**

1. Pada GUANHAO, JORGE, MAINGATE dan ELENA menginstall netcat dengan command:
```
apt-get update
apt-get install netcat -y
```
2. Pada JORGE dan MAINGATE ketikkan perintah `nc -l -p 80`.
3. Pada client ELENA ketikkan perintah `nc 10.27.0.10 80`.
4. Pada client ELENA ketikkan sembarang maka akan terdistribusi ke JORGE dan MAINGATE.

![](./image/img_6.1.jpg)

![](./image/img_6.2.jpg)

![](./image/img_6.3.jpg)

### Kendala Selama Pengerjaan
Adapun beberapa kendala yang kami alami selama pengerjaan soal yaitu sebagai berikut:
- Agak mengalami kesulitan saat menyusun konfigurasi iptables agar sesuai dengan perintah soal.
