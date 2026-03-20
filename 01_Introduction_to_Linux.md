# 01 — Introduction to Linux

> **Difficulty:** Beginner | **Time Estimate:** 45–60 minutes

---

## Learning Objectives

By the end of this lesson you will be able to:

- Explain what Linux is and how it differs from Windows and macOS
- Describe the key characteristics that make Linux unique
- Name at least five major Linux distributions and their use cases
- Identify where Linux is used in the real world
- Understand the layered architecture of a Linux system
- Know why Ubuntu is recommended for beginners

---

## 1. What Is Linux?

Linux is a **free, open-source operating system kernel** created by Linus Torvalds in 1991. Combined with the GNU tools and utilities produced by Richard Stallman's GNU Project, it forms a complete operating system commonly called **GNU/Linux** — though most people simply call it Linux.

Unlike Windows or macOS, Linux is not a single product from a single company. It is a kernel (the core of the OS) that forms the foundation of hundreds of different **distributions** (distros), each assembled by communities or companies for different purposes.

> **Kernel:** The kernel is the bridge between your hardware and your software. It manages memory, CPU scheduling, device drivers, and system calls.

---

## 2. Linux vs Windows vs macOS

| Feature | Linux | Windows | macOS |
|---|---|---|---|
| **Cost** | Free (mostly) | $139–$199 | Free (requires Apple hardware) |
| **Source Code** | Open source | Closed source | Partially open (Darwin kernel) |
| **Security** | Very high | Moderate | High |
| **Customisability** | Unlimited | Limited | Limited |
| **Package Manager** | Yes (apt, dnf, pacman…) | No (winget is limited) | Homebrew (unofficial) |
| **Market Share (Desktop)** | ~3% | ~72% | ~15% |
| **Market Share (Server)** | ~96% | ~3% | <1% |
| **Gaming** | Improving (Steam/Proton) | Excellent | Good |
| **Virus Prevalence** | Very low | High | Low |
| **CLI First-Class** | Yes | No | Partial |
| **File System** | ext4, xfs, btrfs… | NTFS, FAT32 | APFS, HFS+ |
| **Update Control** | Full control | Forced updates | Mostly controlled |

---

## 3. Key Characteristics of Linux

### 3.1 Open Source

Linux is licensed under the **GNU General Public License (GPL)**. This means:

- You can view the source code
- You can modify it
- You can redistribute it (under the same license)

```bash
# View the Linux kernel version on your system
uname -r

# See full system information
uname -a
```

### 3.2 Multi-User

Multiple users can be logged in and running processes simultaneously without interfering with each other.

```bash
# See who is currently logged in
who

# See all users on the system
cat /etc/passwd | cut -d: -f1

# Switch to another user
su - username
```

### 3.3 Multi-Tasking

Linux can run thousands of processes concurrently, managed by the scheduler in the kernel.

```bash
# View running processes
ps aux

# Interactive process viewer
top

# Count total running processes
ps aux | wc -l
```

### 3.4 Security

Linux uses a permissions model where every file has an owner, a group, and access rights (read/write/execute). Ordinary users cannot alter system files without explicit privilege elevation.

```bash
# Show your current user
whoami

# Show your user ID and group memberships
id

# Elevate privilege for a single command (requires sudo access)
sudo apt update
```

### 3.5 Portability

Linux runs on an enormous range of hardware: x86 PCs, ARM chips (Raspberry Pi, phones), RISC-V, SPARC, PowerPC, and more.

---

## 4. Linux Architecture

The following ASCII diagram shows how a Linux system is layered from hardware to user applications:

```
┌─────────────────────────────────────────────────┐
│                  USER SPACE                     │
│                                                 │
│  ┌───────────┐  ┌───────────┐  ┌────────────┐  │
│  │   Web     │  │  Office   │  │  Terminal  │  │
│  │  Browser  │  │   Suite   │  │    App     │  │
│  └───────────┘  └───────────┘  └────────────┘  │
│                    Applications                 │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │              SHELL (bash/zsh)           │   │
│  │   Interprets commands, runs scripts     │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │       GNU C LIBRARY (glibc)             │   │
│  │   System call interface for programs    │   │
│  └─────────────────────────────────────────┘   │
├─────────────────────────────────────────────────┤
│                 KERNEL SPACE                    │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │            LINUX KERNEL                 │   │
│  │  ┌──────────┐  ┌──────────┐            │   │
│  │  │ Process  │  │  Memory  │            │   │
│  │  │ Scheduler│  │ Manager  │            │   │
│  │  └──────────┘  └──────────┘            │   │
│  │  ┌──────────┐  ┌──────────┐            │   │
│  │  │   File   │  │ Network  │            │   │
│  │  │  System  │  │  Stack   │            │   │
│  │  └──────────┘  └──────────┘            │   │
│  │  ┌──────────────────────────────────┐  │   │
│  │  │        Device Drivers            │  │   │
│  │  └──────────────────────────────────┘  │   │
│  └─────────────────────────────────────────┘   │
├─────────────────────────────────────────────────┤
│                  HARDWARE                       │
│                                                 │
│   CPU   │   RAM   │   Disk   │   NIC   │  GPU   │
└─────────────────────────────────────────────────┘
```

---

## 5. Linux Distributions

A **distribution** (distro) bundles the Linux kernel with a package manager, desktop environment, default applications, and configuration tools.

### Major Distributions

| Distribution | Base | Package Manager | Best For |
|---|---|---|---|
| **Ubuntu** | Debian | apt | Beginners, desktops, servers |
| **Debian** | Independent | apt | Stability, servers |
| **Fedora** | Independent | dnf | Cutting-edge features, developers |
| **Arch Linux** | Independent | pacman | Advanced users, total control |
| **Kali Linux** | Debian | apt | Penetration testing, security |
| **CentOS/RHEL** | Independent | dnf/yum | Enterprise servers |
| **Linux Mint** | Ubuntu | apt | Windows refugees, beginners |
| **openSUSE** | Independent | zypper | Enterprise, developers |
| **Raspberry Pi OS** | Debian | apt | IoT, embedded systems |

```bash
# Find out which distro you are running
cat /etc/os-release

# On Debian/Ubuntu-based systems, see the version
lsb_release -a

# Check kernel version
uname -r
```

### Distribution Family Tree (Simplified)

```
UNIX (1969)
└── Linux Kernel (1991)
    ├── Debian
    │   ├── Ubuntu ──── Linux Mint, Pop!_OS, Zorin OS
    │   └── Kali Linux, Raspberry Pi OS
    ├── Red Hat (RHEL)
    │   ├── Fedora
    │   └── CentOS, Rocky Linux, AlmaLinux
    ├── Arch Linux ──── Manjaro, EndeavourOS
    └── openSUSE ────── SUSE Linux Enterprise
```

---

## 6. Real-World Uses of Linux

### 6.1 Web Servers

Over **96% of the world's web servers** run Linux. Every time you visit a website, there is a very high chance a Linux system served your request.

```bash
# See what web server is running (on a Linux server)
systemctl status nginx
systemctl status apache2
```

### 6.2 Android

Android's core is built on the Linux kernel. Your Android smartphone is, at its heart, a Linux device.

### 6.3 Supercomputers

**100% of the TOP500 supercomputers** run Linux. From weather simulation to AI research, Linux is the OS of high-performance computing.

### 6.4 IoT and Embedded Systems

Routers, smart TVs, cars (Tesla runs Linux), drones, and Raspberry Pi projects all rely on Linux.

### 6.5 Cloud Infrastructure

AWS, Google Cloud, and Azure all run their infrastructure predominantly on Linux. Docker containers and Kubernetes orchestration are Linux-native technologies.

```bash
# Check if you are inside a container
cat /proc/1/cgroup | grep docker

# View system uptime (servers often run for years)
uptime
```

### 6.6 Desktop Computing

While desktop market share is modest (~3%), Linux powers developer workstations worldwide, especially at companies like Google, Facebook, and NASA.

---

## 7. Linux Market Share Statistics

```
Server Market Share (2024)
──────────────────────────
Linux    ████████████████████████████████████████ 96%
Windows  ██ 3%
Other    █ 1%

Supercomputer OS (TOP500)
──────────────────────────
Linux    ████████████████████████████████████████ 100%

Mobile OS (Linux kernel-based)
───────────────────────────────
Android  ████████████████████████████████ 72%
iOS      ████████████████ 27%
Other    █ 1%

Desktop Market Share
─────────────────────
Windows  ███████████████████████████████████ 72%
macOS    ████████ 15%
Linux    ██ 3%
ChromeOS █ 3%
Other    ███████ 7%
```

---

## 8. Why Ubuntu for Beginners?

Ubuntu is the recommended starting point for most new Linux users because:

1. **Largest community** — More tutorials, Stack Overflow answers, and forum threads than any other distro
2. **Long-Term Support (LTS)** — LTS releases (e.g., 22.04, 24.04) are supported for **5 years**
3. **Hardware support** — Excellent driver support out of the box
4. **GNOME desktop** — Polished, modern, and intuitive
5. **apt package manager** — Thousands of packages with one command
6. **Snap packages** — Easy installation of complex apps
7. **Strong corporate backing** — Canonical actively develops and maintains it

```bash
# Update package list and upgrade all packages (Ubuntu/Debian)
sudo apt update && sudo apt upgrade -y

# Install a package
sudo apt install vim -y

# Search for a package
apt search python3

# Remove a package
sudo apt remove vim
```

---

## 9. Getting Help

Linux has extensive built-in documentation.

```bash
# Read the manual for any command
man ls
man grep
man bash

# Get a brief description of a command
whatis ls

# Find commands related to a keyword
apropos "copy files"

# Get built-in help for a command
ls --help
cp --help
```

---

## Practice Exercises

### Exercise 1 — Explore Your System
Open a terminal and run the following commands. Note what each returns.

```bash
uname -a
cat /etc/os-release
whoami
id
uptime
```

**Question:** What kernel version is running? What distribution are you using?

### Exercise 2 — User Investigation
Find out how many user accounts exist on the system.

```bash
# Count lines in /etc/passwd
wc -l /etc/passwd

# List only human users (UID >= 1000)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd
```

### Exercise 3 — Process Count
Determine how many processes are currently running.

```bash
ps aux | wc -l
```

**Question:** What is the first process listed? What PID does it have?

### Exercise 4 — Hardware Discovery
Explore your hardware from the command line.

```bash
# CPU information
cat /proc/cpuinfo | grep "model name" | head -1

# Memory information
free -h

# Disk usage
df -h
```

### Exercise 5 — Distribution Research
Visit [distrowatch.com](https://distrowatch.com) (or use the info in this lesson) and answer:

1. What package manager does Arch Linux use?
2. Which distro is best suited for penetration testing?
3. What is the parent distribution of Linux Mint?

---

## Common Mistakes

| Mistake | Why It Happens | Fix |
|---|---|---|
| Running everything as root | Thinking it's faster | Use `sudo` only when needed |
| Ignoring `man` pages | They seem complex | Start with `command --help` then graduate to `man` |
| Expecting Linux = Ubuntu | There are 600+ distros | Research which distro fits your goal |
| Comparing Linux 1:1 with Windows | Different paradigms | Learn the Linux way: CLI first, then GUI |
| Not updating the system | Forgetting after install | Schedule `sudo apt update && sudo apt upgrade` regularly |

---

## Pro Tips

> 💡 **Use `Tab` completion** — Pressing `Tab` completes commands and file names. Press it twice to see all possibilities.

> 💡 **`Ctrl+C` stops a running command** — If something hangs, `Ctrl+C` sends an interrupt signal.

> 💡 **Arrow keys scroll command history** — Press `↑` to cycle through previous commands.

> 💡 **`!!` repeats the last command** — Useful for `sudo !!` when you forgot to prefix with sudo.

> 💡 **Everything is a file** — In Linux, devices, sockets, and even processes are represented as files in the filesystem.

---

## Key Takeaways

- Linux is a free, open-source OS kernel used everywhere from phones to supercomputers
- It differs from Windows/macOS in being open, highly customisable, and CLI-centric
- Distributions package the kernel with tools to make a complete OS
- Ubuntu is the best starting point for new users
- The architecture flows: Hardware → Kernel → Shell → Applications
- Linux powers 96% of web servers and 100% of supercomputers
- Built-in tools (`man`, `--help`, `apropos`) are your first resource

---

## Next Lesson Preview

**02 — Linux History and Philosophy**

Discover how Unix was born at Bell Labs in 1969, how Richard Stallman's GNU project laid the foundation for free software, and how a Finnish student named Linus Torvalds changed computing forever in 1991. We will also explore the Unix Philosophy — a set of design principles that make Linux commands powerful when combined together.
