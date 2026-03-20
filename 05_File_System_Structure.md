# 05 — File System Structure

> **Difficulty:** Beginner–Intermediate | **Time Estimate:** 60–90 minutes

---

## Learning Objectives

By the end of this lesson you will be able to:

- Describe the Linux Filesystem Hierarchy Standard (FHS)
- Explain the purpose of every top-level directory under `/`
- Distinguish between absolute and relative paths
- Identify the seven Linux file types
- Find and understand hidden (dot) files
- Explain Linux file naming conventions
- Locate and read key configuration files

---

## 1. The Linux Filesystem Hierarchy Standard (FHS)

Unlike Windows (which has separate drive letters like `C:\`, `D:\`), Linux uses a **single unified tree** rooted at `/` (called "root" or "slash"). Everything — hard drives, USB sticks, network shares, device files — hangs off this single root.

The **Filesystem Hierarchy Standard (FHS)** is maintained by the Linux Foundation and defines where specific types of files must live. This means a skill you learn on Ubuntu applies on Fedora, Debian, or any other Linux distribution.

```bash
# View the top-level directory structure
ls /

# View as a tree (install tree if needed)
sudo apt install tree -y
tree -L 1 /

# See filesystem disk usage
df -hT

# See directory sizes
du -sh /* 2>/dev/null | sort -rh | head -15
```

---

## 2. The Directory Tree

```
/
├── bin      → Essential user command binaries
├── boot     → Boot loader and kernel files
├── dev      → Device files
├── etc      → Host-specific configuration files
├── home     → User home directories
│   ├── alice
│   └── bob
├── lib      → Essential shared libraries
├── lib64    → 64-bit shared libraries
├── media    → Mount points for removable media
├── mnt      → Temporary mount points
├── opt      → Optional/third-party software
├── proc     → Virtual filesystem: process/kernel info
├── root     → Root user's home directory
├── run      → Run-time variable data (since boot)
├── sbin     → System administration binaries
├── srv      → Data for services (http, ftp, etc.)
├── sys      → Virtual filesystem: hardware/kernel info
├── tmp      → Temporary files (cleared on reboot)
├── usr      → Read-only user data
│   ├── bin  → Most user commands
│   ├── lib  → Libraries for /usr/bin and /usr/sbin
│   ├── local→ Locally installed software
│   ├── sbin → Non-essential system admin commands
│   └── share→ Architecture-independent data
└── var      → Variable data (logs, spools, caches)
    ├── log  → Log files
    ├── mail → Mailbox files
    ├── spool→ Print queues, cron jobs
    └── tmp  → Temporary files (NOT cleared on reboot)
```

---

## 3. Every Directory Explained

### `/` — Root

The top of the entire filesystem tree. Everything starts here.

```bash
# Number of items directly under /
ls / | wc -l

# Total disk space used by everything
du -sh / 2>/dev/null
```

### `/bin` — Essential User Binaries

Contains essential command-line utilities required for basic system operation — even if `/usr` is not mounted. In modern Ubuntu, `/bin` is a symbolic link to `/usr/bin`.

```bash
# Common commands in /bin
ls /bin | head -20

# See that /bin is a symlink to /usr/bin on modern systems
ls -la / | grep " bin"
readlink /bin       # → usr/bin
```

**Examples:** `ls`, `cp`, `mv`, `mkdir`, `rm`, `cat`, `echo`, `bash`, `grep`

### `/boot` — Boot Loader Files

Contains the Linux kernel (`vmlinuz`), initial RAM disk (`initrd`/`initramfs`), and bootloader configuration (GRUB).

```bash
ls -lh /boot

# See the kernel files
ls /boot/vmlinuz*
ls /boot/initrd*

# View GRUB configuration
cat /boot/grub/grub.cfg | head -30

# Show GRUB menu entries
grep "menuentry" /boot/grub/grub.cfg
```

### `/dev` — Device Files

In Linux, **everything is a file** — including hardware devices. `/dev` contains device files that represent hardware or virtual devices.

```bash
ls /dev | head -20

# Common devices:
# /dev/sda    — first SATA/SCSI hard drive
# /dev/sda1   — first partition on sda
# /dev/nvme0n1 — first NVMe SSD
# /dev/tty    — current terminal
# /dev/null   — the black hole (discards all input)
# /dev/zero   — infinite stream of zero bytes
# /dev/random — random number generator
# /dev/mem    — physical memory access

# Read from /dev/random (generates random bytes)
head -c 16 /dev/urandom | xxd

# Write to /dev/null (disappears)
echo "This disappears" > /dev/null

# Get zeros from /dev/zero (useful for creating empty files)
dd if=/dev/zero of=/tmp/empty_1mb bs=1M count=1
ls -lh /tmp/empty_1mb
rm /tmp/empty_1mb
```

### `/etc` — Configuration Files

The most important directory for system administrators. Contains **all system-wide configuration files** — text files that control how the system and its services behave.

```bash
ls /etc | head -30

# View key configuration files
cat /etc/hostname        # System hostname
cat /etc/hosts           # Static hostname to IP mappings
cat /etc/fstab           # Filesystem mount table
cat /etc/os-release      # OS identification
head -5 /etc/passwd      # User account information
head -5 /etc/group       # Group information

# Count how many config files are in /etc
find /etc -maxdepth 1 -type f | wc -l
```

**Critical files in `/etc`:**

| File | Purpose |
|---|---|
| `/etc/passwd` | User accounts (name, UID, GID, home, shell) |
| `/etc/shadow` | Encrypted passwords |
| `/etc/group` | Group definitions |
| `/etc/fstab` | Filesystems mounted at boot |
| `/etc/hosts` | Static hostname resolution |
| `/etc/resolv.conf` | DNS resolver configuration |
| `/etc/hostname` | System hostname |
| `/etc/sudoers` | Who can use sudo |
| `/etc/crontab` | System-wide cron schedule |
| `/etc/ssh/sshd_config` | SSH server configuration |
| `/etc/apt/sources.list` | APT software repositories |
| `/etc/network/interfaces` | Network interface config |
| `/etc/environment` | System-wide environment variables |

### `/home` — User Home Directories

Each regular user has a directory here: `/home/username`. This is the user's private space for documents, downloads, configurations, and settings.

```bash
ls -la /home

# Your home directory
ls -la ~
ls -la $HOME

# See how much space each user is using
du -sh /home/* 2>/dev/null

# Total size of your home directory
du -sh ~
```

### `/lib` and `/lib64` — Shared Libraries

Contains shared library files (`*.so` files) — like Windows DLLs. These are code shared between multiple programs to save memory and disk space.

```bash
ls /lib | head -10
ls /lib/x86_64-linux-gnu/ | head -10

# See what libraries a binary depends on
ldd /bin/ls
ldd /usr/bin/vim
```

### `/media` — Removable Media Mount Points

When you plug in a USB drive or insert a DVD, Ubuntu automatically mounts it under `/media/username/device_label`.

```bash
ls /media

# View mounted removable media
mount | grep /media

# Manually mount a USB drive (if not auto-mounted)
sudo mkdir -p /media/myusb
sudo mount /dev/sdb1 /media/myusb
ls /media/myusb
sudo umount /media/myusb
```

### `/mnt` — Manual Mount Points

Traditionally used for **temporarily mounting filesystems** manually (network shares, additional drives).

```bash
# Mount a network share (NFS example)
# sudo mount -t nfs server:/share /mnt/nfs_share

# Mount an ISO file
sudo mkdir -p /mnt/iso
sudo mount -o loop /path/to/file.iso /mnt/iso
ls /mnt/iso
sudo umount /mnt/iso
```

### `/opt` — Optional Software

Third-party and commercial software that does not follow the standard `/usr` layout is installed here (e.g., Google Chrome, VMware, custom enterprise apps).

```bash
ls /opt

# Google Chrome installs here on Ubuntu
ls /opt/google/chrome 2>/dev/null || echo "Chrome not installed"

# VirtualBox puts extensions here
ls /opt/VirtualBox 2>/dev/null || echo "VirtualBox not installed"
```

### `/proc` — Process and Kernel Virtual Filesystem

`/proc` is a **pseudo-filesystem** — it does not exist on disk. The kernel creates it in memory to expose process and system information as files.

```bash
# Process information (each process has a /proc/PID directory)
ls /proc | grep "^[0-9]" | head -10     # List process IDs

# Get info about a specific process (PID 1 = init/systemd)
cat /proc/1/status | head -10
cat /proc/1/cmdline | tr '\0' ' '
echo ""

# System information
cat /proc/cpuinfo | grep "model name" | head -1
cat /proc/meminfo | head -5
cat /proc/version         # Kernel version
cat /proc/uptime          # Seconds since boot
cat /proc/loadavg         # System load averages
cat /proc/mounts          # Currently mounted filesystems
cat /proc/net/dev         # Network interface statistics
```

### `/root` — Root User's Home

The superuser (`root`)'s home directory. Separate from `/home` so it is accessible even if `/home` is on a different (unmounted) partition.

```bash
# You need root access to view this
sudo ls -la /root

# Root's shell configuration
sudo cat /root/.bashrc 2>/dev/null | head -5
```

### `/run` — Runtime Data

Contains temporary runtime data for processes since the last boot: PID files, sockets, lock files.

```bash
ls /run
ls /run/systemd/ | head -10

# PID file example (contains the PID of a running daemon)
cat /run/sshd.pid 2>/dev/null
```

### `/sbin` — System Administration Binaries

Commands intended primarily for the system administrator (root). On modern systems, `/sbin` is a symlink to `/usr/sbin`.

```bash
ls /sbin | head -15
# Examples: fdisk, mkfs, fsck, iptables, ip, ifconfig, shutdown, reboot
```

### `/srv` — Service Data

Data served by this system (web server files, FTP files, etc.). Not commonly used in Ubuntu desktop.

```bash
ls /srv
# An Apache web server might store files at /srv/http/
```

### `/sys` — System Virtual Filesystem (sysfs)

Like `/proc`, `/sys` is a virtual filesystem exposing kernel and hardware information. Used by the kernel and udev to manage devices.

```bash
ls /sys
ls /sys/class/net/         # Network interfaces
ls /sys/class/block/       # Block devices

# Read CPU frequency (if available)
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq 2>/dev/null

# Read battery level (laptops)
cat /sys/class/power_supply/BAT0/capacity 2>/dev/null || echo "No battery"

# Read backlight level
ls /sys/class/backlight/ 2>/dev/null
```

### `/tmp` — Temporary Files

Programs store temporary files here. **This directory is cleared on every reboot.** World-writable (any user can write here).

```bash
ls -la /tmp | head -10

# Create a temp file safely
tmp_file=$(mktemp)
echo "temporary data" > "$tmp_file"
cat "$tmp_file"
rm "$tmp_file"

# Create a temp directory
tmp_dir=$(mktemp -d)
echo "Working in $tmp_dir"
rmdir "$tmp_dir"
```

### `/usr` — User System Resources (Read-Only)

The largest directory on most systems. Contains the majority of user utilities, libraries, and documentation.

```
/usr
├── bin       → Most user commands (ls, grep, vim, python3…)
├── include   → Header files for C/C++ development
├── lib       → Libraries for /usr/bin programs
├── local     → Locally compiled/installed software
│   ├── bin   → Locally installed executables
│   ├── lib   → Locally installed libraries
│   └── share → Locally installed data files
├── sbin      → Non-essential system admin commands
├── share     → Architecture-independent data
│   ├── doc   → Documentation
│   ├── man   → Manual pages
│   └── locale→ Localization files
└── src       → Source code (optional)
```

```bash
ls /usr
ls /usr/bin | wc -l         # How many commands in /usr/bin?
ls /usr/share/doc | head -10

# Locally installed software goes here
ls /usr/local/bin 2>/dev/null
```

### `/var` — Variable Data

Contains data that changes frequently: log files, mail spools, print queues, caches, lock files.

```bash
ls /var

# Log files (very important for troubleshooting)
ls -lh /var/log/ | head -10
sudo tail -20 /var/log/syslog     # General system log
sudo tail -20 /var/log/auth.log   # Authentication events
sudo tail -20 /var/log/dpkg.log   # Package installation events

# APT cache (downloaded packages)
ls -lh /var/cache/apt/archives/ | tail -5
du -sh /var/cache/apt/

# Mail spools
ls /var/mail/ 2>/dev/null
```

---

## 4. Absolute vs Relative Paths

```bash
# ABSOLUTE PATH — starts from root /
# Always the same regardless of where you are
cd /home/john/Documents
ls /etc/passwd
cat /var/log/syslog

# RELATIVE PATH — starts from your current directory
# Changes meaning depending on where you are!
cd Documents               # Go into Documents (from home)
ls ../etc/passwd           # ../  means "up one level"
cat ../../var/log/syslog   # Up two levels, then into var/log

# Special path symbols:
# .   — current directory
# ..  — parent directory
# ~   — home directory (/home/username)
# -   — previous directory

# Examples
cp file.txt ./backup/          # Copy to backup in current dir
mv ../file.txt .               # Move file from parent dir to here
ls ~/Downloads                 # List your Downloads folder
cd -                           # Return to previous directory
```

```
ABSOLUTE vs RELATIVE — Visual Example:

Filesystem tree:          Current dir: /home/john
/
└── home
    └── john   ←── You are HERE
        ├── Documents
        │   └── report.txt
        └── Downloads

Absolute: /home/john/Documents/report.txt   (always works)
Relative: Documents/report.txt              (works from /home/john)
Relative: ./Documents/report.txt            (same as above)
Relative: ../john/Documents/report.txt      (up to /home, back in)
```

---

## 5. File Types

Linux has seven file types. The `ls -l` command shows the type in the first character:

| Symbol | Type | Example |
|---|---|---|
| `-` | Regular file | `ls`, text files, images |
| `d` | Directory | `/home`, `/etc` |
| `l` | Symbolic link | `/bin` → `/usr/bin` |
| `b` | Block device | `/dev/sda` (hard drive) |
| `c` | Character device | `/dev/tty` (terminal), `/dev/null` |
| `s` | Socket | `/run/docker.sock` |
| `p` | Named pipe (FIFO) | Inter-process communication |

```bash
# See file types with ls -l
ls -la /dev | head -20
# Look at the first character of each line

# Identify file types directly
file /bin/ls                  # ELF 64-bit LSB pie executable
file /etc/passwd              # ASCII text
file /dev/null                # character special (1/3)
file /dev/sda                 # block special (8/0)
file /tmp                     # directory

# Check if something is a symlink
ls -la /bin | head -3         # Note "bin -> usr/bin"
readlink -f /bin              # Resolve to real path

# Create different file types
mkfifo /tmp/my_pipe           # Create a named pipe
ls -la /tmp/my_pipe           # Notice: p prefix
rm /tmp/my_pipe
```

---

## 6. Hidden Files (Dot Files)

Any file or directory whose name **starts with a `.`** (period) is hidden. They are not shown by `ls` without the `-a` flag.

```bash
# Show hidden files
ls -a ~        # All files including hidden
ls -la ~       # Long format with hidden
ls -A ~        # Hidden files but not . and ..

# Common hidden files in your home directory:
ls -la ~ | grep "^\."

# ~/.bashrc         — bash configuration (aliases, functions, etc.)
# ~/.bash_history   — command history
# ~/.profile        — login shell config
# ~/.ssh/           — SSH keys and config
# ~/.gitconfig      — Git global configuration
# ~/.config/        — Application settings (XDG spec)
# ~/.local/         — User-specific data and programs
# ~/.cache/         — Application cache

# View your bash configuration
cat ~/.bashrc | head -30

# Create a hidden file
echo "my secret note" > ~/.my_hidden_note
ls -la ~ | grep my_hidden
cat ~/.my_hidden_note
rm ~/.my_hidden_note
```

---

## 7. File Naming Conventions

Linux filenames are **case-sensitive** and support almost any character, but best practice is:

```bash
# ✅ GOOD naming conventions
my_project/
web-config.txt
README.md
script_v2.sh
data_2024-01-15.csv

# ❌ AVOID these (they work but cause problems)
My Project/        # Space requires quoting everywhere
file?.txt          # ? is a shell glob wildcard
file&name.txt      # & has special meaning in shell
-filename.txt      # Leading dash looks like an option flag

# Dealing with bad filenames (if they exist):
# Use -- to signal end of options
ls -- -badname.txt
rm -- -badname.txt

# Or use ./ prefix to avoid option confusion
rm ./-badname.txt

# Handle files with spaces using quotes
touch "my file with spaces.txt"
ls "my file with spaces.txt"
rm "my file with spaces.txt"

# Or use escape character
touch my\ file\ with\ spaces.txt
ls my\ file\ with\ spaces.txt
```

### Filename Extensions

Unlike Windows, Linux does not use extensions to determine file type — it is purely convention. A file called `data.txt` could be an executable; `script.exe` could be a text file.

```bash
# The 'file' command reads content, not extension
file /bin/ls              # Not named 'ls.elf' but it's an ELF binary
file /etc/passwd          # Not named 'passwd.txt' but it's ASCII text

# Make a shell script without .sh extension
echo '#!/bin/bash' > myscript
echo 'echo "Hello"' >> myscript
chmod +x myscript
./myscript

# Check the actual type
file myscript             # Bourne-Again shell script, ASCII text executable
rm myscript
```

---

## 8. Important Configuration Files Deep Dive

### `/etc/passwd` — User Database

```bash
cat /etc/passwd | head -5
# Format: username:password:UID:GID:comment:home:shell
# root:x:0:0:root:/root:/bin/bash
#   │   │  │ │    │       │       └── Login shell
#   │   │  │ │    │       └── Home directory
#   │   │  │ │    └── GECOS (comment/full name)
#   │   │  │ └── Primary group ID
#   │   │  └── User ID
#   │   └── Password (x = stored in /etc/shadow)
#   └── Username

# Find your own entry
grep "^$(whoami):" /etc/passwd

# List all user shells in use
awk -F: '{print $7}' /etc/passwd | sort -u
```

### `/etc/fstab` — Filesystem Table

```bash
cat /etc/fstab
# Format: device  mountpoint  fstype  options  dump  pass
# UUID=abc123...  /       ext4   errors=remount-ro  0  1
# UUID=def456...  /boot/efi  vfat  umask=0077       0  1
# /swapfile        none    swap    sw                0  0

# Show current mounts vs fstab
mount | grep -E "ext4|vfat|xfs|btrfs"
```

### `/etc/hosts` — Static Host Resolution

```bash
cat /etc/hosts
# 127.0.0.1    localhost
# 127.0.1.1    my-hostname
# ::1          localhost ip6-localhost

# Add a custom hostname mapping (as root)
sudo bash -c 'echo "192.168.1.100  myserver.local myserver" >> /etc/hosts'
# Test it
ping -c1 myserver.local 2>/dev/null || echo "Host added but unreachable (that is OK)"
```

---

## Practice Exercises

### Exercise 1 — Directory Exploration
Navigate the filesystem and answer these questions using only commands:

```bash
# How many items are directly under /usr/bin?
ls /usr/bin | wc -l

# What is the total size of /var/log?
sudo du -sh /var/log

# How many configuration files are in /etc (not counting directories)?
find /etc -maxdepth 1 -type f | wc -l

# What is the newest file in /var/log?
ls -lt /var/log | head -2 | tail -1
```

### Exercise 2 — File Type Investigation
Identify the file type of each of these:

```bash
for item in /dev/null /dev/sda /bin/ls /etc/passwd /proc/cpuinfo /bin /run/systemd.pid 2>/dev/null; do
    printf "%-35s : " "$item"
    file "$item" 2>/dev/null || echo "not accessible"
done
```

### Exercise 3 — Absolute and Relative Paths
Practice path navigation:

```bash
cd /usr/share/doc

# List contents using absolute path
ls /usr/share/doc | head -3

# Go up two levels using relative path
cd ../..
pwd   # Should be /usr

# Go to /etc using both methods:
# Absolute:
cd /etc
# Relative from /usr (go up one, then into etc):
cd /usr && cd ../etc
pwd
```

### Exercise 4 — Hidden Files Exploration
Discover your hidden configuration files:

```bash
# Count hidden files in home directory
ls -la ~ | grep "^\." | wc -l

# Find all hidden directories in home
ls -la ~ | grep "^d" | grep " \."

# View your bash history
tail -10 ~/.bash_history

# See what application configs you have in ~/.config
ls ~/.config 2>/dev/null | head -10
```

### Exercise 5 — /proc Investigation
Extract system information from the /proc virtual filesystem:

```bash
# CPU info
grep "model name" /proc/cpuinfo | head -1

# Memory total
grep "MemTotal" /proc/meminfo

# System uptime in human-readable form
seconds=$(cat /proc/uptime | cut -d. -f1)
echo "Uptime: $((seconds/86400))d $((seconds%86400/3600))h $((seconds%3600/60))m"

# Number of running processes
ls /proc | grep "^[0-9]" | wc -l

# Kernel version from proc
cat /proc/version
```

---

## Common Mistakes

| Mistake | Explanation | Fix |
|---|---|---|
| Modifying `/proc` or `/sys` directly | These are virtual; changes are immediate and can affect the running kernel | Only modify known-safe entries, as root |
| Storing data in `/tmp` long-term | `/tmp` is cleared on reboot | Use `/var/tmp` for data that must survive reboots |
| Filling `/var/log` | Runaway logs can fill the disk | Set up `logrotate` and monitor disk space |
| Editing `/etc/passwd` with `vi` instead of `vipw` | `vi` has no locking mechanism | Always use `vipw` and `vigr` for safety |
| Confusing `/bin` and `/usr/bin` | On modern systems they are the same symlinked dir | Use `which` to find the real path |
| Forgetting `sudo` for `/etc` edits | Most config files are root-owned | Always use `sudo` or `sudo -e` |

---

## Pro Tips

> 💡 **`find / -name "filename" 2>/dev/null`** — search the entire filesystem for a file, suppressing permission errors.

> 💡 **`stat` gives full file metadata** — permissions, owner, timestamps, inode number, and more.

```bash
stat /etc/passwd
```

> 💡 **`lsof +D /directory`** — list all open files in a directory. Useful when you cannot unmount a filesystem.

> 💡 **`inode numbers` are the true file identity** — the filename is just a pointer. Two filenames can point to the same inode (hard links).

```bash
ls -i /etc/passwd    # Show inode number
```

> 💡 **`/proc/sys/` is tunable** — Many kernel parameters can be changed at runtime via `/proc/sys/`, made permanent via `/etc/sysctl.conf`.

---

## Key Takeaways

- Linux uses a single unified tree rooted at `/` — no drive letters
- The FHS defines what belongs where, making the structure consistent across all distros
- `/etc` = configuration, `/var` = variable data, `/proc` = virtual kernel data, `/home` = user files
- Absolute paths start with `/`; relative paths depend on your current location
- Seven file types: regular, directory, symlink, block, character, socket, pipe
- Dot files (`.bashrc`, `.ssh/`) are hidden configuration files in your home directory
- Linux ignores file extensions; the `file` command reads actual content

---

## Next Lesson Preview

**06 — File and Directory Management**

Now that you know where files live, learn to master them: view files with `cat`, `less`, `head`, and `tail`; search with `find` and `grep`; transform text with `sed`, `awk`, and `cut`; archive with `tar` and `gzip`; and control who can read, write, or execute files with `chmod`, `chown`, and Linux permissions.
