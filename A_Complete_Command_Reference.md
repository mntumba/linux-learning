# Appendix A: Complete Linux Command Reference

> A comprehensive reference of Linux commands organized by category with syntax, descriptions, and examples.

---

## Table of Contents

1. [File Operations](#1-file-operations)
2. [File Viewing & Editing](#2-file-viewing--editing)
3. [Text Processing](#3-text-processing)
4. [Archive & Compression](#4-archive--compression)
5. [Permission Management](#5-permission-management)
6. [User Management](#6-user-management)
7. [Process Management](#7-process-management)
8. [System Information](#8-system-information)
9. [Networking](#9-networking)
10. [Package Management](#10-package-management)
11. [Service Management](#11-service-management)
12. [Disk Management](#12-disk-management)
13. [Security Tools](#13-security-tools)
14. [Shell Built-ins](#14-shell-built-ins)
15. [Scripting Helpers](#15-scripting-helpers)

---

## 1. File Operations

| Command | Syntax | Description |
|---------|--------|-------------|
| `ls` | `ls [options] [path]` | List directory contents |
| `cd` | `cd [directory]` | Change current directory |
| `pwd` | `pwd` | Print working directory |
| `mkdir` | `mkdir [options] dir` | Create directories |
| `rm` | `rm [options] file` | Remove files/directories |
| `cp` | `cp [options] src dst` | Copy files/directories |
| `mv` | `mv [options] src dst` | Move or rename files |
| `touch` | `touch [options] file` | Create empty file or update timestamps |
| `ln` | `ln [options] target link` | Create hard or symbolic links |
| `find` | `find [path] [expr]` | Search for files |
| `locate` | `locate pattern` | Find files by name (uses database) |
| `which` | `which command` | Show full path of a command |
| `whereis` | `whereis command` | Locate binary, source, man pages |
| `realpath` | `realpath path` | Resolve canonical absolute path |
| `basename` | `basename path` | Strip directory from filename |
| `dirname` | `dirname path` | Extract directory component |
| `rename` | `rename 's/old/new/' files` | Rename files using regex |
| `shred` | `shred [options] file` | Securely delete files |
| `rsync` | `rsync [options] src dst` | Sync files locally or remotely |

### Examples

```bash
# ls — list with details and hidden files
ls -lah /etc
ls -lS /var/log       # sort by size
ls --color=auto -1    # one per line with color

# cd — navigate directories
cd /var/log           # absolute path
cd ..                 # parent directory
cd -                  # previous directory

# mkdir — create nested directories
mkdir -p ~/projects/web/css
mkdir -m 750 /opt/myapp

# rm — remove safely
rm -i important.txt           # prompt before delete
rm -rf /tmp/old_project       # recursive force
rm --preserve-root /          # safety guard (default)

# find — powerful file search
find /home -name "*.log" -mtime +7 -delete
find / -perm /4000 -type f    # find SUID files
find . -size +100M -exec ls -lh {} \;

# cp — copy with archive preservation
cp -av /etc /backup/etc-$(date +%F)
cp -u src/* dst/              # copy only newer files

# mv — rename and move
mv oldname.txt newname.txt
mv *.jpg ~/Pictures/

# ln — symbolic link
ln -s /usr/local/bin/python3 /usr/bin/python
ln -sf /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

# locate — fast file lookup
locate httpd.conf
sudo updatedb                  # refresh database

# rsync — remote sync
rsync -avz --progress /local/ user@remote:/backup/
rsync -av --delete src/ dst/  # mirror directories
```

---

## 2. File Viewing & Editing

| Command | Syntax | Description |
|---------|--------|-------------|
| `cat` | `cat [options] file` | Concatenate and display files |
| `less` | `less [options] file` | Page through file (backward/forward) |
| `more` | `more [options] file` | Page through file (forward only) |
| `head` | `head [options] file` | Show first N lines |
| `tail` | `tail [options] file` | Show last N lines |
| `nano` | `nano [options] file` | Simple terminal text editor |
| `vim` | `vim [options] file` | Powerful modal text editor |
| `vi` | `vi file` | Original visual editor |
| `gedit` | `gedit file` | GNOME graphical text editor |
| `file` | `file filename` | Determine file type |
| `stat` | `stat [options] file` | Detailed file/filesystem status |
| `wc` | `wc [options] file` | Count lines, words, characters |
| `xxd` | `xxd file` | Hex dump of a file |
| `od` | `od [options] file` | Octal/hex dump |
| `strings` | `strings file` | Extract printable strings |
| `tac` | `tac file` | Reverse cat (last line first) |
| `nl` | `nl file` | Number lines of a file |
| `column` | `column -t file` | Format output in columns |

### Examples

```bash
# cat — view and combine files
cat /etc/os-release
cat -n file.txt               # with line numbers
cat file1 file2 > combined.txt

# less — interactive viewer
less /var/log/syslog
# Inside less: / search, n next, q quit, G end, g start

# head / tail
head -20 /etc/passwd
tail -f /var/log/auth.log     # follow live output
tail -n +5 file.txt           # skip first 4 lines

# wc — count stats
wc -l /etc/passwd             # count users
wc -w document.txt
cat access.log | wc -l

# stat — file metadata
stat /etc/passwd
stat -c "%n %s %y" *.txt      # custom format

# vim — essential commands
# vim file.txt
# i = insert mode, Esc = normal, :w save, :q quit, :wq save+quit
# /pattern search, dd delete line, yy copy line, p paste
# :%s/old/new/g global replace

# file — determine type
file /bin/bash
file *.txt
file -i image.png             # MIME type
```

---

## 3. Text Processing

| Command | Syntax | Description |
|---------|--------|-------------|
| `grep` | `grep [options] pattern file` | Search text with regex |
| `sed` | `sed [options] 'script' file` | Stream editor |
| `awk` | `awk 'program' file` | Pattern scanning and processing |
| `cut` | `cut [options] file` | Remove sections from lines |
| `sort` | `sort [options] file` | Sort lines of text |
| `uniq` | `uniq [options] file` | Filter duplicate lines |
| `tr` | `tr [options] set1 set2` | Translate or delete characters |
| `diff` | `diff [options] file1 file2` | Compare files line by line |
| `patch` | `patch [options] file` | Apply diff patches |
| `strings` | `strings [options] file` | Print printable strings |
| `comm` | `comm file1 file2` | Compare sorted files line by line |
| `join` | `join file1 file2` | Join lines with common field |
| `paste` | `paste file1 file2` | Merge lines of files |
| `expand` | `expand file` | Convert tabs to spaces |
| `fold` | `fold -w N file` | Wrap lines at N characters |
| `fmt` | `fmt -w N file` | Reformat paragraph text |
| `iconv` | `iconv -f enc1 -t enc2 file` | Convert character encoding |

### Examples

```bash
# grep — search patterns
grep -rn "error" /var/log/      # recursive with line numbers
grep -i "failed" auth.log       # case-insensitive
grep -v "^#" /etc/nginx.conf    # exclude comment lines
grep -E "^[0-9]{1,3}\." file    # extended regex
grep -o "([0-9]+\.){3}[0-9]+" file  # extract IPs

# sed — stream editing
sed 's/foo/bar/g' file.txt              # global replace
sed -i.bak 's/old/new/g' config.txt    # in-place with backup
sed -n '10,20p' file.txt               # print lines 10-20
sed '/^#/d' config.txt                 # delete comment lines
sed 's/[[:space:]]*$//' file.txt       # strip trailing whitespace

# awk — data processing
awk '{print $1, $3}' data.txt          # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd       # custom delimiter
awk 'NR==5' file.txt                   # print line 5
awk '$3 > 100 {print $1, $3}' data     # conditional print
awk '{sum += $1} END {print sum}' nums # sum a column

# cut — extract fields
cut -d: -f1,3 /etc/passwd              # fields 1 and 3
cut -c1-10 file.txt                    # first 10 characters
ls -l | cut -d' ' -f5                 # file sizes

# sort — sorting
sort -k2 -n data.txt                   # sort by field 2 numerically
sort -u file.txt                       # sort and deduplicate
sort -r file.txt                       # reverse sort
ps aux | sort -k3 -rn | head -10      # top CPU processes

# uniq — deduplicate
sort file.txt | uniq -c               # count occurrences
sort file.txt | uniq -d               # only duplicates
sort file.txt | uniq -u               # only unique lines

# tr — character translation
echo "Hello World" | tr '[:upper:]' '[:lower:]'
cat file.txt | tr -d '\r'             # remove carriage returns
echo "a:b:c" | tr ':' '\n'           # split on delimiter

# diff — compare files
diff -u original.txt modified.txt     # unified diff
diff -r dir1/ dir2/                   # recursive directory diff
```

---

## 4. Archive & Compression

| Command | Syntax | Description |
|---------|--------|-------------|
| `tar` | `tar [options] archive files` | Tape archive — create/extract archives |
| `gzip` | `gzip [options] file` | Compress with gzip |
| `gunzip` | `gunzip file.gz` | Decompress gzip files |
| `zip` | `zip [options] archive files` | Create ZIP archives |
| `unzip` | `unzip [options] archive` | Extract ZIP archives |
| `bzip2` | `bzip2 [options] file` | Compress with bzip2 |
| `bunzip2` | `bunzip2 file.bz2` | Decompress bzip2 files |
| `xz` | `xz [options] file` | Compress with xz (LZMA) |
| `7z` | `7z [options] archive files` | 7-Zip compression |
| `zcat` | `zcat file.gz` | View gzip file without extracting |
| `zless` | `zless file.gz` | Page through gzip file |
| `bzcat` | `bzcat file.bz2` | View bzip2 file |
| `rar` | `rar [options] archive files` | RAR archive tool |
| `unrar` | `unrar x archive.rar` | Extract RAR archives |
| `pigz` | `pigz file` | Parallel gzip (faster) |
| `lz4` | `lz4 file` | Very fast LZ4 compression |

### Examples

```bash
# tar — archive Swiss Army knife
tar -czvf archive.tar.gz /path/to/dir    # create compressed archive
tar -xzvf archive.tar.gz                  # extract
tar -tzvf archive.tar.gz                  # list contents
tar -xzvf archive.tar.gz -C /opt/        # extract to directory
tar -cjvf archive.tar.bz2 /data/         # use bzip2 compression
tar -cJvf archive.tar.xz /data/          # use xz compression
tar --exclude='*.log' -czvf backup.tar.gz /var/www/

# gzip / gunzip
gzip -9 largefile.txt                     # maximum compression
gzip -k file.txt                          # keep original file
gunzip archive.tar.gz                     # -> archive.tar
zcat file.gz | grep "pattern"            # grep in compressed file

# zip / unzip
zip -r website.zip /var/www/html/
zip -e secret.zip file.txt               # encrypt with password
unzip archive.zip -d /opt/extracted/
unzip -l archive.zip                      # list contents

# 7z — best compression ratio
7z a archive.7z /data/ -mx=9            # max compression
7z x archive.7z -o/opt/               # extract
7z l archive.7z                          # list contents

# bzip2 / xz comparison
bzip2 -9 file.txt                        # slower but better than gzip
xz -9 file.txt                           # best compression, slowest
xz -T 4 file.txt                        # use 4 threads
```

---

## 5. Permission Management

| Command | Syntax | Description |
|---------|--------|-------------|
| `chmod` | `chmod [options] mode file` | Change file permissions |
| `chown` | `chown [options] owner:group file` | Change file owner |
| `chgrp` | `chgrp [options] group file` | Change group ownership |
| `umask` | `umask [mask]` | Set default permission mask |
| `setfacl` | `setfacl [options] acl file` | Set Access Control Lists |
| `getfacl` | `getfacl file` | Get Access Control Lists |
| `chattr` | `chattr [options] file` | Change file attributes |
| `lsattr` | `lsattr [options] file` | List file attributes |

### Permission Notation

```
Symbolic:  rwxrwxrwx  (owner-group-others)
Octal:     7  5  5
           │  │  └── others: r-x
           │  └───── group:  r-x
           └──────── owner:  rwx

Special bits:
  4000 = SUID (run as file owner)
  2000 = SGID (run as file group)
  1000 = Sticky bit (only owner can delete)
```

### Examples

```bash
# chmod — change permissions
chmod 755 script.sh               # rwxr-xr-x
chmod 644 config.txt              # rw-r--r--
chmod u+x,g-w script.sh          # symbolic mode
chmod -R 750 /opt/myapp          # recursive
chmod +s /usr/local/bin/tool     # set SUID bit
chmod +t /tmp                    # set sticky bit

# chown — change ownership
chown alice file.txt
chown alice:developers file.txt
chown -R www-data:www-data /var/www/html
chown --reference=ref.txt target.txt

# umask
umask                            # show current mask (e.g., 0022)
umask 0027                       # set mask: files=640, dirs=750

# setfacl / getfacl
setfacl -m u:bob:rwx /data/project
setfacl -m g:devs:rx /data/
setfacl -R -m u:alice:rwx /project/
getfacl /data/project
setfacl -x u:bob /data/project  # remove ACL entry

# chattr — immutable attribute
chattr +i /etc/resolv.conf       # make immutable
chattr -i /etc/resolv.conf       # remove immutable
lsattr /etc/resolv.conf
```

---

## 6. User Management

| Command | Syntax | Description |
|---------|--------|-------------|
| `useradd` | `useradd [options] username` | Create new user (low-level) |
| `adduser` | `adduser [options] username` | Interactive user creation |
| `userdel` | `userdel [options] username` | Delete user account |
| `usermod` | `usermod [options] username` | Modify user account |
| `passwd` | `passwd [username]` | Change user password |
| `su` | `su [options] [username]` | Switch user |
| `sudo` | `sudo [options] command` | Execute as another user |
| `groups` | `groups [username]` | Show group memberships |
| `id` | `id [username]` | Print user/group IDs |
| `who` | `who [options]` | Show logged-in users |
| `whoami` | `whoami` | Print current username |
| `w` | `w [username]` | Show who is logged in + activity |
| `last` | `last [options]` | Show login history |
| `lastlog` | `lastlog` | Last login for all users |
| `finger` | `finger [username]` | User information lookup |
| `groupadd` | `groupadd [options] group` | Create new group |
| `groupdel` | `groupdel group` | Delete group |
| `groupmod` | `groupmod [options] group` | Modify group |
| `newgrp` | `newgrp group` | Login to a new group |
| `visudo` | `visudo` | Edit sudoers file safely |

### Examples

```bash
# useradd / adduser
useradd -m -s /bin/bash -G sudo,www-data alice
useradd -u 1500 -g developers -d /home/alice alice
adduser bob                      # interactive, Debian-style

# usermod — modify accounts
usermod -aG docker alice        # add to group (keep existing)
usermod -s /bin/zsh alice       # change shell
usermod -L alice                 # lock account
usermod -U alice                 # unlock account
usermod -e 2025-12-31 alice     # set expiry date

# userdel
userdel alice                    # keep home directory
userdel -r alice                 # remove home + mail spool

# passwd
passwd                           # change own password
sudo passwd alice                # change alice's password
passwd -e alice                  # force password change at next login
passwd -l alice                  # lock account

# sudo
sudo apt update
sudo -u postgres psql
sudo -i                          # interactive root shell
sudo -l                          # list allowed commands

# id / groups / who
id alice
groups alice
who
w
last | head -20
last reboot                      # show reboot history
```

---

## 7. Process Management

| Command | Syntax | Description |
|---------|--------|-------------|
| `ps` | `ps [options]` | Report process status |
| `top` | `top [options]` | Interactive process viewer |
| `htop` | `htop` | Enhanced interactive process viewer |
| `kill` | `kill [signal] PID` | Send signal to process |
| `killall` | `killall [options] name` | Kill processes by name |
| `pkill` | `pkill [options] pattern` | Kill processes by pattern |
| `nice` | `nice -n N command` | Run with adjusted priority |
| `renice` | `renice N -p PID` | Change priority of running process |
| `pgrep` | `pgrep [options] pattern` | Find processes by name |
| `pstree` | `pstree [options]` | Display process tree |
| `jobs` | `jobs [options]` | List background jobs |
| `bg` | `bg [job]` | Resume job in background |
| `fg` | `fg [job]` | Bring job to foreground |
| `nohup` | `nohup command &` | Run immune to hangups |
| `screen` | `screen [options]` | Terminal multiplexer |
| `tmux` | `tmux [options]` | Terminal multiplexer (modern) |
| `strace` | `strace command` | Trace system calls |
| `lsof` | `lsof [options]` | List open files |
| `fuser` | `fuser [options] file` | Identify processes using file |
| `watch` | `watch [options] command` | Execute command periodically |

### Examples

```bash
# ps — process snapshot
ps aux                           # all processes, BSD style
ps -ef                           # all processes, UNIX style
ps aux | grep nginx
ps -u alice                      # alice's processes
ps --sort=-%cpu | head -10       # sort by CPU

# top / htop
top -b -n1 > top_snapshot.txt   # batch mode, one iteration
htop -u alice                   # filter by user

# kill — signals
kill 1234                        # SIGTERM (graceful)
kill -9 1234                     # SIGKILL (force)
kill -HUP 1234                   # SIGHUP (reload config)
kill -l                          # list all signals

# nice / renice
nice -n 19 backup_script.sh     # lowest priority
nice -n -10 critical_task.sh    # high priority (needs root)
renice -n 10 -p 5678
renice -n -5 -u alice           # renice all alice's processes

# nohup — persist after logout
nohup ./long_running.sh > output.log 2>&1 &
nohup python3 server.py &

# tmux — session management
tmux new -s mysession
tmux ls
tmux attach -t mysession
tmux kill-session -t mysession
# Inside tmux: Ctrl+B = prefix
# Ctrl+B c = new window, Ctrl+B % = split vertical
# Ctrl+B " = split horizontal, Ctrl+B d = detach

# lsof — open files
lsof -i :80                     # what's on port 80
lsof -u alice                   # alice's open files
lsof /var/log/syslog            # who has this file open
```

---

## 8. System Information

| Command | Syntax | Description |
|---------|--------|-------------|
| `uname` | `uname [options]` | Print system information |
| `hostname` | `hostname [options]` | Show or set hostname |
| `uptime` | `uptime` | System uptime and load averages |
| `dmesg` | `dmesg [options]` | Kernel ring buffer messages |
| `lscpu` | `lscpu` | CPU architecture information |
| `lsusb` | `lsusb [options]` | List USB devices |
| `lspci` | `lspci [options]` | List PCI devices |
| `lsblk` | `lsblk [options]` | List block devices |
| `free` | `free [options]` | Memory usage |
| `vmstat` | `vmstat [options]` | Virtual memory statistics |
| `iostat` | `iostat [options]` | I/O statistics |
| `df` | `df [options]` | Disk space usage |
| `du` | `du [options] path` | Disk usage of files/dirs |
| `lshw` | `lshw [options]` | Detailed hardware information |
| `hwinfo` | `hwinfo [options]` | Hardware information |
| `dmidecode` | `dmidecode [options]` | DMI/SMBIOS information |
| `inxi` | `inxi [options]` | System information tool |
| `sensors` | `sensors` | Read hardware sensors |
| `mpstat` | `mpstat [options]` | CPU statistics per processor |
| `sar` | `sar [options]` | System activity reporter |

### Examples

```bash
# uname — kernel info
uname -a                         # all info
uname -r                         # kernel release
uname -m                         # machine type (x86_64)

# lscpu — CPU details
lscpu | grep -E "CPU|Thread|Core|Socket|MHz"

# memory
free -h                          # human-readable
free -m                          # in megabytes
cat /proc/meminfo                # detailed memory info

# disk
df -h                            # human-readable sizes
df -i                            # inode usage
df -h /home                      # specific filesystem
du -sh /var/log                  # directory size
du -sh /* 2>/dev/null | sort -h  # largest root dirs
du -h --max-depth=1 /var

# dmesg — kernel messages
dmesg -T | tail -50             # with timestamps, last 50
dmesg | grep -i error
dmesg | grep -i usb             # USB events

# vmstat / iostat
vmstat 2 5                       # every 2 seconds, 5 times
iostat -x 1 3                    # extended I/O stats
mpstat -P ALL 1                  # per-CPU stats

# lspci / lsusb
lspci -v | grep -A5 VGA         # GPU info
lsusb -v | head -30
```

---

## 9. Networking

| Command | Syntax | Description |
|---------|--------|-------------|
| `ip` | `ip [options] object command` | Network configuration (modern) |
| `ifconfig` | `ifconfig [interface]` | Network interface config (legacy) |
| `ping` | `ping [options] host` | Test network connectivity |
| `traceroute` | `traceroute host` | Trace network path |
| `tracepath` | `tracepath host` | Trace path (no root needed) |
| `netstat` | `netstat [options]` | Network statistics (legacy) |
| `ss` | `ss [options]` | Socket statistics (modern) |
| `nslookup` | `nslookup [host]` | DNS query tool |
| `dig` | `dig [options] host` | DNS lookup tool |
| `host` | `host [options] name` | DNS lookup |
| `curl` | `curl [options] URL` | Transfer data with URLs |
| `wget` | `wget [options] URL` | Non-interactive download |
| `ssh` | `ssh [options] user@host` | Secure shell |
| `scp` | `scp [options] src dst` | Secure copy |
| `sftp` | `sftp user@host` | Secure FTP |
| `nmap` | `nmap [options] target` | Network scanner |
| `nc` | `nc [options] host port` | Netcat — network Swiss Army knife |
| `tcpdump` | `tcpdump [options]` | Packet capture |
| `arp` | `arp [options]` | ARP table management |
| `route` | `route [options]` | Routing table (legacy) |
| `mtr` | `mtr host` | Combines ping + traceroute |
| `iperf3` | `iperf3 [options]` | Network bandwidth test |
| `whois` | `whois domain` | Domain registration info |
| `rsync` | `rsync [options] src dst` | Remote sync |

### Examples

```bash
# ip — network configuration
ip addr show
ip addr add 192.168.1.100/24 dev eth0
ip link set eth0 up
ip route show
ip route add default via 192.168.1.1
ip neigh show                    # ARP table

# ss / netstat — connections
ss -tuln                         # TCP/UDP listening, no DNS
ss -tulnp                        # with process info
ss -s                            # summary
netstat -tulnp                   # legacy equivalent

# ping / traceroute
ping -c 4 8.8.8.8
ping -i 0.2 -c 100 host         # fast ping
traceroute -n 8.8.8.8           # no DNS resolution
mtr --report google.com         # combined tool

# dig — DNS queries
dig google.com A
dig google.com MX
dig @8.8.8.8 google.com         # specific DNS server
dig -x 8.8.8.8                  # reverse lookup
dig +short google.com           # just the answer

# curl — HTTP requests
curl -I https://example.com     # headers only
curl -L -o file.txt https://url  # follow redirects, save
curl -X POST -d '{"key":"val"}' -H "Content-Type: application/json" https://api.example.com
curl -u user:pass https://protected.com
curl --proxy http://proxy:8080 https://example.com

# wget — download files
wget https://example.com/file.tar.gz
wget -r --no-parent https://site.com/docs/  # recursive
wget -c large_file.tar.gz       # resume download
wget -q --spider https://url    # check without downloading

# ssh — secure connections
ssh alice@192.168.1.100
ssh -p 2222 alice@server
ssh -i ~/.ssh/mykey user@host
ssh -L 8080:localhost:80 user@host  # local port forward
ssh -D 1080 user@host           # dynamic SOCKS proxy

# scp — secure copy
scp file.txt user@host:/remote/path/
scp -r /local/dir user@host:/remote/dir/
scp -P 2222 user@host:/remote/file ./

# nmap — network scanning
nmap 192.168.1.0/24             # host discovery
nmap -sV -p 80,443 target       # service version detection
nmap -O target                  # OS detection (root)
nmap -A target                  # aggressive scan
nmap -sU -p 53,161 target       # UDP scan

# nc — netcat
nc -zv host 1-1000              # port scan
nc -l -p 8888                   # listen on port
nc host 80                      # connect to port
echo "GET / HTTP/1.0" | nc host 80  # raw HTTP

# tcpdump — packet capture
tcpdump -i eth0
tcpdump -i eth0 port 80
tcpdump -i eth0 host 192.168.1.1 -w capture.pcap
tcpdump -r capture.pcap
```

---

## 10. Package Management

### APT (Debian/Ubuntu)

| Command | Description |
|---------|-------------|
| `apt update` | Update package index |
| `apt upgrade` | Upgrade all packages |
| `apt install pkg` | Install package |
| `apt remove pkg` | Remove package (keep config) |
| `apt purge pkg` | Remove package + config |
| `apt autoremove` | Remove unused dependencies |
| `apt search term` | Search packages |
| `apt show pkg` | Package details |
| `apt list --installed` | List installed packages |
| `apt-get dist-upgrade` | Full distribution upgrade |
| `apt-cache policy pkg` | Show package versions |
| `apt-mark hold pkg` | Hold package at current version |

### DPKG (Low-level Debian)

| Command | Description |
|---------|-------------|
| `dpkg -i file.deb` | Install .deb package |
| `dpkg -r package` | Remove package |
| `dpkg -l` | List installed packages |
| `dpkg -L pkg` | List files in package |
| `dpkg -S /path/file` | Find which package owns a file |
| `dpkg --configure -a` | Configure unconfigured packages |

### Examples

```bash
# apt workflow
sudo apt update && sudo apt upgrade -y
sudo apt install nginx curl git -y
sudo apt install --no-install-recommends package
sudo apt remove package
sudo apt purge package
sudo apt autoremove --purge

# dpkg
sudo dpkg -i google-chrome.deb
dpkg -l | grep nginx
dpkg -L nginx | head -20
dpkg -S /usr/bin/python3

# snap
snap find firefox
sudo snap install firefox
sudo snap list
sudo snap refresh
sudo snap remove firefox

# flatpak
flatpak search gimp
flatpak install flathub org.gimp.GIMP
flatpak list
flatpak update
flatpak uninstall org.gimp.GIMP
```

---

## 11. Service Management

| Command | Syntax | Description |
|---------|--------|-------------|
| `systemctl start` | `systemctl start service` | Start a service |
| `systemctl stop` | `systemctl stop service` | Stop a service |
| `systemctl restart` | `systemctl restart service` | Restart a service |
| `systemctl reload` | `systemctl reload service` | Reload config without restart |
| `systemctl enable` | `systemctl enable service` | Enable at boot |
| `systemctl disable` | `systemctl disable service` | Disable at boot |
| `systemctl status` | `systemctl status service` | Show service status |
| `systemctl is-active` | `systemctl is-active service` | Check if active |
| `systemctl list-units` | `systemctl list-units` | List all units |
| `systemctl daemon-reload` | `systemctl daemon-reload` | Reload unit files |
| `journalctl` | `journalctl [options]` | Query systemd journal |

### Examples

```bash
# systemctl — service control
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
sudo systemctl enable --now nginx    # enable + start
sudo systemctl disable nginx
sudo systemctl status nginx
systemctl is-active nginx && echo "running"
systemctl list-units --type=service --state=running
systemctl list-unit-files | grep enabled
sudo systemctl daemon-reload          # after editing unit files

# journalctl — log viewing
journalctl -u nginx                   # nginx logs
journalctl -u nginx -f                # follow live
journalctl -u nginx --since "1 hour ago"
journalctl -p err..crit               # errors and critical
journalctl -b                         # since last boot
journalctl -b -1                      # previous boot
journalctl --disk-usage
journalctl --vacuum-time=7d          # clean logs older than 7 days
```

---

## 12. Disk Management

| Command | Syntax | Description |
|---------|--------|-------------|
| `fdisk` | `fdisk [options] device` | MBR partition editor |
| `gdisk` | `gdisk device` | GPT partition editor |
| `parted` | `parted [options] device` | Partition editor (GPT + MBR) |
| `mkfs` | `mkfs.[type] device` | Create filesystem |
| `mount` | `mount [options] dev mountpoint` | Mount filesystem |
| `umount` | `umount dev/mountpoint` | Unmount filesystem |
| `fsck` | `fsck [options] device` | Filesystem check/repair |
| `lsblk` | `lsblk [options]` | List block devices |
| `blkid` | `blkid [device]` | Block device attributes/UUID |
| `dd` | `dd if=src of=dst` | Low-level copy |
| `sync` | `sync` | Flush filesystem buffers |
| `tune2fs` | `tune2fs [options] device` | Tune ext2/3/4 filesystem |
| `e2fsck` | `e2fsck device` | ext2/3/4 filesystem check |
| `resize2fs` | `resize2fs device [size]` | Resize ext2/3/4 filesystem |
| `pvdisplay` | `pvdisplay` | Display LVM physical volumes |
| `vgdisplay` | `vgdisplay` | Display LVM volume groups |
| `lvdisplay` | `lvdisplay` | Display LVM logical volumes |
| `pvcreate` | `pvcreate device` | Create physical volume |
| `vgcreate` | `vgcreate vg devices` | Create volume group |
| `lvcreate` | `lvcreate [options] vg` | Create logical volume |

### Examples

```bash
# fdisk / gdisk — partitioning
sudo fdisk -l                           # list all partitions
sudo fdisk /dev/sdb                     # interactive partition editor
# Commands: n=new, d=delete, p=print, w=write, q=quit
sudo gdisk /dev/sdb                     # GPT equivalent

# mkfs — create filesystems
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.xfs /dev/sdb2
sudo mkfs.vfat -F32 /dev/sdc1          # FAT32 for USB

# mount / umount
sudo mount /dev/sdb1 /mnt/data
sudo mount -t ext4 /dev/sdb1 /mnt/data
sudo mount -o ro /dev/sdb1 /mnt/data   # read-only
sudo umount /mnt/data
sudo umount -l /mnt/data               # lazy unmount

# /etc/fstab entry
# UUID=xxxx-xxxx  /mnt/data  ext4  defaults  0  2

# blkid — find UUIDs
sudo blkid
sudo blkid /dev/sdb1

# dd — disk operations
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress  # clone disk
sudo dd if=/dev/zero of=/dev/sdb bs=1M                 # wipe disk
sudo dd if=ubuntu.iso of=/dev/sdc bs=4M status=progress # write ISO

# fsck — filesystem check
sudo fsck /dev/sdb1                     # run only on unmounted!
sudo fsck -y /dev/sdb1                  # auto-fix errors

# LVM workflow
sudo pvcreate /dev/sdb /dev/sdc
sudo vgcreate datavg /dev/sdb /dev/sdc
sudo lvcreate -L 50G -n datalv datavg
sudo mkfs.ext4 /dev/datavg/datalv
sudo lvextend -L +20G /dev/datavg/datalv
sudo resize2fs /dev/datavg/datalv      # grow filesystem
```

---

## 13. Security Tools

| Command | Syntax | Description |
|---------|--------|-------------|
| `gpg` | `gpg [options]` | GNU Privacy Guard encryption |
| `openssl` | `openssl [command]` | SSL/TLS toolkit |
| `ssh-keygen` | `ssh-keygen [options]` | Generate SSH key pairs |
| `ssh-copy-id` | `ssh-copy-id user@host` | Copy public key to server |
| `fail2ban-client` | `fail2ban-client [options]` | Fail2ban management |
| `ufw` | `ufw [options]` | Uncomplicated Firewall |
| `iptables` | `iptables [options]` | Packet filter rules |
| `nftables` | `nft [options]` | Next-gen packet filter |
| `auditctl` | `auditctl [options]` | Audit system control |
| `ausearch` | `ausearch [options]` | Search audit logs |
| `chkrootkit` | `chkrootkit` | Rootkit detector |
| `rkhunter` | `rkhunter --check` | Rootkit hunter |
| `lynis` | `lynis audit system` | Security auditing tool |
| `nmap` | `nmap [options] target` | Network scanner |

### Examples

```bash
# gpg — encryption
gpg --gen-key
gpg --list-keys
gpg -e -r alice@example.com file.txt      # encrypt
gpg -d file.txt.gpg                        # decrypt
gpg --export -a "Alice" > alice.pub.asc   # export public key
gpg --import alice.pub.asc                 # import public key
gpg --sign document.txt                    # sign document
gpg --verify document.txt.sig             # verify signature

# openssl — SSL operations
openssl genrsa -out private.key 4096
openssl req -new -key private.key -out cert.csr
openssl x509 -req -days 365 -in cert.csr -signkey private.key -out cert.crt
openssl x509 -in cert.crt -text -noout   # inspect certificate
openssl s_client -connect host:443        # test SSL connection
openssl passwd -6 "password"              # generate SHA-512 hash
openssl rand -hex 32                       # generate random key

# ssh-keygen
ssh-keygen -t ed25519 -C "alice@example.com"
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
ssh-keygen -lf ~/.ssh/id_rsa.pub         # show fingerprint

# ufw — firewall
sudo ufw enable
sudo ufw status verbose
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw deny 23/tcp
sudo ufw delete allow 80/tcp
sudo ufw logging on

# iptables
sudo iptables -L -n -v                    # list rules
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -j DROP           # default deny
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## 14. Shell Built-ins

| Command | Syntax | Description |
|---------|--------|-------------|
| `echo` | `echo [options] text` | Print text to stdout |
| `printf` | `printf format args` | Formatted output |
| `read` | `read [options] var` | Read input from stdin |
| `export` | `export var=value` | Export variable to environment |
| `source` | `source file` | Execute file in current shell |
| `alias` | `alias name='cmd'` | Create command alias |
| `history` | `history [n]` | Command history |
| `type` | `type command` | Show command type |
| `hash` | `hash [command]` | Hash command locations |
| `set` | `set [options]` | Set shell options/variables |
| `unset` | `unset var` | Unset variable or function |
| `env` | `env [options]` | Print/modify environment |
| `exec` | `exec command` | Replace shell with command |
| `exit` | `exit [n]` | Exit shell with code n |
| `return` | `return [n]` | Return from function |
| `shift` | `shift [n]` | Shift positional parameters |
| `trap` | `trap handler signal` | Set signal handlers |
| `wait` | `wait [pid]` | Wait for process |
| `declare` | `declare [options] var` | Declare variables |
| `local` | `local var=value` | Local variable in function |
| `readonly` | `readonly var` | Make variable read-only |

### Examples

```bash
# echo / printf
echo "Hello, World!"
echo -e "Line1\nLine2\tTabbed"       # interpret escapes
echo -n "no newline"
printf "%-20s %5d\n" "Name" 42       # formatted output
printf "0x%X\n" 255                   # hex output

# read — user input
read -p "Enter name: " name
read -s -p "Password: " pass         # silent input
read -t 5 -p "Timeout in 5s: " ans
IFS=: read user pass uid gid <<< "alice:x:1000:1000"

# export / environment
export PATH=$PATH:/opt/myapp/bin
export -p                             # show all exported vars
env | grep HOME
env -i PATH=/usr/bin command         # clean environment

# alias
alias ll='ls -lah'
alias ..='cd ..'
alias grep='grep --color=auto'
alias update='sudo apt update && sudo apt upgrade'
unalias ll
alias                                 # list all aliases

# history
history
history 20
!500                                  # re-run command 500
!!                                    # re-run last command
!ssh                                  # re-run last ssh command
CTRL+R                                # reverse search
history -c                            # clear history

# set — shell options
set -e                                # exit on error
set -x                                # debug mode
set -u                                # error on undefined vars
set -o pipefail                       # catch pipe failures
set +x                                # disable debug

# trap — signal handling
trap "echo 'Interrupted'; exit 1" SIGINT SIGTERM
trap "rm -f /tmp/lockfile" EXIT      # cleanup on exit
trap '' SIGINT                        # ignore Ctrl+C

# declare
declare -i count=0                    # integer
declare -r CONST="immutable"          # readonly
declare -a myarray=("a" "b" "c")     # indexed array
declare -A mymap=(["key"]="value")   # associative array
```

---

## 15. Scripting Helpers

| Command | Syntax | Description |
|---------|--------|-------------|
| `test` | `test expression` | Evaluate expression |
| `[` | `[ expression ]` | Alias for test |
| `[[` | `[[ expression ]]` | Enhanced test (Bash) |
| `expr` | `expr expression` | Evaluate expression |
| `let` | `let var=expr` | Arithmetic evaluation |
| `seq` | `seq [first] [inc] last` | Generate sequence of numbers |
| `date` | `date [options]` | Print or set date/time |
| `sleep` | `sleep n` | Pause for N seconds |
| `wait` | `wait [pid]` | Wait for background jobs |
| `xargs` | `xargs [options] command` | Build commands from stdin |
| `tee` | `tee [options] files` | Read stdin; write stdout + file |
| `paste` | `paste [options] files` | Merge lines of files |
| `bc` | `bc [options]` | Arbitrary precision calculator |
| `timeout` | `timeout N command` | Run with time limit |
| `time` | `time command` | Time command execution |
| `basename` | `basename path [suffix]` | Filename from path |
| `dirname` | `dirname path` | Directory from path |
| `mktemp` | `mktemp [template]` | Create temporary file |
| `logger` | `logger message` | Add entry to syslog |
| `getopt` | `getopt optstring params` | Parse options |
| `getopts` | `getopts optstring var` | Parse options (built-in) |

### Examples

```bash
# test / [ ] / [[ ]]
test -f /etc/passwd && echo "exists"
[ -d /home ] && echo "directory exists"
[[ -z "$var" ]] && echo "empty"
[[ "$str" == *pattern* ]] && echo "matches"
[[ -r file && -w file ]] && echo "readable and writable"

# Comparison operators
# -eq -ne -lt -gt -le -ge  (numeric)
# == != < >                  (string in [[ ]])
# -f -d -e -r -w -x         (file tests)
# -z -n                      (string empty/not-empty)

# seq
seq 1 10
seq 0 2 20                   # 0 2 4 6 ... 20
for i in $(seq 1 5); do echo $i; done

# date
date
date +"%Y-%m-%d %H:%M:%S"
date +"%F"                   # ISO format: 2024-01-15
date -d "yesterday"
date -d "next Friday"
date -d "+7 days" +%F
TIMESTAMP=$(date +%s)       # Unix timestamp

# xargs — build command lines
find . -name "*.tmp" | xargs rm
find . -name "*.txt" | xargs grep "pattern"
echo "file1 file2 file3" | xargs -n1 touch
cat urls.txt | xargs -P4 -I{} curl -O {}   # parallel downloads
ls *.log | xargs -I{} mv {} /archive/

# tee — split output
command | tee output.txt
command | tee -a output.txt  # append
command | tee output.txt | grep "error"

# bc — calculations
echo "scale=4; 22/7" | bc    # pi approximation
echo "2^10" | bc
echo "sqrt(2)" | bc -l       # math library

# timeout / time
timeout 30 long_running_command
time find / -name "*.conf" 2>/dev/null

# mktemp — temp files
TMPFILE=$(mktemp /tmp/script.XXXXXX)
TMPDIR=$(mktemp -d /tmp/workdir.XXXXXX)
trap "rm -rf $TMPDIR" EXIT

# Script template
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

readonly SCRIPT_DIR=$(dirname "$0")
readonly SCRIPT_NAME=$(basename "$0")

log() { echo "[$(date +%F\ %T)] $*" | tee -a /var/log/myscript.log; }
die() { log "ERROR: $*" >&2; exit 1; }

main() {
    log "Starting $SCRIPT_NAME"
    # script logic here
}
main "$@"
```

---

## Quick Reference Tables

### Signal Numbers

| Signal | Number | Description |
|--------|--------|-------------|
| SIGHUP | 1 | Hangup / reload config |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Quit (Ctrl+\) |
| SIGKILL | 9 | Kill (cannot be caught) |
| SIGTERM | 15 | Terminate gracefully |
| SIGSTOP | 19 | Stop process (cannot be caught) |
| SIGTSTP | 20 | Stop (Ctrl+Z) |
| SIGCONT | 18 | Continue stopped process |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of shell built-in |
| 126 | Command not executable |
| 127 | Command not found |
| 128+n | Signal n received |
| 130 | Interrupted (Ctrl+C) |

### File Permission Octal Quick Reference

| Octal | Permissions | Use Case |
|-------|-------------|----------|
| 777 | rwxrwxrwx | World writable (avoid!) |
| 755 | rwxr-xr-x | Executables, directories |
| 750 | rwxr-x--- | Group-accessible dirs |
| 644 | rw-r--r-- | Regular files |
| 640 | rw-r----- | Config files with secrets |
| 600 | rw------- | Private files (SSH keys) |
| 400 | r-------- | Read-only sensitive files |

---

*Back to [Index](INDEX.md) | Next: [Appendix B — Aliases & Acronyms](B_Aliases_and_Acronyms.md)*
