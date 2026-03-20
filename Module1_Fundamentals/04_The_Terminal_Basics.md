# 04 — The Terminal Basics

> **Module 1 · Lesson 4** | Difficulty: ★★☆☆☆ Beginner-Intermediate | Time: ~90 min

---

## Learning Objectives

- Open and navigate the terminal
- Understand command syntax (command options arguments)
- Use essential commands: ls, cd, pwd, mkdir, touch, cat, echo, man
- Master keyboard shortcuts for efficiency
- Understand stdin, stdout, stderr
- Use pipes and redirection
- Navigate command history

---

## Table of Contents

1. [Opening the Terminal](#1-opening-the-terminal)
2. [Shell Types](#2-shell-types)
3. [Command Syntax](#3-command-syntax)
4. [Essential Commands](#4-essential-commands)
5. [Getting Help](#5-getting-help)
6. [Keyboard Shortcuts](#6-keyboard-shortcuts)
7. [Input/Output Streams](#7-inputoutput-streams)
8. [Pipes and Redirection](#8-pipes-and-redirection)
9. [Command History](#9-command-history)
10. [Tab Completion](#10-tab-completion)
11. [Practice Exercises](#11-practice-exercises)
12. [Key Takeaways](#12-key-takeaways)

---

## 1. Opening the Terminal

### Ubuntu Desktop

- **Keyboard shortcut**: `Ctrl + Alt + T`
- **Search**: Press the Super (Windows) key → type "terminal"
- **Right-click**: Right-click on desktop → "Open Terminal"
- **Application menu**: Applications → Utilities → Terminal

### Terminal Emulators

| Emulator | Install | Best Feature |
|----------|---------|-------------|
| **GNOME Terminal** | Default in Ubuntu | Simple, reliable |
| **Konsole** | Default in KDE | Feature-rich |
| **Terminator** | `apt install terminator` | Split panes |
| **Tilix** | `apt install tilix` | Tiling, profiles |
| **Alacritty** | `apt install alacritty` | GPU-accelerated, fast |
| **kitty** | `apt install kitty` | GPU-accelerated, scriptable |
| **tmux** | `apt install tmux` | Multiplexer, remote sessions |

### The Prompt

When you open a terminal, you see the **prompt**:

```
username@hostname:current_directory$
```

Example:
```
alice@ubuntu:~$
```

| Part | Meaning |
|------|---------|
| `alice` | Your username |
| `ubuntu` | Computer hostname |
| `~` | Current directory (~ = home directory) |
| `$` | Regular user (use `#` for root) |

---

## 2. Shell Types

A **shell** is the program that reads your commands and executes them.

| Shell | Config File | Features |
|-------|------------|---------|
| **bash** | `~/.bashrc`, `~/.bash_profile` | Default on Ubuntu, POSIX, scripting |
| **zsh** | `~/.zshrc` | Better completion, themes (Oh My Zsh) |
| **fish** | `~/.config/fish/config.fish` | Auto-suggest, user-friendly |
| **dash** | `/etc/profile` | Fast, POSIX-only, used for /bin/sh |
| **ksh** | `~/.kshrc` | Korn shell, enterprise |
| **csh/tcsh** | `~/.cshrc` | C-like syntax |

```bash
# Check your current shell
echo $SHELL
echo $0

# See available shells
cat /etc/shells

# Temporarily switch to zsh
zsh

# Change default shell permanently
chsh -s /usr/bin/zsh
```

---

## 3. Command Syntax

### Basic Structure

```
command  [options]  [arguments]
   │         │          │
   │         │          └── What to act on (files, directories, text)
   │         └── Modify behavior (short: -x, long: --verbose)
   └── The program to run
```

### Examples

```bash
ls                      # command only — list current directory
ls -l                   # command + option — long listing
ls -la /home            # command + options + argument
ls --all --human-readable /var/log  # long-form options
```

### Short vs Long Options

```bash
# These are equivalent:
ls -a                   # short option
ls --all                # long option

ls -l -a                # multiple short options
ls -la                  # combined short options (same as above)

rm -rf /tmp/test        # combined: -r (recursive) + -f (force)
```

### Arguments

Arguments tell the command **what** to work on:

```bash
cat file.txt            # argument: file.txt
cp source.txt dest.txt  # two arguments: source and destination
mkdir dir1 dir2 dir3    # multiple arguments
```

---

## 4. Essential Commands

### Navigation

```bash
# Print Working Directory — where are you?
pwd
# Output: /home/alice

# Change Directory
cd /home/alice/Documents    # absolute path
cd Documents                # relative path (from current location)
cd ..                       # go up one level
cd ../..                    # go up two levels
cd ~                        # go to home directory
cd -                        # go to previous directory
cd                          # go to home directory (same as cd ~)

# List Directory Contents
ls                          # basic listing
ls -l                       # long format (permissions, size, date)
ls -a                       # show hidden files (start with .)
ls -la                      # long format + hidden files
ls -lh                      # long format with human-readable sizes
ls -lt                      # sorted by modification time
ls -lS                      # sorted by file size
ls -R                       # recursive listing
ls -1                       # one file per line
ls /etc/                    # list a specific directory
```

### Long Listing Format Explained

```bash
$ ls -la
total 48
drwxr-xr-x  8 alice alice 4096 Jan 15 10:23 .
drwxr-xr-x 27 root  root  4096 Jan 14 09:00 ..
-rw-------  1 alice alice 1234 Jan 15 10:20 .bash_history
-rw-r--r--  1 alice alice  220 Jan 10 08:00 .bash_logout
-rw-r--r--  1 alice alice 3771 Jan 10 08:00 .bashrc
drwxr-xr-x  2 alice alice 4096 Jan 15 08:30 Documents
drwxr-xr-x  2 alice alice 4096 Jan 15 08:30 Downloads
-rw-r--r--  1 alice alice 2048 Jan 15 09:45 notes.txt
```

```
Column breakdown:
-rw-r--r--  1  alice  alice  2048  Jan 15 09:45  notes.txt
│            │    │      │     │         │           │
│            │    │      │     │         │           └── filename
│            │    │      │     │         └── modification date
│            │    │      │     └── file size (bytes)
│            │    │      └── group owner
│            │    └── user owner
│            └── number of hard links
└── permissions (d=dir, -=file, l=link | user|group|others)
```

### File and Directory Operations

```bash
# Create files
touch file.txt              # create empty file (or update timestamp)
touch file1.txt file2.txt   # create multiple files

# Create directories
mkdir mydir                 # create directory
mkdir -p parent/child/grandchild  # create nested dirs (-p = parents)
mkdir -p project/{src,tests,docs} # create multiple subdirs

# Display file contents
cat file.txt                # print entire file
cat -n file.txt             # with line numbers
cat file1.txt file2.txt     # concatenate files

# Print text
echo "Hello World"          # print text
echo "Path: $HOME"          # with variable expansion
echo -n "No newline"        # no trailing newline
echo -e "Tab:\there"        # interpret escape sequences

# Copy files/directories
cp file.txt backup.txt      # copy file
cp -r dir/ backup_dir/      # copy directory recursively
cp -p file.txt dest/        # preserve timestamps/permissions
cp -i file.txt dest/        # interactive (ask before overwrite)

# Move/Rename
mv file.txt newname.txt     # rename file
mv file.txt /tmp/           # move to different directory
mv *.txt archive/           # move all .txt files

# Remove (DELETE — no recycle bin!)
rm file.txt                 # remove file
rm -r directory/            # remove directory recursively
rm -f file.txt              # force (no prompt)
rm -rf directory/           # force recursive (DANGEROUS!)
rm -i file.txt              # interactive (ask before each delete)
rmdir emptydir/             # remove empty directory only
```

> ⚠️ **`rm -rf` is permanent!** There is no recycle bin. Be VERY careful!

```bash
# Safe alternative: move to trash
trash-put file.txt          # requires: sudo apt install trash-cli
trash-list                  # see trashed files
trash-restore               # restore from trash
```

---

## 5. Getting Help

### The `man` Command (Manual Pages)

```bash
man ls                      # manual page for ls
man 5 passwd                # section 5: file formats
man -k keyword              # search manual pages
man -f command              # brief description

# Navigate man pages:
# Arrow keys or j/k — scroll line by line
# Space / Ctrl+F — page forward
# Ctrl+B — page backward
# /pattern — search forward
# n — next search match
# q — quit
```

### Manual Page Sections

| Section | Content |
|---------|---------|
| 1 | User commands |
| 2 | System calls |
| 3 | Library functions |
| 4 | Special files/devices |
| 5 | File formats and conventions |
| 6 | Games |
| 7 | Miscellaneous |
| 8 | System administration commands |

```bash
man passwd      # section 1: passwd command
man 5 passwd    # section 5: /etc/passwd file format
```

### Other Help Options

```bash
# Built-in help
ls --help               # many commands have --help
git --help
python3 --help

# Brief description
whatis ls
whatis grep
apropos "list files"    # search by description

# Info pages (more detailed than man)
info ls
info bash

# Command type
type ls                 # bash builtin or external?
type cd                 # "cd is a shell builtin"
which python3           # path to executable
whereis gcc             # binary + man page + source locations
```

---

## 6. Keyboard Shortcuts

These shortcuts dramatically speed up terminal work:

### Navigation

| Shortcut | Action |
|----------|--------|
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+F` | Move forward one character |
| `Ctrl+B` | Move backward one character |
| `Alt+F` | Move forward one word |
| `Alt+B` | Move backward one word |
| `Ctrl+XX` | Toggle between current and beginning of line |

### Editing

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Delete from cursor to end of line |
| `Ctrl+U` | Delete from cursor to beginning of line |
| `Ctrl+W` | Delete word before cursor |
| `Alt+D` | Delete word after cursor |
| `Ctrl+Y` | Paste (yank) last deleted text |
| `Ctrl+T` | Swap last two characters |
| `Alt+T` | Swap last two words |

### Control

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Interrupt (kill) running command |
| `Ctrl+Z` | Suspend (pause) running command |
| `Ctrl+D` | Send EOF / logout (empty line) |
| `Ctrl+L` | Clear screen (same as `clear`) |
| `Ctrl+S` | Freeze output (pause scrolling) |
| `Ctrl+Q` | Unfreeze output |
| `Ctrl+R` | Reverse search history |

### Process Control Example

```bash
# Start a long-running command
sleep 100

# Press Ctrl+Z to suspend it
# [1]+  Stopped    sleep 100

# View background jobs
jobs

# Resume in background
bg %1

# Resume in foreground
fg %1

# Kill a specific job
kill %1
```

---

## 7. Input/Output Streams

Every Linux process has three standard data streams:

```
┌─────────────────────────────────────┐
│                                     │
│  stdin  ──(0)──►  PROCESS  ──(1)──► stdout
│                      │              │
│                      └──(2)──────► stderr
│                                     │
└─────────────────────────────────────┘

stdin  (fd 0): Input (keyboard by default)
stdout (fd 1): Normal output (terminal by default)
stderr (fd 2): Error messages (terminal by default)
```

```bash
# stdin: read input from user
read -p "Enter your name: " name
echo "Hello, $name!"

# stdout: normal output
echo "This goes to stdout"
ls /home

# stderr: error output
ls /nonexistent 2>&1  # stderr appears separately
# ls: cannot access '/nonexistent': No such file or directory

# Check exit code ($? = 0 success, non-zero = failure)
ls /etc/passwd
echo $?    # 0

ls /nonexistent
echo $?    # 2 (non-zero = error)
```

---

## 8. Pipes and Redirection

### Output Redirection

```bash
# Redirect stdout to file (overwrite)
echo "Hello" > file.txt
ls -la > listing.txt

# Redirect stdout to file (append)
echo "World" >> file.txt
date >> log.txt

# Redirect stderr to file
ls /nonexistent 2> errors.txt

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt        # shorthand (bash)

# Discard output (send to /dev/null)
command > /dev/null           # discard stdout
command 2>/dev/null           # discard stderr
command &>/dev/null           # discard everything
```

### Input Redirection

```bash
# Read from file instead of keyboard
sort < unsorted.txt
wc -l < file.txt

# Here document (multi-line input)
cat << EOF > config.txt
[settings]
name = Alice
debug = true
EOF

# Here string
grep "pattern" <<< "string to search"
```

### Pipes (`|`)

Pipes connect the stdout of one command to the stdin of another:

```bash
# Basic pipe
ls -la | grep ".txt"        # list files, filter for .txt
cat /etc/passwd | grep root  # find root entries

# Chain multiple pipes
cat /var/log/syslog | grep ERROR | tail -20 | sort

# Count lines
ls /etc | wc -l              # how many files in /etc?
cat file.txt | wc -l         # how many lines?

# Find and process
find /home -name "*.log" | xargs wc -l  # count lines in all .log files

# Real-world examples
ps aux | grep python          # find python processes
netstat -tuln | grep 80       # check if port 80 is open
history | grep "git commit"   # find git commit commands
cat /etc/passwd | cut -d: -f1 | sort  # sorted list of usernames
```

### The `tee` Command

`tee` reads from stdin and writes to BOTH stdout and a file:

```bash
command | tee output.txt          # write to file AND terminal
command | tee -a output.txt       # append to file AND terminal
command | tee file1.txt file2.txt # write to multiple files
```

---

## 9. Command History

### Viewing and Searching History

```bash
history             # show all command history
history 20          # show last 20 commands
history | grep ssh  # search history for 'ssh'

# History file
cat ~/.bash_history  # view history file directly
```

### History Expansion

```bash
!!          # repeat last command
!42         # run command number 42 from history
!ssh        # run last command starting with 'ssh'
!$          # last argument of previous command
!*          # all arguments of previous command

# Example:
ls /etc/nginx/nginx.conf
vim !$      # same as: vim /etc/nginx/nginx.conf

# Ctrl+R: reverse incremental search
# Type part of a command, press Ctrl+R to search history
```

### History Configuration

```bash
# Add to ~/.bashrc:
export HISTSIZE=10000           # lines to keep in memory
export HISTFILESIZE=20000       # lines to keep in file
export HISTCONTROL=ignoredups   # don't store duplicates
export HISTTIMEFORMAT="%Y-%m-%d %T "  # timestamps

# Reload config
source ~/.bashrc
```

---

## 10. Tab Completion

Tab completion is one of the most powerful productivity features:

```bash
# Complete commands
sys<Tab>         # → systemctl (if unique)
sys<Tab><Tab>    # → show all matches

# Complete file paths
cd /et<Tab>      # → /etc/
cat /etc/pas<Tab> # → /etc/passwd

# Complete options
ls --<Tab><Tab>   # shows all long options

# Complete variables
echo $HOM<Tab>   # → echo $HOME
```

### Enhanced Completion (bash-completion)

```bash
sudo apt install bash-completion

# For git:
sudo apt install git bash-completion
# Now: git chec<Tab> → git checkout
```

---

## 11. Practice Exercises

### Exercise 4.1 — Navigation Challenge

Start in your home directory and navigate:

```bash
# 1. Print your current directory
# 2. Go to /var/log
# 3. List all files including hidden ones
# 4. Go up two levels
# 5. List only directories
# 6. Return home in one command
# 7. Check where you are
```

### Exercise 4.2 — File Operations

```bash
# Create this directory structure:
# project/
# ├── src/
# │   ├── main.py
# │   └── utils.py
# ├── tests/
# │   └── test_main.py
# └── README.txt

mkdir -p project/{src,tests}
touch project/src/main.py project/src/utils.py
touch project/tests/test_main.py
echo "# My Project" > project/README.txt

# Verify:
ls -R project/
```

### Exercise 4.3 — Pipes and Redirection

```bash
# 1. Save list of files in /etc to a file called etc_listing.txt
# 2. Count how many files are in /etc
# 3. Find all files in /etc that contain "ssh" in their name
# 4. Show the 5 largest files in /var/log
# 5. Save all error messages from ls /nonexistent to errors.txt
```

### Exercise 4.4 — Getting Help

```bash
# Look up the man page for these commands and find ONE
# useful option you didn't know about:
man cp
man find
man grep
```

### Exercise 4.5 — History and Shortcuts

1. Type 10 different commands
2. Use `Ctrl+R` to search your history
3. Use `!!` to repeat the last command
4. Use `!$` to reuse the last argument
5. Practice each keyboard shortcut from the table above

---

## 12. Key Takeaways

- The **terminal** is your primary interface to Linux — master it!
- Command syntax: `command [options] [arguments]`
- **Navigation**: `pwd`, `ls`, `cd` are your most-used commands
- **File operations**: `touch`, `mkdir`, `cp`, `mv`, `rm` — be careful with `rm -rf`!
- **Get help**: `man command`, `command --help`, `apropos`
- **Keyboard shortcuts**: `Ctrl+C` (stop), `Ctrl+Z` (suspend), `Ctrl+L` (clear)
- **Streams**: stdin (0), stdout (1), stderr (2)
- **Pipes** (`|`) chain commands; **redirection** (`>`, `>>`, `<`) connects to files
- **Tab completion** and **history** (`Ctrl+R`) save enormous time

---

## Next Lesson

➡️ [05 — File System Structure](05_File_System_Structure.md)

---

*Module 1 · Lesson 4 of 6 | [Course Index](../INDEX.md)*
