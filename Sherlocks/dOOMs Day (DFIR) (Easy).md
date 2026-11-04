## Scenario

Suspicious traffic was observed on one of the internal Dev servers at DevNotes.org. Logs from the server and a packet capture from the subnet have been provided to you for analysis.

## About

In this Sherlock, players will analyze a compromised server that was targeted using the PHP2Shell exploit. The Sherlock focuses on understanding how the exploit works and exploring all the patterns observed in the web logs.

**Artifact:**
![](../Pasted%20image%2020261102105245.png)

```
┌──(kali㉿kali)-[~/HTB/Sherlocks/dOOMs_Day]
└─$ tree 'dOOMs Day'         
dOOMs Day
├── Capture.pcap
└── logs
    ├── access.log
    ├── error.log
    └── other_vhosts_access.log

2 directories, 4 files
```

## Recon

First let's take a look at the access.log:
- Internal Dummy Connection: An **`internal dummy connection`** in an Apache HTTP Server access log is a normal, internal mechanism used by the web server to wake up worker processes so they can handle incoming requests. 
- Apache sends a simple internal HTTP request (often `OPTIONS * HTTP/1.0 200`) to itself to wake up processes that are listening for new connections.
- We can see that this internal process also contains the version of PHP that we are looking for.
```
::1 - - [28/Oct/2026:08:47:44 +0000] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.69 (Debian) PHP/8.4.26 (internal dummy connection)"
::1 - - [28/Oct/2026:08:47:45 +0000] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.69 (Debian) PHP/8.4.26 (internal dummy connection)"
192.168.72.145 - - [28/Oct/2026:08:47:46 +0000] "GET /post/api-design HTTP/1.1" 200 469 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36 Edg/129.0.0.0"
::1 - - [28/Oct/2026:08:47:46 +0000] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.69 (Debian) PHP/8.4.26 (internal dummy connection)"
::1 - - [28/Oct/2026:08:47:47 +0000] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.69 (Debian) PHP/8.4.26 (internal dummy connection)"
::1 - - [28/Oct/2026:08:47:48 +0000] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.69 (Debian) PHP/8.4.26 (internal dummy connection)"
::1 - - [28/Oct/2026:08:47:49 +0000] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.69 (Debian) PHP/8.4.26 (internal dummy connection)"
```
--> Ver: 8.4.26

## CVE Research

The hint points to a **PHP2Shell** exploit. PHP 8.4.26 is affected by two CVEs (CVSS 9.2):

- **CVE-2026-109618:** An out-of-memory (OOM) handling validation flaw during multipart request parsing. When allocations fail, partially initialized data structures are left behind and treated as valid during teardown, causing memory corruption.
- **CVE-2026-109619:** A stale internal global state reuse issue, particularly involving pre-request session handlers like `session.upload_progress`.

**PHP2Shell** (CVE-2026-109618 / CVE-2026-109619) is an **unauthenticated Remote Code Execution** flaw in PHP under Apache `mod_php`. A crafted `multipart/form-data` request triggers an out-of-memory bailout in the pre-request `session.upload_progress` handler, leaving stale pointers that leak the Zend heap and `libphp.so` bases via `Set-Cookie` (defeating ASLR); a forged HashTable then redirects a destructor to `zif_system`, running commands as the web worker (`www-data`). No authentication is required and the vulnerable session settings are on by default.

Reference: [EQSTLab/CVE-2026-109619](https://github.com/EQSTLab/CVE-2026-109619)

Searching `error.log` for the `PHP_SESSION_UPLOAD_PROGRESS` field confirms the attack:
```
[Wed Oct 28 08:42:58.297155 2026] [dumpio:trace7] [pid 79:tid 79] mod_dumpio.c(100): [client 192.168.72.1:63175] mod_dumpio:  dumpio_in (data-HEAP): --SSSSSSSSSSSSSSSSS\r\nContent-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"\r\n\r\n
```

Attacker IP: `192.168.72.1` — distinct from the two legitimate IPs seen in `access.log` (`192.168.72.164` and `.145`).


```
192.168.72.145 - - [28/Oct/2026:08:47:43 +0000] "GET /search HTTP/1.1" 200 533 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36 Edg/129.0.0.0"
192.168.72.164 - - [28/Oct/2026:08:47:43 +0000] "GET /reports HTTP/1.1" 200 469 "https://duckduckgo.com/" "Mozilla/5.0 (iPad; CPU OS 18_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Mobile/15E148 Safari/604.1"
192.168.72.1 - - [28/Oct/2026:08:42:59 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:43:00 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:43:00 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:59 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:57 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:59 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:57 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:59 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:43:00 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:56 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:57 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:43:01 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:43:00 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"
192.168.72.1 - - [28/Oct/2026:08:42:59 +0000] "POST /index.php HTTP/1.1" 200 0 "-" "-"

```
--> We see lots of POST requests to the default index.php file, which is not normal. Looking at the timeline this process start at 28/Oct/2026:08:42:52 end at 28/Oct/2026:08:43:01. Correlating to the error.log we can see

![](../Pasted%20image%2020261102120643.png)

We can see that for each POST request of index.php, multipart/form-data is sent with various boundaries (ZZZZZZZZZZ) for now. The purpose of this is to engage Apache worker processes in open connections, leaving only 1 behind, which the exploit will then exploit to leak heap addresses so it can perform the actual out-of-memory attack

### Task 4 — The exploit deliberately occupied the Apache worker pool with stalled multipart uploads, forcing the exploit's requests to land on the same single worker so that memory is leaked from the same process heap. How many workers did the exploit deliberately occupy?

--> To find this, we need to count the total POST requests we identified before, which were sent by the IP address 192.168.72.1:

```
┌──(kali㉿kali)-[~/…/Sherlocks/dOOMs_Day/dOOMs Day/logs]
└─$ grep -c "POST /index.php HTTP/1.1" access.log 
149
```

Answer: 149 workers

### Task 6 — The exploit used the Out of Memory (OOM) primitive during exploitation. What was the PHP memory limit, in bytes, that the exploit exhausted to trigger the out-of-memory vulnerability?

--> From "https://mohammad-alattas.github.io/posts/From-php2shell-to-Detection-Obfuscated-PHP-Web-Shells-with-Sigma/" we can see The CVE-2026-109618 primitive is an out-of-memory bailout forced inside PHP’s multipart parser. That leaves a very specific error: `Allowed memory size of ... bytes exhausted` lets look for this keyword in error.log
![](../Pasted%20image%2020261102122507.png)

### Task 7 — The exploit leaks heap memory from the server to retrieve a heap address value, which is then used to facilitate the remote code execution process. What is the name of the cookie through which this leaked data is delivered to the attacker?
-The leak happens immediately after the first or second memory exhaustion, depending upon the accuracy of the exploit and the heap address values the exploit uses. The easiest way to spot it is to look at large POST requests. In access logs, this can be easily spotted as it is not as noisy as error logs

![](../Pasted%20image%2020261102125428.png)

We spot a POST request with a very big size. This big request was sent by the exploit, as session.upload_progress handling lives in rfc1867.c , which only runs when PHP parses a multipart/form-data POST body. Correlating with error logs, we see lots more multipart/form-data events, this time with boundary "BBBBBBBBBB", and the POST request matches what we found in access logs. We also spot lots of dummy data being sent to the server by the exploit.![](../Pasted%20image%2020261102130116.png)
![](../Pasted%20image%2020261102130419.png)

### Task 9 — When did the attacker achieve remote code execution capability?

--> here we know that the attacker is  running commands as the web worker (`www-data`) --> we analyze wireshark pcap file with the filter tcp contains "www-data"
![](../Pasted%20image%2020261102131434.png)
```
Frame 23860: Packet, 140 bytes on wire (1120 bits), 140 bytes captured (1120 bits)
    Encapsulation type: Linux cooked-mode capture v1 (25)
    Arrival Time: Oct 28, 2026 15:43:02.458047000 +07
    UTC Arrival Time: Oct 28, 2026 08:43:02.458047000 UTC
    Epoch Arrival Time: 1793176982.458047000
    [Time shift for this packet: 0.000000000 seconds]
    [Time delta from previous captured frame: 13.000 microseconds]
    [Time since reference or first frame: 9 minutes, 57.254330000 seconds]
    Frame Number: 23860
    Frame Length: 140 bytes (1120 bits)
    Capture Length: 140 bytes (1120 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: sll:ethertype:ip:tcp:data]
    Character encoding: ASCII (0)
    [Coloring Rule Name: TCP]
    [Coloring Rule String: tcp]
Linux cooked capture v1
Internet Protocol Version 4, Src: 192.168.72.50, Dst: 192.168.72.1
Transmission Control Protocol, Src Port: 55914, Dst Port: 4444, Seq: 77, Ack: 1, Len: 72
Data (72 bytes)

```

Answer: 2026-10-28 08:43:02 UTC
### Task 11 — When did the exploit cause a second crash of an Apache worker process?
![](../Pasted%20image%2020261102132047.png)
![](../Pasted%20image%2020261102132144.png)

```
┌──(kali㉿kali)-[~/…/Sherlocks/dOOMs_Day/dOOMs Day/logs]
└─$ grep "Segmentation fault" error.log 
[Wed Oct 28 08:46:16.516499 2026] [core:notice] [pid 1:tid 1] AH00052: child pid 145 exit signal Segmentation fault (11)
[Wed Oct 28 08:47:42.631917 2026] [core:notice] [pid 1:tid 1] AH00052: child pid 170 exit signal Segmentation fault (11)
```

Answer: 2026-10-28 08:47:42 UTC

### Task 12 — What is the command executed to find database files on the compromised server?

Following the TCP stream: 
![](../Pasted%20image%2020261102132438.png)
