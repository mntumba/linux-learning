# 11 — Shell Scripting Fundamentals

> **Module 3 · Lesson 1** | Difficulty: ★★★☆☆ Intermediate | Time: ~120 min

---

## Learning Objectives

- Write and execute bash scripts
- Use variables, arrays, and special variables
- Implement control flow (if/elif/else, case, loops)
- Write functions and use parameters
- Handle errors and exit codes
- Process command-line arguments
- Apply string and arithmetic operations

---

## Table of Contents

1. [Your First Script](#1-your-first-script)
2. [Variables](#2-variables)
3. [User Input](#3-user-input)
4. [Conditional Statements](#4-conditional-statements)
5. [Loops](#5-loops)
6. [Functions](#6-functions)
7. [Arrays](#7-arrays)
8. [String Operations](#8-string-operations)
9. [Arithmetic](#9-arithmetic)
10. [Error Handling](#10-error-handling)
11. [Script Arguments](#11-script-arguments)
12. [Practical Scripts](#12-practical-scripts)
13. [Practice Exercises](#13-practice-exercises)
14. [Key Takeaways](#14-key-takeaways)

---

## 1. Your First Script

### Creating and Running Scripts

```bash
# Create script file
nano ~/hello.sh

#!/bin/bash
# This is a comment
# Shebang line ↑ tells OS which interpreter to use

echo "Hello, World!"
echo "Today is: $(date)"
echo "You are: $(whoami)"
echo "Running on: $(hostname)"
```

```bash
# Make executable
chmod +x ~/hello.sh

# Run it
./hello.sh           # relative path
bash ~/hello.sh      # explicit interpreter
/home/alice/hello.sh # absolute path

# Debug mode
bash -x ~/hello.sh   # trace execution
set -x               # enable trace within script
set +x               # disable trace
```

### Shebang Lines

```bash
#!/bin/bash          # bash (most common)
#!/bin/sh            # POSIX sh (more portable)
#!/usr/bin/env bash  # find bash in PATH (more portable)
#!/usr/bin/env python3  # Python script
#!/usr/bin/env perl  # Perl script
```

---

## 2. Variables

```bash
#!/bin/bash

# Variable assignment (NO spaces around =)
name="Alice"
age=30
PI=3.14159

# Reading variables (use $)
echo "Name: $name"
echo "Age: ${age}"       # {} for disambiguation

# Wrong way
name = "Alice"   # ERROR: command not found

# Command substitution
today=$(date +%Y-%m-%d)
kernel=$(uname -r)
user_count=$(who | wc -l)
echo "Today: $today, Kernel: $kernel, Users: $user_count"

# Environment variables
echo "Home: $HOME"
echo "User: $USER"
echo "Path: $PATH"
echo "Shell: $SHELL"
echo "PWD: $PWD"
echo "HOSTNAME: $HOSTNAME"

# Export (make available to child processes)
export MY_VAR="visible to children"
export -p              # list all exported variables

# Readonly variables
readonly CONST="unchangeable"
CONST="new value"   # Error: CONST: readonly variable

# Unsetting variables
unset name
echo $name             # empty
```

### Variable Quoting

```bash
name="Alice Smith"
echo $name           # Alice Smith (two words, split by shell)
echo "$name"         # Alice Smith (preserved as one argument)
echo '$name'         # $name (literal, no expansion)

# Important: always quote variables!
file="my file.txt"
cat $file            # Error: my and file.txt treated as separate args
cat "$file"          # Correct: "my file.txt" as one argument

rm $file             # DANGEROUS if $file has spaces
rm "$file"           # Safe
```

---

## 3. User Input

```bash
#!/bin/bash

# Read with prompt
read -p "Enter your name: " name
echo "Hello, $name!"

# Read password (no echo)
read -s -p "Password: " password
echo ""  # newline after silent input

# Read with timeout
read -t 10 -p "Answer (10 sec): " answer || echo "Timeout!"

# Read multiple values
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read into array
read -a colors -p "Enter colors (space-separated): "
echo "First color: ${colors[0]}"

# Read from file
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Read with default value
read -p "Enter username [$USER]: " username
username="${username:-$USER}"
echo "Using: $username"
```

---

## 4. Conditional Statements

### if/elif/else

```bash
#!/bin/bash

# Basic if
if [ condition ]; then
    commands
fi

# if-else
if [ $age -ge 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi

# if-elif-else
score=75
if [ $score -ge 90 ]; then
    echo "A"
elif [ $score -ge 80 ]; then
    echo "B"
elif [ $score -ge 70 ]; then
    echo "C"
else
    echo "F"
fi
```

### Test Operators

```bash
# Numeric comparisons
[ $a -eq $b ]   # equal
[ $a -ne $b ]   # not equal
[ $a -lt $b ]   # less than
[ $a -le $b ]   # less than or equal
[ $a -gt $b ]   # greater than
[ $a -ge $b ]   # greater than or equal

# String comparisons
[ "$a" = "$b" ]  # equal (POSIX)
[ "$a" == "$b" ] # equal (bash)
[ "$a" != "$b" ] # not equal
[ -z "$a" ]      # empty string
[ -n "$a" ]      # non-empty string
[[ "$a" < "$b" ]] # lexicographically less (use [[ ]])
[[ "$str" =~ ^[0-9]+$ ]]  # regex match (bash only)

# File tests
[ -e file ]     # exists
[ -f file ]     # is regular file
[ -d file ]     # is directory
[ -l file ]     # is symbolic link
[ -r file ]     # is readable
[ -w file ]     # is writable
[ -x file ]     # is executable
[ -s file ]     # is not empty
[ file1 -nt file2 ]  # file1 newer than file2
[ file1 -ot file2 ]  # file1 older than file2

# Logical operators
[ $a -gt 0 ] && [ $b -gt 0 ]  # AND
[ $a -gt 0 ] || [ $b -gt 0 ]  # OR
[ ! -f file ]                  # NOT

# [[ ]] is more powerful (bash only)
[[ $a -gt 0 && $b -gt 0 ]]    # AND
[[ $a -gt 0 || $b -gt 0 ]]    # OR
[[ ! -f file ]]                # NOT
[[ "$str" == *"sub"* ]]        # glob matching
```

### case Statement

```bash
#!/bin/bash

day=$(date +%A)

case "$day" in
    Monday|Tuesday|Wednesday|Thursday|Friday)
        echo "Weekday"
        ;;
    Saturday|Sunday)
        echo "Weekend"
        ;;
    *)
        echo "Unknown day"
        ;;
esac

# Another example: handle command arguments
case "$1" in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    status)
        echo "Checking status..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

---

## 5. Loops

### for Loop

```bash
# Iterate over list
for item in one two three four; do
    echo "$item"
done

# Iterate over files
for file in *.txt; do
    echo "Processing: $file"
    wc -l "$file"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "i = $i"
done

# Range
for i in {1..10}; do
    echo "Number: $i"
done

# Range with step
for i in {0..20..5}; do
    echo "$i"    # 0 5 10 15 20
done

# Iterate over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# Iterate over array
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done
```

### while Loop

```bash
#!/bin/bash

# While condition is true
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts

# Infinite loop with break
while true; do
    read -p "Enter 'quit' to exit: " input
    if [ "$input" = "quit" ]; then
        break
    fi
    echo "You entered: $input"
done

# While loop processing command output
ps aux | while read line; do
    echo "Process: $line"
done
```

### until Loop

```bash
# Until condition becomes true (opposite of while)
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Loop Control

```bash
# break — exit loop
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break        # exits the for loop
    fi
    echo $i
done

# continue — skip to next iteration
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue     # skip even numbers
    fi
    echo $i
done

# break with level (for nested loops)
for i in {1..3}; do
    for j in {1..3}; do
        if [ $j -eq 2 ]; then
            break 2  # breaks outer loop too
        fi
        echo "$i $j"
    done
done
```

---

## 6. Functions

```bash
#!/bin/bash

# Define function
greet() {
    echo "Hello, $1!"    # $1 = first argument
}

# Call function
greet "Alice"
greet "World"

# Function with return value
add() {
    local result=$(($1 + $2))
    echo $result        # return via stdout
}

sum=$(add 5 3)
echo "Sum: $sum"

# Return codes (0-255, 0 = success)
is_file() {
    if [ -f "$1" ]; then
        return 0    # success (true)
    else
        return 1    # failure (false)
    fi
}

if is_file "/etc/passwd"; then
    echo "File exists"
fi

# Local variables (function scope)
my_func() {
    local local_var="I am local"
    global_var="I am global"
    echo "$local_var"
}
my_func
echo "$global_var"   # works
echo "$local_var"    # empty (out of scope)

# Functions with error handling
backup_file() {
    local source="$1"
    local dest="${2:-/tmp/}"   # default destination

    if [ ! -f "$source" ]; then
        echo "ERROR: $source not found" >&2
        return 1
    fi

    cp "$source" "$dest" || {
        echo "ERROR: copy failed" >&2
        return 2
    }

    echo "Backed up $source to $dest"
    return 0
}
```

---

## 7. Arrays

```bash
#!/bin/bash

# Indexed arrays
fruits=("apple" "banana" "cherry")
fruits[3]="date"                    # add element

echo "${fruits[0]}"                 # first element
echo "${fruits[-1]}"                # last element
echo "${fruits[@]}"                 # all elements
echo "${#fruits[@]}"               # array length
echo "${!fruits[@]}"               # all indices

# Iterate
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Slice
echo "${fruits[@]:1:2}"            # elements 1 and 2

# Add to array
fruits+=("elderberry")

# Remove element
unset fruits[2]

# Associative arrays (like hash maps)
declare -A user_info
user_info[name]="Alice"
user_info[age]=30
user_info[email]="alice@example.com"

echo "${user_info[name]}"
echo "${user_info[@]}"             # all values
echo "${!user_info[@]}"           # all keys

for key in "${!user_info[@]}"; do
    echo "$key = ${user_info[$key]}"
done
```

---

## 8. String Operations

```bash
#!/bin/bash

str="Hello, World!"

# Length
echo "${#str}"                # 13

# Uppercase/Lowercase
echo "${str^^}"               # HELLO, WORLD!
echo "${str,,}"               # hello, world!
echo "${str^}"                # Hello, World! (first char upper)

# Substring
echo "${str:7}"               # World!
echo "${str:7:5}"             # World

# Remove prefix/suffix
file="path/to/file.txt"
echo "${file#*/}"             # to/file.txt  (remove shortest prefix)
echo "${file##*/}"            # file.txt     (remove longest prefix = basename)
echo "${file%.txt}"           # path/to/file (remove suffix)
echo "${file%/*}"             # path/to      (remove last / and after = dirname)

# Replace
echo "${str/World/Linux}"     # Hello, Linux! (first occurrence)
echo "${str//l/L}"            # HeLLo, WorLd! (all occurrences)

# Default values
unset myvar
echo "${myvar:-default}"     # default (use default if unset)
echo "${myvar:=default}"     # default AND set myvar to "default"
echo "${myvar:+exists}"      # "" if unset; "exists" if set
echo "${myvar:?error msg}"   # print error and exit if unset

# Check if string contains substring
if [[ "$str" == *"World"* ]]; then
    echo "Contains World"
fi

# Split string
IFS=',' read -ra parts <<< "a,b,c,d"
for part in "${parts[@]}"; do
    echo "$part"
done
```

---

## 9. Arithmetic

```bash
#!/bin/bash

# Arithmetic expansion
echo $((5 + 3))          # 8
echo $((10 - 4))         # 6
echo $((3 * 7))          # 21
echo $((15 / 4))         # 3 (integer division)
echo $((15 % 4))         # 3 (modulo)
echo $((2 ** 10))        # 1024 (exponentiation)

# Increment/Decrement
x=5
((x++))                  # post-increment
((x--))                  # post-decrement
((++x))                  # pre-increment
((x += 10))              # add and assign

# Variables in arithmetic
a=10
b=3
result=$((a * b + 2))
echo "Result: $result"

# let command
let "y = 5 * 3"
echo $y                  # 15

# expr (older, slower)
result=$(expr 5 + 3)     # needs spaces around operators!
echo $result

# Floating point (use bc)
echo "3.14 * 2" | bc                     # basic
echo "scale=4; 22/7" | bc               # 4 decimal places
result=$(echo "scale=2; sqrt(2)" | bc)   # square root
```

---

## 10. Error Handling

```bash
#!/bin/bash

# Exit on error (best practice!)
set -e              # exit immediately if any command fails
set -u              # treat unset variables as errors
set -o pipefail     # catch errors in pipes
set -x              # trace (for debugging)

# Combined: commonly used at top of scripts
set -euo pipefail

# Exit codes
ls /etc/passwd
echo "Exit code: $?"  # 0 = success

ls /nonexistent
echo "Exit code: $?"  # non-zero = failure

# Check exit code explicitly
if ! cp source.txt dest.txt; then
    echo "Copy failed!" >&2
    exit 1
fi

# Error function
die() {
    echo "ERROR: $*" >&2
    exit 1
}

# Trap errors
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/lockfile
}
trap cleanup EXIT          # run cleanup on any exit
trap 'die "Error on line $LINENO"' ERR  # on any error

# Conditional execution
command && echo "Success" || echo "Failed"
mkdir -p /tmp/test && echo "Directory created"

# Ignore errors for specific command
ignore_errors_here || true
```

---

## 11. Script Arguments

```bash
#!/bin/bash
# Usage: ./script.sh [options] arg1 arg2

# Special parameters
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
echo "All as one string: $*"

# Process arguments with shift
while [ $# -gt 0 ]; do
    echo "Arg: $1"
    shift           # remove $1, shift others down
done

# Process options with getopts
while getopts "hvn:f:" opt; do
    case $opt in
        h)  echo "Usage: $0 [-h] [-v] [-n name] [-f file]"; exit 0 ;;
        v)  VERBOSE=true ;;
        n)  NAME="$OPTARG" ;;  # OPTARG contains the value after -n
        f)  FILE="$OPTARG" ;;
        ?)  echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))   # remove processed options, leave positional args

echo "Name: ${NAME:-default}"
echo "File: ${FILE:-none}"
echo "Verbose: ${VERBOSE:-false}"
echo "Remaining args: $@"
```

---

## 12. Practical Scripts

### System Information Script

```bash
#!/bin/bash
# system_info.sh — Display system information

set -euo pipefail

print_header() {
    echo "================================"
    echo "  $1"
    echo "================================"
}

print_header "SYSTEM INFORMATION"
echo "Hostname:    $(hostname)"
echo "OS:          $(lsb_release -ds)"
echo "Kernel:      $(uname -r)"
echo "Uptime:      $(uptime -p)"
echo "Date:        $(date)"

print_header "HARDWARE"
echo "CPU:         $(grep 'model name' /proc/cpuinfo | head -1 | cut -d: -f2 | xargs)"
echo "CPU Cores:   $(nproc)"
echo "RAM:         $(free -h | awk '/Mem/{print $2}')"
echo "Disk:        $(df -h / | awk 'NR==2{print $2}')"

print_header "NETWORK"
ip addr show | awk '/inet /{print "  " $2}' | head -5

print_header "TOP PROCESSES"
ps aux --sort=-%cpu | head -6 | tail -5

echo ""
echo "Report generated: $(date)"
```

### File Backup Script

```bash
#!/bin/bash
# backup.sh — Backup a directory with rotation

set -euo pipefail

SOURCE="${1:?Usage: $0 <source_dir> [dest_dir]}"
DEST="${2:-/tmp/backups}"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"
MAX_BACKUPS=7

# Create destination if needed
mkdir -p "$DEST"

# Create backup
echo "Backing up $SOURCE..."
tar -czf "${DEST}/${BACKUP_FILE}" "$SOURCE"
echo "Created: ${DEST}/${BACKUP_FILE} ($(du -sh "${DEST}/${BACKUP_FILE}" | cut -f1))"

# Rotate old backups (keep only MAX_BACKUPS)
cd "$DEST"
BACKUP_COUNT=$(ls -1 backup_*.tar.gz 2>/dev/null | wc -l)
if [ "$BACKUP_COUNT" -gt "$MAX_BACKUPS" ]; then
    echo "Rotating backups (keeping last $MAX_BACKUPS)..."
    ls -1t backup_*.tar.gz | tail -n +$((MAX_BACKUPS + 1)) | xargs rm -v
fi

echo "Done! Backup count: $(ls -1 backup_*.tar.gz | wc -l)"
```

---

## 13. Practice Exercises

### Exercise 11.1 — Basic Scripts

```bash
# 1. Write a script that takes a username as argument
# and displays: UID, home directory, and groups
cat << 'EOF' > ~/user_info.sh
#!/bin/bash
USER="${1:?Usage: $0 <username>}"
echo "User: $USER"
echo "UID: $(id -u "$USER")"
echo "Home: $(getent passwd "$USER" | cut -d: -f6)"
echo "Groups: $(groups "$USER")"
EOF
chmod +x ~/user_info.sh
./user_info.sh alice
```

### Exercise 11.2 — File Organizer

```bash
# Write a script that organizes files by extension
cat << 'EOF' > ~/organize.sh
#!/bin/bash
# Move files from current directory into subdirs by extension
for file in *; do
    [ -f "$file" ] || continue
    ext="${file##*.}"
    [ "$ext" = "$file" ] && ext="noext"
    mkdir -p "$ext"
    mv "$file" "$ext/"
    echo "Moved: $file → $ext/"
done
EOF
chmod +x ~/organize.sh
```

### Exercise 11.3 — Log Monitor

```bash
cat << 'EOF' > ~/monitor_log.sh
#!/bin/bash
# Monitor a log file for keywords and alert
LOG="${1:-/var/log/syslog}"
KEYWORD="${2:-ERROR}"
INTERVAL=5

echo "Monitoring $LOG for '$KEYWORD' every ${INTERVAL}s..."
echo "Press Ctrl+C to stop"

last_count=$(grep -c "$KEYWORD" "$LOG" 2>/dev/null || echo 0)

while true; do
    sleep "$INTERVAL"
    new_count=$(grep -c "$KEYWORD" "$LOG" 2>/dev/null || echo 0)
    if [ "$new_count" -gt "$last_count" ]; then
        diff=$((new_count - last_count))
        echo "$(date): $diff new '$KEYWORD' entries found!"
        tail -"$diff" "$LOG" | grep "$KEYWORD"
    fi
    last_count=$new_count
done
EOF
chmod +x ~/monitor_log.sh
```

---

## 14. Key Takeaways

- Always start with `#!/bin/bash` shebang and `set -euo pipefail` for robust scripts
- **Quote all variables** — `"$var"` not `$var` to handle spaces
- Use **`[[ ]]`** instead of `[ ]` in bash for more features
- Functions use **`local`** for local scope; communicate via stdout or return codes
- **`$1`, `$2`, `$@`, `$#`** for arguments; `getopts` for option parsing
- Arithmetic: `$((expr))` for integers; `bc` for floating point
- Use **trap** for cleanup and error handling
- Always **test scripts** with `bash -n` (syntax) and `bash -x` (trace)

---

## Next Lesson

➡️ [12 — System Administration](12_System_Administration.md)

---

*Module 3 · Lesson 1 of 5 | [Course Index](../INDEX.md)*
