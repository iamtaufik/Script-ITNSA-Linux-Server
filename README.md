# Script-ITNSA-Linux-Server
# Konfigurasi Jaringan Ubuntu 20.04 Server
1. Edit pada file berikut

		nano /etc/netplan/00-installer-config.yaml

2. Konfigurasi jaringan
		
		network:
 		  ethernets:
 		    enp0s3:
		      dhcp4: false
		      addresses: [192.168.43.20/24]
		      gateway4: 192.168.43.1
		    enp0s8:
		      dhcp4: false
 		      addresses: [192.168.5.1/24]
  		version: 2

# Konfigurasi Name Server
1. Edit pada file berikut

		nano /etc/resolv.conf
		
2. Tambahkan nameserver berikut

		nameserver 8.8.8.8

3. Lakukan Update

		apt-get update

# SSH Server
1. Install Package SSH Server

		apt install ssh

2. Check Apakah SSH Server sudah Running
 
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

16. Perintah untuk kembali ke direktori Home

		cd $HOME

# Load Balancing (Server Side)
1. Install Package Nginx

		apt install nginx

2. Membuat file untuk menyimpan konfigurasi load balance

		nano /etc/nginx/conf.d/load-balance.conf

	Pastikan untuk ekstensi file .conf

3. Menambahkan konfigurasi server block untuk load balancing

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
				proxy_set_header X-Forwarded-For 	$proxy_add_x_forwarded_for;
				proxy_set_header Host $http_host;
				proxy_pass	http://backend;
			}
		}

4. Restart service Nginx

		systemctl restart nginx

5. Check status Nginx

		systemctl status nginx

# Database Server

1. Install Package MariaDB

		apt install mariadb-server

2. Masuk ke Mariadb 

		mysql -u root -p

3. Membuat database baru dengan nama dbwp

		create database dbwp;

4. Berikan hak akses database

		GRANT ALL PRIVILEGES ON dbwp.* TO "dbwp"@"localhost" IDENTIFIED BY "dbwp";

5. Keluar dari MariaDB

		exit;

# Setup CMS (Client Side)

1. Install package web server Nginx

		apt install nginx

2. Install utility PHP7.3

		apt install php7.3 php7.3-fpm php7.3-cgi php7.3-mysql

3. Delete Apache Server

		apt remove apache2

4. Ekstrak CMS Wordpress

		unzip wordpress.zip

5. Move wordpres ke direktori /var/www/

		mv wordpress /var/www/wordpress

6. Konfigurasi Nginx

		nano /etc/nginx/sites-available/default

7. Ubah path agar Web Server mengakses direktori wordpress

		root /var/www/wordpress;

8. Tambahkan index.php 

		 index index.php index.html index.htm index.nginx-debian.html;

9. Ganti server name dengan alamat domain

		server_name example.lan;

10. Tambahkan command berikut agar semua aktivitas Wordpres tercatat

		access_log /var/log/nginx/wordpress_access.log;
        error_log /var/log/nginx/wordpress_error.log;

11. Uncomment pada bagian berikut

		location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.0.1:9000;
        }

12. Masuk ke direktori wordpress

		cd /var/www/wordpress

13. Replace file wp-config-sample.php menjadi wp-config.php

		cp  wp-config-sample.php  wp-config.php

14. Edit file  wp-config.php

		nano  wp-config.php

15. Konfigurasikan Database

		// ** MySQL settings - You can get this info from your web host ** //
		/** The name of the database for WordPress */
		define( 'DB_NAME', 'dbwp' );

		/** MySQL database username */
		define( 'DB_USER', 'dbwp' );

		/** MySQL database password */
		define( 'DB_PASSWORD', 'dbwp' );

		/** MySQL hostname */
		define( 'DB_HOST', 'localhost' );

		/** Database Charset to use in creating database tables. */
		define( 'DB_CHARSET', 'utf8' );

		/** The Database Collate type. Don't change this if in doubt. */
		define( 'DB_COLLATE', '' );
	
15. Ubah user dan group wordpress ke www-data

		chown -R www-data:www-data /var/www/html/wordpress

17. Ubah hak akses direktori wordpress

		chmod 755 -R /var/www/wordpress

18. Restart konfigurasi Web Server

		systemctl restart nginx

19. Check dengan cara mengakses domain Web Server pada Web Browser

# Setup Postfix

1. Install package Postfix

		apt install postfix

# Setup Web Mail

