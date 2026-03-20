# 15. Security Hardening

> **Difficulty:** Advanced | **Time Estimate:** 4–5 hours | **Prerequisites:** Lessons 1–14

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Apply the principle of least privilege to harden a Linux system
- Configure SSH to use key-based authentication and disable unsafe defaults
- Deploy fail2ban to block brute-force attacks automatically
- Use AppArmor to confine applications with mandatory access controls
- Monitor file integrity with AIDE
- Implement password policies and account lockout via PAM
- Generate and manage SSL/TLS certificates and GPG keys
- Run automated security audits with Lynis and rkhunter

---

## 1. Security Hardening Principles

```
Principle of Least Privilege    → Grant only the minimum access required
Defense in Depth                → Multiple layers; no single point of failure
Attack Surface Reduction        → Disable unused services, ports, and accounts
Separation of Duties            → No single account should control everything
Fail Securely                   → Default-deny; allow only what is needed
Auditability                    → Log everything; know who did what and when
Patch Management                → Keep software up to date
```

```bash
# Immediate hardening checklist
sudo apt update && sudo apt upgrade -y    # Patch everything
sudo apt install unattended-upgrades      # Auto security updates
sudo dpkg-reconfigure unattended-upgrades # Configure

# View listening services (reduce attack surface)
ss -tnlp
sudo netstat -tnlp

# Check for running services
systemctl list-units --type=service --state=running

# Disable unused services
sudo systemctl disable --now avahi-daemon  # mDNS (usually unneeded)
sudo systemctl disable --now cups          # Printing (on servers)
```

---

## 2. SSH Hardening

The SSH daemon config lives at `/etc/ssh/sshd_config`. Always backup before editing.

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Recommended hardened sshd_config settings:
cat << 'EOF' | sudo tee -a /etc/ssh/sshd_config.d/99-hardening.conf
# Change default port (reduces automated scan noise)
Port 2222

# Protocol and key exchange
Protocol 2
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group14-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Authentication
PermitRootLogin no
MaxAuthTries 3
MaxSessions 5
AuthenticationMethods publickey
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
PubkeyAuthentication yes

# Access restrictions
AllowUsers alice bob deploy
AllowGroups sshusers
DenyUsers guest nobody

# Timeout settings
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
TCPKeepAlive no

# Disable dangerous features
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no         # Unless tunneling is required
PermitTunnel no

# Banner warning
Banner /etc/ssh/banner.txt

# Logging
LogLevel VERBOSE
EOF

# Validate config before restarting
sudo sshd -t && echo "Config OK"
sudo systemctl restart sshd
```

### SSH Key Management

```bash
# Generate key pair (user side)
ssh-keygen -t ed25519 -C "alice@company.com" -f ~/.ssh/id_ed25519
ssh-keygen -t rsa -b 4096 -C "alice@company.com"   # RSA alternative

# Copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server
# Or manually:
cat ~/.ssh/id_ed25519.pub | ssh alice@server 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# Secure key files
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519

# SSH agent (avoid repeated passphrase entry)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l          # List loaded keys

# Rotate authorized_keys (remove old keys)
sudo grep -r "authorized_keys" /home/*/
sudo find /home -name "authorized_keys" -exec cat {} \;
```

---

## 3. fail2ban

fail2ban monitors log files for repeated failure patterns and automatically bans offending IPs using iptables/nftables.

```bash
# Install
sudo apt install fail2ban

# Configuration (never edit jail.conf — use jail.local)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Key settings in jail.local:
cat << 'EOF'
[DEFAULT]
bantime  = 3600           # Ban for 1 hour (use -1 for permanent)
findtime = 600            # Time window to count failures
maxretry = 5              # Failures before ban
ignoreip = 127.0.0.1/8 192.168.1.0/24  # Never ban these
backend  = systemd        # Use journald for Ubuntu 20.04+

[sshd]
enabled  = true
port     = 2222           # Match your SSH port
filter   = sshd
logpath  = %(sshd_log)s
maxretry = 3
bantime  = 86400          # 24h for SSH failures

[nginx-http-auth]
enabled  = true
filter   = nginx-http-auth
logpath  = /var/log/nginx/error.log
maxretry = 5
EOF

# Custom filter example
cat << 'EOF' | sudo tee /etc/fail2ban/filter.d/myapp.conf
[Definition]
failregex = ^.*\[error\].*authentication failure.*from <HOST>.*$
ignoreregex =
EOF

# Start and manage
sudo systemctl enable --now fail2ban
sudo fail2ban-client status                # Overview
sudo fail2ban-client status sshd          # Jail-specific status
sudo fail2ban-client set sshd unbanip 1.2.3.4   # Manually unban
sudo fail2ban-client reload               # Reload config
sudo fail2ban-client ping                 # Test if daemon is running

# Test a filter
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# View banned IPs
sudo iptables -L f2b-sshd -v -n
```

---

## 4. AppArmor

AppArmor enforces **Mandatory Access Control (MAC)** — restricting what resources a program can access, regardless of the user running it.

```bash
# Status
sudo aa-status                          # All profiles and their modes
sudo apparmor_status | grep -E "enforce|complain"

# Modes:
# enforce   → rules are enforced; violations denied and logged
# complain  → rules are monitored; violations logged but NOT denied
# disabled  → profile not loaded

# Manage profiles
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx    # Enable enforcement
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx   # Switch to complain
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx    # Disable profile
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx  # Reload profile

# Create a new profile interactively
sudo aa-genprof /usr/local/bin/myapp   # Generate profile
# (Run the app in another terminal, then return and press 'S' to scan)

# Profile syntax example
cat << 'EOF'
/usr/local/bin/myapp {
  #include <abstractions/base>

  /usr/local/bin/myapp  mr,          # Can read and mmap itself
  /etc/myapp/**         r,           # Read config files
  /var/log/myapp/       w,           # Write log directory
  /var/log/myapp/*.log  w,           # Write log files
  /tmp/myapp.*          rw,          # Temp files
  network tcp,                       # Allow TCP connections
  deny /etc/passwd      r,           # Explicitly deny
}
EOF

# View AppArmor denials in logs
sudo dmesg | grep apparmor
sudo journalctl -k | grep apparmor
sudo grep apparmor /var/log/syslog | tail -20

# List loaded profiles
ls /etc/apparmor.d/
```

---

## 5. SELinux Concepts (Reference)

SELinux is default on RHEL/CentOS/Fedora systems. Unlike AppArmor (path-based), SELinux uses security labels.

```bash
# Check status (RHEL/CentOS)
getenforce              # Enforcing / Permissive / Disabled
sestatus -v             # Detailed status

# Change mode temporarily
sudo setenforce 0       # Permissive (for debugging)
sudo setenforce 1       # Back to Enforcing

# Persistent: edit /etc/selinux/config
# SELINUX=enforcing | permissive | disabled

# View denials
sudo ausearch -m avc -ts today
sudo audit2why < /var/log/audit/audit.log  # Explain denials

# Common SELinux commands
ls -lZ                  # Show file labels
ps -eZ                  # Show process labels
sudo chcon -t httpd_sys_content_t /var/www/html/file.html  # Change label
sudo restorecon -Rv /var/www/html/                          # Restore default labels
sudo semanage port -a -t http_port_t -p tcp 8080           # Allow new port
sudo audit2allow -M mypolicy < /var/log/audit/audit.log    # Generate policy from denials
```

---

## 6. File Integrity Monitoring with AIDE

AIDE (Advanced Intrusion Detection Environment) detects unauthorised file changes.

```bash
# Install
sudo apt install aide aide-common

# Configuration
cat /etc/aide/aide.conf       # View defaults
# Key settings:
# database = file:@@{DBDIR}/aide.db
# database_out = file:@@{DBDIR}/aide.db.new

# Customise what to monitor
cat << 'EOF' | sudo tee -a /etc/aide/aide.conf
# Monitor critical system files
/etc            PERMS+SHA256
/bin            ReadonlyWithPerms
/sbin           ReadonlyWithPerms
/usr/bin        ReadonlyWithPerms
/boot           ReadonlyWithPerms
!/var/log       # Exclude log files (they change constantly)
!/tmp           # Exclude temp files
EOF

# Initialise database (baseline snapshot)
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run integrity check
sudo aide --check             # Compare current state to baseline
sudo aide --check 2>&1 | tee /var/log/aide-check.log

# Update database after legitimate changes
sudo aide --update
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Automate with cron
echo "0 3 * * * root aide --check | mail -s 'AIDE Report' admin@example.com" \
    | sudo tee /etc/cron.d/aide-check
```

---

## 7. Audit System (auditd)

```bash
# Install and start
sudo apt install auditd audispd-plugins
sudo systemctl enable --now auditd

# Audit rules
sudo auditctl -l                    # List active rules
sudo auditctl -w /etc/passwd -p wa -k passwd-changes  # Watch file
sudo auditctl -w /etc/sudoers -p wa -k sudoers-changes
sudo auditctl -a always,exit -F arch=b64 -S execve -k exec-tracking

# Persistent rules
cat << 'EOF' | sudo tee /etc/audit/rules.d/custom.rules
# Monitor authentication files
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k sudoers

# Monitor privileged commands
-a always,exit -F path=/usr/bin/sudo -F perm=x -k privileged

# Monitor SSH login attempts
-w /var/log/auth.log -p r -k auth-log-access

# Track all commands run as root
-a always,exit -F arch=b64 -F euid=0 -S execve -k root-commands
EOF
sudo augenrules --load

# Searching audit logs
sudo ausearch -k passwd-changes             # By key
sudo ausearch -m USER_LOGIN -ts today       # Login events today
sudo ausearch -ua 1000 -ts recent           # User UID 1000, recently
sudo ausearch -x /usr/bin/sudo              # Sudo usage

# Reports
sudo aureport                               # Summary report
sudo aureport --auth                        # Authentication report
sudo aureport --login                       # Login report
sudo aureport --executable                  # Executable report
sudo aureport -k                            # Report by key
```

---

## 8. Password Policies (PAM)

PAM (Pluggable Authentication Modules) controls authentication, session, account, and password management.

```bash
# Key PAM configuration files
/etc/pam.d/common-auth        # Authentication stack (Debian)
/etc/pam.d/common-password    # Password stack
/etc/pam.d/common-account     # Account management
/etc/pam.d/sshd               # SSH-specific PAM

# Install password quality module
sudo apt install libpam-pwquality

# Configure: /etc/security/pwquality.conf
cat << 'EOF' | sudo tee /etc/security/pwquality.conf
minlen = 14           # Minimum password length
minclass = 3          # At least 3 character classes
maxrepeat = 3         # Max 3 consecutive identical chars
maxsequence = 4       # No sequences like "abcd"
dcredit = -1          # At least 1 digit
ucredit = -1          # At least 1 uppercase
lcredit = -1          # At least 1 lowercase
ocredit = -1          # At least 1 special character
difok = 5             # Differ from old password by 5 chars
gecoscheck = 1        # Don't include username in password
EOF

# PAM for account lockout
sudo apt install libpam-faillock
# Add to /etc/pam.d/common-auth:
# auth required pam_faillock.so preauth silent deny=5 unlock_time=900
# auth [success=1 default=bad] pam_unix.so
# auth [default=die] pam_faillock.so authfail deny=5 unlock_time=900

# /etc/login.defs — system-wide password aging
sudo nano /etc/login.defs
# Key values:
# PASS_MAX_DAYS   90      # Password expires in 90 days
# PASS_MIN_DAYS   7       # Minimum 7 days before changing again
# PASS_WARN_AGE   14      # Warn 14 days before expiry

# Apply aging to existing user
sudo chage -M 90 -m 7 -W 14 alice
sudo chage -l alice        # View password aging info

# Account lockout commands
sudo faillock --user alice                  # View lockout status
sudo faillock --user alice --reset          # Unlock account
sudo pam_tally2 --user alice               # tally2 alternative
sudo pam_tally2 --user alice --reset
```

---

## 9. SSL/TLS Certificates

### OpenSSL

```bash
# Generate self-signed certificate
openssl genrsa -out server.key 4096
openssl req -new -x509 -key server.key -out server.crt -days 365 \
    -subj "/C=US/ST=California/L=San Francisco/O=Example Inc/CN=example.com"

# Generate CSR (for CA signing)
openssl genrsa -out server.key 4096
openssl req -new -key server.key -out server.csr
# Submit server.csr to your CA

# View certificate details
openssl x509 -in server.crt -text -noout
openssl x509 -in server.crt -noout -enddate    # Expiry date
openssl x509 -in server.crt -noout -subject    # Subject

# Verify certificate against CA
openssl verify -CAfile ca.crt server.crt

# Check remote certificate
echo | openssl s_client -connect example.com:443 2>/dev/null | \
    openssl x509 -noout -dates -subject

# Create CA and sign certificate
openssl genrsa -out ca.key 4096
openssl req -new -x509 -key ca.key -out ca.crt -days 3650 \
    -subj "/CN=My CA"
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out server.crt -days 365
```

### Let's Encrypt (Certbot)

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com
sudo certbot certonly --standalone -d example.com  # Without web server

# Renew certificates
sudo certbot renew --dry-run        # Test renewal
sudo certbot renew                  # Renew all near-expiry certs

# Auto-renewal (certbot installs a timer automatically)
systemctl status certbot.timer

# Certificate locations
ls /etc/letsencrypt/live/example.com/
# fullchain.pem  privkey.pem  cert.pem  chain.pem
```

---

## 10. GPG Keys

```bash
# Generate a GPG key pair
gpg --gen-key                              # Interactive (recommended)
gpg --full-generate-key                    # Full options

# List keys
gpg --list-keys                            # Public keys
gpg --list-secret-keys                     # Private keys
gpg --fingerprint alice@example.com        # Show fingerprint

# Export/import keys
gpg --export --armor alice@example.com > alice-public.gpg
gpg --export-secret-keys --armor alice@example.com > alice-private.gpg
gpg --import alice-public.gpg
gpg --import alice-private.gpg

# Encrypt a file (for recipient)
gpg --recipient bob@example.com --encrypt secret.txt
gpg --encrypt --armor --recipient bob@example.com secret.txt

# Decrypt a file
gpg --decrypt secret.txt.gpg > secret.txt
gpg --output secret.txt --decrypt secret.txt.gpg

# Sign a file
gpg --sign document.txt                    # Binary signature
gpg --clearsign document.txt               # Inline clear-text signature
gpg --detach-sign document.txt             # Separate .sig file

# Verify signature
gpg --verify document.txt.sig document.txt

# Key server operations
gpg --keyserver keyserver.ubuntu.com --search-keys alice@example.com
gpg --keyserver keyserver.ubuntu.com --send-keys KEYID

# Sign someone's key (web of trust)
gpg --sign-key bob@example.com
gpg --lsign-key bob@example.com     # Local sign (not exportable)

# Revoke a key
gpg --gen-revoke alice@example.com > revocation.asc
gpg --import revocation.asc         # Publish revocation
```

---

## 11. Security Scanning Tools

### Lynis (System Auditor)

```bash
# Install
sudo apt install lynis

# Run system audit
sudo lynis audit system
sudo lynis audit system --quiet      # Less verbose
sudo lynis audit system --no-colors > /tmp/lynis-report.txt

# View hardening index and suggestions
# Look for: Hardening index: XX [##########]
# Review: Suggestions and Warnings sections

# Update Lynis
sudo lynis update info
sudo lynis update release
```

### rkhunter and chkrootkit

```bash
# Install
sudo apt install rkhunter chkrootkit

# rkhunter — rootkit hunter
sudo rkhunter --update              # Update signature database
sudo rkhunter --check               # Full system scan
sudo rkhunter --check --rwo         # Only show warnings
sudo rkhunter --check --sk          # Skip key press prompts
sudo rkhunter --propupd             # Update file properties database

# Run weekly via cron
echo "0 4 * * 0 root rkhunter --check --cronjob 2>&1 | mail -s 'rkhunter Report' admin@example.com" \
    | sudo tee /etc/cron.d/rkhunter

# chkrootkit
sudo chkrootkit                     # Full scan
sudo chkrootkit -q                  # Quiet (only show infections)
```

---

## 12. CIS Benchmarks Overview

The Center for Internet Security (CIS) publishes hardening guidelines. Key areas:

```
CIS Section    Topic                         Example Controls
───────────────────────────────────────────────────────────────
1. Filesystem   Partition layout, noexec     /tmp on separate partition
2. Software     Package management           Remove unnecessary packages
3. Logging      Rsyslog, auditd config       Log all privileged commands
4. Network      IP forwarding, ICMP          Disable IP source routing
5. Access       PAM, su restrictions         Restrict su to wheel group
6. Cron         Cron daemon permissions      Restrict cron to authorized users
7. SSH          sshd_config hardening        Matches our Section 2
```

```bash
# Quick CIS-inspired checks
# Check /tmp is separate partition
findmnt /tmp

# Verify noexec on /tmp
mount | grep /tmp

# Check world-writable files
find / -xdev -type f -perm -0002 2>/dev/null | head

# Check SUID/SGID files
find / -xdev -perm /6000 -type f 2>/dev/null

# Verify no empty passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Check for duplicate UIDs (should be none except 0)
awk -F: 'seen[$3]++{print $1, $3}' /etc/passwd
```

---

## ASCII Diagram: Defense-in-Depth Model

```
                 ┌─────────────────────────────┐
                 │     Physical Security        │  Layer 7
                 └────────────┬────────────────┘
                              │
                 ┌────────────▼────────────────┐
                 │   Network Perimeter (UFW)   │  Layer 6
                 └────────────┬────────────────┘
                              │
                 ┌────────────▼────────────────┐
                 │  fail2ban / Rate Limiting   │  Layer 5
                 └────────────┬────────────────┘
                              │
                 ┌────────────▼────────────────┐
                 │   SSH Hardening / VPN       │  Layer 4
                 └────────────┬────────────────┘
                              │
                 ┌────────────▼────────────────┐
                 │  AppArmor / SELinux (MAC)   │  Layer 3
                 └────────────┬────────────────┘
                              │
                 ┌────────────▼────────────────┐
                 │ File Integrity (AIDE/auditd)│  Layer 2
                 └────────────┬────────────────┘
                              │
                 ┌────────────▼────────────────┐
                 │  PAM / Password Policies    │  Layer 1
                 └─────────────────────────────┘
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| Disabling `PasswordAuthentication` before testing key auth | Locked out of SSH | Always test key auth in second session first |
| Setting `MaxAuthTries 1` | Locked out by transient errors | Use `3` or `5` as a reasonable minimum |
| Not whitelisting your own IP in fail2ban | Self-ban during testing | Add `ignoreip = your-ip` in `jail.local` |
| Forgetting `sshd -t` before restarting | Bad config breaks SSH | Always validate: `sudo sshd -t` |
| Running AIDE check without updating DB after patch | False positives | Run `aide --update` after system updates |
| Storing private keys without passphrases | Key theft = full access | Always use passphrases on private keys |

---

## Pro Tips

- **Test SSH config in a second session** before closing your current one — a misconfiguration can lock you out permanently
- **`sshd -T`** prints the full effective sshd configuration (including defaults) — use it to audit your settings
- **Encrypt sensitive scripts** with GPG or store secrets in a vault (HashiCorp Vault, pass) — never hardcode passwords
- **`lynis audit system`** produces a Hardening Index score — track it over time to measure improvement
- **`grep -r "FAILED\|WARN\|ERROR" /var/log/audit/`** quickly surfaces issues from auditd
- **Certificate expiry monitoring**: run `certbot renew --dry-run` monthly; add `echo | openssl s_client -connect domain:443` checks to your monitoring

---

## Practice Exercises

1. **SSH Hardening** — Harden `sshd_config` to: disable root login, change port to 2222, require key-only auth, and set `MaxAuthTries 3`. Validate with `sshd -t` and test in a second session.

2. **fail2ban Setup** — Configure fail2ban with a 24-hour SSH ban after 3 failures. Test it by deliberately failing authentication, then confirm the IP is banned and manually unban it.

3. **AppArmor Profile** — Generate an AppArmor profile for a simple Python script that writes to `/tmp`. Set it to enforce mode and verify violations are blocked and logged.

4. **AIDE Baseline** — Initialise an AIDE database, then make a change to `/etc/motd`, run `aide --check`, and observe the change is detected.

5. **Password Policy** — Configure `pwquality.conf` to require 14-character passwords with all four character classes. Test with `passwd` and verify weak passwords are rejected.

6. **SSL Certificate** — Generate a self-signed certificate for `localhost`, configure nginx to serve it on port 443, and verify with `openssl s_client`.

7. **GPG Encryption** — Generate a GPG key pair, encrypt a file to yourself, decrypt it, then sign another file and verify the signature.

8. **Lynis Audit** — Run a full Lynis audit, identify the top 5 hardening suggestions, implement at least 2 of them, and re-run to verify the hardening index improves.

---

## Key Takeaways

- **Defense in depth** means no single control is relied upon alone — layer them
- SSH hardening is the most important step on internet-facing servers; disable passwords, enable keys
- **fail2ban** watches logs and auto-bans IPs — a must-have on any public SSH service
- **AppArmor/SELinux** provide kernel-level mandatory access control that survives privilege escalation
- **AIDE** and **auditd** provide the audit trail needed to detect breaches and prove compliance
- Certificates prove identity; GPG provides encryption and signing for files, emails, and code
- Run **Lynis** regularly and treat the hardening index as a metric to continuously improve

---

## Next Lesson Preview

**Lesson 16: Containers and Virtualization** — We'll explore Linux containers with Docker and Podman, understand namespaces and cgroups that power containerisation, build container images, and introduce Kubernetes orchestration concepts. The security hardening skills from this lesson are directly applicable to container and VM security.
