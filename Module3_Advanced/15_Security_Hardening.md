# 15 — Security Hardening

> **Module 3 · Lesson 5** | Difficulty: ★★★★☆ Advanced | Time: ~120 min

---

## Learning Objectives

- Harden a Linux system against common attacks
- Configure UFW and iptables/nftables firewalls
- Secure SSH configuration
- Implement SELinux and AppArmor policies
- Set up intrusion detection (Fail2ban, Snort)
- Perform system auditing and log analysis
- Apply CIS Benchmark hardening

---

## Table of Contents

1. [Security Hardening Principles](#1-security-hardening-principles)
2. [SSH Hardening](#2-ssh-hardening)
3. [Firewall Configuration](#3-firewall-configuration)
4. [iptables/nftables](#4-iptablesnftables)
5. [SELinux and AppArmor](#5-selinux-and-apparmor)
6. [Fail2ban — Intrusion Prevention](#6-fail2ban--intrusion-prevention)
7. [Intrusion Detection: Snort/Suricata](#7-intrusion-detection-snortsuricata)
8. [System Auditing](#8-system-auditing)
9. [Log Analysis and SIEM Concepts](#9-log-analysis-and-siem-concepts)
10. [CIS Benchmark Hardening](#10-cis-benchmark-hardening)
11. [Practice Exercises](#11-practice-exercises)
12. [Key Takeaways](#12-key-takeaways)

---

## 1. Security Hardening Principles

### Defense in Depth

```
┌─────────────────────────────────────────────────────┐
│                  PERIMETER                          │
│  ┌───────────────────────────────────────────────┐  │
│  │               NETWORK                        │  │
│  │  ┌─────────────────────────────────────────┐ │  │
│  │  │            HOST                        │ │  │
│  │  │  ┌───────────────────────────────────┐ │ │  │
│  │  │  │        APPLICATION               │ │ │  │
│  │  │  │  ┌─────────────────────────────┐ │ │ │  │
│  │  │  │  │          DATA              │ │ │ │  │
│  │  │  │  └─────────────────────────────┘ │ │ │  │
│  │  │  └───────────────────────────────────┘ │ │  │
│  │  └─────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Principle of Least Privilege

Every user, process, and service should have the **minimum permissions** needed to function.

```bash
# Check for unnecessary SUID binaries
find / -perm -4000 2>/dev/null

# Check for world-writable files
find / -perm -o+w -not -type l 2>/dev/null | grep -v /proc | grep -v /sys

# Check for empty password accounts
sudo awk -F: '($2 == "" ) {print $1}' /etc/shadow

# Check accounts with UID 0 (should be only root)
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

---

## 2. SSH Hardening

SSH is often the primary attack vector. Harden it:

```bash
sudo vim /etc/ssh/sshd_config
```

### Key sshd_config Settings

```bash
# /etc/ssh/sshd_config — hardened configuration

# Change default port (reduces automated scanning)
Port 2222

# Only allow SSH protocol 2 (protocol 1 is broken)
Protocol 2

# Listen on specific interface only (not 0.0.0.0)
ListenAddress 192.168.1.100

# Disable root login
PermitRootLogin no

# Require key-based authentication only
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding (unless needed)
X11Forwarding no

# Disable agent forwarding (unless needed)
AllowAgentForwarding no

# Idle timeout (seconds) — disconnect inactive sessions
ClientAliveInterval 300
ClientAliveCountMax 3

# Limit login attempts
MaxAuthTries 3
MaxSessions 5

# Allow specific users only
AllowUsers alice bob
AllowGroups sshusers

# Disable obsolete features
UseDNS no
PermitTunnel no
GatewayPorts no

# Logging
LogLevel VERBOSE
SyslogFacility AUTH
```

```bash
# After editing, test configuration
sudo sshd -t

# Restart SSH
sudo systemctl restart sshd

# Verify your changes (from another terminal before closing current!)
ssh -p 2222 user@server
```

### SSH Key Best Practices

```bash
# Generate strong key (Ed25519 is best)
ssh-keygen -t ed25519 -C "your@email.com" -f ~/.ssh/id_ed25519

# Or RSA 4096-bit (for legacy compatibility)
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# Set correct permissions
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys

# Protect authorized_keys
chattr +i ~/.ssh/authorized_keys   # immutable (root can't modify easily)

# Set password on private key
ssh-keygen -p -f ~/.ssh/id_ed25519  # add passphrase
```

---

## 3. Firewall Configuration

### UFW (Uncomplicated Firewall)

UFW is the recommended firewall for Ubuntu:

```bash
# Install and enable
sudo apt install ufw
sudo ufw enable

# Default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow specific services
sudo ufw allow ssh                 # allow SSH (port 22)
sudo ufw allow 2222/tcp            # allow SSH on custom port
sudo ufw allow http                # allow HTTP (port 80)
sudo ufw allow https               # allow HTTPS (port 443)
sudo ufw allow 8080/tcp            # allow specific port

# Allow from specific IP
sudo ufw allow from 192.168.1.0/24    # allow subnet
sudo ufw allow from 10.0.0.5 to any port 22  # specific IP to SSH

# Deny specific services
sudo ufw deny 23                   # deny Telnet
sudo ufw deny from 192.168.1.50    # deny from specific IP

# Rate limiting (anti-brute-force)
sudo ufw limit ssh                 # limit SSH connections (max 6/30s)

# Delete rules
sudo ufw delete allow http         # delete by rule
sudo ufw delete 3                  # delete rule #3
sudo ufw show numbered             # show rules with numbers

# Status
sudo ufw status verbose
sudo ufw status numbered

# Logging
sudo ufw logging on
sudo ufw logging medium

# Reset all rules
sudo ufw reset
```

---

## 4. iptables/nftables

### iptables — The Classic Linux Firewall

```bash
# View current rules
sudo iptables -L                   # list all
sudo iptables -L -v -n             # verbose, numeric
sudo iptables -L INPUT -v -n       # just INPUT chain
sudo iptables-save                 # save-format output

# iptables tables and chains:
# Tables: filter (default), nat, mangle, raw
# Chains: INPUT, OUTPUT, FORWARD (in filter table)
#         PREROUTING, POSTROUTING (in nat table)

# Set default policies
sudo iptables -P INPUT DROP        # deny all incoming by default
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ICMP (ping)
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Rate limit new SSH connections (anti-brute-force)
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
    -m recent --set --name SSH
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
    -m recent --update --name SSH --seconds 60 --hitcount 4 -j DROP

# Drop invalid packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Log and drop everything else
sudo iptables -A INPUT -j LOG --log-prefix "iptables-dropped: "
sudo iptables -A INPUT -j DROP

# Save rules (persist across reboots)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### nftables — Modern Replacement for iptables

```bash
# Ubuntu 22.04 uses nftables as the backend
sudo nft list ruleset               # view current rules

# nftables configuration file
sudo vim /etc/nftables.conf

# Example nftables ruleset:
cat << 'EOF' | sudo tee /etc/nftables.conf
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # Accept established/related connections
        ct state established,related accept

        # Accept loopback
        iif lo accept

        # Accept ICMP
        ip protocol icmp accept
        ip6 nexthdr ipv6-icmp accept

        # Rate-limit SSH
        tcp dport 22 ct state new limit rate 3/minute accept

        # Accept HTTP/HTTPS
        tcp dport { 80, 443 } accept

        # Log and drop
        log prefix "nft-dropped: " flags all
        drop
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}
EOF

sudo systemctl enable nftables
sudo systemctl start nftables
```

---

## 5. SELinux and AppArmor

### AppArmor (Ubuntu Default)

```bash
# Check AppArmor status
sudo apparmor_status
sudo aa-status

# View profiles
ls /etc/apparmor.d/

# Profile modes:
# enforce — blocks violations and logs
# complain — only logs, doesn't block
# disabled — profile inactive

# Change profile mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx    # enforce
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx   # complain
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx    # disable

# Reload profiles
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx

# View logs for AppArmor violations
sudo journalctl -u apparmor
sudo dmesg | grep apparmor
sudo grep apparmor /var/log/syslog
```

### Create a Custom AppArmor Profile

```bash
# Install tools
sudo apt install apparmor-utils

# Auto-generate profile in complain mode
sudo aa-genprof /path/to/program

# Then run the program, perform all normal operations
# Press (S)can to see what was logged
# Press (F)inish to finalize

# Example profile structure:
cat /etc/apparmor.d/usr.bin.firefox | head -30
```

### SELinux (Red Hat-based systems)

```bash
# On Ubuntu, install SELinux (not default)
sudo apt install selinux-basics selinux-policy-default auditd

# Check SELinux status
sestatus
getenforce          # Enforcing, Permissive, or Disabled

# Change mode temporarily
sudo setenforce 0   # Permissive (allow but log)
sudo setenforce 1   # Enforcing (block)

# Change permanently: edit /etc/selinux/config
sudo sed -i 's/SELINUX=.*/SELINUX=enforcing/' /etc/selinux/config

# View file contexts
ls -Z file.txt
ps -eZ | grep nginx

# Change file context
sudo chcon -t httpd_content_t /var/www/html/
sudo restorecon -Rv /var/www/html/

# View policy booleans
getsebool -a | grep httpd
sudo setsebool -P httpd_can_network_connect on  # -P = persistent
```

---

## 6. Fail2ban — Intrusion Prevention

Fail2ban monitors logs and bans IPs that show malicious signs.

```bash
sudo apt install fail2ban

# Configuration (never edit jail.conf directly!)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```

```ini
# /etc/fail2ban/jail.local

[DEFAULT]
# Ban time in seconds (or m/h/d)
bantime  = 1h
findtime = 10m
maxretry = 5

# Your whitelist
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24

# Email notifications
destemail = admin@example.com
sendername = Fail2Ban
mta = sendmail
action = %(action_mwl)s

[sshd]
enabled = true
port    = 22
filter  = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime  = 24h

[nginx-http-auth]
enabled = true
filter  = nginx-http-auth
port    = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter  = nginx-limit-req
port    = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
```

```bash
# Start and enable
sudo systemctl enable --now fail2ban

# Status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# View banned IPs
sudo fail2ban-client status sshd | grep "Banned IP"
sudo iptables -L fail2ban-sshd

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.50

# Test a filter
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# Reload config
sudo fail2ban-client reload
```

---

## 7. Intrusion Detection: Snort/Suricata

### Suricata (Modern IDS/IPS)

```bash
sudo apt install suricata

# Update rules
sudo suricata-update

# Test configuration
sudo suricata -T -c /etc/suricata/suricata.yaml -v

# Run in IDS mode (detection only)
sudo suricata -c /etc/suricata/suricata.yaml -i eth0

# Run as a service
sudo systemctl enable --now suricata

# View alerts
sudo tail -f /var/log/suricata/fast.log
sudo tail -f /var/log/suricata/eve.json | python3 -m json.tool

# Test with EICAR test
curl http://www.eicar.org/download/eicar.com 2>/dev/null
# Suricata should detect this
```

### OSSEC/Wazuh (Host-Based IDS)

```bash
# Install Wazuh agent
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor | sudo tee /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-agent
```

---

## 8. System Auditing

### auditd — Linux Audit Daemon

```bash
sudo apt install auditd audispd-plugins

# Start and enable
sudo systemctl enable --now auditd

# View audit log
sudo ausearch -k all
sudo aureport --summary

# Add audit rules
# Format: -w path -p perms -k keyword

# Monitor /etc/passwd changes
sudo auditctl -w /etc/passwd -p wa -k passwd_changes

# Monitor /etc/sudoers
sudo auditctl -w /etc/sudoers -p wa -k sudoers_changes

# Monitor SUID execution
sudo auditctl -a always,exit -F arch=b64 -F perm=x -F auid>=1000 \
    -F auid!=unset -k user_exec

# Monitor network connections
sudo auditctl -a always,exit -F arch=b64 -S connect -k network_connect

# Persist rules
sudo vim /etc/audit/rules.d/hardening.rules
# Add your rules, then:
sudo augenrules --load
```

### Lynis — Security Audit Tool

```bash
sudo apt install lynis

# Run full system audit
sudo lynis audit system

# Audit specific category
sudo lynis audit system --tests-from-group authentication
sudo lynis audit system --tests-from-group networking

# View report
sudo cat /var/log/lynis.log
sudo cat /var/log/lynis-report.dat | grep "suggestion"
```

### Rootkit Detection

```bash
# chkrootkit
sudo apt install chkrootkit
sudo chkrootkit

# rkhunter
sudo apt install rkhunter
sudo rkhunter --update
sudo rkhunter --check
sudo rkhunter --check --skip-keypress    # non-interactive

# View results
sudo cat /var/log/rkhunter.log
```

---

## 9. Log Analysis and SIEM Concepts

### Key Log Files

```bash
# Authentication
sudo tail -f /var/log/auth.log          # login attempts, sudo, PAM

# System
sudo tail -f /var/log/syslog            # general system messages
sudo tail -f /var/log/kern.log          # kernel messages

# Application
sudo tail -f /var/log/nginx/access.log  # web access
sudo tail -f /var/log/nginx/error.log   # web errors
sudo tail -f /var/log/mysql/error.log   # database errors

# Mail
sudo tail -f /var/log/mail.log

# Audit
sudo tail -f /var/log/audit/audit.log
```

### Analyzing Logs for Attacks

```bash
# Find failed SSH login attempts
grep "Failed password" /var/log/auth.log | tail -20

# Count attacks by IP
grep "Failed password" /var/log/auth.log | \
    awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10

# Find successful logins
grep "Accepted" /var/log/auth.log

# Find sudo usage
grep "sudo:" /var/log/auth.log | grep "COMMAND"

# Find new user creation
grep "useradd" /var/log/auth.log

# Web server attacks
grep " 404 " /var/log/nginx/access.log | awk '{print $1}' | \
    sort | uniq -c | sort -rn | head -10

# SQL injection attempts
grep -i "union\|select\|insert\|update\|delete\|drop\|script\|<\|>" \
    /var/log/nginx/access.log | head -20

# Port scan detection (multiple connections in short time)
grep "Invalid user" /var/log/auth.log | \
    awk '{print $NF}' | sort | uniq -c | sort -rn
```

### SIEM Concepts

**SIEM** (Security Information and Event Management) centralizes and correlates logs:

| Component | Purpose |
|-----------|---------|
| **Log Collection** | Gather logs from all sources |
| **Normalization** | Parse into standard format |
| **Correlation** | Match events across sources |
| **Alerting** | Notify on suspicious patterns |
| **Forensics** | Historical investigation |

Popular SIEM tools:
- **ELK Stack** (Elasticsearch + Logstash + Kibana) — open source
- **Splunk** — enterprise
- **Graylog** — open source
- **Wazuh** — security-focused SIEM
- **Security Onion** — Linux distro built for SIEM

```bash
# Simple ELK-style log shipping with Filebeat
sudo apt install filebeat
sudo filebeat setup -e
sudo systemctl enable --now filebeat
```

---

## 10. CIS Benchmark Hardening

The **CIS (Center for Internet Security) Benchmark** provides security hardening guidelines.

### Essential CIS Hardening Steps

```bash
# 1. Disable unused network protocols
cat << 'EOF' | sudo tee /etc/modprobe.d/disable-protocols.conf
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true
EOF

# 2. Kernel parameter hardening
cat << 'EOF' | sudo tee /etc/sysctl.d/99-hardening.conf
# Prevent IP spoofing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Disable IP source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Enable SYN cookies (prevent SYN flood)
net.ipv4.tcp_syncookies = 1

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1

# Disable IPv6 if not needed
#net.ipv6.conf.all.disable_ipv6 = 1

# Prevent time-wait assassination
net.ipv4.tcp_rfc1337 = 1

# Randomize ASLR
kernel.randomize_va_space = 2
EOF

sudo sysctl -p /etc/sysctl.d/99-hardening.conf

# 3. Disable core dumps
echo "* hard core 0" | sudo tee -a /etc/security/limits.conf
echo "fs.suid_dumpable = 0" | sudo tee -a /etc/sysctl.d/99-hardening.conf

# 4. Password policy
sudo apt install libpam-pwquality
sudo vim /etc/security/pwquality.conf
# minlen = 14
# dcredit = -1
# ucredit = -1
# ocredit = -1
# lcredit = -1

# 5. Lock screen after inactivity
echo "TMOUT=600" | sudo tee -a /etc/profile.d/timeout.sh  # 10 minute timeout

# 6. Ensure bootloader is password protected (GRUB)
sudo grub-mkpasswd-pbkdf2  # generates password hash
# Add to /etc/grub.d/40_custom:
# set superusers="admin"
# password_pbkdf2 admin <hash>

# 7. Check for and disable unnecessary services
systemctl list-units --type=service --state=running
sudo systemctl disable --now avahi-daemon  # zero-conf (usually not needed)
sudo systemctl disable --now cups          # printing (if not needed)
sudo systemctl disable --now bluetooth     # bluetooth (if not needed)
```

---

## 11. Practice Exercises

### Exercise 15.1 — SSH Hardening

```bash
# 1. Backup current SSH config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# 2. Apply these settings:
sudo vim /etc/ssh/sshd_config
# - Disable root login
# - Disable password authentication
# - Set MaxAuthTries to 3

# 3. Test config
sudo sshd -t

# 4. Generate SSH key pair (if you don't have one)
ssh-keygen -t ed25519 -C "test"

# 5. Restart SSH
sudo systemctl restart sshd

# 6. Verify settings are applied
sudo grep -E "PermitRootLogin|PasswordAuthentication|MaxAuthTries" /etc/ssh/sshd_config
```

### Exercise 15.2 — Firewall Setup

```bash
# 1. Check current UFW status
sudo ufw status verbose

# 2. Set up a web server firewall:
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# 3. Verify
sudo ufw status verbose

# 4. Test - try connecting to different ports (from another machine)
# nc -zv server 80    (should succeed)
# nc -zv server 8080  (should fail)

# 5. Check logs
sudo tail /var/log/ufw.log
```

### Exercise 15.3 — Fail2ban Configuration

```bash
# 1. Install fail2ban
sudo apt install -y fail2ban

# 2. Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# 3. Configure SSH jail
sudo vim /etc/fail2ban/jail.local
# Under [sshd] section:
# enabled = true
# maxretry = 3
# bantime = 1h

# 4. Start fail2ban
sudo systemctl enable --now fail2ban

# 5. Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# 6. Simulate failed logins (from another machine)
# ssh wrongpassword@yourserver
# Check if IP gets banned after 3 failures
```

---

## 12. Key Takeaways

- **Defense in depth**: multiple security layers; no single point of failure
- **SSH hardening**: disable root, use key auth only, change port, rate limit
- **UFW** is the easiest firewall; **iptables/nftables** provide more control
- **AppArmor** profiles restrict program capabilities; **SELinux** on RHEL systems
- **Fail2ban** automatically bans brute-force attackers
- **Snort/Suricata** detect network intrusion attempts
- **auditd** provides detailed system activity logging
- **Lynis** and **rkhunter** audit system security posture
- **SIEM** correlates logs from multiple sources for threat detection
- CIS Benchmarks provide vendor-agnostic hardening checklists

---

## Module 3 Complete! 🎉

You're now proficient in advanced Linux — scripting, administration, storage, networking, and security hardening.

---

## Next Module

➡️ [Module 4: Hacker/Pentesting — Introduction to Ethical Hacking](../Module4_Hacker_Pentesting/16_Introduction_to_Ethical_Hacking.md)

---

*Module 3 · Lesson 5 of 5 | [Course Index](../INDEX.md)*
