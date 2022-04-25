[Home](../../index.md)

# UnderTheWire Cyborg Writeup

[Cyborg](https://underthewire.tech/cyborg) is the next wargame at Under the Wire after Century. If you haven't completed Century, you should do so first. As stated on the Century writeup, you will get the most out of this if you try your best and search the internet before looking for the answers here.

---
## Cyborg 1

To solve this challenge we need to query the Active Directory users, find a specific user, and retrieve the State listed for said user. If we knew the exact username of the user, we could use:

```
Get-ADUser -Identity [username] -Properties State
```

to get the user information, including the State property. Since we don't know the naming scheme, we need to run a search using the -Filter option like so:

```
Get-ADUser -Filter "Name -like '*chris*'" -Properties State
```

---
## Cyborg 2

This is a basic DNS lookup. If you have ever worked with DNS before, you probably know that you can just run:

```
nslookup CYBORG718W100N
```

to get the IP address of CYBORG718W100N. However, since this is a PowerShell training exercise, the more PowerShell-y way to do this is:

```
Resolve-DnsName CYBORG718W100N
```

Remember to append the filename from the desktop.

---
## Cyborg 3

You can get the members of a group with Get-ADGroupMember, and then use the -Identity option to select the group. Then we count it with the ().count method:

```
(Get-ADGroupMember -Identity Cyborg).count
```

Remember to append the filename from the desktop.

---
## Cyborg 4

To get available modules, we can use

```
Get-Module -ListAvailable
```

You can see the answer from just that command, but if you wanted to streamline it you could pipe it into findstr:

```
Get-Module -ListAvailable | findstr 8.9.8.9
```

Remember to append the filename from the desktop.

---
## Cyborg 5

Like Cyborg 1, this will require us to query the AD Users. The best way I can tell to find the user with logon hours set is to show all users, make sure the name and logonhours are included in the output, and then select only these values to be shown together on each line. This makes it obvious who has logon hours set:

```
Get-ADUser -Filter * -Properties name,logonhours | select name,logonhours
```

Remember to append the filename from the desktop.

---
## Cyborg 6

Based on the "ciphertext" (encoding is NOT the same thing as encrypting, I disagree with their calling this a cypher), I can tell that this is a base64-encoded string. You could simply copy the encoded string into a number of online decoders and get the answer, but if you want to do it the PowerShell way, this is the best method I could find with a quick search:

```
$data = Get-Content .\cypher.txt

[System.Text.Encoding]::unicode.getstring([system.convert]::FromBase64String($data))
```

---
## Cyborg 7

Here is where I started leaning pretty heavily on the hints. The hint mentions the Run key. In PowerShell you can view the registry just like you view the rest of the file system. A quick search revealed that the Run key for a user is defined at HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run, so we change to the enclosing directory and show the children:

```
Set-Location HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion

Get-ChildItem
```

---
## Cyborg 8

If you have been participating in CTFs, you probably know that files can contain additional data that is not apparent just by opening the file. In this case there is more than one data stream present in the file, and the answer is contained in the alternate data stream. To view all the available streams for the file:

```
Get-Item .\1_qs5nwlcl7f_-SwNlQvOrAw.png -Stream *
```

Using the output, you can identify the alternate stream (the one that's not $:Data) and view the stream with Get-Content:

```
Get-Content .\1_qs5nwlcl7f_-SNlQvOrAw.png -Stream Zone.Identifier
```

And Yes, the password is a single character, I thought I had to have it wrong at first.

---
## Cyborg 9

This is another AD User challenge, much like Cyborg 1 and 5. In this case we need to find a user with a specific phone number. I didn't know what property name constitued the phone number, so I first ran this to show all property names:

```
Get-ADUser -Filter * -Properties *
```

From that output I found 3 possible phone number fields. This gave me enough info to run the below command to get the answer, including and then selecting all 3 properties:

```
Get-ADUser -Filter * -Properties OfficePhone,HomePhone,MobilePhone 
                | select Name,HomePhone,MobilePhone,OfficePhone | findstr 876-5309
```

Remember to append the filename from the desktop.

---
## Cyborg 10

The hint for this one makes it clear that we should be using Get-AppLockerPolicy for this one. To show the whole effective policy in XML format:

```
Get-AppLockerPolicy -Effective -Xml
```

This includes the policy and description the challenge is asking for, though it may be a bit hard to read. I'm sure there's a better option for the formatting but I didn't bother searching after I saw the answer. Remember to append the filename from the desktop.

---
## Cyborg 11

I knew from experience that the IIS logs are located at C:\inetpub\logs. From there I found the log file in C:\inetpub\logs\logfiles\w3svc1 . In that directory, you can search the logs using the findstr command, specifically using the /v flag to eliminate the two terms they tell you not to look for in the challenge:

```
findstr /v Mozilla .\u_ex160413.log | findstr /v Opera
```

---
## Cyborg 12

All the way back in Century 10 we dealth with getting information on a service. We use the same sort of process here, but I've rewritten it to be a bit more elegant in my opinion, first filtering by the specific server name and then selecting the Pathname:

```
Get-WmiObject win32_service | ?{$_.name -eq "i_heart_robots"} | select PathName
```

You can then use your favorite method of base64 encoding to get the characters noted in the challenge. Remember to append the filename from the desktop.

---
## Cyborg 13

I solved this one with a low-effort search. Use the below command to get the info:

```
Get-DnsServerZoneAging underthewire.tech
```

Remember to append the filename from the desktop.

---
## Cyborg 14

The hint gives away how you should start looking for it. win32_DCOMApplicationSetting is the type of WMI Object you are looking for:

```
Get-WmiObject win32_DCOMApplicationSetting
```

But there's too much there! We need to select only the one with the AppID noted in the challenge:

```
Get-WmiObject win32_DCOMApplicationSetting | ?{$_.AppID -eq "{59B8AFA0-229E-46D9-B980-DDA2C817EC7E}"}
```

Remember to append the filename from the desktop.

---
## Thanks!!!

We have now completed Cyborg. Thanks for following along on this PowerShell journey. If you're ready for more, check out the [Groot writeup](./groot.md).