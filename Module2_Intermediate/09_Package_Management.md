# 09 — Package Management

> **Module 2 · Lesson 3** | Difficulty: ★★★☆☆ Intermediate | Time: ~75 min

---

## Learning Objectives

- Use APT to install, update, and remove packages
- Work with dpkg for low-level package management
- Use Snap and Flatpak for containerized applications
- Add PPAs and third-party repositories
- Manage package sources
- Compile software from source
- Understand package security

---

## Table of Contents

1. [Package Management Overview](#1-package-management-overview)
2. [APT — Advanced Package Tool](#2-apt--advanced-package-tool)
3. [dpkg — Debian Package Manager](#3-dpkg--debian-package-manager)
4. [Snap Packages](#4-snap-packages)
5. [Flatpak](#5-flatpak)
6. [Adding Repositories and PPAs](#6-adding-repositories-and-ppas)
7. [Compiling From Source](#7-compiling-from-source)
8. [Python Packages (pip)](#8-python-packages-pip)
9. [AppImage](#9-appimage)
10. [Practice Exercises](#10-practice-exercises)
11. [Key Takeaways](#11-key-takeaways)

---

## 1. Package Management Overview

A **package manager** handles:
- Downloading software from repositories
- Resolving dependencies automatically
- Installing, upgrading, and removing software
- Verifying package integrity (signatures)

### Package Manager Comparison

| Manager | Format | Distro | Command |
|---------|--------|--------|---------|
| **apt** | .deb | Debian/Ubuntu | `apt install` |
| **dpkg** | .deb | Debian/Ubuntu | `dpkg -i` |
| **dnf** | .rpm | Fedora/RHEL | `dnf install` |
| **yum** | .rpm | CentOS/RHEL (old) | `yum install` |
| **pacman** | .pkg.tar | Arch | `pacman -S` |
| **zypper** | .rpm | openSUSE | `zypper install` |
| **snap** | .snap | Ubuntu | `snap install` |
| **flatpak** | .flatpak | Universal | `flatpak install` |

---

## 2. APT — Advanced Package Tool

APT is Ubuntu's primary package manager.

### Basic Operations

```bash
# Update package list (always do this first!)
sudo apt update

# Upgrade installed packages
sudo apt upgrade                    # safe upgrades only
sudo apt full-upgrade               # includes removing old packages
sudo apt dist-upgrade               # same as full-upgrade

# Search for packages
apt search python3                  # search by name/description
apt-cache search editor             # older command, still works

# Show package information
apt show nginx                      # detailed package info
apt info nginx                      # same as show

# Install packages
sudo apt install nginx              # install single package
sudo apt install nginx curl wget    # install multiple
sudo apt install -y nginx           # non-interactive (assume yes)
sudo apt install --no-install-recommends nginx  # minimal install

# Remove packages
sudo apt remove nginx               # remove (keep config files)
sudo apt purge nginx                # remove + config files
sudo apt autoremove                 # remove unneeded dependencies
sudo apt autoremove --purge         # autoremove + purge configs

# Clean up
sudo apt autoclean                  # remove obsolete downloaded packages
sudo apt clean                      # remove all downloaded .deb files
```

### APT Output Explained

```bash
$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease         # already latest
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease # downloading index
Fetched 2,342 kB in 3s (781 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
34 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

### Package Listing

```bash
# List installed packages
apt list --installed                        # all installed
apt list --installed | grep nginx           # specific package
dpkg -l                                    # also shows installed

# List upgradable packages
apt list --upgradable

# Search what package provides a file
apt-file search /usr/bin/convert           # needs: sudo apt install apt-file
dpkg -S /usr/bin/python3                   # for installed packages

# List package contents
dpkg -L nginx                              # files installed by nginx
apt-file list nginx                        # all files (including not installed)
```

### APT Pinning and Holding

```bash
# Hold a package at current version (don't upgrade)
sudo apt-mark hold nginx
apt-mark showhold               # show held packages
sudo apt-mark unhold nginx      # unhold

# Prefer specific version
apt-cache policy nginx          # show available versions
sudo apt install nginx=1.18.0-6ubuntu14   # install specific version
```

### Sources and Repository Configuration

```bash
cat /etc/apt/sources.list       # main sources

# Sources format:
# deb [options] uri distribution component...
# deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
# ^   ^                                  ^     ^    ^                  ^
# type url                              release section (package categories)
```

| Component | Contains |
|-----------|---------|
| main | Officially supported free software |
| restricted | Supported but non-free (firmware) |
| universe | Community-maintained free software |
| multiverse | Non-free software |

---

## 3. dpkg — Debian Package Manager

`dpkg` is the low-level tool APT uses. Useful for `.deb` files.

```bash
# Install a .deb file
sudo dpkg -i package.deb

# Remove (keep config)
sudo dpkg -r package-name

# Purge (remove + config)
sudo dpkg -P package-name

# List all installed packages
dpkg -l                         # full list
dpkg -l | grep nginx            # grep for specific
dpkg -l "python*"               # wildcard

# Get package information
dpkg -s nginx                   # status/info of installed package
dpkg -p nginx                   # info from package database

# List files installed by a package
dpkg -L nginx                   # list all files
dpkg -L nginx | grep bin        # just binaries

# Find which package owns a file
dpkg -S /usr/bin/python3
dpkg -S /etc/nginx/nginx.conf

# Fix broken packages
sudo dpkg --configure -a        # configure unpacked packages
sudo apt -f install             # fix broken dependencies

# Extract .deb without installing
dpkg-deb -x package.deb /tmp/extracted/
dpkg-deb --info package.deb     # view package metadata
```

---

## 4. Snap Packages

Snaps are containerized applications that bundle all dependencies.

```bash
# Search
snap find spotify
snap search vlc

# Install
sudo snap install spotify
sudo snap install vlc
sudo snap install code --classic    # --classic = more host access

# List installed snaps
snap list

# Get info about a snap
snap info spotify

# Update snaps
sudo snap refresh                  # update all
sudo snap refresh spotify          # update specific

# Remove
sudo snap remove spotify

# Snap channels (like release branches)
sudo snap install --channel=beta vlc       # beta channel
sudo snap install --channel=edge code      # cutting edge

# Revert to previous version
sudo snap revert spotify

# Connect/disconnect interfaces (permissions)
snap connections spotify           # see connected interfaces
sudo snap connect spotify:audio-playback   # connect interface
sudo snap disconnect spotify:audio-playback

# Snap directory structure
ls ~/snap/                         # user data
ls /snap/                          # snap mount points
ls /var/snap/                      # snap state and data
```

### Snap vs APT Trade-offs

| Feature | APT | Snap |
|---------|-----|------|
| Speed | Faster | Slower (containerized) |
| Disk space | Less | More (bundled libs) |
| Auto-updates | Manual | Automatic |
| Isolation | Shared libs | Contained |
| Version choice | Limited | Multiple channels |
| CLI/daemon apps | Better | Limited |

---

## 5. Flatpak

Flatpak is another containerized package system, common outside Ubuntu.

```bash
# Install flatpak
sudo apt install flatpak
sudo apt install gnome-software-plugin-flatpak  # GUI support

# Add Flathub repository (main app source)
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Search
flatpak search vlc

# Install
flatpak install flathub org.videolan.VLC

# Run
flatpak run org.videolan.VLC

# List installed
flatpak list

# Update
flatpak update

# Remove
flatpak uninstall org.videolan.VLC
flatpak uninstall --unused       # remove unused runtimes
```

---

## 6. Adding Repositories and PPAs

### PPAs (Personal Package Archives)

PPAs let Ubuntu developers publish packages for Ubuntu users.

```bash
# Add a PPA
sudo add-apt-repository ppa:user/ppa-name
sudo apt update
sudo apt install package-from-ppa

# Example: Add git-core PPA (latest git)
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git

# Remove a PPA
sudo add-apt-repository --remove ppa:git-core/ppa
# Or: sudo ppa-purge ppa:user/ppa-name  (reverts packages to stock)

# List PPAs
ls /etc/apt/sources.list.d/
cat /etc/apt/sources.list.d/git-core-ubuntu-ppa-jammy.list
```

### Adding Third-Party Repositories

Many software vendors provide their own repositories:

```bash
# Example: Docker official repository
# 1. Add GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 2. Add repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 3. Update and install
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Example: Node.js from NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install nodejs
```

> 🔒 **Security Note**: Only add repositories from trusted sources. Always verify GPG keys before adding.

---

## 7. Compiling From Source

When a package isn't available in repositories, you can compile from source.

### Basic Build Process

```bash
# Install build tools
sudo apt install build-essential

# Install build-time dependencies
sudo apt build-dep package-name    # or install manually

# Basic source compilation
./configure                        # check dependencies, create Makefile
make                               # compile
sudo make install                  # install to system

# With options
./configure --prefix=/usr/local \  # install location
            --with-ssl \           # enable SSL feature
            --without-debug        # no debug symbols
make -j$(nproc)                   # parallel compilation (faster!)
sudo make install
```

### Example: Compiling a Simple Program

```bash
# Download source
wget https://example.com/app-1.0.tar.gz
tar -xzvf app-1.0.tar.gz
cd app-1.0/

# Read the README!
cat README
cat INSTALL

# Check dependencies
sudo apt install libssl-dev libpcre3-dev zlib1g-dev

# Configure
./configure --prefix=/usr/local

# Compile (using all CPU cores)
make -j$(nproc)

# Test (if available)
make test
make check

# Install
sudo make install

# Uninstall (if supported)
sudo make uninstall
```

### CMake Build System

```bash
# Many modern projects use CMake
cd project/
mkdir build && cd build/
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
sudo make install
```

---

## 8. Python Packages (pip)

```bash
# Install pip
sudo apt install python3-pip

# Install package
pip3 install requests               # user install
pip3 install --user requests        # explicit user install
sudo pip3 install requests          # system-wide (not recommended!)

# Virtual environments (RECOMMENDED!)
python3 -m venv ~/.venv/myproject    # create venv
source ~/.venv/myproject/bin/activate  # activate
pip install requests flask           # install in venv
deactivate                           # deactivate venv

# Install from requirements file
pip install -r requirements.txt

# List installed packages
pip list
pip freeze > requirements.txt       # save current packages

# Search packages
pip search requests                  # (often disabled, use pypi.org)

# Upgrade packages
pip install --upgrade requests
pip install -U requests              # same

# Uninstall
pip uninstall requests

# Check for security vulnerabilities
pip install safety
safety check
```

---

## 9. AppImage

AppImages are portable executables — no installation required:

```bash
# Download AppImage from vendor website
wget https://example.com/App-1.0-x86_64.AppImage

# Make executable
chmod +x App-1.0-x86_64.AppImage

# Run directly!
./App-1.0-x86_64.AppImage

# Optional: integrate with desktop
sudo apt install libfuse2          # if needed
./App-1.0-x86_64.AppImage --appimage-extract  # extract contents
```

---

## 10. Practice Exercises

### Exercise 9.1 — APT Mastery

```bash
# 1. Update package list
sudo apt update

# 2. How many packages can be upgraded?
apt list --upgradable | wc -l

# 3. Search for 'image editor' packages
apt search "image editor"

# 4. Get information about gimp package
apt show gimp

# 5. Install htop and tree
sudo apt install -y htop tree

# 6. Verify installation
which htop
htop --version

# 7. List files installed by htop
dpkg -L htop

# 8. Remove htop but keep config
sudo apt remove htop

# 9. Fully remove htop and config
sudo apt purge htop
sudo apt autoremove
```

### Exercise 9.2 — Build a Simple C Program From Source

```bash
# 1. Create a simple C program
cat << 'EOF' > /tmp/hello.c
#include <stdio.h>
int main() {
    printf("Hello from compiled source!\n");
    return 0;
}
EOF

# 2. Install build tools
sudo apt install -y build-essential

# 3. Compile
gcc -o /tmp/hello /tmp/hello.c

# 4. Run
/tmp/hello

# 5. See what it's linked against
ldd /tmp/hello

# 6. Check file type
file /tmp/hello
```

### Exercise 9.3 — Python Virtual Environment

```bash
# 1. Create a project directory and venv
mkdir ~/py_project && cd ~/py_project
python3 -m venv venv

# 2. Activate venv
source venv/bin/activate

# 3. Check Python path (should be inside venv)
which python

# 4. Install requests
pip install requests

# 5. Test it
python -c "import requests; print(requests.get('https://api.github.com').status_code)"

# 6. Save requirements
pip freeze > requirements.txt
cat requirements.txt

# 7. Deactivate
deactivate
```

---

## 11. Key Takeaways

- **`sudo apt update`** always before installing — refreshes package lists
- **`apt install`** for installation; **`apt remove`** keeps config; **`apt purge`** removes everything
- **`dpkg -i`** for manual `.deb` files; **`dpkg -L`** lists package files; **`dpkg -S`** finds package owner
- **Snap** and **Flatpak** provide sandboxed, auto-updating apps
- **PPAs** add third-party repositories — only add trusted sources!
- Always use **virtual environments** for Python projects (`python3 -m venv`)
- Compiling from source: `./configure → make → sudo make install`
- **`sudo apt autoremove`** regularly to clean up orphaned packages

---

## Next Lesson

➡️ [10 — Networking Basics](10_Networking_Basics.md)

---

*Module 2 · Lesson 3 of 4 | [Course Index](../INDEX.md)*
