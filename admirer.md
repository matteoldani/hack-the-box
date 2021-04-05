# Admirer walkthrough

Machine ip: 10.10.10.187

Nmap results:

```
# Nmap 7.80 scan initiated Tue May  5 05:03:39 2020 as: nmap -sS -sV -sC -o nmap.txt -v 10.10.10.187
Nmap scan report for 10.10.10.187
Host is up (0.28s latency).
Not shown: 992 closed ports
PORT      STATE    SERVICE       VERSION
21/tcp    open     ftp           vsftpd 3.0.3
22/tcp    open     ssh           OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey:
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp    open     http          Apache httpd 2.4.25 ((Debian))
| http-methods:
|_  Supported Methods: GET OPTIONS
| http-robots.txt: 1 disallowed entry
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
311/tcp   filtered asip-webadmin
783/tcp   filtered spamassassin
2000/tcp  filtered cisco-sccp
9878/tcp  filtered kca-service
49155/tcp filtered unknown
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue May  5 05:03:57 2020 -- 1 IP address (1 host up) scanned in 18.09 seconds

```

## User

Navigating directly to that page gave me an access forbidden error, so I fired up Dirbuster and ran it on this directory.  This led me to a few useful sounding files: `contacts.txt` and `credentials.txt`.

```text
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb

##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb

#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```

The file `contacts.txt` contained some more potential usernames and a potentially useful email address format.

```text
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

`credentials.txt` contained credentials for a few services.  


Using the ftp credentials, I was able to log into the FTP server.  I found a few interesting files.

The file `dump.sql` contained a dump of the website database. Unfortunately,  it seemed as if the only useful information was the server version information and the database name and the name of a deleted table that looked to contain website files. I thought that this information could come in handy so I made note of it.

* **Database:** admirerdb
* **Table**: items \(deleted\)
* **Version**: MySQL dump 10.16 Distrib 10.1.41-MariaDB, for debian-linux-gnu \(x86\_64\)

The `Employees3` table had another list of potential usernames and email addresses that I added to my lists.


After fully checking out the database, I moved on to the file `html.tar.gz`. It contained a backup of the website's back-end code, including a very interesting PHP file called `admin_tasks.php`

This file looked like a nice little backdoor that the admin had left for me called the "Admin Tasks Web Interface \(v0.01 beta\)".

in the same folder was `db_admin.php` which contained another set of credentials, this time for the user `waldo` who I had seen in the `robots.txt`.

There was also another password for `waldo` in the `index.php` file.  This also referenced the `items` table that had been deleted from the database I exfiltrated.  If I could get a web shell into this table, the page would run it for me when the page loaded.

```text
User-agent: *

# This folder contains personal stuff, so no one (not even robots!) should see it - waldo
Disallow: /w4ld0s_s3cr3t_d1r
```

Inside this HTML backup was a different version of the `robots.txt`.  This time the disallowed folder was called `/w4ld0s_s3cr3t_d1r/`, which I had access to as a folder in the backup. This folder contained the files `contacts.txt` and `credentials.txt` which appeared at first to be the same as before.

```text
[Bank Account]
waldo.11
Ezy]m27}OREc$

[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

The `credentials.txt` had most of the same information as before, but `waldo` seemed to have left his bank account password in this one.  Despite finding a couple more passwords, none of these worked for logging into SSH for any user.

After I checked back on my Dirbuster scan of the `/utility-scripts/` folder, I noticed it had found a new page `adminer.php` where I found an adminer database management portal.

The version of the service that is running on the machine is vulnerable to a file inclusion attack and we can create an sql (https://www.microfocus.com/documentation/idol/IDOL_12_0/MediaServer/Guides/html/English/Content/Getting_Started/Configure/_TRN_Set_up_MySQL_Linux.htm)


I did a bit more research to figure out exactly how to set up the MySQL database.
After creating the database and a table called `admirer`, I created a user named `test` and gave it full permissions to manage the database.  

Next I had to set the binding for the server to the address `0.0.0.0` so that the external service could connect to it by my IP.  The default is `127.0.0.1` which is localhost only.


After changing the server `bind-address` setting to `0.0.0.0` I had to restart the `mysql` service for it to take effect. After that I was able to login to my database in the Adminer portal.

### Finding user creds

This bug bounty write-up detailed what I needed to do next. Essentially, I logged into the remote server's database management portal, but it was my own local database that I logged into.  After that, I abused a feature of MySQL that allows for local files to be imported into the database.  This is a type of local file inclusion \(LFI\) vulnerability.  

* [https://medium.com/bugbountywriteup/adminer-script-results-to-pwning-server-private-bug-bounty-program-fe6d8a43fe6f](https://medium.com/bugbountywriteup/adminer-script-results-to-pwning-server-private-bug-bounty-program-fe6d8a43fe6f)

I could access: `index.php`. The file was changed from the backup and now has the correct password to login with ssh. User, after a lot of rabbitholes is completed.

## Root

Running ```sudo -l``` we get:
```
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
```

Admin access folder script where i can read the content of files.
The sixth function of the script calls another script file that import a file: shutil.py.
We can redefine shultil.py so that in our enviroment python uses our library instead of the defaul one.
Since the script is run with sudo privileges we can make it execute a command such ad ``` car /root/root.txt``` to get our flag.

sudo PYTHONPATH=/var/tmp /opt/scripts/admin_tasks.sh
