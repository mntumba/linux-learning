# 12 — System Administration

> **Module 3 · Lesson 2** | Difficulty: ★★★☆☆ Intermediate-Advanced | Time: ~90 min

---

## Learning Objectives

- Monitor system resources and performance
- Manage system logs and logrotate
- Configure system startup and boot
- Manage users and groups at scale
- Perform system backups
- Understand time synchronization (NTP/chrony)

---

## 1. System Monitoring

```bash
# CPU info
lscpu
cat /proc/cpuinfo | grep "model name" | uniq
nproc                              # number of CPUs

# CPU usage in real-time
top                                # basic
htop                               # advanced (sudo apt install htop)
mpstat -P ALL 1                    # per-CPU stats (sysstat package)
sar -u 1 5                         # CPU activity, 5 samples

# Memory
free -h                            # memory summary
vmstat -s                          # detailed stats
cat /proc/meminfo

# Disk I/O
iostat -x 1                        # extended I/O stats
iotop                              # per-process I/O (sudo apt install iotop)
sudo iotop -o                      # only processes doing I/O

# System load
uptime                             # 1, 5, 15 minute averages
cat /proc/loadavg                  # raw load averages
tload                              # graphical load average

# All-in-one monitoring
sudo apt install glances
glances                            # comprehensive real-time monitor
glances -w                         # web interface on port 61208
```

### Performance Analysis

```bash
# Find resource hogs
ps aux --sort=-%cpu | head -10     # top CPU consumers
ps aux --sort=-%mem | head -10     # top memory consumers

# Specific performance investigation
lsof                               # list open files (all processes)
lsof -u alice                      # files opened by alice
lsof -i :80                        # processes using port 80
lsof -p 1234                       # files opened by PID 1234
lsof +D /var/log/                  # processes accessing a directory

# Strace — trace system calls
strace ls /tmp                     # trace ls command
strace -p 1234                     # attach to running process
strace -f ./program                # follow child processes
strace -c ls /tmp                  # summary of syscalls

# Network performance
iperf3 -s                          # start server
iperf3 -c server_ip                # run client test
```

---

## 2. Log Management

### Key System Logs

```bash
/var/log/syslog          # general system messages
/var/log/auth.log        # authentication events
/var/log/kern.log        # kernel messages
/var/log/dpkg.log        # package installation history
/var/log/apt/history.log # apt command history
/var/log/boot.log        # boot messages
/var/log/cron.log        # cron job output
/var/log/mail.log        # mail system
/var/log/nginx/          # nginx access/error logs
```

### Log Analysis

```bash
# View logs
tail -f /var/log/syslog            # real-time system log
grep "error\|warning" /var/log/syslog -i | tail -20

# journald (systemd logs)
journalctl                         # all logs
journalctl -b                      # since last boot
journalctl -f                      # follow
journalctl -p err                  # errors only
journalctl -u nginx                # service-specific
journalctl --since "1 hour ago"

# Log statistics
awk '{print $5}' /var/log/syslog | sort | uniq -c | sort -rn | head -20
```

### Logrotate

```bash
# View logrotate config
cat /etc/logrotate.conf
ls /etc/logrotate.d/

# Example custom logrotate config
cat << 'EOF' | sudo tee /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 640 www-data adm
    postrotate
        systemctl reload myapp 2>/dev/null || true
    endscript
}
EOF

# Test logrotate
sudo logrotate -d /etc/logrotate.conf     # dry run
sudo logrotate -f /etc/logrotate.conf     # force rotation
```

---

## 3. Boot Process and systemd

### Linux Boot Process

```
BIOS/UEFI
   │
   ▼
Bootloader (GRUB2)
   │  /boot/grub/grub.cfg
   │
   ▼
Linux Kernel (vmlinuz)
   │  Decompress, initialize hardware
   │  Mount initrd (initial RAM disk)
   │
   ▼
initrd/initramfs
   │  Mount real root filesystem
   │
   ▼
PID 1: systemd
   │  Start targets and services
   │
   ▼
Login Prompt
```

```bash
# View boot messages
dmesg                              # kernel ring buffer
dmesg | grep -i error              # boot errors
journalctl -b                      # all boot log
journalctl -b -1                   # previous boot

# Boot time analysis
systemd-analyze                    # total boot time
systemd-analyze blame              # slowest services
systemd-analyze critical-chain     # critical path

# GRUB configuration
cat /boot/grub/grub.cfg
sudo vim /etc/default/grub         # GRUB settings
sudo update-grub                   # regenerate grub.cfg
```

### systemd Targets (Runlevels)

```bash
# View current target
systemctl get-default
systemctl list-units --type=target --all

# Change target (temporary)
sudo systemctl isolate multi-user.target    # like runlevel 3 (no GUI)
sudo systemctl isolate graphical.target     # like runlevel 5 (with GUI)
sudo systemctl isolate rescue.target        # maintenance mode

# Set default target (persistent)
sudo systemctl set-default multi-user.target
sudo systemctl set-default graphical.target

# Emergency mode (minimal system)
sudo systemctl emergency
```

---

## 4. System Backup

```bash
# rsync backup
rsync -avz --delete \
    /home/alice/ \
    /backup/alice/

# Incremental backup with hard links
rsync -av --link-dest=/backup/previous \
    /home/alice/ \
    /backup/$(date +%Y%m%d)/

# Backup to remote server
rsync -avz --delete /home/ user@backup-server:/backups/home/

# Full system backup (exclude /proc, /sys, /dev)
sudo rsync -aAXv \
    --exclude=/proc \
    --exclude=/sys \
    --exclude=/dev \
    --exclude=/run \
    --exclude=/tmp \
    / \
    /mnt/backup/

# Verify backup
rsync -n -avz source/ destination/  # dry run (-n)
```

---

## 5. Time Synchronization (NTP)

```bash
# Check current time
timedatectl status
date

# Set timezone
sudo timedatectl set-timezone America/New_York
timedatectl list-timezones | grep Europe

# chrony (modern NTP client — Ubuntu default)
sudo apt install chrony
chronyc tracking                   # synchronization status
chronyc sources -v                 # NTP sources
chronyc sourcestats                # statistics

# Force sync
sudo chronyc makestep

# Configure chrony
sudo vim /etc/chrony/chrony.conf
# Add: server time.cloudflare.com iburst

sudo systemctl restart chrony
```

---

## Practice Exercises

### Exercise 12.1 — System Health Check Script

```bash
cat << 'SCRIPT' > ~/health_check.sh
#!/bin/bash
set -euo pipefail

echo "=== System Health Report ==="
echo "Generated: $(date)"
echo ""

echo "--- Uptime ---"
uptime

echo "--- Memory ---"
free -h

echo "--- Disk ---"
df -h | grep -v tmpfs

echo "--- CPU Load ---"
cat /proc/loadavg

echo "--- Top Processes ---"
ps aux --sort=-%cpu | head -6

echo "--- Recent Errors ---"
journalctl -p err --since "1 hour ago" | tail -10

echo "=== End Report ==="
SCRIPT
chmod +x ~/health_check.sh
~/health_check.sh
```

### Exercise 12.2 — Log Analysis

```bash
# 1. Count messages per hour in syslog
awk '{print $1, $2, $3}' /var/log/syslog | \
    cut -c1-13 | sort | uniq -c | sort -rn | head -24

# 2. Find the most common message sources
awk '{print $5}' /var/log/syslog | \
    sed 's/\[.*//' | sort | uniq -c | sort -rn | head -20

# 3. Check boot time
systemd-analyze blame | head -10
```

---

## Key Takeaways

- Use **htop**, **iotop**, **glances** for comprehensive system monitoring
- **journalctl** is the primary log viewer for systemd systems
- **logrotate** manages log files to prevent disk space issues
- **systemd targets** replace traditional runlevels
- **rsync** with `--link-dest` enables space-efficient incremental backups
- **chrony** keeps system time synchronized via NTP

---

➡️ [13 — Storage and Disk Management](13_Storage_and_Disk_Management.md)

*Module 3 · Lesson 2 of 5 | [Course Index](../INDEX.md)*
