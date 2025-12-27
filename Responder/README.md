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
Identifying a Local File Inclusion (LFI) vulnerability via a dynamic page parameter

Escalating LFI to Remote File Inclusion (RFI) using UNC paths on a Windows host

Capturing NTLMv2 hashes by forcing authentication to a malicious SMB server (Responder)

Cracking the captured hash using John the Ripper

Gaining remote administrative access through WinRM with Evil-WinRM

Enumerating the filesystem to locate and retrieve the flag

---

## Enumeration
