[Home](./index.md)

# Methodology

This will always be a work in progress

REMEMBER: If you find something, mess with it.  Poke at it.  Connect to the server.  Try to log into the form.  Don't just say "IDK WHAT THIS IS IMA GIVE UP NOW".  Too many times we have resorted to looking at writeups when we could have just tried to mess with the thing and we could have gotten more information.

---
## Recon

---
## Host Enumeration

Nmap - Identify all open ports.

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

https://gtfobins.github.io

