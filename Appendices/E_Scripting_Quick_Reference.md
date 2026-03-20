# Appendix E – Bash Scripting Quick Reference

A concise reference for writing robust, maintainable Bash scripts covering
variables, arrays, strings, loops, conditionals, functions, and error handling.

---

## Table of Contents
1. [Script Basics & Best Practices](#1-script-basics--best-practices)
2. [Variables & Data Types](#2-variables--data-types)
3. [String Operations](#3-string-operations)
4. [Arrays & Associative Arrays](#4-arrays--associative-arrays)
5. [Arithmetic](#5-arithmetic)
6. [Conditionals](#6-conditionals)
7. [Test Operators](#7-test-operators)
8. [Loops](#8-loops)
9. [Functions](#9-functions)
10. [Input & Output](#10-input--output)
11. [Error Handling](#11-error-handling)
12. [Process & Job Control](#12-process--job-control)
13. [Pattern Matching & Parameter Expansion](#13-pattern-matching--parameter-expansion)
14. [Script Templates](#14-script-templates)

---

## 1. Script Basics & Best Practices

```bash
#!/usr/bin/env bash
# ^^^ Recommended shebang: portable, uses PATH to find bash

# Strict mode – recommended for all scripts
set -euo pipefail
# -e: exit on error
# -u: error on undefined variables
# -o pipefail: pipe fails if any command fails

# IFS (Internal Field Separator) – safe default
IFS=$'\n\t'

# Script metadata
readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# Make all variables local inside functions (avoid global pollution)
# Quote all variable expansions: "$var" not $var
# Use [[ ]] instead of [ ] for conditionals
# Use $(command) instead of backticks
```

### Debugging

```bash
# Enable trace mode (print each command before executing)
set -x

# Trace specific section only
set -x
# ... commands to trace ...
set +x

# Run script with debugging
bash -x script.sh

# Verbose mode (print each line before executing)
bash -v script.sh

# Dry-run check (syntax only)
bash -n script.sh
```

---

## 2. Variables & Data Types

```bash
# Assignment (no spaces around =)
name="Alice"
count=42
pi=3.14159

# Read-only variable
readonly MAX_RETRIES=3

# Environment variable (exported to subprocesses)
export DATABASE_URL="postgresql://localhost/mydb"

# Unset a variable
unset name

# Default values
echo "${name:-default}"          # Use "default" if unset or empty
echo "${name:=default}"          # Assign and use "default" if unset
echo "${name:+override}"         # Use "override" if set and non-empty
echo "${name:?Error: not set}"   # Print error and exit if unset

# Integer declaration
declare -i count=0
count+=1     # Arithmetic assignment

# Nameref (reference to another variable, bash 4.3+)
declare -n ref=name
echo "$ref"   # Same as echo "$name"

# Special variables
echo "$0"          # Script name
echo "$1" "$2"     # Positional parameters
echo "$@"          # All parameters as separate words (quoted)
echo "$*"          # All parameters as single word
echo "$#"          # Number of parameters
echo "$$"          # Current PID
echo "$!"          # PID of last background process
echo "$?"          # Exit status of last command
echo "$LINENO"     # Current line number
echo "$BASHPID"    # Current bash PID (differs from $$ in subshells)
```

---

## 3. String Operations

```bash
str="Hello, World!"

# Length
echo "${#str}"                  # 13

# Substring: ${var:offset:length}
echo "${str:0:5}"               # Hello
echo "${str:7}"                 # World!
echo "${str: -6}"               # orld!  (negative offset)

# Case conversion (bash 4+)
echo "${str,,}"                 # hello, world!  (lowercase)
echo "${str^^}"                 # HELLO, WORLD!  (uppercase)
echo "${str^}"                  # Hello, World!  (capitalize first)

# Pattern removal
path="/usr/local/bin/script.sh"
echo "${path#*/}"               # usr/local/bin/script.sh (remove shortest prefix)
echo "${path##*/}"              # script.sh (remove longest prefix = basename)
echo "${path%/*}"               # /usr/local/bin (remove shortest suffix = dirname)
echo "${path%%/*}"              # (remove longest suffix)
echo "${path%.sh}"              # /usr/local/bin/script (remove .sh)

# Substitution
echo "${str/World/Linux}"       # Hello, Linux!  (first occurrence)
echo "${str//l/L}"              # HeLLo, WorLd!  (all occurrences)
echo "${str/#Hello/Hi}"         # Hi, World!     (prefix)
echo "${str/%!/...}"            # Hello, World...  (suffix)

# Check if string contains substring
if [[ "$str" == *"World"* ]]; then
    echo "Found"
fi

# Check if string starts/ends with pattern
[[ "$str" == Hello* ]] && echo "starts with Hello"
[[ "$str" == *"!" ]]   && echo "ends with !"

# String comparison
[[ "abc" < "abd" ]] && echo "abc comes first"
[[ "abc" == "abc" ]] && echo "equal"
```

---

## 4. Arrays & Associative Arrays

```bash
# Indexed arrays
fruits=("apple" "banana" "cherry")
fruits+=("date")                    # Append element
fruits[4]="elderberry"             # Assign at index

echo "${fruits[0]}"                 # apple
echo "${fruits[@]}"                 # all elements
echo "${fruits[*]}"                 # all elements (no quoting protection)
echo "${#fruits[@]}"                # 4 (length)
echo "${!fruits[@]}"                # 0 1 2 3 4 (indices)
echo "${fruits[@]:1:2}"            # banana cherry (slice)

# Unset element
unset fruits[1]

# Iterate
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Read file into array
mapfile -t lines < /etc/passwd
readarray -t lines < /etc/passwd    # same as mapfile

# Read command output into array
mapfile -t processes < <(ps aux | tail -n +2)

# Associative arrays (bash 4+)
declare -A colors
colors["red"]="#FF0000"
colors["green"]="#00FF00"
colors["blue"]="#0000FF"

echo "${colors[red]}"               # #FF0000
echo "${!colors[@]}"                # keys
echo "${colors[@]}"                 # values

for key in "${!colors[@]}"; do
    echo "$key = ${colors[$key]}"
done

# Check if key exists
[[ -v colors["red"] ]] && echo "red exists"
```

---

## 5. Arithmetic

```bash
# Arithmetic expansion
echo $((2 + 3))         # 5
echo $((10 % 3))        # 1
echo $((2 ** 8))        # 256
echo $((i++))           # post-increment
echo $((++i))           # pre-increment

# Assign result
result=$(( (a + b) * c ))

# Arithmetic command (returns exit status 0 if non-zero)
(( count++ ))
(( count > 10 )) && echo "over limit"

# Float arithmetic (use bc or awk)
result=$(echo "scale=2; 10 / 3" | bc)          # 3.33
result=$(awk 'BEGIN {printf "%.2f\n", 10/3}')   # 3.33

# Number base conversions
echo $((16#FF))         # 255  (hex to decimal)
printf "%x\n" 255       # ff   (decimal to hex)
printf "%o\n" 8         # 10   (decimal to octal)
printf "%08b\n" 42      # 00101010  (decimal to binary)
```

---

## 6. Conditionals

```bash
# if / elif / else
if [[ condition ]]; then
    echo "true"
elif [[ other ]]; then
    echo "other"
else
    echo "false"
fi

# One-liner
[[ -f file.txt ]] && echo "exists" || echo "missing"
command && echo "ok" || echo "failed"

# case statement
case "$var" in
    start|begin)
        echo "starting"
        ;;
    stop|end)
        echo "stopping"
        ;;
    restart)
        echo "restarting"
        ;;
    *)
        echo "unknown: $var"
        ;;
esac

# Case with glob patterns
case "$filename" in
    *.tar.gz|*.tgz)  tar xzf "$filename" ;;
    *.zip)           unzip "$filename" ;;
    *.bz2)           bzip2 -d "$filename" ;;
    *)               echo "Unknown format" ;;
esac

# Ternary-style
value=$(( flag ? 1 : 0 ))
```

---

## 7. Test Operators

### File Tests

| Operator | True If |
|----------|---------|
| `-e file` | File exists |
| `-f file` | Regular file |
| `-d file` | Directory |
| `-L file` | Symbolic link |
| `-r file` | Readable |
| `-w file` | Writable |
| `-x file` | Executable |
| `-s file` | Non-empty (size > 0) |
| `-z file` | Empty |
| `f1 -nt f2` | f1 newer than f2 |
| `f1 -ot f2` | f1 older than f2 |

### String Tests

| Operator | True If |
|----------|---------|
| `-z str` | String is empty |
| `-n str` | String is non-empty |
| `s1 == s2` | Strings are equal (`[[ ]]` only) |
| `s1 != s2` | Strings are not equal |
| `s1 =~ regex` | Matches extended regex (`[[ ]]` only) |
| `s1 < s2` | Lexicographically less (`[[ ]]` only) |

### Numeric Tests (use `(( ))` or `-eq` etc.)

| `[ ]` Operator | `(( ))` | Meaning |
|----------------|---------|---------|
| `-eq` | `==` | Equal |
| `-ne` | `!=` | Not equal |
| `-lt` | `<` | Less than |
| `-le` | `<=` | Less than or equal |
| `-gt` | `>` | Greater than |
| `-ge` | `>=` | Greater than or equal |

```bash
# Use [[ ]] for file and string tests
[[ -f /etc/passwd ]] && echo "exists"
[[ "$var" =~ ^[0-9]+$ ]] && echo "is numeric"

# Use (( )) for numeric tests
(( count > 10 )) && echo "over limit"
(( i % 2 == 0 )) && echo "even"
```

---

## 8. Loops

```bash
# for loop (C-style)
for (( i=0; i<10; i++ )); do
    echo "$i"
done

# for loop (in list)
for item in one two three; do
    echo "$item"
done

# for loop (glob)
for file in /var/log/*.log; do
    [[ -f "$file" ]] || continue
    echo "Processing: $file"
done

# for loop (command output)
for user in $(cut -d: -f1 /etc/passwd); do
    echo "$user"
done

# while loop
count=0
while (( count < 5 )); do
    echo "count=$count"
    (( count++ ))
done

# while read loop (preferred for line-by-line reading)
while IFS=: read -r username _ uid gid gecos home shell; do
    echo "$username (UID: $uid)"
done < /etc/passwd

# while read from command
while IFS= read -r line; do
    echo "$line"
done < <(grep "ERROR" /var/log/app.log)

# until loop (opposite of while)
until ping -c1 server.example.com &>/dev/null; do
    echo "Waiting for server..."
    sleep 5
done

# Loop control
for i in {1..10}; do
    (( i == 3 )) && continue    # Skip 3
    (( i == 7 )) && break       # Stop at 7
    echo "$i"
done

# Infinite loop
while true; do
    # do something
    sleep 60
done
```

---

## 9. Functions

```bash
# Function definition
greet() {
    local name="${1:-World}"    # Use first arg or "World"
    local -r greeting="Hello"  # local read-only
    echo "$greeting, $name!"
}

# Call function
greet "Alice"
greet              # Uses default

# Return values
# Functions return exit status (0–255), not data
# Use echo / stdout to return data

get_timestamp() {
    date '+%Y-%m-%d %H:%M:%S'
}

timestamp=$(get_timestamp)    # Capture stdout

# Return exit status
is_even() {
    (( $1 % 2 == 0 ))
}

is_even 4 && echo "even" || echo "odd"

# Recursive function
factorial() {
    local n=$1
    (( n <= 1 )) && echo 1 && return
    echo $(( n * $(factorial $((n - 1))) ))
}

# Functions with local arrays
process_items() {
    local -a items=("$@")
    local result=()
    for item in "${items[@]}"; do
        result+=("${item^^}")
    done
    printf '%s\n' "${result[@]}"
}

# Function documentation pattern
# Usage: backup_dir <source> <destination> [--verbose]
backup_dir() {
    local source="$1"
    local dest="$2"
    local verbose="${3:-}"

    [[ -d "$source" ]] || { echo "Error: $source not a directory" >&2; return 1; }

    local timestamp
    timestamp=$(date +%Y%m%d_%H%M%S)
    local backup="${dest}/backup_${timestamp}.tar.gz"

    tar czf "$backup" "$source" ${verbose:+-v}
    echo "$backup"
}

# Library pattern: source only if script, not sourced
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

---

## 10. Input & Output

```bash
# Read user input
read -p "Enter name: " name
read -sp "Enter password: " password; echo   # -s = silent

# Read with timeout
read -t 10 -p "Continue? [y/N] " answer || answer="n"

# Read multiple values
read -r first last <<< "John Doe"

# Select menu
select choice in "Start" "Stop" "Restart" "Quit"; do
    case "$choice" in
        Start)   start_service ;;
        Stop)    stop_service ;;
        Restart) restart_service ;;
        Quit)    break ;;
        *)       echo "Invalid: $REPLY" ;;
    esac
done

# Logging function
log() {
    local level="${1:-INFO}"
    local message="$2"
    printf '[%s] [%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$level" "$message" | tee -a "$LOG_FILE"
}

log "INFO" "Script started"
log "ERROR" "Something went wrong"

# Colored output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'  # No Color

echo -e "${GREEN}Success${NC}"
echo -e "${RED}Error${NC}"
echo -e "${YELLOW}Warning${NC}"

# Redirect stdout and stderr
exec 1> >(tee -a "$LOG_FILE")    # Redirect all stdout to log
exec 2>&1                         # Merge stderr into stdout

# Here-string
grep "pattern" <<< "$variable"

# Process substitution
diff <(command1) <(command2)
```

---

## 11. Error Handling

```bash
#!/usr/bin/env bash
set -euo pipefail

# Trap on exit
cleanup() {
    local exit_code=$?
    echo "Cleaning up... (exit code: $exit_code)" >&2
    rm -f /tmp/script_lock_$$
    exit $exit_code
}
trap cleanup EXIT

# Trap on error (with ERR)
on_error() {
    local line_num="$1"
    echo "ERROR: Script failed at line $line_num" >&2
    # Optionally send alert
}
trap 'on_error $LINENO' ERR

# Trap on signals
trap 'echo "Received SIGINT"; exit 1' INT TERM

# Error handling function
die() {
    echo "FATAL: $*" >&2
    exit 1
}

# Retry logic
retry() {
    local max_attempts="${1:-3}"
    local delay="${2:-5}"
    local cmd="${@:3}"
    local attempt=1

    while (( attempt <= max_attempts )); do
        if eval "$cmd"; then
            return 0
        fi
        echo "Attempt $attempt/$max_attempts failed, retrying in ${delay}s..." >&2
        sleep "$delay"
        (( attempt++ ))
    done
    echo "All $max_attempts attempts failed" >&2
    return 1
}

# Usage: retry 3 5 curl -f https://api.example.com/health

# Check prerequisites
require_command() {
    command -v "$1" &>/dev/null || die "Required command not found: $1"
}

require_command curl
require_command jq
require_command aws

# Lock file to prevent concurrent execution
LOCKFILE="/tmp/${SCRIPT_NAME}.lock"
exec 200>"$LOCKFILE"
flock -n 200 || die "Another instance is already running"

# Validate input
validate_ip() {
    local ip="$1"
    [[ "$ip" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1
    IFS='.' read -ra octets <<< "$ip"
    for octet in "${octets[@]}"; do
        (( octet >= 0 && octet <= 255 )) || return 1
    done
}

# Check exit codes
if ! command_that_might_fail; then
    echo "Command failed, handling..." >&2
    # handle error
fi

# Ignore errors for specific command
rm -f file_that_may_not_exist || true
```

---

## 12. Process & Job Control

```bash
# Background job
long_running_task &
pid=$!

# Wait for background job
wait $pid
exit_code=$?

# Parallel execution with wait
for server in server1 server2 server3; do
    ssh "$server" "command" &
done
wait   # Wait for all background jobs

# Run with timeout
timeout 30s command_that_might_hang

# Signal sending
kill -TERM $pid     # Graceful shutdown
kill -KILL $pid     # Force kill
kill -HUP $pid      # Reload config (for daemons)
kill -USR1 $pid     # User-defined signal

# Command substitution
result=$(command)
result=$(command 2>&1)        # Include stderr
result=$(command 2>/dev/null) # Discard stderr

# Subshell
(
    cd /tmp
    ls
)   # Current directory unchanged

# Co-processes (bash 4+)
coproc MY_PROC { tail -f /var/log/app.log; }
read -r line <&"${MY_PROC[0]}"
```

---

## 13. Pattern Matching & Parameter Expansion

```bash
# Glob patterns (pathname expansion)
*       # Any string (not starting with .)
?       # Single character
[abc]   # One of a, b, c
[a-z]   # Range
[^abc]  # Not a, b, c

# Extended globs (shopt -s extglob)
shopt -s extglob
?(pattern)   # 0 or 1 occurrences
*(pattern)   # 0 or more
+(pattern)   # 1 or more
@(pattern)   # Exactly 1
!(pattern)   # Anything except

# Examples
ls +(*.txt|*.md)        # Only .txt or .md files
rm !(*.log)             # Delete all except .log files

# nullglob: glob expands to nothing if no match
shopt -s nullglob
for f in *.nonexistent; do
    echo "$f"   # Never executes
done

# Case-insensitive glob
shopt -s nocaseglob

# Parameter expansion summary
${var}           # Value of var
${var:-default}  # Default if unset/empty
${var:=default}  # Assign default if unset/empty
${var:?message}  # Error if unset/empty
${var:+alt}      # Alt if set and non-empty
${var:offset:len} # Substring
${#var}          # Length
${var#pattern}   # Remove shortest prefix match
${var##pattern}  # Remove longest prefix match
${var%pattern}   # Remove shortest suffix match
${var%%pattern}  # Remove longest suffix match
${var/pat/repl}  # Replace first match
${var//pat/repl} # Replace all matches
${var/#pat/repl} # Replace prefix match
${var/%pat/repl} # Replace suffix match
${var,,}         # Lowercase all
${var^^}         # Uppercase all
${var,}          # Lowercase first character
${var^}          # Uppercase first character
```

---

## 14. Script Templates

### Basic Script Template

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <argument>

Options:
  -h, --help       Show this help
  -v, --verbose    Verbose output
  -d, --dry-run    Dry run (no changes)
  -o, --output     Output directory (default: /tmp)

Examples:
  $SCRIPT_NAME -v /path/to/source
  $SCRIPT_NAME --dry-run --output /backup /path/to/source
EOF
}

log()  { printf '[%s] INFO  %s\n'  "$(date '+%H:%M:%S')" "$*"; }
warn() { printf '[%s] WARN  %s\n'  "$(date '+%H:%M:%S')" "$*" >&2; }
die()  { printf '[%s] ERROR %s\n'  "$(date '+%H:%M:%S')" "$*" >&2; exit 1; }

VERBOSE=false
DRY_RUN=false
OUTPUT="/tmp"

parse_args() {
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -h|--help)      usage; exit 0 ;;
            -v|--verbose)   VERBOSE=true; shift ;;
            -d|--dry-run)   DRY_RUN=true; shift ;;
            -o|--output)    OUTPUT="$2"; shift 2 ;;
            --)             shift; break ;;
            -*)             die "Unknown option: $1" ;;
            *)              break ;;
        esac
    done
    ARGS=("$@")
}

main() {
    parse_args "$@"
    [[ ${#ARGS[@]} -gt 0 ]] || die "No arguments provided. Use -h for help."
    log "Starting $SCRIPT_NAME"
    # ... main logic here ...
    log "Done"
}

main "$@"
```

---

*Tip: Use `shellcheck` to statically analyse your scripts for common errors and style issues.*
