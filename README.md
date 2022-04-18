# Script-ITNSA-Linux-Server
# NAT Masquerade
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


# DNS Server
named.conf.default zones
-> zone "example.com"
		/db.domain;   ->db.domain
				-> @	IN	NS	example.com
				   @	IN	A	172.16.2.2->IP Debian
				   @	IN	MX	2	mail.example.com -> Web Mail
				   www	IN	A	192.168.1.2 -> IP Web Server
				   mail	IN	A	192.168.1.2 -> IP Web Mail
				   ldap	IN	A	192.168.1.3 -> IP LDAP Server	
-> zone "example2.com"
		/db.domain2;  ->db.domain2
				-> @	IN	NS	example2.com
				   @	IN	A	192.168.1.10 -> IP Web Server
				   www	IN	A	192.168.1.10
-> zone	"2.16.172.in.addr-arpa"
		/db.IP	      ->db.IP
			        -> @	IN	NS	example.com
				<- 2	IN	PTR	example.com
Angka terakhir IP yang diarahkan<- 2	IN	PTR	mail.example.com
				<- 3	IN	PTR	ldap.example.com
				<- 10	IN	PTR	example2.com  
# Load Balancing
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

# Web Server

chown -R www-data:www-data /var/www/html/wordpress


# MariaDB
mysql -u root -p
create database dbwp;
GRANT ALL PRIVILEGES ON dbwp.* TO "dbwp"@"localhost" IDENTIFIED BY "dbwp";
exit;

