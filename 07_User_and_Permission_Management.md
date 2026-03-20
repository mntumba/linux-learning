# 07 - User and Permission Management

**Difficulty:** Intermediate  
**Time Estimate:** 90–120 minutes  
**Prerequisites:** Lessons 01–06 (especially Terminal Basics and File System Structure)

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Understand how Linux identifies users and groups via UIDs and GIDs
- Read and interpret `/etc/passwd`, `/etc/shadow`, and `/etc/group`
- Create, modify, and delete users and groups
- Manage passwords and password policies with `passwd` and `chage`
- Configure `sudo` access using `visudo` and `/etc/sudoers`
- Interpret and modify file permissions using symbolic and octal notation
- Set and remove special permission bits: SUID, SGID, and sticky bit
- Use Access Control Lists (ACLs) for fine-grained permissions
- Configure `umask` to control default file permissions

---

## 1. The Linux User System

Linux is a multi-user operating system. Every process and file is owned by a user, and every user is identified by a **User ID (UID)** — a number that the kernel uses internally.

### UID Ranges (Typical Defaults)

| UID Range   | Type                          |
|-------------|-------------------------------|
| 0           | root (superuser)              |
| 1–99        | Reserved system accounts      |
| 100–999     | System/service accounts       |
| 1000+       | Regular human users           |

The **Group ID (GID)** works similarly. Every user belongs to a **primary group** and can belong to multiple **supplementary groups**.

```bash
# See your own UID, GID, and group memberships
id

# See another user's info
id alice

# Check who is currently logged in
whoami

# List all logged-in users
who
w
```

### The root User

`root` (UID 0) has unrestricted access to every file, process, and system call. You should avoid working as root directly — use `sudo` instead so that actions are logged and mistakes are harder to make systemwide.

---

## 2. The User Database Files

### /etc/passwd

Each line represents one user account with seven colon-separated fields:

```
username:password:UID:GID:GECOS:home_dir:shell
```

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
bob:x:1002:1002:Bob Jones:/home/bob:/bin/bash
```

The `x` in the password field means the actual (hashed) password is stored in `/etc/shadow`.

```bash
# View the passwd file
cat /etc/passwd

# Find a specific user entry
grep "^alice:" /etc/passwd

# Count total user accounts
wc -l /etc/passwd
```

### /etc/shadow

Contains hashed passwords and password policy settings. Only root (and the shadow group) can read it.

```
alice:$6$rounds=5000$salt$hashedpassword:19500:0:99999:7:::
```

Fields: `username : hashed_password : last_changed : min_days : max_days : warn_days : inactive : expire : reserved`

```bash
# View shadow file (requires root)
sudo cat /etc/shadow

# View a single user's shadow entry
sudo grep "^alice:" /etc/shadow
```

### /etc/group

```
groupname:password:GID:member1,member2,...
```

```
sudo:x:27:alice,bob
docker:x:999:alice
developers:x:1005:alice,carol,dave
```

```bash
# View group file
cat /etc/group

# Find groups a user belongs to
groups alice

# Same info via id
id alice
```

---

## 3. User Management Commands

### Creating Users

```bash
# Low-level: useradd (minimal, no prompts)
sudo useradd -m -s /bin/bash -c "Alice Smith" alice

# High-level: adduser (interactive, creates home dir, etc.)
sudo adduser bob

# Create a system account (no home dir, no login)
sudo useradd --system --no-create-home --shell /usr/sbin/nologin myservice

# Specify UID, GID, home dir explicitly
sudo useradd -u 1050 -g developers -m -d /home/carol -s /bin/bash carol
```

### Modifying Users

```bash
# Change login shell
sudo usermod -s /bin/zsh alice

# Add user to supplementary group (keep existing groups with -aG)
sudo usermod -aG docker alice
sudo usermod -aG sudo,docker alice

# Lock a user account
sudo usermod -L alice

# Unlock a user account
sudo usermod -U alice

# Change username
sudo usermod -l newname oldname

# Change home directory and move files
sudo usermod -m -d /home/newhome alice

# Set account expiry date
sudo usermod -e 2025-12-31 alice
```

### Deleting Users

```bash
# Remove user (keep home directory)
sudo userdel alice

# Remove user AND their home directory
sudo userdel -r alice

# Remove user and all files they own (careful!)
sudo deluser --remove-all-files alice
```

---

## 4. Password Management

### passwd

```bash
# Change your own password
passwd

# Root changes another user's password
sudo passwd alice

# Lock an account (prepends ! to hash in /etc/shadow)
sudo passwd -l alice

# Unlock an account
sudo passwd -u alice

# Expire a password immediately (force change on next login)
sudo passwd -e alice

# Show password status
sudo passwd -S alice
```

### chage — Password Aging

```bash
# Show password aging info
sudo chage -l alice

# Set maximum password age (days)
sudo chage -M 90 alice

# Set minimum days before password can be changed
sudo chage -m 7 alice

# Set warning days before expiry
sudo chage -W 14 alice

# Set account expiry date
sudo chage -E 2025-12-31 alice

# Force password change on next login
sudo chage -d 0 alice
```

---

## 5. Group Management

```bash
# Create a group
sudo groupadd developers

# Create a group with specific GID
sudo groupadd -g 2000 devops

# Delete a group
sudo groupdel developers

# Add a user to a group
sudo gpasswd -a alice developers

# Remove a user from a group
sudo gpasswd -d alice developers

# Set group administrator
sudo gpasswd -A alice developers

# List members of a group
getent group developers

# Show groups for current user
groups

# Show groups for another user
groups alice
```

---

## 6. sudo Configuration

### /etc/sudoers and visudo

**Always** edit `/etc/sudoers` with `visudo` — it validates syntax before saving, preventing lockouts.

```bash
# Open sudoers safely
sudo visudo

# Open sudoers for a specific drop-in file (preferred pattern)
sudo visudo -f /etc/sudoers.d/developers
```

### sudoers Syntax

```
# Allow alice to run any command as any user
alice ALL=(ALL:ALL) ALL

# Allow alice to run apt without a password
alice ALL=(ALL) NOPASSWD: /usr/bin/apt

# Allow the developers group to run all commands
%developers ALL=(ALL:ALL) ALL

# Allow bob to run systemctl restart apache2 only
bob ALL=(root) /bin/systemctl restart apache2

# Allow alice to become any user (not just root)
alice ALL=(ALL) ALL
```

### sudo Usage

```bash
# Run a single command as root
sudo apt update

# Run a command as another user
sudo -u bob ls /home/bob

# Open an interactive root shell (login shell)
sudo -i

# Open an interactive root shell (keep current environment)
sudo -s

# Run as another user with their environment
sudo -u alice -i

# List what sudo permissions you have
sudo -l

# Edit a file safely as root (uses $SUDO_EDITOR)
sudo -e /etc/hosts
```

---

## 7. The Permission System

### Permission Bits Visualized

```
  File Type  User (Owner)   Group        Other (World)
      |        |   |   |    |  |  |      |   |   |
      d        r   w   x    r  -  x      r   -   x
      |        |   |   |    |  |  |      |   |   |
      |        4   2   1    4  0  1      4   0   1
      |         \  |  /      \ | /        \ | /
      |          7 (rwx)      5 (r-x)      5 (r-x)
      |
  d = directory
  - = regular file
  l = symbolic link
  c = character device
  b = block device
  p = named pipe
  s = socket
```

```
Full permission string breakdown:
-rwxr-xr-x  1  alice  developers  4096  Jan 15 10:00  script.sh
│└┬┘└┬┘└┬┘  │    │        │        │
│ │  │  │   │    │        │      file size
│ │  │  │   │    │      group owner
│ │  │  │   │  user owner
│ │  │  │ hard link count
│ │  │ other permissions (r-x = 5)
│ │ group permissions (r-x = 5)
│ user permissions (rwx = 7)
file type (- = regular file)
```

### Octal Notation

| Octal | Binary | Permissions |
|-------|--------|-------------|
| 0     | 000    | ---         |
| 1     | 001    | --x         |
| 2     | 010    | -w-         |
| 3     | 011    | -wx         |
| 4     | 100    | r--         |
| 5     | 101    | r-x         |
| 6     | 110    | rw-         |
| 7     | 111    | rwx         |

**Common permission combinations:**

| Octal | Meaning                                 | Use Case                    |
|-------|-----------------------------------------|-----------------------------|
| 755   | rwxr-xr-x                               | Executable scripts, dirs    |
| 644   | rw-r--r--                               | Regular files, configs      |
| 700   | rwx------                               | Private scripts             |
| 600   | rw-------                               | Private files (SSH keys)    |
| 777   | rwxrwxrwx                               | Avoid! Too permissive        |
| 664   | rw-rw-r--                               | Group-writable files        |
| 750   | rwxr-x---                               | Group-executable scripts    |
| 400   | r--------                               | Read-only files             |

---

## 8. chmod — Changing Permissions

### Octal Mode

```bash
# Set exact permissions with octal
chmod 755 script.sh
chmod 644 config.txt
chmod 700 private_script.sh
chmod 600 ~/.ssh/id_rsa

# Recursive: apply to directory and all contents
chmod -R 755 /var/www/html

# Apply to files only (not dirs) using find
find /var/www -type f -exec chmod 644 {} \;
find /var/www -type d -exec chmod 755 {} \;
```

### Symbolic Mode

```bash
# u = user/owner, g = group, o = other, a = all
# + adds, - removes, = sets exactly

# Add execute for owner
chmod u+x script.sh

# Remove write from group and other
chmod go-w file.txt

# Set read-only for everyone
chmod a=r file.txt

# Add execute for everyone, remove write from other
chmod a+x,o-w script.sh

# Give group same permissions as owner
chmod g=u file.txt

# Set exact permissions symbolically
chmod u=rwx,g=rx,o=rx script.sh
```

---

## 9. chown and chgrp

```bash
# Change file owner
sudo chown alice file.txt

# Change owner and group simultaneously
sudo chown alice:developers file.txt

# Change only the group
sudo chown :developers file.txt
# or equivalently:
sudo chgrp developers file.txt

# Recursive ownership change
sudo chown -R alice:developers /var/www/mysite

# Change symlink ownership (not the target)
sudo chown -h alice symlink.txt

# Copy ownership from another file (reference)
sudo chown --reference=ref_file.txt target_file.txt
```

---

## 10. Special Permission Bits

### SUID (Set User ID) — Octal 4xxx

When set on an **executable**, it runs with the **file owner's** privileges, not the caller's.

```bash
# Classic example: /usr/bin/passwd (owned by root, SUID set)
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
#    ^ 's' here means SUID is set

# Set SUID on a file
chmod u+s /usr/local/bin/mytool
chmod 4755 /usr/local/bin/mytool

# Find all SUID files (security audit)
find / -perm -4000 -type f 2>/dev/null
```

### SGID (Set Group ID) — Octal 2xxx

- On an **executable**: runs with the **file's group** privileges
- On a **directory**: new files inherit the **directory's group** (very useful for shared dirs)

```bash
# Set SGID on a directory
chmod g+s /shared/project
chmod 2775 /shared/project

ls -ld /shared/project
# drwxrwsr-x 2 alice developers ... /shared/project
#        ^ 's' on the group execute bit = SGID

# Find all SGID files
find / -perm -2000 -type f 2>/dev/null
```

### Sticky Bit — Octal 1xxx

On a **directory**: users can only delete files they **own**, even if they have write permission on the directory. Classic use: `/tmp`.

```bash
# /tmp always has sticky bit
ls -ld /tmp
# drwxrwxrwt 20 root root ... /tmp
#          ^ 't' = sticky bit

# Set sticky bit
chmod +t /shared/uploads
chmod 1777 /shared/uploads

# Find directories with sticky bit
find / -type d -perm -1000 2>/dev/null
```

### Combined Special Bits

```bash
# SUID + SGID + sticky on a directory (unusual but possible)
chmod 7755 /special/dir

# Remove all special bits
chmod 0755 file.sh
```

---

## 11. Access Control Lists (ACLs)

Standard Unix permissions only allow one owner, one group. ACLs add fine-grained per-user/per-group permissions.

```bash
# Check if ACLs are supported (look for 'acl' mount option)
mount | grep acl
# Or check filesystem capabilities
tune2fs -l /dev/sda1 | grep "Default mount"

# View ACLs on a file
getfacl file.txt

# Set a specific user ACL
setfacl -m u:bob:rw file.txt

# Set a specific group ACL
setfacl -m g:contractors:r-- /shared/docs

# Set ACL recursively
setfacl -R -m u:bob:rx /shared/project

# Set a default ACL on a directory (inherited by new files)
setfacl -d -m u:bob:rw /shared/project
setfacl -d -m g:developers:rwx /shared/project

# Remove a specific user ACL
setfacl -x u:bob file.txt

# Remove ALL ACLs (back to standard permissions)
setfacl -b file.txt

# Copy ACLs from one file to another
getfacl source.txt | setfacl --set-file=- target.txt
```

---

## 12. umask — Default Permission Mask

`umask` defines which permissions are **removed** from newly created files and directories.

```
Default file permissions:  666 (rw-rw-rw-)
Default dir  permissions:  777 (rwxrwxrwx)
umask:                      022
---
Result for files:  666 - 022 = 644 (rw-r--r--)
Result for dirs:   777 - 022 = 755 (rwxr-xr-x)
```

```bash
# View current umask
umask        # Shows 0022
umask -S     # Shows u=rwx,g=rx,o=rx (symbolic)

# Set a stricter umask (private: only owner can read/write)
umask 077    # files=600, dirs=700

# Set a group-friendly umask
umask 002    # files=664, dirs=775

# Make permanent (add to ~/.bashrc or ~/.profile)
echo "umask 022" >> ~/.bashrc

# System-wide default is in:
cat /etc/login.defs | grep UMASK
```

---

## Common Mistakes

1. **Using `chmod 777` liberally** — This grants full access to everyone on the system. Almost never the right answer; use groups instead.
2. **Forgetting `-a` in `usermod -aG`** — Without `-a`, you *replace* all group memberships instead of appending.
3. **Editing `/etc/sudoers` directly** — Always use `visudo` to prevent syntax errors that could lock you out.
4. **Recursive `chown` on `/`** — Running `sudo chown -R alice /` by mistake can destroy your system. Always double-check recursive commands.
5. **Confusing SUID on directories** — SUID has no standard effect on directories (only SGID does). Don't confuse them.
6. **Not logging out after group changes** — `usermod -aG` changes take effect on next login. Run `newgrp docker` to apply immediately in the current session.
7. **Over-relying on ACLs** — ACLs are powerful but add complexity. Use them only when standard group permissions are insufficient.

---

## Pro Tips

- **Use `/etc/sudoers.d/` drop-in files** rather than editing the main sudoers file. Keep each user or service in its own file.
- **Audit SUID/SGID files regularly**: `find / -perm /6000 -type f 2>/dev/null` — unexpected SUID files can be a security risk.
- **`newgrp groupname`** — Temporarily switch your primary group in the current shell without logging out.
- **`sudo !!`** — Re-run your last command with sudo (in bash).
- **`stat file`** — Shows both symbolic and octal permissions, plus inode info. More detailed than `ls -l`.
- **`getent passwd alice`** — Works even for LDAP/NIS users, unlike directly reading `/etc/passwd`.
- **Password-less SSH** — For service accounts, consider disabling password auth entirely and using key-based auth instead.

---

## Practice Exercises

**Exercise 1 — User Creation**  
Create a user `testuser` with home directory `/home/testuser`, shell `/bin/bash`, and comment "Test User". Verify with `id testuser` and `grep testuser /etc/passwd`.

**Exercise 2 — Group Management**  
Create a group called `webteam`. Add `testuser` to `webteam` without removing any existing group memberships. Confirm with `groups testuser`.

**Exercise 3 — Password Policy**  
Set `testuser`'s password to expire after 60 days, with a 5-day minimum age and a 7-day warning period. Verify with `sudo chage -l testuser`.

**Exercise 4 — File Permissions**  
Create a file `secret.txt`. Set it so only the owner can read and write it. Create a directory `shared/`. Set it so the owner has full access, the group can read and execute, and others have no access.

**Exercise 5 — SGID Directory**  
Create `/shared/team` owned by root:webteam. Set SGID and permissions 2775. Create a file inside it as `testuser` and verify the file's group is `webteam`.

**Exercise 6 — sudo Configuration**  
Using `visudo`, grant `testuser` permission to run `apt update` and `apt upgrade` without a password. Test it with `sudo -l -U testuser`.

**Exercise 7 — ACLs**  
Create a file `report.txt` owned by root. Use `setfacl` to grant `testuser` read-only access without changing the file's standard permissions. Verify with `getfacl report.txt`.

**Exercise 8 — umask**  
Change your umask to `027`. Create a new file and directory and verify the permissions. Then restore your original umask.

---

## Key Takeaways

- Linux identifies users by **UID** (numeric), stored in `/etc/passwd`; passwords live in `/etc/shadow`
- Always use `visudo` to edit sudoers — syntax errors can lock you out
- **`usermod -aG`** appends groups; omitting `-a` replaces them
- Octal permissions: **owner | group | other**, each digit is a sum of r=4, w=2, x=1
- **SUID** runs executables as the file owner; **SGID** on directories makes new files inherit the directory's group; **sticky bit** prevents non-owners from deleting files
- **ACLs** (`setfacl`/`getfacl`) provide per-user/per-group permissions beyond the traditional three-category model
- **umask** subtracts from default permissions (666 for files, 777 for dirs)

---

## Next Lesson Preview

**Lesson 08 — Process Management**  
You've mastered *who* can do what. Next, you'll master *what's running* on your system. We'll explore process states, the `/proc` filesystem, job control (foreground/background), signals (`SIGTERM`, `SIGKILL`), process priority with `nice`/`renice`, and how to automate tasks with `cron` and `at`.
