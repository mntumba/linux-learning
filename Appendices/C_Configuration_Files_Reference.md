# Appendix C – Linux Configuration Files Reference

A reference guide to the most important configuration files in a Linux system,
including their purpose, location, format, and annotated examples.

---

## Table of Contents
1. [User & Authentication](#1-user--authentication)
2. [System & Boot](#2-system--boot)
3. [Filesystem & Mounts](#3-filesystem--mounts)
4. [Network Configuration](#4-network-configuration)
5. [SSH Configuration](#5-ssh-configuration)
6. [Package Management](#6-package-management)
7. [Logging & Journaling](#7-logging--journaling)
8. [Scheduling](#8-scheduling)
9. [Shell & Environment](#9-shell--environment)
10. [Kernel & Modules](#10-kernel--modules)
11. [Web Servers](#11-web-servers)
12. [Database](#12-database)
13. [Security & PAM](#13-security--pam)

---

## 1. User & Authentication

### `/etc/passwd` – Local User Database

```
# Format: username:x:UID:GID:GECOS:home:shell
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
alice:x:1001:1001:Alice Smith,,,:/home/alice:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

| Field | Description |
|-------|-------------|
| username | Login name (max 32 chars) |
| `x` | Password placeholder (actual hash in `/etc/shadow`) |
| UID | User ID (0 = root, 1–999 = system, 1000+ = regular) |
| GID | Primary group ID |
| GECOS | Full name / comment field |
| home | Home directory |
| shell | Login shell (`/usr/sbin/nologin` for service accounts) |

### `/etc/shadow` – Encrypted Password Store

```
# Format: username:hash:lastchange:min:max:warn:inactive:expire:reserved
root:$6$rounds=500000$salt$hashvalue:19200:0:99999:7:::
alice:$6$XyZ...:18900:0:90:14:30::
locked_user:!$6$hash:18500:::::
```

| Field | Description |
|-------|-------------|
| hash | `$id$salt$hash` – `$6$` = SHA-512, `$5$` = SHA-256, `$2y$` = bcrypt |
| lastchange | Days since epoch of last password change |
| min | Minimum days before change allowed |
| max | Maximum days before change required |
| warn | Days before expiry to warn user |
| inactive | Days after expiry before account disabled |
| expire | Account expiry date (days since epoch); empty = never |

### `/etc/group` – Group Database

```
# Format: groupname:x:GID:member_list
root:x:0:
sudo:x:27:alice,bob
docker:x:999:alice
developers:x:1050:alice,bob,carol
```

### `/etc/sudoers` – Sudo Policy

Edit only with `visudo`!

```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL

# Allow members of sudo group to run all commands
%sudo   ALL=(ALL:ALL) ALL

# Allow alice to restart nginx without password
alice   ALL=(root) NOPASSWD: /bin/systemctl restart nginx

# Allow devops group to run specific commands
%devops ALL=(ALL) /usr/bin/docker, /usr/bin/kubectl

# Include drop-in files
@includedir /etc/sudoers.d
```

### `/etc/nsswitch.conf` – Name Service Switch

```
passwd:     files systemd
group:      files systemd
shadow:     files
hosts:      files dns myhostname
networks:   files
protocols:  db files
services:   db files
```

---

## 2. System & Boot

### `/etc/os-release` – OS Identity

```ini
NAME="Ubuntu"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
ID=ubuntu
ID_LIKE=debian
VERSION_ID="22.04"
PRETTY_NAME="Ubuntu 22.04.3 LTS"
HOME_URL="https://www.ubuntu.com/"
```

### `/etc/hostname` – Hostname

```
webserver-prod-01
```

### `/etc/hosts` – Static Name Resolution

```
# Format: IP address   canonical_name   [aliases]
127.0.0.1       localhost
127.0.1.1       myhost.example.com myhost

# IPv6
::1             localhost ip6-localhost ip6-loopback
fe80::1%eth0    link-local

# Custom entries
192.168.1.10    db-primary db1
192.168.1.11    db-replica db2
10.0.0.50       jenkins.internal jenkins
```

### `/etc/fstab` – Filesystem Table

```
# <device>              <mountpoint>  <fstype>  <options>              <dump> <pass>
UUID=abc123-...         /             ext4      errors=remount-ro      0      1
UUID=def456-...         /boot/efi     vfat      umask=0077             0      1
UUID=789ghi-...         /home         ext4      defaults,noatime       0      2
tmpfs                   /tmp          tmpfs     defaults,size=1G,mode=1777  0  0
//nas.local/share       /mnt/nas      cifs      credentials=/etc/nas-creds,iocharset=utf8  0  0
192.168.1.5:/exports    /mnt/nfs      nfs4      defaults,_netdev       0      0
```

| Option | Description |
|--------|-------------|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `noatime` | Do not update access time (performance) |
| `noexec` | Prevent executing binaries |
| `nosuid` | Ignore setuid/setgid bits |
| `ro` | Mount read-only |
| `_netdev` | Mount after network is available |
| `errors=remount-ro` | Remount read-only on error (ext4) |
| pass 1 | Root fs – checked first |
| pass 2 | Other fs – checked second |
| pass 0 | Skip fsck |

### `/etc/default/grub` – GRUB Boot Loader

```bash
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="Ubuntu"
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
# For servers (no quiet, enable serial console)
# GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
```

```bash
# Apply changes after editing
update-grub           # Debian/Ubuntu
grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS
```

---

## 3. Filesystem & Mounts

### `/etc/crypttab` – Encrypted Volumes

```
# name      device           keyfile   options
home_crypt  /dev/sdb1        none      luks
data_crypt  UUID=abc-...     /etc/key  luks,noauto
```

---

## 4. Network Configuration

### `/etc/resolv.conf` – DNS Resolver

```
# DNS search domain
search example.com internal.example.com

# DNS servers (max 3)
nameserver 1.1.1.1
nameserver 8.8.8.8
nameserver 9.9.9.9

# Options
options ndots:5 timeout:2 attempts:3
```

> **Note:** Often managed by `systemd-resolved` or `NetworkManager`. Direct edits may be overwritten.

### `/etc/network/interfaces` – Debian Static Network

```bash
# Loopback
auto lo
iface lo inet loopback

# DHCP
auto eth0
iface eth0 inet dhcp

# Static IP
auto eth1
iface eth1 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
    dns-search example.com
```

### `/etc/netplan/*.yaml` – Ubuntu Netplan

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      addresses:
        - 192.168.1.10/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
        search: [example.com]
      optional: true
```

```bash
netplan try       # Test with auto-revert
netplan apply     # Apply configuration
```

### `/etc/NetworkManager/system-connections/*.nmconnection`

```ini
[connection]
id=Wired connection 1
type=ethernet
interface-name=eth0

[ipv4]
method=manual
addresses=192.168.1.10/24
gateway=192.168.1.1
dns=8.8.8.8;1.1.1.1;

[ipv6]
method=auto
```

---

## 5. SSH Configuration

### `/etc/ssh/sshd_config` – SSH Daemon

```bash
# Port and address
Port 22
AddressFamily inet
ListenAddress 0.0.0.0

# Authentication
PermitRootLogin prohibit-password    # Disable password root login
PasswordAuthentication no            # Require key-based auth
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Security hardening
PermitEmptyPasswords no
X11Forwarding no
AllowTcpForwarding no
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2

# Restrict to specific users/groups
AllowGroups sshusers admins
DenyUsers guest

# Algorithms (modern secure set)
KexAlgorithms curve25519-sha256,diffie-hellman-group14-sha256
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
MACs hmac-sha2-256-etm@openssh.com

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# SFTP subsystem
Subsystem sftp /usr/lib/openssh/sftp-server

# Match block example
Match Group developers
    X11Forwarding yes
    AllowTcpForwarding yes
```

### `~/.ssh/config` – Client Configuration

```
Host jump-host
    HostName bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/bastion_key
    ServerAliveInterval 60

Host prod-*
    ProxyJump jump-host
    User deployer
    IdentityFile ~/.ssh/deploy_key
    StrictHostKeyChecking yes

Host *
    ServerAliveInterval 120
    ServerAliveCountMax 3
    Compression yes
    ControlMaster auto
    ControlPath ~/.ssh/controlmasters/%r@%h:%p
    ControlPersist 10m
```

---

## 6. Package Management

### `/etc/apt/sources.list` – APT Repositories

```
# Ubuntu 22.04
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu jammy-security main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
```

### `/etc/yum.repos.d/*.repo` – YUM/DNF Repositories

```ini
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
baseurl=https://download.fedoraproject.org/pub/epel/$releasever/Everything/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever
```

---

## 7. Logging & Journaling

### `/etc/rsyslog.conf` – Syslog Configuration

```
# Rules: facility.severity  destination
auth,authpriv.*             /var/log/auth.log
*.*;auth,authpriv.none      /var/log/syslog
kern.*                      /var/log/kern.log
mail.*                      /var/log/mail.log
cron.*                      /var/log/cron.log

# Forward to remote syslog server
*.* @@logserver.example.com:514

# High-performance output
$ActionQueueType LinkedList
$ActionQueueMaxDiskSpace 1g
$ActionResumeRetryCount -1
```

### `/etc/systemd/journald.conf` – Journal Configuration

```ini
[Journal]
Storage=persistent          # persistent / volatile / auto
Compress=yes
SystemMaxUse=2G             # Max disk space for journal
SystemMaxFileSize=128M      # Max size per journal file
MaxRetentionSec=1month      # Retention period
ForwardToSyslog=no
RateLimitBurst=1000
RateLimitIntervalSec=30s
```

---

## 8. Scheduling

### `/etc/crontab` – System Crontab

```
# m  h  dom  mon  dow  user   command
17  *    *    *    *   root   cd / && run-parts --report /etc/cron.hourly
25  6    *    *    *   root   test -x /usr/sbin/anacron || run-parts /etc/cron.daily
47  6    *    *    7   root   run-parts /etc/cron.weekly
52  6    1    *    *   root   run-parts /etc/cron.monthly
```

### `/etc/cron.d/` – Drop-in Cron Files

```
# /etc/cron.d/backup
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@example.com

30 2 * * * root /usr/local/bin/backup.sh
```

---

## 9. Shell & Environment

### `/etc/environment` – System-wide Variables

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
LANG="en_US.UTF-8"
JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
```

### `/etc/profile` and `/etc/profile.d/*.sh`

```bash
# /etc/profile.d/custom.sh
export HISTTIMEFORMAT="%F %T "
export HISTSIZE=10000
export HISTFILESIZE=20000
umask 022
ulimit -c 0   # Disable core dumps
```

### `~/.bashrc` – User Interactive Shell Config

```bash
# Aliases
alias ll='ls -lah --color=auto'
alias la='ls -A'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'

# Custom prompt
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# History settings
HISTCONTROL=ignoredups:erasedups
shopt -s histappend

# Load local scripts
[ -f ~/.bash_functions ] && . ~/.bash_functions
```

---

## 10. Kernel & Modules

### `/etc/sysctl.conf` – Kernel Parameters

```bash
# Network hardening
net.ipv4.ip_forward = 0                  # Disable IP forwarding (non-router)
net.ipv4.conf.all.rp_filter = 1          # Reverse path filtering
net.ipv4.conf.all.accept_redirects = 0   # Reject ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1              # SYN flood protection
net.ipv4.conf.all.log_martians = 1       # Log spoofed packets

# Memory
vm.swappiness = 10                       # Prefer RAM over swap
vm.overcommit_memory = 0

# File descriptors
fs.file-max = 2097152
```

```bash
sysctl -p                  # Reload /etc/sysctl.conf
sysctl net.ipv4.ip_forward # Query a parameter
```

### `/etc/modules` – Auto-loaded Modules

```
# Load at boot
loop
nf_conntrack
dm_crypt
```

### `/etc/modprobe.d/*.conf` – Module Options

```bash
# /etc/modprobe.d/blacklist.conf
blacklist pcspkr             # Disable PC speaker
blacklist nouveau             # Blacklist Nouveau GPU driver

# /etc/modprobe.d/options.conf
options usbhid mousepoll=1
```

---

## 11. Web Servers

### `/etc/nginx/nginx.conf` – Nginx Core Config

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    server_tokens off;                 # Hide version
    gzip on;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log warn;

    # Virtual host configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### `/etc/apache2/sites-available/*.conf` – Apache VHost

```apache
<VirtualHost *:443>
    ServerName www.example.com
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile    /etc/ssl/certs/example.pem
    SSLCertificateKeyFile /etc/ssl/private/example.key

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

---

## 12. Database

### `/etc/mysql/mysql.conf.d/mysqld.cnf` – MySQL

```ini
[mysqld]
bind-address = 127.0.0.1    # Listen only on loopback
port = 3306
datadir = /var/lib/mysql
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
max_connections = 200
```

### `/etc/postgresql/14/main/pg_hba.conf` – PostgreSQL Auth

```
# TYPE  DATABASE    USER        ADDRESS             METHOD
local   all         postgres                        peer
local   all         all                             md5
host    all         all         127.0.0.1/32        scram-sha-256
host    all         all         ::1/128             scram-sha-256
hostssl mydb        appuser     10.0.0.0/24         scram-sha-256
```

---

## 13. Security & PAM

### `/etc/pam.d/common-password` – Password Policy

```
# Enforce strong passwords with pam_pwquality
password requisite pam_pwquality.so retry=3 minlen=12 difok=4 \
    ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 reject_username

# Hashing algorithm (sha512 with rounds)
password [success=1 default=ignore] pam_unix.so obscure use_authtok \
    try_first_pass yescrypt rounds=65536
```

### `/etc/security/limits.conf` – Resource Limits

```
# Format: domain type item value
*          soft  nofile      65536
*          hard  nofile      65536
*          soft  nproc       4096
*          hard  nproc       8192
root       soft  nofile      unlimited
@developers hard  core       0        # No core dumps for devs
```

### `/etc/fail2ban/jail.local` – Fail2ban Jails

```ini
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 5
backend  = systemd

[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 86400   # 24 hours
```

---

*Tip: Always back up config files before editing: `cp /etc/ssh/sshd_config{,.bak}`*
