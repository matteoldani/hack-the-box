# Sauna walkthroughs

Machine ip: 10.10.10.175

Running ```nmap``` gave this output:

```
# Nmap 7.80 scan initiated Wed Apr  8 08:44:19 2020 as: nmap -sC -sS -sV -o nmap.txt -p 0-6000 10.10.10.175
Nmap scan report for 10.10.10.175
Host is up (0.060s latency).
Not shown: 5988 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings:
|   DNSVersionBindReqTCP:
|     version
|_    bind
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-04-08 20:49:45Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=4/8%Time=5E8DC7C3%P=x86_64-pc-linux-gnu%r(DNSVe
SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03");
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 8h02m49s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-04-08T20:52:06
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr  8 08:51:54 2020 -- 1 IP address (1 host up) scanned in 455.15 seconds
```

## User

We start enumeration, and on port 80, the below site is deployed. After enumerating the site, we can see that the team member names are displayed on a page.
Moving on with enumeration, since port 389 is also present in the nmap scan results, let’s use nmap ldap scripts to take a look at what we can enumerate. At the end, we can see one names matches from the team member names. That is one “Hugo Smith.”. to obtain further information we need to guess the correct username of Hugo so we can try with common rules:

```
hugo.smith
hugosmith
hsmith
smith.hugo
h.smith
```

Using that list as an input to Impacket’s GetNPUsers, we got a valid assertion on hsmith, which indicates that the internal naming convention used by them is the first letter of first name followed by last name. After that we can get a full list of correct usernames. And then use it as an input to GetNTUsers again. We got a session of fsmith this time and we get an hased password for hashcat to decrypt. Now let’s use evil-winrm to log in with these credentials. As we can see below, we were able to do so successfully. Grab the user flag.

## Root

Enumerating with winPEAS, Autologon credentials have been found. Moving forward with the new credentials, we can use SecretsDump from Impacket to grab other credentials and seems like we grab the administrator credentials. Using evil-winrm again to log in with the discovered administrator hash and grab the root flag.
