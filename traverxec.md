# Traverxec walkthroughs

Machine ip: 10.10.10.165

Running ```nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.165``` gave this output:
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey:
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## User

Searching on google for Nostromo vulnerabilities I've found https://www.rapid7.com/db/modules/exploit/multi/http/nostromo_code_exec
and i managed to get a shell as www-data.

Going into the directory of the site we find the nostromo configuration file (in /var/nostromo/conf), in
which there is the path containing a password: david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/. Wew can crack it with jhon.

The password doesn't work either with ssh or with the command su but there was a "protected_area" inside public_www so let's try there.
So, let's connect to http://10.10.10.165/~david/protected-file-area/, and when we are asked for the
credentials we insert the ones we have. we can login and we find a ssh private key backup. It encrypted but we can force it with jhon. Now we can use
our ssh key to log in as David in the machine and get the user flag.

## Root

Simply GFTObins thanks to a journalctl program run as sudo that we can find in the file in David's home: ```server-stats.sh```.
In this case, simply run the command and type ```!/bin/sh``` and we get e shell with root privileges.
