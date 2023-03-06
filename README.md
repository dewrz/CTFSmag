# CTF: Smag Grotto – Boot to root
Web & Linux Hack With Privilege Escalation

The first thing I did was enumerate the network by scanning via NMAP. I also opened a web browser while scanning to check to see if there is a web server present. There is, so I look at the page source and run Nikto while the NMAP scan finishes. 

There is nothing interesting in the page source, nor is there anything to note from the Nikto scan. The NMAP scan produced ports 22 & 80 open, I then scanned the high ports as well, because CTF creators love to hide open services there to deceive the attackers, but there was nothing to note. 

<b>NMAP Results:</b>

<i>PORT   STATE SERVICE VERSION 

22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; 	protocol 2.0) 

| ssh-hostkey:  

|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA) 

|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA) 

|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (EdDSA) 

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu)) 

|_http-server-header: Apache/2.4.18 (Ubuntu) 

|_http-title: Smag 

MAC Address: 02:49:FD:C1:DE:49 (Unknown) 

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel</i>

I will also run a directory scan to continue the enumeration and after that scan is complete, we have a “/mail” directory. I will run a further scan on that directory to see if there are any sub-directories, but there is nothing special there. 

<b>Gobuster Results:</b>

<i>root@ip-10-10-120-3:/usr/share/wordlists/dirbuster# gobuster dir -u 	h	ttp://smag.thm -w directory-list-lowercase-2.3-medium.txt -x txt,pgp,bak 

=============================================================	== 

Gobuster v3.0.1 

by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_) 

=============================================================	== 

[+] Url:            http://smag.thm 

[+] Threads:        10 

[+] Wordlist:       directory-list-lowercase-2.3-medium.txt 

[+] Status codes:   200,204,301,302,307,401,403 

[+] User Agent:     gobuster/3.0.1 

[+] Extensions:     bak,txt,pgp 

[+] Timeout:        10s 

=============================================================	== 

2023/03/04 15:43:59 Starting gobuster 

=============================================================	== 

/mail (Status: 301) 

/server-status (Status: 403)</i>

When looking at the “/mail” directory we can see there is a pcap file and some user email addresses. 
<br>
<br>
<a href="https://imgur.com/5P7ZCfx"><img src="https://i.imgur.com/5P7ZCfx.jpg" title="source: imgur.com" /></a>
<br>
<br>
They also describe a bug in the email2web software. I downloaded the pcap file and after analyzing the information I was able to find a username and password for “development.smag.thm”. I also took the information and passed it to the Burp Suite repeater, and we received a 200 ok response. I next add the new address to our /etc/hosts file for further enumeration.  
<br>
<br>
<a href="https://imgur.com/5lwvwzl"><img src="https://i.imgur.com/5lwvwzl.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/16PWvB9"><img src="https://i.imgur.com/16PWvB9.jpg" title="source: imgur.com" /></a>
<br>
<br>
After logging in with the credentials we’ve found, it brings you to a page with a command injection form. After testing it we discover it is a blind injection, so we will set up a listener and try to pass a shell command. 
<br>
<br>
<a href="https://imgur.com/qIsv3oI"><img src="https://i.imgur.com/qIsv3oI.jpg" title="source: imgur.com" /></a>
<br>
<br>
After sending the command we received a shell on the web server. 
<br>
<br>
<a href="https://imgur.com/va5kLy2"><img src="https://i.imgur.com/va5kLy2.jpg" title="source: imgur.com" /></a>
<br>
<br>
I spent some time searching around for configuration mismanagement so I could elevate my privileges and I came across a cronjob that can be exploited. 
<br>
<br>
<a href="https://imgur.com/wgVwiFt"><img src="https://i.imgur.com/wgVwiFt.jpg" title="source: imgur.com" /></a>
<br>
<br>
I create a public key on my attack box and then copy it to jakes backup public key, this will give us the ability to log into jakes account via ssh. 
<br>
<br>
<a href="https://imgur.com/dT7K6Qn"><img src="https://i.imgur.com/dT7K6Qn.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/Kr8iYkE"><img src="https://i.imgur.com/Kr8iYkE.jpg" title="source: imgur.com" /></a>
<br>
<br>
After logging in via SSH, we were able to retrieve the first user flag. 
<br>
<br>
<a href="https://imgur.com/PSknXeL"><img src="https://i.imgur.com/PSknXeL.jpg" title="source: imgur.com" /></a>
<br>
<br>
Now we need to escalate our privileges to root. I ran the command “sudo -l” which lets us know that we can take advantage of the binary “apt-get”. 
<br>
<br>
<a href="https://imgur.com/kTbENI7"><img src="https://i.imgur.com/kTbENI7.jpg" title="source: imgur.com" /></a>
<br>
<br>
By running “sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh” we can obtain root privileges and retrieve the root flag. 
<br>
<br>
<a href="https://imgur.com/rmCw9z2"><img src="https://i.imgur.com/rmCw9z2.jpg" title="source: imgur.com" /></a>

