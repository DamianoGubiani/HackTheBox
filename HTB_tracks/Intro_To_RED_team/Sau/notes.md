
# Box: Sau

> damiano gubiani | 19/11/2025

export box ip

```bash
export IP='10.10.11.224'
```

# Enumerating

running nmap on the target to check for open ports

command:

```bash
sudo nmap -sS -sC -sV -oN nmap/scan -p - --open -T4 -vv $IP
```

output:

```
Nmap scan report for 10.10.11.224
Host is up, received echo-reply ttl 63 (0.16s latency).
Scanned at 2025-11-19 10:13:34 EST for 98s
Not shown: 65531 closed tcp ports (reset), 2 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
| ssh-rsa 
55555/tcp open  http    syn-ack ttl 63 Golang net/http server
| http-methods: 
|_  Supported Methods: GET OPTIONS
| http-title: Request Baskets
|_Requested resource was /web
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Wed, 19 Nov 2025 15:14:59 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Wed, 19 Nov 2025 15:14:41 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Wed, 19 Nov 2025 15:14:42 GMT
|     Content-Length: 0
|   OfficeScan: 
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

seems we have a web page on port 55555/tcp 

lets check it out

# WebApp

