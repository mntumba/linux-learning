# 11. Shell Scripting Fundamentals

> **Difficulty:** Advanced | **Time Estimate:** 3–4 hours | **Prerequisites:** Lessons 1–10

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Write well-structured Bash shell scripts using best practices
- Use variables, arrays, and string operations effectively
- Implement control flow: conditionals, loops, and case statements
- Define and call functions with arguments and return values
- Handle errors gracefully using `set -e`, `set -x`, and `trap`
- Build practical scripts for backups, system info, and log rotation

---

## 1. What Is Shell Scripting?

A shell script is a text file containing a sequence of commands that the shell interprets and executes. Scripts automate repetitive tasks, enforce consistency, and form the backbone of Linux system administration.

### The Shebang Line

The first line of every script should specify the interpreter:

```bash
#!/bin/bash          # Use Bash explicitly
#!/usr/bin/env bash  # Portable: finds bash in PATH
#!/bin/sh            # POSIX-compatible shell (fewer features)
```

### Making a Script Executable

```bash
touch myscript.sh
chmod +x myscript.sh
./myscript.sh
```

---

## 2. Variables

### Declaring and Using Variables

```bash
#!/bin/bash

# Assign (no spaces around =)
name="Alice"
age=30
pi=3.14159

# Reference with $
echo "Name: $name"
echo "Age: ${age}"          # Braces recommended for clarity
echo "In 10 years: $((age + 10))"
```

### Special Variables

```bash
#!/bin/bash
# Run as: ./script.sh arg1 arg2

echo "Script name:       $0"
echo "First argument:    $1"
echo "Second argument:   $2"
echo "All arguments:     $@"
echo "Argument count:    $#"
echo "Last exit code:    $?"
echo "Current PID:       $$"
echo "Last background PID: $!"
```

### Reading User Input

```bash
#!/bin/bash

read -p "Enter your name: " username
read -s -p "Enter password: " passwd   # -s hides input
echo ""
echo "Hello, $username!"

# Read with timeout
read -t 10 -p "Answer in 10 seconds: " answer || echo "Timed out!"

# Read into array
read -a fruits -p "Enter fruits (space-separated): "
echo "First fruit: ${fruits[0]}"
```

---

## 3. Arithmetic

```bash
#!/bin/bash

a=10
b=3

# Arithmetic expansion (preferred)
echo $((a + b))     # 13
echo $((a - b))     # 7
echo $((a * b))     # 30
echo $((a / b))     # 3 (integer division)
echo $((a % b))     # 1 (modulo)
echo $((a ** b))    # 1000 (exponentiation)

# let command
let result=a*b
echo "let result: $result"

# expr command (older, requires spaces)
result=$(expr $a + $b)
echo "expr result: $result"

# Floating point with bc
echo "scale=4; $a / $b" | bc    # 3.3333
```

---

## 4. String Operations

```bash
#!/bin/bash

str="Hello, World!"

echo "Length:          ${#str}"              # 13
echo "Uppercase:       ${str^^}"             # HELLO, WORLD!
echo "Lowercase:       ${str,,}"             # hello, world!
echo "Substring:       ${str:7:5}"           # World
echo "Replace first:   ${str/World/Linux}"   # Hello, Linux!
echo "Replace all:     ${str//l/L}"          # HeLLo, WorLd!
echo "Strip prefix:    ${str#Hello, }"       # World!
echo "Strip suffix:    ${str%, *}"           # Hello
echo "Default value:   ${unset_var:-default}" # default

# String comparison
if [[ "$str" == *"World"* ]]; then
    echo "String contains 'World'"
fi
```

---

## 5. Conditional Statements

```bash
#!/bin/bash

score=75

# if / elif / else
if [[ $score -ge 90 ]]; then
    echo "Grade: A"
elif [[ $score -ge 80 ]]; then
    echo "Grade: B"
elif [[ $score -ge 70 ]]; then
    echo "Grade: C"
else
    echo "Grade: F"
fi

# File tests
file="/etc/passwd"
if [[ -f "$file" ]]; then
    echo "$file exists and is a regular file"
fi
if [[ -r "$file" && -s "$file" ]]; then
    echo "$file is readable and non-empty"
fi
```

### Test Operators Reference

```
-f  file exists and is regular
-d  directory exists
-e  path exists
-r  readable     -w  writable    -x  executable
-s  non-empty    -L  symlink
-z  string is empty              -n  string is non-empty
==  string equal  !=  not equal  =~  regex match
-eq -ne -lt -le -gt -ge  (numeric comparisons)
```

### Case Statements

```bash
#!/bin/bash

read -p "Enter day (1-7): " day

case $day in
    1) echo "Monday" ;;
    2) echo "Tuesday" ;;
    3) echo "Wednesday" ;;
    4) echo "Thursday" ;;
    5) echo "Friday" ;;
    6|7) echo "Weekend!" ;;
    *) echo "Invalid day" ;;
esac

# Case with patterns
read -p "Enter a filename: " fname
case "$fname" in
    *.jpg|*.png|*.gif) echo "Image file" ;;
    *.mp3|*.wav)       echo "Audio file" ;;
    *.sh)              echo "Shell script" ;;
    *)                 echo "Unknown type" ;;
esac
```

---

## 6. Loops

```bash
#!/bin/bash

# --- for loop ---
for i in 1 2 3 4 5; do
    echo "Item: $i"
done

# C-style for
for ((i=0; i<5; i++)); do
    echo "Counter: $i"
done

# Iterate over files
for file in /etc/*.conf; do
    echo "Config: $(basename $file)"
done

# --- while loop ---
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hostname

# --- until loop ---
n=10
until [[ $n -le 0 ]]; do
    echo "Countdown: $n"
    ((n--))
done

# --- break and continue ---
for i in {1..10}; do
    [[ $i -eq 3 ]] && continue   # skip 3
    [[ $i -eq 7 ]] && break      # stop at 7
    echo "i = $i"
done
```

---

## 7. Functions

```bash
#!/bin/bash

# Define a function
greet() {
    local name="$1"      # local variable — not global
    local greeting="Hello, ${name}!"
    echo "$greeting"
}

# Call it
greet "Alice"
greet "Bob"

# Return values (via exit code 0-255)
is_even() {
    [[ $(($1 % 2)) -eq 0 ]]   # returns 0 (true) or 1 (false)
}

if is_even 4; then echo "4 is even"; fi

# Return a string via stdout capture
get_timestamp() {
    date +"%Y-%m-%d_%H-%M-%S"
}

ts=$(get_timestamp)
echo "Timestamp: $ts"

# Function with error handling
safe_mkdir() {
    local dir="$1"
    if [[ -d "$dir" ]]; then
        echo "Directory already exists: $dir" >&2
        return 1
    fi
    mkdir -p "$dir" && echo "Created: $dir"
}
```

---

## 8. Arrays and Associative Arrays

```bash
#!/bin/bash

# Indexed arrays
fruits=("apple" "banana" "cherry")
fruits+=("date")                  # append
echo "${fruits[0]}"               # apple
echo "${fruits[@]}"               # all elements
echo "${#fruits[@]}"              # count: 4
echo "${!fruits[@]}"              # indices: 0 1 2 3

# Slice
echo "${fruits[@]:1:2}"           # banana cherry

# Unset element
unset fruits[1]

# Iterate
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Associative arrays (Bash 4+)
declare -A capitals
capitals["France"]="Paris"
capitals["Germany"]="Berlin"
capitals["Japan"]="Tokyo"

echo "Capital of France: ${capitals[France]}"

for country in "${!capitals[@]}"; do
    echo "$country → ${capitals[$country]}"
done
```

---

## 9. File Operations in Scripts

```bash
#!/bin/bash

# Check and create
[[ -d /tmp/mydir ]] || mkdir /tmp/mydir

# Write to file
cat > /tmp/myfile.txt << 'EOF'
Line one
Line two
Line three
EOF

# Append
echo "Line four" >> /tmp/myfile.txt

# Read and process
while IFS=: read -r user pass uid gid info home shell; do
    echo "User: $user | Shell: $shell"
done < /etc/passwd | head -5

# Temporary files (safe)
tmpfile=$(mktemp /tmp/myapp.XXXXXX)
trap "rm -f $tmpfile" EXIT
echo "Working with: $tmpfile"
```

### Heredoc Syntax

```bash
#!/bin/bash

# Heredoc — preserves whitespace
cat << EOF
Server: $(hostname)
Date:   $(date)
User:   $(whoami)
EOF

# Indented heredoc (strip leading tabs with <<-)
cat <<- EOF
	This line is indented with a tab
	So is this one
EOF

# Heredoc into variable
report=$(cat << EOF
=== System Report ===
Uptime: $(uptime -p)
Memory: $(free -h | awk '/^Mem/ {print $3 "/" $2}')
Disk:   $(df -h / | awk 'NR==2 {print $3 "/" $2}')
EOF
)
echo "$report"
```

---

## 10. Error Handling

```bash
#!/bin/bash
set -euo pipefail   # Exit on error, unset vars, pipe failures

# set -e: exit on any non-zero return
# set -u: treat unset variables as errors
# set -o pipefail: catch pipe failures
# set -x: print each command before executing (debug)

# trap — run cleanup on exit/signal
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/lockfile.$$
}
trap cleanup EXIT              # on any exit
trap "echo 'Interrupted!'" INT # on Ctrl+C
trap "echo 'Error on line $LINENO'" ERR

# Create lockfile to prevent concurrent runs
LOCKFILE="/tmp/myscript.lock"
if [[ -f "$LOCKFILE" ]]; then
    echo "Script already running. Exiting." >&2
    exit 1
fi
touch "$LOCKFILE"

echo "Script running..."
```

---

## 11. Practical Script Examples

### Backup Script

```bash
#!/bin/bash
set -euo pipefail

BACKUP_SRC="${1:-$HOME}"
BACKUP_DEST="/backup"
DATE=$(date +%Y-%m-%d)
BACKUP_FILE="$BACKUP_DEST/backup_$DATE.tar.gz"

mkdir -p "$BACKUP_DEST"

echo "Backing up $BACKUP_SRC → $BACKUP_FILE"
tar -czf "$BACKUP_FILE" \
    --exclude="$BACKUP_SRC/.cache" \
    --exclude="$BACKUP_SRC/Downloads" \
    "$BACKUP_SRC"

SIZE=$(du -sh "$BACKUP_FILE" | cut -f1)
echo "Backup complete. Size: $SIZE"

# Keep only last 7 backups
ls -t "$BACKUP_DEST"/backup_*.tar.gz | tail -n +8 | xargs -r rm --
echo "Old backups pruned."
```

### System Info Script

```bash
#!/bin/bash

echo "============================================"
echo "         SYSTEM INFORMATION REPORT         "
echo "============================================"
printf "%-20s %s\n" "Hostname:"    "$(hostname -f)"
printf "%-20s %s\n" "OS:"          "$(. /etc/os-release; echo $PRETTY_NAME)"
printf "%-20s %s\n" "Kernel:"      "$(uname -r)"
printf "%-20s %s\n" "Uptime:"      "$(uptime -p)"
printf "%-20s %s\n" "CPU:"         "$(grep 'model name' /proc/cpuinfo | head -1 | cut -d: -f2 | xargs)"
printf "%-20s %s\n" "CPU Cores:"   "$(nproc)"
printf "%-20s %s\n" "RAM Total:"   "$(free -h | awk '/^Mem/ {print $2}')"
printf "%-20s %s\n" "RAM Used:"    "$(free -h | awk '/^Mem/ {print $3}')"
printf "%-20s %s\n" "Disk /:"      "$(df -h / | awk 'NR==2 {print $3 "/" $2 " (" $5 " used)"}')"
printf "%-20s %s\n" "IP Address:"  "$(hostname -I | awk '{print $1}')"
echo "============================================"
```

### User Management Script

```bash
#!/bin/bash
set -euo pipefail

ACTION="$1"
USERNAME="$2"

case "$ACTION" in
    create)
        if id "$USERNAME" &>/dev/null; then
            echo "User '$USERNAME' already exists." >&2
            exit 1
        fi
        useradd -m -s /bin/bash "$USERNAME"
        passwd "$USERNAME"
        echo "User '$USERNAME' created successfully."
        ;;
    delete)
        userdel -r "$USERNAME" && echo "User '$USERNAME' deleted."
        ;;
    lock)
        usermod -L "$USERNAME" && echo "User '$USERNAME' locked."
        ;;
    unlock)
        usermod -U "$USERNAME" && echo "User '$USERNAME' unlocked."
        ;;
    *)
        echo "Usage: $0 {create|delete|lock|unlock} <username>"
        exit 1
        ;;
esac
```

### Log Rotation Script

```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
MAX_LOGS=5
DATE=$(date +%Y%m%d_%H%M%S)

rotate_log() {
    local logfile="$1"
    if [[ -f "$logfile" && -s "$logfile" ]]; then
        mv "$logfile" "${logfile%.log}_${DATE}.log"
        gzip "${logfile%.log}_${DATE}.log"
        touch "$logfile"
        echo "Rotated: $logfile"
    fi
}

# Rotate all .log files
for log in "$LOG_DIR"/*.log; do
    rotate_log "$log"
done

# Remove old archives
ls -t "$LOG_DIR"/*.log.gz 2>/dev/null | tail -n +$((MAX_LOGS + 1)) | xargs -r rm --
echo "Log rotation complete."
```

---

## ASCII Diagram: Script Execution Flow

```
  #!/bin/bash (shebang)
         │
         ▼
  Variable Setup & Validation
         │
         ▼
  ┌──────────────────┐
  │  Argument Check  │──FAIL──► print usage, exit 1
  └──────────────────┘
         │ OK
         ▼
  ┌──────────────────┐
  │  Main Logic      │
  │  (loops, funcs)  │
  └──────────────────┘
         │
         ▼
  ┌──────────────────┐
  │  trap / cleanup  │◄── EXIT / ERR / INT signals
  └──────────────────┘
         │
         ▼
       exit 0
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| `x=10; if [ $x > 5 ]` | `>` is redirection in `[ ]` | Use `-gt` or `[[ ]]` |
| Missing quotes `$var` | Word splitting on spaces | Always quote: `"$var"` |
| `if [ $# = 0 ]` without quotes | Fails when var is empty | `if [[ $# -eq 0 ]]` |
| Using `==` in `[ ]` | Not POSIX in single brackets | Use `[[ ]]` for `==` |
| Forgetting `local` in functions | Pollutes global namespace | Prefix with `local` |
| `for i in $(ls)` | Breaks on spaces/globs | Use `for f in /path/*` |

---

## Pro Tips

- **Shellcheck** — run `shellcheck yourscript.sh` to catch common bugs before executing
- **Use `set -euo pipefail`** at the top of every production script
- **Prefer `[[ ]]`** over `[ ]` — it supports `&&`, `||`, regex, and avoids word splitting
- **Name scripts clearly**: `backup-db.sh` beats `script1.sh`
- **Log output**: redirect verbose output to a logfile with `exec > >(tee -a /var/log/myscript.log) 2>&1`
- **Dry-run mode**: add a `--dry-run` flag that prints commands without executing them

---

## Practice Exercises

1. **Variables & Input** — Write a script that asks for a user's name and birth year, then calculates and displays their age.

2. **File Validator** — Write a script that accepts a filename as an argument and reports whether it's a file, directory, symlink, or missing.

3. **Grade Calculator** — Write a function `letter_grade()` that accepts a numeric score (0–100) and echoes the letter grade (A/B/C/D/F).

4. **Directory Size Report** — Write a loop that iterates over all directories in `/etc` and prints each directory's name and size using `du -sh`.

5. **Number Guessing Game** — Write a script that generates a random number (1–100) and prompts the user to guess it, giving "too high" / "too low" hints.

6. **CSV Parser** — Write a script that reads a CSV file (columns: name, age, city) line by line and prints a formatted report.

7. **Backup with Rotation** — Extend the backup script to compress multiple directories and keep only the 5 most recent backups.

8. **Error Handling** — Write a script that installs a list of packages (from a file), logs successes and failures separately, and sends an exit code of 1 if any package failed.

---

## Key Takeaways

- The **shebang line** (`#!/bin/bash`) and `chmod +x` are prerequisites for executable scripts
- Always **quote variables** (`"$var"`) to prevent word splitting and glob expansion
- Use `[[ ]]` for conditionals — it is safer and more powerful than `[ ]`
- **Functions** with `local` variables prevent accidental global state pollution
- `set -euo pipefail` + `trap` is the foundation of production-grade error handling
- **Arrays** store ordered lists; **associative arrays** store key-value pairs
- Heredocs (`<< EOF`) cleanly embed multi-line text inside scripts

---

## Next Lesson Preview

**Lesson 12: System Administration** — You'll master `systemd` and `systemctl` to manage services, explore the Linux boot process from BIOS to userspace, schedule tasks with cron and systemd timers, and configure centralised logging with `journalctl`. System administration is where your shell scripting skills start running real infrastructure.
