
# Box: Conversor

> Damiano Gubiani | 19/11/2025

# Enumeration

lets export the target IP address

```bash
export IP='10.10.11.92'
```

## NMAP

running nmap scan for open ports on the target

command:

```bash
sudo nmap -sS -sC -sV -oN nmap/scan --open -p - -T4 -vv $IP
```

output:

```
Nmap scan report for 10.10.11.92
Host is up, received reset ttl 63 (0.24s latency).
Scanned at 2025-11-19 04:44:07 CST for 74s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ9JqBn+xSQHg4I+jiEo+FiiRUhIRrVFyvZWz1pynUb/txOEximgV3lqjMSYxeV/9hieOFZewt/ACQbPhbR/oaE=
|   256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIR1sFcTPihpLp0OemLScFRf8nSrybmPGzOs83oKikw+
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://conversor.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

lets check the WepApp , first off i added the domain in my /etc/hosts file

command:

```bash
echo '10.10.11.92 conversor.htb' | sudo tee -a /etc/hosts
```

## WebApp

