# Gotta Catch'em All!
<a href="https://tryhackme.com/room/pokemon" rel="nofollow">Click here to see the room!</a>

First of all, connect to the VPN and deploy the machine.

The machine IP is 10.10.30.60 for me. (It will be different for you)

Run Nmap to search open ports

```nmap -sC -sV -Pn 10.10.30.60```

```[root@hsn]─[/home/hsn]
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
Nmap done: 1 IP address (1 host up) scanned in 11.11 seconds```

SSH and HTTP ports are open! Let's check ```http://10.10.30.60``` on our browser.

We see Apache2 server welcome page. Let's inspect this page! Right click on the page and click "View Page Source". Check codes carefully!

You can see something different that is shouldn't be on a Apache2 server welcome page. Of course, it was modified!

I found these JavaScript codes on bottom of the head tag:

```<script type="text/javascript">
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


