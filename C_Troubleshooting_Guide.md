# Appendix C: Linux Troubleshooting Guide — 27 Common Problems and Solutions

This appendix is a practical reference for diagnosing and resolving the most common issues encountered on Linux systems, from desktop environments to production servers. Each entry follows a consistent format: symptoms, root causes, step-by-step solutions, and prevention tips.

---

## Quick Reference Index

| # | Problem | Key Commands |
|---|---------|-------------|
| 1 | Permission denied | `chmod`, `chown`, `getfacl` |
| 2 | command not found | `which`, `type`, `apt install` |
| 3 | System won't boot | GRUB rescue, `boot-repair` |
| 4 | Forgot root password | Recovery mode, `passwd` |
| 5 | Disk full | `df -h`, `du`, `apt clean` |
| 6 | Network interface down | `ip link`, `nmcli`, `lshw` |
| 7 | Cannot SSH to server | `ssh -v`, `ufw`, `sshd` |
| 8 | APT lock / broken deps | `dpkg --configure -a` |
| 9 | Slow system performance | `top`, `iotop`, `vmstat` |
| 10 | High CPU usage | `ps aux`, `kill`, `renice` |
| 11 | Out of memory | OOM logs, `free -h`, swap |
| 12 | Service won't start | `journalctl -xe`, `systemctl` |
| 13 | Cron jobs not running | `crontab -l`, cron.log |
| 14 | Filesystem errors (fsck) | `fsck`, `tune2fs` |
| 15 | Failing hard drive | `smartctl`, `badblocks` |
| 16 | DNS resolution failure | `dig`, `resolvectl` |
| 17 | Firewall blocking connections | `ufw`, `iptables -L` |
| 18 | SSL certificate errors | `openssl s_client`, certbot |
| 19 | Running out of inodes | `df -i`, small-file cleanup |
| 20 | Zombie processes | `ps aux` + grep Z |
| 21 | Screen resolution wrong | `xrandr`, xorg.conf |
| 22 | No audio | `alsamixer`, `pulseaudio` |
| 23 | Time/timezone wrong | `timedatectl`, `ntpd` |
| 24 | Locale/encoding errors | `locale`, `dpkg-reconfigure` |
| 25 | Python/pip conflicts | `venv`, `update-alternatives` |
| 26 | Git merge conflicts | `git mergetool`, `git abort` |
| 27 | Docker container won't start | `docker logs`, `docker inspect` |

---

### Problem 1: "Permission denied"

**Symptoms / Error Message:**
```
bash: /path/to/script.sh: Permission denied
cp: cannot open '/etc/shadow' for reading: Permission denied
```

**Possible Causes:**
- File or directory lacks execute/read/write bits for the current user
- File is owned by a different user or group
- SELinux or AppArmor policy is blocking access
- Access Control Lists (ACLs) override standard permissions

**Solution:**
```bash
# 1. Check current permissions
ls -la /path/to/file

# 2. Check who owns the file
stat /path/to/file

# 3. Add execute permission for the owner
chmod u+x /path/to/script.sh

# 4. Make a script executable for everyone
chmod 755 /path/to/script.sh

# 5. Change ownership to your user
sudo chown $USER:$USER /path/to/file

# 6. Recursively change ownership of a directory
sudo chown -R $USER:$USER /path/to/directory

# 7. Check and view ACLs
getfacl /path/to/file

# 8. Grant ACL access to a specific user
sudo setfacl -m u:username:rwx /path/to/file

# 9. If SELinux is enforcing, check context
ls -Z /path/to/file
sudo restorecon -v /path/to/file

# 10. Run a privileged command with sudo
sudo /path/to/restricted-command
```

**Prevention:**
- Follow the principle of least privilege — never set permissions to `777` on shared systems.
- Use groups to manage shared directory access rather than broadening permissions globally.
- Audit sensitive directories periodically with `find / -perm -o+w -type f 2>/dev/null`.

---

### Problem 2: "command not found"

**Symptoms / Error Message:**
```
bash: nmap: command not found
bash: python: command not found
```

**Possible Causes:**
- Package is not installed
- Binary is installed but not on `$PATH`
- Command name changed (e.g., `python` vs `python3`)
- Shell profile was not reloaded after adding to PATH

**Solution:**
```bash
# 1. Check if the command exists anywhere on the system
which nmap
type nmap
command -v nmap

# 2. Search for a binary by name
find /usr /bin /sbin /opt -name "nmap" 2>/dev/null

# 3. Install the missing package (Debian/Ubuntu)
sudo apt update && sudo apt install nmap

# 4. Install the missing package (RHEL/CentOS/Fedora)
sudo dnf install nmap

# 5. Check your current PATH
echo $PATH

# 6. Add a directory to PATH in the current session
export PATH="$PATH:/usr/local/bin"

# 7. Make the PATH change permanent (bash)
echo 'export PATH="$PATH:/usr/local/bin"' >> ~/.bashrc
source ~/.bashrc

# 8. Find which package provides a command (Debian/Ubuntu)
sudo apt install command-not-found
sudo update-command-not-found
apt-file search bin/nmap

# 9. For python/python3 aliasing
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1
```

**Prevention:**
- Always use full paths in scripts (`/usr/bin/python3` instead of `python`).
- After installing new software to non-standard locations, add to `.bashrc` or `.profile`.
- Use `hash -r` to clear the shell's command cache after moving binaries.

---

### Problem 3: System Won't Boot

**Symptoms / Error Message:**
```
error: no such partition.
grub rescue> _

Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

**Possible Causes:**
- GRUB configuration is corrupted or points to wrong partition
- Boot partition UUID changed after disk operations
- Kernel update failed or initramfs is missing
- Root partition is corrupted

**Solution:**
```bash
# --- At the GRUB rescue prompt ---

# 1. List available partitions
grub rescue> ls

# 2. Identify the boot partition (look for /boot/grub)
grub rescue> ls (hd0,msdos1)/

# 3. Set prefix and root, then load normal module
grub rescue> set root=(hd0,msdos1)
grub rescue> set prefix=(hd0,msdos1)/boot/grub
grub rescue> insmod normal
grub rescue> normal

# --- Once booted (or from a Live USB) ---

# 4. Install boot-repair (Ubuntu/Debian)
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt update && sudo apt install boot-repair
sudo boot-repair

# 5. Manually reinstall GRUB from a live session
sudo mount /dev/sda1 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda
update-grub
exit

# 6. Update GRUB after kernel changes
sudo update-grub
```

**Prevention:**
- Take a snapshot/backup before major kernel updates.
- Keep a bootable live USB available at all times.
- After disk partition changes, always run `sudo update-grub`.

---

### Problem 4: Forgot sudo/root Password

**Symptoms / Error Message:**
```
sudo: Authentication failure
su: Authentication failure
```

**Possible Causes:**
- Password forgotten or mistyped
- Account locked after too many failed attempts
- sudoers file misconfigured

**Solution:**
```bash
# --- Method 1: Recovery mode (Ubuntu/Debian) ---

# 1. Reboot and hold SHIFT during BIOS to show GRUB menu
# 2. Select "Advanced options" → "Recovery mode"
# 3. From the recovery menu, select "root — Drop to root shell"

# 4. Remount filesystem as read-write
mount -o remount,rw /

# 5. Reset the root password
passwd root

# 6. Reset a user's password
passwd username

# 7. Re-enable a locked account
usermod -U username

# --- Method 2: Init=/bin/bash kernel parameter ---
# 1. At GRUB, press 'e' to edit the boot entry
# 2. Find the line starting with "linux" and append: init=/bin/bash
# 3. Press Ctrl+X to boot
# 4. Remount and change password
mount -o remount,rw /
passwd root
exec /sbin/init

# --- Verify sudo access ---
grep -E '^%sudo|^username' /etc/sudoers
sudo visudo   # safe way to edit sudoers
```

**Prevention:**
- Store credentials in an encrypted password manager.
- Configure at least one recovery user account with sudo privileges.
- Document emergency access procedures for servers.

---

### Problem 5: Disk Full (`df -h` Shows 100%)

**Symptoms / Error Message:**
```
No space left on device
df -h shows /dev/sda1   50G   50G     0 100% /
```

**Possible Causes:**
- Log files or core dumps consuming space
- APT/package manager cache not cleaned
- Docker images/volumes accumulating
- Large files left in `/tmp` or home directories

**Solution:**
```bash
# 1. Check disk usage overview
df -h

# 2. Find which directories use the most space
du -sh /* 2>/dev/null | sort -rh | head -20
du -sh /var/* 2>/dev/null | sort -rh | head -10

# 3. Find files larger than 100MB
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -20

# 4. Clean APT package cache
sudo apt clean
sudo apt autoremove --purge
sudo apt autoclean

# 5. Vacuum systemd journal logs
sudo journalctl --vacuum-size=200M
sudo journalctl --vacuum-time=2weeks

# 6. Clean /tmp
sudo rm -rf /tmp/*

# 7. Find and remove old log files
sudo find /var/log -name "*.gz" -delete
sudo find /var/log -name "*.old" -delete

# 8. Truncate (not delete) a log file safely
sudo truncate -s 0 /var/log/syslog

# 9. Remove old kernels (Ubuntu)
sudo apt autoremove --purge

# 10. Clean Docker resources
docker system prune -af --volumes
```

**Prevention:**
- Set up log rotation with `logrotate` and configure `journald` size limits in `/etc/systemd/journald.conf`.
- Monitor disk usage with cron + alerting: `df -h | awk '$5+0 > 80 {print $0}'`
- Configure `/etc/apt/apt.conf.d/` to auto-clean cache.

---

### Problem 6: Network Interface Not Working

**Symptoms / Error Message:**
```
RTNETLINK answers: Network is unreachable
eth0: ERROR while getting interface flags: No such device
```

**Possible Causes:**
- Network interface is down
- Driver not loaded for NIC
- NetworkManager service not running
- Cable unplugged or switch misconfigured (physical layer)

**Solution:**
```bash
# 1. List all network interfaces
ip link show
ip addr show

# 2. Bring an interface up manually
sudo ip link set eth0 up

# 3. Check if NetworkManager is running
sudo systemctl status NetworkManager

# 4. Restart NetworkManager
sudo systemctl restart NetworkManager

# 5. Use nmcli to check and manage connections
nmcli device status
nmcli connection show
nmcli device connect eth0

# 6. Check for hardware/driver issues
lshw -c network
lspci | grep -i net
dmesg | grep -i eth

# 7. List loaded kernel modules for networking
lsmod | grep -i net

# 8. Load a specific driver
sudo modprobe r8169   # example for Realtek NICs

# 9. For Wi-Fi issues — scan for networks
sudo iwlist wlan0 scan | grep ESSID

# 10. Check /etc/network/interfaces (Debian legacy)
cat /etc/network/interfaces
sudo systemctl restart networking
```

**Prevention:**
- Keep NIC firmware and drivers updated via `sudo apt install firmware-linux-nonfree`.
- Document static IP configurations in version control.
- Use `nmcli connection export` to back up network profiles.

---

### Problem 7: Cannot SSH to Server

**Symptoms / Error Message:**
```
ssh: connect to host 192.168.1.100 port 22: Connection refused
Permission denied (publickey,gssapi-keyex,gssapi-with-mic)
ssh: connect to host server.example.com port 22: Connection timed out
```

**Possible Causes:**
- SSH daemon not running on the server
- Firewall blocking port 22
- `authorized_keys` file has wrong permissions
- `sshd_config` has `PasswordAuthentication no` and no key configured

**Solution:**
```bash
# 1. Verbose debugging from the client side
ssh -v user@server
ssh -vvv user@server   # maximum verbosity

# 2. Test port connectivity
nc -zv server 22
telnet server 22

# 3. On the server — check sshd status
sudo systemctl status sshd
sudo systemctl start sshd
sudo systemctl enable sshd

# 4. Check firewall rules
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo iptables -L INPUT -n -v | grep 22

# 5. Verify authorized_keys permissions (critical!)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown $USER:$USER ~/.ssh/authorized_keys

# 6. Check sshd configuration
sudo sshd -t   # test config for syntax errors
sudo cat /etc/ssh/sshd_config | grep -v "^#" | grep -v "^$"

# 7. Check SSH logs for denial reason
sudo journalctl -u sshd -n 50
sudo tail -50 /var/log/auth.log

# 8. Temporarily allow password auth for testing
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl reload sshd
```

**Prevention:**
- Change SSH from port 22 to a non-standard port to reduce brute-force noise.
- Use fail2ban to block repeated failed login attempts.
- Always test key-based auth before disabling password auth.

---

### Problem 8: APT Package Lock / Broken Dependencies

**Symptoms / Error Message:**
```
E: Could not get lock /var/lib/dpkg/lock-frontend
E: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a'
E: Unmet dependencies. Try 'apt --fix-broken install'
```

**Possible Causes:**
- Another apt/dpkg process is running (update-manager, unattended-upgrades)
- Previous installation was interrupted (power loss, Ctrl+C)
- Conflicting package versions or broken PPA

**Solution:**
```bash
# 1. Check if another package manager is running
ps aux | grep -E "apt|dpkg"
sudo lsof /var/lib/dpkg/lock-frontend

# 2. Kill any stray apt/dpkg processes (get PID first)
# sudo kill <PID>

# 3. Remove stale lock files (only if no other process is running)
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
sudo rm /var/cache/apt/archives/lock

# 4. Reconfigure interrupted packages
sudo dpkg --configure -a

# 5. Fix broken dependencies
sudo apt --fix-broken install

# 6. Force-remove a problematic package
sudo dpkg --remove --force-remove-reinstreq <package-name>

# 7. Purge and reinstall
sudo apt purge <package-name>
sudo apt install <package-name>

# 8. Clean and update package lists
sudo apt clean
sudo apt update

# 9. Fix a broken PPA
sudo add-apt-repository --remove ppa:problematic/ppa
sudo apt update
```

**Prevention:**
- Never interrupt package installations with Ctrl+C; use `Ctrl+Z` and then `kill %1` if needed.
- Avoid mixing PPAs with official repositories for critical packages.
- Run `sudo apt update && sudo apt upgrade` regularly to prevent dependency drift.

---

### Problem 9: Slow System Performance

**Symptoms / Error Message:**
```
System feels sluggish, high load average in top
Load average: 8.45 3.21 1.78  (on a 2-core system)
```

**Possible Causes:**
- CPU-bound runaway process
- I/O bottleneck (slow disk)
- Memory exhaustion causing swap thrashing
- High swappiness causing premature swapping

**Solution:**
```bash
# 1. Get a real-time overview
top
htop   # more user-friendly (install with: sudo apt install htop)

# 2. Check load average and CPU
uptime
vmstat 1 5   # stats every 1 second for 5 iterations

# 3. Identify I/O bottlenecks
sudo iotop -o   # show only processes doing I/O
iostat -x 1 5

# 4. Check memory pressure
free -h
vmstat -s | head -20

# 5. Check and tune swappiness
cat /proc/sys/vm/swappiness
# Lower value = prefer RAM over swap (default is 60)
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# 6. Find memory-hungry processes
ps aux --sort=-%mem | head -20

# 7. Check for excessive context switching
pidstat -w 1 5

# 8. Profile disk I/O per process
sudo iotop -b -n 5 | sort -k10 -rn | head -10

# 9. Drop caches to reclaim memory (safe to do)
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

**Prevention:**
- Set up resource limits in `/etc/security/limits.conf` for users and services.
- Use `systemd` resource controls (`CPUQuota`, `MemoryMax`) for services.
- Monitor with Prometheus/Grafana or Netdata for trend analysis.

---

### Problem 10: High CPU Usage

**Symptoms / Error Message:**
```
top shows: PID 8432 "java"  %CPU 99.8
System fan running at maximum, system barely responsive
```

**Possible Causes:**
- Runaway process or infinite loop
- Malicious cryptocurrency miner
- Misconfigured service with tight retry loop
- Kernel bug or hardware interrupt storm

**Solution:**
```bash
# 1. Identify top CPU consumers
ps aux --sort=-%cpu | head -20

# 2. Real-time per-process CPU monitoring
top -o %CPU
htop   # press F6 to sort by CPU

# 3. Get more details on a process
ps -p <PID> -o pid,ppid,cmd,%cpu,%mem,etime

# 4. Check threads of a process
ps -p <PID> -L -o pid,tid,pcpu,comm

# 5. Send graceful termination signal first
kill -SIGTERM <PID>

# 6. Force-kill if unresponsive
kill -SIGKILL <PID>
# or
kill -9 <PID>

# 7. Reduce priority of a running process (not kill)
renice +10 -p <PID>

# 8. Start a new process with lower priority
nice -n 15 /path/to/command

# 9. Limit CPU usage with cpulimit
sudo apt install cpulimit
cpulimit -p <PID> -l 30   # limit to 30% CPU

# 10. Check for crypto miners
netstat -tulnp | grep -E "3333|4444|5555|7777|8888|9999"
```

**Prevention:**
- Implement CPU limits in systemd unit files with `CPUQuota=50%`.
- Set up monitoring alerts for sustained high CPU (>80% for >5 minutes).
- Audit running processes regularly: `ps aux | grep -v root | awk '{print $1}' | sort -u`.

---

### Problem 11: Out of Memory (OOM Killer)

**Symptoms / Error Message:**
```
Out of memory: Kill process 1234 (java) score 900 or sacrifice child
Killed
```

**Possible Causes:**
- Application memory leak
- Too many processes for available RAM
- No swap space configured
- Memory-intensive workload exceeds capacity

**Solution:**
```bash
# 1. Check current memory usage
free -h
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree"

# 2. Find OOM kill events in logs
sudo grep -i "oom\|out of memory\|killed process" /var/log/syslog | tail -20
sudo journalctl -k | grep -i "oom\|killed process"

# 3. Identify memory hogs
ps aux --sort=-%mem | head -20
smem -r | head -20   # sudo apt install smem

# 4. Check if swap exists
swapon --show

# 5. Add a swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make it permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 6. Check swap usage
swapon --show
vmstat -s | grep swap

# 7. Kill a memory-leaking process
kill -SIGTERM <PID>

# 8. Protect critical processes from OOM killer
echo -17 | sudo tee /proc/<PID>/oom_adj
```

**Prevention:**
- Always configure at least a swap file equal to the amount of RAM.
- Set `MemoryMax` in systemd unit files for memory-intensive services.
- Use `ulimit -v` to cap virtual memory for untrusted processes.

---

### Problem 12: Service Won't Start (systemctl Failure)

**Symptoms / Error Message:**
```
● nginx.service - A high performance web server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled)
   Active: failed (Result: exit-code)
```

**Possible Causes:**
- Configuration file syntax error
- Port already in use
- Missing dependency (database not running)
- Binary or library missing

**Solution:**
```bash
# 1. Get detailed status
sudo systemctl status nginx.service

# 2. View service logs (most useful command)
sudo journalctl -xe -u nginx.service
sudo journalctl -u nginx.service -n 100 --no-pager

# 3. Test configuration file syntax
sudo nginx -t                    # nginx
sudo apache2ctl configtest       # Apache
sudo sshd -t                     # sshd
sudo named-checkconf             # BIND DNS

# 4. Check if port is already in use
sudo ss -tulnp | grep :80
sudo lsof -i :80

# 5. Kill the conflicting process
sudo kill <PID>

# 6. Check service dependencies
systemctl list-dependencies nginx.service

# 7. Start with verbose output for debugging
sudo journalctl -f &   # follow logs in background
sudo systemctl start nginx

# 8. Enable service to start on boot
sudo systemctl enable nginx

# 9. Reload daemon after editing unit files
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

**Prevention:**
- Always validate configuration files before reloading services.
- Use `systemctl edit <service>` to create override files rather than modifying unit files directly.
- Test changes in staging before production.

---

### Problem 13: Cron Jobs Not Running

**Symptoms / Error Message:**
```
# (no output — job silently fails to execute)
Expected script output not appearing; no email from cron
```

**Possible Causes:**
- Incorrect cron syntax
- PATH not set in crontab environment
- Script lacks execute permissions
- Cron service not running
- Output not redirected (errors disappear silently)

**Solution:**
```bash
# 1. List current user's crontab
crontab -l

# 2. Edit crontab
crontab -e

# 3. Check cron service status
sudo systemctl status cron

# 4. View cron logs
grep CRON /var/log/syslog | tail -20
sudo journalctl -u cron -n 50

# 5. Test cron syntax online or with a validator
# Quick test: add a job that runs every minute
* * * * * /usr/bin/date >> /tmp/cron_test.log 2>&1

# 6. Set PATH explicitly in crontab (critical!)
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# 7. Redirect both stdout and stderr to a log
0 2 * * * /path/to/script.sh >> /var/log/myscript.log 2>&1

# 8. Check script is executable
chmod +x /path/to/script.sh

# 9. Use absolute paths for all commands in scripts
#!/bin/bash
/usr/bin/rsync -av /source/ /backup/

# 10. Check for cron.d and cron.daily entries
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.weekly/
```

**Prevention:**
- Always use absolute paths in cron scripts.
- Redirect output to a log file and monitor it.
- Test scripts manually as the cron user before scheduling.

---

### Problem 14: Filesystem Errors on Boot (fsck)

**Symptoms / Error Message:**
```
/dev/sda1 contains a file system with errors, check forced.
Inodes that were part of a corrupted orphan linked list found.
Give root password for maintenance (or press Control-D to continue):
```

**Possible Causes:**
- Improper shutdown (power loss, hard reboot)
- Scheduled check interval reached
- Failing hard drive producing bad sectors
- Journaling failure

**Solution:**
```bash
# 1. Run fsck on an unmounted partition (do NOT run on mounted /)
sudo umount /dev/sdb1
sudo fsck -y /dev/sdb1   # -y auto-answers yes to all repairs

# 2. Force fsck on next reboot (for root partition)
sudo touch /forcefsck
# or
sudo tune2fs -C 0 /dev/sda1   # reset mount count
sudo tune2fs -c 1 /dev/sda1   # force check on next mount

# 3. Check filesystem without repairs (read-only)
sudo fsck -n /dev/sdb1

# 4. Force a check on / at next boot
sudo shutdown -rF now

# 5. View filesystem info and last check date
sudo tune2fs -l /dev/sda1 | grep -E "Last checked|Check interval|Mount count|Maximum mount"

# 6. Extended fsck with bad-block scan
sudo fsck -c /dev/sdb1   # may take hours on large drives

# 7. Repair from emergency shell
# Mount filesystem read-write in recovery
mount -o remount,rw /
fsck -y /dev/sda1
```

**Prevention:**
- Configure `tune2fs -c 30 /dev/sda1` to force fsck every 30 mounts.
- Use UPS (Uninterruptible Power Supply) to prevent improper shutdowns.
- Consider using XFS or btrfs for better journaling recovery.

---

### Problem 15: Failing Hard Drive

**Symptoms / Error Message:**
```
kernel: ata1.00: error: { UNC }
kernel: Buffer I/O error on dev sda, logical block 12345678
smartctl shows: 197 Current_Pending_Sector  0x0032   100  ... 23
```

**Possible Causes:**
- Physical drive failure (bearing, platter, head crash)
- Bit rot or bad sectors developing
- SATA/power cable issue
- Drive reaching end of life

**Solution:**
```bash
# 1. Install S.M.A.R.T. tools
sudo apt install smartmontools

# 2. Quick SMART health check
sudo smartctl -H /dev/sda

# 3. Full SMART data dump
sudo smartctl -a /dev/sda

# 4. Run a short self-test (~2 minutes)
sudo smartctl -t short /dev/sda
# Check results after 5 minutes:
sudo smartctl -a /dev/sda | grep -A 20 "SMART Self-test log"

# 5. Run a long self-test (hours, run overnight)
sudo smartctl -t long /dev/sda

# 6. Scan for bad blocks (read-only, non-destructive)
sudo badblocks -v /dev/sdb > /tmp/badblocks.txt

# 7. Check kernel I/O error messages
sudo dmesg | grep -i "error\|ata\|sda\|sdb\|I/O"

# 8. Monitor SMART attributes to watch
sudo smartctl -a /dev/sda | grep -E "Reallocated_Sector|Pending_Sector|Offline_Uncorrectable|Spin_Retry"

# 9. Clone a failing drive before it dies completely
sudo ddrescue -d -r3 /dev/sda /dev/sdb /tmp/recovery.log
```

**Prevention:**
- Enable SMART monitoring daemon: `sudo systemctl enable smartd && sudo systemctl start smartd`.
- Configure `/etc/smartd.conf` to email alerts on SMART failures.
- Maintain regular off-site backups — all drives eventually fail.

---

### Problem 16: DNS Resolution Failure

**Symptoms / Error Message:**
```
ping: google.com: Temporary failure in name resolution
curl: (6) Could not resolve host: api.example.com
```

**Possible Causes:**
- `/etc/resolv.conf` missing or pointing to wrong nameserver
- `systemd-resolved` service not running
- ISP DNS outage
- `/etc/hosts` corruption
- Network interface not configured with DNS

**Solution:**
```bash
# 1. Test if DNS works at all
dig @8.8.8.8 google.com        # Google's DNS
dig @1.1.1.1 google.com        # Cloudflare's DNS
nslookup google.com 8.8.8.8

# 2. Check current DNS configuration
cat /etc/resolv.conf
resolvectl status

# 3. Temporarily fix by adding a nameserver
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# 4. Check systemd-resolved status
sudo systemctl status systemd-resolved
sudo systemctl restart systemd-resolved

# 5. Flush DNS cache
sudo resolvectl flush-caches
sudo systemd-resolve --flush-caches

# 6. Permanent DNS fix via NetworkManager
nmcli connection show
nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection up "Wired connection 1"

# 7. Check /etc/hosts for conflicts
cat /etc/hosts

# 8. Test with traceroute to see where resolution fails
traceroute -n google.com

# 9. Check nsswitch.conf order
cat /etc/nsswitch.conf | grep hosts
```

**Prevention:**
- Configure redundant DNS servers (primary and secondary) in all network profiles.
- Use `systemd-resolved` with fallback DNS configured in `/etc/systemd/resolved.conf`.
- Set `DNSFallbackPolicy=any` in resolved.conf for resilience.

---

### Problem 17: Firewall Blocking Connections

**Symptoms / Error Message:**
```
curl: (7) Failed to connect to server port 443: Connection refused
nc -zv server 80 → Connection refused
```

**Possible Causes:**
- UFW or iptables rule blocking the port
- Service only listening on localhost (127.0.0.1) not all interfaces
- Cloud provider security group blocking the port (AWS, GCP, Azure)
- firewalld blocking on RHEL/CentOS

**Solution:**
```bash
# 1. Check UFW status and rules
sudo ufw status verbose
sudo ufw status numbered

# 2. Allow a specific port
sudo ufw allow 443/tcp
sudo ufw allow from 192.168.1.0/24 to any port 22

# 3. Delete a blocking rule
sudo ufw delete deny 80/tcp
sudo ufw delete 3   # delete rule #3

# 4. Check iptables rules directly
sudo iptables -L INPUT -n -v --line-numbers
sudo iptables -L OUTPUT -n -v

# 5. Allow a port with iptables directly
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4

# 6. Check what the service is actually listening on
sudo ss -tulnp | grep :443
sudo netstat -tulnp | grep :443

# 7. For firewalld (RHEL/CentOS/Fedora)
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# 8. Temporarily disable firewall for testing (re-enable after!)
sudo ufw disable
# Test your connection
sudo ufw enable
```

**Prevention:**
- Use `ufw` for desktop/simple servers; consider nftables for complex environments.
- Document all firewall rules in version-controlled configuration management (Ansible, etc.).
- Test connectivity from outside the host after making changes.

---

### Problem 18: SSL Certificate Errors

**Symptoms / Error Message:**
```
SSL_ERROR_RX_RECORD_TOO_LONG
curl: (60) SSL certificate problem: certificate has expired
certificate verify failed: certificate is not yet valid
```

**Possible Causes:**
- Certificate has expired
- System clock is wrong (causes "not yet valid" errors)
- Certificate chain is incomplete (missing intermediate)
- Self-signed certificate not trusted

**Solution:**
```bash
# 1. Check certificate details
openssl s_client -connect example.com:443 -servername example.com

# 2. Quick certificate expiry check
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# 3. Check a certificate file directly
openssl x509 -in /etc/ssl/certs/mycert.pem -noout -text | grep -E "Not Before|Not After|Subject|Issuer"

# 4. Renew Let's Encrypt certificate
sudo certbot renew
sudo certbot renew --dry-run   # test first

# 5. Force renewal for a specific domain
sudo certbot certonly --force-renewal -d example.com

# 6. Check certificate chain
openssl s_client -connect example.com:443 -showcerts 2>/dev/null | grep "s:\|i:"

# 7. Fix system certificate store
sudo update-ca-certificates

# 8. Trust a self-signed cert (Ubuntu/Debian)
sudo cp mycert.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# 9. Check if system time is correct (common cause)
timedatectl
sudo ntpdate pool.ntp.org
```

**Prevention:**
- Use `certbot renew` in a cron job or systemd timer for automatic Let's Encrypt renewal.
- Set calendar alerts 30 days before manual certificate expiry dates.
- Monitor certificate expiry with `ssl-cert-check` or a monitoring platform.

---

### Problem 19: Running Out of Inodes

**Symptoms / Error Message:**
```
No space left on device
df -h shows space available but df -i shows 100% inodes used
```

**Possible Causes:**
- Millions of small files (mail queues, session files, cache)
- PHP sessions or email spool accumulation
- Backup software creating many small metadata files
- Misconfigured application creating temp files

**Solution:**
```bash
# 1. Check inode usage
df -i
df -ih   # human-readable

# 2. Find the directory using the most inodes
for dir in /*; do echo "$dir: $(find $dir -xdev 2>/dev/null | wc -l)"; done | sort -t: -k2 -rn | head -10

# 3. More precise inode usage by directory
sudo find / -xdev -printf '%h\n' 2>/dev/null | sort | uniq -c | sort -rn | head -20

# 4. Clean up PHP session files (common culprit)
sudo find /var/lib/php/sessions/ -type f -mtime +7 -delete

# 5. Clean up mail queue
sudo postsuper -d ALL   # Postfix: delete all queued mail
ls /var/spool/mqueue/ | wc -l

# 6. Find and remove many small files (e.g., old temp files)
sudo find /tmp -type f -atime +7 -delete

# 7. Check what types of files are numerous
sudo find /var -type f -name "*.tmp" 2>/dev/null | wc -l
sudo find /var -type f -name "sess_*" 2>/dev/null | wc -l
```

**Prevention:**
- Configure `tmpfs` for `/tmp` to avoid inode consumption on the main filesystem.
- Set session garbage collection properly in PHP (`session.gc_probability`).
- Monitor inodes alongside disk space in your alerting setup.

---

### Problem 20: Zombie Processes

**Symptoms / Error Message:**
```
ps aux shows: 8921 Z  defunct   php-fpm: pool www <defunct>
Accumulating zombie processes visible in htop
```

**Possible Causes:**
- Parent process not calling `wait()` to collect child exit status
- Parent process crashed, leaving orphan children
- Buggy application that spawns children without cleanup

**Solution:**
```bash
# 1. Find all zombie processes
ps aux | grep "Z"
ps -eo pid,ppid,stat,cmd | grep " Z "

# 2. Identify the parent process
ps -o ppid= -p <ZOMBIE_PID>

# 3. Send SIGCHLD to the parent (asks parent to collect child)
kill -SIGCHLD <PARENT_PID>

# 4. If parent is unresponsive, kill it
kill -SIGTERM <PARENT_PID>

# 5. Force kill parent if needed (orphaned zombies are then adopted by init and cleaned)
kill -9 <PARENT_PID>

# 6. Monitor zombie count
watch -n 5 'ps aux | grep -c "Z"'

# 7. Check if a service is responsible
systemctl status <service-name>
sudo journalctl -u <service-name> | grep "zombie\|defunct"

# 8. Restart the offending service cleanly
sudo systemctl restart <service-name>
```

**Prevention:**
- Zombies are a sign of application bugs — report to the software vendor.
- Ensure applications properly handle `SIGCHLD` and call `waitpid()`.
- Periodically restart services known to accumulate zombies and file bug reports.

---

### Problem 21: Screen Resolution Wrong

**Symptoms / Error Message:**
```
Display stuck at 1024x768; cannot select higher resolution
xrandr: cannot find crtc for output HDMI-1
```

**Possible Causes:**
- Wrong or missing display driver
- Monitor EDID not being read correctly
- Xorg configuration conflict
- Virtual machine display adapter limitations

**Solution:**
```bash
# 1. Check available resolutions and outputs
xrandr
xrandr --query

# 2. Set a specific resolution
xrandr --output HDMI-1 --mode 1920x1080

# 3. Set resolution and refresh rate
xrandr --output HDMI-1 --mode 1920x1080 --rate 60

# 4. Create a custom resolution mode if not available
cvt 1920 1080 60   # generates Modeline
# Example output: Modeline "1920x1080_60.00" 173.00 1920 2048 2248 2576 ...

xrandr --newmode "1920x1080_60.00" 173.00 1920 2048 2248 2576 1080 1083 1088 1120 -hsync +vsync
xrandr --addmode HDMI-1 1920x1080_60.00
xrandr --output HDMI-1 --mode 1920x1080_60.00

# 5. Check installed GPU driver
lspci -v | grep -A 10 VGA
glxinfo | grep "OpenGL renderer"

# 6. Install NVIDIA drivers (Ubuntu)
ubuntu-drivers devices
sudo ubuntu-drivers autoinstall

# 7. Create/edit Xorg configuration
sudo X -configure   # generates /root/xorg.conf.new
sudo cp /root/xorg.conf.new /etc/X11/xorg.conf

# 8. For VMs — install guest additions
# VirtualBox: Devices > Insert Guest Additions CD
```

**Prevention:**
- Install appropriate GPU drivers before expecting full resolution support.
- Back up a working `/etc/X11/xorg.conf` before making changes.
- On servers, consider using Wayland or headless Xvfb for display needs.

---

### Problem 22: No Audio

**Symptoms / Error Message:**
```
No audio output; speaker icon shows muted or unavailable
aplay: main:722: audio open error: No such file or directory
```

**Possible Causes:**
- Channel muted in ALSA
- PulseAudio in bad state
- Wrong output device selected
- Audio driver not loaded

**Solution:**
```bash
# 1. List audio playback devices
aplay -l
aplay -L

# 2. Test audio directly through ALSA
speaker-test -t wav -c 2
aplay /usr/share/sounds/alsa/Front_Left.wav

# 3. Open ALSA mixer and unmute channels
alsamixer
# Press M to toggle mute on highlighted channel
# Press arrow keys to adjust volume

# 4. Check PulseAudio status
pulseaudio --check
pulseaudio --check && echo "Running" || echo "Not running"

# 5. Kill and restart PulseAudio
pulseaudio --kill
pulseaudio --start

# 6. GUI mixer for PulseAudio
sudo apt install pavucontrol
pavucontrol   # check Output Devices and Playback tabs

# 7. List PulseAudio sinks
pactl list sinks
pactl list short sinks

# 8. Set default output sink
pactl set-default-sink <sink-name>

# 9. Check if audio module is loaded
lsmod | grep snd
dmesg | grep -i audio

# 10. Reload ALSA
sudo alsa force-reload
```

**Prevention:**
- Install `pavucontrol` for a graphical PulseAudio mixer — it prevents most audio confusion.
- Add your user to the `audio` group: `sudo usermod -aG audio $USER`.
- Keep audio firmware updated: `sudo apt install alsa-firmware-loaders`.

---

### Problem 23: Time/Timezone Wrong

**Symptoms / Error Message:**
```
System shows wrong time; SSL certificates show as expired
Cron jobs running at wrong hours; log timestamps incorrect
```

**Possible Causes:**
- Timezone not set or set incorrectly
- NTP synchronization not enabled
- Hardware clock (RTC) drifted
- Dual-boot with Windows changing hardware clock

**Solution:**
```bash
# 1. Check current time and timezone
timedatectl
date

# 2. List available timezones
timedatectl list-timezones | grep -i america
timedatectl list-timezones | grep -i london

# 3. Set timezone
sudo timedatectl set-timezone America/New_York
sudo timedatectl set-timezone UTC

# 4. Alternative: interactive timezone selection
sudo dpkg-reconfigure tzdata

# 5. Enable NTP synchronization
sudo timedatectl set-ntp true
timedatectl   # verify NTPSynchronized: yes

# 6. Manually sync time with NTP
sudo ntpdate pool.ntp.org
sudo chronyc makestep   # if using chrony

# 7. Check NTP service
sudo systemctl status systemd-timesyncd
sudo systemctl status ntp   # if using ntpd
sudo journalctl -u systemd-timesyncd

# 8. Fix hardware clock
sudo hwclock --systohc   # write system time to hardware clock
sudo hwclock --hctosys   # read hardware clock to system time

# 9. For dual-boot Windows fix
sudo timedatectl set-local-rtc 0 --adjust-system-clock
```

**Prevention:**
- Always set UTC on servers; configure timezone per user with `TZ` environment variable.
- Enable and verify `systemd-timesyncd` or `chrony` on all servers.
- Monitor NTP sync status with `timedatectl show`.

---

### Problem 24: Locale/Encoding Errors

**Symptoms / Error Message:**
```
perl: warning: Setting locale failed.
locale: Cannot set LC_ALL to default locale: No such file or directory
UnicodeDecodeError: 'ascii' codec can't decode byte
```

**Possible Causes:**
- Locale not generated on the system
- `LANG` or `LC_ALL` environment variable not set
- SSH connection not forwarding locale
- Application using ASCII where UTF-8 is needed

**Solution:**
```bash
# 1. Check current locale settings
locale
echo $LANG
echo $LC_ALL

# 2. List available locales
locale -a

# 3. Generate locale (Ubuntu/Debian)
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

# 4. Interactive locale reconfiguration
sudo dpkg-reconfigure locales

# 5. Set locale in current session
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
export LC_CTYPE=en_US.UTF-8

# 6. Make permanent for all users
echo 'LANG=en_US.UTF-8' | sudo tee /etc/default/locale
echo 'LC_ALL=en_US.UTF-8' >> /etc/default/locale

# 7. Fix SSH locale forwarding issue (on server)
sudo sed -i 's/AcceptEnv LANG LC_\*/# AcceptEnv LANG LC_*/' /etc/ssh/sshd_config
sudo systemctl reload sshd

# 8. Rebuild locale database
sudo locale-gen --purge en_US.UTF-8
sudo update-locale
```

**Prevention:**
- Always specify `en_US.UTF-8` (or your locale) in `/etc/default/locale` during system setup.
- In Docker containers, add `ENV LANG=C.UTF-8` to avoid locale warnings.
- For Python scripts, add `# -*- coding: utf-8 -*-` and use `sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')`.

---

### Problem 25: Python/pip Version Conflicts

**Symptoms / Error Message:**
```
ModuleNotFoundError: No module named 'requests'
ERROR: Cannot uninstall 'urllib3'. It is a distutils installed project
pip: command not found
```

**Possible Causes:**
- Multiple Python versions installed (2.7, 3.8, 3.11...)
- Pip installing to wrong Python version
- System packages conflicting with pip packages
- Virtual environment not activated

**Solution:**
```bash
# 1. Check Python versions installed
python --version 2>/dev/null
python3 --version
ls /usr/bin/python*

# 2. Always use python3 and pip3 explicitly
python3 -m pip install requests
python3 -m pip list

# 3. Check which pip is which
which pip
which pip3
pip3 --version   # should show Python 3.x

# 4. Create a virtual environment (best practice)
python3 -m venv myenv
source myenv/bin/activate
pip install requests   # installs only in this venv
deactivate   # exit venv

# 5. Manage multiple Python versions with update-alternatives
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 2
sudo update-alternatives --config python3

# 6. Use pipx for CLI tools (avoids global conflicts)
sudo apt install pipx
pipx install black
pipx install ansible

# 7. Fix broken pip installation
python3 -m ensurepip --upgrade
python3 -m pip install --upgrade pip

# 8. List and clean up conflicting packages
pip3 check   # show broken requirements
pip3 list --outdated
```

**Prevention:**
- **Always** use virtual environments for project dependencies.
- Use `python3 -m pip` instead of `pip` or `pip3` to ensure you're using the right interpreter.
- Consider `pyenv` for managing multiple Python versions cleanly.

---

### Problem 26: Git Merge Conflicts

**Symptoms / Error Message:**
```
CONFLICT (content): Merge conflict in src/main.py
Automatic merge failed; fix conflicts and then commit the result.
error: Your local changes to the following files would be overwritten by merge
```

**Possible Causes:**
- Two branches modified the same lines differently
- Rebasing or cherry-picking onto a diverged branch
- Force-push rewrote shared history

**Solution:**
```bash
# 1. Check status to see all conflicted files
git status

# 2. See what conflicts exist
git diff

# 3. Open each conflicted file — look for conflict markers:
# <<<<<<< HEAD
# your changes
# =======
# their changes
# >>>>>>> branch-name

# 4. Use a merge tool
git mergetool          # uses default tool
git mergetool --tool=vimdiff
git mergetool --tool=meld   # needs: sudo apt install meld

# 5. After manually editing, mark as resolved
git add src/main.py
git commit   # complete the merge

# 6. Accept entirely "ours" or "theirs" for a file
git checkout --ours src/config.py
git checkout --theirs src/config.py
git add src/config.py

# 7. Abort the merge and start over
git merge --abort
git rebase --abort
git cherry-pick --abort

# 8. See what commit introduced conflicting changes
git log --merge -p src/main.py
git blame src/main.py

# 9. Use rebase instead of merge for cleaner history
git fetch origin
git rebase origin/main
# Fix conflicts then:
git rebase --continue
```

**Prevention:**
- Communicate with teammates about which files are being modified.
- Keep branches short-lived and merge/rebase frequently to minimize divergence.
- Configure a visual merge tool: `git config --global merge.tool meld`.

---

### Problem 27: Docker Container Won't Start

**Symptoms / Error Message:**
```
docker: Error response from daemon: driver failed programming external connectivity
Error: No such container: myapp
container exited with code 1
```

**Possible Causes:**
- Port already in use on the host
- Volume mount path doesn't exist or has wrong permissions
- Missing environment variables
- Image not found or corrupted
- Out of disk space

**Solution:**
```bash
# 1. Check container logs (most useful first step)
docker logs mycontainer
docker logs mycontainer --tail 50
docker logs mycontainer 2>&1 | grep -i error

# 2. Inspect container for configuration issues
docker inspect mycontainer
docker inspect mycontainer | python3 -m json.tool | grep -A 5 "Error\|State"

# 3. Check why container exited
docker ps -a   # shows exit codes
docker inspect mycontainer | grep -A 5 '"ExitCode"'

# 4. Check for port conflicts
sudo ss -tulnp | grep :8080
docker ps | grep 8080

# 5. Run container interactively for debugging
docker run -it --entrypoint /bin/bash myimage
docker run -it --rm myimage /bin/bash

# 6. Check volume permissions
docker inspect mycontainer | grep -A 10 Mounts
ls -la /path/on/host   # check ownership

# Fix volume permissions
sudo chown -R 1000:1000 /path/on/host   # adjust UID/GID as needed

# 7. Check disk space for Docker
df -h
docker system df   # show Docker disk usage

# 8. Clean up Docker resources
docker system prune -f
docker volume prune -f

# 9. Check environment variables
docker inspect mycontainer | grep -A 20 '"Env"'

# 10. Rebuild the image
docker build --no-cache -t myimage .
docker run -d --name mycontainer -p 8080:80 myimage
```

**Prevention:**
- Use `docker-compose` with health checks and restart policies.
- Always log container output to a persistent logging driver.
- Run `docker system prune` regularly to avoid disk exhaustion.

---

## General Troubleshooting Methodology

When facing an unknown Linux problem, apply this systematic approach:

```
1. OBSERVE   → What exactly is the error message? When does it occur?
2. LOCATE    → Which component is involved? (hardware, kernel, service, app)
3. DIAGNOSE  → Check logs: journalctl, /var/log/syslog, dmesg, app logs
4. ISOLATE   → Reproduce the issue in the simplest possible way
5. FIX       → Apply the solution; test thoroughly
6. PREVENT   → Document what happened and add monitoring to catch it early
```

### Essential Log Locations

| Log | Purpose |
|-----|---------|
| `journalctl -xe` | Systemd service failures (most useful) |
| `/var/log/syslog` | General system messages |
| `/var/log/auth.log` | Authentication and sudo events |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dpkg.log` | Package installation history |
| `dmesg` | Kernel ring buffer (hardware, boot) |
| `/var/log/nginx/error.log` | Nginx errors |
| `/var/log/apache2/error.log` | Apache errors |

---

*See also: Appendix A — Complete Command Reference, Appendix B — Aliases and Acronyms*
