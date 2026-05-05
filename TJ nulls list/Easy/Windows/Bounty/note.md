# Box: Bounty

> Damiano Gubiani | 1/5/2026

export box ip

```bash
export IP='10.129.29.199'
```

# Enumerating

## NMAP

running nmap scan for open ports

```bash
sudo nmap -sSCV -oN nmap/fst -T4 -vv --min-rate 10000 $IP
```

output

```
Nmap scan report for 10.129.29.199
Host is up, received echo-reply ttl 127 (0.045s latency).
Scanned at 2026-05-01 12:26:17 EDT for 12s
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 7.5
|_http-title: Bounty
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

lets check the webapp since is the only open port

## WebApp

after connecting theres nothing interesting so i just ran a directory bruteforce, witch we found something interesting a transfer.aspx file

command ran

```bash
gobuster dir -u http://10.129.29.199/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -t 200 --no-error -x php,html,txt,asp,aspx
```

output

![](Screen/Pasted%20image%2020260501183031.png)

after connecting to the transfer page the first thing i try is to upload an aspx file to get RCE on the machine

link of the backdoor

> https://raw.githubusercontent.com/tennc/webshell/refs/heads/master/net-friend/aspx/ASP.NET%20Web%20BackDoor.aspx

but after trying to upload it we get an error

![](Screen/Pasted%20image%2020260501183322.png)


