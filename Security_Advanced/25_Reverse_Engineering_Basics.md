# Lesson 25: Reverse Engineering Basics

> **Security Advanced · Lesson 25** | Difficulty: ★★★★★ Expert | Time: ~160 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Describe the ELF binary format and navigate its sections
- Use GDB for interactive debugging and reverse engineering
- Trace system calls with `strace` and library calls with `ltrace`
- Disassemble and analyse binaries with `objdump` and `readelf`
- Perform basic static and dynamic analysis in Radare2
- Identify common binary protections (ASLR, NX, canaries, PIE)

---

## Prerequisites

- Lesson 24 (Web Application Security) or equivalent
- Familiarity with C programming and assembly language basics
- Comfort with hex notation and memory addressing concepts
- GDB, binutils, strace, ltrace, and Radare2 installed

---

## Key Concepts

### 1. The ELF Binary Format

ELF (Executable and Linkable Format) is the standard binary format for Linux executables, shared libraries, core dumps, and object files.

```
ELF Header (fixed 64 bytes for 64-bit)
├── Magic number: \x7fELF
├── Class: 32-bit or 64-bit
├── Data: endianness
├── Type: ET_EXEC, ET_DYN, ET_CORE, ET_REL
├── Machine: x86-64 (0x3E), ARM (0x28), etc.
├── Entry point address
├── Program header table offset
└── Section header table offset

Program Headers (runtime view – used by OS loader)
├── PHDR  – program header table itself
├── INTERP – path to dynamic linker (/lib64/ld-linux-x86-64.so.2)
├── LOAD  – segments mapped into memory
├── DYNAMIC – dynamic linking information
├── NOTE  – OS/build notes
└── GNU_STACK – stack permissions

Section Headers (compile-time view – used by linker/debugger)
├── .text   – executable machine code
├── .rodata – read-only data (string literals, constants)
├── .data   – initialised read-write data (global variables)
├── .bss    – uninitialised data (zeroed at load time)
├── .plt    – Procedure Linkage Table (lazy binding for shared libs)
├── .got    – Global Offset Table (resolved addresses)
├── .symtab – symbol table (stripped in release builds)
├── .strtab – string table
└── .debug_* – DWARF debug information
```

```bash
# Install required tools
sudo apt install -y binutils gdb strace ltrace file readelf radare2

# Identify a binary
file /bin/ls

# Inspect ELF header
readelf -h /bin/ls

# List all sections
readelf -S /bin/ls

# List program headers (segments)
readelf -l /bin/ls

# List dynamic dependencies (like ldd)
readelf -d /bin/ls | grep NEEDED
ldd /bin/ls

# List symbols (works even on stripped binaries for dynamic symbols)
readelf -s /bin/ls
nm /bin/ls 2>/dev/null || echo "Binary is stripped"
nm -D /bin/ls  # Dynamic symbols only

# Show strings embedded in the binary
strings /bin/ls | head -30
strings -n 8 suspicious_binary | grep -iE "http|password|key|secret|/etc/"
```

### 2. Binary Protections

Before reversing, identify what protections are in place:

```bash
# Check all protections at once with checksec
# Install: sudo apt install checksec  OR  pip3 install checksec.py
checksec --file=/path/to/binary

# Manual checks:

# 1. ASLR (system-wide)
cat /proc/sys/kernel/randomize_va_space
# 0=disabled, 1=partial, 2=full (default)

# 2. NX bit (no execute on stack/heap)
readelf -l binary | grep GNU_STACK
# RWE = no NX; RW = NX enabled

# 3. Stack canary
strings binary | grep "__stack_chk_fail" | head -1
# or check PLT/GOT entries

# 4. PIE (Position Independent Executable)
file binary | grep "shared object"   # PIE shows as 'shared object'
readelf -h binary | grep "Type:"     # ET_DYN = PIE, ET_EXEC = no PIE

# 5. RELRO (Relocation Read-Only)
readelf -l binary | grep GNU_RELRO
readelf -d binary | grep BIND_NOW   # FULL RELRO requires BIND_NOW
```

### 3. objdump – Disassembly and Analysis

```bash
# Disassemble the entire binary (AT&T syntax by default)
objdump -d /bin/ls | head -80

# Intel syntax (more readable for x86)
objdump -d -M intel /bin/ls | head -80

# Disassemble a specific section
objdump -d -j .text binary | head -100

# Show source with disassembly (requires debug symbols)
objdump -d -S binary_with_debug | head -100

# Display the relocation table
objdump -r binary

# Display the PLT (shows which library functions are called)
objdump -d -j .plt binary

# Display the GOT
objdump -d -j .got binary

# Examine data sections
objdump -s -j .rodata binary | head -30
objdump -s -j .data binary | head -30

# Disassemble a specific function (use readelf -s to find address first)
ADDR=$(nm binary | grep " main$" | awk '{print $1}')
objdump -d binary | awk "/^$ADDR/,/^[0-9a-f]+ </"
```

### 4. GDB – GNU Debugger

GDB is the primary debugger for Linux binaries.

```bash
# Basic GDB session
gdb ./target_binary

# GDB with pretty-printing (install gef, pwndbg, or peda for better UX)
# Install GEF (GDB Enhanced Features)
bash -c "$(curl -4 fsSL https://gef.blah.cat/sh)" 2>/dev/null || \
  wget -q -O ~/.gdbinit-gef.py https://gef.blah.cat/py && \
  echo "source ~/.gdbinit-gef.py" >> ~/.gdbinit
```

```
# Essential GDB commands (inside gdb)

# Run and control execution
(gdb) run [args]           # Start the program
(gdb) run < input.txt      # Run with stdin redirected
(gdb) continue             # Continue after breakpoint (c)
(gdb) step                 # Step into function call (s)
(gdb) next                 # Step over function call (n)
(gdb) finish               # Run until current function returns
(gdb) quit                 # Exit GDB (q)

# Breakpoints
(gdb) break main           # Break at function name
(gdb) break *0x401234      # Break at address
(gdb) break file.c:42      # Break at source line
(gdb) info breakpoints     # List breakpoints
(gdb) delete 2             # Delete breakpoint #2
(gdb) condition 1 i==5     # Conditional breakpoint

# Watchpoints (break when memory changes)
(gdb) watch *0x601020      # Watch a specific address
(gdb) watch variable_name  # Watch a variable

# Examine memory and registers
(gdb) info registers       # All registers (i r)
(gdb) print $rsp           # Print stack pointer
(gdb) print $rip           # Print instruction pointer
(gdb) x/20xb $rsp          # Examine 20 bytes at RSP (hex bytes)
(gdb) x/10xg $rsp          # Examine 10 qwords (64-bit) at RSP
(gdb) x/5i $rip            # Examine 5 instructions at RIP
(gdb) x/s 0x400abc         # Examine as string

# Disassembly
(gdb) disassemble main      # Disassemble main function
(gdb) disassemble /m main   # Include source (needs debug info)
(gdb) layout asm            # TUI mode with assembly view
(gdb) layout regs           # TUI mode with registers

# Backtrace
(gdb) backtrace             # Show call stack (bt)
(gdb) frame 2               # Switch to stack frame 2

# Variables and expressions
(gdb) print variable        # Print variable value
(gdb) set variable = 42     # Modify a variable
(gdb) display $rax          # Auto-print register after each step
```

**GDB scripting example**:

```python
# gdb_script.py – automate analysis
import gdb

class BreakOnStrlen(gdb.Breakpoint):
    def stop(self):
        arg = gdb.parse_and_eval("(char*)$rdi")
        print(f"strlen called with: {arg.string()}")
        return False  # Don't stop; just log

gdb.execute("file ./target")
BreakOnStrlen("strlen")
gdb.execute("run")
```

```bash
# Run GDB with a script
gdb -x gdb_script.py ./target
gdb -batch -x gdb_commands.txt ./target
```

### 5. strace – System Call Tracer

`strace` intercepts and logs system calls made by a process.

```bash
# Trace all system calls
strace ./target 2>&1 | head -50

# Filter to specific system calls
strace -e trace=open,read,write,execve ./target

# Trace network calls
strace -e trace=network ./target

# Follow child processes
strace -f ./target

# Attach to a running process
sudo strace -p $(pgrep suspicious_process)

# Show timestamps
strace -t ./target          # Absolute time
strace -r ./target          # Relative time (time since previous syscall)
strace -T ./target          # Time spent in each syscall

# Decode strings (show up to 256 chars)
strace -s 256 ./target

# Count syscalls and show statistics
strace -c ./target

# Save output to file
strace -o trace.log ./target

# Common patterns to look for:
strace ./malware 2>&1 | grep -E "open\(|execve\(|connect\(|sendto\("
strace ./malware 2>&1 | grep -E "openat.*O_RDWR|write.*passwd"
```

### 6. ltrace – Library Call Tracer

`ltrace` traces calls to shared library functions (e.g., `strcmp`, `malloc`, `printf`).

```bash
# Trace all library calls
ltrace ./target 2>&1 | head -50

# Show both library calls and system calls
ltrace -S ./target 2>&1 | head -100

# Filter to specific library functions
ltrace -e strcmp+strcpy+malloc ./target

# Follow forks
ltrace -f ./target

# Count library calls
ltrace -c ./target

# Common password-checking pattern
ltrace ./crackme 2>&1 | grep -E "strcmp|strncmp|memcmp"
# Often reveals the expected password in plaintext!

# Attach to running process
sudo ltrace -p <PID>
```

### 7. Radare2

Radare2 (r2) is a comprehensive, scriptable reverse engineering framework.

```bash
# Open a binary for analysis
r2 ./target
r2 -A ./target     # Open and auto-analyse (slow on large binaries)
r2 -d ./target     # Open in debug mode

# Analyse without opening interactively
r2 -qc 'aaa; afl' ./target        # Analyse and list functions
r2 -qc 'aaa; pdf @main' ./target  # Print disassembly of main
```

```
# Radare2 command reference (inside r2)

# Analysis
(r2) aa          # Analyse all (basic)
(r2) aaa         # Analyse all (deeper, slower)
(r2) afl         # List all functions
(r2) afn name addr  # Rename a function

# Navigation
(r2) s main      # Seek to main function
(r2) s 0x401000  # Seek to address
(r2) s sym.imp.puts  # Seek to PLT entry for puts

# Disassembly
(r2) pdf         # Print disassembly of current function
(r2) pd 20       # Print 20 instructions at current location
(r2) pds         # Print strings referenced in current function

# Strings and data
(r2) iz          # List strings in data section
(r2) izz         # List all strings (including in code section)
(r2) is          # List symbols

# Cross-references
(r2) axt 0x401234  # Find code that references address
(r2) axf           # List cross-references from current function

# Visual mode
(r2) V           # Enter visual mode
(r2) VV          # Visual graph mode (control flow graph)
(r2) v           # Visual panels mode

# Debugging (with r2 -d)
(r2) db main     # Set breakpoint at main
(r2) dc          # Continue execution
(r2) ds          # Step into
(r2) dr          # Show registers
(r2) dbt         # Backtrace

# Patching
(r2) oo+         # Reopen in write mode
(r2) wa nop      # Write NOP at current position
(r2) wao nop     # Write NOP at current instruction (correct width)
```

---

## Practical Exercises

### Exercise 1 – ELF Anatomy of a Real Binary

```bash
# Compile a simple C program with different options
cat > hello.c << 'EOF'
#include <stdio.h>
#include <string.h>

int check_password(const char *input) {
    return strcmp(input, "secret123") == 0;
}

int main(int argc, char *argv[]) {
    if (argc < 2) { printf("Usage: %s <password>\n", argv[0]); return 1; }
    if (check_password(argv[1])) {
        printf("Access granted!\n");
        return 0;
    }
    printf("Access denied.\n");
    return 1;
}
EOF

# Compile with debug symbols
gcc -g -o hello_debug hello.c

# Compile without debug symbols (like a real release binary)
gcc -O2 -o hello_stripped hello.c && strip hello_stripped

# Compare
ls -la hello_debug hello_stripped
file hello_debug hello_stripped
nm hello_debug | grep check_password
nm hello_stripped 2>&1 || echo "Stripped - no symbols"

# ELF inspection
readelf -h hello_debug
objdump -d -M intel hello_debug | grep -A20 "<check_password>"
```

### Exercise 2 – Defeating a Simple Crackme with ltrace

```bash
# The stripped binary hides the password – ltrace reveals it
ltrace ./hello_stripped wrongpassword 2>&1
# Look for: strcmp("wrongpassword", "secret123")  = non-zero

# Or use strace to see the process
strace ./hello_stripped wrongpassword 2>&1 | grep -v "^---\|^+++"
```

### Exercise 3 – GDB Binary Analysis

```bash
gdb -q ./hello_debug << 'EOF'
set disassembly-flavor intel
break check_password
run testinput
disassemble
info registers
x/s $rdi
x/s $rsi
quit
EOF
```

### Exercise 4 – Radare2 Function Graph

```bash
# List functions and disassemble main
r2 -qc 'aaa; afl; pdf @main' ./hello_debug

# Analyse control flow
r2 -qc 'aaa; agf @main' ./hello_debug 2>/dev/null | head -40 || \
  r2 -qc 'aaa; pdf @sym.check_password' ./hello_debug
```

---

## Common Pitfalls

| Pitfall | Impact | Fix |
|---------|--------|-----|
| Forgetting ASLR when analysing addresses | Addresses change each run | Disable ASLR for testing: `echo 0 > /proc/sys/kernel/randomize_va_space` |
| Analysing in GDB without `set follow-fork-mode child` | Miss child process code | Set `follow-fork-mode` before `run` |
| ltrace not working on statically linked binaries | No library calls to trace | Use `strace` + `gdb` instead |
| Running unknown binaries on host system | Malware infection | Always use a disposable VM or container |
| Getting lost in large binaries without `aaa` | Miss important functions | Always run full analysis first; rename functions as you go |
| Confusing virtual addresses with file offsets | Incorrect seeks | Use `readelf -l` to understand address mapping |

---

## Summary

- **ELF format** organises Linux binaries into sections (`.text`, `.data`, `.bss`, `.plt`, `.got`) and segments
- **Binary protections** (ASLR, NX, canaries, PIE, RELRO) must be identified before exploitation
- **objdump** and **readelf** are quick static analysis tools; always start here
- **GDB** enables interactive dynamic analysis; use GEF/pwndbg for a better experience
- **strace** and **ltrace** are invaluable for rapidly understanding a binary's behaviour without full reversing
- **Radare2** provides a complete RE framework: scripting, visual mode, debugging, and patching
- Never run unknown binaries on a production or trusted system

---

## Further Reading

- [ELF Specification](https://refspecs.linuxbase.org/elf/elf.pdf)
- [GDB Documentation](https://www.gnu.org/software/gdb/documentation/)
- [GEF – GDB Enhanced Features](https://hugsy.github.io/gef/)
- [Radare2 Book](https://book.rada.re/)
- [pwn.college](https://pwn.college/) – free interactive binary exploitation labs
- *Hacking: The Art of Exploitation* by Jon Erickson (No Starch Press)
- *Practical Binary Analysis* by Dennis Andriesse (No Starch Press)
