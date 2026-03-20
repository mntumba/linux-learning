# Appendix B: Aliases Guide & Acronyms Reference

---

## Table of Contents

**Part 1: Aliases Guide**
1. [What Are Aliases?](#1-what-are-aliases)
2. [Creating Aliases](#2-creating-aliases)
3. [Permanent Aliases](#3-permanent-aliases)
4. [Managing Aliases](#4-managing-aliases)
5. [Complete Alias Collection (80+)](#5-complete-alias-collection)

**Part 2: Acronyms Reference**
6. [A–C](#a-c)
7. [D–F](#d-f)
8. [G–I](#g-i)
9. [J–L](#j-l)
10. [M–O](#m-o)
11. [P–R](#p-r)
12. [S–T](#s-t)
13. [U–Z](#u-z)

---

# Part 1: Aliases Guide

## 1. What Are Aliases?

An **alias** is a user-defined shortcut that replaces a longer command or adds default options to existing commands. Aliases:

- **Save typing** — `ll` instead of `ls -lah --color=auto`
- **Prevent mistakes** — `rm='rm -i'` prompts before deletion
- **Standardize workflow** — consistent shortcuts across teams
- **Improve safety** — guard dangerous commands with confirmation

```bash
# Without alias:
ls -lah --color=auto --group-directories-first

# With alias:
alias ll='ls -lah --color=auto --group-directories-first'
ll   # runs the full command
```

---

## 2. Creating Aliases

### Temporary Aliases (Current Session Only)

```bash
# Basic syntax
alias name='command'

# Examples
alias ll='ls -lah'
alias ..='cd ..'
alias grep='grep --color=auto'

# Alias with arguments (use functions for dynamic args)
alias today='date +"%A, %B %d, %Y"'
```

> **Note:** Aliases do not accept arguments directly. Use shell functions for that:
> ```bash
> mkcd() { mkdir -p "$1" && cd "$1"; }
> ```

### Checking Existing Aliases

```bash
alias             # list all aliases
alias ll          # show definition of specific alias
type ll           # show type (alias, function, builtin, file)
```

---

## 3. Permanent Aliases

To persist aliases across sessions, add them to a startup file:

### Option A: `~/.bashrc` (Bash)

```bash
# ~/.bashrc
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade -y'
```

Then reload:
```bash
source ~/.bashrc
# or
. ~/.bashrc
```

### Option B: `~/.bash_aliases` (Recommended for Bash)

Create a dedicated file:
```bash
# ~/.bash_aliases
alias ll='ls -lah'
alias la='ls -A'
```

Ensure `~/.bashrc` includes it (usually present by default):
```bash
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

### Option C: `~/.zshrc` (Zsh)

```bash
# ~/.zshrc
alias ll='ls -lah'
alias gst='git status'
```

### Option D: System-wide (All Users)

```bash
# /etc/bash.bashrc  or  /etc/profile.d/aliases.sh
alias ll='ls -lah'
```

---

## 4. Managing Aliases

```bash
# Remove a single alias
unalias ll

# Remove ALL aliases
unalias -a

# List all aliases (portable)
alias -p

# Temporarily bypass an alias (use backslash or command)
\ls             # runs real ls, not alias
command ls      # same effect
'ls'            # quotes bypass alias too

# Check if something is an alias
type -a ls
alias ls 2>/dev/null && echo "is an alias"
```

---

## 5. Complete Alias Collection

Copy this entire block into `~/.bash_aliases` or `~/.zshrc`:

```bash
#================================================================
# ~/.bash_aliases — Complete Alias Collection
#================================================================

# ── SAFETY ALIASES ──────────────────────────────────────────────
alias rm='rm -i'                     # prompt before delete
alias cp='cp -i'                     # prompt before overwrite
alias mv='mv -i'                     # prompt before overwrite
alias ln='ln -i'                     # prompt before overwrite
alias mkdir='mkdir -pv'              # create parents, verbose
alias chown='chown --preserve-root'  # prevent chown on /
alias chmod='chmod --preserve-root'  # prevent chmod on /
alias chgrp='chgrp --preserve-root'  # prevent chgrp on /

# ── NAVIGATION ──────────────────────────────────────────────────
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias .....='cd ../../../..'
alias ~='cd ~'
alias -- -='cd -'                    # go to previous directory
alias home='cd ~'
alias root='cd /'
alias downloads='cd ~/Downloads'
alias desktop='cd ~/Desktop'
alias projects='cd ~/projects'

# ── FILE LISTING ────────────────────────────────────────────────
alias ls='ls --color=auto'
alias ll='ls -lah --group-directories-first'
alias la='ls -A'                     # all except . and ..
alias l='ls -CF'
alias lt='ls -lath'                  # sort by time
alias lS='ls -laSh'                  # sort by size
alias lx='ls -laXB'                  # sort by extension
alias l.='ls -d .* --color=auto'    # hidden files only
alias dir='ls -l --color=auto'
alias vdir='ls -la --color=auto'

# ── GREP ────────────────────────────────────────────────────────
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias rg='rg --color=auto'           # ripgrep with color
alias grepcase='grep -i'             # case insensitive

# ── SYSTEM UPDATE ────────────────────────────────────────────────
alias update='sudo apt update'
alias upgrade='sudo apt update && sudo apt upgrade -y'
alias install='sudo apt install'
alias remove='sudo apt remove'
alias purge='sudo apt purge'
alias autoremove='sudo apt autoremove --purge'
alias search='apt search'
alias showpkg='apt show'
alias dist-upgrade='sudo apt dist-upgrade'
alias fix-broken='sudo apt --fix-broken install'

# ── SYSTEM INFO ─────────────────────────────────────────────────
alias meminfo='free -h'
alias cpuinfo='lscpu'
alias diskinfo='df -h'
alias diskusage='du -sh *'           # size of items in current dir
alias myip='curl -4 icanhazip.com'  # public IPv4
alias localip="ip route get 1 | awk '{print \$7}' | head -1"
alias ports='ss -tulnp'             # listening ports
alias connections='ss -tnp'         # established connections
alias topcpu='ps aux --sort=-%cpu | head -15'
alias topmem='ps aux --sort=-%mem | head -15'
alias psmem='ps auxf | sort -nr -k 4 | head -10'
alias pscpu='ps auxf | sort -nr -k 3 | head -10'
alias resources='top -b -n1 | head -30'

# ── NETWORKING ──────────────────────────────────────────────────
alias ping='ping -c 5'
alias fastping='ping -c 100 -i 0.2'
alias pping='ping -c 3 8.8.8.8'    # ping Google DNS
alias tracert='traceroute'
alias flush-dns='sudo resolvectl flush-caches'
alias interfaces='ip -s link'
alias routes='ip route show'
alias arp-table='arp -n'
alias open-ports='sudo nmap -sT -O localhost'
alias headers='curl -I'

# ── PROCESS & MONITORING ───────────────────────────────────────
alias psg='ps aux | grep -v grep | grep -i'
alias pstree='pstree -p'
alias killall9='killall -9'
alias free='free -h'
alias watch='watch -d -n2'           # highlight changes, 2s interval
alias iotop='sudo iotop'
alias nethogs='sudo nethogs'
alias cpufreq='watch -n1 "cat /proc/cpuinfo | grep MHz"'

# ── HISTORY ─────────────────────────────────────────────────────
alias h='history'
alias h10='history 10'
alias h20='history 20'
alias hgrep='history | grep'
alias histfile='cat ~/.bash_history'
alias clearhist='cat /dev/null > ~/.bash_history && history -c'

# ── GIT ─────────────────────────────────────────────────────────
alias gs='git status'
alias ga='git add'
alias gaa='git add --all'
alias gc='git commit'
alias gcm='git commit -m'
alias gca='git commit --amend'
alias gp='git push'
alias gpf='git push --force-with-lease'
alias gpl='git pull'
alias gl='git log --oneline --graph --decorate'
alias gll='git log --all --oneline --graph --decorate'
alias gd='git diff'
alias gds='git diff --staged'
alias gb='git branch'
alias gba='git branch -a'
alias gco='git checkout'
alias gcb='git checkout -b'
alias gm='git merge'
alias gr='git remote -v'
alias gst='git stash'
alias gstp='git stash pop'
alias gstl='git stash list'
alias gclean='git clean -fd'
alias gunstage='git restore --staged'
alias greset='git reset HEAD~1'
alias gtag='git tag -l'

# ── DOCKER ──────────────────────────────────────────────────────
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dlog='docker logs -f'
alias drun='docker run -it'
alias dstop='docker stop'
alias drm='docker rm'
alias drmi='docker rmi'
alias dprune='docker system prune -f'
alias dip="docker inspect --format='{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}'"
alias dc='docker compose'
alias dcu='docker compose up -d'
alias dcd='docker compose down'
alias dcl='docker compose logs -f'

# ── PYTHON ──────────────────────────────────────────────────────
alias python='python3'
alias pip='pip3'
alias venv='python3 -m venv'
alias activate='source venv/bin/activate'
alias pyserver='python3 -m http.server'
alias json='python3 -m json.tool'

# ── EDITOR & FILES ───────────────────────────────────────────────
alias edit='$EDITOR'
alias nano='nano -c'                 # show line numbers in nano
alias vi='vim'
alias svi='sudo vim'
alias cls='clear'
alias c='clear'
alias q='exit'
alias x='exit'
alias :q='exit'                      # for vim muscle memory

# ── CONVENIENCE ─────────────────────────────────────────────────
alias please='sudo'
alias sudo='sudo '                   # allows aliases with sudo
alias fuck='sudo $(history -p \!\!)'  # redo last command as sudo
alias reload='source ~/.bashrc'
alias path='echo $PATH | tr ":" "\n"'
alias now='date +"%T"'
alias today='date +"%Y-%m-%d"'
alias timestamp='date +%s'
alias week='date +%V'
alias countdown='for i in $(seq 10 -1 1); do echo $i; sleep 1; done; echo Go!'

# ── DISK & FILES ────────────────────────────────────────────────
alias duf='df -h | grep -E "^/dev|Filesystem"'
alias biggest='du -h --max-depth=1 | sort -rh | head -10'
alias findlarge='find . -size +100M -exec ls -lh {} \;'
alias countfiles='find . -type f | wc -l'

# ── WEATHER & FUN ───────────────────────────────────────────────
alias weather='curl wttr.in'
alias moon='curl wttr.in/Moon'
alias matrix='cmatrix'
alias ascii='figlet'

# ── CUSTOM FUNCTIONS (alongside aliases) ────────────────────────
# Create and enter directory
mkcd() { mkdir -p "$1" && cd "$1"; }

# Extract any archive
extract() {
    case "$1" in
        *.tar.bz2)   tar xjf "$1"    ;;
        *.tar.gz)    tar xzf "$1"    ;;
        *.tar.xz)    tar xJf "$1"    ;;
        *.bz2)       bunzip2 "$1"    ;;
        *.gz)        gunzip "$1"     ;;
        *.tar)       tar xf "$1"     ;;
        *.zip)       unzip "$1"      ;;
        *.7z)        7z x "$1"       ;;
        *)           echo "Unknown format: $1" ;;
    esac
}

# Quick backup of a file
bak() { cp "$1"{,.bak-$(date +%F)}; }

# Find in files
ff() { grep -rn "$1" . 2>/dev/null; }

# Swap two files
swap() {
    local tmp=$(mktemp)
    mv "$1" "$tmp" && mv "$2" "$1" && mv "$tmp" "$2"
}
```

---

---

# Part 2: Acronyms Reference

> **300+ Linux, networking, security, and development acronyms** with full explanations.

---

## A–C

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **ABI** | Application Binary Interface | Low-level interface between binary programs; defines calling conventions, data structures |
| **ACL** | Access Control List | Extended file permission system; allows per-user/group permissions beyond standard rwx |
| **AES** | Advanced Encryption Standard | Symmetric block cipher; industry standard (128/192/256-bit); used in SSL/TLS, disk encryption |
| **ARP** | Address Resolution Protocol | Maps IP addresses to MAC addresses on local networks; Layer 2/3 boundary |
| **APT** | Advanced Package Tool | Debian/Ubuntu package manager; handles dependencies automatically |
| **APT** | Advanced Persistent Threat | Long-term targeted cyberattack, often state-sponsored |
| **API** | Application Programming Interface | Interface for software components to communicate; defines endpoints, methods, data formats |
| **AS** | Autonomous System | Collection of IP networks under single routing policy; assigned an ASN |
| **ASN** | Autonomous System Number | Unique number identifying an AS; used in BGP routing |
| **ATT&CK** | Adversarial Tactics, Techniques & Common Knowledge | MITRE framework for cyber adversary behavior |
| **AWS** | Amazon Web Services | Cloud computing platform by Amazon |
| **AZ** | Availability Zone | Isolated location within AWS/Azure region |
| **BDD** | Behavior-Driven Development | TDD evolution focusing on behavioral specifications in plain language |
| **BGP** | Border Gateway Protocol | Inter-AS routing protocol; the "routing protocol of the internet" |
| **BIOS** | Basic Input/Output System | Firmware initializing hardware at boot; predecessor to UEFI |
| **BSD** | Berkeley Software Distribution | Unix variant from UC Berkeley; basis for macOS, FreeBSD, OpenBSD |
| **BTRFS** | B-Tree File System | Linux copy-on-write filesystem with snapshotting, RAID capabilities |
| **C2** | Command and Control | Infrastructure used by attackers to communicate with compromised systems |
| **CA** | Certificate Authority | Trusted entity issuing digital certificates (SSL/TLS) |
| **CDN** | Content Delivery Network | Geographically distributed servers to deliver content with low latency |
| **CEH** | Certified Ethical Hacker | Security certification from EC-Council |
| **CIDR** | Classless Inter-Domain Routing | IP addressing notation: 192.168.0.0/24 (24-bit prefix mask) |
| **CIFS** | Common Internet File System | Microsoft network file sharing protocol; extension of SMB |
| **CIS** | Center for Internet Security | Non-profit producing security benchmarks |
| **CISSP** | Certified Information Systems Security Professional | ISC² advanced security certification |
| **CLI** | Command Line Interface | Text-based interface for interacting with the OS |
| **CORS** | Cross-Origin Resource Sharing | HTTP mechanism allowing cross-origin resource requests |
| **CPU** | Central Processing Unit | Primary processor handling computation |
| **CSRF** | Cross-Site Request Forgery | Attack tricking users into unwanted actions on authenticated sites |
| **CSR** | Certificate Signing Request | Request to CA for issuing a digital certificate |
| **CVE** | Common Vulnerabilities and Exposures | Standard identifier for publicly known security vulnerabilities |
| **CVSS** | Common Vulnerability Scoring System | Framework for rating security vulnerability severity (0–10 scale) |
| **CWE** | Common Weakness Enumeration | List of software security weakness types |

---

## D–F

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **DAST** | Dynamic Application Security Testing | Testing running applications for vulnerabilities |
| **DDoS** | Distributed Denial of Service | Overwhelming a target with traffic from multiple sources |
| **DES** | Data Encryption Standard | Symmetric cipher; 56-bit key; deprecated (insecure) |
| **3DES** | Triple DES | Applies DES three times; stronger than DES but slower than AES |
| **DHCP** | Dynamic Host Configuration Protocol | Automatically assigns IP addresses to network devices |
| **DLP** | Data Loss Prevention | Technologies preventing unauthorized data exfiltration |
| **DMZ** | Demilitarized Zone | Network segment between public internet and internal network |
| **DNS** | Domain Name System | Translates domain names to IP addresses |
| **DNSSEC** | DNS Security Extensions | Adds cryptographic signatures to DNS records |
| **DoH** | DNS over HTTPS | Encrypts DNS queries using HTTPS protocol |
| **DoS** | Denial of Service | Attack exhausting resources to make service unavailable |
| **DoT** | DNS over TLS | Encrypts DNS queries using TLS protocol |
| **DRY** | Don't Repeat Yourself | Coding principle: avoid duplicate code/logic |
| **EBS** | Elastic Block Store | AWS persistent block storage for EC2 |
| **EC2** | Elastic Compute Cloud | AWS virtual machine (instance) service |
| **EDR** | Endpoint Detection and Response | Security platform monitoring and responding to endpoint threats |
| **EFI** | Extensible Firmware Interface | Interface between firmware and OS; basis of UEFI |
| **EXT4** | Fourth Extended Filesystem | Default Linux filesystem; journaled, supports large files |
| **FAT32** | File Allocation Table 32 | Legacy filesystem; max 4GB file, 2TB partition; wide compatibility |
| **FHS** | Filesystem Hierarchy Standard | Defines directory structure and contents in Linux |
| **FP** | Functional Programming | Programming paradigm using functions as first-class values |
| **FOSS** | Free and Open Source Software | Software that is both free (libre) and has publicly available source code |
| **FQDN** | Fully Qualified Domain Name | Complete domain name: www.example.com (includes all labels) |
| **FTP** | File Transfer Protocol | Protocol for transferring files; unencrypted (port 21) |
| **FTPS** | FTP Secure | FTP with SSL/TLS encryption |

---

## G–I

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **GDPR** | General Data Protection Regulation | EU privacy law governing personal data handling |
| **GID** | Group Identifier | Numeric ID for a group in Unix/Linux |
| **Git** | Global Information Tracker | Distributed version control system by Linus Torvalds |
| **GNU** | GNU's Not Unix | Recursive acronym; free OS project by Richard Stallman |
| **GPL** | GNU General Public License | Copyleft license requiring derivative works to remain open source |
| **LGPL** | GNU Lesser General Public License | GPL variant allowing linking from proprietary software |
| **GPG** | GNU Privacy Guard | Free implementation of PGP; used for encryption and signing |
| **GPT** | GUID Partition Table | Modern disk partitioning standard; replaces MBR; supports 128+ partitions |
| **GPU** | Graphics Processing Unit | Processor for graphics and parallel computation |
| **GRUB** | GRand Unified Bootloader | Linux bootloader managing OS selection at startup |
| **GUI** | Graphical User Interface | Visual interface using windows, icons, menus |
| **HDD** | Hard Disk Drive | Magnetic storage device with spinning platters |
| **HMAC** | Hash-based Message Authentication Code | Cryptographic authentication using hash + shared key |
| **HTTP** | HyperText Transfer Protocol | Foundation of web data communication; port 80 |
| **HTTPS** | HTTP Secure | HTTP over SSL/TLS; port 443 |
| **HTTP2** | HTTP version 2 | Binary protocol with multiplexing, header compression |
| **HTTP3** | HTTP version 3 | HTTP over QUIC (UDP-based transport) |
| **IaaS** | Infrastructure as a Service | Cloud model providing virtualized computing resources |
| **IDOR** | Insecure Direct Object Reference | Access control vulnerability exposing internal objects |
| **IDS** | Intrusion Detection System | Monitors network/system for malicious activity |
| **IMAP** | Internet Message Access Protocol | Email retrieval protocol; port 143/993; keeps mail on server |
| **IOC** | Indicator of Compromise | Evidence of a security breach (IP, hash, domain, etc.) |
| **IPS** | Intrusion Prevention System | IDS that also blocks detected threats |
| **IP** | Internet Protocol | Network layer protocol for addressing and routing packets |
| **IPv4** | Internet Protocol version 4 | 32-bit addresses; ~4.3 billion addresses; e.g., 192.168.1.1 |
| **IPv6** | Internet Protocol version 6 | 128-bit addresses; e.g., 2001:db8::1; solves IPv4 exhaustion |
| **ISP** | Internet Service Provider | Company providing internet access to customers |
| **ISO 27001** | ISO/IEC Information Security Standard | International standard for information security management systems |

---

## J–L

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **JWT** | JSON Web Token | Compact URL-safe token for transmitting claims (auth/authorization) |
| **K8s** | Kubernetes | Container orchestration platform (8 letters between K and s) |
| **KISS** | Keep It Simple, Stupid | Design principle favoring simplicity |
| **LAN** | Local Area Network | Network covering a small geographic area (home, office) |
| **LFI** | Local File Inclusion | Web vulnerability reading local files via path traversal |
| **LGPL** | Lesser General Public License | See G section |
| **LTS** | Long-Term Support | Distribution version with extended maintenance (e.g., Ubuntu LTS = 5yr) |
| **LV** | Logical Volume | LVM abstraction layer above physical/virtual storage |
| **LVM** | Logical Volume Manager | Linux storage abstraction providing flexible disk management |
| **ls** | List (from 'listdir') | Command to list directory contents |
| **ln** | Link | Command to create hard or symbolic links |

---

## M–O

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **MAC** | Media Access Control | Unique hardware address of a network interface (e.g., aa:bb:cc:dd:ee:ff) |
| **MAC** | Mandatory Access Control | Security model where OS enforces access policies (SELinux, AppArmor) |
| **MAN** | Metropolitan Area Network | Network spanning a city or campus |
| **MBR** | Master Boot Record | Legacy disk boot sector; supports disks up to 2TB, 4 primary partitions |
| **MD5** | Message Digest 5 | Cryptographic hash function; 128-bit output; deprecated for security use |
| **MDR** | Managed Detection and Response | Outsourced security monitoring and incident response |
| **MFA** | Multi-Factor Authentication | Authentication using 2+ factors (something you know/have/are) |
| **MITRE** | MITRE Corporation | US non-profit maintaining CVE, ATT&CK, and other security frameworks |
| **MSS** | Maximum Segment Size | Largest TCP data segment negotiated during handshake |
| **mTLS** | Mutual TLS | TLS where both client and server authenticate with certificates |
| **MTU** | Maximum Transmission Unit | Largest packet size a network link can transmit |
| **MVC** | Model-View-Controller | UI design pattern separating data, display, and control logic |
| **MVVM** | Model-View-ViewModel | MVC variant popular in data-binding frameworks |
| **NAT** | Network Address Translation | Maps private IPs to public IP(s); enables internet access for private networks |
| **NFS** | Network File System | Protocol for sharing files over a network (Unix/Linux native) |
| **NIC** | Network Interface Card | Hardware providing network connectivity |
| **NIST** | National Institute of Standards and Technology | US standards body for cybersecurity frameworks |
| **NoSQL** | Not Only SQL | Non-relational databases (MongoDB, Redis, Cassandra) |
| **NTP** | Network Time Protocol | Protocol for clock synchronization over networks; port 123 |
| **NVD** | National Vulnerability Database | US government repository of CVE vulnerability data |
| **NVMe** | Non-Volatile Memory Express | Fast interface for SSDs over PCIe |
| **NTFS** | New Technology File System | Default Windows filesystem; supports large files, permissions, journaling |
| **OOM** | Out of Memory | Condition where system exhausts RAM; triggers OOM killer in Linux |
| **OIDC** | OpenID Connect | Identity layer on top of OAuth 2.0 for authentication |
| **OOP** | Object-Oriented Programming | Programming paradigm using objects and classes |
| **ORM** | Object-Relational Mapping | Maps database tables to programming language objects |
| **OSCP** | Offensive Security Certified Professional | Hands-on penetration testing certification by Offensive Security |
| **OSS** | Open Source Software | Software with publicly available source code |
| **OSPF** | Open Shortest Path First | Link-state routing protocol for internal networks |
| **OWASP** | Open Web Application Security Project | Non-profit producing web security resources (Top 10 list) |
| **OAuth** | Open Authorization | Standard protocol for delegated authorization |

---

## P–R

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **PaaS** | Platform as a Service | Cloud model providing platform for developing/deploying apps |
| **PAN** | Personal Area Network | Very small network (Bluetooth, USB) |
| **PAT** | Port Address Translation | NAT variant mapping multiple private IPs to one public IP using ports |
| **PCIe** | Peripheral Component Interconnect Express | High-speed serial bus for connecting hardware components |
| **PGP** | Pretty Good Privacy | Encryption program for data/email; basis for OpenPGP standard |
| **PID** | Process Identifier | Unique integer assigned to each running process |
| **PKI** | Public Key Infrastructure | Framework managing digital certificates and encryption keys |
| **POP3** | Post Office Protocol 3 | Email retrieval protocol; downloads mail; port 110/995 |
| **POSIX** | Portable Operating System Interface | IEEE standard for Unix compatibility |
| **PPA** | Personal Package Archive | Ubuntu/Launchpad user-hosted package repository |
| **PPID** | Parent Process Identifier | PID of a process's parent process |
| **PTY** | Pseudo-Terminal | Software terminal emulating hardware terminal |
| **PV** | Physical Volume | LVM physical disk/partition |
| **QoS** | Quality of Service | Network mechanisms prioritizing traffic types |
| **RAID** | Redundant Array of Independent Disks | Combining multiple disks for performance and/or redundancy |
| **RAM** | Random Access Memory | Volatile, fast memory for active data |
| **RARP** | Reverse ARP | Maps MAC addresses to IP addresses (predecessor to DHCP) |
| **RAT** | Remote Access Trojan | Malware enabling unauthorized remote control |
| **RCE** | Remote Code Execution | Vulnerability allowing attacker to execute code on target |
| **RFI** | Remote File Inclusion | Web vulnerability loading remote malicious files |
| **REST** | Representational State Transfer | Architectural style for web APIs using HTTP methods |
| **RHCSA** | Red Hat Certified System Administrator | Red Hat's foundation Linux admin certification |
| **RHCE** | Red Hat Certified Engineer | Advanced Red Hat Linux certification |
| **ROM** | Read-Only Memory | Non-volatile memory storing firmware |
| **RSA** | Rivest–Shamir–Adleman | Asymmetric cryptographic algorithm; key sizes 2048–4096 bits |

---

## S–T

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **SaaS** | Software as a Service | Cloud model delivering applications over the internet |
| **SAML** | Security Assertion Markup Language | XML-based standard for SSO authentication |
| **SAST** | Static Application Security Testing | Analyzing source code for vulnerabilities without execution |
| **SATA** | Serial ATA | Interface standard for connecting HDDs/SSDs to motherboard |
| **SCP** | Secure Copy Protocol | Transfers files over SSH; uses same authentication |
| **SDK** | Software Development Kit | Tools/libraries for building applications on a platform |
| **sed** | Stream Editor | Unix utility for parsing and transforming text |
| **SFTP** | SSH File Transfer Protocol | Secure file transfer subsystem of SSH (not FTP over SSL) |
| **SHA** | Secure Hash Algorithm | Cryptographic hash family (SHA-1 deprecated; SHA-256/SHA-3 current) |
| **SIEM** | Security Information and Event Management | Platform aggregating and analyzing security logs |
| **SLA** | Service Level Agreement | Contract defining expected service quality metrics |
| **SMB** | Server Message Block | Microsoft network protocol for file/printer sharing |
| **SMTP** | Simple Mail Transfer Protocol | Protocol for sending email; port 25/587/465 |
| **SOC** | Security Operations Center | Team monitoring and responding to security events |
| **SOC2** | Service Organization Control 2 | Auditing standard for service provider security controls |
| **SOAP** | Simple Object Access Protocol | XML-based web services protocol |
| **SOLID** | Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion | Five OOP design principles |
| **SQL** | Structured Query Language | Language for relational database queries |
| **SQLi** | SQL Injection | Attack inserting malicious SQL into queries |
| **SRE** | Site Reliability Engineering | Google-originated discipline combining software and operations |
| **SSD** | Solid State Drive | Non-volatile flash storage; faster than HDD |
| **SSH** | Secure Shell | Encrypted remote access protocol; port 22 |
| **SSL** | Secure Sockets Layer | Cryptographic protocol (deprecated; replaced by TLS) |
| **SSO** | Single Sign-On | Authentication allowing one login to access multiple services |
| `SSRF` | Server-Side Request Forgery | Attack making server perform unintended internal requests |
| **SSTI** | Server-Side Template Injection | Injecting template directives to execute code on server |
| **STS** | Short-Term Support | Distribution version with shorter maintenance window |
| **SVN** | Subversion | Centralized version control system |
| **TCP** | Transmission Control Protocol | Connection-oriented transport protocol; reliable delivery; Layer 4 |
| **TDD** | Test-Driven Development | Writing tests before implementation code |
| **TLS** | Transport Layer Security | Cryptographic protocol securing network communications; successor to SSL |
| **TOTP** | Time-based One-Time Password | OTP algorithm using current time (Google Authenticator) |
| **TTL** | Time to Live | Packet field limiting hops before discard; also DNS cache duration |
| **TTY** | Teletypewriter | Terminal device; `/dev/tty*` in Linux |
| **TUI** | Text User Interface | Terminal-based graphical interface (ncurses, etc.) |

---

## U–Z

| Acronym | Full Form | Context & Explanation |
|---------|-----------|----------------------|
| **UID** | User Identifier | Numeric ID for a user in Unix/Linux (root = 0) |
| **UEFI** | Unified Extensible Firmware Interface | Modern firmware replacing BIOS; supports GPT, Secure Boot |
| **UDP** | User Datagram Protocol | Connectionless transport protocol; fast but no delivery guarantee |
| **URL** | Uniform Resource Locator | Address for web resources: https://example.com/path |
| **USB** | Universal Serial Bus | Standard for connecting peripheral devices |
| **UUID** | Universally Unique Identifier | 128-bit identifier for filesystems, devices (e.g., in /etc/fstab) |
| **VCS** | Version Control System | Software managing code history (Git, SVN, Mercurial) |
| **VG** | Volume Group | LVM pool of physical volumes |
| **VLAN** | Virtual Local Area Network | Logical segmentation of a physical network |
| **VM** | Virtual Machine | Emulated computer running on physical hardware |
| **VMM** | Virtual Machine Monitor | Hypervisor software (KVM, VMware, VirtualBox) |
| **VoIP** | Voice over Internet Protocol | Transmitting voice calls over IP networks |
| **VPN** | Virtual Private Network | Encrypted tunnel over public network |
| **VPC** | Virtual Private Cloud | Isolated network within a cloud environment |
| **VFS** | Virtual File System | Kernel abstraction layer for multiple filesystem types |
| **WAF** | Web Application Firewall | Filters HTTP traffic to protect web applications |
| **WAN** | Wide Area Network | Network spanning large geographic areas (internet is a WAN) |
| **WebRTC** | Web Real-Time Communication | Browser API for peer-to-peer audio/video/data |
| **X.509** | ITU-T Standard for Certificates | Standard format for public key certificates (SSL/TLS uses it) |
| **XFS** | XFS Filesystem | High-performance 64-bit journaling filesystem |
| **XML** | eXtensible Markup Language | Structured data format using tags |
| **XXE** | XML External Entity | Attack exploiting XML parsers to read files or SSRF |
| **XSS** | Cross-Site Scripting | Injecting malicious scripts into web pages viewed by others |
| **YAGNI** | You Aren't Gonna Need It | Design principle: don't add functionality until necessary |
| **YAML** | YAML Ain't Markup Language | Human-readable data serialization format |
| **ZFS** | Zettabyte File System | Advanced filesystem with built-in RAID, snapshotting, checksums |

---

## Linux Command Acronym Origins

| Command | Stands For | Note |
|---------|-----------|------|
| `ls` | list | Lists directory contents |
| `cd` | change directory | Navigate directories |
| `pwd` | print working directory | Show current path |
| `rm` | remove | Delete files |
| `cp` | copy | Copy files |
| `mv` | move | Move or rename |
| `cat` | concatenate | Joins and displays files |
| `grep` | global regular expression print | Pattern search |
| `sed` | stream editor | Text transformation |
| `awk` | Aho, Weinberger, Kernighan | Named after its creators |
| `tar` | tape archive | Archive utility |
| `dd` | disk dump (data definition) | Low-level copy |
| `ps` | process status | Show processes |
| `df` | disk free | Show disk space |
| `du` | disk usage | Show file sizes |
| `su` | substitute user | Switch users |
| `sudo` | superuser do | Run as root |
| `ssh` | secure shell | Remote access |
| `scp` | secure copy | Remote file copy |
| `sftp` | SSH file transfer protocol | Secure FTP |
| `ftp` | file transfer protocol | File transfer |
| `chmod` | change mode | Modify permissions |
| `chown` | change owner | Change file ownership |
| `chgrp` | change group | Change file group |

---

*Back to [Index](INDEX.md) | Previous: [Appendix A — Command Reference](A_Complete_Command_Reference.md) | Next: [Appendix C — Troubleshooting Guide](C_Troubleshooting_Guide.md)*
