# 10 — Networking Basics

> **Module 2 · Lesson 4** | Difficulty: ★★★☆☆ Intermediate | Time: ~90 min

---

## Learning Objectives

- Understand the OSI and TCP/IP models
- Configure and inspect network interfaces
- Understand IP addressing (IPv4 and IPv6)
- Use DNS tools (dig, nslookup, host)
- Transfer files with curl, wget, scp, and sftp
- Use basic SSH
- Troubleshoot network connectivity

---

## Table of Contents

1. [The OSI Model](#1-the-osi-model)
2. [TCP/IP Model and Protocols](#2-tcpip-model-and-protocols)
3. [IP Addressing](#3-ip-addressing)
4. [Network Interfaces](#4-network-interfaces)
5. [DNS](#5-dns)
6. [Routing](#6-routing)
7. [Network Testing Tools](#7-network-testing-tools)
8. [File Transfer Tools](#8-file-transfer-tools)
9. [SSH Basics](#9-ssh-basics)
10. [Network Monitoring](#10-network-monitoring)
11. [Practice Exercises](#11-practice-exercises)
12. [Key Takeaways](#12-key-takeaways)

---

## 1. The OSI Model

The **OSI (Open Systems Interconnection) Model** describes network communication in 7 layers:

```
OSI Model
┌─────┬──────────────────┬─────────────────────────────────────────┐
│  7  │  APPLICATION     │  HTTP, FTP, SSH, DNS, SMTP, IMAP        │
│     │                  │  User-facing protocols                  │
├─────┼──────────────────┼─────────────────────────────────────────┤
│  6  │  PRESENTATION    │  SSL/TLS, JPEG, MPEG, ASCII             │
│     │                  │  Encryption, compression, encoding      │
├─────┼──────────────────┼─────────────────────────────────────────┤
│  5  │  SESSION         │  NetBIOS, RPC, SQL sessions             │
│     │                  │  Session establishment and management   │
├─────┼──────────────────┼─────────────────────────────────────────┤
│  4  │  TRANSPORT       │  TCP, UDP, SCTP                         │
│     │                  │  Port numbers, end-to-end delivery      │
├─────┼──────────────────┼─────────────────────────────────────────┤
│  3  │  NETWORK         │  IP, ICMP, ARP, routing protocols       │
│     │                  │  IP addresses, routing                  │
├─────┼──────────────────┼─────────────────────────────────────────┤
│  2  │  DATA LINK       │  Ethernet, WiFi (802.11), MAC addresses │
│     │                  │  Switches, frames                       │
├─────┼──────────────────┼─────────────────────────────────────────┤
│  1  │  PHYSICAL        │  Cables, fiber, radio waves             │
│     │                  │  Bits over the physical medium          │
└─────┴──────────────────┴─────────────────────────────────────────┘

Memory trick: "Please Do Not Throw Sausage Pizza Away"
             Physical Data Network Transport Session Presentation Application
```

---

## 2. TCP/IP Model and Protocols

The **TCP/IP model** (practical internet model) has 4 layers:

```
TCP/IP Model              OSI Equivalent
┌──────────────────┐
│   APPLICATION    │ ←── OSI layers 5, 6, 7
│  HTTP, FTP, SSH  │
├──────────────────┤
│   TRANSPORT      │ ←── OSI layer 4
│    TCP / UDP     │
├──────────────────┤
│   INTERNET       │ ←── OSI layer 3
│     IP, ICMP     │
├──────────────────┤
│  NETWORK ACCESS  │ ←── OSI layers 1, 2
│ Ethernet, WiFi   │
└──────────────────┘
```

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | Best-effort |
| Ordering | In-order packets | No ordering |
| Error correction | Yes (retransmit) | No |
| Speed | Slower | Faster |
| Use cases | HTTP, SSH, FTP, email | DNS, video streaming, gaming |

### TCP Three-Way Handshake

```
Client                          Server
  │                                │
  │──── SYN (seq=100) ────────────►│
  │                                │
  │◄── SYN-ACK (seq=200,ack=101) ──│
  │                                │
  │──── ACK (ack=201) ────────────►│
  │                                │
  │    [Connection Established]    │
  │──── DATA ─────────────────────►│
```

### Common Ports

| Port | Protocol | Service |
|------|----------|---------|
| 20, 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet (insecure) |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 8080 | TCP | HTTP alternate |
| 27017 | TCP | MongoDB |

```bash
# View all well-known port assignments
cat /etc/services | head -30
grep "^ssh" /etc/services
```

---

## 3. IP Addressing

### IPv4 Addressing

IPv4 addresses are 32-bit numbers written in dotted decimal notation:

```
192.168.1.100
─┬──.─┬──.─.─┬──
 │    │      └── Host portion
 │    └── Network portion
 └── Class/range

Binary: 11000000.10101000.00000001.01100100
```

### IPv4 Address Classes and Ranges

| Class | Range | Default Mask | Use |
|-------|-------|-------------|-----|
| A | 1.0.0.0 – 126.255.255.255 | /8 | Large networks |
| B | 128.0.0.0 – 191.255.255.255 | /16 | Medium networks |
| C | 192.0.0.0 – 223.255.255.255 | /24 | Small networks |

**Private IP Ranges (RFC 1918):**

| Range | CIDR | Hosts |
|-------|------|-------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | ~16 million |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | ~1 million |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65,536 |

### CIDR Notation

```
192.168.1.0/24
─────────────┬
             └── 24 bits for network, 8 bits for hosts

/24 = 256 addresses (254 usable, 2 reserved: network + broadcast)
/25 = 128 addresses
/26 = 64 addresses
/27 = 32 addresses
/28 = 16 addresses
/29 = 8 addresses (6 usable)
/30 = 4 addresses (2 usable)
/32 = single host
```

### IPv6

IPv6 addresses are 128-bit:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
──── Full form ────

2001:db8:85a3::8a2e:370:7334
──── Compressed (leading zeros removed, :: for groups of zeros) ────

::1              = loopback (127.0.0.1 equivalent)
fe80::/10        = link-local addresses
fc00::/7         = unique local addresses (private, like RFC 1918)
2000::/3         = global unicast (public internet)
```

```bash
# View IPv6 addresses
ip -6 addr show
ip addr show | grep inet6
ping6 ::1                    # IPv6 loopback
ping6 google.com             # IPv6 ping
```

---

## 4. Network Interfaces

### Viewing Network Interfaces

```bash
# Modern way (ip command)
ip addr                      # show all interfaces
ip addr show                 # same
ip addr show eth0            # specific interface
ip -brief addr               # brief format
ip link                      # link layer info
ip link show eth0

# Legacy way (ifconfig — still common)
ifconfig                     # all interfaces
ifconfig eth0                # specific interface
sudo apt install net-tools   # if not installed

# Output explained:
$ ip addr show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::211:22ff:fe33:4455/64 scope link
#   ^     ^               ^
#   proto address         prefix length
```

### Network Interface Naming

Modern Linux uses **predictable network interface names**:
- `eth0`, `eth1` — old style (still used in VMs)
- `ens3`, `ens33` — Ethernet
- `enp3s0` — Ethernet (bus/slot/function)
- `wlan0`, `wlp2s0` — WiFi
- `lo` — loopback (127.0.0.1)
- `docker0` — Docker bridge
- `virbr0` — KVM/QEMU bridge

### Configuring Network Interfaces

```bash
# Temporary (lost on reboot or interface restart)

# Add IP address
sudo ip addr add 192.168.1.200/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.200/24 dev eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

### Persistent Configuration (Netplan — Ubuntu 17.10+)

```bash
# View current netplan config
cat /etc/netplan/*.yaml

# Example DHCP configuration
cat << 'EOF' | sudo tee /etc/netplan/01-network.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
EOF

# Example static IP configuration
cat << 'EOF' | sudo tee /etc/netplan/01-network.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
EOF

# Apply configuration
sudo netplan apply
```

---

## 5. DNS

**DNS (Domain Name System)** translates domain names to IP addresses.

### How DNS Works

```
Your Browser              DNS Resolver            Root Server
     │                         │                      │
     │── "What is example.com?" ──────────────────────►│
     │                         │                      │
     │                         │◄── "Ask .com server" ─│
     │                         │
     │                    .com Server
     │                         │──► "Ask example.com NS"
     │                         │
     │                  example.com NS
     │                         │──► "It's 93.184.216.34"
     │                         │
     │◄── "example.com = 93.184.216.34" ──────────────│
     │                         │
     └── Connect to 93.184.216.34
```

### DNS Configuration

```bash
# View DNS configuration
cat /etc/resolv.conf           # DNS servers
# nameserver 8.8.8.8
# nameserver 8.8.4.4
# search example.com           # domain search list

# View /etc/hosts (local DNS overrides)
cat /etc/hosts
# 127.0.0.1 localhost
# 127.0.1.1 hostname
# 192.168.1.50 myserver.local myserver

# Add local DNS entry
echo "192.168.1.50 devserver.local" | sudo tee -a /etc/hosts
```

### DNS Query Tools

```bash
# nslookup — simple DNS lookup
nslookup google.com           # forward lookup
nslookup 8.8.8.8              # reverse lookup
nslookup google.com 8.8.8.8   # use specific DNS server

# host — simple DNS lookup
host google.com               # forward lookup
host -t A google.com          # A records only
host -t MX gmail.com          # MX records (mail servers)
host -t NS google.com         # Name servers
host 8.8.8.8                  # reverse lookup

# dig — detailed DNS query (most powerful)
dig google.com                        # default (A record)
dig google.com A                      # A record
dig google.com MX                     # mail servers
dig google.com NS                     # name servers
dig google.com ANY                    # all records (limited)
dig @8.8.8.8 google.com              # query specific DNS server
dig +short google.com                 # just the answer
dig -x 8.8.8.8                        # reverse lookup

# Trace DNS delegation
dig +trace google.com

# DNS record types
# A      — IPv4 address
# AAAA   — IPv6 address
# CNAME  — Canonical name (alias)
# MX     — Mail exchanger
# NS     — Name server
# PTR    — Pointer (reverse DNS)
# SOA    — Start of Authority
# TXT    — Text records (SPF, DKIM, domain verification)
# SRV    — Service records
```

---

## 6. Routing

```bash
# View routing table
ip route                      # modern way
ip route show
route -n                      # legacy (net-tools)
netstat -rn                   # legacy

# Output:
$ ip route
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.100 metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100

# Add a route
sudo ip route add 10.0.0.0/8 via 192.168.1.254 dev eth0

# Add default gateway
sudo ip route add default via 192.168.1.1 dev eth0

# Delete a route
sudo ip route del 10.0.0.0/8

# Flush all routes
sudo ip route flush cache
```

---

## 7. Network Testing Tools

### `ping` — Test Connectivity

```bash
ping google.com                # ping until stopped (Ctrl+C)
ping -c 4 google.com           # send 4 packets only
ping -i 0.5 google.com         # 0.5 second interval
ping -s 1000 google.com        # 1000 byte packets (test MTU)
ping -W 2 google.com           # 2 second timeout

# IPv6 ping
ping6 ::1
ping6 google.com

# Ping with output:
# PING google.com: 64 bytes of data
# 64 bytes from 142.250.80.46: icmp_seq=1 ttl=116 time=15.3 ms
# --- google.com ping statistics ---
# 4 packets transmitted, 4 received, 0% packet loss
# rtt min/avg/max/mdev = 14.8/15.3/15.9/0.4 ms
```

### `traceroute` / `tracepath`

```bash
sudo apt install traceroute
traceroute google.com          # trace route to destination
traceroute -n google.com       # no DNS lookups (faster)
traceroute -T google.com       # use TCP SYN (bypass firewalls)
traceroute -p 80 google.com    # use port 80

tracepath google.com           # simpler, no root needed
mtr google.com                 # continuous traceroute (install: sudo apt install mtr)
```

### `netstat` / `ss` — Socket Statistics

```bash
# Modern: ss (socket statistics)
ss                             # all sockets
ss -t                          # TCP sockets
ss -u                          # UDP sockets
ss -l                          # listening sockets
ss -tuln                       # listening TCP+UDP, no DNS
ss -tlnp                       # listening TCP with process info
ss -s                          # summary statistics
ss -o state established        # established connections

# Find what's listening on port 80
ss -tlnp | grep :80

# Legacy: netstat
netstat -tuln                  # listening ports
netstat -tunap                 # all connections with PIDs
netstat -rn                    # routing table
netstat -s                     # protocol statistics
```

### `curl` — Transfer Data from URLs

```bash
# Basic HTTP request
curl http://example.com              # GET request, output to terminal
curl -s http://example.com           # silent (no progress)
curl -o output.html http://example.com  # save to file

# HTTP methods
curl -X GET http://api.example.com/users
curl -X POST http://api.example.com/users \
     -H "Content-Type: application/json" \
     -d '{"name":"Alice","email":"alice@example.com"}'
curl -X DELETE http://api.example.com/users/1

# Headers
curl -H "Authorization: Bearer TOKEN" http://api.example.com
curl -H "User-Agent: MyApp/1.0" http://example.com
curl -I http://example.com           # HEAD request (headers only)

# Authentication
curl -u username:password http://example.com
curl -u username http://example.com  # prompt for password

# HTTPS
curl https://example.com             # works with valid cert
curl -k https://self-signed.com      # ignore cert errors (INSECURE!)

# Follow redirects
curl -L http://github.com            # follow redirects

# Timing
curl -w "%{time_total}" -o /dev/null -s http://example.com

# Download file
curl -O http://example.com/file.tar.gz    # keep remote filename
curl -o myfile.tar.gz http://example.com/file.tar.gz  # specify name
curl -C - -O http://example.com/bigfile.zip  # resume download
```

### `wget` — Download Files

```bash
wget http://example.com/file.tar.gz          # download file
wget -q http://example.com/file.tar.gz       # quiet mode
wget -O output.html http://example.com       # specify output name
wget -c http://example.com/bigfile.zip       # continue/resume
wget -r http://example.com/                  # recursive download
wget -r --level=2 http://example.com/        # 2 levels deep
wget --mirror http://example.com/            # mirror a website
wget -i urls.txt                             # download from list
wget --limit-rate=1m http://example.com/file # rate limit

# Check if URL is accessible
wget --spider http://example.com 2>&1 | grep -i "200 OK"
```

---

## 8. File Transfer Tools

### `scp` — Secure Copy

```bash
# Copy local file to remote
scp file.txt user@server:/home/user/

# Copy remote file to local
scp user@server:/var/log/app.log ./

# Copy directory recursively
scp -r mydir/ user@server:/home/user/

# Specify port
scp -P 2222 file.txt user@server:/tmp/

# Use identity file
scp -i ~/.ssh/mykey.pem file.txt ec2-user@server:/tmp/
```

### `sftp` — Interactive File Transfer

```bash
sftp user@server
# sftp commands:
# ls              — list remote directory
# lls             — list local directory
# cd /remote/dir  — change remote directory
# lcd /local/dir  — change local directory
# get file.txt    — download file
# put file.txt    — upload file
# mkdir newdir    — create remote directory
# rm file.txt     — delete remote file
# quit / exit     — disconnect
```

### `rsync` — Efficient Sync

```bash
# Sync local directory to remote
rsync -avz mydir/ user@server:/home/user/mydir/

# Sync remote to local
rsync -avz user@server:/home/user/mydir/ ./mydir/

# Options:
# -a = archive (recursive + preserve permissions/timestamps)
# -v = verbose
# -z = compress during transfer
# -n or --dry-run = test without actually syncing
# -P = show progress + partial transfers
# --delete = delete files on dest that don't exist on source
# --exclude = exclude patterns
# --include = include patterns

# Backup example
rsync -avz --backup --backup-dir=/backup/$(date +%Y%m%d) \
      /home/alice/ user@backup-server:/backups/alice/
```

---

## 9. SSH Basics

**SSH (Secure Shell)** provides encrypted remote access.

```bash
# Connect to remote server
ssh user@server               # connect with password
ssh -p 2222 user@server       # non-standard port
ssh -i ~/.ssh/mykey user@server  # use specific key

# Run remote command without interactive session
ssh user@server "uptime"
ssh user@server "df -h && free -h"

# Enable SSH key authentication
# 1. Generate key pair (client side)
ssh-keygen -t ed25519 -C "your@email.com"
# Creates: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)

# 2. Copy public key to server
ssh-copy-id user@server
# Or manually:
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# 3. Verify key permissions
ls -la ~/.ssh/
# drwx------ .ssh/
# -rw------- id_ed25519
# -rw-r--r-- id_ed25519.pub
# -rw------- authorized_keys

# SSH config file (~/.ssh/config)
cat << 'EOF' >> ~/.ssh/config
Host myserver
    HostName 192.168.1.100
    User alice
    Port 22
    IdentityFile ~/.ssh/mykey

Host bastion
    HostName bastion.example.com
    User admin
    ForwardAgent yes
EOF

# Now can connect with:
ssh myserver
```

---

## 10. Network Monitoring

```bash
# Monitor network traffic in real-time
sudo apt install iftop
sudo iftop                    # per-connection traffic

sudo apt install nethogs
sudo nethogs                  # per-process traffic

sudo apt install nload
sudo nload eth0               # per-interface bandwidth

# Capture packets (Wireshark CLI)
sudo apt install tcpdump
sudo tcpdump -i eth0          # capture on eth0
sudo tcpdump -i eth0 port 80  # only HTTP traffic
sudo tcpdump -i eth0 host google.com  # traffic to/from google
sudo tcpdump -i eth0 -w capture.pcap  # save to file
sudo tcpdump -r capture.pcap          # read saved file

# Check open ports on local machine
sudo ss -tlnp
sudo nmap -sV localhost       # install nmap first
```

---

## 11. Practice Exercises

### Exercise 10.1 — Network Discovery

```bash
# 1. Check your IP address
ip addr show | grep inet

# 2. Check your default gateway
ip route show default

# 3. Check your DNS servers
cat /etc/resolv.conf

# 4. Ping the gateway
GATEWAY=$(ip route show default | awk '{print $3}')
ping -c 4 $GATEWAY

# 5. Ping Google
ping -c 4 8.8.8.8

# 6. Ping Google by name
ping -c 4 google.com

# If step 5 works but 6 fails: DNS problem
```

### Exercise 10.2 — DNS Investigation

```bash
# 1. Find the IP of google.com
dig +short google.com

# 2. Find the mail servers for gmail.com
dig +short gmail.com MX

# 3. Find the name servers for google.com
dig +short google.com NS

# 4. Do a reverse DNS lookup
dig -x 8.8.8.8 +short

# 5. Query TXT records (for SPF etc)
dig google.com TXT +short
```

### Exercise 10.3 — Port Scanning and Monitoring

```bash
# 1. See what ports are listening
ss -tlnp

# 2. Install and start nginx
sudo apt install -y nginx
sudo systemctl start nginx

# 3. Verify port 80 is listening
ss -tlnp | grep :80

# 4. Test with curl
curl -s http://localhost | head -5

# 5. Check with netstat
netstat -tlnp | grep nginx

# 6. Stop nginx
sudo systemctl stop nginx
```

---

## 12. Key Takeaways

- **OSI model**: 7 layers from Physical to Application
- **TCP** = reliable, ordered; **UDP** = fast, unreliable; know common ports
- **IPv4**: 32-bit; **IPv6**: 128-bit; understand CIDR notation (`/24` = 256 addresses)
- **`ip addr`** shows interfaces; **`ip route`** shows routing table
- **`dig`** is the most powerful DNS tool; `host` and `nslookup` for quick lookups
- **`ss -tlnp`** shows listening services (modern `netstat`)
- **`curl`** for HTTP requests; **`wget`** for downloading; **`rsync`** for syncing
- **SSH key authentication** is more secure than passwords
- **`tcpdump`** captures raw packets; `wireshark` provides GUI analysis

---

## Module 2 Complete! 🎉

You now understand user management, process control, package management, and networking basics.

---

## Next Module

➡️ [Module 3: Advanced — Shell Scripting Fundamentals](../Module3_Advanced/11_Shell_Scripting_Fundamentals.md)

---

*Module 2 · Lesson 4 of 4 | [Course Index](../INDEX.md)*
