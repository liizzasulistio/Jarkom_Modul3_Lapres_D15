# Jarkom_Modul3_Lapres_D15
- Ammar Alifian Fahdan (05111840000007)
- Lii'zza Aisyah Putri Sulistio (05111840000073)
---
## Ketentuan
1. Default memori UML adalah 64M, kecuali:
	* SURABAYA : 256M
	* MALANG : 160M
	* MOJOKERTO : 128M
	* TUBAN: 128M
2. Menghitung dan menggunakan IP sesuai dengan NID Tuntap dan NID DMZ masing-masing kelompok
	* NID Tuntap : 10.151.78.64/30
	* NID DMZ : 10.151.79.128/29
3. IP Tuntap : NID_tuntap + 1 = 10.151.78.65
4. IP Inferface Router SURABAYA:
	* eth0 : NID_tuntap + 2 = 10.151.78.66
	* eth3 : NID_DMZ +1 = 10.151.79.129
	* eth1 : 192.168.0.1
	* eth2 : 192.168.1.1
5. IP Server (SUBNET 2):
	* `DNS Server` MALANG : NID_DMZ + 2 = 10.151.79.130
	* `Proxy Server` MOJOKERTO : NID_DMZ + 3 = 10.151.79.131
	* `DHCP Server` TUBAN : NID_DMZ + 4 = 10.151.79.132
---
## Kendala
Pada UML GRESIK (client) sering force close tetapi tidak bisa di-halt juga, sehingga pengecekan sulit untuk dilakukan.

---
## Dynamic Host Configuration Protocol
1. Membuat topologi jaringan sesuai dengan gambar berikut, konfigurasi untuk topologinya dapat dilihat pada [topologi.sh](https://github.com/liizzasulistio/Jarkom_Modul3_Lapres_D15/blob/main/topologi.sh) atau pada gambar di bawah ini
	* Gambar Topologi
	* Konfigurasi Topologi
2. Seluruh client tidak diperbolehkan menggunakan IP statis dengan SURABAYA ditunjuk sebagai `DHCP Relay`
3. Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200
4. Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70
5. Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP
6. Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan client pada subnet 3 mendapatkan peminjaman IP selama 10 menit

### Langkah-langkah pengerjaan:
1. Membuat konfigurasi interfaces pada Router dan Server, kemudian jalankan `service networking restart`
	* SURABAYA
	* Jangan lupa untuk melakukan `export proxy`, melakukan uncomment pada line `net.ipv4.ip_forward=1` di `/etc/sysctl.conf` dan menjalankan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.168.0.0/16` di SURABAYA.
	* MALANG
	* MOJOKERTO
	* TUBAN
2. Melakukan `apt-get update` pada Router dan Server kemudian melakukan instalasi pada masing-masing Server dengan cara `apt-get install isc-dhcp-server` pada TUBAN, `apt-get install squid` pada MOJOKERTO dan `apt-get install bind9 -y` pada MALANG.
3. Karena SURABAYA ditunjuk sebagai DHCP Relay, maka lakukan juga instalasi `apt-get install isc-dhcp-relay` di SURABAYA.
4. Membuat konfigurasi interfaces pada Client
	* GRESIK
	* SIDOARJO
	* BANYUWANGI
	* MADIUN
5. Di TUBAN buka file `/etc/default/isc-dhcp-server` kemudian isikan `INTERFACES = "eth0"` seperti pada gambar di bawah ini
6. Isikan konfigurasi seperti pada gambar atau pada potongan konfigurasi di bawah ini pada file `/etc/dchp/dhcpd.conf` di TUBAN
	* Screenshot
	* Konfigurasi untuk subnet 2 (dari Server ke Router)
	* Konfigurasi untuk subnet 1 (dengan ketentuan seperti pada soal di atas)
	* Konfigurasi untuk subnet 3 (dengan ketentuan seperti pada soal di atas)
7. Di SURABAYA tambahkan konfigurasi seperti pada gambar atau pada potongan konfigurasi di bawah ini
	* Screenshot
	* Line `SERVERS="10.151.79.132"` yang merupakan alamat IP dari TUBAN dan `INTERFACES="eth1 eth2 eth3"`
8. Lakukan `service isc-dhcp-server restart` pada TUBAN dan `service isc-dhcp-relay restart` pada SURABAYA
9. Lakukan pengecekan dengan cara `service networking restart` kemudian cek IP dengan `ifconfig` pada masing-masing client
	* GRESIK
	* SIDOARJO
	* BANYUWANGI
	* MADIUN
10. Lakukan juga pengecekan dengan cara `cat /etc/resolv.conf` pada masing-masing client untuk mengetahui DNS mana yang dituju, jika menunjukkan `nameserver 10.151.79.130` dan `nameserver 202.46.129.2` maka ketentuan pada soal sudah terpenuhi.
	* GRESIK
	* SIDOARJO
	* BANYUWANGI
	* MADIUN


---
## Proxy Server
