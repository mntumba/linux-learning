# 12. System Administration

> **Difficulty:** Advanced | **Time Estimate:** 3–4 hours | **Prerequisites:** Lessons 1–11

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Manage Linux services with `systemd` and `systemctl`
- Understand and navigate the full Linux boot process
- Read and filter system logs with `journalctl` and traditional log files
- Monitor system performance with `vmstat`, `iostat`, `sar`, and `free`
- Schedule jobs with cron, `at`, and systemd timers
- Configure environment variables at the system and user level
- Implement robust backup strategies using `rsync` and `tar`

---

## 1. The Linux Boot Process

```
BIOS/UEFI
    │  Power-on self test, finds boot device
    ▼
GRUB2 (bootloader)
    │  Loads kernel image + initramfs from /boot
    ▼
Linux Kernel
    │  Hardware init, mounts root filesystem
    ▼
initramfs (initial RAM filesystem)
    │  Temporary root, loads drivers, mounts real root
    ▼
systemd (PID 1)
    │  Reads unit files, reaches target
    ▼
default.target (usually graphical.target or multi-user.target)
    │
    ▼
Login prompt / Display Manager
```

```bash
# View kernel boot messages
dmesg | head -40
dmesg --level=err,warn      # Only errors and warnings

# Check GRUB configuration
cat /boot/grub/grub.cfg | grep menuentry
cat /etc/default/grub       # GRUB settings

# Update GRUB after changes
sudo update-grub             # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS
```

---

## 2. systemd and systemctl

### Service Management

```bash
# Start, stop, restart a service
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx       # Reload config without full restart

# Enable/disable at boot
sudo systemctl enable nginx
sudo systemctl disable nginx
sudo systemctl enable --now nginx # Enable AND start immediately

# Check status
systemctl status nginx
systemctl is-active nginx         # active / inactive
systemctl is-enabled nginx        # enabled / disabled / static

# After modifying unit files
sudo systemctl daemon-reload
```

### Listing and Filtering Units

```bash
systemctl list-units                          # All active units
systemctl list-units --type=service           # Services only
systemctl list-units --state=failed           # Failed units
systemctl list-unit-files --type=service      # All service unit files
systemctl list-dependencies nginx             # Dependency tree
```

### Systemd Targets (Runlevels)

```
SysV Runlevel │ systemd Target
─────────────────────────────────
0             │ poweroff.target
1             │ rescue.target
2,3,4         │ multi-user.target
5             │ graphical.target
6             │ reboot.target
```

```bash
# Get current target
systemctl get-default

# Change default target
sudo systemctl set-default multi-user.target

# Switch immediately (without rebooting)
sudo systemctl isolate rescue.target
```

### Writing a Custom Service Unit

```bash
# /etc/systemd/system/myapp.service
cat << 'EOF' | sudo tee /etc/systemd/system/myapp.service
[Unit]
Description=My Application Service
After=network.target
Requires=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
ExecStop=/opt/myapp/stop.sh
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

### Systemd Timers (Replacement for Cron)

```bash
# /etc/systemd/system/backup.timer
cat << 'EOF' | sudo tee /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
Persistent=true         # Run if missed (e.g., system was off)

[Install]
WantedBy=timers.target
EOF

# /etc/systemd/system/backup.service
cat << 'EOF' | sudo tee /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup Job

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemctl list-timers                   # View all active timers
```

---

## 3. System Logs

### journalctl

```bash
journalctl                             # All logs (oldest first)
journalctl -r                          # Reverse (newest first)
journalctl -f                          # Follow live
journalctl -n 50                       # Last 50 lines
journalctl -u nginx                    # Logs for nginx unit
journalctl -u nginx --since "1 hour ago"
journalctl --since "2024-01-01" --until "2024-01-02"
journalctl -p err                      # Priority: emerg alert crit err warning notice info debug
journalctl -k                          # Kernel messages only
journalctl --disk-usage                # Storage used by journal
sudo journalctl --vacuum-time=7d       # Remove entries older than 7 days
journalctl -b                          # Current boot logs
journalctl -b -1                       # Previous boot logs
journalctl -o json-pretty -u sshd | head  # JSON output
```

### Traditional Log Files

```bash
# Key log locations
/var/log/syslog         # General system messages (Debian/Ubuntu)
/var/log/messages       # General system messages (RHEL/CentOS)
/var/log/auth.log       # Authentication events (Debian/Ubuntu)
/var/log/secure         # Authentication events (RHEL/CentOS)
/var/log/kern.log       # Kernel messages
/var/log/dmesg          # Boot kernel messages
/var/log/dpkg.log       # Package installations (Debian)
/var/log/yum.log        # Package installations (RHEL)
/var/log/nginx/         # Nginx logs
/var/log/apache2/       # Apache logs

# Watch logs in real time
tail -f /var/log/syslog
tail -f /var/log/auth.log | grep -i "failed\|invalid"
```

### Log Rotation with logrotate

```bash
# View global config
cat /etc/logrotate.conf

# Per-application configs
ls /etc/logrotate.d/

# Example custom config
cat << 'EOF' | sudo tee /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser adm
    postrotate
        systemctl reload myapp
    endscript
}
EOF

# Test logrotate config (dry run)
sudo logrotate --debug /etc/logrotate.d/myapp

# Force rotation now
sudo logrotate --force /etc/logrotate.d/myapp
```

---

## 4. System Monitoring

### Memory and CPU

```bash
# Memory overview
free -h
free -h -s 2            # Refresh every 2 seconds

# Virtual memory stats
vmstat 2 5              # 5 readings, 2-second interval
# Output: procs/memory/swap/io/system/cpu columns

# CPU and I/O stats (from sysstat package)
iostat -xz 2 3          # Extended disk stats, every 2s, 3 times
mpstat -P ALL 1         # Per-CPU usage, 1-second interval

# Historical stats (sar)
sar -u 1 5              # CPU utilization
sar -r 1 5              # Memory usage
sar -d 1 5              # Disk activity
sar -n DEV 1 5          # Network interface stats
sar -b 1 5              # I/O and transfer rates

# Continuous monitoring
watch -n 2 'free -h && echo && df -h'
```

### Disk Usage

```bash
df -h                   # Disk free (human readable)
df -hT                  # Include filesystem type
df -i                   # Inode usage (important to check!)

du -sh /var/log/*       # Size of each item in /var/log
du -sh /* 2>/dev/null   # Top-level directory sizes
du -ah /etc | sort -rh | head -20  # Largest files in /etc

# Find large files
find / -xdev -type f -size +100M 2>/dev/null | sort -k5 -n
```

---

## 5. Scheduled Tasks

### Cron

```bash
# Edit user's crontab
crontab -e
crontab -l              # List current crontab
crontab -r              # Remove crontab (DANGEROUS)
crontab -l -u alice     # View another user's crontab (root)

# Cron syntax
# ┌───────── minute (0-59)
# │ ┌─────── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌─── month (1-12)
# │ │ │ │ ┌─ day of week (0-7, Sun=0 or 7)
# │ │ │ │ │
# * * * * *  command

# Examples
0 2 * * *     /usr/local/bin/backup.sh       # Daily at 2:00 AM
*/15 * * * *  /usr/local/bin/check-disk.sh   # Every 15 minutes
0 9 * * 1-5   /usr/local/bin/report.sh       # Mon-Fri at 9 AM
@reboot       /usr/local/bin/startup.sh      # On every reboot
@weekly       /usr/local/bin/weekly-clean.sh # Weekly

# System-wide crontab (has username field)
cat /etc/crontab
ls /etc/cron.{hourly,daily,weekly,monthly}/
```

### at Command (One-time Tasks)

```bash
# Schedule a one-time command
at now + 5 minutes <<< "echo 'Hello' > /tmp/at-test.txt"
at 14:30 <<< "systemctl restart nginx"
at midnight tomorrow <<< "/usr/local/bin/monthly-report.sh"

atq            # List queued at jobs
atrm 3         # Remove job number 3
```

---

## 6. Backup Strategies

### rsync

```bash
# Local sync
rsync -av /source/dir/ /backup/dir/          # Archive + verbose
rsync -avz /source/ user@remote:/backup/     # Compress over SSH
rsync -av --delete /source/ /backup/         # Mirror (delete removed files)
rsync -av --exclude='*.log' --exclude='.cache/' /home/ /backup/home/

# Dry run first (always!)
rsync -avhn /source/ /backup/    # -n = dry run

# Incremental backup using hard links (time-machine style)
DATE=$(date +%Y-%m-%d)
rsync -av --link-dest=/backup/latest /source/ /backup/$DATE/
ln -snf /backup/$DATE /backup/latest
```

### tar Backups

```bash
# Create compressed archive
tar -czf backup_$(date +%Y%m%d).tar.gz /home/user/

# With exclusions
tar -czf backup.tar.gz \
    --exclude='/home/user/.cache' \
    --exclude='*.tmp' \
    /home/user/

# List archive contents
tar -tzf backup.tar.gz | head -20

# Extract archive
tar -xzf backup.tar.gz -C /restore/path/

# Incremental backup
tar -czf full_backup.tar.gz -g /var/tmp/snapshot.snar /home/
tar -czf inc_backup_1.tar.gz -g /var/tmp/snapshot.snar /home/
```

---

## 7. Environment Variables

```bash
# View all environment variables
env
printenv
printenv HOME

# Temporary (current session only)
export MY_VAR="hello"
unset MY_VAR

# Persistent — scope matters:
#
# /etc/environment       → system-wide, all users, simple KEY=VALUE
# /etc/profile           → system-wide, login shells
# /etc/profile.d/*.sh    → system-wide drop-in scripts
# ~/.bash_profile        → user login shells (runs once at login)
# ~/.bashrc              → user interactive non-login shells
# ~/.bash_logout         → runs on logout

# Add to user environment permanently
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
echo 'export EDITOR=vim' >> ~/.bashrc
source ~/.bashrc    # Apply without reopening terminal

# System-wide variable
echo 'MY_GLOBAL_VAR="production"' | sudo tee -a /etc/environment
```

---

## 8. Network Time Protocol

```bash
# timedatectl (systemd)
timedatectl status                   # Show current time settings
timedatectl set-timezone America/New_York
timedatectl set-ntp true             # Enable NTP sync
timedatectl list-timezones | grep America

# Check NTP sync status
timedatectl show-timesync

# Traditional ntpd
ntpq -p              # Show NTP peer status
ntpstat              # Sync status

# chrony (modern NTP)
chronyc tracking     # Current tracking info
chronyc sources -v   # NTP sources
chronyc makestep     # Force immediate sync
```

---

## ASCII Diagram: systemd Unit Hierarchy

```
                    default.target
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
      network.target  sshd.service  nginx.service
              │
    ┌─────────┴──────────┐
    ▼                    ▼
NetworkManager      systemd-resolved
    .service             .service
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| Editing unit file without `daemon-reload` | Changes not picked up | Always run `systemctl daemon-reload` |
| `crontab -r` instead of `-e` | Deletes entire crontab | Double-check the flag; keep a backup |
| Cron jobs not running | Environment differs from user shell | Use absolute paths; set PATH in crontab |
| Forgetting `@reboot` delay | Service starts before dependencies | Use `After=` in unit `[Unit]` section |
| Using `rm -rf` during cleanup | Irreversible data loss | Test with `--dry-run` first |
| Not setting `Restart=on-failure` | Service stays down after crash | Add `Restart=on-failure` to `[Service]` |

---

## Pro Tips

- **`systemd-analyze blame`** — ranks services by startup time to identify boot bottlenecks
- **`systemd-analyze critical-chain`** — shows the critical path in the boot dependency graph
- **`journalctl --no-pager -u nginx | grep -i error`** — combine journal with grep for fast filtering
- **Persistent journal**: edit `/etc/systemd/journald.conf` and set `Storage=persistent` so logs survive reboots
- **`watch -d`** highlights cells that changed between refreshes — great with `vmstat` or `df`
- **Cron email alerts**: set `MAILTO=admin@example.com` in crontab to receive output via email

---

## Practice Exercises

1. **Service Control** — Install `nginx`, write a systemd unit override that limits it to 256MB RAM (`MemoryLimit=256M`), enable it, and verify status.

2. **Custom Timer** — Create a systemd timer that runs a script every 5 minutes and logs the current memory usage to `/var/log/mem-monitor.log`.

3. **Log Analysis** — Use `journalctl` to find all SSH login failures in the last 24 hours and count how many occurred per IP address.

4. **Boot Optimization** — Run `systemd-analyze blame`, identify the top 3 slowest services, and research what each one does.

5. **Cron Backup** — Write a cron job that backs up `/etc` to `/backup/etc-YYYY-MM-DD.tar.gz` daily at 3 AM and keeps only the last 10 backups.

6. **Environment Setup** — Write a `/etc/profile.d/custom-env.sh` that sets `HISTTIMEFORMAT`, adds `/opt/tools` to `PATH`, and sets `EDITOR=vim` for all users.

7. **Performance Baseline** — Use `vmstat`, `iostat`, and `sar` to capture 60 seconds of system metrics and save the output to a report file.

8. **Log Rotation Config** — Write a `logrotate` config for an imaginary app at `/var/log/webapp/` that: rotates daily, keeps 30 days, compresses, and sends SIGHUP to the app after rotation.

---

## Key Takeaways

- **systemd** is PID 1; it manages the entire service lifecycle via declarative unit files
- Use `systemctl enable --now` to simultaneously enable and start a service
- **journalctl** is the primary tool for structured log inspection on systemd systems
- **Cron** uses a 5-field time spec; `@reboot` and `@daily` shortcuts simplify common patterns
- **rsync --link-dest** enables space-efficient incremental backups using hard links
- Monitor `df -i` (inodes) as well as `df -h` (blocks) — inode exhaustion stops writes even when disk space is available
- Environment variable scope: `/etc/environment` → all users; `~/.bashrc` → interactive shells; `~/.bash_profile` → login shells

---

## Next Lesson Preview

**Lesson 13: Storage and Disk Management** — We'll go deep on partitioning (MBR vs GPT), filesystems (ext4, xfs, btrfs), LVM for flexible disk management, RAID configurations, and network filesystems. Understanding storage is critical for any Linux administrator managing servers.
