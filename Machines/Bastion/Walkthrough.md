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


