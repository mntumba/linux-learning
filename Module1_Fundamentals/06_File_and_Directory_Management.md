# 06 — File and Directory Management

> **Module 1 · Lesson 6** | Difficulty: ★★☆☆☆ Beginner-Intermediate | Time: ~90 min

---

## Learning Objectives

- Master file management commands (cp, mv, rm, find)
- Compress and archive files (tar, gzip, zip)
- View and process file contents (cat, less, head, tail, grep)
- Use text editors (nano and basic vim)
- Understand file types and the `file` command
- Work with disk usage tools (du, df)

---

## Table of Contents

1. [File Operations Deep Dive](#1-file-operations-deep-dive)
2. [File Viewing](#2-file-viewing)
3. [Text Search with grep](#3-text-search-with-grep)
4. [Sorting and Counting](#4-sorting-and-counting)
5. [Archiving and Compression](#5-archiving-and-compression)
6. [Disk Usage](#6-disk-usage)
7. [Text Editors: nano](#7-text-editors-nano)
8. [Text Editors: vim](#8-text-editors-vim)
9. [File Information](#9-file-information)
10. [Practice Exercises](#10-practice-exercises)
11. [Key Takeaways](#11-key-takeaways)

---

## 1. File Operations Deep Dive

### Copying Files (`cp`)

```bash
# Basic copy
cp source.txt destination.txt   # copy to new name
cp file.txt /tmp/               # copy to directory
cp file.txt ~/backup/           # copy to home/backup

# Copy multiple files
cp file1.txt file2.txt /tmp/   # multiple sources, one destination
cp *.txt /backup/               # copy all .txt files

# Directory copy (must use -r)
cp -r source_dir/ destination_dir/    # recursive copy
cp -r /etc/nginx /tmp/nginx_backup    # backup config

# Useful options
cp -p file.txt dest/       # preserve permissions + timestamps
cp -u file.txt dest/       # only copy if source is newer
cp -i file.txt dest/       # interactive (ask before overwrite)
cp -v file.txt dest/       # verbose (show what's copied)
cp -a dir/ dest/           # archive mode (= -dpr, best for backups)
cp --backup file.txt dest/ # backup existing before overwrite
```

### Moving and Renaming (`mv`)

```bash
# Rename
mv oldname.txt newname.txt     # rename in same directory
mv mydir/ newdir/              # rename directory

# Move
mv file.txt /tmp/              # move to directory
mv file1.txt file2.txt /tmp/   # move multiple files
mv *.log /var/log/archive/     # move with wildcard

# Useful options
mv -i file.txt dest/           # interactive (ask before overwrite)
mv -u file.txt dest/           # only if source is newer
mv -v file.txt dest/           # verbose

# Move and rename simultaneously
mv /home/alice/notes.txt /home/alice/Documents/meeting_notes.txt
```

### Removing Files (`rm`)

```bash
# Remove files
rm file.txt                    # remove single file
rm file1.txt file2.txt         # remove multiple files
rm *.tmp                       # remove all .tmp files

# Remove directories
rm -r directory/               # recursive removal
rm -rf /tmp/testdir/           # force recursive (no prompts)

# Safe options
rm -i file.txt                 # ask for each file
rm -I *.txt                    # ask once for multiple files
rm -v file.txt                 # verbose

# Safer alternative
trash-put file.txt             # sudo apt install trash-cli
```

> ⚠️ **WARNING**: `rm -rf` has no undo. Always double-check the path!
> A famous accident: `rm -rf /` (destroys entire system)
> Protection: add `--preserve-root` (default in modern rm)

### Creating Files and Directories

```bash
# Create empty file
touch file.txt

# Create multiple files
touch file{1,2,3,4,5}.txt      # creates file1.txt through file5.txt
touch notes_$(date +%Y%m%d).txt  # date-stamped file

# Create directory
mkdir newdir

# Create nested directories
mkdir -p /tmp/project/src/tests/  # all parent dirs

# Create with specific permissions
mkdir -m 755 publicdir
mkdir -m 700 privatedir

# Template: create project structure
mkdir -p myproject/{src,tests,docs,data,scripts}
touch myproject/{README.md,requirements.txt,.gitignore}
ls -R myproject/
```

---

## 2. File Viewing

### `cat` — Concatenate and Print

```bash
cat file.txt                    # print entire file
cat -n file.txt                 # with line numbers
cat -A file.txt                 # show non-printing chars ($ for newline)
cat -s file.txt                 # squeeze blank lines
cat file1.txt file2.txt         # concatenate two files
cat file1.txt file2.txt > merged.txt  # merge into new file
```

### `less` — Interactive Viewer (Recommended for Large Files)

```bash
less /var/log/syslog            # interactive viewing
less +F /var/log/syslog         # follow mode (like tail -f)
less -N file.txt                # show line numbers

# Navigation in less:
# j / Down arrow — next line
# k / Up arrow   — previous line
# Space / PgDn   — next page
# b / PgUp       — previous page
# g              — go to beginning
# G              — go to end
# /pattern       — search forward
# ?pattern       — search backward
# n              — next match
# N              — previous match
# q              — quit
# :n             — next file (if multiple)
```

### `head` and `tail`

```bash
# head: first N lines (default 10)
head file.txt                   # first 10 lines
head -20 file.txt               # first 20 lines
head -n 5 file.txt              # first 5 lines
head -c 100 file.txt            # first 100 bytes

# tail: last N lines (default 10)
tail file.txt                   # last 10 lines
tail -20 file.txt               # last 20 lines
tail -n 5 file.txt              # last 5 lines

# VERY USEFUL: follow a file in real-time
tail -f /var/log/syslog         # watch log file grow
tail -f /var/log/auth.log       # watch auth events
tail -f /var/log/nginx/access.log  # watch web requests

# Combine head and tail
head -20 file.txt | tail -10    # lines 11-20
```

### `more` — Simple Pager (Older Alternative)

```bash
more file.txt       # simple pager
# Space = next page, q = quit, /pattern = search
# less is generally better — use less instead
```

---

## 3. Text Search with grep

`grep` searches for patterns in text files.

```bash
# Basic grep
grep "pattern" file.txt             # find lines with "pattern"
grep "error" /var/log/syslog        # find errors in log
grep "root" /etc/passwd             # find root entries

# Case insensitive
grep -i "ERROR" file.txt            # matches Error, error, ERROR

# Inverted match (lines WITHOUT pattern)
grep -v "pattern" file.txt          # exclude lines with pattern
grep -v "^#" /etc/ssh/sshd_config  # show non-comment lines

# Line numbers
grep -n "TODO" script.py            # show line numbers

# Count matches
grep -c "error" logfile.txt         # count matching lines

# Show only matching text (not whole line)
grep -o "pattern" file.txt          # extract only matches

# Recursive (search in all files under directory)
grep -r "password" /etc/            # search config files
grep -r "TODO" ~/projects/          # find TODOs in code
grep -rl "pattern" /path/           # only show filenames

# Context lines
grep -A 2 "error" file.txt          # 2 lines after match
grep -B 2 "error" file.txt          # 2 lines before match
grep -C 3 "error" file.txt          # 3 lines before AND after

# Extended regex (-E or egrep)
grep -E "error|warning" file.txt    # OR pattern
grep -E "^(root|alice):" /etc/passwd  # lines starting with root or alice
grep -E "[0-9]{1,3}\.[0-9]{1,3}" file.txt  # IP addresses

# Fixed string (faster, no regex interpretation)
grep -F "exact.string[with.special*chars]" file.txt

# Useful real-world examples
grep -i "failed" /var/log/auth.log         # failed login attempts
grep "sshd" /var/log/auth.log | tail -20   # recent SSH events
grep -r "api_key" ~/.config/ 2>/dev/null  # find API keys
ps aux | grep "python"                     # find python processes
```

---

## 4. Sorting and Counting

### `sort` — Sort Lines

```bash
sort file.txt                    # alphabetical sort
sort -r file.txt                 # reverse sort
sort -n numbers.txt              # numeric sort
sort -rn numbers.txt             # reverse numeric
sort -k2 file.txt                # sort by column 2
sort -k2 -n file.txt             # sort by column 2 numerically
sort -t: -k3 -n /etc/passwd      # sort passwd by UID (field 3)
sort -u file.txt                 # sort + remove duplicates
sort -h sizes.txt                # human-readable sizes (1K, 2M, 1G)

# Sort and save
sort file.txt > sorted.txt
sort -o file.txt file.txt        # sort in-place
```

### `uniq` — Remove Duplicate Lines

```bash
uniq file.txt                    # remove consecutive duplicates
uniq -c file.txt                 # count occurrences
uniq -d file.txt                 # show only duplicates
uniq -u file.txt                 # show only unique lines

# Always sort first for meaningful results
sort file.txt | uniq             # remove all duplicates
sort file.txt | uniq -c | sort -rn  # count and sort by frequency
```

### `wc` — Word Count

```bash
wc file.txt                      # lines, words, characters
wc -l file.txt                   # lines only
wc -w file.txt                   # words only
wc -c file.txt                   # bytes
wc -m file.txt                   # characters

# Count files
ls /etc | wc -l                  # number of files in /etc
find /home -type f | wc -l       # number of files in /home
```

### `cut` — Extract Columns

```bash
cut -d: -f1 /etc/passwd          # extract field 1 (username)
cut -d: -f1,3 /etc/passwd        # fields 1 and 3 (username + UID)
cut -d, -f2 data.csv             # CSV second column
cut -c1-10 file.txt              # first 10 characters per line
cut -d' ' -f2- sentence.txt      # from field 2 to end
```

### `awk` — Text Processing (Introduction)

```bash
# Print field N (space-separated)
awk '{print $1}' file.txt         # first field
awk '{print $1, $3}' file.txt     # fields 1 and 3
awk -F: '{print $1}' /etc/passwd  # custom delimiter

# Filter rows
awk '$3 > 1000' /etc/passwd       # users with UID > 1000
awk '/pattern/ {print}' file.txt  # lines matching pattern

# Math
awk '{sum += $1} END {print sum}' numbers.txt  # sum column
awk 'NR > 5 && NR < 10' file.txt              # lines 6-9
```

### `sed` — Stream Editor (Introduction)

```bash
# Substitution
sed 's/old/new/' file.txt         # replace first occurrence per line
sed 's/old/new/g' file.txt        # replace all occurrences
sed 's/old/new/gi' file.txt       # case-insensitive replace all
sed -i 's/old/new/g' file.txt     # edit in place (modifies file!)

# Delete lines
sed '/pattern/d' file.txt         # delete lines with pattern
sed '5d' file.txt                 # delete line 5
sed '1,3d' file.txt               # delete lines 1-3

# Print specific lines
sed -n '5p' file.txt              # print line 5 only
sed -n '5,10p' file.txt           # print lines 5-10
```

---

## 5. Archiving and Compression

### `tar` — Tape Archive

```bash
# Create archive
tar -cvf archive.tar files/       # create, verbose, file
tar -czvf archive.tar.gz files/   # create + gzip compress
tar -cjvf archive.tar.bz2 files/  # create + bzip2 compress
tar -cJvf archive.tar.xz files/   # create + xz compress (best compression)

# Extract archive
tar -xvf archive.tar              # extract
tar -xzvf archive.tar.gz          # extract gzipped
tar -xjvf archive.tar.bz2         # extract bzip2
tar -xJvf archive.tar.xz          # extract xz

# Extract to specific directory
tar -xzvf archive.tar.gz -C /tmp/   # extract to /tmp

# List contents without extracting
tar -tvf archive.tar              # list contents
tar -tzvf archive.tar.gz          # list gzip archive

# Extract single file
tar -xzvf archive.tar.gz file.txt  # extract only file.txt

# Common options:
# c = create    x = extract   t = list
# v = verbose   f = filename  z = gzip
# j = bzip2     J = xz        p = preserve permissions
```

### `gzip` / `gunzip` — GNU Zip

```bash
gzip file.txt                    # compress → file.txt.gz (removes original)
gzip -k file.txt                 # keep original (GNU gzip 1.6+)
gzip -9 file.txt                 # maximum compression
gzip -l archive.gz               # list compression ratio

gunzip file.txt.gz               # decompress
gunzip -k file.txt.gz            # keep archive

zcat file.txt.gz                 # view compressed file without extracting
zgrep "pattern" file.txt.gz      # grep in compressed file
```

### `bzip2` — Better Compression

```bash
bzip2 file.txt                   # compress → file.txt.bz2
bunzip2 file.txt.bz2             # decompress
bzcat file.txt.bz2               # view without extracting
```

### `xz` — Excellent Compression

```bash
xz file.txt                      # compress → file.txt.xz
xz -k file.txt                   # keep original
xz -9 file.txt                   # maximum compression
unxz file.txt.xz                 # decompress
xzcat file.txt.xz                # view without extracting
```

### `zip` / `unzip` — Cross-Platform Archives

```bash
zip archive.zip file1.txt file2.txt   # create zip
zip -r archive.zip directory/         # recursive
zip -e secret.zip file.txt            # encrypted zip
zip -9 archive.zip file.txt           # max compression

unzip archive.zip                     # extract to current dir
unzip archive.zip -d /tmp/            # extract to specific dir
unzip -l archive.zip                  # list contents
unzip -p archive.zip file.txt         # extract to stdout

# 7zip (better compression than zip)
sudo apt install p7zip-full
7z a archive.7z files/               # create 7z archive
7z x archive.7z                      # extract
7z l archive.7z                      # list
```

### Compression Comparison

| Format | Command | Ratio | Speed | Cross-Platform |
|--------|---------|-------|-------|----------------|
| gzip | tar.gz | Medium | Fast | Yes |
| bzip2 | tar.bz2 | Better | Slow | Yes |
| xz | tar.xz | Best | Slowest | Most |
| zip | .zip | Medium | Medium | Yes (Windows) |
| 7zip | .7z | Excellent | Medium | Yes |

---

## 6. Disk Usage

### `df` — Disk Free (Filesystem Usage)

```bash
df                               # disk usage for all mounted filesystems
df -h                            # human-readable (GB, MB)
df -H                            # SI units (1000 not 1024)
df -T                            # show filesystem type
df /home                         # specific mount point
df -i                            # inode usage

# Example output:
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G   12G   35G  26% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/sda2       197G   45G  142G  24% /home
```

### `du` — Disk Usage (Directory/File Size)

```bash
du file.txt                      # size of file (blocks)
du -h file.txt                   # human-readable
du -h directory/                 # size of directory recursively
du -sh directory/                # summary only (total)
du -sh *                         # size of each item in current dir
du -ah directory/                # all files and dirs
du --max-depth=1 /var/log        # one level deep only

# Find the biggest directories
du -sh /* 2>/dev/null | sort -rh | head -20
du -sh /var/* 2>/dev/null | sort -rh | head -10

# Find large files
find / -type f -size +100M 2>/dev/null | du -h
```

---

## 7. Text Editors: nano

`nano` is the easiest terminal text editor — great for beginners.

```bash
nano file.txt               # open file
nano +10 file.txt           # open at line 10
nano -l file.txt            # show line numbers
nano -v file.txt            # view only (read-only)
```

### nano Key Reference

```
FILE OPERATIONS:
  Ctrl+S  — Save file
  Ctrl+O  — Write Out (save to filename)
  Ctrl+X  — Exit (prompts to save if unsaved)

NAVIGATION:
  Arrow keys — move cursor
  Ctrl+A     — beginning of line
  Ctrl+E     — end of line
  Ctrl+Y     — page up
  Ctrl+V     — page down
  Ctrl+_     — go to line number

EDITING:
  Ctrl+K  — cut line
  Ctrl+U  — paste line
  Alt+6   — copy line
  Ctrl+D  — delete character (or Delete key)

SEARCH & REPLACE:
  Ctrl+W  — search
  Ctrl+R  — search and replace
  Alt+W   — search next

MISCELLANEOUS:
  Ctrl+G  — help
  Ctrl+Z  — suspend nano
  Alt+U   — undo
  Alt+E   — redo
```

### nano Configuration

```bash
# ~/.nanorc - nano configuration
cat << 'EOF' > ~/.nanorc
set linenumbers
set autoindent
set tabsize 4
set tabstospaces
set mouse
set softwrap
EOF
```

---

## 8. Text Editors: vim

`vim` (Vi Improved) is the most powerful terminal editor. It has a steep learning curve but is incredibly powerful.

### The Modal Editor Concept

vim has different **modes**:

```
NORMAL MODE (default)
    │
    ├── press i → INSERT MODE (type text)
    │   └── press Esc → back to NORMAL
    │
    ├── press v → VISUAL MODE (select text)
    │   └── press Esc → back to NORMAL
    │
    ├── press : → COMMAND MODE (execute commands)
    │   └── press Esc → back to NORMAL
    │
    └── press R → REPLACE MODE (overwrite)
        └── press Esc → back to NORMAL
```

### vim Quick Start

```bash
vim file.txt           # open file
```

```
STARTING:
  vim file.txt  — open file
  vim +50 file  — open at line 50
  vim -R file   — read-only mode

ENTERING INSERT MODE:
  i  — insert before cursor
  a  — insert after cursor
  I  — insert at beginning of line
  A  — insert at end of line
  o  — new line below
  O  — new line above

RETURNING TO NORMAL:
  Esc — always returns to Normal mode

SAVING AND QUITTING (in Normal mode, press :):
  :w          — save (write)
  :q          — quit
  :wq or :x   — save and quit
  :q!         — quit without saving (force)
  ZZ          — save and quit (shortcut)
  ZQ          — quit without saving (shortcut)

NAVIGATION (Normal mode):
  h j k l     — left, down, up, right
  w           — next word
  b           — previous word
  0           — beginning of line
  $           — end of line
  gg          — top of file
  G           — bottom of file
  :42         — go to line 42
  Ctrl+F      — page forward
  Ctrl+B      — page backward

EDITING (Normal mode):
  x           — delete character
  dd          — delete line
  yy          — yank (copy) line
  p           — paste after cursor
  P           — paste before cursor
  u           — undo
  Ctrl+R      — redo
  .           — repeat last change
  cw          — change word (delete + insert mode)
  cc          — change line
  dw          — delete word
  d$          — delete to end of line

SEARCH:
  /pattern    — search forward
  ?pattern    — search backward
  n           — next match
  N           — previous match
  *           — search word under cursor

SEARCH AND REPLACE:
  :%s/old/new/g        — replace all in file
  :%s/old/new/gc       — replace with confirmation
  :5,20s/old/new/g     — replace in lines 5-20
```

### vim Survival Guide

If you accidentally open vim and don't know how to exit:

```
Step 1: Press Esc (multiple times if needed — get to Normal mode)
Step 2: Type :q! and press Enter (quit without saving)
```

---

## 9. File Information

### `file` — Determine File Type

```bash
file document.pdf           # PDF document
file image.jpg              # JPEG image
file script.sh              # POSIX shell script
file /bin/ls                # ELF 64-bit executable
file archive.tar.gz         # gzip compressed data
file /dev/sda               # block special file
file unknown_binary         # (identifies type even without extension)

# Check multiple files
file *
file *.{txt,py,sh,jpg}
```

### `stat` — Detailed File Statistics

```bash
stat file.txt
# File: file.txt
# Size: 2048          Blocks: 8      IO Block: 4096   regular file
# Device: 8,1         Inode: 131074   Links: 1
# Access: (0644/-rw-r--r--)  Uid: (1000/alice)   Gid: (1000/alice)
# Access: 2024-01-15 10:23:45.123456789 +0000
# Modify: 2024-01-14 09:00:12.000000000 +0000
# Change: 2024-01-14 09:00:12.000000000 +0000
# Birth: 2024-01-14 09:00:12.000000000 +0000
```

### `xxd` and `hexdump` — View Binary Files

```bash
xxd file.bin | head             # hex dump
xxd -l 64 file.bin              # first 64 bytes
hexdump -C file.bin | head      # hexdump with ASCII

# Useful for: analyzing binaries, checking file signatures
xxd image.jpg | head -2         # JPEG starts with FF D8 FF
xxd archive.gz | head -1        # Gzip starts with 1F 8B
```

### `strings` — Extract Text from Binaries

```bash
strings /bin/ls | head -20       # printable strings in binary
strings -n 8 binary_file         # min 8-char strings
strings malware_sample           # (security analysis)
```

---

## 10. Practice Exercises

### Exercise 6.1 — Archive Challenge

```bash
# 1. Create a directory structure to backup
mkdir -p ~/backup_test/{documents,images,code}
touch ~/backup_test/documents/report.txt
touch ~/backup_test/images/photo.jpg
touch ~/backup_test/code/script.py

# 2. Create a gzip-compressed tar archive
tar -czvf backup_$(date +%Y%m%d).tar.gz ~/backup_test/

# 3. Verify the archive
tar -tzvf backup_*.tar.gz

# 4. Extract to /tmp
tar -xzvf backup_*.tar.gz -C /tmp/

# 5. Compare sizes
du -sh ~/backup_test/
ls -lh backup_*.tar.gz
```

### Exercise 6.2 — Text Processing Pipeline

```bash
# 1. Create a sample log file
for i in {1..100}; do
    echo "$(date) [INFO] Process $i completed"
done > /tmp/app.log

for i in {1..20}; do
    echo "$(date) [ERROR] Connection failed to host$i" >> /tmp/app.log
done

# 2. Count total lines
wc -l /tmp/app.log

# 3. Count errors
grep -c "ERROR" /tmp/app.log

# 4. Show last 10 errors
grep "ERROR" /tmp/app.log | tail -10

# 5. Extract unique hostnames from errors
grep "ERROR" /tmp/app.log | grep -o "host[0-9]*" | sort | uniq
```

### Exercise 6.3 — vim Practice

```bash
# Open a file in vim
vim ~/practice.txt

# Practice these operations:
# 1. Enter insert mode (i) and type some text
# 2. Return to normal mode (Esc)
# 3. Navigate with h,j,k,l
# 4. Copy a line (yy) and paste it (p)
# 5. Delete a line (dd)
# 6. Search for a word (/)
# 7. Replace a word (:%s/old/new/g)
# 8. Save and quit (:wq)
```

### Exercise 6.4 — Disk Analysis

```bash
# 1. Check filesystem usage
df -h

# 2. Find top 10 largest directories in /var
du -sh /var/* 2>/dev/null | sort -rh | head -10

# 3. Find files larger than 10MB
find / -type f -size +10M 2>/dev/null | head -10

# 4. Check inode usage
df -i | sort -k5 -rn | head

# 5. Find the total size of all .log files
find /var/log -name "*.log" -type f | xargs du -sh 2>/dev/null
```

---

## 11. Key Takeaways

- **cp** and **mv**: always use `-i` for interactive to avoid accidental overwrites
- **rm -rf** is permanent — prefer `trash-put` for important files
- **tar.gz** is the standard Linux archive format; use `tar -czvf` to create, `-xzvf` to extract
- **grep** with `-r`, `-i`, `-n`, `-v` covers most search needs
- **less** is the best pager — use it for large files (`less +F` for live following)
- **nano** is beginner-friendly; **vim** is powerful once you learn the modes
- **du -sh** and **df -h** are your go-to commands for disk space analysis
- `file` command identifies file type regardless of extension

---

## Module 1 Complete! 🎉

You now have solid foundations in Linux fundamentals. You can navigate the file system, manage files, use the terminal efficiently, and understand the Linux ecosystem.

---

## Next Module

➡️ [Module 2: Intermediate — User and Permission Management](../Module2_Intermediate/07_User_and_Permission_Management.md)

---

*Module 1 · Lesson 6 of 6 | [Course Index](../INDEX.md)*
