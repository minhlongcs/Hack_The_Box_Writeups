# Management (Easy)

Enumerating:

```
──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV 10.129.4.125   
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-12 15:22 +0700
Nmap scan report for 10.129.4.125
Host is up (0.19s latency).
Not shown: 995 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp    open  http     nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://10.129.4.125/
443/tcp   open  ssl/http nginx 1.24.0 (Ubuntu)
| ssl-cert: Subject: commonName=management.htb/organizationName=Management Managed Services Ltd
| Subject Alternative Name: DNS:management.htb, DNS:*.management.htb
| Not valid before: 2026-06-02T01:21:44
|_Not valid after:  2126-05-09T01:21:44
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
|_http-title: Did not follow redirect to https://management.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
4444/tcp  open  ssl/ldap
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=sso.management.htb/organizationName=Administration Connector RSA Self-Signed Certificate
| Not valid before: 2026-06-02T01:23:59
|_Not valid after:  2046-05-28T01:23:59
| fingerprint-strings: 
|   LDAPSearchReq: 
|     0<0:
|     objectClass1+
|     ds-root-dse
|_    ds-cfg-root-dse-backend0
50389/tcp open  ldap     (Anonymous bind OK)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4444-TCP:V=7.99%T=SSL%I=7%D=10/12%Time=6ACC9917%P=x86_64-pc-linux-g
SF:nu%r(LDAPSearchReq,55,"0E\x02\x01\x07d@\x04\x000<0:\x04\x0bobjectClass1
SF:\+\x04\x03top\x04\x0bds-root-dse\x04\x17ds-cfg-root-dse-backend0\x0c\x0
SF:2\x01\x07e\x07\n\x01\0\x04\0\x04\0");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.69 seconds
```

```
sudo nano etc/hosts
```

- box tự chế ( ip private ) -> thêm ip + management.htb -> ánh xạ 

truy cập http://<ip>

![[Pasted image 20261013211513.png]]

ấn vào client login --> 

```
redirect to https://sso.management.htb/openam/XUI/#login/
```
we see that this website use openam to authenticate

![[Pasted image 20261013220446.png]]

--> we found out that openam is on ver16.0.5

Next we search online for openam v16.0.5 CVE

https://github.com/OpenIdentityPlatform/OpenAM/security/advisories/GHSA-2cqq-rpvq-g5qj
