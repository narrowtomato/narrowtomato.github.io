[Home](../index.md)

# SSH Tunneling By Use Case

**Credit:** Most of this has been paraphrased from the [Wreath network on TryHackMe](https://tryhackme.com/room/wreath).  I wrote this article to reinforce my own knowlege as I was struggling with this subject.  The room is created by [Muirlandoracle](https://muirlandoracle.co.uk).

SSH Tunneling can be used to pivot from a partially or completely compromised machine to the rest of the network.  This is especially useful when the rest of the network is either firewalled or not routable from the attacker's standpoint.  This guide is organized by situation.

### For the purposes of this guide:

The IP of the attacking machine will always be listed as `77.77.77.77`

The IP of the compromised machine will always be listed as `100.100.100.111`

The IP of the target machine will always be listed as `100.100.100.222`

---
## We have compromised `100.100.100.111`, and wish to use it to enumerate additional services on `100.100.100.222`, which we cannot enumerate from `77.77.77.77`.

It is assumed that we have probably already run host discovery from the compromised machine and have discovered `100.100.100.222`, but do not have sufficient tools to enumerate `100.100.100.222` further.

First, we set up a tunnel

```
$ ssh -D 9999 user@100.100.100.111 -N
```

`-D` sets up a proxy on our local machine.  Now any time we direct traffic to `77.77.77.77:9999` (or `127.0.0.1:9999` if we're running commands from that machine), it will tunnel over SSH, exiting the tunnel on `100.100.100.111` inside of the desired network.  If we do not use `-N`, it will expect a command to run.  You can also use `-f` to send this to the background if you wish to keep using the terminal instance you run this in (I usually use `tmux` so I like to keep things running in their own terminal).

Now all we need to do is use proxychains to direct traffic appropriately and we can run programs such as `nmap` against the target network.  Example proxychains configuration below.

```
[ProxyList]
socks4 127.0.0.1 9999
```

Then an example command

```
$ proxychains nmap -p- 100.100.100.222
```

Note that port scans can be extremely slow through proxychains.  Finding a way to run the scan directly from `100.100.100.111` may be a better solution (static binary?).

---
## We have compromised `100.100.100.111`, and know of a specific service on `100.100.100.222` that we cannot access from `77.77.77.77`.

Since we know exactly what service we want to hit, we don't have to use proxychains for this.  We set up a tunnel:

```
$ ssh -L 9999:100.100.100.222:80 user@100.100.100.111 -N
```

`-L` opens a local port (on `77.77.77.77` or `127.0.0.1`) that we can use to access port 80 (or whatever you want) on `100.100.100.222`.  The traffic is tunneled through SSH, exits on `100.100.100.111`, and continues to port 80 on `100.100.100.222`.

After running the SSH command, here are some examples of interacting with port 80 on `100.100.100.222`:

```
$ nmap -sC -p 9999 127.0.0.1

$ curl http://127.0.0.1:9999/
```

The point is that we interact with port 9999 on our local machine to go through the tunnel.

---
## You get a shell on `100.100.100.111` but do not have full SSH access to create a tunnel

In this case we cannot create a tunnel as before, because we have no SSH access to the machine.  What we CAN do is create a tunnel back to our attacking machine and forward the port that way.

First, on the attacking machine, we create a new set of SSH keys

```
$ ssh-keygen
```

Next, we need to add our generated .pub file to `~/.ssh/authorized_keys`

**IF** we want to make sure the operators of `100.100.100.111` won't be able to SSH back to our attacking machine, we need to paste the contents of .pub with the below information prepended:

```
command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-x11-forwarding,no-pty <YOUR_PUB_CONTENTS_PASTED_HERE>
```

Now we make sure SSH is running on `77.77.77.77`

```
$ sudo systemctl status ssh

... (if needed) ...

$ sudo systemctl start ssh
```

Now, transfer your private key file to `100.100.100.111`

Then from our shell on `100.100.100.111`:

```
ssh -R 9999:100.100.100.222:80 user@77.77.77.77 -i NAME_OF_KEYFILE -N
```

`-R` works like `-L`, except it attempts to open the port on the remote server (our attacking machine, `77.77.77.77`)

You can access the port the same way as we did with `-L`

```
$ nmap -sC -p 9999 127.0.0.1

$ curl http://127.0.0.1:9999/
```