# Postman walkthroughs

Machine ip: 10.10.10.160

Running ```db_nmap --min-hostgroup 96 -p 1–65535 -n -T4 -A -v 10.10.10.160``` gave this output:

```
[*] Nmap: 22/tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
[*] Nmap: 80/tcp open http Apache httpd 2.4.29 ((Ubuntu))
[*] Nmap: 6379/tcp open redis Redis key-value store 4.0.9
[*] Nmap: 10000/tcp open http MiniServ 1.910 (Webmin httpd)
```
## User

In addition to ports 22 and 80, we also have 10000 (Webmin).
By connecting to the latter service there is not much we can do, as to access we must have the credentials
of a user or root (by default Webmin uses the username and password of the root user)

Going forward with the inspection, the website on port 80 does not seem to be of great use, as it is a
simple page under construction --> Rabbithole

We can find also an instance of the redis database on port 6379, version 4.0.9, and with a quick search we can find an exploit
available on Github.

With the default username "redis" we can get a shell for he redis user

``` python redis.py 10.10.10.160 redis ```

With a little enumeration, we notice that there is a user called Matt in the system, and inside his home
folder, there is the user flag (so we must get to be him). Furthermore, in the opt folder, there is a file called
id_rsa.bak, which contains a backup of a private key.
Once the file is transferred to the machine, we use John to decrypt it

The password we find cannot guarantee access through ssh because it was disabled but it was also the user password itself
so we can do from the redis user:

``` su Matt ``` and put the password

Mission completed! We logged in as Matt and have our user flag.

## User

I come back to the webmin system. There are differents exploit solution to apply after a quick look on google

Exploit that work is ``` linux/http/webmin_packageup_rce ```
Try more than one time, I managed to have an administrator shell at my third time (the session could not be created). Finalli I got a shell and grab the root.txt flag

``` $ cat /root/root.txt ```
