# 07 — User and Permission Management

> **Module 2 · Lesson 1** | Difficulty: ★★★☆☆ Intermediate | Time: ~90 min

---

## Learning Objectives

- Understand Linux user and group model
- Create, modify, and delete users and groups
- Understand and set file permissions (rwx, octal, symbolic)
- Use chmod, chown, and chgrp
- Configure sudo and the sudoers file
- Understand special permissions (SUID, SGID, sticky bit)
- Use Access Control Lists (ACLs)

---

## Table of Contents

1. [Linux Users and Groups](#1-linux-users-and-groups)
2. [User Account Files](#2-user-account-files)
3. [Managing Users](#3-managing-users)
4. [Managing Groups](#4-managing-groups)
5. [File Permissions](#5-file-permissions)
6. [Changing Permissions](#6-changing-permissions)
7. [sudo and Privilege Escalation](#7-sudo-and-privilege-escalation)
8. [Special Permissions](#8-special-permissions)
9. [Access Control Lists (ACLs)](#9-access-control-lists-acls)
10. [Practice Exercises](#10-practice-exercises)
11. [Key Takeaways](#11-key-takeaways)

---

## 1. Linux Users and Groups

### Every Process Runs as a User

Linux is a multi-user system. **Every process runs with the privileges of a specific user**. This is the foundation of Linux security.

```
Types of users:
┌─────────────────────────────────────────────┐
│ root (UID 0)     — Superuser, unlimited power│
│ System users     — Run daemons/services      │
│   (UID 1-999)      (www-data, daemon, syslog)│
│ Regular users    — Human users              │
│   (UID 1000+)      (alice, bob, charlie)    │
└─────────────────────────────────────────────┘
```

```bash
# Your user info
id                    # uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo)
whoami                # alice
id alice              # info about a specific user
id -u                 # just your UID
id -g                 # just your primary GID
id -G                 # all group IDs
```

### UID and GID Ranges

| Range | Purpose |
|-------|---------|
| 0 | root |
| 1-99 | Statically allocated system accounts |
| 100-999 | Dynamically allocated system accounts |
| 1000-65533 | Regular users |
| 65534 | nobody (anonymous user) |

---

## 2. User Account Files

### `/etc/passwd` — User Information

```bash
cat /etc/passwd
# Format: username:password:UID:GID:comment:home:shell
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# alice:x:1000:1000:Alice Smith:/home/alice:/bin/bash
```

| Field | Meaning |
|-------|---------|
| username | Login name |
| password | `x` = in /etc/shadow; `*` = login disabled |
| UID | User ID number |
| GID | Primary group ID |
| comment | Full name or description (GECOS) |
| home | Home directory |
| shell | Login shell (`/usr/sbin/nologin` = no login) |

```bash
# Parse passwd file
cut -d: -f1 /etc/passwd | sort   # list all users
awk -F: '$3 >= 1000' /etc/passwd  # regular users only
awk -F: '$7 == "/bin/bash"' /etc/passwd  # users with bash shell
```

### `/etc/shadow` — Password Hashes (root only)

```bash
sudo cat /etc/shadow
# alice:$6$salt$hash:19732:0:99999:7:::
# Format: user:hash:lastchange:min:max:warn:inactive:expire:
```

| Field | Meaning |
|-------|---------|
| user | Username |
| hash | Password hash (`$6$` = SHA-512) |
| lastchange | Days since Jan 1, 1970 of last change |
| min | Min days between password changes |
| max | Max days password is valid |
| warn | Warn N days before expiry |
| inactive | Account disabled N days after expiry |
| expire | Account expiry date |

Password hash formats:
- `$1$` = MD5 (old, weak)
- `$5$` = SHA-256
- `$6$` = SHA-512 (current standard)
- `$y$` = yescrypt (Ubuntu 22.04+)
- `!` or `*` = login disabled

### `/etc/group` — Group Definitions

```bash
cat /etc/group
# Format: groupname:password:GID:members
# sudo:x:27:alice,bob
# docker:x:998:alice

# List your groups
groups
groups alice

# Show group members
getent group sudo        # members of sudo group
```

---

## 3. Managing Users

### Creating Users

```bash
# useradd — low level (minimal, no home dir by default)
sudo useradd username

# adduser — high level, interactive (Ubuntu recommended)
sudo adduser alice

# adduser with options
sudo adduser --home /home/alice \
             --shell /bin/bash \
             --gecos "Alice Smith" \
             alice

# useradd with all options specified
sudo useradd -m \               # create home directory
             -d /home/alice \   # home directory path
             -s /bin/bash \     # login shell
             -c "Alice Smith" \ # comment/full name
             -g alice \         # primary group
             -G sudo,docker \   # supplementary groups
             -u 1001 \          # specific UID
             alice
```

### Setting/Changing Passwords

```bash
# Set your own password
passwd

# Set another user's password (as root)
sudo passwd alice

# Set password to expire immediately (force change at next login)
sudo passwd -e alice

# Lock an account (prepend ! to hash)
sudo passwd -l alice

# Unlock an account
sudo passwd -u alice

# Check password status
sudo passwd -S alice
# alice P 2024-01-15 0 99999 7 -1
# P=usable, L=locked, NP=no password
```

### Modifying Users

```bash
# usermod — modify user account
sudo usermod -s /bin/zsh alice          # change shell
sudo usermod -c "Alice B. Smith" alice  # change comment
sudo usermod -d /home/newhome -m alice  # change home, move files
sudo usermod -l newname alice           # rename user
sudo usermod -u 1500 alice              # change UID

# Add user to supplementary group
sudo usermod -aG docker alice      # -a = append (don't remove existing!)
sudo usermod -aG sudo alice        # add to sudo group
sudo usermod -aG group1,group2 alice  # multiple groups

# Note: user must log out and back in for group changes to take effect
# Or: newgrp docker  (temporarily switch primary group)
```

### Deleting Users

```bash
# Remove user (keep home directory)
sudo userdel alice

# Remove user AND home directory
sudo userdel -r alice

# Remove user, home dir, and mail spool
sudo userdel -r -f alice    # -f = force even if logged in
```

### User Information Commands

```bash
finger alice              # detailed user info (if installed)
getent passwd alice       # look up user in NSS database
last alice                # login history
lastlog                   # last login for all users
lastlog -u alice          # last login for specific user
w                         # currently logged-in users + activity
who                       # who is logged in
```

---

## 4. Managing Groups

```bash
# Create group
sudo groupadd developers
sudo groupadd -g 2000 ops    # specific GID

# Add user to group
sudo usermod -aG developers alice
sudo gpasswd -a alice developers    # alternative

# Remove user from group
sudo gpasswd -d alice developers

# Delete group
sudo groupdel developers

# Modify group
sudo groupmod -n newname developers  # rename
sudo groupmod -g 2001 developers     # change GID

# Set group password (rare)
sudo gpasswd developers

# View group info
getent group developers
cat /etc/group | grep developers
```

---

## 5. File Permissions

### Permission Basics

```bash
$ ls -la
-rwxr-xr--  2  alice  developers  4096  Jan 15  script.sh
│││││││││    │    │        │         │       │      │
│││││││││    │    │        │         │       │      └── filename
│││││││││    │    │        │         │       └── date modified
│││││││││    │    │        │         └── file size
│││││││││    │    │        └── group owner
│││││││││    │    └── user (owner)
│││││││││    └── hard link count
│││││││││
│├┤├┤├┤│
│ │ │ │└── other permissions (r--)
│ │ │ └─── group permissions (r-x)
│ │ └───── user permissions  (rwx)
└─────────── file type (- = regular, d = dir, l = link)
```

### Permission Characters

| Character | File | Directory |
|-----------|------|-----------|
| `r` (read) | Read file content | List directory contents |
| `w` (write) | Modify file content | Create/delete files in dir |
| `x` (execute) | Execute as program | Enter (cd into) directory |
| `-` | Permission denied | Permission denied |

### Octal Notation

Each permission set (user/group/other) is a 3-bit binary number:

```
r w x   →  binary   →  octal
─────────────────────────────
0 0 0   →  000      →  0  (no permissions)
0 0 1   →  001      →  1  (execute only)
0 1 0   →  010      →  2  (write only)
0 1 1   →  011      →  3  (write + execute)
1 0 0   →  100      →  4  (read only)
1 0 1   →  101      →  5  (read + execute)
1 1 0   →  110      →  6  (read + write)
1 1 1   →  111      →  7  (read + write + execute)

Common permissions:
755 = rwxr-xr-x  (typical for programs/dirs)
644 = rw-r--r--  (typical for files)
600 = rw-------  (private files like SSH keys)
777 = rwxrwxrwx  (DANGEROUS — avoid!)
700 = rwx------  (private executable/directory)
```

---

## 6. Changing Permissions

### `chmod` — Change File Mode

```bash
# Octal notation
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 600 ~/.ssh/id_rsa  # rw------- (private key)
chmod 700 ~/.ssh/        # rwx------ (SSH directory)
chmod 777 public_dir/    # (AVOID — security risk)

# Symbolic notation: [who][+/-/=][permissions]
# who: u=user, g=group, o=others, a=all
chmod u+x script.sh       # add execute for user
chmod g-w file.txt        # remove write from group
chmod o=r file.txt        # set others to read-only
chmod a+r file.txt        # add read for all
chmod u+x,g-w file.txt    # multiple changes
chmod ug+rw,o-rwx file.txt  # complex change
chmod +x script.sh        # shorthand: add execute for all

# Recursive
chmod -R 755 directory/   # apply to all files in dir
chmod -R u+rwX,go+rX directory/  # X = execute only if dir/executable

# Copy permissions from another file
chmod --reference=file1.txt file2.txt
```

### `chown` — Change Ownership

```bash
# Change user owner
sudo chown alice file.txt
sudo chown alice directory/

# Change group
sudo chown :developers file.txt    # change group only
sudo chown alice:developers file.txt  # change both

# Recursive
sudo chown -R alice:alice /home/alice/

# Change without following symlinks
sudo chown -h alice symlink

# Examples
sudo chown www-data:www-data /var/www/html/   # web server files
sudo chown root:root /usr/local/bin/script    # system script
```

### `chgrp` — Change Group

```bash
sudo chgrp developers project/
sudo chgrp -R developers project/
```

### `umask` — Default Permission Mask

`umask` defines permissions that are **removed** from newly created files:

```bash
umask           # show current umask (e.g., 0022)
umask 027       # set umask

# Calculation:
# Files:   666 - umask = default permissions
# Dirs:    777 - umask = default permissions

# umask 022:
# Files: 666 - 022 = 644 (rw-r--r--)
# Dirs:  777 - 022 = 755 (rwxr-xr-x)

# umask 027 (more restrictive):
# Files: 666 - 027 = 640 (rw-r-----)
# Dirs:  777 - 027 = 750 (rwxr-x---)

# Set permanently in ~/.bashrc
echo "umask 027" >> ~/.bashrc
```

---

## 7. sudo and Privilege Escalation

### Understanding sudo

`sudo` (superuser do) allows a permitted user to run commands as root (or another user) without knowing the root password.

```bash
sudo command              # run as root
sudo -u bob command       # run as user 'bob'
sudo -l                   # list your sudo permissions
sudo -i                   # interactive root shell (login shell)
sudo -s                   # interactive root shell (current shell)
sudo su -                 # switch to root user
sudo !!                   # re-run last command with sudo
```

### The sudoers File

`/etc/sudoers` controls who can use sudo and what they can do.

```bash
# ALWAYS edit with visudo (validates syntax before saving)
sudo visudo

# Or edit distribution-specific include files
sudo visudo -f /etc/sudoers.d/myfile
```

### sudoers Syntax

```
# Format:
# user  host=(runas)  command

# Allow alice to run any command as root
alice   ALL=(ALL:ALL)   ALL

# Allow bob to run specific commands
bob     ALL=(root)      /usr/bin/apt, /sbin/reboot

# Allow the sudo group (no password required)
%sudo   ALL=(ALL:ALL)   NOPASSWD: ALL

# Allow alice to restart nginx without password
alice   ALL=(root)      NOPASSWD: /bin/systemctl restart nginx

# Allow developers group specific commands
%developers ALL=(root)  /usr/bin/apt update, /usr/bin/apt upgrade

# Include additional files
@includedir /etc/sudoers.d
```

### Common sudo Configurations

```bash
# Add user to sudo group (Ubuntu way)
sudo usermod -aG sudo alice

# Create custom sudoers file
sudo tee /etc/sudoers.d/alice << 'EOF'
# Allow alice to manage web services
alice ALL=(root) NOPASSWD: /bin/systemctl restart nginx
alice ALL=(root) NOPASSWD: /bin/systemctl restart apache2
alice ALL=(root) /usr/bin/certbot
EOF
sudo chmod 440 /etc/sudoers.d/alice

# Verify sudoers file
sudo visudo -c          # check for syntax errors
sudo -l -U alice        # see alice's sudo permissions
```

---

## 8. Special Permissions

### SUID (Set User ID) — 4xxx

When set on an **executable**, it runs as the file's **owner** (usually root) regardless of who executes it.

```bash
# Find SUID files
find / -perm -4000 2>/dev/null | sort

# Common SUID binaries
ls -la /usr/bin/passwd    # -rwsr-xr-x (s = SUID)
ls -la /usr/bin/sudo
ls -la /usr/bin/ping
ls -la /usr/bin/su

# Set SUID
chmod u+s program
chmod 4755 program        # rwsr-xr-x

# Why it works:
# /usr/bin/passwd needs to write to /etc/shadow (root-owned)
# SUID lets regular users change their own password via passwd
```

> ⚠️ **Security Risk**: SUID programs that can be abused allow privilege escalation. See Module 4!

### SGID (Set Group ID) — 2xxx

On **executables**: runs as the file's **group**.
On **directories**: new files inherit the directory's group.

```bash
# SGID on file
ls -la /usr/bin/wall     # -rwxr-sr-x (s = SGID)
chmod g+s program
chmod 2755 program       # rwxr-sr-x

# SGID on directory (very useful for shared directories)
mkdir /shared
sudo chown :developers /shared
sudo chmod 2775 /shared  # rwxrwsr-x
# All files created in /shared will belong to 'developers' group

ls -la /shared/newfile.txt
# -rw-rw-r-- alice developers ...  (inherits developers group!)
```

### Sticky Bit — 1xxx

On **directories**: users can only delete their **own** files, even if they have write permission.

```bash
ls -la /tmp/
# drwxrwxrwt  (t = sticky bit)
#        └── sticky bit (T = sticky without execute)

# Set sticky bit
chmod +t directory/
chmod 1777 directory/    # rwxrwxrwt

# Example: /tmp is world-writable but sticky
# alice can't delete bob's files in /tmp
# Only bob (or root) can delete bob's files
```

### Permission Summary Table

| Permission | Octal | On File | On Directory |
|-----------|-------|---------|-------------|
| r | 4 | Read content | List files |
| w | 2 | Write content | Create/delete files |
| x | 1 | Execute | cd into directory |
| SUID | 4000 | Runs as owner | Ignored |
| SGID | 2000 | Runs as group | Inherit group |
| Sticky | 1000 | Ignored | Protect files |

---

## 9. Access Control Lists (ACLs)

ACLs allow **more granular** permissions beyond the user/group/other model.

```bash
# Check if ACL support is available
mount | grep acl
# ext4 usually has ACL support by default

# Install ACL tools
sudo apt install acl

# View ACLs
getfacl file.txt
# file: file.txt
# owner: alice
# group: alice
# user::rw-
# group::r--
# other::r--

# Set ACL for specific user
setfacl -m u:bob:rw file.txt        # give bob read+write
setfacl -m u:charlie:r file.txt     # give charlie read only
setfacl -m g:developers:rx dir/     # give group read+execute

# Set default ACLs (inherited by new files)
setfacl -d -m g:developers:rwx shared_dir/   # default ACL

# Remove ACL
setfacl -x u:bob file.txt           # remove bob's ACL
setfacl -b file.txt                  # remove all ACLs

# Copy ACL from one file to another
getfacl source.txt | setfacl --set-file=- dest.txt
```

---

## 10. Practice Exercises

### Exercise 7.1 — User Management

```bash
# 1. Create user 'testuser' with home directory and bash shell
sudo adduser testuser

# 2. Check user information
id testuser
grep testuser /etc/passwd
sudo grep testuser /etc/shadow

# 3. Create group 'webteam'
sudo groupadd webteam

# 4. Add testuser to webteam
sudo usermod -aG webteam testuser

# 5. Verify
groups testuser
getent group webteam

# 6. Lock the account
sudo passwd -l testuser
sudo passwd -S testuser

# 7. Delete the user and home directory
sudo userdel -r testuser
```

### Exercise 7.2 — Permission Challenge

```bash
# Create test files
mkdir ~/perm_test
touch ~/perm_test/{public.txt,private.txt,script.sh}

# 1. Set public.txt to be readable by everyone
chmod 644 ~/perm_test/public.txt

# 2. Set private.txt to be readable only by you
chmod 600 ~/perm_test/private.txt

# 3. Make script.sh executable by you only
chmod 700 ~/perm_test/script.sh

# 4. Verify with ls -la
ls -la ~/perm_test/

# 5. What is the octal for rwxr-x---?
# Answer: 750

# 6. Set the directory so others cannot enter
chmod 700 ~/perm_test/
```

### Exercise 7.3 — sudo Configuration

```bash
# 1. Test your sudo access
sudo whoami  # should return: root

# 2. List your sudo privileges
sudo -l

# 3. Create a script that requires root
echo '#!/bin/bash
echo "Running as: $(whoami)"
cat /etc/shadow | head -3' > /tmp/test_root.sh
chmod +x /tmp/test_root.sh

# 4. Try running it normally (will fail)
/tmp/test_root.sh

# 5. Run with sudo
sudo /tmp/test_root.sh
```

---

## 11. Key Takeaways

- Linux has three types of users: **root** (UID 0), **system users** (1-999), **regular users** (1000+)
- **`/etc/passwd`** stores user info; **`/etc/shadow`** stores password hashes (root only)
- **`adduser`** is the interactive, recommended way to create users on Ubuntu
- Permissions: **r=4, w=2, x=1** for each of user/group/other
- **chmod 755** = rwxr-xr-x (typical program); **chmod 644** = rw-r--r-- (typical file)
- **SUID** (4xxx) = run as owner; **SGID** (2xxx) = inherit group; **Sticky** (1xxx) = protect files
- **`sudo visudo`** to safely edit sudoers; **`usermod -aG sudo`** to grant sudo access
- **ACLs** provide fine-grained permissions beyond user/group/other

---

## Next Lesson

➡️ [08 — Process Management](08_Process_Management.md)

---

*Module 2 · Lesson 1 of 4 | [Course Index](../INDEX.md)*
