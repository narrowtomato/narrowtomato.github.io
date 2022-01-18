[Home](./index.md)

# Methodology

This will always be a work in progress

---
## Recon

---
## Host Enumeration

Nmap - Identify all open ports.

First -sS on all ports, then -sV and -sC on open ports.

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

sudo -l

find / -type f -perm -u=s 2>/dev/null | xargs ls -l

cat /etc/crontab

https://gtfobins.github.io