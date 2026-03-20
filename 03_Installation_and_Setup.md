# 03 — Installation and Setup

> **Difficulty:** Beginner | **Time Estimate:** 60–90 minutes

---

## Learning Objectives

By the end of this lesson you will be able to:

- Check if your hardware meets Ubuntu's requirements
- Download an Ubuntu ISO and verify its integrity
- Create a bootable USB drive using `dd` or a GUI tool
- Navigate BIOS/UEFI to boot from USB
- Partition a disk appropriately for a Linux installation
- Complete the Ubuntu installation wizard
- Perform essential post-installation setup
- Install and configure a virtual machine for safe experimentation

---

## 1. System Requirements

### Minimum Requirements (Ubuntu 24.04 LTS)

| Component | Minimum | Recommended |
|---|---|---|
| **CPU** | 2 GHz dual-core | 4+ core, 64-bit |
| **RAM** | 4 GB | 8 GB+ |
| **Disk** | 25 GB | 50 GB+ |
| **Display** | 1024×768 | 1920×1080 |
| **USB/DVD** | For installation | — |
| **Internet** | Optional | Recommended |

```bash
# Check existing hardware (run on any Linux system)
# CPU cores and model
nproc
cat /proc/cpuinfo | grep "model name" | head -1

# Total RAM
free -h

# Disk space available
df -h /

# Check if CPU supports 64-bit (look for lm flag = long mode)
grep -o 'lm' /proc/cpuinfo | head -1
```

---

## 2. Downloading Ubuntu

### Step 1 — Download the ISO

Visit **ubuntu.com/download/desktop** and download the latest LTS release (e.g., Ubuntu 24.04 LTS).

```bash
# Download from command line using wget
wget https://releases.ubuntu.com/24.04/ubuntu-24.04-desktop-amd64.iso

# Or with curl
curl -L -o ubuntu-24.04-desktop-amd64.iso \
  https://releases.ubuntu.com/24.04/ubuntu-24.04-desktop-amd64.iso

# Check download progress with a status bar
wget --progress=bar:force \
  https://releases.ubuntu.com/24.04/ubuntu-24.04-desktop-amd64.iso
```

### Step 2 — Verify the ISO (Critical Security Step)

Always verify the ISO hash to ensure it was not corrupted or tampered with.

```bash
# Download the SHA256 checksum file
wget https://releases.ubuntu.com/24.04/SHA256SUMS

# Download the signature file
wget https://releases.ubuntu.com/24.04/SHA256SUMS.gpg

# Verify the ISO hash matches
sha256sum -c SHA256SUMS --ignore-missing

# Example expected output:
# ubuntu-24.04-desktop-amd64.iso: OK

# Manual verification
sha256sum ubuntu-24.04-desktop-amd64.iso
# Compare output with the expected hash in SHA256SUMS

# GPG verification (advanced)
gpg --keyid-format long --verify SHA256SUMS.gpg SHA256SUMS
```

---

## 3. Creating a Bootable USB Drive

### Method 1 — Using `dd` (Linux/macOS)

`dd` is the classic Unix data duplicator. It copies raw bytes from source to destination.

> ⚠️ **WARNING:** `dd` is sometimes called "disk destroyer" because a wrong target device will overwrite data irreversibly. Triple-check your device name.

```bash
# Step 1: Identify your USB drive device name
lsblk

# Sample output:
# NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
# sda      8:0    0 500.1G  0 disk
# ├─sda1   8:1    0   512M  0 part /boot/efi
# └─sda2   8:2    0 499.6G  0 part /
# sdb      8:16   1   7.5G  0 disk       ← USB drive (8GB, no partitions)

# Step 2: Unmount the USB (if auto-mounted)
sudo umount /dev/sdb1 2>/dev/null || true

# Step 3: Write the ISO to the USB drive
# bs=4M sets block size to 4 megabytes (faster)
# status=progress shows live progress
# conv=fsync ensures data is flushed before exit
sudo dd if=ubuntu-24.04-desktop-amd64.iso \
        of=/dev/sdb \
        bs=4M \
        status=progress \
        conv=fsync

# Step 4: Ensure all data is written
sync

# Expected output during write:
# 1234567168 bytes (1.2 GB, 1.1 GiB) copied, 45 s, 27.4 MB/s
```

### Method 2 — Using `cp` (Simpler, Linux only)

```bash
# On modern Linux, a simple cp can work for hybrid ISOs
sudo cp ubuntu-24.04-desktop-amd64.iso /dev/sdb
sync
```

### Method 3 — GUI Tools

| Tool | OS | Notes |
|---|---|---|
| **Balena Etcher** | Linux/macOS/Windows | Recommended for beginners |
| **Rufus** | Windows only | Excellent Windows tool |
| **GNOME Disks** | Linux | Built into Ubuntu |
| **Startup Disk Creator** | Ubuntu | Built-in, simple |

```bash
# Install Balena Etcher on Ubuntu
# Download AppImage from balena.io/etcher
chmod +x balenaEtcher-*.AppImage
./balenaEtcher-*.AppImage

# Install GNOME Disks (usually pre-installed)
sudo apt install gnome-disk-utility -y
gnome-disks
```

---

## 4. BIOS/UEFI Settings

### BIOS vs UEFI

```
BIOS (Basic Input/Output System)        UEFI (Unified Extensible Firmware Interface)
───────────────────────────────         ──────────────────────────────────────────
• Legacy system (pre-2010)              • Modern standard (post-2010)
• 16-bit, runs in real mode             • 64-bit, supports large disks
• Limited to 2 TB disks                 • Supports disks > 2 TB
• MBR partition table                   • GPT partition table preferred
• Text-only interface                   • Often has GUI interface
• No Secure Boot                        • Secure Boot (may need to disable)
```

### Accessing BIOS/UEFI

Press the correct key during system POST (Power-On Self-Test):

| Manufacturer | Key(s) |
|---|---|
| Dell | F2 or F12 |
| HP | F1, F2, F10, or Esc |
| Lenovo | F1, F2, or Fn+F2 |
| ASUS | Delete or F2 |
| Acer | Delete or F2 |
| MSI | Delete |
| Gigabyte | Delete or F2 |
| Virtual Box | F12 |

### Key BIOS Settings for Linux Installation

1. **Boot Order** — Move USB drive to first position
2. **Secure Boot** — Disable if Ubuntu installer fails to load
3. **Fast Boot** — Disable to allow USB boot detection
4. **SATA Mode** — Set to AHCI (not RAID or IDE) for best Linux compatibility

---

## 5. Partitioning Strategies

### Option A — Simple (Entire Disk)

For a dedicated Linux machine, use the automatic partitioner. It creates:

```
┌────────────────────────────────────┐
│  EFI System Partition (512 MB)     │  /boot/efi — bootloader files
├────────────────────────────────────┤
│  Root Partition (remaining space)  │  / — entire system
└────────────────────────────────────┘
```

### Option B — Recommended Manual Layout

```
┌────────────────────────────────────┐
│  EFI System Partition   512 MB     │  /boot/efi  (FAT32)
├────────────────────────────────────┤
│  Boot Partition         1 GB       │  /boot      (ext4)
├────────────────────────────────────┤
│  Swap Partition         = RAM size │  swap
├────────────────────────────────────┤
│  Root Partition         30 GB      │  /          (ext4)
├────────────────────────────────────┤
│  Home Partition         Remaining  │  /home      (ext4)
└────────────────────────────────────┘
```

**Why separate `/home`?**
If you ever need to reinstall Linux, your personal files in `/home` survive on a separate partition.

```bash
# After installation: view your partition layout
lsblk -f

# See detailed partition table
sudo fdisk -l /dev/sda

# View filesystem information
sudo blkid

# See mount points and filesystem usage
df -hT
```

### Swap Size Recommendations

| RAM | Swap (No Hibernation) | Swap (With Hibernation) |
|---|---|---|
| 2 GB | 2 GB | 4 GB |
| 4 GB | 2 GB | 8 GB |
| 8 GB | 2–4 GB | 16 GB |
| 16 GB | 4–8 GB | 32 GB |
| 32 GB+ | 8 GB | 64 GB |

---

## 6. Installation Walkthrough

### Step-by-Step

```
1. Boot from USB
   └─► Select "Try or Install Ubuntu"

2. Welcome Screen
   └─► Choose language → Click "Install Ubuntu"

3. Keyboard Layout
   └─► Select your keyboard layout and variant

4. Installation Type
   └─► Choose "Normal Installation" or "Minimal Installation"
   └─► Check "Download updates during installation"
   └─► Check "Install third-party software" (for codecs/drivers)

5. Installation Type (Disk)
   └─► "Erase disk and install Ubuntu" (simplest)
   └─► OR "Something else" (manual partitioning)

6. Timezone
   └─► Select your city/timezone

7. User Setup
   └─► Your name: Full Name
   └─► Computer name: hostname (e.g., my-ubuntu-pc)
   └─► Username: lowercase, no spaces (e.g., john)
   └─► Password: Use a strong password
   └─► Choose "Require password to log in"

8. Installation Proceeds
   └─► Wait 10-20 minutes

9. Restart
   └─► Remove USB when prompted
   └─► Press Enter
```

---

## 7. Post-Installation Setup

### 7.1 System Update

**Always update immediately after installation.**

```bash
# Update package index
sudo apt update

# Upgrade all installed packages
sudo apt upgrade -y

# Upgrade the distribution (kernel, etc.)
sudo apt full-upgrade -y

# Remove unneeded packages
sudo apt autoremove -y

# Clean package cache
sudo apt autoclean
```

### 7.2 Install Essential Software

```bash
# Build essentials (gcc, make, etc.)
sudo apt install build-essential -y

# Git version control
sudo apt install git -y

# Text editors
sudo apt install vim nano -y

# Network tools
sudo apt install curl wget net-tools -y

# System utilities
sudo apt install htop tree file unzip -y

# Python 3 and pip
sudo apt install python3 python3-pip python3-venv -y

# Multiple packages at once
sudo apt install -y \
  build-essential \
  git \
  vim \
  curl \
  wget \
  htop \
  tree \
  net-tools \
  python3 \
  python3-pip \
  unzip \
  zip
```

### 7.3 Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "vim"
git config --global init.defaultBranch main

# Verify configuration
git config --list
```

### 7.4 SSH Setup

```bash
# Install SSH server
sudo apt install openssh-server -y

# Start and enable SSH
sudo systemctl start ssh
sudo systemctl enable ssh

# Check SSH status
sudo systemctl status ssh

# Generate SSH key pair
ssh-keygen -t ed25519 -C "your_email@example.com"

# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your key to the agent
ssh-add ~/.ssh/id_ed25519

# View your public key (to add to GitHub, servers, etc.)
cat ~/.ssh/id_ed25519.pub

# Test SSH connection to GitHub
ssh -T git@github.com
```

### 7.5 Firewall Setup

```bash
# UFW (Uncomplicated Firewall) is included with Ubuntu
# Enable UFW
sudo ufw enable

# Allow SSH (important — do this BEFORE enabling if connecting remotely)
sudo ufw allow ssh

# Allow common services
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS

# Check firewall status
sudo ufw status verbose

# See all rules with numbers
sudo ufw status numbered
```

### 7.6 Hostname Configuration

```bash
# View current hostname
hostname

# Change hostname (survives reboot)
sudo hostnamectl set-hostname my-new-hostname

# Verify change
hostnamectl

# Update /etc/hosts to match
sudo nano /etc/hosts
# Change: 127.0.1.1  old-hostname
# To:    127.0.1.1  my-new-hostname
```

---

## 8. Virtual Machine Setup

If you want to try Linux without installing it on your main machine, use a virtual machine (VM).

### VirtualBox Setup

```bash
# Install VirtualBox on an existing Ubuntu system
sudo apt install virtualbox virtualbox-ext-pack -y

# Or download from virtualbox.org for Windows/macOS host

# After creating a VM, install Guest Additions for better integration
# Inside the VM:
sudo apt install virtualbox-guest-additions-iso -y
sudo apt install virtualbox-guest-utils -y
```

### Recommended VM Settings for Ubuntu

```
┌─────────────────────────────────────────┐
│         VirtualBox VM Settings          │
│─────────────────────────────────────────│
│  Type:      Linux                       │
│  Version:   Ubuntu (64-bit)             │
│  RAM:       4096 MB (4 GB minimum)      │
│  CPU:       2 cores                     │
│  Video:     128 MB VRAM                 │
│  Storage:   25 GB (dynamic VDI)         │
│  Network:   NAT (for internet access)   │
│  Display:   3D Acceleration ON          │
└─────────────────────────────────────────┘
```

### VMware Workstation Player (Alternative)

```bash
# Download VMware Player from vmware.com/go/getplayer-linux
chmod +x VMware-Player-*.bundle
sudo ./VMware-Player-*.bundle

# Install VMware tools inside Ubuntu VM
sudo apt install open-vm-tools open-vm-tools-desktop -y
```

---

## 9. Useful Post-Install Checks

```bash
# System information summary
uname -a
lsb_release -a
hostnamectl

# Hardware overview
lshw -short 2>/dev/null | head -30

# Check which drivers are loaded
lsmod | head -20

# See all mounted filesystems
mount | grep -E "ext4|xfs|btrfs|vfat"

# Check battery (laptops)
upower -i /org/freedesktop/UPower/devices/battery_BAT0 2>/dev/null

# Check disk health with SMART
sudo apt install smartmontools -y
sudo smartctl -a /dev/sda

# See systemd boot time breakdown
systemd-analyze
systemd-analyze blame | head -10
```

---

## Practice Exercises

### Exercise 1 — Verify a Download
Download any small file and verify its SHA256 hash:

```bash
# Download a small test file
wget https://releases.ubuntu.com/24.04/SHA256SUMS -O /tmp/SHA256SUMS_test

# Generate the hash of the downloaded file
sha256sum /tmp/SHA256SUMS_test

# Note the hash — this is how you would verify against a known-good value
```

### Exercise 2 — Explore Your Partitions
After installation, map your disk layout:

```bash
lsblk -f
df -hT
cat /proc/mounts | grep -v "^proc\|^sys\|^dev\|^run" | head -10
```

**Question:** What filesystem type is your root partition using?

### Exercise 3 — Update and Audit
Perform a full system update and check how many packages were upgraded:

```bash
sudo apt update 2>&1 | grep "packages can be upgraded"
sudo apt upgrade -y 2>&1 | tail -5
```

### Exercise 4 — SSH Key Generation
Generate an SSH key pair and examine both keys:

```bash
ssh-keygen -t ed25519 -C "learning@example.com" -f /tmp/test_key -N ""
echo "=== Private Key (first 5 lines) ==="
head -5 /tmp/test_key
echo "=== Public Key ==="
cat /tmp/test_key.pub
rm /tmp/test_key /tmp/test_key.pub
```

### Exercise 5 — Service Management
Explore services that start at boot:

```bash
# List all active services
systemctl list-units --type=service --state=active | head -20

# Check a specific service
systemctl status NetworkManager

# See what is enabled to start at boot
systemctl list-unit-files --type=service --state=enabled | head -15
```

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---|---|---|
| Not verifying ISO hash | May install corrupted/tampered OS | Always run `sha256sum -c` |
| Wrong `dd` target device | Data loss on wrong disk | Run `lsblk` and triple-check device name |
| Skipping post-install update | Security vulnerabilities | Run `apt update && apt upgrade` immediately |
| No separate `/home` partition | Reinstall wipes personal data | Use manual partitioning with separate `/home` |
| Disabling Secure Boot and forgetting | Reduced security | Re-enable after confirming Linux boots |
| Too little swap | System crashes under memory pressure | Follow swap size table above |
| Not configuring SSH keys | Password-only auth is less secure | Set up key-based authentication |

---

## Pro Tips

> 💡 **Use LTS releases for stability.** Ubuntu LTS (Long-Term Support) releases are supported for 5 years. For servers, prefer LTS over regular releases.

> 💡 **`timeshift` for system snapshots.** Install `timeshift` immediately after setup to create restore points before major changes.

```bash
sudo apt install timeshift -y
sudo timeshift --create --comments "Fresh install"
```

> 💡 **`aptitude` for complex dependency resolution.** When `apt` cannot resolve conflicts, `aptitude` often finds a solution.

> 💡 **Keep a live USB.** Your bootable USB also functions as a recovery tool if Linux fails to boot.

> 💡 **`/etc/apt/sources.list.d/`** is where additional software repositories go. Always use `apt-key` or signed repositories to avoid untrusted sources.

---

## Key Takeaways

- Always verify ISO integrity with SHA256 before installation
- `dd` writes disk images byte-for-byte — verify the target device carefully
- UEFI with GPT is the modern standard; BIOS/MBR is legacy
- A separate `/home` partition protects your data during reinstalls
- Post-install: update, configure SSH, set up firewall, install essentials
- Virtual machines are ideal for learning without risking your main system
- `systemctl` manages services that start on boot

---

## Next Lesson Preview

**04 — The Terminal Basics**

The terminal is where Linux's true power lives. We will cover the difference between terminal, shell, console and TTY, master 20+ essential commands, learn keyboard shortcuts that will make you 10× faster, and understand how standard input, output, and error streams work — the foundation of everything in Linux.
