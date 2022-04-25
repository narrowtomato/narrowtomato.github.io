[Home](../../index.md)

# UnderTheWire Groot Writeup

[Groot](https://underthewire.tech/groot) is the next wargame at Under the Wire after Cyborg. If you haven't completed Century and Cyborg, you should do so first. As stated on the Century writeup, you will get the most out of this if you try your best and search the internet before looking for the answers here.

---
## Groot 1

The first thing we need to know is the location of the hosts file in Windows. This is a file that is used to resolve DNS. Any entries in this file will be used first before the PC reached out to a DNS server to resolve. An internet search should yield quick results:

```
C:\Windows\System32\drivers\etc\hosts
```

Now that we know where the hosts file is, we need to know how to get the MD5 hash. Another search shows that there is a built-in way to do this on newer versions on Powershell:

```
Get-FileHash C:\Windows\System32\drivers\etc\hosts -Algorithm MD5
```

---
## Groot 2

Most programming languages have a way to chop strings up into smaller parts, or substrings. Solving this challenge is as easy as searching for the PowerShell way to do substrings. Also remember that we can act upon the output of a command using parentheses.

```
(Get-Content .\elements.txt).substring(1481110, 7)
```

The numbers come from the index of the character you want to start the substring at (1481110), followed by the number of characters you want the substring to contain past that point (1481117 - 1481110 = 7)

---
## Groot 3

This is quite similar to the method used back in Century 14. We split the file into individual words, find the lines that contain the word we're looking for, and then count the result:

```
(Get-Content .\words.txt -Delimiter ' ' | findstr beetle).count
```

---
## Groot 4

We need to find the Drax subkey in the registry. The challenge gives us the registry root we need to work with, so let's go search for it:

```
Set-Location HKCU:

Get-ChildItem -Recurse | findstr Drax
```

Now that we know where it's located, take a look:

```
Get-ChildItem .\Software\Microsoft\Assistance\Drax
```

---
## Groot 5

Another AD user query. Here it gives you the username outright. I didn't know the correct property name for the username, so I had to run

```
Get-ADUser -Filter *
```

to get an overview of what was available. From there I found that I should be searching by SamAccountName, and that I needed the LogonWorkstations property:

```
Get-ADUser -Filter "SamAccountName -eq 'baby.groot'" -Properties LogonWorkstations
```

Don't forget to append the filename from the desktop.

---
## Groot 6

This is exactly the same as Cyborg 7. We need to look at the Run key.

```
Set-Location HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion

Get-ChildItem | findstr Run
```

Don't forget to append the filename from the desktop.

---
## Groot 7

To solve this we need to learn where to look in registry for information about services. A search reveals the following location:

```
Set-Location HKLM:\SYSTEM\CurrentControlSet\services
```

Then we just need to take a look at the service of interest:

```
Get-Item applockerfltr
```

Don't forget to append the filename from the desktop.

---
## Groot 8

Time to learn how to look at Windows Firewall. The base command is:

```
Get-NetFirewallRule
```

This gives us ALL the firewall rules. We need to narrow it down. I was able to find the rule they're talking about by just looking at the blocking rules:

```
Get-NetFirewallRule | ?{$_.Action -eq "Block"}
```

Don't forget to append the filename from the desktop.

---
## Groot 9

This one has us looking at AD OUs. To get all OUs, the command is:

```
Get-ADOrganizationalUnit -Filter *
```

We need to only view the names and whether or not they're protected from accidental deletion. To find out what the property name we need is:

```
Get-ADOrganizationalUnit -Filter * -Properties *
```

Then once you have found the property:

```
Get-ADOrganizationalUnit -Filter * -Properties ProtectedFromAccidentalDeletion | select Name,ProtectedFromAccidentalDeletion
```

Don't forget to append the filename from the desktop.

---
## Groot 10

We need to learn how to find differences between files. We can do so using the Compare-Object command. This takes two arguments containing the full text, so you need to call it like this, with two Get-Content commands in parentheses:

```
Compare-Object (Get-Content .\old.txt) (Get-Content .\new.txt)
```

---
## Groot 11

This uses the same method we used back in Cyborg 8. First you need to run this on all the files to find the one with an extra data stream:

```
Get-Item [file] -Stream *
```

Then you can get the data from the alternate stream:

```
Get-Content [file] -Stream [stream]
```

---
## Groot 12

You would think that you could get the owner of a file using Get-ChildItem or Get-ItemProperty, but turns out that information is kept in the file's ACL. To access it use the Get-Acl command:

```
Get-Acl '.\Nine Realms'
```

---
## Groot 13

This one threw me off. Up to this point I have been using Get-ChildItem and Get-Item to gain the information I need from the registry. That is because so far we have just been looking at keys and subkeys. To get the actual values and data from the registry, we need to use Get-ItemProperty. After an internet search to find where the Registered Owner is stored in registry:

```
Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion'
```

Don't forget to append the filename from the desktop.

---
## Groot 14

The first search results I found for enumerating file shares indicated that I should use

```
Get-FileShare
```

But this just returns an error. Digging a little deeper I found the one that works:

```
Get-SmbShare
```

---
## Thanks!!!

We have now completed Groot. If you want more PowerShell, check out the [Oracle writeup](./oracle.md).