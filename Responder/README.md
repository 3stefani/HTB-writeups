# ⭐ Responder – Hack The Box Write-up ⭐

![HTB](https://img.shields.io/badge/HTB-Starting_Point-success)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![OS](https://img.shields.io/badge/OS-Windows-blue)
![Category](https://img.shields.io/badge/Category-LFI_NTLM_WinRM-red)

## Machine Info

| Property       | Value                                           |
| -------------- | ----------------------------------------------- |
| **Difficulty** | Easy                                            |
| **OS**         | Windows                                         |
| **IP**         | 10.129.x.x                                      |
| **Category**   | Starting Point                                  |
| **Tags**       | LFI, RFI, NTLM, Responder, Hash Cracking, WinRM |

## Overview

In this lab, we exploit a Local File Inclusion (LFI) vulnerability to trigger an NTLM authentication request from a Windows server. Using Responder, we capture the NTLMv2 hash of the Administrator account, crack it with John the Ripper, and gain remote access via WinRM to retrieve the flag.

This machine belongs to Hack The Box – Starting Point (Tier 0) and introduces fundamental concepts such as:

Local & Remote File Inclusion (LFI / RFI)

NTLM authentication

Hash capture and cracking

Windows Remote Management (WinRM)

### Key Steps:
**Exploiting Local File Inclusion (LFI)** via unsanitized page parameter

MITRE ATT&CK: T1005 – Data from Local System

OWASP Top 10: A03:2021 – Injection

**Escalating LFI to Remote File Inclusion (RFI)** using UNC paths on a Windows host

MITRE ATT&CK: **T1105 – Ingress Tool Transfer**

OWASP Top 10: **A05:2021 – Security Misconfiguration**

**Capturing NTLMv2 authentication hashes** via forced SMB authentication (Responder)

MITRE ATT&CK: **T1557.001 – Adversary-in-the-Middle (LLMNR/NBT-NS Poisoning)**

OWASP Top 10: **A02:2021 – Cryptographic Failures**

**Cracking captured NTLMv2 hashes** to recover valid credentials

MITRE ATT&CK: **T1110 – Brute Force**

OWASP Top 10: **A07:2021 – Identification and Authentication Failures**

**Remote access via WinRM (Evil-WinRM)** using compromised administrator credentials

MITRE ATT&CK: **T1021.006 – Remote Services: WinRM**

OWASP Top 10: **A07:2021 – Identification and Authentication Failures**

**Post-exploitation enumeration** to locate and retrieve the flag

MITRE ATT&CK: **T1083 – File and Directory Discovery**

OWASP Top 10: **A01:2021 – Broken Access Control**

---

## Connectivity check

Before starting the enumeration phase, we performed a basic connectivity check using ping to verify that the target machine was reachable from our system.

<pre> ping 10.129.x.x </pre> 


The response confirmed successful communication and revealed the following:

TTL = 127

A TTL value close to 128 typically indicates that the target system is running Windows, as Windows-based operating systems commonly use an initial TTL of 128.

This information helped us infer the target operating system early in the assessment and guided our subsequent enumeration and exploitation approach.

## Enumeration
Web Enumeration

We begin by identifying the technologies used by the target:

whatweb http://10.129.x.x


Key findings:

Apache running on Windows

PHP backend

Redirect to the virtual host unika.htb

We add the domain to /etc/hosts:

10.129.x.x unika.htb


Accessing http://unika.htb reveals a PHP-based website with dynamic page loading.
