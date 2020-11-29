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
Pada UML GRESIK dan MADIUN (client) sering force close tetapi tidak bisa di-*halt* juga, sehingga pengecekan sulit untuk dilakukan.

---
## Dynamic Host Configuration Protocol
1. Membuat topologi jaringan sesuai dengan gambar berikut, konfigurasi untuk topologinya dapat dilihat pada [topologi.sh](https://github.com/liizzasulistio/Jarkom_Modul3_Lapres_D15/blob/main/topologi.sh) atau pada gambar di bawah ini
	* Gambar Topologi 
	<img width="740" alt="topologi_gambar" src="https://user-images.githubusercontent.com/58472359/100222562-dc00ec00-2f4c-11eb-865f-68e13b26ef59.png">
	
	* [Konfigurasi Topologi](https://user-images.githubusercontent.com/58472359/100222567-dc998280-2f4c-11eb-9ad5-cb5c6ec841a4.png)
	~~~
	# switch
	uml_switch -unix switch1 > /dev/null < /dev/null &
	uml_switch -unix switch2 > /dev/null < /dev/null &
	uml_switch -unix switch3 > /dev/null < /dev/null &

	# router
	xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.65 eth1=daemon,,,switch1 eth2=daemon,,,switch3 eth3=daemon,,,switch2 mem=256M &

	# server
	xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
	xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &
	xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &

	# client
	xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &
	xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
	xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
	xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &
	~~~
	
	* Buat juga script untuk halt UML jika telah selesai digunakan [bye.sh](https://github.com/liizzasulistio/Jarkom_Modul3_Lapres_D15/blob/main/bye.sh) atau [screenshot bye](https://user-images.githubusercontent.com/58472359/100222784-2c784980-2f4d-11eb-86d1-d708e1330096.png)
	~~~
	uml_mconsole SURABAYA halt
	uml_mconsole MALANG halt
	uml_mconsole TUBAN halt
	uml_mconsole MOJOKERTO halt
	uml_mconsole GRESIK halt
	uml_mconsole SIDOARJO halt
	uml_mconsole BANYUWANGI halt
	uml_mconsole MADIUN halt
	~~~
2. Seluruh client tidak diperbolehkan menggunakan IP statis dengan SURABAYA ditunjuk sebagai `DHCP Relay`
3. Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200
4. Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70
5. Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP
6. Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan client pada subnet 3 mendapatkan peminjaman IP selama 10 menit

### Langkah-langkah pengerjaan:
1. Membuat konfigurasi interfaces pada Router dan Server, kemudian jalankan `service networking restart`
	* **SURABAYA**
	~~~
	auto eth0
	iface eth0 inet static
	address 10.151.78.66
	netmask 255.255.255.252
	gateway 10.151.78.65
	
	auto eth1
	iface eth1 inet static
	address 192.168.0.1
	netmask 255.255.255.0
	
	auto eth2
	iface eth2 inet static
	address 192.168.1.1
	netmask 255.255.255.0
	
	auto eth3
	iface eth3 inet static
	address 10.151.79.129
	netmask 255.255.255.248
	~~~
	* [Screenshot Interfaces SURABAYA (01)](https://user-images.githubusercontent.com/58472359/100222955-6ea18b00-2f4d-11eb-9daa-de8bec7775d6.png)
	* [Screenshot Interfaces SURABAYA (02)](https://user-images.githubusercontent.com/58472359/100222965-73663f00-2f4d-11eb-8936-d000201a721e.png)
	* Jangan lupa untuk melakukan `export proxy`, melakukan uncomment pada line `net.ipv4.ip_forward=1` di `/etc/sysctl.conf` dan menjalankan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.168.0.0/16` di SURABAYA.
	* **MALANG**
	~~~
	auto eth0
	iface eth0 inet static
	address 10.151.79.130
	netmask 255.255.255.248
	gateway 10.151.79.129
	~~~
	* [Screenshot Interfaces MALANG](https://user-images.githubusercontent.com/58472359/100222976-76612f80-2f4d-11eb-98a1-451b33b01cd5.png)
	* **MOJOKERTO**
	~~~
	auto eth0
	iface eth0 inet static
	address 10.151.79.131
	netmask 255.255.255.248
	gateway 10.151.79.129
	~~~
	* [Screenshot Interfaces MOJOKERTO](https://user-images.githubusercontent.com/58472359/100222980-782af300-2f4d-11eb-92fc-2a66c090d461.png)
	* **TUBAN**
	~~~
	auto eth0
	iface eth0 inet static
	address 10.151.79.132
	netmask 255.255.255.248
	gateway 10.151.79.129
	~~~
	* [Screenshot Interfaces TUBAN](https://user-images.githubusercontent.com/58472359/100222984-795c2000-2f4d-11eb-93d2-5cbe91070935.png)

2. Melakukan `apt-get update` pada Router dan Server kemudian melakukan instalasi pada masing-masing Server dengan cara `apt-get install isc-dhcp-server` pada TUBAN, `apt-get install squid` pada MOJOKERTO dan `apt-get install bind9 -y` pada MALANG.
3. Karena SURABAYA ditunjuk sebagai DHCP Relay, maka lakukan juga instalasi `apt-get install isc-dhcp-relay` di SURABAYA.

4. Membuat konfigurasi interfaces pada Client
	* Konfigurasi (karena tidak diperbolehkan menggunakan IP statis dan bergantung pada DHCP Server yang sama, maka konfigurasinya sama)
	~~~
	auto eth0
	iface eth0 inet dhcp
	~~~
	* [Screenshot Interfaces GRESIK](https://user-images.githubusercontent.com/58472359/100223111-a14b8380-2f4d-11eb-9a13-f27e7943e680.png)
	* [Screenshot Interfaces SIDOARJO](https://user-images.githubusercontent.com/58472359/100223115-a27cb080-2f4d-11eb-8147-9d515a8f5f2c.png)
	* [Screenshot Interfaces BANYUWANGI](https://user-images.githubusercontent.com/58472359/100223125-a577a100-2f4d-11eb-977f-9a1912b7745f.png)
	* [Screenshot Interfaces MADIUN](https://user-images.githubusercontent.com/58472359/100223128-a6a8ce00-2f4d-11eb-909d-ed815db204b6.png)
5. Di TUBAN buka file `/etc/default/isc-dhcp-server` kemudian isikan `INTERFACES = "eth0"` seperti pada gambar di bawah ini:<img width="496" alt="dhcp_server_TUBAN_01" src="https://user-images.githubusercontent.com/58472359/100223224-c809ba00-2f4d-11eb-99b3-4552417eacf0.png">
6. Isikan konfigurasi seperti pada gambar atau pada potongan konfigurasi di bawah ini pada file `/etc/dchp/dhcpd.conf` di TUBAN	
	* [Screenshot DHCP Server di TUBAN (1)](https://user-images.githubusercontent.com/58472359/100223253-d1932200-2f4d-11eb-8fd4-88388d64a15c.png)
	* [Screenshot DHCP Server di TUBAN (2)](https://user-images.githubusercontent.com/58472359/100223258-d35ce580-2f4d-11eb-83ae-a0179e5989c4.png)
	* [Screenshot DHCP Server di TUBAN (3)](https://user-images.githubusercontent.com/58472359/100223263-d48e1280-2f4d-11eb-9c71-bb0dacf5932a.png)
	* Konfigurasi untuk subnet 2 (dari Server ke Router)
	~~~
	subnet 10.151.79.128 netmask 255.255.255.248
	{
	}
	~~~
	* Konfigurasi untuk subnet 1 (dengan ketentuan seperti pada soal di atas)
	~~~
	subnet 192.168.0.0 netmask 255.255.255.0
	{
		range 192.168.0.10  192.168.0.110;
		range 192.168.0.110 192.168.0.200;
		option routers 192.168.0.1;
		option broadcast-address 192.168.0.255;
		option domain-name-servers 10.151.79.130, 202.46.129.2;
		default-lease-time 300;
		max-lease-time 300;
	}
	~~~
	* Konfigurasi untuk subnet 3 (dengan ketentuan seperti pada soal di atas)
	~~~
	subnet 192.168.1.0 netmask 255.255.255.0
	{
		range 192.168.1.50 192.168.1.70;
		option routers 192.168.1.1;
		option broadcast-address 192.168.1.255;
		option domain-name-servers 10.151.79.130, 202.46.129.2;
		default-lease-time 600;
		max-lease-time 600;
	}
	~~~
7. Di SURABAYA tambahkan konfigurasi seperti pada gambar atau pada potongan konfigurasi di bawah ini
	* Line `SERVERS="10.151.79.132"` yang merupakan alamat IP dari TUBAN dan `INTERFACES="eth1 eth2 eth3"`
	* Screenshot
	<img width="496" alt="dhcp_relay_SURABAYA" src="https://user-images.githubusercontent.com/58472359/100223784-888f9d80-2f4e-11eb-912f-7b351eb7af39.png">
8. Lakukan `service isc-dhcp-server restart` pada TUBAN dan `service isc-dhcp-relay restart` pada SURABAYA
9. Lakukan pengecekan dengan cara `service networking restart` kemudian cek IP dengan `ifconfig` pada masing-masing client
	* [Screenshot GRESIK (subnet 1) 01](https://user-images.githubusercontent.com/58472359/100224057-e328f980-2f4e-11eb-903f-4b7bccb52506.png)
	* [Screenshot GRESIK (subnet 1) 02](https://user-images.githubusercontent.com/58472359/100224069-e6bc8080-2f4e-11eb-8f87-0ebbc5baba0c.png)
	* [Screenshot SIDOARJO (subnet 1) 01](https://user-images.githubusercontent.com/58472359/100224133-00f65e80-2f4f-11eb-84ed-b989bcb8d9d7.png)
	* [Screenshot SIDOARJO (subnet 1) 02](https://user-images.githubusercontent.com/58472359/100224135-02c02200-2f4f-11eb-9987-50dae3863e42.png)
	* [Screenshot BANYUWANGI (subnet 3) 01](https://user-images.githubusercontent.com/58472359/100224150-0653a900-2f4f-11eb-9349-1a001c14499f.png)
	* [Screenshot BANYUWANGI (subnet 3) 02](https://user-images.githubusercontent.com/58472359/100224152-0784d600-2f4f-11eb-987c-4d8c639ff773.png)
	* [Screenshot MADIUN (subnet 3) 01](https://user-images.githubusercontent.com/58472359/100224154-08b60300-2f4f-11eb-92a3-4018da040bbb.png)
	* [Screenshot MADIUN (subnet 3) 02](https://user-images.githubusercontent.com/58472359/100224156-09e73000-2f4f-11eb-9f6c-e74f48f5975c.png)
10. Lakukan juga pengecekan dengan cara `cat /etc/resolv.conf` pada masing-masing client untuk mengetahui DNS mana yang dituju, jika menunjukkan `nameserver 10.151.79.130` dan `nameserver 202.46.129.2` maka ketentuan pada soal sudah terpenuhi.
	* [Screenshot GRESIK](https://user-images.githubusercontent.com/58472359/100224336-49158100-2f4f-11eb-9e9e-fe8ceec2171f.png)
	* [Screenshot SIDOARJO](https://user-images.githubusercontent.com/58472359/100224340-4a46ae00-2f4f-11eb-9545-111390bd1ec7.png)
	* [Screenshot BANYUWANGI](https://user-images.githubusercontent.com/58472359/100224347-4b77db00-2f4f-11eb-8db2-922b4c336e62.png)
	* [Screenshot MADIUN](https://user-images.githubusercontent.com/58472359/100224350-4ca90800-2f4f-11eb-9e25-18a493bf47a5.png)
---
## Proxy Server
7. Buat user authentication dengan ketentuan `username = userta_d15` dan `password = inipassw0rdta_d15`
8. Pembatasan penggunaan internet pada hari Selasa - Rabu pukul 13:00 - 18:00
9. Pembatasan penggunaan internet pada hari Selasa - Kamis pukul 21:00 - 09:00 (Jumat keesokan harinya)
10. Setiap mengakses `google.com` akan redirect ke `monta.if.its.ac.id`
11. Mengubah default error page squid sesuai dengan `wget 10.151.36.202/ERR_ACCESS_DENIED`
12. Akses proxy cukup dengan mengetikkan `janganlupa-ta.d15.pw` dan memasukkan port 8080

### Langkah-langkah pengerjaan:
11. Di MOJOKERTO buat konfigurasi username dan password dengan cara `htpasswd -c /etc/squid/passwd userta_d15` kemudian masukkan passwordnya. Jika berhasil, maka pada saat mengakses proxy, proxy akan meminta username dan password.
<img width="512" alt="Proxy_Username_Password_01" src="https://user-images.githubusercontent.com/58472359/100533326-99cdf800-3235-11eb-8194-1ed963862e9c.png">
<img width="532" alt="Proxy_Username_Password_02" src="https://user-images.githubusercontent.com/58472359/100533321-963a7100-3235-11eb-8721-524b4eceb35b.png">

12. Di MOJOKERTO buat file `etc/squid/acl.conf` dengan isian seperti pada konfigurasi atau screenshot di bawah ini
	* Konfigurasi
	~~~
	acl AVAILABLE_WORKING time TW 13:00-18:00
	acl AVAILABLE_GOOGLE_NIGHT time TWH 21:00-23:59
	acl AVAILABLE_GOOGLE_DAY time WHF 00:00-09:00
	~~~
	* [Screenshot Proxy Waktu](https://user-images.githubusercontent.com/58472359/100533324-976b9e00-3235-11eb-8d84-698717eaf7f9.png)
	
13. Konfigurasi di `/etc/squid/squid.conf` dan tampilan ketika mencoba mengakses google.com dan _redirect_ monta.if.its.ac.id
	* Konfigurasi
	~~~
	...
	acl  BLACKLISTS google.com

	~~~
	* [Screenshot Redirect Monta](https://user-images.githubusercontent.com/58472359/100533426-a7d04880-3236-11eb-8a99-57e57bdcc116.png)
	
14. Konfigurasi dan screenshot keseluruhan pada `etc/squid/squid.conf` di MOJOKERTO

	- Konfigurasi
		- Tambahkan baris `error_directory /usr/share/squid/error403`
		- Ekstrak file tar.gz yang diberikan pada `/usr/share/squid/`
		- Unggah file error.jpg di website eksternal
		- Pada file error.html, ganti `url('error.jpg')` dengan `url([Hotlink Image])`. Pada kasus kami, kami menggunakan [http://167.99.66.187:9999/error.jpg](http://167.99.66.187:9999/error.jpg).
		- Ganti nama file `error.html` manjadi `ERR_DNS_FAIL`
		- Restart squid.
	
	- Screenshot
	![Screenshot Error Page](https://user-images.githubusercontent.com/24503760/100541230-ef75c500-3274-11eb-8169-33abfc7fbd92.png)
15. Di MALANG buka `/etc/bind/named.conf.local` kemudian isikan sesuai dengan konfigurasi atau screenshot di bawah ini:
	~~~
	...
	zone "janganlupa-ta.d15.pw
	{
		type master;
		file "/etc/bind/jarkom/janganlupa-ta.d15.pw";
	};
	~~~
	<img width="496" alt="DNS_Server_Malang_01" src="https://user-images.githubusercontent.com/58472359/100533293-5c696a80-3235-11eb-8271-d1b9042a0cb6.png">
16. Buat direktori `/etc/bind/jarkom` kemudian copy `/etc/bind/db.local` ke `/etc/bind/jarkom/janganlupa-ta.d15.pw`, kemudian isikan sesuai dengan konfigurasi atau screenshot di bawah ini, dan restart:
	~~~
	...
	@	IN	 SOA	janganlupa-ta.d15.pw. root.janganlupa-ta.d15.pw.
	...
	@	IN	 NS	janganlupa-ta.d15.pw.
	@	IN	 A	10.151.79.131 ;IP MOJOKERTO
	~~~
	<img width="496" alt="DNS_Server_Malang_02" src="https://user-images.githubusercontent.com/58472359/100533294-5d020100-3235-11eb-89fd-970ac18475e5.png">
17. Lakukan pengecekan dengan cara terkoneksi pada proxy dan `ping janganlupa-ta.d15.pw` di terminal, seperti pada dua gambar di bawah ini
<img width="780" alt="Akses_Proxy_01" src="https://user-images.githubusercontent.com/58472359/100533285-56738980-3235-11eb-939e-54d72b30b111.png">
<img width="717" alt="Akses_Proxy_02" src="https://user-images.githubusercontent.com/58472359/100533287-57a4b680-3235-11eb-8f4d-543aeda31c96.png">

18. Ubah konfigurasi proxy untuk mengetahui apakah berhasil menggunakan proxy jika memasukkan `janganlupa-ta.d15.pw` dan port 8080 saja
<img width="780" alt="Akses_Proxy_03" src="https://user-images.githubusercontent.com/58472359/100533288-583d4d00-3235-11eb-821e-022b76e78c05.png">
