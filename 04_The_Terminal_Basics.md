# 04 — The Terminal Basics

> **Difficulty:** Beginner | **Time Estimate:** 90–120 minutes

---

## Learning Objectives

By the end of this lesson you will be able to:

- Explain the difference between terminal, shell, console, and TTY
- Open a terminal in Ubuntu multiple ways
- Read and construct commands using the syntax `command [options] [arguments]`
- Use 20+ essential Linux commands confidently
- Navigate the filesystem with absolute and relative paths
- Use tab completion and keyboard shortcuts for speed
- Understand stdin, stdout, and stderr
- Redirect output to files and chain commands with pipes

---

## 1. Terminal vs Console vs Shell vs TTY

These terms are often used interchangeably but have distinct meanings:

```
┌─────────────────────────────────────────────────────────────────┐
│  TERMINAL (Terminal Emulator)                                   │
│  ─────────────────────────────                                  │
│  A GUI application that provides a window for text interaction  │
│  Examples: GNOME Terminal, Alacritty, Kitty, xterm, tmux       │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  SHELL                                                  │   │
│  │  ────────────────────────────────────────────────────   │   │
│  │  A command interpreter — reads your input, runs it.    │   │
│  │  Examples: bash, zsh, fish, sh, ksh, dash              │   │
│  │                                                         │   │
│  │  $ echo "Hello"    ← You type this                     │   │
│  │  Hello             ← Shell runs it via the kernel      │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

CONSOLE  — Physical or virtual terminal directly attached to the system
           (Think: the monitor and keyboard plugged directly into a server)

TTY      — TeleTYpewriter. Linux's name for any terminal device.
           /dev/tty1 through /dev/tty6 are virtual consoles.
           /dev/pts/0 is a pseudo-terminal (your terminal emulator).
```

```bash
# Find out which shell you are using
echo $SHELL
echo $0

# See your current TTY
tty

# List available shells on the system
cat /etc/shells

# Switch to zsh (if installed)
zsh

# Switch back to bash
bash
```

---

## 2. Opening the Terminal in Ubuntu

| Method | How |
|---|---|
| **Keyboard shortcut** | `Ctrl + Alt + T` |
| **Right-click desktop** | "Open Terminal" |
| **Activities search** | Press Super key, type "terminal" |
| **Virtual console** | `Ctrl + Alt + F2` through `F6` |
| **Run dialog** | `Alt + F2`, type `gnome-terminal` |
| **File Manager** | Right-click in folder → Open Terminal Here |

```bash
# Open a new terminal window from within the terminal
gnome-terminal &

# Open a new terminal tab
# Ctrl + Shift + T (in GNOME Terminal)

# Go to a virtual console (text mode)
# Ctrl + Alt + F3
# Return to graphical session
# Ctrl + Alt + F1  (or F7 on older systems)
```

---

## 3. Understanding the Prompt

Before typing commands, understand what you are looking at:

```
┌──── Username ──────────────────────── Working directory ─────────────┐
│                                                                       │
john@ubuntu-pc:~/Documents$  _
         │                │
         │                └── $ = regular user   # = root user
         └── Hostname
```

```bash
# Customise your prompt (PS1 variable)
# Show current prompt setting
echo $PS1

# Set a colourful prompt with username@host:path$
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '

# To make permanent, add to ~/.bashrc
echo "PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '" >> ~/.bashrc
source ~/.bashrc
```

---

## 4. Command Syntax

Every Linux command follows this structure:

```
command  [options]  [arguments]
   │         │           │
   │         │           └── What to act ON (files, paths, text)
   │         └── How to modify behaviour (flags)
   └── What to DO
```

```bash
# Example breakdown:
ls -la /home/john
# └─┘ └┘ └───────┘
#  │   │      └── Argument: directory to list
#  │   └── Options: -l (long format) -a (show hidden)
#  └── Command: list directory contents

# Short options can be combined:
ls -l -a -h   # Equivalent to:
ls -lah

# Long options use double dash:
ls --all --human-readable --long
```

---

## 5. Essential Commands

### 5.1 Navigation

```bash
# Print Working Directory — where am I?
pwd
# Output: /home/john/Documents

# List files and directories
ls                    # Basic listing
ls -l                 # Long format (permissions, size, date)
ls -a                 # Show hidden files (starting with .)
ls -lah               # Long + hidden + human-readable sizes
ls -lt                # Sort by modification time (newest first)
ls -lS                # Sort by file size (largest first)
ls /etc               # List a specific directory

# Change Directory
cd /etc               # Go to absolute path
cd Documents          # Go to relative path
cd ..                 # Go up one level
cd ../..              # Go up two levels
cd ~                  # Go to home directory (also just: cd)
cd -                  # Go to previous directory (toggle)

# Make Directory
mkdir projects                        # Create one directory
mkdir -p projects/web/css             # Create nested directories
mkdir -p /tmp/{dir1,dir2,dir3}        # Create multiple dirs at once
```

### 5.2 File Creation and Viewing

```bash
# Create an empty file / update timestamp
touch file.txt
touch file1.txt file2.txt file3.txt   # Create multiple files

# View file contents
cat file.txt                           # Print entire file
cat -n file.txt                        # Print with line numbers
cat /etc/os-release                    # View system info file

# View large files (one page at a time)
less /etc/passwd                       # Scroll with arrow keys, q to quit
more /etc/passwd                       # Older pager (only scrolls forward)

# Show first/last N lines
head /var/log/syslog                   # First 10 lines (default)
head -20 /var/log/syslog               # First 20 lines
tail /var/log/syslog                   # Last 10 lines
tail -50 /var/log/syslog               # Last 50 lines
tail -f /var/log/syslog                # Follow file (live updates)

# Print text
echo "Hello, World!"
echo -e "Line 1\nLine 2\nLine 3"       # -e enables escape sequences
echo -n "No newline at end"            # -n suppresses trailing newline
printf "Name: %s, Age: %d\n" "Alice" 30
```

### 5.3 Copy, Move, Delete

```bash
# Copy files
cp source.txt destination.txt          # Copy a file
cp -r source_dir/ destination_dir/     # Copy a directory recursively
cp -v file.txt /tmp/                   # Verbose: show what was copied
cp -p file.txt backup.txt              # Preserve permissions and timestamps
cp -i file.txt existing.txt            # Interactive: ask before overwriting

# Move / Rename files
mv old_name.txt new_name.txt           # Rename a file
mv file.txt /tmp/                      # Move file to /tmp
mv -v *.txt /tmp/text_files/           # Move all .txt files, verbose
mv -i file.txt existing.txt            # Ask before overwriting

# Delete files
rm file.txt                            # Remove a file
rm -v file.txt                         # Verbose removal
rm -i file.txt                         # Interactive: ask to confirm
rm *.log                               # Remove all .log files
rm -r directory/                       # Remove directory and contents
rm -rf /tmp/old_dir/                   # Force remove (no prompts) — use carefully!

# Delete empty directory
rmdir empty_dir/
```

> ⚠️ **`rm -rf` has no undo.** Unlike Windows Recycle Bin, deleted files in Linux are gone. Always double-check before running `rm -rf`.

### 5.4 System Information

```bash
# Clear the terminal screen
clear                  # Or press Ctrl + L

# Show command history
history                # All previous commands
history 20             # Last 20 commands
history | grep ssh     # Search history for ssh commands

# Re-run a command from history
!!                     # Re-run the last command
!42                    # Re-run command number 42
!ssh                   # Re-run last command starting with "ssh"
sudo !!                # Re-run last command with sudo

# System information
uname -a               # Kernel and OS info
hostname               # Computer name
whoami                 # Current username
id                     # User ID, group ID, group memberships
uptime                 # How long the system has been running
date                   # Current date and time
cal                    # Display a calendar
```

### 5.5 Help and Documentation

```bash
# Manual pages — the built-in documentation system
man ls                 # Manual for ls command
man -k "copy"          # Search manual pages for keyword
man 5 passwd           # Section 5 (file formats) for passwd

# Inside man: navigate with arrows, search with /, quit with q

# Quick help
ls --help              # Brief usage information
cp --help | head -20   # First 20 lines of help

# What is this command? (one-line description)
whatis ls
whatis grep
whatis bash

# Find commands related to a word
apropos "compress"
apropos "network" | head -10

# Find the location of a command
which ls               # /usr/bin/ls
which python3
whereis ls             # More info: binary, man page, source locations
type ls                # What type of command is ls? (builtin/alias/binary)
```

---

## 6. Standard Input, Output, and Error

Every process in Linux has three standard streams:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│    Keyboard ──────► stdin  (fd 0) ──────► ┌──────────┐        │
│                                           │ PROCESS  │        │
│    Terminal ◄────── stdout (fd 1) ◄────── │          │        │
│                                           │  (ls,    │        │
│    Terminal ◄────── stderr (fd 2) ◄────── │  grep,   │        │
│                                           │  etc.)   │        │
│                                           └──────────┘        │
│                                                                 │
│   fd = file descriptor                                         │
│   0 = stdin  (standard input)                                  │
│   1 = stdout (standard output)                                 │
│   2 = stderr (standard error)                                  │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Demonstration: stdin and stdout
echo "test"           # echo writes to stdout
ls /nonexistent       # ls writes error to stderr

# Read from stdin manually (type, then Ctrl+D to end)
cat
Hello World
Ctrl+D
```

---

## 7. Redirection

```bash
# ─── Output Redirection ───────────────────────────────────────

# > Redirect stdout to a file (overwrites)
echo "Hello" > output.txt
ls -la > file_list.txt

# >> Append stdout to a file (does not overwrite)
echo "Line 1" > log.txt
echo "Line 2" >> log.txt
echo "Line 3" >> log.txt
cat log.txt

# 2> Redirect stderr to a file
ls /nonexistent 2> errors.txt
cat errors.txt

# &> or 2>&1 Redirect both stdout AND stderr
ls /nonexistent > all_output.txt 2>&1
ls /nonexistent &> all_output.txt        # Bash shorthand

# Discard output entirely (send to /dev/null — the black hole)
command_with_lots_of_output > /dev/null
ls /nonexistent 2> /dev/null             # Suppress errors
ls /nonexistent &> /dev/null             # Suppress everything

# ─── Input Redirection ────────────────────────────────────────

# < Read stdin from a file
cat < /etc/hostname
sort < unsorted.txt

# << Here Document (inline multi-line input)
cat << EOF
Line 1
Line 2
Line 3
EOF

# <<< Here String (single-line string to stdin)
grep "root" <<< "root:x:0:0:root:/root:/bin/bash"
```

---

## 8. Pipes

The **pipe** (`|`) connects the stdout of one command to the stdin of the next:

```
cmd1 | cmd2 | cmd3 | cmd4
 │         │         │
 └─stdout──► stdin   │
              └─stdout──► stdin
                          └─stdout──► terminal
```

```bash
# Basic pipe examples
ls -la | grep "\.txt"              # List only .txt files
cat /etc/passwd | head -5          # Show first 5 lines of passwd
ps aux | grep firefox              # Find Firefox process

# Multi-stage pipelines
# "What are the 5 biggest directories in /usr?"
du -sh /usr/* 2>/dev/null | sort -rh | head -5

# "Show unique error messages in syslog"
grep "error" /var/log/syslog 2>/dev/null | awk '{print $5}' | sort -u

# "How many open TCP connections?"
ss -tn | tail -n +2 | wc -l

# "Most recently modified files in /etc"
ls -lt /etc | head -10

# Count words in all text files in current directory
cat *.txt 2>/dev/null | wc -w

# "Which users are currently logged in, deduplicated?"
who | awk '{print $1}' | sort -u
```

---

## 9. Keyboard Shortcuts

These shortcuts work in bash and most terminals:

### Navigation Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + A` | Move cursor to beginning of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + ←` | Move cursor one word left |
| `Ctrl + →` | Move cursor one word right |
| `Alt + B` | Move cursor back one word |
| `Alt + F` | Move cursor forward one word |

### Editing Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + U` | Cut everything before cursor |
| `Ctrl + K` | Cut everything after cursor |
| `Ctrl + W` | Cut one word before cursor |
| `Ctrl + Y` | Paste (yank) cut text |
| `Ctrl + _` | Undo last edit |
| `Alt + D` | Delete word after cursor |

### Process Control

| Shortcut | Action |
|---|---|
| `Ctrl + C` | Interrupt/kill current process |
| `Ctrl + Z` | Suspend process (send to background) |
| `Ctrl + D` | Send EOF / exit shell |
| `Ctrl + L` | Clear screen (same as `clear`) |
| `Ctrl + S` | Pause terminal output |
| `Ctrl + Q` | Resume terminal output |

### History Shortcuts

| Shortcut | Action |
|---|---|
| `↑` / `↓` | Navigate command history |
| `Ctrl + R` | Reverse search history |
| `Ctrl + G` | Cancel reverse search |
| `Ctrl + P` | Previous command (same as ↑) |
| `Ctrl + N` | Next command (same as ↓) |

```bash
# Reverse search demonstration
# Press Ctrl+R, then type part of a previous command
# Press Ctrl+R again to cycle through matches
# Press Enter to execute, or Esc to edit

# Example: search for a previous git command
# Ctrl+R, type "git", then cycle through git commands
```

---

## 10. Tab Completion

Tab completion is one of the most important efficiency tools in the terminal:

```bash
# Complete a command name
gi<TAB>         → git
sy<TAB><TAB>    → systemctl  systemd  (shows all matches)

# Complete a file or directory path
ls /etc/passw<TAB>  → ls /etc/passwd
cd ~/Doc<TAB>       → cd ~/Documents/

# Complete a command option
ls --<TAB><TAB>     → shows all available options for ls

# Complete after a variable
echo $HO<TAB>   → echo $HOME
echo $PA<TAB>   → echo $PATH

# Cycle through matches
ls /etc/p<TAB><TAB>   # Shows all files in /etc starting with p
```

---

## 11. Command Chaining

```bash
# ; Run commands sequentially (regardless of success/failure)
mkdir /tmp/test ; cd /tmp/test ; touch file.txt

# && Run next command ONLY if previous succeeded (exit code 0)
sudo apt update && sudo apt upgrade -y

# || Run next command ONLY if previous FAILED (non-zero exit code)
mkdir /tmp/mydir || echo "Directory already exists"

# Combine && and ||
ping -c1 google.com > /dev/null && echo "Internet: OK" || echo "Internet: OFFLINE"

# Check exit code of last command
ls /etc/passwd
echo $?     # 0 = success

ls /nonexistent 2>/dev/null
echo $?     # 2 = failure (non-zero)
```

---

## Practice Exercises

### Exercise 1 — Navigation Challenge
Without using the mouse, navigate to and explore your system:

```bash
# Go to your home directory
cd ~
# Go to /etc
cd /etc
# Go back to home without typing the full path
cd -
# Go up two directory levels from /usr/share/doc
cd /usr/share/doc && cd ../..
# Confirm where you are
pwd
```

### Exercise 2 — File Manipulation
Create a directory structure and manipulate files:

```bash
# Create the structure
mkdir -p ~/practice/subdir1 ~/practice/subdir2

# Create files
touch ~/practice/file1.txt ~/practice/file2.txt
echo "Hello Linux" > ~/practice/file1.txt
echo "Learning is fun" >> ~/practice/file1.txt

# Copy and move
cp ~/practice/file1.txt ~/practice/subdir1/
mv ~/practice/file2.txt ~/practice/subdir2/

# Verify
ls -R ~/practice

# Clean up
rm -r ~/practice
```

### Exercise 3 — Redirection Mastery
Practice all forms of redirection:

```bash
# Redirect stdout to file
ls -la /etc > /tmp/etc_listing.txt
wc -l /tmp/etc_listing.txt

# Append to file
date >> /tmp/etc_listing.txt
tail -3 /tmp/etc_listing.txt

# Redirect stderr
ls /nonexistent 2> /tmp/errors.txt
cat /tmp/errors.txt

# Redirect both to separate files
ls /etc /nonexistent > /tmp/good.txt 2> /tmp/bad.txt
cat /tmp/good.txt
cat /tmp/bad.txt

# Cleanup
rm /tmp/etc_listing.txt /tmp/errors.txt /tmp/good.txt /tmp/bad.txt
```

### Exercise 4 — Pipeline Building
Build pipelines to answer these questions:

```bash
# Q1: How many files are in /etc?
ls /etc | wc -l

# Q2: What are the 5 most recently modified files in /var/log?
ls -lt /var/log | head -6 | tail -5

# Q3: How many lines contain the word "error" (case insensitive) in syslog?
grep -ic "error" /var/log/syslog 2>/dev/null || echo "Log not accessible"

# Q4: What is the longest filename in /usr/bin?
ls /usr/bin | awk '{print length, $0}' | sort -rn | head -5
```

### Exercise 5 — History Investigation
Explore your command history:

```bash
history | wc -l             # How many commands in history?
history | tail -10          # Your last 10 commands
history | grep "sudo" | wc -l  # How many times have you used sudo?
```

### Exercise 6 — Keyboard Shortcuts Practice
Practice these shortcuts (they become muscle memory with repetition):

1. Type a long command, then press `Ctrl+A` — cursor jumps to start
2. Press `Ctrl+E` — cursor jumps to end
3. Press `Ctrl+W` — deletes last word
4. Press `Ctrl+U` — deletes entire line
5. Press `Ctrl+R`, type "ls" — reverse searches history

### Exercise 7 — stdin/stdout Experiment
Observe the three streams in action:

```bash
# This sends to stdout (fd 1)
echo "stdout message"

# This sends to stderr (fd 2)
ls /nonexistent

# Capture only stdout
ls /etc /nonexistent > /tmp/stdout_only.txt 2>/dev/null
cat /tmp/stdout_only.txt | head -5

# Capture only stderr
ls /etc /nonexistent 2> /tmp/stderr_only.txt > /dev/null
cat /tmp/stderr_only.txt

rm /tmp/stdout_only.txt /tmp/stderr_only.txt
```

### Exercise 8 — Command Discovery
Use `man`, `whatis`, and `--help` to discover commands you have not seen yet:

```bash
whatis cal          # What does cal do?
whatis factor       # What does factor do?
whatis fortune      # What does fortune do?
cal 2025            # Try running it
echo "16" | factor  # And this

# Install fortune if not present
sudo apt install fortune -y 2>/dev/null
fortune
```

---

## Common Mistakes

| Mistake | Example | Fix |
|---|---|---|
| `>` instead of `>>` | `echo "line" > log.txt` (destroys previous content) | Use `>>` to append |
| `rm -rf` without care | `rm -rf /home/user /` (note the space!) | Use `rm -ri` while learning |
| Missing spaces in commands | `ls-la` instead of `ls -la` | Commands and flags need spaces |
| Wrong quote types | Using curly `"` instead of straight `"` | Use only ASCII straight quotes |
| `cd` vs path argument | Running `cd ls /home` | Navigate first, then act |
| Forgetting `2>/dev/null` | Seeing error messages in pipeline output | Redirect stderr appropriately |

---

## Pro Tips

> 💡 **`Ctrl+R` is your best friend.** Reverse history search finds any previous command in milliseconds. Master it early.

> 💡 **`!$` is the last argument of the previous command.** If you typed `vim /etc/nginx/nginx.conf`, then `cat !$` reads that same file.

> 💡 **`alias` saves keystrokes.** Add `alias ll='ls -lah'` to `~/.bashrc` for a handy long listing.

> 💡 **`Ctrl+Z` + `bg` sends a process to background.** `fg` brings it back. `jobs` lists background jobs.

> 💡 **`script` records your terminal session.** `script session.log` records everything; `exit` stops recording.

---

## Key Takeaways

- Shell = command interpreter; Terminal = window showing the shell; TTY = any terminal device
- Command syntax: `command [options] [arguments]` — learn to read man pages
- Essential commands: `ls`, `cd`, `pwd`, `mkdir`, `touch`, `cp`, `mv`, `rm`, `cat`, `echo`, `man`
- stdin (0), stdout (1), stderr (2) are the three streams every process has
- `>` overwrites, `>>` appends, `2>` captures errors, `|` pipes output to next command
- Tab completion and `Ctrl+R` history search are essential speed tools
- Exit codes: 0 = success, non-zero = failure — used by `&&` and `||` chaining

---

## Next Lesson Preview

**05 — File System Structure**

Every file has a place, and every place has a purpose. We will explore the Linux Filesystem Hierarchy Standard (FHS) — what lives in `/bin`, `/etc`, `/home`, `/proc`, `/var` and every other directory in the tree. You will also learn about file types, hidden files, and important configuration files that control how your system behaves.
