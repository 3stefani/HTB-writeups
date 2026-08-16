# ⭐ Hack The Box - Unified ⭐

![Hack The Box](https://img.shields.io/badge/Hack%20The%20Box-Unified-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-blue?style=for-the-badge&logo=linux)
![Category](https://img.shields.io/badge/Category-Starting%20Point-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-success?style=for-the-badge)

> **Machine:** Unified  
> **Platform:** Hack The Box  
> **Difficulty:** Easy  
> **Operating System:** Linux  
> **IP:** `10.129.96.149`

---

## 📋 Machine Information

| Property | Value |
|----------|-------|
| **Machine** | Unified |
| **Difficulty** | Easy |
| **OS** | Linux |
| **IP** | `10.129.96.149` |
| **Main Service** | UniFi Network |
| **Version** | 6.4.54 |
| **Vulnerability** | CVE-2021-44228 |
| **Vulnerability Name** | Log4Shell |
| **Database** | MongoDB |
| **Database Port** | 27117 |
| **Initial Access** | Remote Code Execution |
| **Privilege Escalation** | Credential manipulation + SSH |
| **Status** | ✅ Complete |

---

# 🧭 Overview

Unified is a Linux machine based on the **UniFi Network Application**.

The attack starts with network enumeration, where port `8443` exposes the UniFi management interface. The application is identified as **UniFi Network 6.4.54**, which is vulnerable to **CVE-2021-44228 (Log4Shell)**.

By exploiting the Log4Shell vulnerability through a JNDI/LDAP payload, it is possible to obtain a reverse shell as the `unifi` user.

Once inside the system, MongoDB is discovered running locally on port `27117`. The `ace` database contains UniFi user information, including password hashes.

The administrator's credentials can be modified through MongoDB. After logging into the UniFi administration panel with the modified credentials, the **Device Authentication** section reveals credentials that can be used to connect to the machine via SSH as `root`.

### Attack Chain

```text
┌──────────────────────┐
│      Nmap Scan       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ UniFi Network 6.4.54 │
│      Port 8443       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ CVE-2021-44228       │
│     Log4Shell        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    JNDI → LDAP       │
│     RogueJNDI        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Remote Code Execution│
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Reverse Shell        │
│      unifi           │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ MongoDB :27117       │
│ Database: ace        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Modify Administrator │
│     Credentials      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ UniFi Web Panel      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Device Authentication│
│     Credentials      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ SSH → root           │
└──────────┬───────────┘
           │
           ▼
       ROOT FLAG
