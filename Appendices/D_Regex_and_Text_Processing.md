# Appendix D – Regex and Text Processing

A practical reference for regular expressions and the core Linux text-processing tools:
`grep`, `sed`, `awk`, `cut`, `tr`, `sort`, `uniq`, `column`, and composing pipelines.

---

## Table of Contents
1. [Regular Expression Syntax](#1-regular-expression-syntax)
2. [grep – Pattern Searching](#2-grep--pattern-searching)
3. [sed – Stream Editor](#3-sed--stream-editor)
4. [awk – Text Processing Language](#4-awk--text-processing-language)
5. [cut – Extract Fields](#5-cut--extract-fields)
6. [tr – Translate Characters](#6-tr--translate-characters)
7. [sort – Sort Lines](#7-sort--sort-lines)
8. [uniq – Remove Duplicates](#8-uniq--remove-duplicates)
9. [Practical One-Liners](#9-practical-one-liners)

---

## 1. Regular Expression Syntax

### Basic vs Extended vs Perl-Compatible

| Type | Flag | Description |
|------|------|-------------|
| BRE (Basic) | `grep` (default) | `\+`, `\?`, `\|` must be backslash-escaped |
| ERE (Extended) | `grep -E` or `egrep` | `+`, `?`, `|` are metacharacters directly |
| PCRE | `grep -P` | Full Perl-compatible (`\d`, `\w`, lookaheads, etc.) |

### Anchors

| Pattern | Matches |
|---------|---------|
| `^` | Start of line |
| `$` | End of line |
| `\b` | Word boundary (PCRE/ERE) |
| `\B` | Non-word boundary |
| `\A` | Start of string (PCRE) |
| `\Z` | End of string (PCRE) |

### Character Classes

| Pattern | Matches |
|---------|---------|
| `.` | Any single character (except newline) |
| `[abc]` | Any one of `a`, `b`, or `c` |
| `[^abc]` | Any character NOT `a`, `b`, or `c` |
| `[a-z]` | Any lowercase letter |
| `[A-Za-z0-9]` | Alphanumeric |
| `[[:alpha:]]` | POSIX letter class |
| `[[:digit:]]` | POSIX digit class |
| `[[:alnum:]]` | POSIX alphanumeric |
| `[[:space:]]` | POSIX whitespace |
| `[[:upper:]]` | POSIX uppercase |
| `[[:lower:]]` | POSIX lowercase |
| `\d` | Digit `[0-9]` (PCRE) |
| `\D` | Non-digit (PCRE) |
| `\w` | Word character `[A-Za-z0-9_]` (PCRE) |
| `\W` | Non-word character (PCRE) |
| `\s` | Whitespace (PCRE) |
| `\S` | Non-whitespace (PCRE) |

### Quantifiers

| Pattern | Matches |
|---------|---------|
| `*` | 0 or more of preceding |
| `+` | 1 or more of preceding |
| `?` | 0 or 1 of preceding |
| `{n}` | Exactly `n` occurrences |
| `{n,}` | `n` or more |
| `{n,m}` | Between `n` and `m` |
| `*?` `+?` `??` | Non-greedy (PCRE) |

### Groups and Alternation

| Pattern | Matches |
|---------|---------|
| `(abc)` | Capture group |
| `(?:abc)` | Non-capturing group (PCRE) |
| `a\|b` | Either `a` or `b` (BRE) |
| `a|b` | Either `a` or `b` (ERE/PCRE) |
| `\1` | Back-reference to group 1 |
| `(?=abc)` | Positive lookahead (PCRE) |
| `(?!abc)` | Negative lookahead (PCRE) |
| `(?<=abc)` | Positive lookbehind (PCRE) |

### Common Regex Patterns

```bash
# IPv4 address
\b([0-9]{1,3}\.){3}[0-9]{1,3}\b

# Email address (simplified)
[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}

# URL
https?://[^\s/$.?#].[^\s]*

# Date YYYY-MM-DD
[0-9]{4}-[0-1][0-9]-[0-3][0-9]

# MAC address
([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}

# Port number (1-65535)
([1-9][0-9]{0,3}|[1-5][0-9]{4}|6[0-4][0-9]{3}|65[0-4][0-9]{2}|655[0-2][0-9]|6553[0-5])

# SHA-256 hash
[0-9a-f]{64}
```

---

## 2. grep – Pattern Searching

### Flags Reference

| Flag | Description |
|------|-------------|
| `-E` | Extended regex (ERE) |
| `-P` | Perl-compatible regex (PCRE) |
| `-F` | Fixed string, no regex |
| `-i` | Case-insensitive |
| `-v` | Invert match (non-matching lines) |
| `-n` | Show line numbers |
| `-c` | Count matching lines |
| `-l` | Print only filenames with matches |
| `-L` | Print only filenames without matches |
| `-r` / `-R` | Recursive search; `-R` follows symlinks |
| `-A n` | Print `n` lines after match |
| `-B n` | Print `n` lines before match |
| `-C n` | Print `n` lines of context (before + after) |
| `-o` | Print only matched part |
| `-w` | Match whole words only |
| `-x` | Match whole line only |
| `-m n` | Stop after `n` matches |
| `--include=GLOB` | Search only matching filenames |
| `--exclude=GLOB` | Exclude matching filenames |
| `-H` | Print filename with match |
| `-h` | Suppress filename |
| `-q` | Quiet; exit 0 if match found |

```bash
# Recursive search for "password" in /etc, case-insensitive
grep -rni "password" /etc 2>/dev/null

# Find lines with an IPv4 address using PCRE
grep -oP '\b(?:\d{1,3}\.){3}\d{1,3}\b' access.log | sort -u

# Show 3 lines of context around error messages
grep -C 3 "ERROR" /var/log/app.log

# Search only .conf and .ini files
grep -r --include="*.conf" --include="*.ini" "PermitRoot" /etc/

# Grep for lines NOT matching a pattern
grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"

# Count occurrences per file
grep -rc "Exception" /var/log/ 2>/dev/null | grep -v ":0"

# Print filename and line number, context
grep -rn -B1 -A2 "panic" /var/log/kern.log
```

---

## 3. sed – Stream Editor

### Syntax

```
sed [OPTIONS] 'SCRIPT' [FILE...]
sed [OPTIONS] -e 'SCRIPT1' -e 'SCRIPT2' [FILE...]
sed [OPTIONS] -f SCRIPT_FILE [FILE...]
```

### Common Commands

| Command | Description | Example |
|---------|-------------|---------|
| `s/pattern/replace/flags` | Substitute | `sed 's/foo/bar/g'` |
| `d` | Delete line | `sed '/^#/d'` |
| `p` | Print line | `sed -n '5,10p'` |
| `i text` | Insert before line | `sed '1i # Header'` |
| `a text` | Append after line | `sed '$a # Footer'` |
| `c text` | Change (replace) line | `sed '/^old/c new line'` |
| `y/src/dst/` | Transliterate | `sed 'y/abc/ABC/'` |
| `q` | Quit after line | `sed '5q'` |
| `n` | Next line | `sed 'n; d'` (delete even lines) |
| `=` | Print line number | `sed -n '/error/='` |
| `r file` | Read and append file | `sed '/marker/r insert.txt'` |
| `w file` | Write to file | `sed -n '/error/w errors.txt'` |

### Address Ranges

```bash
# Line number
sed '5d'                  # Delete line 5
sed '5,10d'               # Delete lines 5–10
sed '5,/end/d'            # Delete from line 5 to "end"

# Regex address
sed '/^#/d'               # Delete comment lines
sed '/start/,/end/d'      # Delete between markers (inclusive)
sed '/start/,/end/ { /start/!{/end/!d} }' # Between markers (exclusive)

# Step addresses (sed >= 4.3)
sed '0~2d'                # Delete every even line
sed '1~2d'                # Delete every odd line
```

### Substitution Flags

| Flag | Description |
|------|-------------|
| `g` | Replace all occurrences per line |
| `i` | Case-insensitive match |
| `n` | Replace only nth occurrence |
| `p` | Print if substitution made (use with `-n`) |
| `w file` | Write modified lines to file |

### In-place Editing

```bash
# Edit in-place (GNU sed)
sed -i 's/old/new/g' file.txt

# Edit in-place with backup
sed -i.bak 's/old/new/g' file.txt

# Edit multiple files
sed -i 's/192\.168\.1\.1/10.0.0.1/g' /etc/nginx/sites-available/*.conf
```

### Practical sed Examples

```bash
# Remove blank lines
sed '/^[[:space:]]*$/d' file.txt

# Remove comments and blank lines
sed '/^[[:space:]]*#/d; /^[[:space:]]*$/d' /etc/ssh/sshd_config

# Add line number prefix
sed = file.txt | sed 'N;s/\n/\t/'

# Uncomment a specific line
sed -i '/^#.*PasswordAuthentication/s/^#//' /etc/ssh/sshd_config

# Extract text between two markers
sed -n '/BEGIN/,/END/p' file.txt

# Double-space a file
sed G file.txt

# Strip trailing whitespace
sed -i 's/[[:space:]]*$//' file.txt

# Replace nth occurrence on each line
sed 's/foo/bar/2' file.txt   # Replace 2nd occurrence

# Multi-line substitution
sed ':a;N;$!ba;s/\n/ /g' file.txt   # Join all lines
```

---

## 4. awk – Text Processing Language

### Syntax

```
awk [OPTIONS] 'PROGRAM' [FILE...]
awk [OPTIONS] -f program.awk [FILE...]
```

### Structure

```awk
BEGIN { initialization }
/pattern/ { action }             # Runs on matching lines
{ action }                       # Runs on every line
END { finalization }
```

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `$0` | Entire current record (line) |
| `$1`, `$2`, ... | Fields 1, 2, ... |
| `$NF` | Last field |
| `$(NF-1)` | Second to last field |
| `NF` | Number of fields |
| `NR` | Current record number (across all files) |
| `FNR` | Record number in current file |
| `FS` | Input field separator (default: whitespace) |
| `OFS` | Output field separator (default: space) |
| `RS` | Input record separator (default: newline) |
| `ORS` | Output record separator (default: newline) |
| `FILENAME` | Current input file name |

### Operators

```awk
# Comparison: == != < > <= >=
# Regex: ~ (match)  !~ (no match)
# Logical: && || !
# Arithmetic: + - * / % ^
# String concatenation: space or ""
```

### awk Examples

```bash
# Print specific fields from /etc/passwd
awk -F: '{print $1, $3}' /etc/passwd

# Print lines where 3rd field > 1000 (regular users)
awk -F: '$3 >= 1000 {print $1, $3, $6}' /etc/passwd

# Sum a column
awk '{sum += $5} END {print "Total:", sum}' data.txt

# Print lines matching a pattern
awk '/error|warn/ {print NR": "$0}' /var/log/app.log

# Custom field separator output
awk -F: 'OFS="\t" {print $1,$3,$6}' /etc/passwd

# Count occurrences of each value in column 1
awk '{count[$1]++} END {for (k in count) print count[k], k}' log.txt | sort -rn

# Print lines between markers (inclusive)
awk '/START/,/END/' file.txt

# Remove duplicate lines (preserving order)
awk '!seen[$0]++' file.txt

# Print only unique lines in field 1
awk -F, '!seen[$1]++ {print}' data.csv

# Calculate average
awk '{sum+=$1; count++} END {printf "Avg: %.2f\n", sum/count}' numbers.txt

# Print lines with more than 5 fields
awk 'NF > 5' file.txt

# Format output with printf
awk -F: '{printf "%-20s UID=%-6d Home=%s\n", $1, $3, $6}' /etc/passwd

# Multi-file: print filename with each line
awk '{print FILENAME": "$0}' *.log

# Inline variable passing
awk -v threshold=100 '$1 > threshold {print}' data.txt
```

---

## 5. cut – Extract Fields

```bash
# Syntax
cut OPTION [FILE...]

# By delimiter (field)
cut -d: -f1 /etc/passwd           # First field
cut -d: -f1,3 /etc/passwd         # Fields 1 and 3
cut -d: -f1-3 /etc/passwd         # Fields 1 through 3
cut -d, -f2- data.csv             # Field 2 to end

# By character position
cut -c1-10 file.txt               # Characters 1–10
cut -c1,5,10 file.txt             # Characters 1, 5, 10

# By bytes
cut -b1-4 file.bin

# Complement (all except)
cut -d: --complement -f3 /etc/passwd

# Practical examples
# Extract usernames from /etc/passwd
cut -d: -f1 /etc/passwd

# Extract IP from access log (first field)
cut -d' ' -f1 /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Get path components
echo "/usr/local/bin" | cut -d/ -f2-   # local/bin
```

---

## 6. tr – Translate Characters

```bash
# Syntax
tr [OPTIONS] SET1 [SET2]

# Translate (replace)
echo "Hello World" | tr 'A-Z' 'a-z'        # lowercase
echo "hello world" | tr 'a-z' 'A-Z'        # uppercase
echo "hello" | tr 'aeiou' '*'              # replace vowels

# Delete (-d)
echo "h-e-l-l-o" | tr -d '-'              # hello
echo "abc123" | tr -d '[:digit:]'         # abc
cat file.txt | tr -d '\r'                 # Remove Windows CR

# Squeeze (-s): collapse repeated chars
echo "hello   world" | tr -s ' '          # single space
echo "aaabbbccc" | tr -s 'a-z'           # abc

# Complement (-c): operate on complement
echo "abc123" | tr -cd '[:alnum:]'        # keep only alphanumeric
echo "hello\nworld" | tr -cd '[:print:]\n'  # keep printable

# Useful character classes
# [:alpha:] [:digit:] [:alnum:] [:space:] [:upper:] [:lower:]
# [:punct:] [:print:] [:blank:] [:cntrl:]

# Remove non-printable characters
cat binary_file | tr -cd '[:print:]\n'

# Replace newlines with spaces (join lines)
tr '\n' ' ' < file.txt

# Replace spaces with newlines (one word per line)
tr -s ' ' '\n' < file.txt
```

---

## 7. sort – Sort Lines

```bash
# Basic sort (alphabetical)
sort file.txt

# Reverse sort
sort -r file.txt

# Numeric sort
sort -n numbers.txt

# Human-readable number sort (1K, 2M, 3G)
sort -h sizes.txt

# Sort by specific field (column 2, colon delimiter)
sort -t: -k2 /etc/passwd

# Numeric sort by field 3
sort -t: -k3 -n /etc/passwd

# Multiple sort keys
sort -t, -k2,2n -k1,1 data.csv   # Sort by col2 numerically, then col1

# Sort and remove duplicates
sort -u file.txt

# Sort by month name
sort -M months.txt    # Jan Feb ... Dec

# Stable sort (preserve order of equal elements)
sort -s -k1,1 file.txt

# Random shuffle
sort -R file.txt

# Check if sorted (exit 0 = sorted)
sort -c file.txt

# Sort version numbers
sort -V versions.txt    # 1.2 1.10 2.0 (not 1.10 1.2 2.0)

# Sort by file size (with du)
du -sh /var/log/* | sort -rh | head -10
```

---

## 8. uniq – Remove Duplicates

> `uniq` requires **sorted** input (or use `awk '!seen[$0]++'` for unsorted).

```bash
# Remove consecutive duplicate lines
sort file.txt | uniq

# Count occurrences (-c)
sort file.txt | uniq -c | sort -rn

# Show only duplicated lines (-d)
sort file.txt | uniq -d

# Show only unique (non-repeated) lines (-u)
sort file.txt | uniq -u

# Case-insensitive comparison (-i)
sort file.txt | uniq -i

# Compare only first N characters (-w N)
sort file.txt | uniq -w 5

# Skip first N fields (-f N)
sort file.txt | uniq -f 2

# Top 10 most common values
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

---

## 9. Practical One-Liners

### Log Analysis

```bash
# Top 10 IP addresses in Nginx access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Count HTTP status codes
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Find requests taking > 5 seconds (field: $NF = response time)
awk '$NF > 5 {print $0}' /var/log/nginx/access.log

# Failed SSH login attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Extract all email addresses from files
grep -oP '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' *.txt | sort -u

# Extract all IPv4 addresses from a file
grep -oP '\b(?:\d{1,3}\.){3}\d{1,3}\b' file.txt | sort -u
```

### File Processing

```bash
# Remove blank lines and comment lines
grep -v '^[[:space:]]*$' file.txt | grep -v '^[[:space:]]*#'

# Convert CSV to TSV
sed 's/,/\t/g' data.csv

# Remove duplicate lines without sorting
awk '!seen[$0]++' file.txt

# Print lines between line 50 and 100
awk 'NR>=50 && NR<=100' file.txt
sed -n '50,100p' file.txt

# Count words in a directory of text files
cat *.txt | tr -s '[:space:]' '\n' | sort | uniq -c | sort -rn | head -20

# Find lines present in file1 but not file2
comm -23 <(sort file1.txt) <(sort file2.txt)

# Number the non-blank lines
grep -n "." file.txt

# Reverse field order (3 fields)
awk -F, '{print $3,$2,$1}' OFS=, data.csv
```

### System Administration

```bash
# List all listening ports with process names
ss -tulnp | awk 'NR>1 {print $1, $5, $7}' | column -t

# Find top memory-consuming processes
ps aux --sort=-%mem | awk 'NR<=11 {printf "%-10s %-6s %s\n", $1, $4, $11}'

# Extract failed units from systemctl
systemctl --failed | grep '●' | awk '{print $2}'

# Check which ports are open on a remote host (bash built-in)
for port in 22 80 443 8080; do
  (echo >/dev/tcp/host/$port) 2>/dev/null && echo "$port open" || echo "$port closed"
done

# Parse /etc/passwd into a readable table
awk -F: '{printf "%-15s %-6d %-30s %s\n", $1, $3, $5, $6}' /etc/passwd | column -t

# Find files containing a pattern and display with context
grep -rl "TODO" src/ | xargs -I{} grep -n --color "TODO" {}

# Monitor a log file and alert on errors
tail -F /var/log/app.log | grep --line-buffered "ERROR" | while read line; do
    echo "ALERT: $line" | mail -s "App Error" admin@example.com
done
```

---

*Tip: Combine tools in pipelines — `grep` to filter, `awk` to extract, `sort` to order, `uniq -c` to count.*
