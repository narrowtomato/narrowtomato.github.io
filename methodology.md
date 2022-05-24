[Home](./index.md)

# Methodology

This will always be a work in progress

REMEMBER: If you find something, mess with it.  Poke at it.  Connect to the server.  Try to log into the form.  Don't just say "IDK WHAT THIS IS IMA GIVE UP NOW".  Too many times we have resorted to looking at writeups when we could have just tried to mess with the thing and we could have gotten more information.

---
## Recon

---
## Host Enumeration

Nmap - Identify all open ports.

`sudo nmap -v -p- --min-rate 5000 -sV -sC <IP>`

First `-sS` on all ports, then `-sV` and `-sC` on open ports.

1. Check versions of all services for known vulnerabilities.

2. Check each service in turn for misconfiguration.

Run `enum4linux`

---
## Web Enumeration

`gobuster`, `feroxbuster` - find files and directories

`nikto`

Review source code for further information.

Wappalyzer Extension to identify versions - check each version for known vulnerabilities.

Look for specific tools to specialize in CMS (for example `wpscan` for Wordpress)

Use BurpSuite to intercept traffic to see where things are actually coming from.  This may reveal directories/files you couldn't see before.

---
## Login Forms - SQL Injection

`admin'#`   (comments out the rest of the code)

`' or 1=1--`

---
## SMB

`smbclient -L IP -N`  checks for shares, without auth

`smbclient //IP/SHARE -N` logs in, without auth

---
## RDP

```
nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 -T4 <IP>
```

Log in without auth: `xfreerdp /v:<IP> /cert:ignore /u:Administrator`


---
## Databases

### Mysql

Use `mysql` to log in, 


### Redis 

https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis

Usually port 6379

`nc -vn 10.10.10.10 6379`

`info`

look for `# Keyspace`, this will show databases

`SELECT 0` (where 0 is the database number)

`KEYS *` (lists keys)

`GET <KEY>`


---
## Suspicious Image

`exiftool`, `strings`, `steghide`

---
## Privilege Escalation - Linux

If possible - run LinPEAS or other

`sudo -l`

`find / -type f -perm -u=s 2>/dev/null | xargs ls -l`

`find -type f / -user YOURUSERHERE 2>/dev/null | xargs ls -l`

`cat /etc/crontab`

`ls /etc/sudoers.d`
(`cat` what's there)

`getcap -r / 2>/dev/null`

`ss -tl` shows what's listening internally

`ps aux`, look for interesting processes

`sudo -V`, if less than 1.8.28, `sudo -u#-1 COMMAND`, where COMMAND is something you have permissions to run as another user but not root.

`groups`, if in sudo group, use pkexec (see road.md)

Am I in a docker container?  Check `/proc/self/cgroup`

https://gtfobins.github.io

