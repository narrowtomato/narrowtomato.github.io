[Home](../index.md)

# Initial Kali Hardening

Perform basic security steps before using a new image (images found at https://www.kali.org/get-kali/).

---
## 1. Before Booting Your Image, Verify the Download Hash

```
PS C:\Users\me\Downloads> Get-FileHash .\kali-linux-2021.4-virtualbox-amd64.7z

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          EF647E7763E3F666F3A5B186F8273B7D124D3096DCEACF964C66FA7855E2E109       C:\Users\me\Downloads\kali-linux-2021.4-virtualbox-amd64.7z
```

---
## 2. Change the default password

```
┌──(kali㉿kali)-[~]
└─$ passwd
Changing password for kali.
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully
```

---
## 3. Update

```
┌─(kali㉿kali)-[~]
└─$ sudo apt update

┌─(kali㉿kali)-[~]
└─$ sudo apt full-upgrade
```

Optional, this will remove packages you don't need if disk space is a concern:

```
┌─(kali㉿kali)-[~]
└─$ sudo apt autoremove  
```

---
## 4. Get New SSH Keys (if using an image)

```
┌─(kali㉿kali)-[~]
└─$ cd /etc/ssh    
                                                                                                                                          
┌──(kali㉿kali)-[/etc/ssh]
└─$ ls    
moduli      ssh_config.d  sshd_config.d       ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
ssh_config  sshd_config   ssh_host_ecdsa_key  ssh_host_ed25519_key    ssh_host_rsa_key
                                                                                                                          
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo mkdir old_keys                                                                                                    
                                                                                                                                          
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo mv ssh_host* old_keys 
                                                                                                                                        
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo dpkg-reconfigure openssh-server
Creating SSH2 RSA key; this may take some time ...
3072 SHA256:ZfiiKts01Keknwhgdxyge06D6NRBS8yINgukB+yCbxk root@kali (RSA)
Creating SSH2 ECDSA key; this may take some time ...
256 SHA256:LodzbArquHDW9ASTpzsUWh81zLSl3tEJ59FRNly4ilY root@kali (ECDSA)
Creating SSH2 ED25519 key; this may take some time ...
256 SHA256:VCqJiiWyOMO6DFykmd3ILnQ8qw0BQwGZU9Py69TQqG4 root@kali (ED25519)
rescue-ssh.target is a disabled or a static unit not running, not starting it.
```
Verify the keys are different:
```
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo md5sum ssh_host_*                                                                                                     
2ed42a6424a44a02947296815c0ea3d2  ssh_host_ecdsa_key
c136c3bf1b480b45346302935d3a7323  ssh_host_ecdsa_key.pub
f237162520e2f997815072e85a4c6389  ssh_host_ed25519_key
4dd7935e4e370a69bb9d3f084dc1e597  ssh_host_ed25519_key.pub
32646b2c8ccb384f0ccf056f7ff44d66  ssh_host_rsa_key
65ee24c7c3b9643ef2aabfc7816fc769  ssh_host_rsa_key.pub
                                                                                                                                          
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo md5sum old_keys/ssh_host_*
5cd779953b0c7fd8ab408fee63093f1f  old_keys/ssh_host_ecdsa_key
06956b6a384a0afe51e4811857520424  old_keys/ssh_host_ecdsa_key.pub
dbaaf5d1e3f0c0dd4f013066729f3d46  old_keys/ssh_host_ed25519_key
500b601c9648fe55b9d0f4054cf5809c  old_keys/ssh_host_ed25519_key.pub
ca85a8ced95e3c1e905b4885b7e93a88  old_keys/ssh_host_rsa_key
9924e22e19960aaf1d610308db0e3a50  old_keys/ssh_host_rsa_key.pub
                 
```

IF you would like to remove inbound SSH entirely:

```
$ sudo apt remove openssh-server
```

---
## 5. Check for Rootkits, etc... (should probably be done at every boot)

```
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo apt install chkrootkit

┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo chkrootkit
```

This was ripped off/consolidated from the below 2 articles, all credit goes there.

* https://linuxconfig.org/hardening-kali-linux
* https://alphacybersecurity.tech/how-to-secure-your-kali-linux-machine/