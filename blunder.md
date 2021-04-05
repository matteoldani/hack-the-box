# Blunder walkthrough

Machine ip: 10.10.10.191

Nmap results:

```
# Nmap 7.80 scan initiated Sun Sep 20 05:36:03 2020 as: nmap -sC -sS -sV -v -o nmap.out -p- 10.10.10.191
Nmap scan report for 10.10.10.191
Host is up (0.26s latency).
Not shown: 65533 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: A0F0E5D852F0E3783AF700B6EE9D00DA
|_http-generator: Blunder
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Sep 20 05:48:51 2020 -- 1 IP address (1 host up) scanned in 767.40 seconds

```

## User

Using Dirbuster on port 80 and looking for file with .php and .txt extension we can find a file todo.txt that tells us a username.

We can find the pasword for this user reading the blog because every article is taken from wikipedia but a paragraph is added with a strange word. That's the password for
the login found before during the Dirbuster enumeration. The service hosted is bludit and after a search on google i found a cve that report a script to get a reverse shell.
The problem is in the uploading of an image https://github.com/cybervaca/CVE-2019-16113.

Thanks to that script we can get in as www-data but browsing through the folders of the site, in the next version of bludit we find a database file containing the
users, including hugo. By trying to crack the hash, on Crackstation you can easily get the plaintext password. With the command: ```su hugo```

we go in and get the user flag.

## Root

Running ```sudo -l``` we get it looks like it can run any command as root, but by running bash for example we get the message:
Sorry, user hugo is not allowed to execute '/bin/bash' as root on blunder.

An online article helped me: https://n0w4n.nl/sudo-security-bypass/
Root done!
