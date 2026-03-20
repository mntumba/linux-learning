# Appendix A – Linux Command Reference

A comprehensive cheat-sheet of essential Linux commands organized by category.
Each entry includes a short description and a practical example.

---

## Table of Contents
1. [File & Directory Operations](#1-file--directory-operations)
2. [File Viewing & Searching](#2-file-viewing--searching)
3. [File Permissions & Ownership](#3-file-permissions--ownership)
4. [Archiving & Compression](#4-archiving--compression)
5. [User & Group Management](#5-user--group-management)
6. [Process Management](#6-process-management)
7. [System Information](#7-system-information)
8. [Disk & Filesystem](#8-disk--filesystem)
9. [Networking](#9-networking)
10. [Package Management](#10-package-management)
11. [Text Processing](#11-text-processing)
12. [Job Scheduling](#12-job-scheduling)
13. [Shell Built-ins & Redirection](#13-shell-built-ins--redirection)
14. [SSH & Remote Access](#14-ssh--remote-access)

---

## 1. File & Directory Operations

| Command | Description | Example |
|---------|-------------|---------|
| `ls` | List directory contents | `ls -lah /var/log` |
| `cd` | Change directory | `cd /etc/nginx` |
| `pwd` | Print working directory | `pwd` |
| `mkdir` | Create directory | `mkdir -p /opt/app/data` |
| `rmdir` | Remove empty directory | `rmdir /tmp/empty` |
| `cp` | Copy files/directories | `cp -r /src /dst` |
| `mv` | Move or rename | `mv old.txt new.txt` |
| `rm` | Remove files/directories | `rm -rf /tmp/junk` |
| `touch` | Create empty file / update timestamp | `touch notes.txt` |
| `ln` | Create hard or symbolic links | `ln -s /usr/bin/python3 /usr/local/bin/python` |
| `find` | Search for files | `find /home -name "*.log" -mtime -7` |
| `locate` | Fast file lookup via database | `locate sshd_config` |
| `tree` | Display directory tree | `tree -L 2 /etc` |
| `stat` | Detailed file metadata | `stat /etc/passwd` |
| `file` | Determine file type | `file /bin/bash` |
| `readlink` | Resolve symlink target | `readlink -f /usr/bin/python` |

```bash
# Recursively copy and preserve attributes
cp -a /var/www/html /var/backups/html-$(date +%F)

# Find files modified in the last 24 hours
find /etc -type f -mtime -1

# Find files larger than 100 MB
find / -type f -size +100M 2>/dev/null
```

---

## 2. File Viewing & Searching

| Command | Description | Example |
|---------|-------------|---------|
| `cat` | Print entire file | `cat /etc/hosts` |
| `less` | Paginated view | `less /var/log/syslog` |
| `more` | Forward-only pager | `more /etc/services` |
| `head` | First N lines (default 10) | `head -20 access.log` |
| `tail` | Last N lines; follow live | `tail -f /var/log/auth.log` |
| `grep` | Search patterns in text | `grep -rn "error" /var/log/` |
| `strings` | Print printable strings in binary | `strings /usr/bin/ls` |
| `wc` | Word/line/char count | `wc -l /etc/passwd` |
| `diff` | Compare two files | `diff file1.conf file2.conf` |
| `cmp` | Byte-level file comparison | `cmp binary1 binary2` |
| `xxd` | Hex dump | `xxd /bin/ls | head -5` |

```bash
# Follow multiple log files simultaneously
tail -f /var/log/syslog /var/log/auth.log

# Search case-insensitively and print surrounding context
grep -i -C 3 "failed password" /var/log/auth.log

# Count unique IP addresses in access log
grep -oP '^\d+\.\d+\.\d+\.\d+' access.log | sort -u | wc -l
```

---

## 3. File Permissions & Ownership

| Command | Description | Example |
|---------|-------------|---------|
| `chmod` | Change file permissions | `chmod 750 script.sh` |
| `chown` | Change file owner | `chown www-data:www-data /var/www` |
| `chgrp` | Change group ownership | `chgrp developers project/` |
| `umask` | Set default permission mask | `umask 027` |
| `getfacl` | Get POSIX ACLs | `getfacl /srv/share` |
| `setfacl` | Set POSIX ACLs | `setfacl -m u:alice:rwx /srv/share` |
| `lsattr` | List extended attributes | `lsattr /etc/passwd` |
| `chattr` | Change extended attributes | `chattr +i /etc/resolv.conf` |

```bash
# Symbolic mode: add execute for owner, remove write for group/other
chmod u+x,go-w deploy.sh

# Octal quick reference:
# 7 = rwx  |  6 = rw-  |  5 = r-x  |  4 = r--  |  0 = ---
chmod 644 config.ini   # owner rw, group/other r

# Recursively set directory permissions
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;
```

---

## 4. Archiving & Compression

| Command | Description | Example |
|---------|-------------|---------|
| `tar` | Tape archive (bundle + compress) | `tar czf backup.tar.gz /etc` |
| `gzip` | GNU compression | `gzip -9 logfile.txt` |
| `gunzip` | Decompress gzip | `gunzip logfile.txt.gz` |
| `bzip2` | Higher-ratio compression | `bzip2 data.bin` |
| `xz` | Best-ratio compression | `xz -T0 large.img` |
| `zip` | ZIP archive | `zip -r archive.zip folder/` |
| `unzip` | Extract ZIP | `unzip archive.zip -d /tmp/out` |
| `7z` | 7-Zip archive | `7z a archive.7z folder/` |

```bash
# Create compressed tar, verbose output
tar czvf /backup/etc-$(date +%F).tar.gz /etc

# Extract to specific directory
tar xzf archive.tar.gz -C /opt/app

# List contents without extracting
tar tzf archive.tar.gz | head -20
```

---

## 5. User & Group Management

| Command | Description | Example |
|---------|-------------|---------|
| `useradd` | Create new user | `useradd -m -s /bin/bash alice` |
| `usermod` | Modify user account | `usermod -aG sudo alice` |
| `userdel` | Delete user | `userdel -r bob` |
| `passwd` | Set/change password | `passwd alice` |
| `groupadd` | Create group | `groupadd devops` |
| `groupdel` | Delete group | `groupdel devops` |
| `groups` | Show groups for user | `groups alice` |
| `id` | Show UID/GID info | `id alice` |
| `whoami` | Current user | `whoami` |
| `who` | Logged-in users | `who -a` |
| `w` | Active sessions with load | `w` |
| `last` | Login history | `last -n 20` |
| `su` | Switch user | `su - alice` |
| `sudo` | Execute as another user | `sudo -u postgres psql` |
| `visudo` | Safely edit sudoers | `visudo` |

```bash
# Create system service account (no login shell, no home)
useradd -r -s /usr/sbin/nologin -d /var/lib/myapp myapp

# Lock / unlock account
passwd -l alice
passwd -u alice

# Add user to multiple groups at once
usermod -aG docker,sudo,developers alice
```

---

## 6. Process Management

| Command | Description | Example |
|---------|-------------|---------|
| `ps` | Process snapshot | `ps aux --sort=-%cpu` |
| `top` | Dynamic process viewer | `top -u www-data` |
| `htop` | Interactive process viewer | `htop` |
| `kill` | Send signal to PID | `kill -9 1234` |
| `killall` | Kill by name | `killall nginx` |
| `pkill` | Kill by pattern | `pkill -f "python worker"` |
| `pgrep` | Find PID by name | `pgrep -la sshd` |
| `nice` | Launch with priority | `nice -n 10 ./cpu-task.sh` |
| `renice` | Change priority | `renice +5 -p 1234` |
| `nohup` | Immune to hangup | `nohup ./long-job.sh &` |
| `bg` / `fg` | Background / foreground job | `bg %1` |
| `jobs` | List shell jobs | `jobs -l` |
| `strace` | Trace system calls | `strace -p 1234` |
| `lsof` | List open files/sockets | `lsof -i :80` |
| `fuser` | Identify file/socket users | `fuser -k 8080/tcp` |

```bash
# Show full command line for all processes
ps auxww | grep "[n]ginx"

# Find and kill a process by name
kill $(pgrep httpd)

# Run command in background, immune to logout
nohup python3 server.py > /var/log/server.log 2>&1 &
```

---

## 7. System Information

| Command | Description | Example |
|---------|-------------|---------|
| `uname` | Kernel information | `uname -a` |
| `hostname` | Show/set hostname | `hostname -I` |
| `uptime` | System uptime and load | `uptime` |
| `date` | System date/time | `date '+%Y-%m-%d %H:%M:%S'` |
| `timedatectl` | Manage time/timezone | `timedatectl set-timezone UTC` |
| `dmesg` | Kernel ring buffer | `dmesg -T | tail -30` |
| `journalctl` | systemd journal | `journalctl -u sshd -n 50 -f` |
| `systemctl` | Manage systemd services | `systemctl status nginx` |
| `lscpu` | CPU architecture info | `lscpu` |
| `free` | Memory usage | `free -h` |
| `vmstat` | Virtual memory stats | `vmstat 1 5` |
| `lspci` | PCI devices | `lspci -v` |
| `lsusb` | USB devices | `lsusb` |
| `dmidecode` | Hardware info from BIOS | `dmidecode -t memory` |
| `env` | Show environment variables | `env | sort` |

```bash
# One-liner system summary
echo "Host: $(hostname) | Kernel: $(uname -r) | Uptime: $(uptime -p)"

# Check enabled services at boot
systemctl list-unit-files --state=enabled

# Follow journal for specific unit
journalctl -fu nginx --since "10 minutes ago"
```

---

## 8. Disk & Filesystem

| Command | Description | Example |
|---------|-------------|---------|
| `df` | Disk space usage | `df -hT` |
| `du` | Directory disk usage | `du -sh /var/log/*` |
| `lsblk` | List block devices | `lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT` |
| `fdisk` | Partition table editor | `fdisk -l /dev/sda` |
| `parted` | Modern partition tool | `parted /dev/sdb print` |
| `mkfs` | Format filesystem | `mkfs.ext4 /dev/sdb1` |
| `mount` | Mount filesystem | `mount /dev/sdb1 /mnt/data` |
| `umount` | Unmount filesystem | `umount /mnt/data` |
| `blkid` | Block device attributes | `blkid /dev/sda1` |
| `fsck` | Filesystem check/repair | `fsck -n /dev/sdb1` |
| `tune2fs` | Adjust ext2/3/4 params | `tune2fs -l /dev/sda1` |
| `ncdu` | ncurses disk usage | `ncdu /home` |
| `iostat` | I/O statistics | `iostat -xz 1 3` |

```bash
# Top 10 largest directories under /var
du -sh /var/* 2>/dev/null | sort -rh | head -10

# Check inode usage
df -i

# Show filesystem type and UUID for all mounts
lsblk -o NAME,FSTYPE,UUID,MOUNTPOINT | column -t
```

---

## 9. Networking

| Command | Description | Example |
|---------|-------------|---------|
| `ip addr` | Show IP addresses | `ip addr show eth0` |
| `ip route` | Show/manage routes | `ip route show` |
| `ip link` | Manage network interfaces | `ip link set eth0 up` |
| `ss` | Socket statistics | `ss -tulnp` |
| `netstat` | Network connections (legacy) | `netstat -anp` |
| `ping` | ICMP connectivity test | `ping -c 4 8.8.8.8` |
| `traceroute` | Trace packet path | `traceroute google.com` |
| `nslookup` | DNS query | `nslookup github.com` |
| `dig` | Detailed DNS query | `dig +short MX gmail.com` |
| `host` | Simple DNS lookup | `host -t A example.com` |
| `curl` | HTTP client | `curl -I https://example.com` |
| `wget` | File downloader | `wget -q -O - https://example.com` |
| `nmap` | Network scanner | `nmap -sV -p 22,80,443 target` |
| `tcpdump` | Packet capture | `tcpdump -i eth0 port 80 -w cap.pcap` |
| `iptables` | Firewall rules (legacy) | `iptables -L -n -v` |
| `nft` | Modern netfilter tables | `nft list ruleset` |
| `ufw` | Uncomplicated Firewall | `ufw allow 22/tcp` |

```bash
# Show listening services with PID
ss -tulnp | column -t

# Capture first 100 packets on any interface
tcpdump -c 100 -nn -i any -w /tmp/capture.pcap

# Test HTTP response code
curl -o /dev/null -sw "%{http_code}\n" https://example.com
```

---

## 10. Package Management

### Debian / Ubuntu (APT)

```bash
apt update                          # Refresh package index
apt upgrade -y                      # Upgrade all packages
apt install -y nginx                # Install package
apt remove nginx                    # Remove (keep config)
apt purge nginx                     # Remove including config
apt autoremove                      # Remove orphaned deps
apt search keyword                  # Search packages
apt show nginx                      # Show package details
dpkg -l | grep nginx                # List installed matching
dpkg -L nginx                       # Files installed by package
```

### Red Hat / CentOS / Fedora (DNF/YUM)

```bash
dnf update -y                       # Upgrade all packages
dnf install -y httpd                # Install package
dnf remove httpd                    # Remove package
dnf search keyword                  # Search packages
dnf info httpd                      # Package details
rpm -qa | grep httpd                # List installed matching
rpm -ql httpd                       # Files installed by package
```

### Universal Package Managers

```bash
snap install code --classic         # Snap package
flatpak install flathub org.gimp.GIMP  # Flatpak
pip install requests                # Python packages
npm install -g typescript           # Node packages
```

---

## 11. Text Processing

| Command | Description | Example |
|---------|-------------|---------|
| `grep` | Pattern search | `grep -E "err|warn" syslog` |
| `sed` | Stream editor | `sed 's/foo/bar/g' file.txt` |
| `awk` | Text processing language | `awk '{print $1,$NF}' access.log` |
| `cut` | Extract columns | `cut -d: -f1,3 /etc/passwd` |
| `sort` | Sort lines | `sort -t, -k2 -n data.csv` |
| `uniq` | Remove duplicates | `sort file | uniq -c | sort -rn` |
| `tr` | Translate characters | `echo "HELLO" | tr A-Z a-z` |
| `column` | Columnate output | `column -t -s, data.csv` |
| `tee` | Write to file and stdout | `ls -la | tee listing.txt` |
| `xargs` | Build commands from stdin | `find . -name "*.log" | xargs rm` |
| `paste` | Merge lines | `paste file1.txt file2.txt` |
| `join` | Join lines on common field | `join -t, -1 1 -2 1 a.csv b.csv` |

---

## 12. Job Scheduling

```bash
# Edit current user's crontab
crontab -e

# List crontab
crontab -l

# Cron syntax: min hour dom month dow command
# Run backup daily at 2:30 AM
30 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# Run on reboot
@reboot /opt/app/start.sh

# One-time scheduled task
at 23:00 < /dev/stdin <<'EOF'
/usr/local/bin/maintenance.sh
EOF

# systemd timer (modern alternative)
systemctl list-timers --all
```

---

## 13. Shell Built-ins & Redirection

```bash
# Standard redirection
command > file.txt        # Redirect stdout (overwrite)
command >> file.txt       # Redirect stdout (append)
command 2> err.txt        # Redirect stderr
command &> all.txt        # Redirect stdout + stderr
command < input.txt       # Redirect stdin
command1 | command2       # Pipe stdout to stdin

# Here-documents
cat <<'EOF' > /etc/motd
Welcome to the system!
EOF

# Process substitution
diff <(ls dir1) <(ls dir2)

# Command grouping
{ echo "line1"; echo "line2"; } >> output.txt
(cd /tmp && ls)           # Subshell

# Variables and arithmetic
VAR="hello"
echo ${VAR^^}             # Uppercase: HELLO
echo $((2 ** 10))         # 1024
```

---

## 14. SSH & Remote Access

| Command | Description | Example |
|---------|-------------|---------|
| `ssh` | Secure shell login | `ssh -p 2222 alice@host` |
| `scp` | Secure copy | `scp file.txt alice@host:/tmp/` |
| `rsync` | Efficient sync/copy | `rsync -avz /src/ user@host:/dst/` |
| `ssh-keygen` | Generate key pair | `ssh-keygen -t ed25519 -C "work"` |
| `ssh-copy-id` | Deploy public key | `ssh-copy-id user@host` |
| `ssh-agent` | Key management daemon | `eval $(ssh-agent)` |
| `sftp` | Interactive secure FTP | `sftp user@host` |

```bash
# SSH tunnel: forward local port 8080 to remote :80
ssh -L 8080:localhost:80 user@remotehost

# Reverse tunnel: expose local :3000 on remote :9000
ssh -R 9000:localhost:3000 user@remotehost

# Run command without interactive login
ssh user@host "sudo systemctl restart nginx"

# Copy entire directory structure with compression
rsync -avz --progress --delete /local/dir/ user@host:/remote/dir/

# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -b 256 -C "$(whoami)@$(hostname)-$(date +%F)"
```

---

*Last updated: see repository commit history*
