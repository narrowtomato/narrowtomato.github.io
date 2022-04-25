[Home](../../index.md)

# UnderTheWire Trebek Writeup

[Trebek](https://underthewire.tech/trebek) is the next wargame at Under the Wire after Groot. If you haven't completed Century, Cyborg, Oracle, and Groot, you should do so first. As stated on the Century writeup, you will get the most out of this if you try your best and search the internet before looking for the answers here.

---
## Trebek 1

Per the hint, we need to find out what a log message looks like when a task is deleted. Once we know that, this is similar to the last few Oracle problems:

```
Get-WinEvent -Path .\security.evtx | Where-Object { $_.Message -Like '*A scheduled task was deleted*' } | Format-List
```

---
## Trebek 2

Looking back to Cyborg 12, this is the same concept, different service name:

```
Get-WmiObject win32_service | ?{$_.Name -eq 'C-3PO'} | select PathName
```

---
## Trebek 3

Like Trebek 1, it helps to look up what a remote connection looks like in the logs. We also want to look for Yoda:

```
Get-WinEvent -Path .\security.evtx | ?{$_.Message -Like '*Source Network Address*' -and $_.Message -Like '*yoda*'} | Format-List
```

---
## Trebek 4

I found the hints here misleading. I was unable to query event logs for lack of permissions, and the FireEye article references registry keys that are not present on this system. I ended up solving it by looking in the Windows Prefetch directory:

```
Get-ChildItem C:\Windows\Prefetch | findstr /i access
```

---
## Trebek 5

This one gives us a cut-and-dried timestamp, it's just a matter of searching in the given log using that timestamp:

```
Get-WinEvent -Path .\application.evtx | ?{$_.TimeCreated -eq '3/23/2017 8:08:53 PM'} | Format-List
```

---
## Trebek 6

Here we search recursively for DLLs. In order to get the output in a format on which we can use findstr, I first pipe it through Out-String. Finally, we count the results:

```
(Get-ChildItem -Recurse | Out-String -Stream | findstr dll).count
```

---
## Trebek 7

The hint references Image File Execution Options. Searching for this term quickly reveals a Microsoft artile pointing us to the correct registry location. A little research about stickykeys reveals the executable to be "sethc.exe".

```
Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options'
```

---
## Trebek 8

Get-Content has an option that lets us view byte-by-byte, and we want the first 8 bytes:

```
Get-Content .\Clone_Trooper_data.pdf -Encoding Byte -TotalCount 8
```

---
## Trebek 9

This can be solved using the same method as Groot 14:

```
Get-SmbShare
```

---
## Trebek 10

Another event log search, easily solved by searching for an expected string:

```
Get-WinEvent -Path .\security.evtx | ?{$_.Message -Like '*kenobi*'} | Format-List
```

---
## Trebek 11

Like Trebek 5, we are looking for a date, but the hour and seconds are not explicit, so we have to chain together a few queries:

```
Get-WinEvent -Path .\security.evtx | ?{$_.TimeCreated -Like '*5/11/2017*' -and $_.TimeCreated -Like '*:26:*' -and
                $_.Message -Like '*A user account was created*'} | Format-List
```

---
## Trebek 12

You should be getting good at searching event logs by now. Same method, I found it by searching for when users were created:

```
Get-WinEvent -Path .\security.evtx | ?{$_.Message -Like '*A user account was created*'} | Format-List
```

---
## Trebek 13

Looking at all users' Name and City reveals the answer:

```
Get-ADUser -Filter * -Properties Name,City | select Name,City
```

---
## Trebek 14

First we need to view the entire encoded output from the last question:

```
Get-ADUser -Filter * -Properties Name,City | ?{$_.Name -eq 'Prindel'} | select Name,City | Format-List
```

Then paste it into your favorit base64 decoder.

---
## Thanks!!!

We have now completed Trebek, and all the wargames currently available on UnderTheWire. Thanks for coming along, stay tuned for more.