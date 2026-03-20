# 09 - Package Management

**Difficulty:** Beginner–Intermediate  
**Time Estimate:** 60–90 minutes  
**Prerequisites:** Lessons 01–08 (especially Terminal Basics and Process Management)

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain what a package is and why package managers exist
- Use `apt` to install, update, remove, and search for software
- Use `dpkg` to manage `.deb` files directly
- Configure software repositories and add PPAs
- Install and manage Snap and Flatpak packages
- Compile and install software from source
- Manage package holds to prevent unwanted upgrades
- Understand package dependencies and how to resolve conflicts

---

## 1. What Is a Package Manager?

A **package** is a compressed archive containing a program's files, metadata, and installation instructions. A **package manager** automates:

- Downloading software from trusted repositories
- Verifying integrity (checksums, GPG signatures)
- Resolving and installing **dependencies** automatically
- Keeping an inventory of installed software
- Upgrading and removing software cleanly

```
┌─────────────────────────────────────────────────────────────┐
│                  PACKAGE MANAGEMENT LAYERS                  │
├─────────────────────────────────────────────────────────────┤
│  High-Level Tools    │  apt, apt-get, aptitude              │
│  (dependency-aware)  │  (handle deps, talk to dpkg)         │
├──────────────────────┼──────────────────────────────────────┤
│  Low-Level Tool      │  dpkg                                │
│  (no dep resolution) │  (installs/removes .deb files)       │
├──────────────────────┼──────────────────────────────────────┤
│  Package Format      │  .deb (Debian Package)               │
├──────────────────────┼──────────────────────────────────────┤
│  Repository          │  /etc/apt/sources.list               │
│  Configuration       │  /etc/apt/sources.list.d/            │
└──────────────────────┴──────────────────────────────────────┘
```

---

## 2. APT — Advanced Package Tool

`apt` is the modern, user-friendly frontend to Debian's package management system. It replaced `apt-get` and `apt-cache` for most day-to-day tasks (though `apt-get` is still preferred in scripts for its stable output format).

### Updating the Package Index

```bash
# Sync local package index with repositories (does NOT upgrade packages)
sudo apt update

# Verbose update (see each repository being checked)
sudo apt update -v

# Check what upgrades are available without installing
apt list --upgradable
```

### Installing Packages

```bash
# Install a single package
sudo apt install vim

# Install multiple packages at once
sudo apt install vim curl wget git

# Install a specific version
sudo apt install nginx=1.18.0-0ubuntu1

# Install without prompting for confirmation
sudo apt install -y htop

# Reinstall a package (useful if files are corrupted)
sudo apt install --reinstall vim

# Install a downloaded .deb file via apt (handles deps)
sudo apt install ./package.deb

# Only download, don't install
sudo apt install --download-only vim
# Downloaded to: /var/cache/apt/archives/
```

### Removing Packages

```bash
# Remove package but keep configuration files
sudo apt remove vim

# Remove package AND its configuration files (purge)
sudo apt purge vim
sudo apt remove --purge vim    # Equivalent

# Remove automatically installed dependencies no longer needed
sudo apt autoremove

# Purge removed packages that left config files behind
sudo apt purge $(dpkg -l | grep '^rc' | awk '{print $2}')

# Remove downloaded package archives from cache
sudo apt clean           # Remove all cached packages
sudo apt autoclean       # Remove only outdated cached packages
```

### Upgrading Packages

```bash
# Upgrade all upgradable packages (won't remove packages)
sudo apt upgrade

# Full upgrade (may remove packages to resolve conflicts)
sudo apt full-upgrade
sudo apt dist-upgrade    # Equivalent (older name)

# Upgrade a single package
sudo apt install --only-upgrade vim

# Simulate an upgrade without applying it
sudo apt upgrade --simulate
sudo apt upgrade -s      # Shorthand

# Update and upgrade in one line
sudo apt update && sudo apt upgrade -y
```

### Searching and Information

```bash
# Search for packages by name or description
apt search nginx
apt search "web server"

# Show detailed information about a package
apt show nginx
apt show nginx | grep -E "Version|Depends|Description"

# List installed packages
apt list --installed
apt list --installed 2>/dev/null | grep "python"

# List all available packages
apt list 2>/dev/null | wc -l

# List packages that can be upgraded
apt list --upgradable

# Show which package provides a specific file
apt-file search /usr/bin/vim     # Requires: sudo apt install apt-file
# Or use dpkg:
dpkg -S /usr/bin/vim

# Show files installed by a package
dpkg -L vim
```

---

## 3. dpkg — Debian Package Manager

`dpkg` is the low-level package tool. It doesn't resolve dependencies automatically, so it's usually used for inspecting packages or installing `.deb` files when `apt` isn't available.

```bash
# Install a .deb file
sudo dpkg -i package.deb

# Remove a package (keep config)
sudo dpkg -r packagename

# Remove a package and its config (purge)
sudo dpkg -P packagename

# List all installed packages
dpkg -l
dpkg -l | grep nginx

# Check if a specific package is installed
dpkg -l vim
dpkg -s vim           # More detailed status

# List files installed by a package
dpkg -L vim

# Find which package owns a file
dpkg -S /usr/bin/vim

# Extract a .deb without installing
dpkg -x package.deb /tmp/extracted/

# Fix broken dependencies after dpkg installs
sudo dpkg --configure -a
sudo apt install -f      # Alternative: fix broken deps

# Show the package database
cat /var/lib/dpkg/status | grep -A5 "^Package: vim"
```

### dpkg Status Codes (from dpkg -l)

```
ii = installed (desired=install, current=installed)
rc = removed but config files remain
un = unknown/not installed
hi = hold installed
```

---

## 4. Repository Configuration

### /etc/apt/sources.list

This file defines where `apt` downloads packages from.

```
# Format:
# deb [options] uri suite component1 [component2 ...]
# deb-src [options] uri suite component1 [component2 ...]

deb http://archive.ubuntu.com/ubuntu jammy main restricted
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted
deb http://archive.ubuntu.com/ubuntu jammy universe
deb http://archive.ubuntu.com/ubuntu jammy-security main restricted
```

- **deb** = binary packages; **deb-src** = source packages
- **suite** = distribution codename (e.g., `jammy`, `focal`, `bookworm`)
- **components**: `main` (free, supported), `restricted` (non-free drivers), `universe` (community), `multiverse` (non-free)

```bash
# View your sources.list
cat /etc/apt/sources.list

# View additional repository files
ls /etc/apt/sources.list.d/

# Check which repo a package comes from
apt-cache policy nginx
apt-cache policy vim
```

### Adding PPAs (Personal Package Archives)

PPAs are Ubuntu-specific repositories hosted on Launchpad.

```bash
# Install add-apt-repository tool
sudo apt install software-properties-common

# Add a PPA
sudo add-apt-repository ppa:deadsnakes/ppa
# This automatically: adds repo, fetches GPG key, runs apt update

# Manually add a PPA (equivalent to above)
echo "deb http://ppa.launchpad.net/deadsnakes/ppa/ubuntu jammy main" \
  | sudo tee /etc/apt/sources.list.d/deadsnakes-ppa.list

# Remove a PPA
sudo add-apt-repository --remove ppa:deadsnakes/ppa

# Update after adding repo
sudo apt update
sudo apt install python3.12
```

### Adding Third-Party Repositories

```bash
# Modern method: signed-by keyring (recommended)
# Example: Docker official repository

# Step 1: Import the GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

# Step 2: Add the repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu jammy stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list

# Step 3: Update and install
sudo apt update
sudo apt install docker-ce

# List all configured repositories
grep -r "^deb" /etc/apt/sources.list /etc/apt/sources.list.d/
```

---

## 5. Package Holds

Prevent a package from being upgraded automatically.

```bash
# Hold a package (won't be upgraded)
sudo apt-mark hold nginx
sudo apt-mark hold linux-image-generic

# Hold multiple packages
sudo apt-mark hold nginx php8.1 mysql-server

# Remove hold (allow upgrades again)
sudo apt-mark unhold nginx

# Show all held packages
apt-mark showhold
dpkg --get-selections | grep hold

# Hold via dpkg
echo "nginx hold" | sudo dpkg --set-selections
```

---

## 6. Snap Packages

Snaps are self-contained packages developed by Canonical that include all dependencies. They run in a sandbox and update automatically.

```bash
# Check if snapd is installed
snap version

# Install snapd (if needed)
sudo apt install snapd

# Search for a snap
snap find vlc
snap find "code editor"

# Install a snap
sudo snap install vlc
sudo snap install code --classic      # --classic: less sandboxed
sudo snap install discord --beta      # Install from beta channel

# List installed snaps
snap list

# Update a specific snap
sudo snap refresh vlc

# Update all snaps
sudo snap refresh

# Remove a snap
sudo snap remove vlc
sudo snap remove vlc --purge          # Also remove saved data

# Show info about a snap
snap info vlc

# Show snap connections (permissions)
snap connections vlc

# Enable/disable a snap (without removing)
sudo snap disable vlc
sudo snap enable vlc

# Revert a snap to previous version
sudo snap revert vlc

# View snap logs
snap logs vlc
```

### Snap Channels

```
stable → candidate → beta → edge
           (increasingly unstable / bleeding-edge)
```

```bash
# Install from a specific channel
sudo snap install code --channel=latest/stable
sudo snap install code --edge

# Switch a snap's channel
sudo snap switch --channel=beta vlc
sudo snap refresh vlc
```

---

## 7. Flatpak Packages

Flatpak is a distribution-agnostic packaging system with strong sandboxing, commonly used for desktop applications.

```bash
# Install flatpak
sudo apt install flatpak

# Add Flathub repository (main source of Flatpak apps)
flatpak remote-add --if-not-exists flathub \
  https://flathub.org/repo/flathub.flatpakrepo

# Reboot or re-login for Flatpak to fully integrate

# Search for apps
flatpak search vlc
flatpak search "video player"

# Install an app
flatpak install flathub org.videolan.VLC
flatpak install flathub com.spotify.Client

# Run a Flatpak app
flatpak run org.videolan.VLC

# List installed Flatpak apps
flatpak list
flatpak list --app     # Only applications (not runtimes)

# Update all Flatpak apps
flatpak update

# Update a specific app
flatpak update org.videolan.VLC

# Remove a Flatpak app
flatpak uninstall org.videolan.VLC
flatpak uninstall --unused    # Remove unused runtimes

# Show info about an installed app
flatpak info org.videolan.VLC

# Show Flatpak permissions for an app
flatpak info --show-permissions org.videolan.VLC

# Grant/revoke filesystem access
flatpak override org.videolan.VLC --filesystem=home
flatpak override org.videolan.VLC --nofilesystem=home
```

---

## 8. Compiling from Source

Sometimes software isn't available as a package, or you need a specific version or custom compile options.

```bash
# Step 0: Install build essentials
sudo apt install build-essential
sudo apt install build-essential autoconf automake libtool pkg-config

# Step 1: Download and extract source code
wget https://example.com/software-1.0.tar.gz
tar -xzf software-1.0.tar.gz
cd software-1.0/

# Step 2: Read the README and INSTALL files
cat README
cat INSTALL

# Step 3: Check dependencies and configure the build
./configure
./configure --prefix=/usr/local       # Install to /usr/local instead of /usr
./configure --prefix=/opt/myapp       # Install to a custom location
./configure --enable-feature          # Enable optional features
./configure --disable-feature         # Disable optional features
./configure --with-library=/path      # Specify library path
./configure --help                    # Show all available options

# Step 4: Compile (use -j for parallel jobs — speeds up compilation)
make
make -j$(nproc)        # Use all CPU cores
make -j4               # Use 4 cores

# Step 5: Run tests (optional but recommended)
make check
make test

# Step 6: Install
sudo make install

# --- Uninstalling source-compiled software ---
# If the Makefile supports it:
sudo make uninstall

# Or track installed files with checkinstall (creates a .deb):
sudo apt install checkinstall
./configure && make
sudo checkinstall         # Creates .deb, installs it, and tracks files
```

### Using cmake

```bash
# For projects using CMake build system
sudo apt install cmake

# Out-of-tree build (recommended)
mkdir build && cd build
cmake ..
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
cmake .. -DCMAKE_BUILD_TYPE=Release

make -j$(nproc)
sudo make install
```

---

## 9. Finding What Provides a File

```bash
# Which installed package owns a file?
dpkg -S /usr/bin/curl

# Which package provides a command? (even if not installed)
sudo apt install apt-file
sudo apt-file update
apt-file search vim

# Alternative: command-not-found (usually built in to Ubuntu)
# Type a missing command and it suggests the package to install
htop      # If not installed: "Command 'htop' not found, but can be installed with: sudo apt install htop"

# Show package dependencies
apt-cache depends nginx
apt-cache rdepends nginx   # What depends ON nginx (reverse deps)
apt-cache showpkg nginx    # Full package info including deps
```

---

## 10. Useful apt Commands Reference

```bash
# Full system update in one line
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt clean

# List recently installed packages
grep " install " /var/log/apt/history.log | tail -20

# Show apt history
cat /var/log/apt/history.log

# Check apt log for errors
cat /var/log/apt/term.log | tail -50

# Verify package integrity
sudo debsums -s nginx    # Requires: sudo apt install debsums

# Download source code of a package
apt-get source vim       # Downloads to current directory
sudo apt-get build-dep vim  # Install build dependencies for vim

# Show package changelog
apt-get changelog vim

# Check disk space used by apt cache
du -sh /var/cache/apt/archives/
```

---

## Common Mistakes

1. **Forgetting `apt update` before installing** — The local package index can be stale. Always run `sudo apt update` first, or you may get "package not found" errors or install old versions.
2. **Using `apt-get` in scripts but `apt` interactively** — `apt`'s output format may change between versions. Use `apt-get` (stable output) in scripts.
3. **`apt remove` vs `apt purge`** — `remove` leaves config files behind; `purge` removes everything. Use `purge` for a clean uninstall.
4. **Not running `sudo apt autoremove`** — Installing and removing packages leaves orphaned dependencies. Run `autoremove` regularly.
5. **Installing unverified PPAs** — PPAs can contain malicious code and run as root. Only add PPAs from trusted sources.
6. **Compiling without `checkinstall`** — Installing with `make install` leaves no dpkg record, making it hard to uninstall or upgrade later. Use `checkinstall` to create a proper `.deb`.
7. **Mixing Snap and apt versions** — Having the same app from both Snap and apt can cause conflicts. Pick one source.

---

## Pro Tips

- **`apt list --installed 2>/dev/null | grep -v automatic`** — Show only manually installed packages.
- **`aptitude`** — A more powerful TUI package manager: `sudo apt install aptitude && sudo aptitude`.
- **`apt-cache madison nginx`** — Show all available versions of `nginx` from all repos.
- **`DEBIAN_FRONTEND=noninteractive`** — Suppress interactive prompts in scripts (e.g., during Docker builds): `DEBIAN_FRONTEND=noninteractive sudo apt install -y tzdata`.
- **`apt-mark auto/manual`** — Mark packages as automatically or manually installed to affect `autoremove` behavior.
- **Snap vs Flatpak vs apt**: Prefer `apt` for server software and system tools. Use Snap or Flatpak for desktop GUI apps that need newer versions than the OS repos provide.
- **`/var/cache/apt/archives/`** — Packages are cached here after download. You can reinstall from cache offline.

---

## Practice Exercises

**Exercise 1 — Basic Installation**  
Run `sudo apt update`. Install `tree`, `htop`, and `curl` in a single command. Verify each is installed using `which`. Then check their installed versions with `apt show`.

**Exercise 2 — Searching and Information**  
Search for packages related to "disk usage". Pick one you haven't installed and view its full info with `apt show`. Note its dependencies and size.

**Exercise 3 — Repository Exploration**  
View `/etc/apt/sources.list` and list all files in `/etc/apt/sources.list.d/`. Use `apt-cache policy curl` to see which repository your version of `curl` comes from.

**Exercise 4 — dpkg Inspection**  
Use `dpkg -l` to list all installed packages. Count them. Use `dpkg -L vim` (or another installed package) to list all files it installed. Use `dpkg -S /bin/ls` to find which package owns `ls`.

**Exercise 5 — Removing and Purging**  
Install `cowsay`. Verify it works: `cowsay "Hello Linux!"`. Remove it with `apt remove`, then verify config remains with `dpkg -l | grep cowsay`. Purge it completely and verify.

**Exercise 6 — Package Hold**  
Install `nano`. Place a hold on it with `apt-mark hold`. Verify the hold is set with `apt-mark showhold`. Attempt `sudo apt upgrade` and confirm nano is excluded. Then unhold it.

**Exercise 7 — Snap Exploration**  
Install the `hello-world` snap: `sudo snap install hello-world`. Run it. List your snaps. Then remove it. Check the snap store for an app you're interested in using `snap find`.

**Exercise 8 — Build from Source (Simulation)**  
Install `build-essential`. Download `hello-2.12.tar.gz` from GNU: `wget https://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz`. Extract it, run `./configure`, `make`, then `sudo make install`. Verify with `hello`. Then uninstall with `sudo make uninstall`.

---

## Key Takeaways

- **`apt`** is the high-level tool for daily use; **`dpkg`** is the low-level engine underneath
- Always run **`sudo apt update`** before installing to sync the package index
- **`apt remove`** keeps config files; **`apt purge`** removes everything
- Repositories are configured in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/`
- **PPAs** provide newer package versions but should only be trusted if the source is reputable
- **Snap** (Canonical) and **Flatpak** (cross-distro) are containerised package formats for desktop apps
- Compile from source only when necessary; use `checkinstall` to keep dpkg informed
- **`apt-mark hold`** prevents specific packages from being upgraded automatically

---

## Next Lesson Preview

**Lesson 10 — Networking Basics**  
With your system fully managed, it's time to connect to the world. We'll cover the OSI and TCP/IP models, IP addressing, key network tools (`ping`, `traceroute`, `ss`, `dig`), SSH key-based authentication, firewall configuration with `ufw`, and network management with `nmcli`.
