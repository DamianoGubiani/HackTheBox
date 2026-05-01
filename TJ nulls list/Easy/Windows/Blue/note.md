# Box: Blue

> Damiano Gubiani | 1/5/2026

export the box ip

```bash
export IP='10.129.29.42'
```

# Enumerating

## NMAP

running nmap to check for open ports

```bash
sudo nmap -sSCV -oN nmap/fst -T4 -vv --min-rate 10000 $IP
```

output:

```
Nmap scan report for 10.129.29.42
Host is up, received echo-reply ttl 127 (0.027s latency).
Scanned at 2026-04-30 21:37:53 EDT for 68s
Not shown: 991 closed tcp ports (reset)
PORT      STATE SERVICE      REASON          VERSION
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds syn-ack ttl 127 Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49155/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49156/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49157/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# Exploit

## MS17-010

we can see the version of the machine wich is windows 7 pro 7601 SP 1 this version is affect by a vulnerability called eternal blue wich is on metasploit ready to use

to use it you need to load the module:

> exploit/windows/smb/ms17_010_eternalblue

and after running the module we get a call back

![](Screen/Pasted%20image%2020260501034542.png)


