# Script-ITNSA-Linux-Server
# SSH Server
1. Install Package SSH Server

		apt install ssh

2. Check Apakah sudah Running

		systemctl status ssh

# FTP Server
1. Install Package FTP Server

		apt install proftpd

2. Check apakah FTP Server sudah running

		systemctl status proftpd


# NAT Masquerade (Server Side)
1. Aktifkan IP Forward dengan perintah berikut

		nano /etc/sysctl.conf

2. Kemudian uncoment pada baris kode berikut

		net.ipv4.ip_forward=1
	
3. Install Package Iptables
	
		apt install iptables-persistent -y

4. Ketikan Perintah Routing NAT berikut
	
		/sbin/iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

5. Check apakah sudah ada NAT

		/sbin/iptables -t nat -nL
		
6. Save Konfigurasi IP Tables

		/sbin/iptables-save > /etc/iptables/rules.v4

7. Membuat Restore Point dengan cara Menambah command di interface jaringan 

		nano /etc/network/interfaces
		
		up command /sbin/iptables-restore < /etc/iptables/rules.v4


# DNS Server (Server Side)
1. Install Package bind9

		apt install bind9

2. Masuk ke direktori bind dengan perintah

		cd /etc/bind/

3. Ketikan perintah berikut untuk menampilkan semua isi dari dalam direktori bind

		ls

4. Kemudian copy file db.127 dan db.local dengan nama db.ip dan db.domain 

		cp db.127 db.ip | cp db.local db.domain

5. Konfigurasi file db.ip

		nano db.ip

6. Lalu replace localhost dengan nama domain yang ingin dipakai



		; BIND reverse data file for local loopback interface
		;
		$TTL    604800
		@       IN      SOA     example.lan. root.example.lan. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
		;
		@       IN      NS      example.lan.
		2       IN      PTR     example.lan.
	2 -> merupakan angka terakhir dari IP DNS Server (Alamat IP Server)

7. Konfigurasi file db.domain

		nano db.domain

8.  Replace localhost dengan nama domain 


		; BIND data file for local loopback interface
		;
		$TTL    604800
		@       IN      SOA     example.lan. root.example.lan. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
		;
		@       IN      NS      example.lan.
		@       IN      A       192.168.43.2

		
	Anda bisa menambahkan sub domain lainya Ex.

		@       IN      MX      mail.example.lan
		www     IN      A       192.168.3.2
		mail    IN      A       192.168.3.3

9. Konfigurasi file named.conf.default-zones

		nano  named.conf.default-zones

10. Replace localhost dengan nama domain

		
		zone "example.lan" {
        type master;
        file "/etc/bind/db.domain";
		};

		zone "43.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.ip";
		};

	Dan ubah path sesuai dengan konfigurasi yang sudah dibuat tadi. sesuaikan Alamat IP pada server (nb. Penulisan IP dari belakang ke depan)

11. Restart Konfigurasi bind9

		systemctl restart bind9

12. Check apakah DNS Server sudah running

		systemctl status bind9

13. Setting nameserver

		nano /etc/resolv.conf

14. Tambahkan alamat DNS Server

		nameserver 192.168.43.2

15. Lakukan test ping kepada domain yang telah dibuat

		ping example.lan
# Load Balancing (Server Side)
1. Install Package Nginx

		apt install nginx


upstream backend {
	server 192.168.1.2;
	server 192.168.1.6;
	server 192.168.1.10;
}
	server {
	listen 80;
	listen [::]:80;
	server_name example.com;
	location /{
		proxy_redirect off;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $http_host;
		proxy_pass	http://backend;
	}
}

# Web Server (Client Side)

chown -R www-data:www-data /var/www/html/wordpress


# MariaDB
mysql -u root -p
create database dbwp;
GRANT ALL PRIVILEGES ON dbwp.* TO "dbwp"@"localhost" IDENTIFIED BY "dbwp";
exit;

