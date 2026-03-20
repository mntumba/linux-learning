# 01 — Introduction to Linux

> **Module 1 · Lesson 1** | Difficulty: ★☆☆☆☆ Beginner | Time: ~60 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Explain what Linux is and why it matters
- Describe the role of the Linux kernel vs the rest of the OS
- Compare major Linux distributions and choose one for your needs
- Identify common Linux use cases (server, desktop, embedded, security)
- Understand the open-source philosophy and the GNU/GPL license

---

## Table of Contents

1. [What Is Linux?](#1-what-is-linux)
2. [The Linux Kernel](#2-the-linux-kernel)
3. [Linux Distributions](#3-linux-distributions)
4. [Why Learn Linux?](#4-why-learn-linux)
5. [Linux vs Windows vs macOS](#5-linux-vs-windows-vs-macos)
6. [Open Source & the GNU Philosophy](#6-open-source--the-gnu-philosophy)
7. [Where Is Linux Used?](#7-where-is-linux-used)
8. [Practice Exercises](#8-practice-exercises)
9. [Key Takeaways](#9-key-takeaways)

---

## 1. What Is Linux?

**Linux** is a free, open-source, Unix-like operating system kernel created by **Linus Torvalds** in 1991 while he was a student at the University of Helsinki.

An **operating system (OS)** is the software layer that sits between your hardware and your applications. It manages:

- CPU time (who runs when)
- Memory allocation
- Storage (files and file systems)
- Networking
- Devices (keyboard, disk, GPU, …)

Linux is technically just the **kernel** — the core. When people say "Linux," they usually mean a complete operating system built around the Linux kernel, often called a **Linux distribution** (or "distro").

```
┌──────────────────────────────────┐
│          USER SPACE              │
│  ┌──────┐ ┌───────┐ ┌────────┐  │
│  │ bash │ │Firefox│ │ Python │  │
│  └──────┘ └───────┘ └────────┘  │
│  ┌──────────────────────────┐    │
│  │   Standard C Library     │    │
│  │         (glibc)          │    │
│  └──────────────────────────┘    │
├──────────────────────────────────┤
│          KERNEL SPACE            │
│  ┌──────────────────────────┐    │
│  │     Linux Kernel         │    │
│  │  (process mgmt, memory,  │    │
│  │   file systems, network) │    │
│  └──────────────────────────┘    │
├──────────────────────────────────┤
│            HARDWARE              │
│  CPU  │  RAM  │  Disk  │  NIC   │
└──────────────────────────────────┘
```

---

## 2. The Linux Kernel

The **kernel** is the heart of the OS. Its responsibilities:

| Responsibility | Description |
|---|---|
| **Process Management** | Creates, schedules, and terminates processes |
| **Memory Management** | Allocates RAM, manages virtual memory & swap |
| **Device Drivers** | Communicates with hardware via drivers |
| **File Systems** | Provides ext4, btrfs, xfs, tmpfs, etc. |
| **Networking** | TCP/IP stack, sockets, packet filtering |
| **Security** | User permissions, namespaces, cgroups, SELinux |
| **System Calls** | API between user programs and the kernel |

The kernel runs in **privileged mode** (ring 0). Regular programs run in **user mode** (ring 3) and request kernel services through **system calls** (e.g., `read()`, `write()`, `fork()`).

---

## 3. Linux Distributions

A **distribution (distro)** is the Linux kernel bundled with:

- A package manager (apt, dnf, pacman…)
- Init system (systemd, OpenRC…)
- Desktop environment (GNOME, KDE, XFCE…) — optional
- Pre-installed software

### Major Distribution Families

```
                        Linux Kernel
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
        Debian            Red Hat            Arch
          │                 │                 │
     ┌────┴────┐       ┌────┴────┐       ┌────┴────┐
   Debian    Ubuntu   RHEL     Fedora   Arch   Manjaro
              │                  │
         ┌────┴────┐         CentOS
       Ubuntu    Kali           Rocky
       Server    Linux          AlmaLinux
                  │
              ParrotOS
```

### Popular Distros at a Glance

| Distro | Base | Package Mgr | Best For |
|--------|------|-------------|----------|
| **Ubuntu 22.04 LTS** | Debian | apt | Beginners, servers, desktops |
| **Debian** | — | apt | Stability, servers |
| **Kali Linux** | Debian | apt | Penetration testing |
| **Fedora** | Red Hat | dnf | Cutting-edge desktop |
| **CentOS Stream** | Red Hat | dnf | Enterprise servers |
| **Arch Linux** | — | pacman | Advanced users, DIY |
| **Manjaro** | Arch | pacman | Arch-based + ease of use |
| **Parrot OS** | Debian | apt | Security + privacy |
| **Rocky Linux** | Red Hat | dnf | Enterprise (RHEL clone) |
| **Alpine Linux** | — | apk | Containers, minimal footprint |

> 📌 **This course uses Ubuntu 22.04 LTS** (Long-Term Support — supported until April 2027). It's the most popular distro for beginners and professionals alike.

### Ubuntu Release Naming

Ubuntu releases are named `YY.MM` and have animal nicknames:

- **22.04 LTS** — Jammy Jellyfish (supported 5 years)
- **23.10** — Mantic Minotaur (standard 9 months)
- **24.04 LTS** — Noble Numbat (supported 5 years)

LTS releases are recommended for production systems and learning.

---

## 4. Why Learn Linux?

### Career Demand

- **90%** of public cloud workloads run on Linux
- **96.3%** of the world's top 1 million web servers run Linux (W3Techs, 2024)
- **100%** of the top 500 supercomputers run Linux
- Every major cloud provider (AWS, GCP, Azure) runs Linux VMs by default
- Android (3 billion+ devices) is built on the Linux kernel

### Security & Hacking

- All major penetration testing tools (Metasploit, Nmap, Wireshark, Burp Suite) run natively on Linux
- Kali Linux is the industry-standard platform for ethical hacking
- Understanding Linux is essential for:
  - CTF competitions
  - Bug bounty programs
  - Red/Blue team operations
  - Malware analysis and reverse engineering

### Developer Power

- Bash scripting automates repetitive tasks
- Linux tools (grep, sed, awk, curl, ssh) are industry standards
- Native support for Docker, Kubernetes, Python, Go, Rust, etc.

### Cost & Freedom

- **Free** — no license fees
- **Open source** — inspect, modify, redistribute
- Runs on almost any hardware (old laptops, Raspberry Pi, mainframes)

---

## 5. Linux vs Windows vs macOS

| Feature | Linux | Windows | macOS |
|---------|-------|---------|-------|
| **Cost** | Free | $139+ | Free (requires Apple HW) |
| **License** | GPL (open source) | Proprietary | Proprietary |
| **Source code** | Public | Closed | Partially open (Darwin) |
| **Package manager** | apt, dnf, pacman | winget (limited) | Homebrew (unofficial) |
| **CLI power** | Excellent (bash, zsh) | PowerShell (limited) | Good (zsh on Apple) |
| **Customisation** | Total control | Limited | Moderate |
| **Security** | Very strong | Target-rich | Strong |
| **Gaming** | Improving (Steam/Proton) | Best | Moderate |
| **Server use** | Dominant | Common (IIS) | Rare |
| **Learning curve** | Moderate → High | Low | Low → Moderate |

### Key Philosophical Differences

- **Linux**: text files configure everything; commands do one thing well; everything is a file
- **Windows**: GUI-centric; registry-based configuration; proprietary formats
- **macOS**: polished GUI; Unix underpinnings; locked to Apple hardware

---

## 6. Open Source & the GNU Philosophy

### What Is Open Source?

**Open source** software provides:

1. **Freedom 0** — Run the program for any purpose
2. **Freedom 1** — Study and modify the source code
3. **Freedom 2** — Redistribute copies
4. **Freedom 3** — Distribute modified versions

### The GNU Project

- Founded by **Richard Stallman** in 1983 at MIT
- Goal: create a completely free Unix-like OS
- Created: gcc, glibc, bash, grep, sed, awk, emacs, make
- Missing: a kernel (the "GNU Hurd" was never finished)

### Linux Fills the Gap

In 1991, Linus Torvalds wrote the Linux kernel and released it under the **GPL** license. Combined with GNU tools → **GNU/Linux**.

### GPL License (GNU General Public License)

The GPL ensures:

- Anyone can use, study, and modify the software
- Modified versions must also be released under GPL ("copyleft")
- You cannot make GPL software proprietary

Other open source licenses: MIT, Apache 2.0, BSD (more permissive — allow proprietary use).

---

## 7. Where Is Linux Used?

### Servers & Cloud

```bash
# The majority of web servers on the internet run Linux
# Example: nginx running on Ubuntu
systemctl status nginx
```

- Web servers: Apache, Nginx
- Databases: PostgreSQL, MySQL, MongoDB
- Cloud: AWS EC2, Google Cloud, Azure VMs (most use Linux)
- Containers: Docker, Kubernetes (always Linux at the base)

### Embedded & IoT

- **Android** — Linux kernel with Android framework
- **Routers** — Cisco, MikroTik, OpenWRT
- **Smart TVs** — Samsung Tizen, LG webOS
- **Raspberry Pi** — Raspberry Pi OS (Debian-based)
- **Automotive** — Tesla runs Linux; GENIVI/AGL standard
- **Space** — SpaceX Falcon 9 uses Linux for flight control!

### Desktop

- GNOME desktop (Ubuntu default)
- KDE Plasma — highly customisable
- XFCE — lightweight for older hardware
- i3/sway — tiling window managers (power users)

### Security & Hacking

- **Kali Linux** — 600+ pre-installed security tools
- **Parrot OS** — lightweight alternative to Kali
- **Tails OS** — anonymity/privacy focused
- **BlackArch** — Arch-based, 2800+ tools

### Supercomputing

All 500 of the world's fastest supercomputers run Linux. NASA's Pleiades, CERN's computing grid, and most AI training clusters run Linux.

---

## 8. Practice Exercises

### Exercise 1.1 — Research Linux Distributions

1. Visit [distrowatch.com](https://distrowatch.com) (or read offline if no internet)
2. Find 3 distributions you haven't heard of and note their:
   - Base distribution
   - Package manager
   - Target audience
3. Answer: Which distro would you choose for a Raspberry Pi home server and why?

### Exercise 1.2 — Understand the OS Layers

Draw (on paper or text) the relationship between:
- Hardware → Kernel → System Libraries → Shell → User Applications

Label each arrow with what travels across it (system calls, library calls, etc.)

### Exercise 1.3 — Open Source Investigation

1. Go to [github.com/torvalds/linux](https://github.com/torvalds/linux)
2. Look at the top-level folders: `kernel/`, `net/`, `drivers/`, `fs/`
3. What is in each folder? What does this tell you about kernel responsibilities?

### Exercise 1.4 — Linux in the Wild

List 10 devices around you (phone, router, TV, car, etc.) that might be running Linux. For each, explain what evidence you have.

---

## 9. Key Takeaways

- **Linux** = Linux kernel + GNU tools + package manager + applications
- The kernel manages hardware resources; user programs use system calls to access them
- Linux comes in many **distributions** — Ubuntu is the best starting point
- Linux powers **90%+ of cloud infrastructure** and all supercomputers
- It's the platform of choice for **security professionals and hackers**
- **Open source** means you can inspect, modify, and redistribute the software
- The **GPL license** ensures software stays free forever

---

## Next Lesson

➡️ [02 — Linux History and Philosophy](02_Linux_History_and_Philosophy.md)

---

*Module 1 · Lesson 1 of 6 | [Course Index](../INDEX.md)*
