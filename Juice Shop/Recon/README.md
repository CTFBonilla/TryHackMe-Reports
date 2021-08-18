nmap -sV -sC -T5 -vv -oN nmap_initial $IP

	80/tcp open  http    syn-ack ttl 63
	| fingerprint-strings: 
	|   FourOhFourRequest, GetRequest: 
	|     HTTP/1.1 200 OK
	|     Access-Control-Allow-Origin: *
	|     X-Content-Type-Options: nosniff
	|     X-Frame-Options: SAMEORIGIN
	|     Feature-Policy: payment 'self'
	|     Accept-Ranges: bytes
	|     Cache-Control: public, max-age=0
	|     Last-Modified: Wed, 18 Aug 2021 08:52:20 GMT
	|     ETag: W/"786-17b58761d34"
	|     Content-Type: text/html; charset=UTF-8
	|     Content-Length: 1926
	|     Vary: Accept-Encoding
	|     Date: Wed, 18 Aug 2021 09:49:32 GMT
	|     Connection: close
	|     <!--
	|     Copyright (c) 2014-2020 Bjoern Kimminich.
	|     SPDX-License-Identifier: MIT
	|     <!doctype html>
	|     <html lang="en">
	|     <head>
	|     <meta charset="utf-8">
	|     <title>OWASP Juice Shop</title>
	|     <meta name="description" content="Probably the most modern and sophisticated insecure web application">
	|     <meta name="viewport" content="width=device-width, initial-scale=1">
	|     <link id="favicon" rel="icon" type="image/x-icon" href="assets/public/favicon_ctf.ico">
	|     <link rel="stylesheet" typ
	|   HTTPOptions: 
	|     HTTP/1.1 204 No Content
	|     Access-Control-Allow-Origin: *
	|     Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE
	|     Vary: Access-Control-Request-Headers
	|     Content-Length: 0
	|     Date: Wed, 18 Aug 2021 09:49:32 GMT
	|     Connection: close
	|   RTSPRequest, X11Probe: 
	|     HTTP/1.1 400 Bad Request
	|_    Connection: close
	|_http-cors: HEAD GET POST PUT DELETE PATCH
	|_http-favicon: Unknown favicon MD5: 50D0BF5D28E28C2661A4E48860D0F6E0
	| http-methods: 
	|_  Supported Methods: GET HEAD POST OPTIONS
	| http-robots.txt: 1 disallowed entry 
	|_/ftp
	|_http-title: OWASP Juice Shop
	1 service unrecognized despite returning data.


wappalyzer

	JavaScript frameworks: Zone.js | Angular 9.1.9

	Programming languages: TypeScript

	Font scripts: Font Awesome | Google Font API

	JavaScript libraries: jQuery 2.2.4 | Hammer.js 2.0.7

	UI frameworks: Material Design Lite 1.3.0

	Miscellaneous: webpack | Gravatar

	Cookie compliance: Osano


gobuster dir -u http://$IP/ -w directory-list-2.3-medium.txt -o gobuster_$IP


	Error: the server returns a status code that matches the provided options for non existing urls. http://$IP/c6cd4309-b90b-4fa6-be93-6de7f49dae89 => 200. To force processing of Wildcard responses, specify the '--wildcard' switch

rude

gobuster dir -u http://$IP/ -w directory-list-2.3-medium.txt -s "204,301,302,307,401,403" -o gobuster_$IP

	Taking far too long, let's just walk through the application


## Walking Through the App ##

### Looking through reviews reveals several emails ###


	admin@juice-sh.op

	bender@juice-sh.op
		likes Futurama

	uvogin@juice-sh.op

	jim@juice-sh.op
		likes Star Trek

	mc.safesearch@juice-sh.op

	morty@juice-sh.op
		likes Rick & Morty



### Search function ###

	Search Parameter in URL
		/#/search?q=[search input]


### Customer Feedback ###

/#/contact

	Author
	Comment (max 160 chars)
	Rating (1-5 stars)
	CAPTCHA: [math problem]
	Submit
	Returns "Thank you for your feedback"


### About Us ###

/#/about

	lorem ipsum and reviews displayed alongside images

	link to terms of use: /ftp/legal.md
		lorem ipsum


### Photo Wall ###

/#/photo-wall
	
	two photos of merch


### Account Login ###

/#/login

	email
	pass
	forgot pass
	remember me
	submit
	link to registration

> Inputting "test:test"
	
	Returns "Invalid email or password"


### User Registration ###

/#/register
	
	email
	pass
	repeat pass

	toggle show password advice
		
		contains at least one lower character

		contains at least one upper character

		contains at least one digit

		contains at least one special character

		contains at least 8 characters

	security question
	answer
	register
	link to login


### Forgot Password ###

/#/forgot-password
	
	email
	security question
	new password
	repeat password
	toggle show password advice
	change


-------------------------------------
	Making Account and Logging In
-------------------------------------

Credentials: 

> test@mail.com | Passw0rd1*

> Security Question: Eldest sibling name? | test 


### User Profile ###

/#/profile

	Email:
	Username:
	File Upload
		"Upload Picture" OR "Image URL"


### Order History ###

/#/order-history

	nothing here yet, assuming this is where orders go 


### Recycle ###

/#/recycle

	Requestor: email
	Quantity: 10-1000 liters
	Addresses: "My Saved Addresses"
		checkbox to pickup at above address


### Addresses ###

/#/address/saved
	
	Add New Address

	/#/address/create
		Country
		Name
		Mobile Number
		ZIP code
		Address
		City
		State
		Back or Submit

	Edit and delete buttons appear once address is added


### Payment Methods ###

/#/saved-payment-methods

	"Add a credit or debit card" dropdown
		Name
		Card Number
		Expiry Month
		Expiry Year
		Submit

	Card is censored aside from last 4 digits
	Delete button appears once card is added


### Digital Wallet ###

/#/wallet

	Add Money
	Amount input (10 to 1000)


### Privacy Policy ###

/#/privacy-security/privacy-policy


### Request Data Export ###

/#/privacy-security/data-export

	Export Format: JSON | PDF | Excel
	Request

### Data Erasure Request ###

/#/privacy-security/erasure-request
	
	Request Data Erasure
		input Email
	Delete User Data button


### Change Password ###

/#/privacy-security/change-password
	
	Current Password
	New Password
	Repeat New Password
	Change

### 2FA Configuration ###

/#/privacy-security/two-factor-authentication
	
	QR Code Scan
	Current Password
	Initial Token
	Save

### Last Login IP ###

/#/privacy-security/last-login-ip
	
	IP Address [IP]


### Complaint ###

/#/complain
	
	Customer email
	Message
	!Invoice File Upload
	Submit

### Deluxe Membership ###

/#/deluxe-membership
	
	Become a member

	redirects to /#/payment/deluxe

### Basket ###

Items now have "Add to Basket" option

Clicking cart redirects to /#/basket
	
	Contains added items, quantity, and delete option
	Total Price
	Checkout button 
	"Points gained from this purchase"

		> Redirect to /#/address/select
			Radio select from added addresses
			Continue button 

				>> Redirect to /#/delivery-method
					Choose delivery method 
						1 day, Fast (3 days), Standard (5 days)
					Continue button

						>>> Redirect to /#/payment/shop
							Radio select from added cards
							Pay
							Add coupon
							Other payment methods
							Continue button

								>>>> Redirect to /#/order-summary
									"Place your order and pay" button



## Dev Tools Debugger ##

Find "path: '"

	administration

	accounting

	about

	address/select

	address/saved

	address/create

	address/edit/:addressId

	delivery-method

	deluxe-membership

	saved-payment-methods

	basket

	order-completeion/:id

	contact

	photo-wall

	complain

	order-summary

	order-history

	paymet/:entity

	wallet

	login

	forgot-password

	recycle

	register

	hacking-instructor

	score-board

	track-result

	track-result/new

	2fa/enter

	privacy-security

	privacy-policy

	change-password

	two-factor-authentication

	data-export

	erasure-request

	last-login-ip

	403

	**


The adventure continues in the individual challenges...
