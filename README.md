# 🐧 Complete Linux (Ubuntu) Course: Beginner to Expert/Hacker Level

> **"From zero to root — master Linux the right way."**

[![Course Level](https://img.shields.io/badge/Level-Beginner%20to%20Expert-brightgreen)]()
[![Modules](https://img.shields.io/badge/Modules-24%20Chapters%20%2B%204%20Appendices-blue)]()
[![Words](https://img.shields.io/badge/Content-50%2C000%2B%20Words-orange)]()
[![Examples](https://img.shields.io/badge/Examples-500%2B-yellow)]()
[![Acronyms](https://img.shields.io/badge/Acronyms-300%2B-red)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey)]()

---

## 📖 Course Overview

Welcome to the **Complete Linux (Ubuntu) Course**, the most comprehensive open-source Linux curriculum available. This course takes you from your very first terminal command all the way to advanced penetration testing, system hardening, and ethical hacking techniques — all on Ubuntu Linux.

Whether you are a complete beginner who has never opened a terminal, a developer wanting to improve your command-line productivity, a system administrator managing servers, or an aspiring cybersecurity professional, this course has a carefully structured learning path designed for you.

Every chapter features:
- 📝 **Detailed explanations** written in plain English
- 💻 **500+ real-world command examples** you can run immediately
- 🖼️ **ASCII diagrams** illustrating system architecture concepts
- 🏋️ **Practice exercises** at the end of every module
- 📚 **300+ acronyms and aliases** defined in Appendix B
- ⚠️ **Security warnings** and best-practice callouts
- 🔗 **Cross-references** linking related topics across chapters

---

## ✅ Prerequisites

### Absolute Beginners
- A computer (Windows, macOS, or Linux) with at least **4 GB RAM** and **20 GB free disk space**
- Curiosity and willingness to type commands
- No prior Linux experience required

### For Security/Hacker Path
- Completion of Modules 1–3 (Fundamentals)
- Basic understanding of networking concepts (covered in Module 10)
- A dedicated virtual machine or test lab — **never practice on production systems**
- Commitment to ethical and legal use of all techniques taught

### Recommended Tools
| Tool | Purpose | Install |
|------|---------|---------|
| Ubuntu 22.04 LTS | Primary OS / VM | [ubuntu.com](https://ubuntu.com) |
| VirtualBox or VMware | Virtualization | Free download |
| VS Code | Text/script editing | `snap install code` |
| Terminator | Multi-pane terminal | `apt install terminator` |
| Wireshark | Network analysis | `apt install wireshark` |

---

## 🗺️ Learning Paths

### 🟢 Beginner Path — "I just want to use Linux"
**Estimated time: 20–30 hours**

Follow this path if you are new to Linux and want to become comfortable with day-to-day usage, file management, and basic shell commands.

```
Module 1 → Module 2 → Module 3 → Module 4 → Module 5 → Module 6
01 → 02 → 03 → 04 → 05 → 06
Then: Appendix A (Command Reference) + Appendix C (Troubleshooting)
```

**Outcome:** You will be able to navigate the filesystem, manage files and directories, install software, and feel at home in the terminal.

---

### 🔵 System Administrator Path — "I want to manage Linux servers"
**Estimated time: 60–80 hours**

Build on the Beginner Path to cover user management, process control, networking, shell scripting, and full system administration.

```
Beginner Path → Module 7 → Module 8 → Module 9 → Module 10
→ Module 11 → Module 12 → Module 13 → Module 14 → Module 15
Then: All Appendices
```

**Outcome:** You will be able to administer Ubuntu servers, write automation scripts, configure networks, manage storage, and implement security hardening.

---

### 🔴 Security / Hacker Path — "I want to learn ethical hacking"
**Estimated time: 100–150 hours**

The complete course from start to finish, culminating in penetration testing methodology, vulnerability assessment, exploitation techniques, and post-exploitation forensics.

```
Full System Admin Path → Module 16 → Module 17 → Module 18 → Module 19 → Module 20
Then: All Appendices + external CTF labs
```

**Outcome:** You will understand the full ethical hacking lifecycle — from reconnaissance through exploitation to reporting — using Linux-native tools.

> ⚠️ **Legal Disclaimer:** All hacking techniques in Modules 16–20 are for **educational purposes only**. Only perform security testing on systems you own or have explicit written permission to test. Unauthorized access is illegal in most jurisdictions.

---

## 📚 Complete Module Listing

### 🟩 Module 1: Fundamentals

| # | File | Topic | Est. Time |
|---|------|--------|-----------|
| 01 | [Introduction to Linux](01_Introduction_to_Linux.md) | What Linux is, distributions, the kernel, open-source philosophy | 1.5 hrs |
| 02 | [Linux History and Philosophy](02_Linux_History_and_Philosophy.md) | Unix origins, GNU project, Linus Torvalds, FOSS movement | 1.5 hrs |
| 03 | [Installation and Setup](03_Installation_and_Setup.md) | Dual boot, VM install, live USB, partitioning, GRUB | 2.5 hrs |
| 04 | [The Terminal Basics](04_The_Terminal_Basics.md) | Shell types, command syntax, man pages, history, shortcuts | 2.0 hrs |
| 05 | [File System Structure](05_File_System_Structure.md) | FHS, /etc /var /home /proc /dev explained, mount points | 2.0 hrs |
| 06 | [File and Directory Management](06_File_and_Directory_Management.md) | ls, cp, mv, rm, find, links, wildcards, archives | 2.5 hrs |

### 🟦 Module 2: Intermediate Concepts

| # | File | Topic | Est. Time |
|---|------|--------|-----------|
| 07 | [User and Permission Management](07_User_and_Permission_Management.md) | Users, groups, chmod, chown, ACLs, sudo, PAM | 3.0 hrs |
| 08 | [Process Management](08_Process_Management.md) | ps, top, htop, kill, nice, systemd, jobs, cron | 2.5 hrs |
| 09 | [Package Management](09_Package_Management.md) | apt, dpkg, snap, flatpak, PPAs, compiling from source | 2.0 hrs |
| 10 | [Networking Basics](10_Networking_Basics.md) | TCP/IP, interfaces, ip/ifconfig, DNS, SSH, firewall basics | 3.0 hrs |

### 🟧 Module 3: Advanced Topics

| # | File | Topic | Est. Time |
|---|------|--------|-----------|
| 11 | [Shell Scripting Fundamentals](11_Shell_Scripting_Fundamentals.md) | Bash scripting, variables, loops, functions, error handling | 4.0 hrs |
| 12 | [System Administration](12_System_Administration.md) | systemd, logs, backups, user quotas, monitoring | 3.5 hrs |
| 13 | [Storage and Disk Management](13_Storage_and_Disk_Management.md) | fdisk, LVM, RAID, filesystem types, fstab, NFS | 3.0 hrs |
| 14 | [Advanced Networking](14_Advanced_Networking.md) | VLANs, VPNs, iptables, routing, traffic analysis | 3.5 hrs |
| 15 | [Security Hardening](15_Security_Hardening.md) | SSH hardening, AppArmor, SELinux, fail2ban, audit | 3.5 hrs |

### 🟥 Module 4: Hacker / Penetration Testing

| # | File | Topic | Est. Time |
|---|------|--------|-----------|
| 16 | [Introduction to Ethical Hacking](16_Introduction_to_Ethical_Hacking.md) | Pen test methodology, legal framework, lab setup, Kali | 2.5 hrs |
| 17 | [Network Reconnaissance](17_Network_Reconnaissance.md) | nmap, netdiscover, OSINT, passive/active recon, Shodan | 3.5 hrs |
| 18 | [Vulnerability Assessment](18_Vulnerability_Assessment.md) | Nessus, OpenVAS, CVEs, CVSS scoring, reporting | 3.0 hrs |
| 19 | [Exploitation Basics](19_Exploitation_Basics.md) | Metasploit, exploits, payloads, shells, buffer overflows | 4.0 hrs |
| 20 | [Post Exploitation and Forensics](20_Post_Exploitation_Forensics.md) | Privilege escalation, persistence, log analysis, forensics | 4.0 hrs |

### 📘 Appendices

| File | Content | Est. Time |
|------|---------|-----------|
| [A — Complete Command Reference](A_Complete_Command_Reference.md) | Alphabetical index of 200+ Linux commands with syntax | Reference |
| [B — Aliases and Acronyms](B_Aliases_and_Acronyms.md) | 300+ acronyms, abbreviations, and shell aliases defined | Reference |
| [C — Troubleshooting Guide](C_Troubleshooting_Guide.md) | Common errors, boot issues, network failures, fixes | Reference |
| [D — Resources and Further Learning](D_Resources_and_Further_Learning.md) | Books, websites, certifications, CTF platforms, communities | Reference |

---

## 📊 Course Statistics

| Metric | Value |
|--------|-------|
| 📄 Total files | 28 (24 chapters + 4 appendices + README + INDEX) |
| 📝 Estimated total words | **50,000+** |
| 💻 Command examples | **500+** |
| 📖 Acronyms defined | **300+** |
| 🏋️ Practice exercises | **100+** |
| 🖼️ ASCII diagrams | **50+** |
| ⏱️ Total learning time | **60–150 hours** (path-dependent) |
| 🎯 Skill levels covered | Beginner → Expert → Hacker |
| 🐧 Ubuntu version | 22.04 LTS (Jammy Jellyfish) |

---

## 🚀 Quick Start Guide

### Step 1 — Get Ubuntu Running
```bash
# Option A: Install VirtualBox and Ubuntu 22.04 LTS in a VM
# Option B: Boot from a live USB
# Option C: Use WSL2 on Windows (good for beginners)
wsl --install -d Ubuntu
```

### Step 2 — Open the Terminal
- **Ubuntu Desktop:** Press `Ctrl + Alt + T`
- **WSL2:** Open "Ubuntu" from the Start menu
- **Server:** You are already in the terminal!

### Step 3 — Run Your First Commands
```bash
# Check your Ubuntu version
lsb_release -a

# See who you are
whoami

# See where you are in the filesystem
pwd

# List files in your current directory
ls -la

# Update your system (always do this first!)
sudo apt update && sudo apt upgrade -y
```

### Step 4 — Start the Course
Open [01_Introduction_to_Linux.md](01_Introduction_to_Linux.md) and begin your journey.

---

## 🧭 How to Use This Course

### For Self-Study
1. **Choose your learning path** from the section above
2. **Read each chapter sequentially** within your path
3. **Type every command yourself** — do not copy-paste blindly
4. **Complete the exercises** at the end of each module before moving on
5. **Use the appendices** as reference material throughout

### For Instructors
- Each module is self-contained and can be taught independently
- The appendices function as student handbooks
- Exercises are suitable for both individual and group lab sessions
- Modules 16–20 require a controlled lab environment

### Navigation Tips
- 📌 Use **[INDEX.md](INDEX.md)** to find any topic by keyword
- 📌 Use **[A_Complete_Command_Reference.md](A_Complete_Command_Reference.md)** when you need command syntax fast
- 📌 Use **[B_Aliases_and_Acronyms.md](B_Aliases_and_Acronyms.md)** when you encounter an unfamiliar term
- 📌 Use **[C_Troubleshooting_Guide.md](C_Troubleshooting_Guide.md)** when something breaks

---

## 🔗 Key Cross-References

| If you want to learn... | Start here |
|------------------------|------------|
| What Linux actually is | [01_Introduction_to_Linux.md](01_Introduction_to_Linux.md) |
| How to install Ubuntu | [03_Installation_and_Setup.md](03_Installation_and_Setup.md) |
| Basic terminal commands | [04_The_Terminal_Basics.md](04_The_Terminal_Basics.md) |
| File permissions | [07_User_and_Permission_Management.md](07_User_and_Permission_Management.md) |
| Writing shell scripts | [11_Shell_Scripting_Fundamentals.md](11_Shell_Scripting_Fundamentals.md) |
| Setting up a firewall | [15_Security_Hardening.md](15_Security_Hardening.md) |
| Network scanning with nmap | [17_Network_Reconnaissance.md](17_Network_Reconnaissance.md) |
| Using Metasploit | [19_Exploitation_Basics.md](19_Exploitation_Basics.md) |
| Any command syntax | [A_Complete_Command_Reference.md](A_Complete_Command_Reference.md) |
| Any acronym | [B_Aliases_and_Acronyms.md](B_Aliases_and_Acronyms.md) |

---

## 🎓 Certifications This Course Prepares You For

| Certification | Issuer | Relevant Modules |
|--------------|--------|-----------------|
| Linux Essentials | LPI | 01–06 |
| LPIC-1 | LPI | 01–13 |
| CompTIA Linux+ | CompTIA | 01–15 |
| CompTIA Security+ | CompTIA | 10, 14, 15, 16 |
| CompTIA PenTest+ | CompTIA | 16–20 |
| CEH | EC-Council | 16–20 |
| OSCP | Offensive Security | 16–20 + lab work |

---

## 🤝 Contributing

This is an open-source course. Contributions are welcome:
- Fix typos or factual errors
- Add new examples or exercises
- Translate content
- Submit new appendix entries

---

## 📄 License

This course is released under the **MIT License**. You are free to use, share, and modify the content for personal or educational use. Attribution appreciated.

---

## 🌐 Further Learning

| Resource | Type | Topic |
|---------|------|-------|
| [D_Resources_and_Further_Learning.md](D_Resources_and_Further_Learning.md) | Appendix | Books, sites, certifications |
| [OverTheWire: Bandit](https://overthewire.org) | CTF wargame | Linux command practice |
| [TryHackMe](https://tryhackme.com) | Online lab | Guided hacking rooms |
| [HackTheBox](https://hackthebox.com) | Online lab | Advanced CTF machines |
| [Linux Journey](https://linuxjourney.com) | Tutorial | Interactive Linux basics |

---

*Last updated: 2025 | Ubuntu 22.04 LTS (Jammy Jellyfish) | Course version 2.0*

> 🐧 **"The more you learn about Linux, the more you realize it's not just an operating system — it's a superpower."**
