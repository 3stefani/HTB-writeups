# ⭐ Three – Hack The Box Write-up ⭐

![HTB](https://img.shields.io/badge/HTB-Starting_Point-success)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Category](https://img.shields.io/badge/Category-AWS_S3_RCE-red)

---

## Machine Info

| Property | Value |
|--------|-------|
| **Difficulty** | Easy |
| **OS** | Linux |
| **IP** | 10.129.x.x |
| **Category** | Starting Point (Tier 0) |
| **Tags** | AWS S3, Subdomain Enumeration, RCE, Misconfiguration |

---

## Overview

In this lab, we exploit a misconfigured Amazon S3-compatible service exposed through a discovered subdomain.  
By enumerating the S3 bucket, we identify that it allows unauthenticated file uploads.  
Uploading a malicious PHP file results in **remote code execution (RCE)** on the web server, allowing us to enumerate the filesystem and retrieve the flag.

This machine belongs to **Hack The Box – Starting Point (Tier 1)** and introduces fundamental concepts such as:

- Subdomain enumeration  
- Amazon S3 bucket interaction  
- Cloud storage misconfigurations  
- Remote Code Execution (RCE)  
- Linux filesystem enumeration  

---

## Attack Flow Summary

1. Network & service enumeration  
2. Subdomain discovery (`s3.thetoppers.htb`)  
3. Identification of exposed S3-compatible service  
4. Unauthorized access to S3 bucket  
5. Malicious PHP file upload  
6. Remote command execution  
7. Flag retrieval  

---

## Enumeration

### Port Scanning

```bash
nmap -sS 10.129.x.x
