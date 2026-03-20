# 📑 Complete Linux (Ubuntu) Course — Master Index

> Use this index to navigate the entire course. Find any topic, command, or acronym instantly.

**[← Back to README](README.md)**

---

## 🗂️ Table of Contents

1. [Module 1: Fundamentals (Chapters 01–06)](#module-1-fundamentals)
2. [Module 2: Intermediate Concepts (Chapters 07–10)](#module-2-intermediate-concepts)
3. [Module 3: Advanced Topics (Chapters 11–15)](#module-3-advanced-topics)
4. [Module 4: Hacker / Penetration Testing (Chapters 16–20)](#module-4-hacker--penetration-testing)
5. [Appendices (A, B, C, D)](#appendices)
6. [Alphabetical Topic Index](#alphabetical-topic-index)
7. [Command Quick-Reference Index](#command-quick-reference-index)
8. [Acronym Index](#acronym-index)

---

## Module 1: Fundamentals

*Target audience: Complete beginners. No prior Linux experience required.*

### [01 — Introduction to Linux](01_Introduction_to_Linux.md)

| Section | Topics Covered |
|---------|---------------|
| What is Linux? | Kernel vs. OS, monolithic kernel, open-source definition |
| Linux Distributions | Ubuntu, Debian, Fedora, Arch, Kali, CentOS explained |
| Ubuntu Architecture | Desktop vs. Server vs. Core editions |
| Why Learn Linux? | Server dominance, cloud computing, cybersecurity, IoT |
| Linux vs. Windows vs. macOS | Comparison table, use cases, pros and cons |
| Key Terminology | Kernel, shell, daemon, init, runlevels, POSIX |
| Your First Boot | What happens when Ubuntu starts |

**Key commands introduced:** `uname`, `lsb_release`, `hostnamectl`

---

### [02 — Linux History and Philosophy](02_Linux_History_and_Philosophy.md)

| Section | Topics Covered |
|---------|---------------|
| Unix Origins (1969) | Bell Labs, Thompson and Ritchie, C language birth |
| The GNU Project | Richard Stallman, GNU Manifesto, GPL license |
| Linux Kernel (1991) | Linus Torvalds, first announcement, version history |
| FOSS Philosophy | Free as in freedom, copyleft, open development model |
| Major Milestones | Key kernel versions, Ubuntu founding (2004) |
| Linux Today | Market share, Android, cloud servers, supercomputers |
| The Hacker Culture | Hacker ethic, collaborative development, meritocracy |

**Key concepts:** GPL, FOSS, POSIX, Unix philosophy

---

### [03 — Installation and Setup](03_Installation_and_Setup.md)

| Section | Topics Covered |
|---------|---------------|
| Installation Methods | Dual boot, VM, live USB, WSL2, cloud instance |
| VirtualBox Setup | Download, create VM, allocate RAM/storage |
| Creating a Live USB | Rufus (Windows), Etcher, dd command |
| Ubuntu Installer | Ubiquity walkthrough, disk partitioning options |
| GRUB Bootloader | How GRUB works, dual-boot configuration |
| First Boot Tasks | Update system, install guest additions, snapshots |
| WSL2 Quick Setup | Windows Subsystem for Linux installation guide |
| Post-Install Checklist | 10 things to do after installing Ubuntu |

**Key commands introduced:** `dd`, `fdisk`, `lsblk`, `mount`, `update-grub`

---

### [04 — The Terminal Basics](04_The_Terminal_Basics.md)

| Section | Topics Covered |
|---------|---------------|
| What is a Shell? | bash, zsh, sh, fish — comparison and defaults |
| Terminal Emulators | gnome-terminal, Terminator, tmux, alacritty |
| Command Syntax | command [options] [arguments] structure |
| Getting Help | man, --help, info, tldr, apropos |
| Command History | history, reverse search shortcuts |
| Keyboard Shortcuts | Ctrl+C, Ctrl+D, Ctrl+Z, Ctrl+L, Tab completion |
| Input/Output Redirection | >, >>, <, 2>, /dev/null |
| Pipes | Pipe operator, combining commands |
| Variables | Shell variables, export, PATH, HOME |
| Aliases | Creating and persisting command aliases |

**Key commands introduced:** `echo`, `cat`, `less`, `more`, `head`, `tail`, `grep`, `wc`, `sort`, `uniq`

---

### [05 — File System Structure](05_File_System_Structure.md)

| Section | Topics Covered |
|---------|---------------|
| Filesystem Hierarchy Standard (FHS) | Why Linux uses a unified tree |
| / (Root) | The top of the hierarchy |
| /bin and /sbin | Essential user and system binaries |
| /etc | System-wide configuration files |
| /home | User home directories |
| /var | Variable data: logs, spool, cache |
| /tmp | Temporary files, cleared on reboot |
| /proc | Virtual filesystem: kernel and process info |
| /sys | Sysfs: hardware and driver information |
| /dev | Device files: block, character, special |
| /usr | Secondary hierarchy: apps, libraries, docs |
| /opt | Optional third-party software |
| /boot | Kernel images, GRUB files |
| /mnt and /media | Mount points for removable media |
| Absolute vs. Relative Paths | Navigation concepts |

**Key commands introduced:** `tree`, `df`, `du`, `file`, `stat`, `lsblk`

---

### [06 — File and Directory Management](06_File_and_Directory_Management.md)

| Section | Topics Covered |
|---------|---------------|
| Navigating the Filesystem | cd, pwd, ls options |
| Creating Files and Dirs | touch, mkdir -p |
| Copying and Moving | cp, mv with recursion and verbose flags |
| Removing Files | rm, rmdir, safety tips, trash-cli |
| Finding Files | find with expressions, locate, which, whereis |
| Searching Content | grep, grep -r, egrep, fgrep |
| File Compression | tar, gzip, bzip2, xz, zip, unzip |
| Hard Links and Symlinks | ln, ln -s, difference explained |
| Wildcards and Globbing | *, ?, [abc], {a,b} patterns |
| File Viewing Tools | cat, less, head, tail -f |
| Text Processing | awk, sed, cut, tr, paste |

**Key commands introduced:** `find`, `tar`, `ln`, `grep`, `awk`, `sed`

---

## Module 2: Intermediate Concepts

*Build on fundamentals to manage users, processes, packages, and networks.*

### [07 — User and Permission Management](07_User_and_Permission_Management.md)

| Section | Topics Covered |
|---------|---------------|
| Linux User Model | UID/GID, /etc/passwd, /etc/shadow, /etc/group |
| Creating Users | useradd, adduser, passwd, chage |
| Managing Groups | groupadd, usermod -aG, gpasswd, newgrp |
| File Permissions | Read/write/execute, octal notation (755, 644) |
| chmod | Symbolic and numeric modes, recursive changes |
| chown and chgrp | Changing ownership |
| Special Permissions | SUID (4), SGID (2), Sticky bit (1) |
| Access Control Lists (ACLs) | setfacl, getfacl - per-user permissions |
| sudo and sudoers | /etc/sudoers, visudo, NOPASSWD entries |
| PAM | Pluggable Authentication Modules overview |
| Switching Users | su, sudo su, sudo -u user |

**Key commands:** useradd, passwd, chmod, chown, sudo, visudo, setfacl

---

### [08 — Process Management](08_Process_Management.md)

| Section | Topics Covered |
|---------|---------------|
| What is a Process? | PID, PPID, process states (R, S, D, Z, T) |
| Viewing Processes | ps aux, ps -ef, pstree |
| Real-time Monitoring | top, htop, btop, resource columns explained |
| Sending Signals | Terminate signals: SIGTERM, SIGKILL, SIGHUP |
| Process Priority | nice, renice, priority values (-20 to 19) |
| Background/Foreground | &, jobs, fg, bg, Ctrl+Z |
| systemd and Services | systemctl start/stop/enable/disable/status |
| Service Units | Writing a basic .service unit file |
| Scheduling with cron | crontab -e, cron syntax |
| at and batch | One-time scheduled commands |
| /proc Filesystem | Inspecting process details via /proc/[PID]/ |

**Key commands:** ps, top, htop, kill, nice, systemctl, crontab

---

### [09 — Package Management](09_Package_Management.md)

| Section | Topics Covered |
|---------|---------------|
| Debian Package System | .deb format, package metadata, dependencies |
| apt (Advanced Package Tool) | install, remove, purge, update, upgrade |
| dpkg | Low-level package management, listing, querying |
| Package Repositories | /etc/apt/sources.list, PPAs, third-party repos |
| Snap Packages | snap install, snap list, confinement model |
| Flatpak | Universal packages, Flathub, pros and cons |
| AppImage | Portable executables, no installation needed |
| Compiling from Source | ./configure, make, make install workflow |
| Package Security | GPG key verification, apt-key, signed repos |
| Cleaning Up | apt autoremove, apt autoclean, deborphan |

**Key commands:** apt, apt-get, dpkg, snap, flatpak

---

### [10 — Networking Basics](10_Networking_Basics.md)

| Section | Topics Covered |
|---------|---------------|
| OSI and TCP/IP Models | Layer overview and mapping |
| IP Addressing | IPv4, IPv6, subnetting, CIDR notation |
| Network Interfaces | ip addr, ifconfig, ip link |
| Routing | ip route, default gateway, route command |
| DNS | /etc/resolv.conf, dig, nslookup, host |
| DHCP | How DHCP works, dhclient |
| NetworkManager | nmcli, nmtui, connection profiles |
| SSH | ssh, scp, sftp, key-based auth, ~/.ssh/config |
| Firewall Basics | ufw (Uncomplicated Firewall), rules, logging |
| Network Diagnostics | ping, traceroute, netstat, ss, tcpdump |
| /etc/hosts | Local DNS overrides |

**Key commands:** ip, ss, ping, dig, ssh, scp, ufw, tcpdump

---

## Module 3: Advanced Topics

*Deep-dive into scripting, administration, storage, advanced networking, and hardening.*

### [11 — Shell Scripting Fundamentals](11_Shell_Scripting_Fundamentals.md)

| Section | Topics Covered |
|---------|---------------|
| Variables | Declaration, quoting, variable syntax |
| User Input | read, command-line arguments |
| Arithmetic | Double-paren syntax, let, bc for floats |
| Conditionals | if/elif/else, test operators |
| Case Statements | case...esac pattern matching |
| Loops | for, while, until, break, continue |
| Functions | Defining, calling, return values, local variables |
| Arrays | Indexed and associative arrays |
| String Manipulation | Substrings, replacement, length |
| Error Handling | set -e, trap, exit codes |
| Debugging Scripts | bash -x, set -xv, shellcheck |
| Practical Examples | Backup script, log monitor, user provisioner |

**Key concepts:** shebang, exit codes, process substitution, heredoc

---

### [12 — System Administration](12_System_Administration.md)

| Section | Topics Covered |
|---------|---------------|
| systemd Deep Dive | Targets, dependencies, journal, timer units |
| Log Management | journalctl, /var/log/, logrotate, syslog |
| Backup Strategies | rsync, tar with exclusions, cron + rsync |
| System Monitoring | sar, vmstat, iostat, dmesg |
| Disk Usage and Quotas | du, df, quota, edquota |
| User Account Policies | Password aging, account locking, chage |
| Time Synchronization | timedatectl, chrony, ntpd, NTP pools |
| System Updates | unattended-upgrades, update policies |
| Kernel Management | uname -r, installed kernels, update-initramfs |
| Remote Administration | SSH tunneling, tmux for persistent sessions |
| Performance Tuning | sysctl, /proc/sys/, kernel parameters |

**Key commands:** journalctl, systemctl, rsync, sar, timedatectl, sysctl

---

### [13 — Storage and Disk Management](13_Storage_and_Disk_Management.md)

| Section | Topics Covered |
|---------|---------------|
| Block Devices | /dev/sda, /dev/nvme0n1, lsblk output |
| Partition Tables | MBR vs. GPT, fdisk, gdisk, parted |
| Filesystem Types | ext4, xfs, btrfs, tmpfs, swap |
| Creating Filesystems | mkfs.ext4, mkswap, swapon |
| Mounting | mount, umount, /etc/fstab, blkid |
| LVM (Logical Volume Manager) | PV, VG, LV - create, extend, reduce |
| RAID | Levels 0,1,5,10; mdadm software RAID |
| Network Storage | NFS setup, fstab NFS entries, Samba intro |
| Disk Health | smartctl, smartmontools, failure prediction |
| Filesystem Repair | fsck, e2fsck, forced check on boot |
| Storage Performance | fio, hdparm, IOPS, throughput concepts |

**Key commands:** fdisk, mkfs, mount, lvm, mdadm, smartctl, fsck

---

### [14 — Advanced Networking](14_Advanced_Networking.md)

| Section | Topics Covered |
|---------|---------------|
| Network Namespaces | Isolation, ip netns, containers foundation |
| VLANs | IEEE 802.1Q tagging, ip link add link |
| Bonding and Bridging | bond0, bridge interfaces, brctl |
| Advanced Routing | Policy routing, multiple routing tables |
| iptables Deep Dive | Tables (filter/nat/mangle), chains, rules, MASQUERADE |
| nftables | Modern iptables replacement, rule syntax |
| VPN Setup | OpenVPN, WireGuard installation and config |
| Traffic Analysis | tcpdump filters, Wireshark, tshark |
| Network Performance | iperf3, bandwidth testing, latency optimization |
| Proxy and Load Balancing | HAProxy basics, nginx as a reverse proxy |
| IPv6 Configuration | Link-local, global addresses, ip -6 commands |

**Key commands:** iptables, nft, ip netns, tcpdump, iperf3, wg

---

### [15 — Security Hardening](15_Security_Hardening.md)

| Section | Topics Covered |
|---------|---------------|
| Attack Surface Reduction | Minimal install, disabling services, removing packages |
| SSH Hardening | Disable root login, key-only auth, sshd_config |
| Firewall Configuration | ufw profiles, iptables rules, default-deny policy |
| Fail2ban | Installing, jail configuration, custom filters |
| AppArmor | Profiles, modes (enforce/complain), aa-status |
| File Integrity Monitoring | AIDE, tripwire - baseline and checking |
| Audit Framework | auditd, audit rules, aureport, ausearch |
| Kernel Hardening | sysctl settings: ASLR, exec-shield, network params |
| Vulnerability Scanning | lynis, openscap - local system auditing |
| Security Checklist | CIS Benchmark Ubuntu - key controls |

**Key commands:** ufw, fail2ban-client, apparmor_status, auditctl, lynis

---

## Module 4: Hacker / Penetration Testing

> ⚠️ **Legal and Ethical Use Only.** All content in this module is for authorized security testing and education.

### [16 — Introduction to Ethical Hacking](16_Introduction_to_Ethical_Hacking.md)

| Section | Topics Covered |
|---------|---------------|
| Penetration Testing Methodology | Reconnaissance, Scanning, Exploitation, Reporting |
| Legal Framework | Computer Misuse Act, CFAA, written authorization |
| Black / White / Grey Box | Testing types explained |
| Kali Linux Setup | Installation, tools overview, updating |
| Lab Environment | Setting up vulnerable VMs (Metasploitable, DVWA) |
| OWASP Top 10 | Web application vulnerability overview |
| CVE and NVD | Understanding the vulnerability database |
| Reporting | Executive summary, technical findings, CVSS scoring |

---

### [17 — Network Reconnaissance](17_Network_Reconnaissance.md)

| Section | Topics Covered |
|---------|---------------|
| OSINT Techniques | theHarvester, Maltego, Shodan, Google dorking |
| Passive Reconnaissance | DNS enumeration, WHOIS, certificate transparency |
| Active Reconnaissance | Network scanning, service detection |
| nmap Mastery | Host discovery, port scanning, OS detection, NSE scripts |
| netdiscover | ARP scanning on local networks |
| Service Enumeration | Banner grabbing, version detection |
| SMB Enumeration | enum4linux, smbclient, rpcclient |
| SNMP Enumeration | snmpwalk, community strings |
| Web Enumeration | dirb, gobuster, nikto, robots.txt |

**Key tools:** nmap, netdiscover, theHarvester, enum4linux, nikto

---

### [18 — Vulnerability Assessment](18_Vulnerability_Assessment.md)

| Section | Topics Covered |
|---------|---------------|
| Vulnerability vs. Exploit | Definitions and the difference |
| CVE System | Common Vulnerabilities and Exposures database |
| CVSS Scoring | Base, Temporal, Environmental metrics |
| OpenVAS / GVM | Installation, scanning, report interpretation |
| Nessus Essentials | Free tier, policy creation, scan analysis |
| Manual Assessment | Checking versions, misconfigurations by hand |
| Web App Assessment | Burp Suite Community, OWASP ZAP, manual testing |
| Database Assessment | SQLMap basics, injection detection |
| Reporting Standards | PTES, OWASP Testing Guide, reporting templates |

**Key tools:** openvas, nikto, sqlmap, burpsuite

---

### [19 — Exploitation Basics](19_Exploitation_Basics.md)

| Section | Topics Covered |
|---------|---------------|
| Metasploit Framework | Architecture, msfconsole, modules, payloads |
| Searching for Exploits | searchsploit, ExploitDB, GitHub advisories |
| Exploit Module Workflow | use, set, options, run |
| Payloads | Meterpreter, reverse shells, bind shells |
| Manual Exploitation | Python exploit scripts, compiling PoC code |
| Buffer Overflows | Stack layout, EIP overwrite, Python PoC |
| Web Exploitation | SQLi, XSS, LFI, RFI, command injection |
| Password Attacks | hydra, medusa, john, hashcat |
| Social Engineering | SET (Social Engineering Toolkit) overview |

**Key tools:** msfconsole, searchsploit, hydra, john, hashcat

---

### [20 — Post Exploitation and Forensics](20_Post_Exploitation_Forensics.md)

| Section | Topics Covered |
|---------|---------------|
| Post-Exploitation Goals | Maintain access, pivot, exfiltrate, cover tracks |
| Privilege Escalation | Sudo misconfig, SUID binaries, kernel exploits |
| LinPEAS / LinEnum | Automated local enumeration tools |
| Persistence Mechanisms | Cron jobs, SSH keys, systemd services |
| Lateral Movement | SSH pivoting, proxychains, port forwarding |
| Covering Tracks | Log cleaning (educational - defenders detect this) |
| Digital Forensics Intro | Incident response, evidence collection principles |
| Log Analysis | Parsing auth.log, syslog, last, utmpdump |
| Memory Forensics | volatility overview, dumping memory |
| Incident Response | The 6-phase IR framework |

**Key tools:** linpeas, gtfobins, proxychains, volatility

---

## Appendices

### [A — Complete Command Reference](A_Complete_Command_Reference.md)

Alphabetical index of 200+ Linux commands with full syntax, flags, and examples.

Quick Jump: a–e | f–j | k–o | p–t | u–z

---

### [B — Aliases and Acronyms](B_Aliases_and_Acronyms.md)

300+ acronyms and abbreviations used throughout Linux, networking, and cybersecurity. See [Acronym Index](#acronym-index) below.

---

### [C — Troubleshooting Guide](C_Troubleshooting_Guide.md)

Common problems organized by category: Boot issues, network failures, permission errors, package conflicts, disk full, SSH problems.

---

### [D — Resources and Further Learning](D_Resources_and_Further_Learning.md)

Curated books, websites, YouTube channels, certifications, CTF platforms, and online communities.

---

## Alphabetical Topic Index

| Topic | File(s) |
|-------|---------|
| Access Control Lists (ACLs) | [07](07_User_and_Permission_Management.md) |
| AppArmor | [15](15_Security_Hardening.md) |
| apt / apt-get | [09](09_Package_Management.md), [A](A_Complete_Command_Reference.md) |
| Archives (tar, zip) | [06](06_File_and_Directory_Management.md) |
| AIDE (file integrity) | [15](15_Security_Hardening.md) |
| auditd / audit framework | [15](15_Security_Hardening.md) |
| awk | [06](06_File_and_Directory_Management.md), [11](11_Shell_Scripting_Fundamentals.md) |
| Bash scripting | [11](11_Shell_Scripting_Fundamentals.md) |
| Boot process | [01](01_Introduction_to_Linux.md), [03](03_Installation_and_Setup.md) |
| Buffer overflow | [19](19_Exploitation_Basics.md) |
| Burp Suite | [18](18_Vulnerability_Assessment.md) |
| chmod | [07](07_User_and_Permission_Management.md), [A](A_Complete_Command_Reference.md) |
| chown | [07](07_User_and_Permission_Management.md), [A](A_Complete_Command_Reference.md) |
| cron / crontab | [08](08_Process_Management.md), [12](12_System_Administration.md) |
| CVSS scoring | [18](18_Vulnerability_Assessment.md) |
| CVE / NVD | [16](16_Introduction_to_Ethical_Hacking.md), [18](18_Vulnerability_Assessment.md) |
| Debian packages (.deb) | [09](09_Package_Management.md) |
| dig / DNS | [10](10_Networking_Basics.md), [17](17_Network_Reconnaissance.md) |
| dpkg | [09](09_Package_Management.md), [A](A_Complete_Command_Reference.md) |
| Encryption / GPG | [09](09_Package_Management.md), [15](15_Security_Hardening.md) |
| enum4linux | [17](17_Network_Reconnaissance.md) |
| Ethical hacking intro | [16](16_Introduction_to_Ethical_Hacking.md) |
| ExploitDB / searchsploit | [19](19_Exploitation_Basics.md) |
| fail2ban | [15](15_Security_Hardening.md) |
| fdisk / gdisk | [13](13_Storage_and_Disk_Management.md) |
| File permissions | [07](07_User_and_Permission_Management.md) |
| File system hierarchy | [05](05_File_System_Structure.md) |
| find | [06](06_File_and_Directory_Management.md), [A](A_Complete_Command_Reference.md) |
| Flatpak | [09](09_Package_Management.md) |
| FOSS philosophy | [02](02_Linux_History_and_Philosophy.md) |
| fsck | [13](13_Storage_and_Disk_Management.md) |
| GNU project | [02](02_Linux_History_and_Philosophy.md) |
| gobuster | [17](17_Network_Reconnaissance.md) |
| grep | [04](04_The_Terminal_Basics.md), [06](06_File_and_Directory_Management.md) |
| groups / groupadd | [07](07_User_and_Permission_Management.md) |
| GRUB bootloader | [03](03_Installation_and_Setup.md) |
| hashcat | [19](19_Exploitation_Basics.md) |
| htop | [08](08_Process_Management.md) |
| hydra | [19](19_Exploitation_Basics.md) |
| Incident response | [20](20_Post_Exploitation_Forensics.md) |
| ip / ifconfig | [10](10_Networking_Basics.md), [14](14_Advanced_Networking.md) |
| iptables / nftables | [14](14_Advanced_Networking.md), [15](15_Security_Hardening.md) |
| john (John the Ripper) | [19](19_Exploitation_Basics.md) |
| journalctl | [12](12_System_Administration.md) |
| Kernel | [01](01_Introduction_to_Linux.md), [02](02_Linux_History_and_Philosophy.md) |
| LinPEAS / privilege escalation | [20](20_Post_Exploitation_Forensics.md) |
| Linux distributions | [01](01_Introduction_to_Linux.md) |
| Linux history | [02](02_Linux_History_and_Philosophy.md) |
| LVM | [13](13_Storage_and_Disk_Management.md) |
| lynis | [15](15_Security_Hardening.md) |
| man pages | [04](04_The_Terminal_Basics.md) |
| mdadm / RAID | [13](13_Storage_and_Disk_Management.md) |
| Metasploit | [19](19_Exploitation_Basics.md) |
| mount / fstab | [05](05_File_System_Structure.md), [13](13_Storage_and_Disk_Management.md) |
| netcat (nc) | [19](19_Exploitation_Basics.md), [A](A_Complete_Command_Reference.md) |
| NetworkManager / nmcli | [10](10_Networking_Basics.md) |
| nmap | [17](17_Network_Reconnaissance.md) |
| nftables | [14](14_Advanced_Networking.md) |
| nikto | [17](17_Network_Reconnaissance.md), [18](18_Vulnerability_Assessment.md) |
| NFS | [13](13_Storage_and_Disk_Management.md) |
| OpenVAS / GVM | [18](18_Vulnerability_Assessment.md) |
| OSINT | [17](17_Network_Reconnaissance.md) |
| OWASP Top 10 | [16](16_Introduction_to_Ethical_Hacking.md), [18](18_Vulnerability_Assessment.md) |
| Package management | [09](09_Package_Management.md) |
| PAM | [07](07_User_and_Permission_Management.md) |
| Partitioning | [03](03_Installation_and_Setup.md), [13](13_Storage_and_Disk_Management.md) |
| passwd / shadow | [07](07_User_and_Permission_Management.md) |
| Penetration testing | [16](16_Introduction_to_Ethical_Hacking.md) |
| Pipes and redirection | [04](04_The_Terminal_Basics.md) |
| Post-exploitation | [20](20_Post_Exploitation_Forensics.md) |
| Privilege escalation | [20](20_Post_Exploitation_Forensics.md) |
| /proc filesystem | [05](05_File_System_Structure.md), [08](08_Process_Management.md) |
| proxychains | [20](20_Post_Exploitation_Forensics.md) |
| ps / pstree | [08](08_Process_Management.md) |
| RAID | [13](13_Storage_and_Disk_Management.md) |
| Reconnaissance | [17](17_Network_Reconnaissance.md) |
| rsync | [12](12_System_Administration.md) |
| Samba / SMB | [17](17_Network_Reconnaissance.md) |
| scp | [10](10_Networking_Basics.md) |
| sed | [06](06_File_and_Directory_Management.md), [11](11_Shell_Scripting_Fundamentals.md) |
| SELinux | [15](15_Security_Hardening.md) |
| Shell scripting | [11](11_Shell_Scripting_Fundamentals.md) |
| Snap packages | [09](09_Package_Management.md) |
| SQLMap | [18](18_Vulnerability_Assessment.md) |
| SSH | [10](10_Networking_Basics.md), [15](15_Security_Hardening.md) |
| sysctl | [12](12_System_Administration.md), [15](15_Security_Hardening.md) |
| systemd | [08](08_Process_Management.md), [12](12_System_Administration.md) |
| tcpdump | [10](10_Networking_Basics.md), [14](14_Advanced_Networking.md) |
| Terminal basics | [04](04_The_Terminal_Basics.md) |
| Torvalds, Linus | [02](02_Linux_History_and_Philosophy.md) |
| Ubuntu installation | [03](03_Installation_and_Setup.md) |
| ufw | [10](10_Networking_Basics.md), [15](15_Security_Hardening.md) |
| Unix history | [02](02_Linux_History_and_Philosophy.md) |
| useradd / adduser | [07](07_User_and_Permission_Management.md) |
| Variables (shell) | [04](04_The_Terminal_Basics.md), [11](11_Shell_Scripting_Fundamentals.md) |
| Vim / Nano | [04](04_The_Terminal_Basics.md), [A](A_Complete_Command_Reference.md) |
| VirtualBox setup | [03](03_Installation_and_Setup.md) |
| VPN | [14](14_Advanced_Networking.md) |
| Vulnerability assessment | [18](18_Vulnerability_Assessment.md) |
| Wireshark / tshark | [14](14_Advanced_Networking.md) |
| WireGuard | [14](14_Advanced_Networking.md) |
| WSL2 | [03](03_Installation_and_Setup.md) |
| xargs | [06](06_File_and_Directory_Management.md), [A](A_Complete_Command_Reference.md) |

---

## Command Quick-Reference Index

| Command | Category | Chapter |
|---------|----------|---------|
| `arp` | Networking | [10](10_Networking_Basics.md) |
| `at` | Scheduling | [08](08_Process_Management.md) |
| `awk` | Text processing | [06](06_File_and_Directory_Management.md) |
| `cat` | File viewing | [04](04_The_Terminal_Basics.md) |
| `cd` | Navigation | [06](06_File_and_Directory_Management.md) |
| `chmod` | Permissions | [07](07_User_and_Permission_Management.md) |
| `chown` | Permissions | [07](07_User_and_Permission_Management.md) |
| `cp` | File ops | [06](06_File_and_Directory_Management.md) |
| `crontab` | Scheduling | [08](08_Process_Management.md) |
| `curl` | Networking | [10](10_Networking_Basics.md) |
| `cut` | Text processing | [06](06_File_and_Directory_Management.md) |
| `dd` | Disk ops | [03](03_Installation_and_Setup.md) |
| `df` | Storage | [05](05_File_System_Structure.md) |
| `dig` | DNS | [10](10_Networking_Basics.md) |
| `dpkg` | Packages | [09](09_Package_Management.md) |
| `du` | Storage | [05](05_File_System_Structure.md) |
| `echo` | Output | [04](04_The_Terminal_Basics.md) |
| `fdisk` | Disk ops | [13](13_Storage_and_Disk_Management.md) |
| `file` | File info | [05](05_File_System_Structure.md) |
| `find` | Search | [06](06_File_and_Directory_Management.md) |
| `fsck` | Disk repair | [13](13_Storage_and_Disk_Management.md) |
| `grep` | Search | [04](04_The_Terminal_Basics.md) |
| `groupadd` | Users | [07](07_User_and_Permission_Management.md) |
| `gzip` | Compression | [06](06_File_and_Directory_Management.md) |
| `head` | File viewing | [04](04_The_Terminal_Basics.md) |
| `hostnamectl` | System info | [01](01_Introduction_to_Linux.md) |
| `htop` | Monitoring | [08](08_Process_Management.md) |
| `ip` | Networking | [10](10_Networking_Basics.md) |
| `iperf3` | Network perf | [14](14_Advanced_Networking.md) |
| `iptables` | Firewall | [14](14_Advanced_Networking.md) |
| `journalctl` | Logs | [12](12_System_Administration.md) |
| `kill` | Processes | [08](08_Process_Management.md) |
| `less` | File viewing | [04](04_The_Terminal_Basics.md) |
| `ln` | Links | [06](06_File_and_Directory_Management.md) |
| `lsblk` | Storage | [05](05_File_System_Structure.md) |
| `lsof` | Processes | [08](08_Process_Management.md) |
| `man` | Help | [04](04_The_Terminal_Basics.md) |
| `mkdir` | Directory ops | [06](06_File_and_Directory_Management.md) |
| `mount` | Storage | [13](13_Storage_and_Disk_Management.md) |
| `mv` | File ops | [06](06_File_and_Directory_Management.md) |
| `nmap` | Recon | [17](17_Network_Reconnaissance.md) |
| `passwd` | Users | [07](07_User_and_Permission_Management.md) |
| `ping` | Networking | [10](10_Networking_Basics.md) |
| `ps` | Processes | [08](08_Process_Management.md) |
| `pwd` | Navigation | [04](04_The_Terminal_Basics.md) |
| `rm` | File ops | [06](06_File_and_Directory_Management.md) |
| `rsync` | Backup | [12](12_System_Administration.md) |
| `scp` | File transfer | [10](10_Networking_Basics.md) |
| `sed` | Text processing | [06](06_File_and_Directory_Management.md) |
| `smartctl` | Disk health | [13](13_Storage_and_Disk_Management.md) |
| `snap` | Packages | [09](09_Package_Management.md) |
| `sort` | Text processing | [04](04_The_Terminal_Basics.md) |
| `ss` | Networking | [10](10_Networking_Basics.md) |
| `ssh` | Remote access | [10](10_Networking_Basics.md) |
| `stat` | File info | [05](05_File_System_Structure.md) |
| `sudo` | Privilege | [07](07_User_and_Permission_Management.md) |
| `sysctl` | Kernel | [12](12_System_Administration.md) |
| `systemctl` | Services | [08](08_Process_Management.md) |
| `tail` | File viewing | [04](04_The_Terminal_Basics.md) |
| `tar` | Archives | [06](06_File_and_Directory_Management.md) |
| `tcpdump` | Network capture | [14](14_Advanced_Networking.md) |
| `timedatectl` | Time | [12](12_System_Administration.md) |
| `top` | Monitoring | [08](08_Process_Management.md) |
| `touch` | File ops | [06](06_File_and_Directory_Management.md) |
| `traceroute` | Networking | [10](10_Networking_Basics.md) |
| `tree` | Navigation | [05](05_File_System_Structure.md) |
| `ufw` | Firewall | [10](10_Networking_Basics.md) |
| `uname` | System info | [01](01_Introduction_to_Linux.md) |
| `uniq` | Text processing | [04](04_The_Terminal_Basics.md) |
| `useradd` | Users | [07](07_User_and_Permission_Management.md) |
| `visudo` | Privilege | [07](07_User_and_Permission_Management.md) |
| `wc` | Text processing | [04](04_The_Terminal_Basics.md) |
| `wget` | Download | [10](10_Networking_Basics.md) |
| `which` | Search | [06](06_File_and_Directory_Management.md) |
| `whoami` | Users | [07](07_User_and_Permission_Management.md) |
| `xargs` | Utilities | [06](06_File_and_Directory_Management.md) |

> 📖 For complete syntax and all flags, see **[Appendix A: Complete Command Reference](A_Complete_Command_Reference.md)**

---

## Acronym Index

All acronyms are fully defined in **[Appendix B: Aliases and Acronyms](B_Aliases_and_Acronyms.md)**.

| Acronym | Stands For | See Also |
|---------|-----------|---------|
| ACL | Access Control List | [07](07_User_and_Permission_Management.md) |
| ASLR | Address Space Layout Randomization | [15](15_Security_Hardening.md) |
| API | Application Programming Interface | [B](B_Aliases_and_Acronyms.md) |
| APT | Advanced Package Tool | [09](09_Package_Management.md) |
| BIOS | Basic Input/Output System | [03](03_Installation_and_Setup.md) |
| CA | Certificate Authority | [15](15_Security_Hardening.md) |
| CEH | Certified Ethical Hacker | [16](16_Introduction_to_Ethical_Hacking.md) |
| CIDR | Classless Inter-Domain Routing | [10](10_Networking_Basics.md) |
| CLI | Command-Line Interface | [04](04_The_Terminal_Basics.md) |
| CPU | Central Processing Unit | [08](08_Process_Management.md) |
| CTF | Capture The Flag | [D](D_Resources_and_Further_Learning.md) |
| CVE | Common Vulnerabilities and Exposures | [18](18_Vulnerability_Assessment.md) |
| CVSS | Common Vulnerability Scoring System | [18](18_Vulnerability_Assessment.md) |
| DHCP | Dynamic Host Configuration Protocol | [10](10_Networking_Basics.md) |
| DNS | Domain Name System | [10](10_Networking_Basics.md) |
| DVWA | Damn Vulnerable Web Application | [16](16_Introduction_to_Ethical_Hacking.md) |
| FHS | Filesystem Hierarchy Standard | [05](05_File_System_Structure.md) |
| FOSS | Free and Open-Source Software | [02](02_Linux_History_and_Philosophy.md) |
| GID | Group Identifier | [07](07_User_and_Permission_Management.md) |
| GNU | GNU Is Not Unix | [02](02_Linux_History_and_Philosophy.md) |
| GPL | General Public License | [02](02_Linux_History_and_Philosophy.md) |
| GPT | GUID Partition Table | [13](13_Storage_and_Disk_Management.md) |
| GRUB | Grand Unified Bootloader | [03](03_Installation_and_Setup.md) |
| GUI | Graphical User Interface | [01](01_Introduction_to_Linux.md) |
| IANA | Internet Assigned Numbers Authority | [10](10_Networking_Basics.md) |
| IDS | Intrusion Detection System | [15](15_Security_Hardening.md) |
| IP | Internet Protocol | [10](10_Networking_Basics.md) |
| IPS | Intrusion Prevention System | [15](15_Security_Hardening.md) |
| IoT | Internet of Things | [01](01_Introduction_to_Linux.md) |
| IOPS | Input/Output Operations Per Second | [13](13_Storage_and_Disk_Management.md) |
| LDAP | Lightweight Directory Access Protocol | [07](07_User_and_Permission_Management.md) |
| LFI | Local File Inclusion | [19](19_Exploitation_Basics.md) |
| LVM | Logical Volume Manager | [13](13_Storage_and_Disk_Management.md) |
| MAC | Mandatory Access Control | [15](15_Security_Hardening.md) |
| MBR | Master Boot Record | [13](13_Storage_and_Disk_Management.md) |
| MTU | Maximum Transmission Unit | [14](14_Advanced_Networking.md) |
| NAT | Network Address Translation | [14](14_Advanced_Networking.md) |
| NFS | Network File System | [13](13_Storage_and_Disk_Management.md) |
| NTP | Network Time Protocol | [12](12_System_Administration.md) |
| NVD | National Vulnerability Database | [18](18_Vulnerability_Assessment.md) |
| OSCP | Offensive Security Certified Professional | [16](16_Introduction_to_Ethical_Hacking.md) |
| OSINT | Open-Source Intelligence | [17](17_Network_Reconnaissance.md) |
| OSI | Open Systems Interconnection | [10](10_Networking_Basics.md) |
| OWASP | Open Web Application Security Project | [16](16_Introduction_to_Ethical_Hacking.md) |
| PAM | Pluggable Authentication Modules | [07](07_User_and_Permission_Management.md) |
| PID | Process Identifier | [08](08_Process_Management.md) |
| POSIX | Portable Operating System Interface | [01](01_Introduction_to_Linux.md) |
| PPA | Personal Package Archive | [09](09_Package_Management.md) |
| PPID | Parent Process Identifier | [08](08_Process_Management.md) |
| RAID | Redundant Array of Independent Disks | [13](13_Storage_and_Disk_Management.md) |
| RAM | Random Access Memory | [08](08_Process_Management.md) |
| RCE | Remote Code Execution | [19](19_Exploitation_Basics.md) |
| RFI | Remote File Inclusion | [19](19_Exploitation_Basics.md) |
| RPM | RPM Package Manager | [09](09_Package_Management.md) |
| SGID | Set Group ID | [07](07_User_and_Permission_Management.md) |
| SMB | Server Message Block | [17](17_Network_Reconnaissance.md) |
| SMTP | Simple Mail Transfer Protocol | [10](10_Networking_Basics.md) |
| SNMP | Simple Network Management Protocol | [17](17_Network_Reconnaissance.md) |
| SQL | Structured Query Language | [18](18_Vulnerability_Assessment.md) |
| SQLi | SQL Injection | [18](18_Vulnerability_Assessment.md) |
| SSH | Secure Shell | [10](10_Networking_Basics.md) |
| SSL/TLS | Secure Sockets Layer / Transport Layer Security | [15](15_Security_Hardening.md) |
| SUID | Set User ID | [07](07_User_and_Permission_Management.md) |
| TCP | Transmission Control Protocol | [10](10_Networking_Basics.md) |
| TTY | Teletypewriter (terminal) | [04](04_The_Terminal_Basics.md) |
| UDP | User Datagram Protocol | [10](10_Networking_Basics.md) |
| UID | User Identifier | [07](07_User_and_Permission_Management.md) |
| UEFI | Unified Extensible Firmware Interface | [03](03_Installation_and_Setup.md) |
| USB | Universal Serial Bus | [03](03_Installation_and_Setup.md) |
| VM | Virtual Machine | [03](03_Installation_and_Setup.md) |
| VPN | Virtual Private Network | [14](14_Advanced_Networking.md) |
| VLAN | Virtual Local Area Network | [14](14_Advanced_Networking.md) |
| WSL | Windows Subsystem for Linux | [03](03_Installation_and_Setup.md) |
| XSS | Cross-Site Scripting | [19](19_Exploitation_Basics.md) |

> 📖 Full definitions, usage notes, and shell aliases: **[Appendix B: Aliases and Acronyms](B_Aliases_and_Acronyms.md)**

---

*Index version 2.0 | Ubuntu 22.04 LTS | [Back to README](README.md)*

> 🐧 **Linux mastery is a journey, not a destination. Every command you learn is a new key to the system.**
