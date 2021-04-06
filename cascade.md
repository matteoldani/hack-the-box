# Cascade walkthrough

Machine ip: 10.10.10.182

Nmap output:

```
# Nmap 7.80 scan initiated Thu Apr 23 16:33:10 2020 as: nmap -sC -sS -sV -p 0-8081 -o nmap.txt 10.10.10.182
Nmap scan report for 10.10.10.182
Host is up (0.12s latency).
Not shown: 8072 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-04-23 20:47:44Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3m09s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-04-23T20:47:56
|_  start_date: 2020-04-23T19:53:50

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 23 16:47:54 2020 -- 1 IP address (1 host up) scanned in 884.59 seconds

```

## User

After discovering default port for a windows machine i can get a list of username with enum4linux:
```
enum4linux -U 10.10.10.182
```
To get other useful information we have to user ldapsearch:

```
ldapsearch -h 10.10.10.182 -x -s base defaultNamingcontext
```

After that we can get the correct namingcontext we can re run the same command with the flag -b.

As result we get a lot of information but with a ```grep psw``` we can find a legacy password encoded with base64.
This password referred to the user r.thompson but we cannot use this credentials with evil-winrm.
Moving to smbclient we can get access to the shares

```
smbclient //10.10.10.182/Data -U r.thompson
```

In the shares we can find a configuration file for ThightVNC that stores the password for the user smith.
We can now user evil-winrm to get a shell and grab the user flag.

## Root

Thanks to our new privileges we can explore another share so back to smcbclint we can enumerate a little more.
We find two intrsting files, a .exe program and a .dll config file. With a little bit of reverse engineering on IDA
we can get the decrypted password of arksvc.

We are now able to restore the credential of TempUser, deleted by arksvc, that we now have the same password of the administrator
thanks to a file that we find before.

Connecting with evil-winrm we now have to actually perform action to restore the objects.

At this site we can find some information about what we are supposed to do: https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/

So executing the following command, we can get the based64 encoded password of TempUser:
```
Get-ADObject -Filter {SamAccountName -eq 'TempAdmin'} -IncludeDeletedObjects -Properties *
```
Evil-winRM is now capable of getting a shell with the administrator privileges and we can get the root flag.
