# Gotta Catch'em All!
<a href="https://tryhackme.com/room/pokemon" rel="nofollow">Click here to see the room!</a>

First of all, connect to the VPN and deploy the machine.

The machine IP is 10.10.30.60 for me. (It will be different for you)

Run Nmap to search open ports

```nmap -sC -sV -Pn 10.10.30.60```

The result is:

```
[root@hsn]─[/home/hsn]
└──╼ #nmap -sC -sV -Pn 10.10.30.60
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-21 06:48 BST
Nmap scan report for 10.10.30.60
Host is up (0.19s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
|   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
|_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Can You Find Them All?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.11 seconds
```

SSH and HTTP ports are open!

It's better to check directories. Run gobuster!

```
┌─[✗]─[root@hsn]─[/home/hsn]
└──╼ #gobuster dir -u http://10.10.30.60 -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.30.60
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/21 05:40:35 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/server-status (Status: 403)
===============================================================
2020/08/21 05:43:10 Finished
===============================================================

```
Nothing found :(

Let's check http://10.10.30.60 on our browser.

We see Apache2 server welcome page. Let's inspect this page! Right click on the page and click "View Page Source". Check codes carefully!

You can see something different that is shouldn't be on a Apache2 server welcome page. Of course, it was modified!

I found these JavaScript codes on bottom of the head tag:

```
<script type="text/javascript">
    	const randomPokemon = [
    		'Bulbasaur', 'Charmander', 'Squirtle',
    		'Snorlax',
    		'Zapdos',
    		'Mew',
    		'Charizard',
    		'Grimer',
    		'Metapod',
    		'Magikarp'
    	];
    	const original = randomPokemon.sort((pokemonName) => {
    		const [aLast] = pokemonName.split(', ');
    	});
    	console.log(original);
    </script>
```
and
```
<pokemon>:<hack_the_pokemon>
        	<!--(Check console for extra surprise!)-->
```
Its pattern looks like a username:password combination. Let's check it and try to connect SSH.

```
┌─[✗]─[root@hsn]─[/home/hsn/Desktop/ctf]
└──╼ #ssh pokemon@10.10.30.60
pokemon@10.10.30.60's password: hack_the_pokemon
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

84 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

pokemon@root:~$
```
Yes! We are in! Check the directories.

```
pokemon@root:~$ ls
Desktop    Downloads         Music     Public     Videos
Documents  examples.desktop  Pictures  Templates

pokemon@root:~$ cd Desktop/
pokemon@root:~/Desktop$ ls
P0kEmOn.zip
```
Unzip this file.

```
pokemon@root:~/Desktop$ unzip P0kEmOn.zip 
Archive:  P0kEmOn.zip
   creating: P0kEmOn/
  inflating: P0kEmOn/grass-type.txt  
pokemon@root:~/Desktop$ ls
P0kEmOn  P0kEmOn.zip

pokemon@root:~/Desktop$ cd P0kEmOn/
pokemon@root:~/Desktop/P0kEmOn$ ls -la
total 12
drwxrwxr-x 2 pokemon pokemon 4096 Jun 22 22:37 .
drwxr-xr-x 3 pokemon pokemon 4096 Aug 21 01:23 ..
-rw-rw-r-- 1 pokemon pokemon   53 Jun 22 22:37 grass-type.txt
pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt 
50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d
```
Looks like our first flag but hex coded. Let's decode it!
Google it "hexadecimal to text" and grab the "#1 Find the Grass-Type Pokemon" flag!

Keep checking the other directories!
```
pokemon@root:~$ cd Music/
pokemon@root:~/Music$ ls
pokemon@root:~/Music$ cd ..
pokemon@root:~$ cd Public/
pokemon@root:~/Public$ ls
pokemon@root:~/Public$ cd ..
pokemon@root:~$ cd Videos/
pokemon@root:~/Videos$ ls
Gotta
pokemon@root:~/Videos$ cd Gotta/
pokemon@root:~/Videos/Gotta$ ls
Catch
pokemon@root:~/Videos/Gotta$ cd Catch/
pokemon@root:~/Videos/Gotta/Catch$ ls
Them
pokemon@root:~/Videos/Gotta/Catch$ cd Them/
pokemon@root:~/Videos/Gotta/Catch/Them$ ls
ALL!
pokemon@root:~/Videos/Gotta/Catch/Them$ cd ALL!
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ ls
Could_this_be_what_Im_looking_for?.cplusplus
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ cat Could_this_be_what_Im_looking_for\?.cplusplus 
# include <iostream>

int main() {
	std::cout << "ash : pikapika"
	return 0;
}
```
Another username:password is found in the "Could_this_be_what_Im_looking_for?.cplusplus" file.

Let's try it on SSH again!

```
┌─[root@hsn]─[/home/hsn]
└──╼ #ssh ash@10.10.30.60
ash@10.10.30.60's password: pikapika
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

84 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Could not chdir to home directory /home/ash: Permission denied

$ whoami
ash
```
Connected! Let's check the directories.

```
$ ls
bin    dev   initrd.img      lib64	 mnt   root  snap  tmp	vmlinuz
boot   etc   initrd.img.old  lost+found  opt   run   srv   usr	vmlinuz.old
cdrom  home  lib	     media	 proc  sbin  sys   var
$ cd home
$ ls
ash  pokemon  roots-pokemon.txt
$ cat roots-pokemon.txt
Pikachu!
```
Yes! We found "#4 Who is Root's Favorite Pokemon?" flag!

Let's check that ash has root privileges!

```
ash@root:/home/pokemon$ sudo -l
[sudo] password for ash: 
Matching Defaults entries for ash on root:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ash may run the following commands on root:
    (ALL : ALL) ALL
```
The answer is yes!

```
ash@root:/home/pokemon$ sudo su
```
Being root!

```
root@root:/home/pokemon# ls
Desktop    Downloads         Music     Public     Videos
Documents  examples.desktop  Pictures  Templates
```
Try to find other flags!
```
root@root:/home# locate water-type
/var/www/html/water-type.txt
root@root:/home# cat /var/www/html/water-type.txt
Ecgudfxq_EcGmP{Ecgudfxq}
```

We found "#2 Find the Water-Type Pokemon" flag! But encrypted. Looks like ROT13 encryption. When I google it, I found this website: https://rot13.com/
We can check all ROTs from ROT1 to ROT25. At the ROT14, the text is being meaningful!

```
root@root:/home# locate fire-type
/etc/why_am_i_here?/fire-type.txt
root@root:/home# cat /etc/why_am_i_here?/fire-type.txt
UDBrM20wbntDaGFybWFuZGVyfQ==
```

We found "#3 Find the Fire-Type Pokemon" flag! But encrypted again. Now it looks like Base64 encryption. On this https://www.base64decode.org/ website you can decode it. And of course Charmander was my favorite pokemon :)

Many thanks to creator <a href="https://tryhackme.com/p/GhostlyPy" rel="nofollow">GhostlyPy</a> and <a href="https://tryhackme.com" rel="nofollow">TryHackMe.com</a> for this free room.

PS: This is my first write-up. If you think something is wrong, send a mail to hsnckkgl@gmail.com

Best wishes all the CTF Players,

hsnckkgl
