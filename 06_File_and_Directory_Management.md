# 06 — File and Directory Management

> **Difficulty:** Intermediate | **Time Estimate:** 120–150 minutes

---

## Learning Objectives

By the end of this lesson you will be able to:

- View and paginate file contents using multiple tools
- Search for files and text patterns with `find` and `grep`
- Transform and manipulate text with `sed`, `awk`, `cut`, `sort`, and `uniq`
- Create and extract archives with `tar`, `gzip`, and `zip`
- Set and interpret file permissions using symbolic and octal notation
- Change file ownership with `chown` and `chgrp`
- Create symbolic and hard links
- Verify file integrity with checksums

---

## 1. Viewing File Contents

### `cat` — Concatenate and Print

```bash
# Print file contents
cat /etc/os-release

# Print with line numbers
cat -n /etc/passwd

# Print with non-printing characters shown
cat -A /etc/hosts      # Shows ^I for tabs, $ for line ends

# Concatenate multiple files
cat file1.txt file2.txt > combined.txt

# Create a file with content (here-doc)
cat > /tmp/sample.txt << 'EOF'
Line one
Line two
Line three
EOF
```

### `less` — Interactive File Viewer

`less` is the preferred pager for reading long files. It does not load the entire file into memory.

```bash
less /var/log/syslog

# Inside less:
# ↑ ↓           — scroll one line
# Space / b     — page forward / backward
# g / G         — jump to start / end
# /pattern      — search forward
# ?pattern      — search backward
# n / N         — next / previous search match
# q             — quit
# F             — follow mode (like tail -f)
# &pattern      — show only matching lines
```

### `head` and `tail`

```bash
# First N lines
head /etc/passwd              # First 10 (default)
head -20 /var/log/syslog      # First 20 lines
head -c 100 /bin/ls           # First 100 bytes

# Last N lines
tail /var/log/syslog          # Last 10 (default)
tail -50 /var/log/syslog      # Last 50 lines
tail -c 100 /bin/ls           # Last 100 bytes

# Follow a file in real time (watch it grow)
tail -f /var/log/syslog

# Follow multiple files simultaneously
tail -f /var/log/syslog /var/log/auth.log

# Lines 20 through 30 of a file (skip first 19, take 11)
head -30 /etc/passwd | tail -11
# Or with sed:
sed -n '20,30p' /etc/passwd
```

### `file` — Identify File Type

```bash
file /bin/ls               # ELF 64-bit executable
file /etc/passwd           # ASCII text
file /usr/share/pixmaps/*.png 2>/dev/null | head -5  # PNG image data
file /var/log/syslog       # ASCII text (may be gzip compressed)
file /dev/null             # character special (1/3)

# Check a batch of files
file /etc/* | grep "ASCII" | head -5
```

### `stat` — Detailed File Metadata

```bash
stat /etc/passwd
# Output includes:
# File: /etc/passwd
# Size: 2847         Blocks: 8       IO Block: 4096  regular file
# Device: fd01h/64769d  Inode: 917514  Links: 1
# Access: (0644/-rw-r--r--)  Uid: (0/root)   Gid: (0/root)
# Access: 2024-01-15 09:23:11.000000000 +0000
# Modify: 2024-01-10 14:30:05.000000000 +0000
# Change: 2024-01-10 14:30:05.000000000 +0000
# Birth:  2024-01-01 00:00:00.000000000 +0000

# Show only specific info
stat -c "%n %s %U %G %A" /etc/passwd
# /etc/passwd 2847 root root -rw-r--r--

# Format: filename, size in bytes, owner, group, permissions
```

---

## 2. Searching for Files

### `find` — Find Files by Any Criteria

`find` is the most powerful file-search tool in Linux.

```bash
# Basic syntax: find [path] [criteria] [action]

# Find by name
find /etc -name "*.conf"           # All .conf files in /etc
find ~ -name "*.txt"               # All .txt files in home
find / -name "passwd" 2>/dev/null  # Any file named passwd

# Case-insensitive name search
find /etc -iname "*.CONF"

# Find by type
find /tmp -type f                  # Regular files only
find /tmp -type d                  # Directories only
find /dev -type b                  # Block devices
find /run -type s                  # Sockets
find ~ -type l                     # Symbolic links

# Find by size
find / -size +100M 2>/dev/null     # Files larger than 100 MB
find /var -size +10M -size -50M 2>/dev/null  # Between 10-50 MB
find ~ -empty                      # Empty files and directories

# Find by modification time
find /etc -mtime -7                # Modified in last 7 days
find /var/log -mtime +30           # Modified more than 30 days ago
find /tmp -atime +1                # Accessed more than 1 day ago
find /home -newer /etc/passwd      # Modified more recently than /etc/passwd

# Find by owner and permissions
find /var/log -user root           # Owned by root
find /tmp -perm 777                # Exactly 777 permissions
find /etc -perm -u+w               # Owner-writable files
find /usr/bin -perm /4000          # SUID files

# Combine criteria with AND (default), OR (-o), NOT (!)
find /etc -name "*.conf" -user root          # AND: conf files owned by root
find /tmp -name "*.tmp" -o -name "*.bak"    # OR: tmp or bak files
find ~ ! -name "*.txt"                       # NOT: everything except .txt

# Execute actions on found files
find /tmp -name "*.tmp" -delete              # Delete found files
find /var/log -name "*.log" -exec ls -lh {} \;   # Run ls on each
find . -name "*.py" -exec grep -l "import os" {} \;  # Find Python files importing os
find . -name "*.txt" -exec wc -l {} +        # Count lines (batch mode, faster)

# Find and copy (xargs method — faster for many files)
find /etc -name "*.conf" | xargs -I{} cp {} /tmp/conf_backup/
```

### `locate` — Fast File Search (Uses Database)

```bash
# Install and update the locate database
sudo apt install mlocate -y
sudo updatedb

# Find files instantly
locate passwd
locate "*.conf" | head -10
locate -i "readme"      # Case-insensitive

# Update the database (run after creating new files)
sudo updatedb

# Limit results
locate nginx | head -5
```

### `which` and `whereis`

```bash
# Find the path of an executable
which python3          # /usr/bin/python3
which vim              # /usr/bin/vim
which nonexistent      # (no output if not found)

# Find binary, manual, and source locations
whereis python3        # python3: /usr/bin/python3 /usr/share/man/man1/python3.1.gz
whereis vim
whereis -b bash        # Binary location only

# type is even more comprehensive
type ls                # ls is aliased to 'ls --color=auto'
type cd                # cd is a shell builtin
type python3           # python3 is /usr/bin/python3
```

### `grep` — Search Text Patterns

```bash
# Basic syntax: grep [options] pattern [files]

# Simple search
grep "root" /etc/passwd
grep "error" /var/log/syslog | head -5

# Case-insensitive
grep -i "error" /var/log/syslog | head -5

# Invert match (lines NOT matching)
grep -v "^#" /etc/hosts          # Exclude comments

# Show line numbers
grep -n "Port" /etc/ssh/sshd_config 2>/dev/null

# Count matches
grep -c "Failed" /var/log/auth.log 2>/dev/null

# Show filename only (useful with multiple files)
grep -l "root" /etc/passwd /etc/shadow 2>/dev/null

# Recursive search in directory
grep -r "TODO" ~/projects/ 2>/dev/null | head -10

# Extended regex (-E) for patterns like + ? | ()
grep -E "^[0-9]{1,3}\.[0-9]{1,3}" /etc/hosts

# Fixed string (no regex, faster)
grep -F "127.0.0.1" /etc/hosts

# Show context around matches
grep -A 2 "error" /var/log/syslog 2>/dev/null | head -15   # 2 lines After
grep -B 2 "error" /var/log/syslog 2>/dev/null | head -15   # 2 lines Before
grep -C 2 "error" /var/log/syslog 2>/dev/null | head -20   # 2 lines Context

# Extract only the matching part (-o)
grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /etc/hosts

# Colour output (usually default on Ubuntu)
grep --color=always "root" /etc/passwd
```

---

## 3. Text Transformation

### `sed` — Stream Editor

`sed` processes text line by line — ideal for substitution, deletion, and insertion.

```bash
# Substitution: s/old/new/
echo "Hello World" | sed 's/World/Linux/'

# Global substitution (all occurrences on each line): /g flag
echo "aaa bbb aaa" | sed 's/aaa/xxx/g'

# Case-insensitive substitution: /I flag
echo "Hello WORLD" | sed 's/world/Linux/I'

# Delete lines matching pattern: d
cat /etc/hosts | sed '/^#/d'           # Delete comment lines
cat /etc/hosts | sed '/^$/d'           # Delete empty lines

# Print specific lines: p (use with -n to suppress default output)
sed -n '5,10p' /etc/passwd             # Print lines 5 to 10
sed -n '/root/p' /etc/passwd           # Print lines containing "root"

# Insert before a line: i
echo "line2" | sed 'i\line1'           # Insert line1 before line2

# Append after a line: a
echo "line1" | sed 'a\line2'           # Append line2 after line1

# Edit a file in-place: -i flag
sed -i 's/old_text/new_text/g' file.txt

# Backup while editing in-place: -i.bak
sed -i.bak 's/old_text/new_text/g' file.txt
# Creates file.txt.bak as backup

# Delete lines 3 to 7
sed '3,7d' /etc/passwd | head -10

# Multiple operations with -e
echo "Hello World" | sed -e 's/Hello/Hi/' -e 's/World/Linux/'
```

### `awk` — Pattern Scanning and Processing

`awk` is a powerful text-processing language, especially for columnar data.

```bash
# Print specific columns (fields separated by whitespace by default)
awk '{print $1}' /etc/passwd           # Print first field... but /etc/passwd uses :
awk -F: '{print $1}' /etc/passwd       # -F: sets colon as field separator
awk -F: '{print $1, $3}' /etc/passwd   # Print username and UID

# Print all users with UID >= 1000 (human users)
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd

# Compute sum of a column
ps aux | awk '{sum += $3} END {print "Total CPU:", sum "%"}'

# Count lines matching a pattern
awk '/error/{count++} END {print count}' /var/log/syslog 2>/dev/null

# Print line numbers with content
awk '{print NR": "$0}' /etc/hosts

# Format output like printf
awk -F: '{printf "User: %-20s UID: %d\n", $1, $3}' /etc/passwd | head -5

# Process CSV-like data
echo "Alice,30,Engineer
Bob,25,Designer
Carol,35,Manager" | awk -F, '$2 > 28 {print $1, "is over 28"}'

# BEGIN and END blocks
awk 'BEGIN{print "Users:"} -F: {print $1} END{print "Done"}' /etc/passwd | head -5
awk -F: 'BEGIN{print "=== Users ==="} {print $1} END{print "=== End ==="}' /etc/passwd | head -7
```

### `cut` — Extract Columns

```bash
# Cut by delimiter (-d) and field (-f)
cut -d: -f1 /etc/passwd                    # Usernames only
cut -d: -f1,3 /etc/passwd                  # Username and UID
cut -d: -f1-3 /etc/passwd                  # Fields 1 through 3

# Cut by character position
echo "Hello World" | cut -c1-5             # Characters 1-5: Hello
echo "Hello World" | cut -c7-             # From character 7 to end: World

# Practical example: get IP address from hostname output
hostname -I | cut -d' ' -f1               # First IP address
```

### `sort` — Sort Lines

```bash
# Basic sort (alphabetical)
sort /etc/passwd | head -5

# Reverse sort
sort -r /etc/passwd | head -5

# Numeric sort (important! "10" sorts before "9" alphabetically)
echo -e "10\n2\n20\n1\n15" | sort         # Wrong: alphabetical
echo -e "10\n2\n20\n1\n15" | sort -n      # Correct: numeric

# Sort by column (-k)
awk -F: '{print $1, $3}' /etc/passwd | sort -k2 -n | head -5

# Sort by file size (human-readable)
du -sh /var/log/* 2>/dev/null | sort -h | tail -5

# Sort and remove duplicates
sort -u list.txt

# Sort by multiple fields
sort -t: -k3,3n -k1,1 /etc/passwd | head -5  # Sort by UID then username
```

### `uniq` — Filter Duplicate Lines

```bash
# Remove consecutive duplicate lines (must sort first!)
echo -e "apple\napple\nbanana\napple" | sort | uniq

# Count occurrences
echo -e "apple\napple\nbanana\napple" | sort | uniq -c

# Show only duplicated lines
echo -e "apple\nbanana\napple\ncherry" | sort | uniq -d

# Show only unique (non-duplicated) lines
echo -e "apple\nbanana\napple\ncherry" | sort | uniq -u

# Practical: find the most common errors in a log
grep "error\|Error\|ERROR" /var/log/syslog 2>/dev/null \
  | awk '{print $5}' | sort | uniq -c | sort -rn | head -10
```

### `wc` — Word Count

```bash
wc /etc/passwd            # Lines, words, bytes
wc -l /etc/passwd         # Lines only
wc -w /etc/passwd         # Words only
wc -c /etc/passwd         # Bytes only
wc -m /etc/passwd         # Characters only

# Count files in a directory
ls /usr/bin | wc -l

# Measure pipeline output
cat /var/log/syslog 2>/dev/null | grep "error" | wc -l
```

### `diff` — Compare Files

```bash
# Create two test files
echo -e "line1\nline2\nline3" > /tmp/file_a.txt
echo -e "line1\nLINE2\nline3\nline4" > /tmp/file_b.txt

# Compare them
diff /tmp/file_a.txt /tmp/file_b.txt
# < indicates lines in file_a
# > indicates lines in file_b

# Side-by-side comparison
diff -y /tmp/file_a.txt /tmp/file_b.txt

# Unified format (used in patch files and git)
diff -u /tmp/file_a.txt /tmp/file_b.txt

# Ignore whitespace
diff -w /tmp/file_a.txt /tmp/file_b.txt

# Recursive directory comparison
diff -r /etc/ssh /tmp/ssh_backup 2>/dev/null | head -10

rm /tmp/file_a.txt /tmp/file_b.txt
```

---

## 4. Archives and Compression

### `tar` — Tape Archive (Most Common)

```bash
# Create archive (no compression)
tar -cvf archive.tar /path/to/directory/

# Create gzip-compressed archive (.tar.gz or .tgz)
tar -czvf archive.tar.gz /etc/ssh/

# Create bzip2-compressed archive (.tar.bz2) — smaller but slower
tar -cjvf archive.tar.bz2 /etc/ssh/

# Create xz-compressed archive (.tar.xz) — best compression
tar -cJvf archive.tar.xz /etc/ssh/

# List contents of archive (without extracting)
tar -tvf archive.tar.gz

# Extract archive (to current directory)
tar -xvf archive.tar.gz

# Extract to specific directory
tar -xvf archive.tar.gz -C /tmp/extracted/

# Extract single file from archive
tar -xvf archive.tar.gz etc/ssh/sshd_config

# Tar flags mnemonic: "Create/eXtract, Verbose, File"
# c = create, x = extract, t = list
# v = verbose, f = file (always needed), z = gzip, j = bzip2, J = xz
```

### `gzip` / `gunzip` — GNU Compression

```bash
# Compress a file (replaces original with .gz)
gzip file.txt           # Creates file.txt.gz, removes file.txt

# Compress keeping original
gzip -k file.txt

# Decompress
gunzip file.txt.gz      # Restores file.txt, removes file.txt.gz

# Decompress keeping .gz file
gunzip -k file.txt.gz

# Compress with maximum compression (-9 = best, -1 = fastest)
gzip -9 large_file.txt
gzip -1 log_file.txt

# View contents of gzipped file without extracting
zcat file.txt.gz
zless file.txt.gz
zgrep "error" file.txt.gz
```

### `zip` / `unzip` — ZIP Format (Windows-compatible)

```bash
# Create a zip archive
zip archive.zip file1.txt file2.txt
zip -r archive.zip directory/      # Recursive (include directory)

# Add compression level (0 = no compression, 9 = maximum)
zip -9 archive.zip large_file.txt

# List zip contents
unzip -l archive.zip

# Extract zip
unzip archive.zip
unzip archive.zip -d /tmp/extracted/   # Extract to specific directory

# Extract single file
unzip archive.zip file1.txt

# Password-protected zip
zip -P secret archive.zip file.txt
unzip -P secret archive.zip
```

### `bzip2` / `bunzip2` — Better Compression

```bash
# Compress (slower than gzip, smaller output)
bzip2 file.txt          # Creates file.txt.bz2

# Decompress
bunzip2 file.txt.bz2

# Keep original
bzip2 -k file.txt

# View without extracting
bzcat file.txt.bz2
```

---

## 5. File Permissions

### Understanding the Permission Model

Every file has three permission sets and three permission types:

```
-  r w x  r w x  r w x
│  └─┬─┘  └─┬─┘  └─┬─┘
│    │       │       └── Other (everyone else)
│    │       └────────── Group
│    └────────────────── User (owner)
└─────────────────────── File type (- = regular, d = dir, l = link)

r = read    (4)
w = write   (2)
x = execute (1)
- = no permission (0)
```

```bash
# View permissions
ls -la /etc/passwd
# -rw-r--r-- 1 root root 2847 Jan 10 14:30 /etc/passwd
#  │││││││││
#  ││││││││└── other: execute
#  │││││││└─── other: write
#  ││││││└──── other: read
#  │││││└───── group: execute
#  ││││└────── group: write
#  │││└─────── group: read
#  ││└──────── owner: execute
#  │└───────── owner: write
#  └────────── owner: read

# Read permissions as octal
stat -c "%a %n" /etc/passwd     # 644 /etc/passwd
stat -c "%A %n" /etc/passwd     # -rw-r--r-- /etc/passwd
```

### `chmod` — Change Permissions

#### Symbolic Method

```bash
# Syntax: chmod [who][operator][permissions] file
# who: u (user/owner), g (group), o (other), a (all)
# operator: + (add), - (remove), = (set exactly)
# permissions: r, w, x

# Add execute for owner
chmod u+x script.sh

# Remove write from group and other
chmod go-w sensitive.txt

# Set exact permissions (no more, no less)
chmod u=rwx,g=rx,o= private_script.sh

# Add read for all
chmod a+r public_file.txt

# Make a file private to owner
chmod go= private.txt

# Common patterns
chmod +x script.sh          # Make executable (all users)
chmod 755 script.sh         # Standard executable script
chmod 644 config.txt        # Standard config file
chmod 600 private_key       # Private key (owner read/write only)
chmod 700 private_dir/      # Private directory
```

#### Octal Method

```bash
# Each permission: r=4, w=2, x=1. Sum them for each set.
# rwx = 4+2+1 = 7
# rw- = 4+2+0 = 6
# r-x = 4+0+1 = 5
# r-- = 4+0+0 = 4
# --- = 0+0+0 = 0

# Common octal patterns:
chmod 755 script.sh      # rwxr-xr-x (standard executable)
chmod 644 file.txt       # rw-r--r-- (standard file)
chmod 600 ~/.ssh/id_rsa  # rw------- (private key)
chmod 700 ~/.ssh/        # rwx------ (private directory)
chmod 777 /tmp/shared/   # rwxrwxrwx (world writable — avoid!)
chmod 400 backup.key     # r-------- (read-only by owner)

# Recursive chmod (affects all files in directory)
chmod -R 755 /var/www/html/

# Show numeric permissions
stat -c "%a" /etc/passwd   # 644
```

#### Special Permission Bits

```bash
# SUID (Set User ID) — runs as the file's owner, not the caller
# Displayed as 's' in owner's execute position
chmod u+s /path/to/executable
ls -la /usr/bin/passwd    # Note: -rwsr-xr-x (root-owned SUID)
find /usr/bin -perm /4000 | head -5  # Find all SUID binaries

# SGID (Set Group ID) — runs with file's group, or inherits group for dirs
chmod g+s shared_directory/
ls -la shared_directory/  # drwxrwsr-x

# Sticky Bit — only owner can delete files in the directory
chmod +t /tmp
ls -la / | grep tmp       # drwxrwxrwt (note the 't' at end)

# Octal representation of special bits:
# 4000 = SUID, 2000 = SGID, 1000 = sticky
chmod 4755 executable     # SUID + 755
chmod 1777 shared_dir/    # Sticky + 777
```

### `chown` — Change Ownership

```bash
# Change owner
sudo chown alice file.txt

# Change owner and group
sudo chown alice:developers file.txt

# Change group only
sudo chown :developers file.txt
# Or equivalently:
sudo chgrp developers file.txt

# Recursive ownership change
sudo chown -R www-data:www-data /var/www/html/

# Copy ownership from another file
sudo chown --reference=/etc/passwd new_file.txt

# View current owner/group
ls -la file.txt
stat -c "%U %G" file.txt    # owner group
```

### `chgrp` — Change Group

```bash
# Change group
sudo chgrp staff file.txt
sudo chgrp -R developers /var/www/html/

# View group
ls -la file.txt | awk '{print $4}'    # Group name column
```

---

## 6. Symbolic and Hard Links

### Understanding Links

```
HARD LINK                           SYMBOLIC LINK (Symlink)
─────────────────────────────────   ─────────────────────────────────
filename_A ──┐                      symlink ──────► filename_A ──► inode
filename_B ──┤──► inode ──► data
             │
  Both point to the SAME inode      symlink is a SEPARATE file that
  Delete one: data still exists     contains a path to the target.
  Can only link within same         Can cross filesystems.
  filesystem.                       Delete target: symlink breaks.
```

```bash
# Create a hard link
ln original.txt hardlink.txt
ls -li original.txt hardlink.txt    # Same inode number!

# Create a symbolic link
ln -s /etc/hosts /tmp/hosts_link
ls -la /tmp/hosts_link              # Shows: hosts_link -> /etc/hosts

# Symbolic link to a directory
ln -s /var/log /tmp/log_link
ls /tmp/log_link                    # Shows contents of /var/log

# Check if a file is a symlink
test -L /tmp/hosts_link && echo "It's a symlink"
file /tmp/hosts_link

# Find the real path a symlink points to
readlink /tmp/hosts_link            # /etc/hosts
readlink -f /bin/ls                 # Fully resolved path

# Update a symlink (replace it)
ln -sf /etc/hostname /tmp/my_link   # -f = force overwrite

# Remove a symlink (use rm, NOT rmdir)
rm /tmp/hosts_link
rm /tmp/log_link                    # Removes symlink, not the directory!

# Find all broken symlinks
find /tmp -xtype l 2>/dev/null      # -xtype follows symlinks for type check
```

---

## 7. File Integrity and Checksums

### `md5sum` and `sha256sum`

```bash
# Generate MD5 checksum
md5sum /etc/passwd
# Output: 3fc7c4c2a... /etc/passwd

# Generate SHA256 checksum (more secure)
sha256sum /etc/passwd

# SHA512 checksum (most secure of the common ones)
sha512sum /etc/passwd

# Verify a file against a known checksum
echo "expectedhash  /etc/passwd" > /tmp/expected.md5
md5sum -c /tmp/expected.md5

# Create checksums for multiple files
sha256sum /etc/passwd /etc/hosts /etc/hostname > /tmp/checksums.sha256
cat /tmp/checksums.sha256

# Verify all files
sha256sum -c /tmp/checksums.sha256

# Check an ISO file (see Lesson 03)
sha256sum -c SHA256SUMS --ignore-missing

rm /tmp/expected.md5 /tmp/checksums.sha256 2>/dev/null
```

---

## 8. Putting It All Together — Advanced Examples

```bash
# Example 1: Find the top 10 largest files in /var
find /var -type f -printf "%s %p\n" 2>/dev/null \
  | sort -rn | head -10 \
  | awk '{printf "%10.2f MB  %s\n", $1/1024/1024, $2}'

# Example 2: Archive all .log files older than 7 days and compress them
find /var/log -name "*.log" -mtime +7 \
  | tar -czvf /tmp/old_logs.tar.gz -T -

# Example 3: Find files with SUID bit set (security audit)
find / -perm /4000 -type f 2>/dev/null | sort

# Example 4: Count lines of code by file type in a project
find . -type f | grep -E "\.(py|sh|js|html)$" \
  | xargs wc -l 2>/dev/null \
  | sort -rn | head -10

# Example 5: Replace a hostname across all config files
grep -rl "old_hostname" /etc/ 2>/dev/null \
  | xargs sed -i 's/old_hostname/new_hostname/g'

# Example 6: Batch rename files (add timestamp prefix)
for f in *.txt; do
  mv "$f" "$(date +%Y%m%d)_$f"
done

# Example 7: Generate a directory report
find . -type f | awk -F. '{print $NF}' | sort | uniq -c | sort -rn

# Example 8: Verify integrity of all files in a directory
find /etc -maxdepth 1 -type f | while read f; do
  sha256sum "$f"
done > /tmp/etc_checksums.sha256
echo "Checksums written: $(wc -l < /tmp/etc_checksums.sha256) files"
```

---

## Practice Exercises

### Exercise 1 — Text Transformation Pipeline
Process `/etc/passwd` to produce a formatted user report:

```bash
# List all users with their UID and home directory, sorted by UID
awk -F: '{print $3, $1, $6}' /etc/passwd \
  | sort -n \
  | awk '{printf "UID: %-6s User: %-20s Home: %s\n", $1, $2, $3}' \
  | head -10
```

### Exercise 2 — Archive Mastery
Create, inspect, and extract archives:

```bash
# Create test data
mkdir -p /tmp/archive_test/{docs,images}
touch /tmp/archive_test/docs/{file1.txt,file2.txt,notes.md}
touch /tmp/archive_test/images/{photo1.jpg,photo2.jpg}

# Create a compressed archive
tar -czvf /tmp/backup.tar.gz -C /tmp archive_test/

# List contents without extracting
tar -tzvf /tmp/backup.tar.gz

# Extract to a new location
tar -xzvf /tmp/backup.tar.gz -C /tmp/restored/

# Verify contents match
diff -r /tmp/archive_test/ /tmp/restored/archive_test/ && echo "Perfect match!"

# Clean up
rm -r /tmp/archive_test /tmp/backup.tar.gz /tmp/restored
```

### Exercise 3 — Permission Workshop
Explore and modify file permissions:

```bash
# Create test files
touch /tmp/perm_test_{private,shared,executable}.txt

# Set permissions
chmod 600 /tmp/perm_test_private.txt
chmod 644 /tmp/perm_test_shared.txt
chmod 755 /tmp/perm_test_executable.txt

# Display permissions in both formats
ls -la /tmp/perm_test_*.txt
stat -c "%a %A %n" /tmp/perm_test_*.txt

# Try to interpret: who can do what to each file?
# Clean up
rm /tmp/perm_test_*.txt
```

### Exercise 4 — find Command Challenge

```bash
# Find all files in /etc modified in the last 30 days
find /etc -type f -mtime -30 2>/dev/null | wc -l

# Find all executable files in /usr/bin starting with 'g'
find /usr/bin -name "g*" -perm /111 -type f | head -10

# Find all files larger than 1MB in /var
find /var -type f -size +1M 2>/dev/null | head -10

# Find all world-writable files (potential security issue)
find /tmp -perm -o+w -type f 2>/dev/null | head -10
```

### Exercise 5 — grep Mastery

```bash
# Find all lines in /etc/ssh/sshd_config that are not comments or empty
grep -v "^#\|^$" /etc/ssh/sshd_config 2>/dev/null

# Count how many times "root" appears in /var/log/auth.log
grep -c "root" /var/log/auth.log 2>/dev/null || echo "Log not accessible"

# Extract all IP addresses from /etc/hosts
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" /etc/hosts

# Find files containing a specific string recursively
grep -r "PermitRootLogin" /etc/ 2>/dev/null
```

### Exercise 6 — Checksum Verification

```bash
# Create test files
echo "Important data" > /tmp/important.txt
echo "Config settings" > /tmp/config.txt

# Generate checksums
sha256sum /tmp/important.txt /tmp/config.txt > /tmp/checksums.sha256
cat /tmp/checksums.sha256

# Verify integrity
sha256sum -c /tmp/checksums.sha256
echo "Exit code: $?"

# Tamper with a file
echo "Tampered!" >> /tmp/important.txt

# Detect tampering
sha256sum -c /tmp/checksums.sha256
echo "Exit code: $?"

# Clean up
rm /tmp/important.txt /tmp/config.txt /tmp/checksums.sha256
```

### Exercise 7 — Symbolic Links

```bash
# Create a directory with content
mkdir -p /tmp/link_test/original
echo "Content" > /tmp/link_test/original/data.txt

# Create a symlink to the directory
ln -s /tmp/link_test/original /tmp/link_test/shortcut

# Access data through the symlink
cat /tmp/link_test/shortcut/data.txt

# See that it's a symlink
ls -la /tmp/link_test/

# Create a hard link to the file
ln /tmp/link_test/original/data.txt /tmp/link_test/hardlink.txt
ls -li /tmp/link_test/original/data.txt /tmp/link_test/hardlink.txt
# Note: same inode number!

# Clean up
rm -r /tmp/link_test
```

### Exercise 8 — sed and awk Deep Dive

```bash
# Create test data
cat > /tmp/employees.csv << 'EOF'
Alice,Engineering,95000
Bob,Marketing,75000
Carol,Engineering,105000
Dave,HR,65000
Eve,Engineering,115000
EOF

# Extract Engineering employees using awk
awk -F, '$2 == "Engineering" {print $1, $3}' /tmp/employees.csv

# Calculate average salary with awk
awk -F, '{sum += $3; count++} END {printf "Average salary: $%.0f\n", sum/count}' \
  /tmp/employees.csv

# Use sed to mask salaries (replace numbers)
sed 's/,[0-9]*/,REDACTED/g' /tmp/employees.csv

# Clean up
rm /tmp/employees.csv
```

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| `chmod -R 777 /var/www` | Any user can modify web files | Use `chmod -R 755` for dirs, `644` for files |
| `rm -rf` without quotes on variables | `rm -rf $EMPTY_VAR /` deletes root | Always quote variables: `rm -rf "$var"` |
| Using `cp` without `-p` for backups | Loses original timestamps and permissions | Use `cp -a` for archive copy (preserves all) |
| `sed -i` without testing first | Permanently modifies files incorrectly | Test with `sed 's/x/y/g' file` before `-i` |
| Mixing hard links across filesystems | Hard links cannot cross filesystem boundaries | Use symbolic links for cross-filesystem links |
| Forgetting `sudo updatedb` after `locate` | `locate` returns old (stale) results | Run `sudo updatedb` after installing software |
| `find` without `2>/dev/null` | Flooded with "Permission denied" messages | Always add `2>/dev/null` to find commands |

---

## Pro Tips

> 💡 **`cp -a` is the best backup copy.** `-a` = archive mode, which preserves permissions, timestamps, symlinks, and ownership.

> 💡 **`rsync` beats `cp` for backups.** `rsync -av source/ destination/` is faster (skips unchanged files), preserves everything, and can work over SSH.

> 💡 **`tar` can pipe directly over SSH.** `tar -czf - /data | ssh server "tar -xzf - -C /backup"` — no temp files needed.

> 💡 **Use `grep -P` for Perl-compatible regex.** Unlock `\d`, `\w`, `\s`, `(?:)`, and other powerful patterns.

> 💡 **`awk` is a full programming language.** You can write loops, functions, and arrays in awk — it's not just for column extraction.

> 💡 **ACLs go beyond standard permissions.** `getfacl` and `setfacl` allow per-user permissions beyond the owner/group/other model.

```bash
# Set fine-grained permissions with ACL
setfacl -m u:bob:rw file.txt      # Give bob read+write
getfacl file.txt                  # View ACL
```

---

## Key Takeaways

- `cat`, `less`, `head`, `tail` — master these file viewers first
- `find` is the most powerful file-search tool; learn its `-name`, `-type`, `-size`, `-mtime`, and `-exec` flags
- `grep` searches text; `-r` for recursive, `-E` for extended regex, `-v` to invert
- `sed` is for substitution and line-level editing; `awk` is for column-based processing
- `tar -czvf` creates, `tar -xzvf` extracts; `z` = gzip, `j` = bzip2, `J` = xz
- Permissions: `644` for files, `755` for directories and executables, `600` for private keys
- Symbolic links cross filesystems; hard links cannot — both point to the same data
- Always verify downloaded files with `sha256sum -c`

---

## Next Lesson Preview

**07 — Users, Groups, and Permissions**

Dive deeper into Linux's security model. Learn how to create and manage user accounts, organise users into groups, use `sudo` safely, switch users, manage the `/etc/sudoers` file, and understand how the permission system protects a multi-user system. We'll also cover PAM (Pluggable Authentication Modules) and password policies.
