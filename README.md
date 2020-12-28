# Jarkom_Modul5_Lapres_E12
* Restu Agung Parama - 05111840000123

## VLSM
![Screenshot from 2020-12-29 02-40-20](https://user-images.githubusercontent.com/58405725/103239625-b8ccc080-4980-11eb-8ed1-9a4a3a3e0344.png)

Untuk tree VLSM yg dibuat dapat dilihat di link berikut:

https://gitmind.com/app/doc/5311376195

Tabel berikut menampilkan subnet dan jumlah IP untuk mendapatkan netmask tiap subnet:

|                  |                   |                 |
|------------------|-------------------|-----------------|
| A1               | Network ID        | 192.168.0.0     |
|                  | Netmask           | 255.255.255.252 |
|                  | Broadcast Address | 192.168.0.3     |
| A2               | Network ID        | 192.168.0.4     |
|                  | Netmask           | 255.255.255.252 |
|                  | Broadcast Address | 192.168.0.7     |
| A3               | Network ID        | 192.168.0.8     |
|                  | Netmask           | 255.255.255.248 |
|                  | Broadcast Address | 192.168.0.15    |
| A4               | Network ID        | 192.168.2.0     |
|                  | Netmask           | 255.255.255.0   |
|                  | Broadcast Address | 192.168.2.255   |
| A5               | Network ID        | 192.168.3.0     |
|                  | Netmask           | 255.255.255.0   |
|                  | Broadcast Address | 192.168.3.255   |
| DMZ              | Network ID        | 10.151.71.104   |
|                  | Netmask           | 255.255.255.248 |
|                  | Broadcast Address | 10.151.71.111   |

</justify>

## DHCP
<p>Konfigurasi <code>/etc/default/isc-dhcp-server</code> pada Mojokerto (DHCP Server) : </p>

    INTERFACES="eth0"

<p>Konfigurasi <code>/etc/dhcp/dhcpd.conf</code> pada Mojokerto (DHCP Server) : </p>

```
subnet 192.168.2.0 netmask 255.255.255.0 {
    range 192.168.2.2 192.168.2.254;
    option routers 192.168.2.1;
    option broadcast-address 192.168.2.255;
    option domain-name-servers 10.151.71.106,202.46.129.2;
    default-lease-time 60;
    max-lease-time 300;
}

subnet 192.168.3.0 netmask 255.255.255.0 {
    range  192.168.3.2 192.168.3.254;
    option routers 192.168.3.1;
    option broadcast-address 192.168.3.255;
    option domain-name-servers 10.151.71.106,202.46.129.2;
    default-lease-time 60;
    max-lease-time 300;
}

subnet 10.151.71.104 netmask 255.255.255.248 {
}
```

<p>Konfigurasi <code>/etc/default/isc-dhcp-relay</code> pada DHCP Relay (Batu dan Kediri): </p>

<p>
    - <code>SERVERS</code> : Menuju IP Server DHCP Server, yaitu IP Mojokerto <br>
    - <code>INTERFACES</code> : Menuju interface client dan server <br>
</p>


## Firewall

<p>1. Konfigurasi iptables pada Surabaya agar topologi dapat mengakses keluar tetapi tidak menggunakan MASQUERADE:</p>

    iptables -t nat -A POSTROUTING -s 192.168.0.0/22 -o eth0 -j SNAT --to-source 10.151.70.54

<p>Keterangan :</p>
<p>- <code>-t nat</code> : menggunakan table nat</p>
<p>- <code>-A POSTROUTING</code> : karena perubahan dilakukan setelah proses routing</p>
<p>- <code>-o eth0</code> : karena paket keluar melewati interface eth0</p>
<p>- <code>-j SNAT</code> : untuk mengubah source ip address</p>
<p>- <code>--to 10.151.70.54</code> : source ip address siubah dengan ip eth0 Surabaya</p>
<p>- <code>–s 192.168.0.0/22</code> : NID yang akan diubah dengan SNAT</p>


<p>2. Konfigurasi iptables pada Surabaya untuk mendrop semua akses SSH dari luar Topologi (UML) pada DHCP dan DNS SERVER dan melakukan logging:</p>

    iptables -N LOGGING
    iptables -A FORWARD -p tcp --dport 22 -d 10.151.71.104/29 -i eth0 -j LOGGING
    iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
    iptables -A LOGGING -j DROP

<p>Keterangan :</p>
<p>- <code>-A FORWARD</code> : menggunakan chain FORWARD</p>
<p>- <code>-p tcp</code> : menggunakan protokol tcp (karena ssh)</p>
<p>- <code>-d 10.151.79.32/29</code> : NID tujuan (NID DHCP dan DNS SERVER)</p>
<p>- <code>--dport 22</code> : port ssh</p>
<p>- <code>--log-level 4</code> : log level warning</p>
<p>- <code>-j DROP</code> : untuk mendrop paket</p>


<p>3. Membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan ​iptables pada masing masing server​ , selebihnya akan di DROP dan melakukan logging:</p>

    iptables -N LOGGING
    iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
    iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
    iptables -A LOGGING -j DROP

<p>Keterangan :</p>
<p>- <code>-A INPUT</code> : menggunakan chain INPUT</p>
<p>- <code>-p icmp</code> : menggunakan protokol icmp</p>
<p>- <code>--connlimit-above 3</code> : membatasi 3 koneksi icmp</p>
<p>- <code>--connlimit-mask 0</code> : menandakan koneksi untuk semua sumber (berasal dari mana saja)</p>



<p>4. Membatasi akses ke MALANG yang berasal dari Subnet Sidoarjo pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat selain itu paket akan di REJECT. Untuk iptables saya meletakannya pada router Batu.</p>

    iptables -A INPUT -s 192.168.3.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
    iptables -A INPUT -s 192.168.3.0/24 -m time --timestart 07:00 --timestop 17:01 --weekdays Sat,Sun -j REJECT
    iptables -A INPUT -s 192.168.3.0/24 -m time --timestart 17:01 --timestop 06:59 -j REJECT

<p>Keterangan :</p>
<p>- <code>--timestart</code> : menandakan waktu mulai</p>
<p>- <code>--timestop</code> : menandakan waktu selesai</p>
<p>- <code>--weekdays</code> : menentukan hari yang ingin di setting pada iptables</p>
</justify>


<p>5. Membatasi akses ke MALANG yang berasal dari Subnet Gresik yang hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya selain itu paket akan di REJECT.Untuk iptables saya meletakannya pada router Batu.</p>

    iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:01 --timestop 16:59 -j REJECT


<p>6. Saya SKIP, ada segfault saat revisi.. jadinya rusak :)

<p>7. Paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap
UML yang memiliki aturan drop</p>

    telah dicantumkan pada soal yang menggunakan iptables DROP
