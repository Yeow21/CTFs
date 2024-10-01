nmap scan:
```
nmap -T4 -A -p- 192.168.144.135
```
UDP scan
```
nmap -T4 -A -p <open ports> 192.168.144.135
```
Notable findings:
ports 80 and 8080 open;
apache 2.4.38 on both.
	80 - Bolt installation error. What's this?
	8080 - script returns potential open proxy 
	PHP 7.3.27-1~deb10u1 - phpinfo()
OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)

Visit to http://192.168.144.135:
	bolt installation error page - nothing particularly interesting.
visit to 192.168.144.135:8080
	php config file:
	OS: 	Linux dev 4.19.0-16-amd64
	User/group: www-data(33)/33 
	server root: /etc/apache2 
	DOCUMENT_ROOT 	/var/www/htdev 
	REMOTE_PORT 	43904 
	curl:
		OpenSSL/1.1.1d 

Nessus scan:
	CRITICAL 10.0* 5.9 11356 **NFS Exported Share Information Disclosure**

dirbuster
	http://192.168.144.135/app/cache/config-cache.json
***	username	"bolt"
	password	"I_love_java"***
	user	"bolt"
	wrapperClass	"Bolt\\Storage\\Database\\Connection"
	path	"/var/www/html/app/database/bolt.db"

	http://192.168.144.135:8080/dev/pages/member.thisisatest
	~data~
	password: thisisatest
	~

http://192.168.144.135:8080/dev/pages/member.admin
	~data~
	password: I_love_java
	~