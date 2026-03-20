# 05 — File System Structure

> **Module 1 · Lesson 5** | Difficulty: ★★☆☆☆ Beginner-Intermediate | Time: ~60 min

---

## Learning Objectives

- Understand the Linux Filesystem Hierarchy Standard (FHS)
- Know the purpose of each top-level directory
- Navigate the /proc and /sys virtual filesystems
- Understand absolute vs relative paths
- Work with hidden files and symbolic links
- Understand inodes and hard links

---

## Table of Contents

1. [Everything Is a File](#1-everything-is-a-file)
2. [The Filesystem Hierarchy Standard](#2-the-filesystem-hierarchy-standard)
3. [Top-Level Directories](#3-top-level-directories)
4. [Virtual Filesystems](#4-virtual-filesystems)
5. [Paths: Absolute vs Relative](#5-paths-absolute-vs-relative)
6. [Hidden Files and Dotfiles](#6-hidden-files-and-dotfiles)
7. [Links: Symbolic and Hard](#7-links-symbolic-and-hard)
8. [Inodes and the Filesystem](#8-inodes-and-the-filesystem)
9. [Finding Files](#9-finding-files)
10. [Practice Exercises](#10-practice-exercises)
11. [Key Takeaways](#11-key-takeaways)

---

## 1. Everything Is a File

Linux inherits from Unix the principle that **everything is a file** (or behaves like one):

| Type | Example | Accessed With |
|------|---------|--------------|
| Regular file | `/etc/passwd` | cat, vim, cp |
| Directory | `/home/alice/` | ls, cd |
| Symbolic link | `/bin → /usr/bin` | ls -l shows `->` |
| Block device | `/dev/sda` | fdisk, dd |
| Character device | `/dev/tty` | echo, cat |
| Socket | `/run/docker.sock` | programs connect |
| Named pipe (FIFO) | `/tmp/mypipe` | mkfifo |
| Pseudo-file | `/proc/cpuinfo` | cat, read |

This abstraction means you can use the same tools (`cat`, `echo`, `read`, `write`) for files, hardware, and processes.

---

## 2. The Filesystem Hierarchy Standard

The **FHS** (Filesystem Hierarchy Standard) defines where files go on a Linux system. It ensures consistency across distributions.

```
/                          Root directory (everything starts here)
├── bin  → usr/bin         Essential user command binaries
├── boot                   Boot loader files and kernel
├── dev                    Device files
├── etc                    System configuration files
├── home                   User home directories
│   ├── alice/
│   └── bob/
├── lib  → usr/lib         Essential shared libraries
├── lib64 → usr/lib        64-bit libraries
├── media                  Mount points for removable media
├── mnt                    Temporary mount points
├── opt                    Optional/third-party software
├── proc                   Virtual filesystem (process/kernel info)
├── root                   Home directory for root user
├── run                    Runtime data (PIDs, sockets)
├── sbin → usr/sbin        System administration binaries
├── srv                    Data for services (www, ftp)
├── sys                    Virtual filesystem (hardware/kernel)
├── tmp                    Temporary files (cleared on reboot)
├── usr                    Secondary hierarchy (read-only)
│   ├── bin                User commands (most of your tools)
│   ├── include            C header files
│   ├── lib                Libraries
│   ├── local              Locally installed software
│   ├── sbin               Non-essential system binaries
│   └── share              Architecture-independent data
└── var                    Variable data (logs, spools, databases)
    ├── cache              Application cache
    ├── lib                Persistent application data
    ├── log                Log files
    ├── mail               Mail spools
    ├── run                PID files and sockets
    ├── spool              Print/mail spools
    └── tmp                Temporary files preserved between reboots
```

---

## 3. Top-Level Directories

### `/` — Root Directory

```bash
ls /
# bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

All paths start here. Only root can write to most directories.

### `/home` — User Home Directories

```bash
ls /home/
# alice  bob  carol

echo $HOME            # your home: /home/alice
ls ~                  # your home directory
ls ~alice             # another user's home (if readable)
```

Contains user personal files, configs, and data. Protected from other users.

### `/etc` — Configuration Files

"Editable Text Configuration" — all system config files.

```bash
ls /etc/ | head -20
cat /etc/hostname           # system hostname
cat /etc/hosts              # local DNS overrides
cat /etc/fstab              # filesystem mount table
cat /etc/os-release         # OS version info
cat /etc/passwd             # user account info (NOT passwords)
cat /etc/shadow             # hashed passwords (root only)
cat /etc/group              # group definitions
cat /etc/crontab            # system cron jobs
ls /etc/ssh/                # SSH configuration
ls /etc/apt/                # APT package manager config
ls /etc/network/            # network configuration
```

### `/var` — Variable Data

Data that changes frequently:

```bash
ls /var/
ls /var/log/                # all log files
tail -f /var/log/syslog     # watch system log in real-time
tail -f /var/log/auth.log   # authentication events
cat /var/log/dpkg.log       # package installation history
ls /var/lib/apt/            # APT package cache
ls /var/www/                # web server files (if installed)
```

### `/usr` — User System Resources

Secondary hierarchy (most programs live here):

```bash
ls /usr/bin | head -20     # user commands (python3, gcc, git...)
ls /usr/lib | head -10     # shared libraries
ls /usr/share/doc | head   # documentation
ls /usr/local/bin          # locally installed programs
ls /usr/include            # C header files
```

### `/bin`, `/sbin`, `/lib` — Essential Binaries

In modern Ubuntu, these are **symlinks** to `/usr/bin`, `/usr/sbin`, `/usr/lib`:

```bash
ls -la /bin    # lrwxrwxrwx 1 root root 7 → usr/bin
ls -la /sbin   # lrwxrwxrwx 1 root root 8 → usr/sbin
ls -la /lib    # lrwxrwxrwx 1 root root 7 → usr/lib
```

These binaries must be available even if `/usr` isn't mounted:
- `/bin`: cat, ls, cp, mv, rm, mkdir, bash, sh
- `/sbin`: fdisk, fsck, ip, iptables, mount, init

### `/boot` — Boot Files

```bash
ls /boot/
# config-5.15.0-88-generic     kernel config
# grub/                         GRUB bootloader config
# initrd.img-5.15.0-88-generic  initial RAM disk
# vmlinuz-5.15.0-88-generic     compressed kernel

cat /boot/grub/grub.cfg | head -40  # bootloader config
```

### `/dev` — Device Files

```bash
ls /dev/
# Block devices (disks):
ls /dev/sd*             # SATA/SCSI disks
ls /dev/nvme*           # NVMe SSDs
ls /dev/vd*             # Virtual disks (VMs)

# Character devices:
ls /dev/tty*            # terminals
ls /dev/null            # discard output
ls /dev/zero            # infinite zeros
ls /dev/random          # random data
ls /dev/urandom         # non-blocking random

# Useful examples:
dd if=/dev/zero of=empty.img bs=1M count=10  # create 10MB empty file
cat /dev/urandom | head -c 16 | xxd          # 16 random bytes
echo "test" > /dev/null                      # discard output
```

### `/tmp` — Temporary Files

```bash
ls /tmp/                # often contains runtime files
# Cleared on every reboot
# World-writable (anyone can create files)
# Watch: sticky bit prevents deletion by others

ls -la / | grep tmp
# drwxrwxrwt 14 root root ... tmp
#          ^
#          Sticky bit (t) = only owner can delete their own files
```

### `/opt` — Optional Software

Third-party applications not in package manager:

```bash
ls /opt/
# google/           (Google Chrome)
# sublime_text/     (Sublime Text editor)
# teamviewer/       (TeamViewer)
```

### `/root` — Root User's Home

```bash
# Home directory for the superuser (NOT /home/root)
sudo ls /root/         # only accessible with sudo
sudo ls -la /root/
```

---

## 4. Virtual Filesystems

These directories don't contain real files — they're windows into the kernel:

### `/proc` — Process and Kernel Information

```bash
ls /proc/
# Numbers = PIDs (running processes)
# Files = kernel and process info

# System information
cat /proc/cpuinfo           # CPU details
cat /proc/meminfo           # memory details
cat /proc/version           # kernel version
cat /proc/uptime            # system uptime (seconds)
cat /proc/loadavg           # system load
cat /proc/partitions        # disk partitions
cat /proc/mounts            # mounted filesystems
cat /proc/net/tcp           # active TCP connections

# Process information
cat /proc/1/status          # init process (PID 1)
cat /proc/$$/cmdline        # current shell command
ls /proc/$$/fd/             # file descriptors for current shell
cat /proc/$$/maps           # memory map of current shell

# Kernel tunables (readable and writable!)
cat /proc/sys/net/ipv4/ip_forward     # IP forwarding
cat /proc/sys/vm/swappiness           # swap tendency
cat /proc/sys/kernel/hostname         # hostname
```

### `/sys` — Hardware and Kernel Interface

```bash
ls /sys/
# class/  block/  bus/  devices/  ...

# Network interfaces
ls /sys/class/net/
cat /sys/class/net/eth0/speed     # link speed
cat /sys/class/net/eth0/address   # MAC address
cat /sys/class/net/eth0/operstate # up or down?

# CPU frequency scaling
ls /sys/devices/system/cpu/cpu0/cpufreq/
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq

# Power/battery (laptop)
ls /sys/class/power_supply/
cat /sys/class/power_supply/BAT0/capacity  # battery %
```

### `/run` — Runtime Data

```bash
ls /run/
# Created fresh on each boot
# Contains: PIDs, sockets, locks

ls /run/*.pid               # process ID files
ls /run/systemd/            # systemd socket files
ls /run/user/1000/          # user runtime directory
```

---

## 5. Paths: Absolute vs Relative

### Absolute Paths

Start from root `/` — always works regardless of current directory:

```bash
cd /home/alice/Documents    # absolute
cat /etc/hosts              # absolute
ls /var/log/syslog          # absolute
```

### Relative Paths

Relative to your current directory:

```bash
# If you're in /home/alice:
cd Documents               # relative (same as /home/alice/Documents)
cat ../bob/readme.txt      # up one level, then into bob's dir
ls ../../etc               # up two levels, then etc
```

### Special Path Symbols

| Symbol | Meaning | Example |
|--------|---------|---------|
| `.` | Current directory | `ls .` |
| `..` | Parent directory | `cd ..` |
| `~` | Home directory | `cd ~/Downloads` |
| `-` | Previous directory | `cd -` |
| `*` | Wildcard (any chars) | `ls *.txt` |
| `?` | Wildcard (one char) | `ls file?.txt` |
| `[abc]` | Character class | `ls file[123].txt` |
| `{a,b}` | Brace expansion | `mkdir {src,tests,docs}` |

### Path Examples

```bash
# You're in: /home/alice

ls ./Documents          # → /home/alice/Documents
ls ../bob               # → /home/bob
ls ../../var/log        # → /var/log
ls ~/Downloads          # → /home/alice/Downloads (always)
ls ~bob/Documents       # → /home/bob/Documents
```

---

## 6. Hidden Files and Dotfiles

Files starting with `.` are **hidden** — `ls` doesn't show them by default.

```bash
ls ~                    # normal files
ls -a ~                 # show hidden files
ls -la ~                # hidden + long format

# Common hidden files/dirs in home:
~/.bashrc              # bash configuration
~/.bash_history        # command history
~/.ssh/                # SSH keys and config
~/.gitconfig           # git configuration
~/.config/             # application configs (XDG standard)
~/.local/              # local user data and binaries
~/.cache/              # application caches
~/.profile             # login shell profile
```

### Creating and Working with Hidden Files

```bash
# Create hidden file
touch .secret_file
echo "password123" > .credentials    # BAD practice — but demonstrates the concept

# Show hidden files
ls -A    # -A shows hidden but NOT . and ..
ls -a    # shows hidden INCLUDING . and ..

# Dotfiles for configuration
echo 'alias ll="ls -alF"' >> ~/.bashrc   # add alias
source ~/.bashrc                          # reload config
```

---

## 7. Links: Symbolic and Hard

### Symbolic Links (Symlinks)

A symlink is a pointer to another file or directory:

```bash
# Create symbolic link
ln -s /target/path /link/path
ln -s /usr/bin/python3 ~/mypy     # link in home
ln -s /var/log /tmp/logs          # link to directory

# View symlinks
ls -la /bin        # lrwxrwxrwx ... bin -> usr/bin
file /bin          # /bin: symbolic link to usr/bin
readlink /bin      # usr/bin
readlink -f /bin   # /usr/bin (fully resolved)

# Dangling symlink (target doesn't exist)
ln -s /nonexistent /tmp/broken
ls -la /tmp/broken    # shows as broken (red in color terminals)
```

### Hard Links

A hard link is another name for the same inode (same file data):

```bash
# Create hard link
ln original.txt hardlink.txt

# Both point to the same data
ls -li original.txt hardlink.txt
# Shows same inode number!

echo "Modified via hardlink" >> hardlink.txt
cat original.txt    # Shows the modification!

# Deleting one doesn't delete the other
rm hardlink.txt
cat original.txt    # Still exists!

# Hard links cannot span filesystems
# Hard links cannot point to directories (usually)
```

### Symlinks vs Hard Links

| Feature | Symlink | Hard Link |
|---------|---------|-----------|
| Different filesystem | ✅ Yes | ❌ No |
| Point to directory | ✅ Yes | ❌ No (usually) |
| If target deleted | ❌ Becomes broken | ✅ File persists |
| Inode | Different | Same |
| Size | Small (path length) | Same as original |
| `ls -l` shows | `->` target | Nothing special |

---

## 8. Inodes and the Filesystem

### What Is an Inode?

An **inode** (index node) stores metadata about a file:

```
Inode Table
┌────────────┬────────────────────────────┐
│ Inode #    │ Metadata                   │
├────────────┼────────────────────────────┤
│ 131074     │ File type: regular file    │
│            │ Permissions: rw-r--r--     │
│            │ Links: 1                   │
│            │ Owner UID: 1000            │
│            │ Owner GID: 1000            │
│            │ Size: 2048 bytes           │
│            │ atime: 2024-01-15 10:23    │
│            │ mtime: 2024-01-14 09:00    │
│            │ ctime: 2024-01-14 09:00    │
│            │ Data block pointers        │
└────────────┴────────────────────────────┘
```

An inode does **NOT** contain the filename — that's stored in the directory.

```bash
# View inode information
ls -li file.txt         # shows inode number
stat file.txt           # detailed inode info
stat /etc/passwd

# View inode usage
df -i                   # inode usage per filesystem
```

### File Timestamps (atime, mtime, ctime)

| Timestamp | Updated When | Can Be Forged? |
|-----------|-------------|----------------|
| **atime** | File is read/accessed | Yes (touch -a) |
| **mtime** | File content modified | Yes (touch -m) |
| **ctime** | Inode changed (perms, links) | No |

```bash
stat file.txt
# Access: 2024-01-15 10:23:45  (atime)
# Modify: 2024-01-14 09:00:12  (mtime)
# Change: 2024-01-14 09:00:12  (ctime)

# Fake timestamps (useful for scripts; abused by malware)
touch -t 202001010000 file.txt   # set to 2020-01-01 00:00
```

---

## 9. Finding Files

### `find` Command

```bash
# Basic find
find /home -name "*.txt"                    # find .txt files
find /etc -name "sshd_config"               # find specific file
find / -name "passwd" 2>/dev/null           # search everywhere

# By type
find /home -type f -name "*.sh"             # files only
find /var -type d -name "log*"              # directories only
find /tmp -type l                           # symbolic links only

# By size
find /var/log -size +10M                    # larger than 10MB
find /tmp -size -1k                         # smaller than 1KB
find / -size +100M -size -1G 2>/dev/null   # between 100MB and 1GB

# By time
find /home -mtime -7                        # modified in last 7 days
find /tmp -atime +30                        # accessed 30+ days ago
find /etc -newer /etc/passwd                # newer than passwd

# By permissions
find / -perm 777 2>/dev/null               # world-writable
find / -perm -4000 2>/dev/null             # SUID files (security check!)
find / -perm -2000 2>/dev/null             # SGID files

# By owner
find /home -user alice                      # owned by alice
find /var -group www-data                   # owned by www-data

# Execute commands on results
find /tmp -name "*.tmp" -delete            # delete .tmp files
find /home -name "*.txt" -exec cat {} \;   # print each .txt file
find /home -name "*.log" -exec ls -lh {} +  # list each .log file
```

### `locate` Command

```bash
# Fast search using database index
locate passwd               # find all files named passwd
locate -i README            # case-insensitive
locate "*.conf" | grep ssh  # find ssh config files

# Update database first
sudo updatedb               # rebuild locate database

# locate vs find:
# locate = fast (pre-built index, may be stale)
# find = slow but always current, more options
```

### `which` and `whereis`

```bash
which python3       # /usr/bin/python3
which -a python     # find all versions in PATH
whereis python3     # binary + man pages + source
whereis -b python3  # binary locations only
type python3        # "python3 is /usr/bin/python3"
```

---

## 10. Practice Exercises

### Exercise 5.1 — Directory Exploration

```bash
# 1. List all contents of / including hidden items
# 2. Find the 5 largest directories in /var
du -h /var/* 2>/dev/null | sort -rh | head -5

# 3. Count total files in /etc
find /etc -type f | wc -l

# 4. Find all .conf files in /etc
find /etc -name "*.conf" | head -10

# 5. Explore /proc
cat /proc/cpuinfo | grep "model name" | head -1
cat /proc/meminfo | grep MemTotal
```

### Exercise 5.2 — Symlinks

```bash
# 1. Create a symlink called 'logs' pointing to /var/log
ln -s /var/log ~/logs
ls ~/logs

# 2. Verify it's a symlink
ls -la ~/logs
file ~/logs
readlink ~/logs

# 3. Create a hard link and verify same inode
touch original.txt
ln original.txt hardlink.txt
ls -li original.txt hardlink.txt

# 4. Delete original, verify hardlink still works
rm original.txt
cat hardlink.txt    # should still work
```

### Exercise 5.3 — Finding Files

```bash
# 1. Find all SUID binaries (potential security check)
find /usr -perm -4000 2>/dev/null | sort

# 2. Find files modified in the last hour
find /tmp -mmin -60 2>/dev/null

# 3. Find the largest files in /var/log
find /var/log -type f -exec ls -lh {} + 2>/dev/null | sort -k5 -rh | head -5

# 4. Find all world-writable files in /tmp
find /tmp -perm -o+w -type f 2>/dev/null
```

### Exercise 5.4 — /proc Exploration

```bash
# 1. Find your shell's PID
echo $$
ls /proc/$$

# 2. Read your process's command line
cat /proc/$$/cmdline | tr '\0' ' '

# 3. View open file descriptors
ls -la /proc/$$/fd

# 4. Check system memory (detailed)
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable"
```

---

## 11. Key Takeaways

- The **FHS** provides a consistent directory structure across all Linux distros
- `/etc` = configuration; `/var` = variable data; `/usr` = user programs; `/home` = user files
- `/proc` and `/sys` are **virtual** filesystems — windows into the kernel
- **Absolute paths** start with `/`; **relative paths** start from current directory
- **Hidden files** start with `.` and are shown with `ls -a`
- **Symlinks** point to other files; **hard links** are alternate names for the same inode
- **Inodes** store file metadata (permissions, size, timestamps) — not the filename
- Use `find` for powerful file searching; `locate` for fast indexed searching

---

## Next Lesson

➡️ [06 — File and Directory Management](06_File_and_Directory_Management.md)

---

*Module 1 · Lesson 5 of 6 | [Course Index](../INDEX.md)*
