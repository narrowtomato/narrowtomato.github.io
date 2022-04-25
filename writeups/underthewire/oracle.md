[Home](../../index.md)

# UnderTheWire Oracle Writeup

[Oracle](https://underthewire.tech/oracle) is the next wargame at Under the Wire after Groot. If you haven't completed Century, Cyborg, and Groot, you should do so first. As stated on the Century writeup, you will get the most out of this if you try your best and search the internet before looking for the answers here.

---
## Oracle 1

Oracle leans heavily on the registry-related challenges, many are just a matter of searching the internet for the location of the registry key they want you to find. This is one of such challenges:

```
Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\control\TimeZoneInformation
```

---
## Oracle 2

This is similar to Groot 1, but this time we have to hash multiple files at once. We can do so with pipe magic:

```
Get-ChildItem | Get-FileHash -Algorithm MD5 | select Hash
```

---
## Oracle 3

Here we are introduced to the Get-WinEvent command. This is used to view the Widnows Event Log files such as the one available for this challenge. In this case the answer is obvious near the end of the log, so all that is needed is to successfully run the command on the file:

```
Get-WinEvent -Path .\Oracle3_Security.evtx
```

---
## Oracle 4

This challenge and the next teach you to use the Get-GPO command. To make retrieving the answer a bit easier, we can select just the name and creation time and sort it:

```
Get-GPO -All | select CreationTime,DisplayName | Sort-Object
```

---
## Oracle 5

This is really just a repetition of Oracle 4:

```
Get-GPO -All | select Description,DisplayName
```

---
## Oracle 6

Instead of looking at GPOs, here we are looking at OUs. First I ran this so I knew what properties I needed to look for:

```
Get-ADOrganizationalUnit -Filter * -Properties *
```

Then I added a bit more to the command to reveal the answer:

```
Get-ADOrganizationalUnit -Filter * -Properties LinkedGroupPolicyObjects | select Name,LinkedGroupPolicyObjects
```

---
## Oracle 7

To find info on domains we trust, use the command:

```
Get-ADTrust
```

---
## Oracle 8

This log file is a flat text file, and we know what string we're looking for:

```
Get-Content .\logs.txt | findstr guardian.galaxy
```

---
## Oracle 9

I struggled with this one for a while, powershell has a specific command for looking at all the records in a zone:

```
Get-DnsServerResourceRecord -ZoneName underthewire.tech
```

To get the hostname of the mail server, look for the MX record.

---
## Oracle 10

The hint indicates we should look in the registry. I wasn't able to find where this registry key was supposed to be, but since we are looking for data for only the current user, I could guess that I should be searching HKCU:

```
Set-Location HKCU:

Get-ChildItem -Recurse | findstr .biz
```

---
## Oracle 11

Another registry challenge. Search the internet to find the right location and take a look:

```
Get-ChildItem HKCU:\Network
```

---
## Oracle 12

More registry, same methods:

```
Get-ChildItem 'HKCU:\Software\Microsoft\Terminal Server Client'
```

---
## Oracle 13

We're back to Windows Event files again, but this time we will need to do some additonal searching to find the event we're looking for, and some additional formatting so we can read it:

```
Get-WinEvent -Path .\security.evtx | Where-Object { $_.Message -Like '*Galaxy*' } | Format-List
```

---
## Oracle 14

You can use the same process as Orace 13 here, just with a different search term:

```
Get-WinEvent -Path .\security.evtx | Where-Object { $_.Message -Like '*Bereet*' } | Format-List
```

---
## Thanks!!!

We have now completed Oracle. Thanks for coming along. If you want more powershell, continue with the [Trebek writeup](./trebek.md).