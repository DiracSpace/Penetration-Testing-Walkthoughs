# [Advent of Cyber - Day 2](https://tryhackme.com/room/adventofcyber2)

`machine IP - 10.10.180.67`

So, after reading the long text of information we were instructed to visit the website. Also, we were previously given an ID to which we use and login.

```
At the bottom of the dossier is a sticky note containing the following message:

For Elf McEager:
You have been assigned an ID number for your audit of the system: **************** . Use this to gain access to the upload section of the site.
Good luck!
```

`Question 1 - What string of text needs adding to the URL to get access to the upload page?`

It's pretty obvious we need the ID given to us to pass the authentication. What we need is to pass the value as a parameter, like so `http://machine-ip/?id=****************`.

[1.png]

From here, we are redirected to the "dashboard" and we can see two buttons. One is obviously asking us to select and submit a file for uploading. 

`Question 2 - What type of file is accepted by the site?`

While it doesn't actually tell us, we can check the `HTML` and view the input types. What I had done previously was go to `inspect element` and view the `sources` tab. Here I found `CSS, JS and IMG` folders, so that's how i answered this one.

[2.png]

What I did for this, I remembered two things. The first was that when I did the room [Ignite](https://tryhackme.com/room/ignite), we used `netcat` to open a listener for incoming connections. Plus, [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell) has a `PHP Reverse Shell` which was mentioned to us in the task introductory. So, what I eventually did was change the file extensión to `.jpg.php` and uploaded that bad boy

[3.png]

```
> nc -lvnp port
listening on [any] port ...
```

`Question 3 - In which directory are the uploaded files stored?`

This question was also given to us in the introductory, the only problem was that they didn't specify that the ID parameter was going to always be present. So the directory is `http://machine-ip/uploads/?id=****************`.

[4.png]

`Activate your reverse shell and catch it in a netcat listener!`

Here, we just need to get `PHP` to read and interprete the code we uploaded. To do that, we go to the url of the file. Like so `http://machine-ip/uploads/shell.jpg.php/?id=****************`

`Question 4 - What is the flag in /var/www/flag.txt?`

Finally, since the flag is in the directory were `Apache` has read permissions, all we needed to do is either `cat` the file (given the possibility that we have the direct filepath) or `cd` into that directory to finally print the results.

```
sh-4.4$ cat /var/www/flag.txt
cat flag.txt


==============================================================


You've reached the end of the Advent of Cyber, Day 2 -- hopefully you're enjoying yourself so far, and are learning lots! 
This is all from me, so I'm going to take the chance to thank the awesome @Vargnaar for his invaluable design lessons, without which the theming of the past two websites simply would not be the same. 


Have a flag -- you deserve it!
*************************************


Good luck on your mission (and maybe I'll see y'all again on Christmas Eve)!
 --Muiri (@MuirlandOracle)


==============================================================

```