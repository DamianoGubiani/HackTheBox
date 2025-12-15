# Box: admirer

> Damiano Gubiani | 11/12/2025

export box ip

```bash
export IP='10.129.229.101'
```

# Enumeration

running nmap scan for open ports

```bash
sudo nmap -sS -sC -oN nmap/scan -vv -T4 10.129.229.101
```

output

```
# Nmap 7.95 scan initiated Thu Dec 11 02:29:00 2025 as: /usr/lib/nmap/nmap -sS -sC -oN nmap/scan -vv -T4 10.129.229.101
Nmap scan report for 10.129.229.101
Host is up, received echo-reply ttl 63 (0.087s latency).
Scanned at 2025-12-11 02:29:01 EST for 6s
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDaQHjxkc8zeXPgI5C7066uFJaB6EjvTGDEwbfl0cwM95npP9G8icv1F/YQgKxqqcGzl+pVaAybRnQxiZkrZHbnJlMzUzNTxxI5cy+7W0dRZN4VH4YjkXFrZRw6dx/5L1wP4qLtdQ0tLHmgzwJZO+111mrAGXMt0G+SCnQ30U7vp95EtIC0gbiGDx0dDVgMeg43+LkzWG+Nj+mQ5KCQBjDLFaZXwCp5Pqfrpf3AmERjoFHIE8Df4QO3lKT9Ov1HWcnfFuqSH/pl5+m83ecQGS1uxAaokNfn9Nkg12dZP1JSk+Tt28VrpOZDKhVvAQhXWONMTyuRJmVg/hnrSfxTwbM9
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNHgxoAB6NHTQnBo+/MqdfMsEet9jVzP94okTOAWWMpWkWkT+X4EEWRzlxZKwb/dnt99LS8WNZkR0P9HQxMcIII=
|   256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBqp21lADoWZ+184z0m9zCpORbmmngq+h498H9JVf7kP
80/tcp open  http    syn-ack ttl 63
|_http-title: Admirer
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

Read data files from: /usr/share/nmap
# Nmap done at Thu Dec 11 02:29:07 2025 -- 1 IP address (1 host up) scanned in 7.63 seconds
```

running the default script of nmap we alredy know the FTP anonymous login isnt allowed

we see that the webapp has an /admin-dir directory

lets enumerate the webapp directory with ffuf , since we know we have an alredy disclosed directory , i will use the big.txt wordlist from seclist

command:

```bash
ffuf -u "http://10.129.229.101/admin-dir/FUZZ" -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -t 200 -i -ic -e .txt,.html,.php
```

output:

![](screen/Pasted%20image%2020251211112517.png)

we got a credential and contacts file 

after reviewing the file content i made 2 file one with usernames and one for the passwords found and i run a spray attack on the FTP protocol

command

```bash
nxc ftp 10.129.229.101 -u user -p pwd --ls
```

output:

![](screen/Pasted%20image%2020251211114616.png)

after connecting with FTP i downloaded all the resources found with mget

i extracted the html web app file structure and found some credentials

![](screen/Pasted%20image%2020251211120545.png)

while checking around i found other credentials

![](screen/Pasted%20image%2020251211132313.png)

![](screen/Pasted%20image%2020251211132328.png)

since this is a backup the application may have been changed in the time , 



