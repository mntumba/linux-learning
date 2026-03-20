# 03 — Installation and Setup

> **Module 1 · Lesson 3** | Difficulty: ★★☆☆☆ Beginner-Intermediate | Time: ~90 min

---

## Learning Objectives

- Create a bootable Ubuntu USB drive
- Install Ubuntu 22.04 LTS (bare metal or dual-boot)
- Set up Ubuntu in VirtualBox (recommended for beginners)
- Enable WSL2 on Windows 10/11
- Perform essential post-installation configuration
- Update the system and install basic tools

---

## Table of Contents

1. [Choosing Your Installation Method](#1-choosing-your-installation-method)
2. [Downloading Ubuntu](#2-downloading-ubuntu)
3. [VirtualBox Installation (Recommended)](#3-virtualbox-installation-recommended)
4. [Creating a Bootable USB](#4-creating-a-bootable-usb)
5. [Bare Metal / Dual-Boot Installation](#5-bare-metal--dual-boot-installation)
6. [WSL2 on Windows](#6-wsl2-on-windows)
7. [Post-Installation Setup](#7-post-installation-setup)
8. [Essential First Commands](#8-essential-first-commands)
9. [Practice Exercises](#9-practice-exercises)
10. [Key Takeaways](#10-key-takeaways)

---

## 1. Choosing Your Installation Method

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| **VirtualBox VM** | Safe, reversible, snapshots | Slower, shared resources | Learning, experiments |
| **Dual-boot** | Full performance, real hardware | Risk of data loss, partitioning | Serious use alongside Windows |
| **Bare metal** | Maximum performance | Replaces existing OS | Dedicated Linux machine |
| **WSL2** | Windows integration, easy setup | No GUI by default, limited kernel | Windows users wanting CLI |
| **Live USB** | No installation, portable | No persistence (by default) | Testing, rescue |
| **Cloud VM** | Accessible anywhere | Costs money, network dependent | Remote learning |

> 🎯 **Recommendation for beginners**: Start with **VirtualBox**. It's safe, free, and you can snapshot before making dangerous changes.

---

## 2. Downloading Ubuntu

### Ubuntu 22.04 LTS (Jammy Jellyfish)

1. Go to **[ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)**
2. Click "Download 22.04.x LTS"
3. You'll get an `.iso` file (~5 GB)

### Verify the Download (Important for Security!)

```bash
# On Linux/macOS — verify SHA256 checksum
sha256sum ubuntu-22.04.x-desktop-amd64.iso

# Compare with official checksum from ubuntu.com/download/desktop
# They must match exactly
```

```powershell
# On Windows PowerShell
Get-FileHash ubuntu-22.04.x-desktop-amd64.iso -Algorithm SHA256
```

If the checksums don't match, the file is corrupt or tampered with — download again.

---

## 3. VirtualBox Installation (Recommended)

### Step 1: Install VirtualBox

1. Download from **[virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)**
2. Choose your host OS (Windows, macOS, or Linux)
3. Install VirtualBox and the **VirtualBox Extension Pack** (for USB 3.0, RDP, etc.)

### Step 2: Create a New Virtual Machine

1. Open VirtualBox → Click **"New"**
2. Configure:

```
Name:    Ubuntu 22.04
Type:    Linux
Version: Ubuntu (64-bit)
```

3. **Memory (RAM)**: Minimum 2048 MB; recommend **4096 MB** (4 GB)
4. **Hard Disk**: Create a virtual hard disk
   - VDI (VirtualBox Disk Image)
   - Dynamically allocated
   - Size: **25 GB minimum** (50 GB recommended)

### Step 3: Configure VM Settings

Before starting, click **Settings**:

**System tab:**
- Processors: Set to 2+ CPUs (half your physical cores)
- Enable EFI: Optional (leave off for simplicity)

**Display tab:**
- Video Memory: 128 MB
- Enable 3D Acceleration: Yes (if available)

**Storage tab:**
- Click the empty CD icon → "Choose a disk file"
- Select your Ubuntu `.iso` file

**Network tab:**
- Adapter 1: NAT (internet access)
- Adapter 2: Host-only Adapter (for host-VM communication)

### Step 4: Install Ubuntu

1. Click **Start** to boot the VM
2. Select **"Try or Install Ubuntu"**
3. Language: English → **"Install Ubuntu"**
4. Keyboard layout → select yours → Continue
5. **Updates**: "Normal installation" + check both update boxes
6. **Installation type**: "Erase disk and install Ubuntu" (safe — only erases the virtual disk!)
7. Select timezone → Create your user account
8. Wait ~15 minutes for installation
9. **Restart Now** when prompted → Press Enter when asked

### Step 5: Install VirtualBox Guest Additions

Guest Additions improve performance and enable features like:
- Shared clipboard
- Drag and drop
- Auto-resize display
- Shared folders

```bash
# After Ubuntu starts, open a terminal (Ctrl+Alt+T)
# VirtualBox menu: Devices → Insert Guest Additions CD Image

sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
sudo mount /dev/cdrom /media/cdrom
sudo /media/cdrom/VBoxLinuxAdditions.run
sudo reboot
```

### Step 6: Take a Snapshot

Before doing anything else:
- VirtualBox → Machine → **Take Snapshot**
- Name: "Clean Install"
- This is your safety net — you can always restore to this point!

---

## 4. Creating a Bootable USB

If you want to install on real hardware, you need a bootable USB drive (8 GB+).

### On Windows: Using Rufus

1. Download **Rufus** from [rufus.ie](https://rufus.ie)
2. Insert USB drive (8 GB+) — **all data will be erased**
3. Open Rufus:
   - Device: Select your USB drive
   - Boot selection: Select Ubuntu `.iso`
   - Partition scheme: **GPT** (for UEFI) or MBR (for legacy BIOS)
   - Click **START**
4. Wait ~10 minutes

### On Linux/macOS: Using `dd`

```bash
# Find your USB device (look for your USB size)
lsblk
# or
diskutil list  # macOS

# Write the ISO (REPLACE /dev/sdX with your actual device!)
# WARNING: This WILL destroy all data on the target device
sudo dd if=ubuntu-22.04.x-desktop-amd64.iso \
        of=/dev/sdX \
        bs=4M \
        status=progress \
        conv=fsync

# Sync and eject
sync
sudo eject /dev/sdX
```

> ⚠️ **DANGER**: The `dd` command will destroy data on the wrong device. Triple-check `of=/dev/sdX` before running!

### On Any OS: Balena Etcher

[Etcher](https://www.balena.io/etcher) is a safer, graphical tool:
1. Download and install Etcher
2. Select ISO → Select USB drive → Flash!
3. It automatically validates the write

---

## 5. Bare Metal / Dual-Boot Installation

### Before You Start — CRITICAL STEPS

1. **Back up all important data** to an external drive
2. **Disable BitLocker** (Windows) before dual-booting
3. **Disable Secure Boot** in BIOS (some Linux installs require this)
4. **Create a recovery drive** for Windows
5. **Free up disk space**: Windows Disk Management → Shrink Volume (at least 50 GB)

### Booting from USB

1. Insert bootable USB
2. Restart computer
3. Access boot menu (usually **F12**, **F2**, **Del**, or **Esc** at startup)
4. Select USB drive
5. Ubuntu installer starts

### Installation Steps

1. **Try Ubuntu** first — verify hardware works (WiFi, display, etc.)
2. Click **"Install Ubuntu"** on the desktop
3. Language + keyboard → Continue
4. Network: Connect to WiFi if needed
5. **Installation type** (most important step!):

```
┌─────────────────────────────────────────────────┐
│ Installation Type                               │
│                                                 │
│ ○ Install Ubuntu alongside Windows Boot Manager│  ← Dual-boot
│   (Easiest - Ubuntu handles partitioning)       │
│                                                 │
│ ○ Erase disk and install Ubuntu                 │  ← Replaces everything
│                                                 │
│ ● Something else                                │  ← Manual partitioning
└─────────────────────────────────────────────────┘
```

### Manual Partitioning (Recommended for Dual-Boot)

You should have pre-allocated free/unallocated space on your drive.

Recommended partition scheme:

| Partition | Size | Type | Mount Point | Format |
|-----------|------|------|-------------|--------|
| EFI System | 512 MB | EFI | /boot/efi | fat32 |
| Boot | 1 GB | Primary | /boot | ext4 |
| Root | 30-50 GB | Primary | / | ext4 |
| Swap | = RAM size | Swap | swap | swap |
| Home | Remaining | Primary | /home | ext4 |

> 📌 Separating `/home` means you can reinstall Ubuntu without losing personal files.

---

## 6. WSL2 on Windows

**WSL2** (Windows Subsystem for Linux 2) runs a real Linux kernel inside Windows.

### Requirements
- Windows 10 version 2004+ or Windows 11
- Virtualization enabled in BIOS

### Installation

```powershell
# Open PowerShell as Administrator

# Install WSL2 with Ubuntu (one command!)
wsl --install

# Or install a specific distro
wsl --install -d Ubuntu-22.04

# Restart Windows when prompted
```

### First Time Setup

After restart, Ubuntu opens automatically:
```
Installing, this may take a few minutes...
Please create a default UNIX user account.
Enter new UNIX username: yourname
New password: (won't show while typing)
Retype new password:
```

### Accessing WSL2

```powershell
# From PowerShell or CMD
wsl                     # Start default distro
wsl -d Ubuntu-22.04    # Start specific distro
wsl --list --verbose    # List installed distros
wsl --shutdown          # Stop all WSL instances
```

### Accessing Windows Files from WSL2

```bash
# Windows drives are mounted under /mnt/
ls /mnt/c/Users/YourName/Documents
cd /mnt/c/Users/YourName/Desktop

# Access WSL2 files from Windows Explorer:
# Type in address bar: \\wsl$\Ubuntu-22.04
```

### WSL2 Limitations

- No systemd by default (available in WSL2 with modern Windows)
- No direct hardware access (USB devices, etc.) without USB/IP
- No GUI by default (use WSLg on Windows 11 for GUI apps)
- Some kernel features unavailable

---

## 7. Post-Installation Setup

After installation (any method), run these essential setup steps:

### Step 1: Update Everything

```bash
# Update package list from servers
sudo apt update

# Upgrade all installed packages
sudo apt upgrade -y

# Upgrade to new package versions (kernel, etc.)
sudo apt full-upgrade -y

# Remove unneeded packages
sudo apt autoremove -y
sudo apt autoclean
```

### Step 2: Install Essential Tools

```bash
# Development essentials
sudo apt install -y build-essential git curl wget

# Text editors
sudo apt install -y vim nano

# Network tools
sudo apt install -y net-tools nmap netcat-openbsd

# System tools
sudo apt install -y htop tree unzip p7zip-full

# Security tools (for later lessons)
sudo apt install -y ufw fail2ban
```

### Step 3: Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git config --global core.editor "nano"
git config --list
```

### Step 4: Set Up SSH (Optional but Useful)

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "your@email.com"
# Press Enter for default location, set a passphrase

# View your public key (share this, never the private key)
cat ~/.ssh/id_ed25519.pub

# Start SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Step 5: Configure the Firewall

```bash
# Enable UFW (Uncomplicated Firewall)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh         # Allow SSH (port 22)
sudo ufw enable
sudo ufw status verbose
```

### Step 6: Customize the Shell

```bash
# Install zsh (optional but popular)
sudo apt install -y zsh

# Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Or customize bash — add to ~/.bashrc:
echo 'export PS1="\u@\h:\w\$ "' >> ~/.bashrc
echo 'alias ll="ls -alF"' >> ~/.bashrc
echo 'alias la="ls -A"' >> ~/.bashrc
echo 'alias l="ls -CF"' >> ~/.bashrc
source ~/.bashrc
```

### Step 7: Set Correct Timezone

```bash
# Check current timezone
timedatectl status

# List available timezones
timedatectl list-timezones | grep America

# Set your timezone
sudo timedatectl set-timezone America/New_York
# or: America/Los_Angeles, Europe/London, Asia/Tokyo, etc.
```

---

## 8. Essential First Commands

Try these commands to confirm your installation is working:

```bash
# System information
uname -a                    # Kernel version and architecture
lsb_release -a              # Ubuntu version
hostnamectl                 # Hostname and OS info
uptime                      # How long system has been running

# Hardware information
lscpu                       # CPU information
free -h                     # RAM usage (human-readable)
df -h                       # Disk space usage
lsblk                       # Block devices (disks)
lspci                       # PCI devices (GPU, NIC, etc.)
lsusb                       # USB devices

# Network information
ip addr                     # IP addresses
ip route                    # Routing table
cat /etc/resolv.conf        # DNS servers
ping -c 4 google.com        # Test internet connectivity

# User information
whoami                      # Current username
id                          # User ID, group ID
groups                      # Groups you belong to
w                           # Who is logged in
```

### Expected Output Examples

```bash
$ uname -a
Linux ubuntu 5.15.0-88-generic #98-Ubuntu SMP Mon Oct 2 15:18:56 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux

$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.3 LTS
Release:        22.04
Codename:       jammy

$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       1.2Gi       5.1Gi        42Mi       1.4Gi       6.2Gi
Swap:          2.0Gi          0B       2.0Gi
```

---

## 9. Practice Exercises

### Exercise 3.1 — VirtualBox Setup

1. Download Ubuntu 22.04 LTS ISO
2. Create a VirtualBox VM with 4 GB RAM and 50 GB disk
3. Install Ubuntu
4. Take a snapshot called "Clean Install"
5. Run `uname -a` and record the output

### Exercise 3.2 — System Information Report

After installing Ubuntu, run each command and record the output:

```bash
uname -a
lsb_release -a
free -h
df -h
lscpu | grep -E "Model name|CPU\(s\)"
ip addr | grep inet
```

### Exercise 3.3 — Post-Install Checklist

Complete all 7 post-installation steps above and verify each:

```bash
# Verify updates applied
apt list --upgradable 2>/dev/null | wc -l  # Should be 0

# Verify git config
git config --list

# Verify firewall
sudo ufw status

# Verify timezone
timedatectl status
```

### Exercise 3.4 — WSL2 (Windows Users)

If on Windows:
1. Install WSL2 with Ubuntu
2. Open Ubuntu and run: `cat /etc/os-release`
3. Access your Windows Documents folder from WSL2
4. Create a file in WSL2 and access it from Windows Explorer

---

## 10. Key Takeaways

- **VirtualBox** is the safest and easiest way to learn Linux
- Always **verify checksums** of downloaded ISOs
- **Take snapshots** before making system changes
- **WSL2** gives Windows users a real Linux environment
- After installation, always **update the system** immediately
- A good firewall (UFW) should be configured from day one
- Separate `/home` partition makes reinstalls easier

---

## Next Lesson

➡️ [04 — The Terminal Basics](04_The_Terminal_Basics.md)

---

*Module 1 · Lesson 3 of 6 | [Course Index](../INDEX.md)*
