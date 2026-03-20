# 16. Introduction to Ethical Hacking

> **Difficulty:** Expert | **Module:** Hacker/Penetration Testing

---

## ⚠️ ETHICAL AND LEGAL DISCLAIMER ⚠️

> **CRITICAL WARNING:** The techniques, tools, and knowledge presented in this module are provided **strictly for educational purposes** and **authorized security testing only**.
>
> - **NEVER** attempt to access, scan, or test any system, network, or application without **explicit written permission** from the owner.
> - Unauthorized hacking is a **criminal offense** in virtually every jurisdiction worldwide.
> - Violations can result in **federal prison sentences, massive fines, civil lawsuits, and permanent career destruction**.
> - The authors and contributors of this course bear **no responsibility** for misuse of this information.
> - **Always obtain written authorization before conducting any security testing.**
> - This knowledge is intended for **defenders, security professionals, and ethical researchers** only.

---

## Table of Contents

1. [What is Ethical Hacking?](#1-what-is-ethical-hacking)
2. [Types of Hackers](#2-types-of-hackers)
3. [Penetration Testing Phases](#3-penetration-testing-phases)
4. [Legal Framework](#4-legal-framework)
5. [Getting Permission](#5-getting-permission)
6. [Bug Bounty Programs](#6-bug-bounty-programs)
7. [Certifications](#7-certifications)
8. [Career Paths](#8-career-paths)
9. [Setting Up a Safe Lab](#9-setting-up-a-safe-lab)
10. [Kali Linux Introduction](#10-kali-linux-introduction)
11. [CTF Competitions](#11-ctf-competitions)
12. [OWASP Top 10 Overview](#12-owasp-top-10-overview)
13. [Ethical Hacking Tools Overview](#13-ethical-hacking-tools-overview)
14. [Practice Exercises](#14-practice-exercises)

---

## 1. What is Ethical Hacking?

**Ethical hacking** (also called penetration testing or white-hat hacking) is the authorized, legal practice of probing computer systems, networks, and applications to discover security vulnerabilities before malicious actors can exploit them.

The core distinction between ethical and malicious hacking is **permission and intent**:

| Aspect | Ethical Hacking | Malicious Hacking |
|--------|----------------|-------------------|
| Authorization | Written permission required | No permission |
| Intent | Improve security | Cause harm, steal data |
| Disclosure | Report findings to owner | Exploit or sell findings |
| Legal status | Legal and encouraged | Criminal offense |
| Outcome | Strengthened defenses | Compromised systems |

### The Ethical Hacker's Mindset

An ethical hacker must think like an attacker while acting with integrity. This requires:

```bash
# The ethical hacker's checklist before ANY engagement:
# 1. Written authorization obtained? YES/NO
# 2. Scope clearly defined? YES/NO
# 3. Rules of engagement documented? YES/NO
# 4. Emergency contacts established? YES/NO
# 5. Legal counsel reviewed contract? YES/NO
# If ANY answer is NO -> STOP. Do not proceed.
```

### Why Organizations Need Ethical Hackers

Security testing reveals vulnerabilities in a controlled manner:

```bash
# Statistics that justify ethical hacking:
# - Average cost of a data breach (2023): $4.45 million (IBM)
# - Time to identify a breach: average 197 days
# - 60% of SMBs close within 6 months of a cyberattack
# - Ethical testing costs: $5,000-$100,000+ per engagement
# - ROI: preventing ONE breach justifies years of testing costs
```

---

## 2. Types of Hackers

### White Hat Hackers
Security professionals who hack with explicit permission to improve security.

```bash
# White hat activities (all authorized):
# - Penetration testing
# - Vulnerability assessments
# - Security code reviews
# - Red team exercises
# - Bug bounty hunting
```

### Grey Hat Hackers
Operate in a moral gray area—may hack without permission but typically disclose findings rather than exploit them maliciously.

```bash
# Grey hat scenario (still potentially illegal!):
# - Discovers vulnerability in a company's website
# - Did NOT have permission to test
# - Reports the vulnerability to the company
# Note: Even well-intentioned, this can result in prosecution
# The CFAA does not have a "good intent" exception
```

### Black Hat Hackers
Malicious actors who hack for personal gain, espionage, or destruction.

```bash
# Black hat motivations (for awareness):
# - Financial gain (ransomware, credit card theft)
# - Espionage (nation-state actors)
# - Hacktivism (political/social motivation)
# - Personal grievance
# - Notoriety/ego
# Result: Criminal prosecution, imprisonment
```

### Script Kiddies
Unskilled individuals using pre-built tools without understanding them:

```bash
# Script kiddie characteristics:
# - Use existing exploits without understanding the underlying technique
# - No knowledge of how attacks work
# - Often cause significant damage accidentally
# - High prosecution rate due to poor operational security
```

---

## 3. Penetration Testing Phases

### Phase 1: Reconnaissance (Information Gathering)

```bash
# Passive reconnaissance (no direct contact with target):
whois example.com                    # Domain registration info
nslookup example.com                 # DNS lookup
dig example.com ANY                  # DNS records
theHarvester -d example.com -b google  # Email/subdomain harvesting

# Active reconnaissance (only on authorized targets):
nmap -sn 192.168.1.0/24             # Network host discovery
```

### Phase 2: Scanning and Enumeration

```bash
# Port and service scanning (authorized targets only):
nmap -sV -sC -p- target.com         # Full service scan
nmap -O target.com                   # OS detection
nikto -h http://target.com           # Web vulnerability scan
```

### Phase 3: Exploitation

```bash
# Using discovered vulnerabilities to gain access:
# (Metasploit example - authorized lab environment only)
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
run
```

### Phase 4: Post-Exploitation

```bash
# Actions after gaining access (authorized testing only):
# - Privilege escalation
# - Lateral movement
# - Data exfiltration testing
# - Persistence mechanisms (to demonstrate risk)
# - Pivoting through network
```

### Phase 5: Reporting

```bash
# Penetration test report structure:
# 1. Executive Summary
#    - Business impact of findings
#    - Risk ratings
# 2. Technical Findings
#    - Vulnerability descriptions
#    - Evidence (screenshots, logs)
#    - CVSS scores
# 3. Remediation Recommendations
#    - Prioritized fix list
#    - Short-term and long-term actions
# 4. Appendices
#    - Raw tool output
#    - Methodology details
```

---

## 4. Legal Framework

### Computer Fraud and Abuse Act (CFAA) - USA

```bash
# CFAA Key Provisions:
# 18 U.S.C. § 1030 - Covers:
# - Unauthorized access to protected computers
# - Intentional damage to computers
# - Trafficking in passwords
# 
# Penalties:
# - First offense: up to 5-10 years federal prison
# - Repeat offense: up to 20 years
# - Civil liability: damages + attorney fees
#
# Key case: Van Buren v. United States (2021)
# Narrowed definition of "exceeds authorized access"
```

### UK Computer Misuse Act 1990

```bash
# CMA Offenses:
# Section 1: Unauthorized access - up to 2 years prison
# Section 2: Access with intent to commit further offense - up to 5 years
# Section 3: Unauthorized modification - up to 10 years
# Section 3ZA: Unauthorized acts causing serious damage - life imprisonment
#
# Amendment 2015 (Serious Crime Act):
# - Added offenses for attacks on critical infrastructure
# - Life imprisonment for attacks causing significant harm
```

### Other International Laws

```bash
# International legal framework:
# EU: Directive 2013/40/EU on attacks against information systems
# Canada: Criminal Code Section 342.1 (unauthorized computer use)
# Australia: Criminal Code Act 1995, Division 477
# Germany: §202a StGB (Data espionage)
#
# Key principle: Even if legal in your country,
# testing systems in another country may violate THEIR laws
```

### The Golden Rule

```bash
# Before ANY security testing activity, ask:
echo "Do I have WRITTEN PERMISSION to test this specific system?"
echo "Is this system within the agreed SCOPE?"
echo "Am I within the agreed TIMEFRAME?"

# If any answer is "NO" or "UNSURE":
echo "STOP. Do not proceed. Seek legal guidance."
```

---

## 5. Getting Permission

### Scope Definition

```bash
# A proper scope document includes:
# - Specific IP ranges authorized for testing
# - Domains and subdomains in scope
# - Systems explicitly OUT of scope
# - Testing windows (time of day/week)
# - Allowed testing techniques
# - Prohibited techniques (e.g., DDoS, physical testing)
#
# Example scope statement:
# IN SCOPE: 192.168.10.0/24, *.testapp.example.com
# OUT OF SCOPE: 192.168.10.50 (production payment server)
# TESTING WINDOW: Monday-Friday 9pm-5am EST only
```

### Rules of Engagement (RoE)

```bash
# Rules of Engagement document must specify:
cat rules_of_engagement_template.txt
# 1. Emergency contact procedures
# 2. Incident response protocols
# 3. Data handling requirements
# 4. Report format and delivery method
# 5. Confidentiality requirements
# 6. Methods explicitly permitted/prohibited
# 7. Physical access authorization (if applicable)
# 8. Social engineering authorization (if applicable)
```

### Written Authorization

```bash
# Authorization letter must include:
# - Date range for testing
# - Specific systems authorized
# - Name and signature of authorized company representative
# - Tester name/company
# - Contact information
#
# ALWAYS carry physical AND digital copies during testing
# Store securely - this is your legal protection
```

---

## 6. Bug Bounty Programs

### HackerOne

```bash
# HackerOne platform:
# URL: https://hackerone.com
# Features:
# - 1000+ active programs
# - Rewards: $100 to $1,000,000+
# - Triaged by platform before company review
# - Reputation system for researchers
# - Verified programs (legal safe harbor)
#
# Notable programs: Uber, Twitter, US DoD, GitHub
```

### Bugcrowd

```bash
# Bugcrowd platform:
# URL: https://bugcrowd.com
# Features:
# - Managed bug bounty programs
# - Skills-based researcher matching
# - Vulnerability rating taxonomy (VRT)
# - Both public and private programs
#
# Notable programs: Tesla, Mastercard, Fitbit
```

### Synack

```bash
# Synack platform:
# URL: https://synack.com
# Features:
# - Invite-only, vetted researcher network
# - Higher-value, enterprise targets
# - Structured testing environment
# - Higher average payouts
#
# Entry requirement: Application + skills assessment
```

### Responsible Disclosure Best Practices

```bash
# When you find a vulnerability:
# 1. Document findings thoroughly (screenshots, PoC)
# 2. DO NOT access more data than necessary to prove the bug
# 3. DO NOT share with third parties
# 4. Contact security team via official channels
# 5. Allow reasonable time to fix (90 days standard)
# 6. Coordinate public disclosure timeline
# 7. Never demand payment as a condition of disclosure
```

---

## 7. Certifications

### Entry Level

```bash
# CompTIA Security+
# - Vendor-neutral fundamentals
# - Cost: ~$392
# - Validity: 3 years (CEUs required)
# - Recommended first certification
# - Covers: threats, cryptography, PKI, access control

# CompTIA Network+
# - Networking fundamentals prerequisite
# - Essential before Security+
```

### Intermediate

```bash
# Certified Ethical Hacker (CEH) - EC-Council
# - Cost: ~$1,199 (exam only)
# - Covers 20 hacking domains
# - Multiple choice format
# - Criticized for being too theoretical
# - Still valued by many employers

# CompTIA PenTest+
# - Practical penetration testing skills
# - Cost: ~$392
# - Performance-based questions
# - Good stepping stone to OSCP
```

### Advanced

```bash
# OSCP (Offensive Security Certified Professional)
# - Provider: Offensive Security
# - Cost: ~$1,499 (includes lab time)
# - 24-hour practical exam
# - Highly respected in industry
# - Requires: compromise specific machines in lab
# - Motto: "Try Harder"

# GPEN (GIAC Penetration Tester)
# - Provider: SANS/GIAC
# - Cost: ~$949 (exam only, training extra)
# - Rigorous technical content
# - Open book format
```

### Specialist Certifications

```bash
# GWAPT - Web Application Penetration Testing
# GXPN - Exploit Researcher and Advanced Penetration Tester
# CREST - UK-based, required for government contracts
# CISSP - Management-focused, 5 years experience required
# CISM - IT Security Management
```

---

## 8. Career Paths

```bash
# Cybersecurity career paths:
# 
# Offensive (Red Team):
# Junior Pentester -> Pentester -> Senior Pentester -> Red Team Lead
# Salary range: $60,000 - $200,000+
#
# Defensive (Blue Team):
# SOC Analyst -> Incident Responder -> Threat Hunter -> CISO
# Salary range: $45,000 - $250,000+
#
# Purple Team (Offensive + Defensive):
# Security Engineer -> Principal Security Engineer -> VP Security
#
# Specialized:
# - Malware Analyst
# - Forensics Investigator
# - Security Researcher (CVE hunting)
# - Bug Bounty Hunter (independent)
# - Security Consultant
# - Application Security Engineer (AppSec)
```

---

## 9. Setting Up a Safe Lab

### Virtualization Setup

```bash
# Recommended virtualization software:
# VirtualBox (free): https://www.virtualbox.org
# VMware Workstation Pro: https://www.vmware.com

# Download VirtualBox:
wget https://download.virtualbox.org/virtualbox/7.0.0/VirtualBox-7.0.pkg

# Create isolated NAT network in VirtualBox:
VBoxManage natnetwork add --netname pentestlab --network 10.10.10.0/24 --enable
VBoxManage natnetwork modify --netname pentestlab --dhcp on

# Recommended lab VMs:
# - Kali Linux (attacker)
# - Metasploitable 2/3 (vulnerable target - intentionally insecure!)
# - DVWA (Damn Vulnerable Web Application)
# - VulnHub machines (many free intentionally vulnerable VMs)
```

### Network Isolation (Critical!)

```bash
# NEVER connect vulnerable lab VMs to:
# - Your production network
# - The internet (without specific need)
# - Any network with sensitive data

# VirtualBox isolated network setup:
# File -> Host Network Manager -> Create
# Set adapter to "Host-only" for isolated lab
# This prevents lab traffic from leaving your machine

# Verify isolation:
# From Kali: ping 8.8.8.8  # Should FAIL if properly isolated
```

### Essential Lab Software

```bash
# Download intentionally vulnerable applications:
# Metasploitable 2 (Linux):
# https://sourceforge.net/projects/metasploitable/

# DVWA (Web app):
git clone https://github.com/digininja/DVWA.git

# VulnHub (many options):
# https://www.vulnhub.com

# HackTheBox (online lab):
# https://www.hackthebox.com (requires VPN)

# TryHackMe (guided labs):
# https://tryhackme.com
```

---

## 10. Kali Linux Introduction

```bash
# Kali Linux is a Debian-based distro for penetration testing
# Developed and maintained by Offensive Security
# Contains 600+ pre-installed security tools

# Installation options:
# 1. Full installation (dedicated machine/VM)
# 2. Live boot (USB drive)
# 3. WSL2 on Windows
# 4. Docker container (limited)

# First steps after Kali installation:
sudo apt update && sudo apt full-upgrade -y    # Update everything
sudo apt install kali-linux-everything         # All tools metapackage

# Key tool categories in Kali:
# Information Gathering: nmap, theHarvester, maltego
# Vulnerability Analysis: nikto, openvas, legion
# Exploitation: metasploit-framework, sqlmap, beef
# Post Exploitation: empire, crackmapexec
# Forensics: autopsy, volatility, binwalk
# Wireless: aircrack-ng, wifite, kismet
# Password Attacks: hashcat, john, hydra
# Reverse Engineering: ghidra, radare2, gdb

# Essential Kali commands:
kali-tweaks                    # Configuration helper
sudo service postgresql start  # Start Metasploit database
msfdb init                     # Initialize Metasploit database
```

---

## 11. CTF Competitions

### What are CTFs?

```bash
# Capture The Flag competitions test security skills
# Participants find hidden "flags" (text strings) to score points
# Flags typically formatted as: FLAG{some_secret_text}
# or CTF{...}, picoCTF{...}, etc.

# CTF formats:
# - Jeopardy: Categories of challenges, solve for points
# - Attack/Defense: Protect your service, attack opponents
# - King of the Hill: Maintain control of a system
```

### CTF Categories

```bash
# Common CTF challenge categories:
# 
# Web: SQL injection, XSS, cookie manipulation, SSRF
# Pwn/Binary: Buffer overflows, format strings, ROP chains
# Crypto: Classical ciphers, RSA, AES weaknesses
# Forensics: File analysis, steganography, memory dumps
# Reverse Engineering: Disassemble/decompile binaries
# OSINT: Open source intelligence gathering
# Misc: Anything else (trivia, programming challenges)
```

### CTF Platforms

```bash
# Practice platforms:
# HackTheBox: https://hackthebox.com (realistic machines)
# TryHackMe: https://tryhackme.com (guided learning)
# PicoCTF: https://picoctf.org (beginner friendly)
# CTFtime: https://ctftime.org (list of upcoming competitions)
# OverTheWire: https://overthewire.org (wargames)
# PentesterLab: https://pentesterlab.com

# Starting a CTF challenge:
# 1. Connect to VPN (for HackTheBox/TryHackMe)
openvpn --config lab.ovpn

# 2. Enumerate the target
nmap -sV -sC -p- 10.10.10.X -oN initial_scan.txt

# 3. Research, exploit, escalate privileges
# 4. Find and submit flags
```

---

## 12. OWASP Top 10 Overview

```bash
# OWASP Top 10 (2021) - Most Critical Web App Security Risks:
#
# A01: Broken Access Control (was A05 in 2017)
# - Users can act outside their intended permissions
# Example: Changing URL parameter to access another user's data
# ?user_id=1234 -> ?user_id=1235
#
# A02: Cryptographic Failures
# - Weak encryption, sensitive data exposure
# Example: MD5 hashed passwords, unencrypted HTTP
#
# A03: Injection (SQL, LDAP, OS Command injection)
# Example SQL injection: ' OR '1'='1
#
# A04: Insecure Design
# - Flaws in architecture, not just implementation
#
# A05: Security Misconfiguration
# - Default credentials, exposed admin panels
#
# A06: Vulnerable and Outdated Components
# - Using libraries/frameworks with known vulnerabilities
#
# A07: Identification and Authentication Failures
# - Weak passwords, improper session management
#
# A08: Software and Data Integrity Failures
# - CI/CD pipeline attacks, insecure deserialization
#
# A09: Security Logging and Monitoring Failures
# - Not detecting, logging, or responding to breaches
#
# A10: Server-Side Request Forgery (SSRF)
# - Server fetches resources from attacker-controlled URL
```

---

## 13. Ethical Hacking Tools Overview

```bash
# Information Gathering:
nmap          # Network mapper - port/service scanning
theHarvester  # Email, subdomain, host gathering
shodan        # Search engine for internet-connected devices
maltego       # Visual link analysis
recon-ng      # Web reconnaissance framework
whois         # Domain registration lookup
dig           # DNS interrogation

# Vulnerability Scanning:
nikto         # Web server vulnerability scanner
openvas       # Full-featured vulnerability scanner
nessus        # Commercial vulnerability scanner
wapiti        # Web application vulnerability scanner

# Exploitation:
metasploit    # Penetration testing framework
sqlmap        # Automated SQL injection tool (authorized only)
burpsuite     # Web application testing proxy
beef          # Browser exploitation framework

# Password Attacks:
hashcat       # GPU-accelerated password cracker
john          # John the Ripper password cracker
hydra         # Online password brute-forcer
medusa        # Fast parallel password cracker

# Post-Exploitation:
mimikatz      # Windows credential extraction
empire        # PowerShell post-exploitation framework
meterpreter   # Metasploit payload for post-exploitation

# Forensics:
autopsy       # Digital forensics GUI
volatility    # Memory forensics framework
wireshark     # Network packet analyzer
tcpdump       # Command-line packet capture
```

---

## 14. Practice Exercises

> ⚠️ **All exercises must be performed in an isolated lab environment on systems you own.**

### Exercise 1: Set Up Your Lab Environment
```bash
# Task: Create an isolated penetration testing lab
# 1. Install VirtualBox
# 2. Download and import Kali Linux OVA
# 3. Download Metasploitable 2
# 4. Create Host-Only network adapter
# 5. Verify isolation (Kali cannot reach internet, can reach Metasploitable)
# Expected: Two VMs communicating on isolated network
```

### Exercise 2: Explore Kali Linux Tools
```bash
# Task: Familiarize yourself with Kali Linux
# 1. Open Kali Linux terminal
# 2. List all installed tools in a category:
ls /usr/share/nmap/scripts/ | head -20    # Nmap scripts
ls /usr/share/metasploit-framework/modules/exploits/ | head -10

# 3. Check Metasploit database status:
sudo service postgresql start
msfdb status
msfconsole -q -x "db_status; exit"
```

### Exercise 3: Research OWASP Top 10
```bash
# Task: Research each OWASP Top 10 vulnerability
# 1. Visit: https://owasp.org/www-project-top-ten/
# 2. For each vulnerability, document:
#    - Description
#    - Real-world example
#    - Prevention technique
# 3. Create a summary document

# Install DVWA in your lab for hands-on practice:
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo chmod -R 777 DVWA/
# Configure DVWA database settings
```

### Exercise 4: Bug Bounty Reconnaissance (on authorized programs only)
```bash
# Task: Practice passive reconnaissance on a bug bounty target
# 1. Create account on HackerOne: https://hackerone.com
# 2. Find a program that allows subdomain enumeration
# 3. Perform ONLY passive recon:
whois target.com
dig target.com ANY
# Use subfinder (passive only):
subfinder -d target.com -o subdomains.txt
# Review findings without active scanning
```

### Exercise 5: Plan a Penetration Test
```bash
# Task: Create a penetration test plan document
# Imagine you've been hired to test a fictional company
# Create documents for:
# 1. Statement of Work (SOW)
# 2. Scope definition
# 3. Rules of Engagement
# 4. Testing methodology
# 5. Report template
#
# Use this template:
cat > pentest_plan.md << 'EOF'
# Penetration Test Plan
## Client: FICTIONAL_COMPANY
## Tester: YOUR_NAME
## Date: $(date)
## Scope: 192.168.100.0/24 (lab only)
## Rules of Engagement:
- No DDoS
- Testing window: 9pm-5am
- Emergency contact: admin@fictional.lab
EOF
```

---

## Summary

Ethical hacking is a critical discipline that requires technical skill, legal knowledge, and unwavering integrity. Key takeaways:

- **Always get written permission** before any security testing
- Understand the **legal frameworks** that govern your jurisdiction
- Follow the **5 phases** of penetration testing methodically
- Build skills safely through **CTFs and lab environments**
- Pursue **certifications** to validate and advance your skills
- The goal is always to **improve security**, never to cause harm

---

*Next: [17. Network Reconnaissance](17_Network_Reconnaissance.md)*
