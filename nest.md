# Nest walkthroughs

Machine ip: 10.10.10.177

I started off my enumeration with an nmap scan of `10.10.10.177`.

```bash
$ nmap -p- -A -oA nest.full 10.10.10.178 -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-30 15:47 EDT
Nmap scan report for 10.10.10.178
Host is up (0.14s latency).
Not shown: 65534 filtered ports
PORT    STATE SERVICE       VERSION
445/tcp open  microsoft-ds?

Host script results:
|_clock-skew: 4m00s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-05-30T19:56:10
|_  start_date: 2020-05-30T17:04:39

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 330.76 seconds
```
## User

let's try to connect with anonymous authentication with smbclient. Luckily it worked, and I got back a listing of the shares on this machine. `Data`, `Secure$`, and `Users` all sound interesting as they are not default shares. I first tried connecting to `Secure$` but was denied. Next I tried to connect to `Data`.
We proceed with the enumeration until we find a file called Welcome Email.txt. By downloading the file with the get command, we have some credentials.

By reconnecting with smbclient, as a TempUser user, with the credentials just obtained, we can navigate
within the Data folder. Going to the \IT\Configs\RU Scanner\ path, we can download the RU_config.xml file,
with the c.smith encrypted credentials:
```
<?xml version="1.0"?>
<ConfigFile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema">
<Port>389</Port>
<Username>c.smith</Username>
<Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>

```
Instead in the path \IT\Configs\NotepadPlusPlus, there is the config.xml file, of the Notepad ++ application,
and there are the recently opened files:

```
<File filename="C:\windows\System32\drivers\etc\hosts" />
<File filename="\\HTB-NEST\Secure$\IT\Carl\Temp.txt" />
<File filename="C:\Users\C.Smith\Desktop\todo.txt" />
```
After searching around in the `/IT/Carl/` folder for awhile, I found a Visual Basic coding project that he had apparently been working on. `RU Scanner` was also the name of that config file I found earlier with the encrypted credentials for the `c.smith` user, so I knew I had to be on the right track.

```text
smb: \IT\Carl\VB Projects\WIP\RU\> ls
  .                                   D        0  Fri Aug  9 11:36:45 2019
  ..                                  D        0  Fri Aug  9 11:36:45 2019
  RUScanner                           D        0  Wed Aug  7 18:05:54 2019
  RUScanner.sln                       A      871  Tue Aug  6 10:45:36 2019

                10485247 blocks of size 4096. 6545777 blocks available
```
Using an online .NET decompiler, from the project's Utils.vb file, I copied and pasted the functions to
decrypt the string. By connecting with smb with the new credentials, it will be possible to obtain the flag, within the user
c.smith folder

## Root

I continued searching through the available `\HQK Reporting\` directory, where I found a few very interesting files. The `"Debug Mode Password.txt"` appeared to be empty, but from my past experiences with doing CTFs in Windows environments I have learned that if a filename says it has a password or flag in it, it probably does. You might just have to try a little harder to get it. In this case, the user tried to hide the password from common snooping by inserting it into an NTFS alternate data stream.

In order to locate any alternate data streams in this file over SMB, you need to use the `allinfo` command, then `Get` the file with the appropriate stream name appended to it.

```text
smb: \C.Smith\HQK Reporting\> allinfo
allinfo <file>
smb: \C.Smith\HQK Reporting\> allinfo "Debug Mode Password.txt"
altname: DEBUGM~1.TXT
create_time:    Thu Aug  8 07:06:12 PM 2019 EDT
access_time:    Thu Aug  8 07:06:12 PM 2019 EDT
write_time:     Thu Aug  8 07:08:17 PM 2019 EDT
change_time:    Thu Aug  8 07:08:17 PM 2019 EDT
attributes: A (20)
stream: [::$DATA], 0 bytes
stream: [:Password:$DATA], 15 bytes
smb: \C.Smith\HQK Reporting\> get "Debug Mode Password.txt:Password:$DATA"
smb: exit

$ cat 'Debug Mode Password.txt:Password:$DATA'
WBQ201953D8w
```
Let's try to connect in some way with the second port through telnet (port: 4386)
Now, if we move to the LDAP folder, we are able to obtain the administrator's encrypted password

To decipher it, simply download the HqkLdap.exe program and run it on Windows by placing the
configuration file Ldap.conf as an argument, and we will get the credentials

If we try to log in with smbclient, it will not be possible to get the flag. We must obtain a shell, through an
Impacket tool, psexec.py. After that we can get a root shell and grab the flag
