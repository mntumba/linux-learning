# 17. Network Reconnaissance

> **Difficulty:** Expert | **Module:** Hacker/Penetration Testing

---

## ⚠️ ETHICAL AND LEGAL DISCLAIMER ⚠️

> **CRITICAL WARNING — READ BEFORE PROCEEDING:**
>
> - **ALL reconnaissance techniques described here must ONLY be used on systems you personally own OR have received explicit written authorization to test.**
> - Scanning systems without permission is **illegal** under the Computer Fraud and Abuse Act (USA), Computer Misuse Act (UK), and equivalent laws worldwide.
> - Even "passive" tools can generate traffic that constitutes unauthorized access in some jurisdictions.
> - **Shodan, WHOIS, and DNS lookups on public records are generally safe; active scanning requires explicit authorization.**
> - This material is for **authorized security professionals, students in lab environments, and defensive security personnel only.**
> - When in doubt: **Do not scan. Get written permission first.**

---

## Table of Contents

1. [Reconnaissance Overview](#1-reconnaissance-overview)
2. [Passive Reconnaissance](#2-passive-reconnaissance)
3. [OSINT Techniques](#3-osint-techniques)
4. [Active Reconnaissance (Authorized Only)](#4-active-reconnaissance-authorized-only)
5. [Nmap: The Network Mapper](#5-nmap-the-network-mapper)
6. [Service Enumeration](#6-service-enumeration)
7. [Web Application Reconnaissance](#7-web-application-reconnaissance)
8. [Windows and Samba Enumeration](#8-windows-and-samba-enumeration)
9. [Email and Domain Reconnaissance](#9-email-and-domain-reconnaissance)
10. [Maltego Concepts](#10-maltego-concepts)
11. [Practice Exercises](#11-practice-exercises)

---

## 1. Reconnaissance Overview

Reconnaissance (recon) is the first phase of any penetration test—gathering as much information about a target as possible before attempting exploitation.

```
Reconnaissance Types:
┌─────────────────────────────────────────────────────┐
│ PASSIVE RECON          │ ACTIVE RECON               │
│ No direct contact       │ Direct contact with target │
│ with target systems     │ systems required           │
├─────────────────────────────────────────────────────┤
│ - WHOIS lookups         │ - Port scanning            │
│ - DNS record lookups    │ - Service enumeration      │
│ - Shodan searches       │ - Banner grabbing          │
│ - Google dorking        │ - Vulnerability scanning   │
│ - Social media research │ - OS fingerprinting        │
│ - Job postings analysis │ - Network mapping          │
└─────────────────────────────────────────────────────┘
```

```bash
# Reconnaissance goal: Answer these questions about the target:
# - What IP addresses/ranges does the target own?
# - What domains and subdomains exist?
# - What services are running on which ports?
# - What software versions are in use?
# - Who are the employees? (for social engineering scope)
# - What technologies does the web app use?
# - Are there known vulnerabilities in the technology stack?
```

---

## 2. Passive Reconnaissance

### WHOIS Lookups

```bash
# WHOIS provides domain registration information
# This is public information - no authorization needed

# Basic WHOIS lookup:
whois example.com

# WHOIS on an IP address:
whois 93.184.216.34

# Parse specific fields:
whois example.com | grep -E "Registrar:|Name Server:|Expiry"

# Python-based whois:
pip install python-whois
python3 -c "import whois; w = whois.whois('example.com'); print(w)"

# Online WHOIS alternatives:
# https://who.is
# https://lookup.icann.org
# https://www.whois.com
```

### DNS Enumeration

```bash
# DNS records reveal infrastructure information
# A     - IPv4 address
# AAAA  - IPv6 address
# MX    - Mail server
# NS    - Name server
# TXT   - Text records (SPF, DKIM, verification)
# CNAME - Canonical name (alias)
# SOA   - Start of Authority
# PTR   - Reverse DNS lookup

# Basic DNS lookups:
nslookup example.com                  # Simple A record lookup
nslookup -type=mx example.com         # Mail server records
nslookup -type=ns example.com         # Name server records
nslookup -type=txt example.com        # TXT records

# Using dig (more detailed):
dig example.com                       # A record
dig example.com MX                    # Mail records
dig example.com NS                    # Name servers
dig example.com TXT                   # TXT records
dig example.com ANY                   # All records
dig @8.8.8.8 example.com             # Query specific DNS server
dig +short example.com                # Short output format

# Reverse DNS lookup:
dig -x 93.184.216.34                  # PTR record lookup
host 93.184.216.34                    # Alternative method

# DNS zone transfer attempt (usually blocked):
dig axfr @ns1.example.com example.com
# Zone transfers reveal ALL DNS records if misconfigured
# Legitimate on your own DNS servers for testing
```

### Subdomain Enumeration (Passive)

```bash
# Passive subdomain discovery (no direct target contact):

# Certificate Transparency logs - no scanning required:
# Visit: https://crt.sh/?q=%.example.com
curl -s "https://crt.sh/?q=%.example.com&output=json" | \
  python3 -m json.tool | grep "name_value" | \
  sed 's/.*"name_value": "\(.*\)".*/\1/' | sort -u

# Using subfinder (passive mode):
apt install subfinder
subfinder -d example.com -passive -o passive_subdomains.txt

# Using amass (passive mode):
apt install amass
amass enum --passive -d example.com

# Google dorking for subdomains:
# Search: site:*.example.com
# This reveals indexed subdomains without scanning

# SecurityTrails (API-based passive lookup):
# https://securitytrails.com
```

---

## 3. OSINT Techniques

### Shodan - Search Engine for Devices

```bash
# Shodan indexes internet-connected devices
# Use Shodan to research YOUR OWN infrastructure
# or targets where you have written authorization

# Shodan CLI installation:
pip install shodan
shodan init YOUR_API_KEY

# Basic Shodan searches:
shodan search "Apache 2.4.49"           # Search by banner
shodan search "hostname:example.com"    # Search by hostname
shodan search "org:\"Target Company\""  # Search by organization
shodan search "port:22 country:US"      # SSH servers in US
shodan search "default password"        # Devices with default creds
shodan search "Webcam 7 unauthorized"   # Misconfigured webcams

# Shodan host information:
shodan host 93.184.216.34               # Info on specific IP

# Common Shodan filters:
# hostname: - hostname filter
# org:      - organization name
# port:     - specific port
# os:       - operating system
# country:  - 2-letter country code
# product:  - software product name
# version:  - software version
```

### Google Dorking

```bash
# Google dorks use advanced search operators to find information
# This is passive - uses publicly indexed pages

# Common Google dork operators:
# site:example.com          - results only from this domain
# filetype:pdf              - specific file types
# inurl:admin               - URL contains "admin"
# intitle:"index of"        - directory listings
# intext:"password"         - page content contains text
# cache:example.com         - cached version of page

# Example dorks:
# Find login pages:
# site:example.com inurl:login
# Find exposed configuration files:
# site:example.com filetype:cfg OR filetype:conf
# Find exposed documents:
# site:example.com filetype:pdf OR filetype:xls
# Find directory listings:
# site:example.com intitle:"index of"

# GHDB (Google Hacking Database):
# https://www.exploit-db.com/google-hacking-database
# Thousands of useful dorks for security research
```

### theHarvester

```bash
# theHarvester gathers emails, subdomains, hosts, and names

# Installation (included in Kali):
apt install theharvester

# Basic usage:
theHarvester -d example.com -b google
# -d: domain to search
# -b: data source (google, bing, linkedin, twitter, etc.)

# Search multiple sources:
theHarvester -d example.com -b google,bing,linkedin -l 200

# Available sources:
theHarvester --list-sources

# All available sources example:
theHarvester -d example.com -b all -l 500 -f output_report

# Save results to files:
theHarvester -d example.com -b google -f results
# Creates results.html and results.xml
```

---

## 4. Active Reconnaissance (Authorized Only)

> ⚠️ **The following techniques require explicit written authorization. Only perform on systems you own or have permission to test.**

### Host Discovery

```bash
# Ping sweep to find live hosts (authorized network only):
nmap -sn 192.168.1.0/24                    # No port scan, just discovery
nmap -sn 192.168.1.0/24 --exclude 192.168.1.1  # Exclude specific hosts

# Alternative: fping
fping -a -g 192.168.1.0/24 2>/dev/null    # -a: show alive, -g: generate range

# arping for local network:
arping -I eth0 192.168.1.1                 # ARP request to host

# netdiscover:
netdiscover -r 192.168.1.0/24             # ARP-based discovery
```

---

## 5. Nmap: The Network Mapper

### Installation

```bash
# Install nmap:
sudo apt install nmap       # Debian/Ubuntu/Kali
sudo yum install nmap       # RHEL/CentOS
brew install nmap           # macOS

# Verify installation:
nmap --version
```

### Basic Scanning

```bash
# ⚠️ Only scan systems you own or have written authorization to test

# Basic scan (top 1000 ports):
nmap 192.168.1.100

# Scan a hostname:
nmap target.lab

# Scan multiple targets:
nmap 192.168.1.100 192.168.1.101 192.168.1.102

# Scan a range:
nmap 192.168.1.1-254

# Scan a subnet:
nmap 192.168.1.0/24

# Scan from a file list:
nmap -iL targets.txt
```

### Scan Types

```bash
# SYN scan (stealth scan) - requires root:
sudo nmap -sS 192.168.1.100
# Sends SYN, waits for SYN-ACK, sends RST (never completes handshake)
# Faster and less likely to appear in application logs

# TCP Connect scan (no root required):
nmap -sT 192.168.1.100
# Completes full TCP handshake
# More detectable but works without root

# UDP scan:
sudo nmap -sU 192.168.1.100
# Slower, but reveals UDP services (DNS, SNMP, DHCP)

# UDP + TCP combined:
sudo nmap -sS -sU 192.168.1.100

# NULL scan (sends no flags):
sudo nmap -sN 192.168.1.100

# FIN scan:
sudo nmap -sF 192.168.1.100

# Xmas scan (FIN, PSH, URG flags):
sudo nmap -sX 192.168.1.100
```

### Version and OS Detection

```bash
# Service version detection:
nmap -sV 192.168.1.100
# Reveals software names and versions (e.g., Apache 2.4.49, OpenSSH 7.4)

# OS detection (requires root):
sudo nmap -O 192.168.1.100
# Guesses operating system from TCP/IP stack fingerprint

# Aggressive scan (OS + version + scripts + traceroute):
sudo nmap -A 192.168.1.100
# WARNING: Loud and easily detected - use only in authorized tests
```

### Port Ranges

```bash
# Scan specific port:
nmap -p 80 192.168.1.100

# Scan multiple ports:
nmap -p 80,443,8080,8443 192.168.1.100

# Scan port range:
nmap -p 1-1000 192.168.1.100

# Scan all 65535 ports:
nmap -p- 192.168.1.100
# This takes much longer but finds non-standard ports

# Top N most common ports:
nmap --top-ports 100 192.168.1.100
nmap --top-ports 1000 192.168.1.100

# Ports by service name:
nmap -p http,https,ssh 192.168.1.100
```

### Nmap Scripting Engine (NSE)

```bash
# Default scripts scan:
nmap -sC 192.168.1.100
# Runs safe, non-intrusive default scripts

# Run specific script:
nmap --script http-title 192.168.1.100
nmap --script banner 192.168.1.100

# Run multiple scripts:
nmap --script http-title,http-methods 192.168.1.100

# Run all scripts in a category:
nmap --script "safe" 192.168.1.100      # Safe scripts only
nmap --script "discovery" 192.168.1.100 # Discovery scripts
nmap --script "vuln" 192.168.1.100      # Vulnerability checks (⚠️ more intrusive)
nmap --script "exploit" 192.168.1.100   # Exploitation scripts (⚠️ authorized only)

# Script with arguments:
nmap --script http-brute --script-args http-brute.path=/admin 192.168.1.100

# List all available scripts:
ls /usr/share/nmap/scripts/
ls /usr/share/nmap/scripts/ | grep smb   # SMB-related scripts
```

### Output Formats

```bash
# Normal output to file:
nmap -oN scan_results.txt 192.168.1.100

# XML output (for import to other tools):
nmap -oX scan_results.xml 192.168.1.100

# Grepable output:
nmap -oG scan_results.gnmap 192.168.1.100

# All formats simultaneously:
nmap -oA scan_results 192.168.1.100
# Creates: scan_results.nmap, scan_results.xml, scan_results.gnmap

# Verbose output:
nmap -v 192.168.1.100     # Verbose
nmap -vv 192.168.1.100    # More verbose
nmap -d 192.168.1.100     # Debug output

# Parse grepable output:
grep "open" scan_results.gnmap | awk '{print $2}'
```

### Comprehensive Scan Examples

```bash
# Recommended initial scan (authorized only):
sudo nmap -sS -sV -sC -O -p- --min-rate 5000 \
  -oA full_scan 192.168.1.100

# Explanation:
# -sS: SYN scan
# -sV: version detection
# -sC: default scripts
# -O: OS detection
# -p-: all ports
# --min-rate 5000: send at least 5000 packets/sec (faster)
# -oA: output all formats

# Comprehensive web scan:
nmap -sV -sC -p 80,443,8080,8443,8000,8888 \
  --script http-title,http-methods,http-robots.txt \
  -oN web_scan.txt 192.168.1.100

# SMB vulnerability scan (EternalBlue detection):
nmap -p 445 --script smb-vuln-ms17-010 192.168.1.100

# Check for common vulnerabilities:
sudo nmap --script vuln -p 80,443,445,22,21 192.168.1.100
```

---

## 6. Service Enumeration

### Banner Grabbing

```bash
# Manual banner grabbing with netcat:
nc -v 192.168.1.100 80
# Type: HEAD / HTTP/1.0
# Then press Enter twice

# Banner grab via telnet:
telnet 192.168.1.100 25    # SMTP banner
telnet 192.168.1.100 21    # FTP banner
telnet 192.168.1.100 22    # SSH banner

# Banner grab with curl:
curl -I http://192.168.1.100     # HTTP headers (shows Server, X-Powered-By)

# Banner grab with openssl (HTTPS):
openssl s_client -connect 192.168.1.100:443
HEAD / HTTP/1.0

# Automated banner grabbing:
nmap -sV --script banner 192.168.1.100

# Netcat UDP banner grab:
nc -u 192.168.1.100 161    # SNMP

# Python banner grabber:
python3 -c "
import socket
s = socket.socket()
s.connect(('192.168.1.100', 80))
s.send(b'HEAD / HTTP/1.0\r\n\r\n')
print(s.recv(1024).decode())
"
```

### SNMP Enumeration

```bash
# SNMP (Simple Network Management Protocol) often reveals system info
# Default community strings: public, private

# Install SNMP tools:
sudo apt install snmp snmp-mibs-downloader

# SNMP walk (enumerate all OIDs):
snmpwalk -v2c -c public 192.168.1.100

# Common SNMP OIDs:
snmpget -v2c -c public 192.168.1.100 1.3.6.1.2.1.1.1.0  # System description
snmpget -v2c -c public 192.168.1.100 1.3.6.1.2.1.1.5.0  # Hostname

# Try common community strings:
for community in public private manager admin; do
  snmpwalk -v2c -c $community 192.168.1.100 2>/dev/null && echo "Found: $community"
done
```

---

## 7. Web Application Reconnaissance

### Nikto - Web Server Scanner

```bash
# Nikto scans web servers for common vulnerabilities and misconfigurations
# ⚠️ Only on authorized targets - nikto is noisy and detectable

# Install nikto:
sudo apt install nikto

# Basic scan:
nikto -h http://192.168.1.100

# Scan specific port:
nikto -h http://192.168.1.100 -p 8080

# HTTPS scan:
nikto -h https://192.168.1.100 -ssl

# Scan with authentication:
nikto -h http://192.168.1.100 -id admin:password

# Save output:
nikto -h http://192.168.1.100 -o nikto_results.txt -Format txt
nikto -h http://192.168.1.100 -o nikto_results.html -Format html

# Scan multiple hosts:
nikto -h hosts.txt
```

### Gobuster / Dirb - Directory Enumeration

```bash
# Directory busting finds hidden paths on web servers
# ⚠️ Can generate heavy traffic - authorized targets only

# Install gobuster:
sudo apt install gobuster

# Basic directory enumeration:
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt

# With file extension search:
gobuster dir -u http://192.168.1.100 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,txt,html,bak

# DNS subdomain enumeration:
gobuster dns -d example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# VHOST enumeration:
gobuster vhost -u http://192.168.1.100 -w subdomains.txt

# Save results:
gobuster dir -u http://192.168.1.100 \
  -w /usr/share/wordlists/dirb/common.txt \
  -o gobuster_results.txt

# Using dirb (alternative):
dirb http://192.168.1.100 /usr/share/wordlists/dirb/common.txt

# Using feroxbuster (faster recursive):
feroxbuster --url http://192.168.1.100 --wordlist common.txt --recursive
```

### WhatWeb and Technology Detection

```bash
# Identify web technologies:
whatweb http://192.168.1.100

# Aggressive detection:
whatweb -a 3 http://192.168.1.100

# Wappalyzer CLI alternative:
npm install -g wappalyzer
wappalyzer http://192.168.1.100

# curl to check headers manually:
curl -I http://192.168.1.100 2>&1 | grep -E "Server:|X-Powered-By:|Set-Cookie:"

# Check for common admin panels:
for path in /admin /administrator /wp-admin /phpmyadmin /cpanel /login; do
  code=$(curl -s -o /dev/null -w "%{http_code}" http://192.168.1.100$path)
  echo "$path -> HTTP $code"
done
```

---

## 8. Windows and Samba Enumeration

### Enum4linux

```bash
# Enum4linux enumerates Windows/Samba systems for information
# ⚠️ Active scanning - authorized systems only

# Install:
sudo apt install enum4linux

# Basic enumeration:
enum4linux 192.168.1.100

# Specific enumeration flags:
enum4linux -U 192.168.1.100    # Users
enum4linux -S 192.168.1.100    # Shares
enum4linux -P 192.168.1.100    # Password policy
enum4linux -G 192.168.1.100    # Groups
enum4linux -o 192.168.1.100    # OS information

# Full enumeration:
enum4linux -a 192.168.1.100

# With credentials:
enum4linux -u admin -p password 192.168.1.100

# Enum4linux-ng (updated version):
pip install enum4linux-ng
enum4linux-ng -A 192.168.1.100
```

### SMB Enumeration

```bash
# List SMB shares (null session):
smbclient -L //192.168.1.100 -N
# -L: list shares, -N: no password

# Connect to a share:
smbclient //192.168.1.100/share_name -N

# CrackMapExec SMB enumeration:
crackmapexec smb 192.168.1.0/24    # Scan subnet for SMB hosts
crackmapexec smb 192.168.1.100 --shares  # List shares
crackmapexec smb 192.168.1.100 --users   # Enumerate users

# Nmap SMB scripts:
nmap -p 445 --script smb-enum-shares,smb-enum-users 192.168.1.100
nmap -p 445 --script smb-security-mode 192.168.1.100
nmap -p 445 --script smb-os-discovery 192.168.1.100
```

---

## 9. Email and Domain Reconnaissance

### theHarvester (Advanced)

```bash
# Comprehensive email/domain harvest:
theHarvester -d targetdomain.com \
  -b google,bing,linkedin,twitter,dnsdumpster \
  -l 500 \
  -f harvest_results

# Available data sources:
# google, bing, yahoo, baidu     - Search engines
# linkedin                        - Professional network
# twitter                         - Social media
# dnsdumpster                     - DNS data
# virustotal                      - Domain intelligence
# shodan                          - Device data (API key required)
# hunter                          - Email verification (API key required)
```

### DNSRecon

```bash
# Comprehensive DNS reconnaissance:
dnsrecon -d example.com -t std    # Standard record types
dnsrecon -d example.com -t brt -D /usr/share/wordlists/dnsmap.txt  # Brute force
dnsrecon -d example.com -t axfr  # Zone transfer attempt
dnsrecon -d example.com -t rvl   # Reverse lookup

# Output to file:
dnsrecon -d example.com -t std --xml output.xml
```

---

## 10. Maltego Concepts

```bash
# Maltego is a visual link analysis and OSINT tool
# Community edition available free: https://www.maltego.com

# Maltego concepts:
# - Entities: Objects representing real-world items
#   (Domain, IP Address, Email, Person, Phone, etc.)
# - Transforms: Queries that discover relationships between entities
# - Graphs: Visual representation of relationships

# Workflow example:
# 1. Start with a domain entity: example.com
# 2. Run "To DNS Name - MX" transform -> finds mail servers
# 3. Run "To IP Address [DNS]" -> resolves to IPs
# 4. Run "To Netblock [Using ARIN]" -> finds IP ownership
# 5. Run "To Website [Quick Lookup]" -> finds related websites

# Maltego Community Edition limitations:
# - 12 results per transform
# - No access to commercial data sources
# - Good for learning, limited for professional use

# Alternative OSINT frameworks:
# recon-ng: https://github.com/lanmaster53/recon-ng
# SpiderFoot: https://github.com/smicallef/spiderfoot
recon-ng
> marketplace install all
> modules load recon/domains-hosts/google_site_web
> options set SOURCE example.com
> run
```

---

## 11. Practice Exercises

> ⚠️ **All exercises must be performed exclusively in your own isolated lab environment.**

### Exercise 1: DNS Deep Dive (Safe - Uses Public Records)

```bash
# Task: Perform comprehensive DNS reconnaissance on a domain you own
# or a CTF/practice domain like: scanme.nmap.org (Nmap's authorized scan target)

# Nmap specifically authorizes scanning scanme.nmap.org
# For all other targets, you need explicit permission

dig scanme.nmap.org ANY
nslookup -type=mx scanme.nmap.org
nslookup -type=ns scanme.nmap.org

# Compare results from different DNS servers:
dig @8.8.8.8 scanme.nmap.org A
dig @1.1.1.1 scanme.nmap.org A

# Document all records found
```

### Exercise 2: Authorized Nmap Lab Scan

```bash
# Task: Perform comprehensive nmap scan on YOUR lab VM (Metasploitable 2)
# Assuming Metasploitable is at 192.168.56.101

# Step 1: Basic discovery
nmap -sn 192.168.56.0/24

# Step 2: Full port scan
sudo nmap -sS -p- 192.168.56.101 -oN full_ports.txt

# Step 3: Version and script scan on open ports
# (Replace with open ports found in step 2)
sudo nmap -sV -sC -p 21,22,23,25,80 192.168.56.101 -oN services.txt

# Step 4: Analyze results
# - What services are running?
# - What versions? Are they outdated?
# - What vulnerabilities might exist?
```

### Exercise 3: Web Application Reconnaissance

```bash
# Task: Recon on DVWA or other intentionally vulnerable web app in your lab

# Set up DVWA on your lab server first, then:

# Identify web technologies:
whatweb http://192.168.56.101/dvwa

# Directory enumeration:
gobuster dir -u http://192.168.56.101/dvwa \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt

# Web server scan:
nikto -h http://192.168.56.101

# Document: What hidden paths were found? What vulnerabilities?
```

### Exercise 4: SMB Enumeration on Lab

```bash
# Task: Enumerate SMB on Metasploitable 2 lab target

# Discover SMB version and configuration:
nmap -p 445 --script smb-os-discovery,smb-security-mode,smb-enum-shares \
  192.168.56.101

# List shares anonymously:
smbclient -L //192.168.56.101 -N

# Enumerate with enum4linux:
enum4linux -a 192.168.56.101

# Questions to answer:
# - What shares exist?
# - What usernames were discovered?
# - What OS and SMB version is running?
```

### Exercise 5: Build a Reconnaissance Report

```bash
# Task: Compile findings from exercises 1-4 into a structured report

cat > recon_report.md << 'EOF'
# Reconnaissance Report
## Target: [Lab VM IP]
## Date: $(date)
## Tester: [Your Name]
## Authorization: Lab environment - self-owned

## Executive Summary
[Brief overview of findings]

## Open Ports and Services
| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 22   | SSH     | OpenSSH X.X | |
| 80   | HTTP    | Apache X.X | |

## Identified Vulnerabilities
[List vulnerabilities found]

## Recommendations
[How to fix each issue]
EOF

echo "Report template created: recon_report.md"
```

---

## Summary

Reconnaissance is the foundation of security testing. Key principles:

- **Passive recon** (WHOIS, DNS, Shodan) is generally legal on public data
- **Active recon** (port scanning, service enumeration) requires **explicit authorization**
- Nmap is the industry-standard tool for network reconnaissance
- Always **document everything** and **save output** for your report
- `scanme.nmap.org` is the only public host you can scan without permission
- **Written permission first, then scan** — never the reverse

---

*Previous: [16. Introduction to Ethical Hacking](16_Introduction_to_Ethical_Hacking.md) | Next: [18. Vulnerability Assessment](18_Vulnerability_Assessment.md)*
