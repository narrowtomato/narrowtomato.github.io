[Home](../../index.md)

# UnderTheWire Century Writeup

[Under the Wire](https://underthewire.tech/) is a gamified approach to learning PowerShell, meant as sort of a sister site to [overthewire](https://overthewire.org/wargames/). Where overthewire focuses on Linux/Unix, underthewire focuses on Windows/PowerShell. I had fun and learned a lot with this, so I created a writeup below to solidify, share, and prove my knowledge.

Note that I will not be providing the answers here, just how I arrived at them and the command I ran to get there. If you want to get the most out of these exercises, I suggest you try it by yourself without looking at the writeup, search the internet, and refer here only when you are absolutely stuck.

---
## Century 1

This challenge asks for the build version of Powershell. A quick search got me the answer (this will be a recurring theme). Entering the variable

```
$PSVersionTable
```

at the prompt will get you the full details, or you can use

```
$PSVersionTable.BuildVersion
```

to get just the build version by itself. If you do the latter, make sure to adjust the formatting as noted in the challenge notes.

---
## Century 2

This is a two-part challenge, the first half asks for the "name of the built-in cmdlet that performs the wget like function within PowerShell". This can be found with a quick Google search.

```
Invoke-WebRequest
```

The rest is the name of the file on the desktop (current directory). You can find this by running the command

```
Get-ChildItem
```

or one of its aliases.

```
ls

dir

gci
```

Put those together and you get the next password, making sure to use all lowercase when logging in to the next challenge.

---
## Century 3

This asks for the number of files on the desktop. If you just run

```
Get-ChildItem
```

you will see that there are far too many to eyeball it, so we need to automate the counting process. A quick search brought up this method, where you enter your command in parentheses and append .count to the end. This will return the number of objects returned by the original command.

```
(Get-ChildItem).count
```

---
## Century 4

This challenge teaches you how to look in other directories other than your current one. You need to look in the directory with multiple spaces in order to retrieve a file name. The quickest way to do this is to run Get-ChildItem on the directory.

```
Get-ChildItem '.\Can You Open Me'
```

You can also change to the directory and then list the contents.

```
Set-Location '.\Can You Open Me'

Get-ChildItem
```

Note that you can use

```
cd
```

as an alias for Set-Location

---
## Century 5

To get information about the domain, run

```
Get-ADDomain
```

You can get the answer from that, but this is a good time to demonstrate using piping (|) to select specific properties from the output. This command will give you just the Name of the domain:

```
Get-ADDomain | select Name
```

Don't forget to add the name of the file on the desktop to your answer.

---
## Century 6

This one can be answered using the same command as Century 3. I think the intention is to get you to count only the directories, but since there are only directories present, you don't need to discriminate. Still, below is what I think is the intended command to solve this challenge, getting only the items which are directories.

```
(Get-ChildItem -Directory).count
```

---
## Century 7

You first need to find the readme file. While you can use the methods from Century 4 to change into each directory (keep in mind that you start in desktop, you will first need to "cd .."), it is much faster to run the following from your user directory:

```
Get-ChildItem -Recurse
```

This will list the contents of all directories recursively, revealing the location of the readme file.

The next order of business is to list the contents of that file. Change to the directory containing the file and run:

```
Get-Content [file]
```

Obviously replacing [file] with the name of the readme file.

---
## Century 8

So we know how to list the contents of a file, and we know how to count the objects returned by a command (for file content, .count defaults to number of lines, more on how to change that later). How do we only count the lines that are unique? Using a pipe to redirect the content into Sort-Object, and using the -Unique option:

```
(Get-Content .\unique.txt | Sort-Object -Unique).count
```

Here you can really start to see how you can chain together commands and perform operations on the results to get what you want. ().count here acts on the results of the entire command enclosed in ().

---
## Century 9

To find the 161st word in the file, we first need to convert the words to an iterable format. Currently they are all on the first line in the file. I had mentioned in Century 8 that the count for Get-Content defaults to one item per line. We need to change that using the -Delimiter option to separate the objects by spaces:

```
Get-Content .\Word_File.txt -Delimiter ' '
```

This gives you a full list of words, but how do we get the 161st? You can use either the -TotalCount or the -Head option:

```
Get-Content .\Word_File.txt -Delimiter " " -TotalCount 161

Get-Content .\Word_File.txt -Delimiter " " -Head 161
```

The last line of the output will be your answer.

---
## Century 10

My first thought for this one was to try and use

```
Get-Service wuauserv
```

but that does not return the relevant information. After searching I was able to find an effective method:

```
Get-wmiobject win32_service | select Name,Description | findstr wuauserv
```

First we get all of the win32 services, then we select only the Name and Description, then we filter by the Windows Update service name. You can then count out the words in the description and get your answer. Don't forget to append the name of the file on the desktop.

---
## Century 11

This is very similar to the first part of Century 7, but an additional option is needed to show hidden files:

```
Get-ChildItem -Recurse -Hidden
```

Remember to run this from the user folder, not the desktop


## Century 12

Using the below command will get you information about the domain controller:

```
Get-ADDomainController
```

But the object returned does not have a description. You need to use the Name found by running that command in an additional command to get the description of the computer:

```
Get-ADComputer [name] -Properties Description
```

Replace [name] with the name you found with the first command. The -Properties option has to be included or you will not see the Description.

Make sure to append the file name from the desktop (not sure why they keep including this).

---
## Century 13

In my opinion, this is a dumbed down or maybe remixed version of Century 9. You should be able to do this without help at this point, but here is the command:

```
(Get-Content .\countmywords -Delimiter ' ').count
```

---
## Century 14

In order to get the file into a format we can work with, we use the same method as Century 13. We then can use findstr (this is similar to 'grep' for Linux/Unix users) to only show what matches 'polo'.

```
Get-ChildItem .\countpolos -Delimiter " " | findstr polo
```

You will notice a few items that contain 'polo', but they are not just the word by itself. To filter these out, you can add a '^' before 'polo' to only match instances where it is at the beginning of the line. Finally, count the items to get the answer:

```
(Get-ChildItem .\countpolos -Delimiter " " | findstr ^polo).count
```

---
## Thanks!!!

Thanks for reading, now we can do some basic and useful tasks in Powershell, and hopefully you have a good grasp on how the language is put together. If you're ready for more, see my [writeup on Cyborg](./cyborg.md).