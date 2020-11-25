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
1. Membuat topologi jaringan sesuai dengan gambar berikut, konfigurasi untuk topologinya dapat dilihat pada [!topologi.sh](https://github.com/liizzasulistio/Jarkom_Modul3_Lapres_D15/blob/main/topologi.sh)
2. Seluruh client tidak diperbolehkan menggunakan IP statis dengan SURABAYA ditunjuk sebagai `DHCP Relay`
3. Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200
4. Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70
5. Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP
6. Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan client pada subnet 3 mendapatkan peminjaman IP selama 10 menit





---
## Proxy Server
