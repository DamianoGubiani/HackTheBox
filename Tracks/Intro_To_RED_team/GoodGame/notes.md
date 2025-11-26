# Box: GoodGame

> Damiano Gubiani | 26/11/2025

export the box ip

```bash
export IP='10.10.11.130'
```

# Enumeration

running nmap for open ports

command:

```bash
sudo nmap -sS -sC -oN nmap/scan -vv -T4 $IP
```

output:

```
# Nmap 7.95 scan initiated Wed Nov 26 06:32:33 2025 as: /usr/lib/nmap/nmap -sS -sC -oN nmap/scan -vv -T4 10.10.11.130
Nmap scan report for 10.10.11.130
Host is up, received reset ttl 63 (0.15s latency).
Scanned at 2025-11-26 06:32:34 EST for 6s
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
|_http-favicon: Unknown favicon MD5: 61352127DC66484D3736CACCF50E7BEB
|_http-title: GoodGames | Community and Store
| http-methods: 
|_  Supported Methods: HEAD GET OPTIONS POST

Read data files from: /usr/share/nmap
# Nmap done at Wed Nov 26 06:32:40 2025 -- 1 IP address (1 host up) scanned in 6.80 seconds
```

# WebApp




