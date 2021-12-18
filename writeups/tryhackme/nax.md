[Home](../../index.md)

# TryHackMe: Nax

https://tryhackme.com/room/nax

---
## Enumeration and Foothold

```
┌──(kali㉿kali)-[~/tryhackme/nax]
└─$ rustscan -a 10.10.156.183 -r 1-65535 -- -sV

PORT     STATE SERVICE    REASON  VERSION
22/tcp   open  ssh        syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
25/tcp   open  smtp       syn-ack Postfix smtpd
80/tcp   open  http       syn-ack Apache httpd 2.4.18 ((Ubuntu))
389/tcp  open  ldap       syn-ack OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   syn-ack Apache httpd 2.4.18 ((Ubuntu))
5667/tcp open  tcpwrapped syn-ack
Service Info: Host:  ubuntu.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I thought the 'tcpwrapped' service was interesting, so I found this as a good explanation at https://security.stackexchange.com/questions/23407/how-to-bypass-tcpwrapped-with-nmap-scan

>"tcpwrapped" refers to tcpwrapper, a host-based network access control program on Unix and Linux. When Nmap labels something tcpwrapped, it means that the behavior of the port is consistent with one that is protected by tcpwrapper. Specifically, it means that a full TCP handshake was completed, but the remote host closed the connection without receiving any data.
>
>It is important to note that tcpwrapper protects programs, not ports. This means that a valid (not false-positive) tcpwrapped response indicates a real network service is available, but you are not on the list of hosts allowed to talk with it. When such a large number of ports are shown as tcpwrapped, it is unlikely that they represent real services, so the behavior probably means something else.
>
>What you are probably seeing is a network security device like a firewall or IPS. Many of these are configured to respond to TCP portscans, even for IP addresses which are not assigned to them. This behavior can slow down a port scan and cloud the results with false positives.

Attempted to enumerate LDAP based on this article, but wasn't able to find anything https://book.hacktricks.xyz/pentesting/pentesting-ldap

I was able to confirm 'nagios' as a username using SMTP and VRFY (credit goes here: https://book.hacktricks.xyz/pentesting/pentesting-smtp)

```
┌──(kali㉿kali)-[~/tryhackme/nax]
└─$ nmap -p25 --script smtp-commands 10.10.156.183      
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-17 15:48 EST
Nmap scan report for 10.10.156.183
Host is up (0.21s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-commands: ubuntu.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN

Nmap done: 1 IP address (1 host up) scanned in 1.42 seconds
                                                                   
┌──(kali㉿kali)-[~/tryhackme/nax]
└─$ telnet 10.10.156.183 25
Trying 10.10.156.183...
Connected to 10.10.156.183.
Escape character is '^]'.
220 ubuntu.localdomain ESMTP Postfix (Ubuntu)
HELO
501 Syntax: HELO hostname
VRFY root
252 2.0.0 root
VRFY nagios
252 2.0.0 nagios
VRFY blah
550 5.1.1 <blah>: Recipient address rejected: User unknown in local recipient table
VRFY nagiosxi
550 5.1.1 <nagiosxi>: Recipient address rejected: User unknown in local recipient table
VRFY user
550 5.1.1 <user>: Recipient address rejected: User unknown in local recipient table
```
This led me to briefly try to brute SSH with Hydra, but I quickly got bored and gave up.
```
┌──(kali㉿kali)-[~/tryhackme/nax]                                                    
└─$ hydra -l nagios -P /usr/share/wordlists/rockyou.txt 10.10.156.183 ssh 
```

`http://10.10.156.183/` or `https://10.10.156.183/` shows this:
```
		     ,+++77777++=:,                    +=                      ,,++=7++=,,
		    7~?7   +7I77 :,I777  I          77 7+77 7:        ,?777777??~,=+=~I7?,=77 I
		=7I7I~7  ,77: ++:~+777777 7     +77=7 =7I7     ,I777= 77,:~7 +?7, ~7   ~ 777?
		77+7I 777~,,=7~  ,::7=7: 7 77   77: 7 7 +77,7 I777~+777I=   =:,77,77  77 7,777,
		  = 7  ?7 , 7~,~  + 77 ?: :?777 +~77 77? I7777I7I7 777+77   =:, ?7   +7 777?
		      77 ~I == ~77=77777~: I,+77?  7  7:?7? ?7 7 7 77 ~I   7I,,?7 I77~
		       I 7=77~+77+?=:I+~77?     , I 7? 77 7   777~ +7 I+?7  +7~?777,77I
		         =77 77= +7 7777         ,7 7?7:,??7     +7    7   77??+ 7777,
		             =I, I 7+:77?         +7I7?7777 :             :7 7
		                7I7I?77 ~         +7:77,     ~         +7,::7   7
		               ,7~77?7? ?:         7+:77           77 :7777=
		                ?77 +I7+,7         7~  7,+7  ,?       ?7?~?777:
		                   I777=7777 ~     77 :  77 =7+,    I77  777
		                     +      ~?     , + 7    ,, ~I,  = ? ,
		                                    77:I+
		                                    ,7
		                                     :777
		                                        :
						Welcome to elements.
					Ag - Hg - Ta - Sb - Po - Pd - Hg - Pt - Lr
```

I looked these up on the periodic table and got the atomic numbers

* 47 Silver
* 80 Mercury
* 73 Tantalum
* 51 Antimony
* 84 Polonium
* 46 Palladium
* 80 Mercury
* 78 Platinum
* 103 Lawrencium

These numbers translate to the below when treated as ASCII

```
/PI3T.PNg
```

Looks like a filename.  Adding this to the path in our browser gets us a very interesting image.

![PI3T.PNG](./img/PI3T.PNG "PI3T.PNG")

When I saw this I immediately thought of a YouTube video I watched a while back - [A Brief Introduction to Esoteric Programming Languages](https://youtu.be/cQ7bcCrJMHc)

The language we are looking at here is [Piet](https://www.dangermouse.net/esoteric/piet.html)

You can find an interpreter [Here](https://www.bertnase.de/npiet/)

```
┌──(kali㉿kali)-[~/tryhackme/nax]                                                      
└─$ wget https://www.bertnase.de/npiet/npiet-1.3f.c 

┌──(kali㉿kali)-[~/tryhackme/nax]  
└─$ gcc npiet-1.3f.c -o npiet 

──(kali㉿kali)-[~/tryhackme/nax]   
└─$ wget http://10.10.156.183/PI3T.PNg

┌──(kali㉿kali)-[~/tryhackme/nax]                                        
└─$ ./npiet PI3T.PNg                       
cannot read from `PI3T.PNg'; reason: unknown PPM format
```

Here I had to download Gimp and export as a PPM as instructed in the room.

```
┌──(kali㉿kali)-[~/tryhackme/nax]                                                              
└─$ ./npiet PI3T.ppm     
nagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9BjLp4$7uhWdYnagiosadmin%n3p3UQ&9B (and so on, since the program is written in a square it is an infinite loop)
```

Viewing the source code of the root page references a directory:

```
<! --/nagiosxi/ --> 
```

Browsing to this directory gets us access to NagiosXI and the respective login.

---
## Exploitation

Now that we can log in, we can find the version.  You can find it on the bottom left of the page:  Nagios XI 5.5.6

Googling for an exploit quickly reveals a CVE and [exploit](https://www.exploit-db.com/exploits/48191), which fortunately has a Metasploit module

I used linux/http/nagios_xi_authenticated_rce with meterpreter, but based on the room questions they want you to use exploit/linux/http/nagios_xi_plugins_check_plugin_authenticated_rce either way it should work as long as you set the PASSWORD, RHOSTS and LHOSTS options correctly.

---
## User Flag

Looking around at the usual locations, user.txt can be found in the only accessible /home directory.

---
## Root Flag

We are already root, so grabbing the flag out of /root is trivial.