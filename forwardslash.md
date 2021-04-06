# ForwardSlash walkthrough

Machine ip: 10.10.10.183

Nmap output:

```
$ nmap -p- -sC -sV -oN forwardslash.nmap 10.10.10.183
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-06 15:59 EDT
Nmap scan report for forwardslash.htb (10.10.10.183)
Host is up (0.046s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 3c:3b:eb:54:96:81:1d:da:d7:96:c7:0f:b4:7e:e1:cf (RSA)
|   256 f6:b3:5f:a2:59:e3:1e:57:35:36:c3:fe:5e:3d:1f:66 (ECDSA)
|_  256 1b:de:b8:07:35:e8:18:2c:19:d8:cc:dd:77:9c:f2:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Backslash Gang
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.01 seconds
```
## User & Root

For this machine only my personal notes are available (and they are written in italian)
For a better writeup here on github I sugget reading: https://github.com/zweilosec/htb-writeups/blob/master/linux-machines/hard/forwardslash-write-up.md


da nmap vedo solo le porte 80 e 22 aperte
colleandosi alla porta 80 scopro che il sito è stato hacketato e con un po' di enumerazione trovo una nota
che mi dice che hanno un sito di backup
non trovando altro enuerando il dominio principale uso wfuzz per enumerare i sottodomini e scopro il dominio
del sito di backup --> backup.forwardslash.htb

a questo sito è possibile creare un account e una volta fatto il login scopro che, tra le tante cose, hanno
disabilitato la possibilità di cambiare la foto di profilo
riattivando i bottoni e intercettando con burp scopro una LFI che ci permette di vedere alcuni file.
(nel campo url devo mettere il path al file che voglio che venga incluso)
tramite un'altra enumerazione scopro che ci sono altri file .php sul dominio, in particolare trovo la certella
/dev che contiene index.php e il file config.php
cercando di accedere a quelli mi viene dato un messaggio di errore allora provo a usae php wrapper con fileter
e encodig in base64 per baypassare le restrizioni e funziona
il payload esatto è: url=php://filter/convert.base64-encode/resource=file://config.php
(volendo posso anche mettere il path completo /var/www/backup.forwardslash.htb/config.php)
in questo modo riesco anche a leggere il file index.php nella cartella dev e trovare la password per ftp di Chiv

non essendoci fpt abilitato provo la password su shh e riesco ad entrare.
entro e continuando a enuerare sia manualmente che con linpeas/lineneum trovo la cartella con i backups in /var/backups
con anche una nota che dice che dentro config ci sono le password importanti
trmite gli script di enumerazione noto un binary di Pain (lo user che devo explotare). Provando ad eseguirlo vedo che
cerca di aprire un file che ha come nome un hash e come nota mi dice che il file si apre solo se prendo il secondo
giusto. Come altra info mi da il current time a cui è stato eseguito il comando. Facendo un md5 encryption dell'ora
noto che corrisponde al nome del file che cerca di aprire. A questo punto facccio un link che punta al file config.pph.bak
che voglio leggere con nome uguale all'md5 del tempo. Per non dover calcolare veramente l'hash mi limito a chimare
backup, prendere l'errrore estrarre l'hash e usarlo per creare il link e poi richiamare backup

lo script non riesco ad eseguirlo in /tmp ma riesco a farlo nella home di chiv, non so perchè

i=$(backup  | grep ERROR | awk '{print $2}');
ln -s /var/backups/config.php.bak /home/chiv/$i;/usr/bin/backup;

in questo modo leggo la password di pain, su - pain --> user.txt

facendo un briuteforce del codice di criptazione che troviamo sulla home di pain trovimao in messaggio
here is the key to the encrypted image from /var/backups/recovery: cB!6%sdH8Lj^@Y*$C2cf

usando il comando su -l si scopre che pain può usare diersi comandi come sudo senza dover mettere la password
User pain may run the following commands on forwardslash:
    (root) NOPASSWD: /sbin/cryptsetup luksOpen *
    (root) NOPASSWD: /bin/mount /dev/mapper/backup ./mnt/
    (root) NOPASSWD: /bin/umount ./mnt/

il primo ci permette di creare il backup decriptato se passiamo come argomenti l'immagine in recovery. Se come output mettiamo 'backup'
sappiamo che tramite il secondo comando possiamo montare il backup in una cartella mnt
con il terzo comado poi possiamo smontare

dentro la partizione montata troviamo una chiave privata che se proviamo ad usarla per collagarci come root
tramite ssh scoprimao che funziona. Otteniamo così la root
