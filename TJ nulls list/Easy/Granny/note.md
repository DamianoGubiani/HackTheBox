# Box: Granny

> Damiano Gubiani | 9/5/2026

export the box ip

```bash
export ip='10.129.95.234'
```

# Enumeration

## NMAP

running nmap scan for open ports

```bash
sudo nmap -sSCV -oN nmap/all -p - -min-rate 10000 $ip -vv -Pn -T4
```

output:

```text
Nmap scan report for 10.129.95.234
Host is up, received user-set (0.039s latency).
Scanned at 2026-05-09 01:01:07 CEST for 25s
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
| http-webdav-scan:
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Fri, 08 May 2026 23:01:27 GMT
|   WebDAV type: Unknown
|   Server Type: Microsoft-IIS/6.0
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
| http-ntlm-info:
|   Target_Name: GRANNY
|   NetBIOS_Domain_Name: GRANNY
|   NetBIOS_Computer_Name: GRANNY
|   DNS_Domain_Name: granny
|   DNS_Computer_Name: granny
|_  Product_Version: 5.2.3790
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-title: Under Construction
|_http-server-header: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

the version of the web server _Microsoft IIS httpd 6.0_ is subject to a CVE that grant us remote code execution on the machine

# RCE

to get remote code execution we use the following metasploit module

> exploit/windows/iis/iis_webdav_scstoragepathfromurl

we put the target IP and our vpn ip as listener and than we run the exploit

![](Screen/Pasted%20image%2020260509030243.png)

we got a reverse shell on the target.

# Privilege Escaletion

than i ran the exploit suggester to get a privilege escalation vector using this module on metasploit

> exploit/windows/local/ms10_015_kitrap0d

i than set up the session and my ip address and ran the script

![](Screen/Pasted%20image%2020260509170235.png)

we got a system shell on the target