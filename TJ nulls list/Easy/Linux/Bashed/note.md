# Box: Bashed

> Damiano Gubiani | 1/5/2026

export the box ip

```bash
export IP='10.129.29.47'
```

# Enumerating

## NMAP

running nmap to scan for open ports

command

```bash
sudo nmap -sSCV -oN nmap/fst -T4 -vv --min-rate 10000 $IP
```

output

```
Nmap scan report for 10.129.29.47
Host is up, received reset ttl 63 (0.053s latency).
Scanned at 2026-04-30 21:48:38 EDT for 7s
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
```