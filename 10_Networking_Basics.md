# 10 - Networking Basics

**Difficulty:** Intermediate  
**Time Estimate:** 90–120 minutes  
**Prerequisites:** Lessons 01–09 (especially Terminal Basics and Package Management)

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Describe the OSI and TCP/IP models and where common protocols fit
- Interpret IP addresses, CIDR notation, and subnets
- Explain the roles of DNS, DHCP, gateway, and netmask
- Inspect and configure network interfaces with `ip` and `nmcli`
- Use diagnostic tools: `ping`, `traceroute`, `ss`, `netstat`, `dig`, `nslookup`
- Connect to remote systems securely with SSH and manage keys
- Transfer files with `scp`, `sftp`, `curl`, and `wget`
- Configure basic firewall rules with `ufw`
- Edit key network configuration files

---

## 1. The OSI Model

The **OSI (Open Systems Interconnection)** model provides a conceptual framework for how network protocols interact in seven layers.

```
┌──────────────────────────────────────────────────────────────┐
│                    OSI MODEL LAYERS                          │
├───────┬──────────────┬───────────────────────────────────────┤
│ Layer │ Name         │ Description & Examples                │
├───────┼──────────────┼───────────────────────────────────────┤
│   7   │ Application  │ User-facing protocols                 │
│       │              │ HTTP, HTTPS, FTP, SSH, DNS, SMTP      │
├───────┼──────────────┼───────────────────────────────────────┤
│   6   │ Presentation │ Data encoding, encryption, compression│
│       │              │ TLS/SSL, JPEG, ASCII, Unicode         │
├───────┼──────────────┼───────────────────────────────────────┤
│   5   │ Session      │ Managing connections between hosts    │
│       │              │ NetBIOS, RPC, SQL sessions            │
├───────┼──────────────┼───────────────────────────────────────┤
│   4   │ Transport    │ End-to-end delivery, ports            │
│       │              │ TCP (reliable), UDP (fast)            │
├───────┼──────────────┼───────────────────────────────────────┤
│   3   │ Network      │ Logical addressing and routing        │
│       │              │ IP, ICMP, OSPF, BGP                   │
├───────┼──────────────┼───────────────────────────────────────┤
│   2   │ Data Link    │ Physical addressing (MAC), framing    │
│       │              │ Ethernet, Wi-Fi (802.11), ARP         │
├───────┼──────────────┼───────────────────────────────────────┤
│   1   │ Physical     │ Cables, signals, bits on the wire     │
│       │              │ Ethernet cables, fiber, radio waves   │
└───────┴──────────────┴───────────────────────────────────────┘

Memory aid: "All People Seem To Need Data Processing"
```

### TCP/IP Model (Practical 4-Layer Version)

```
OSI Layer(s)    TCP/IP Layer       Examples
────────────    ────────────────   ──────────────────────────
7, 6, 5         Application        HTTP, HTTPS, DNS, SSH, FTP
4               Transport          TCP, UDP
3               Internet           IP, ICMP, ARP
2, 1            Network Access     Ethernet, Wi-Fi, drivers
```

---

## 2. IP Addresses

### IPv4

IPv4 addresses are 32-bit numbers written as four decimal octets separated by dots.

```
192.168.1.100
│   │   │  │
│   │   │  └── host portion
│   │   └───── host portion
│   └───────── network portion
└───────────── network portion
```

**Address classes (legacy, largely replaced by CIDR):**

| Class | Range              | Default Mask  | Use               |
|-------|--------------------|---------------|-------------------|
| A     | 1.0.0.0–126.x.x.x  | 255.0.0.0     | Large networks    |
| B     | 128.0.0.0–191.x.x.x| 255.255.0.0   | Medium networks   |
| C     | 192.0.0.0–223.x.x.x| 255.255.255.0 | Small networks    |

**Private (RFC 1918) address ranges — not routed on the public internet:**

| Range                    | CIDR           | Common Use             |
|--------------------------|----------------|------------------------|
| 10.0.0.0–10.255.255.255  | 10.0.0.0/8     | Large internal networks|
| 172.16.0.0–172.31.255.255| 172.16.0.0/12  | Medium networks        |
| 192.168.0.0–192.168.255.255| 192.168.0.0/16| Home/small office      |
| 127.0.0.0–127.255.255.255| 127.0.0.0/8    | Loopback (localhost)   |

### CIDR Notation and Subnetting

**CIDR (Classless Inter-Domain Routing)** uses a `/prefix` to indicate how many bits are the network portion.

```
192.168.1.0/24

Network:   192.168.1.0
Mask:      255.255.255.0   (24 bits = 1s)
Hosts:     192.168.1.1 – 192.168.1.254  (254 usable)
Broadcast: 192.168.1.255

Prefix  Mask              Hosts       Use case
/8      255.0.0.0         16,777,214  Very large network
/16     255.255.0.0       65,534      Large network
/24     255.255.255.0     254         Typical LAN
/25     255.255.255.128   126         Half subnet
/26     255.255.255.192   62          Quarter subnet
/27     255.255.255.224   30          Small subnet
/28     255.255.255.240   14          Very small subnet
/30     255.255.255.252   2           Point-to-point link
/32     255.255.255.255   1           Single host (loopback, routes)
```

### IPv6

IPv6 addresses are 128-bit numbers written as eight groups of four hexadecimal digits:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
2001:db8:85a3::8a2e:370:7334   (shortened: consecutive zeros → ::)

::1                   = loopback (equivalent to 127.0.0.1)
fe80::/10             = link-local addresses (auto-configured)
```

---

## 3. Core Networking Concepts

### DNS (Domain Name System)

Translates human-readable names (e.g., `google.com`) into IP addresses.

```
Your browser               DNS Resolver           Authoritative
requests google.com  ──►  (e.g., 8.8.8.8)  ──►  DNS Server
                          checks cache         for google.com
                    ◄──  returns 142.250.x.x  ◄──
```

### DHCP (Dynamic Host Configuration Protocol)

Automatically assigns IP address, subnet mask, default gateway, and DNS servers to network clients.

### Gateway

The router that connects your local network to other networks (typically the internet). All traffic destined outside your subnet is sent to the gateway.

### Key Files

```bash
# Local hostname-to-IP mapping (checked before DNS)
cat /etc/hosts

# DNS server configuration
cat /etc/resolv.conf

# Network interface configuration (Debian/Ubuntu traditional)
cat /etc/network/interfaces

# systemd-resolved stub resolver
cat /etc/systemd/resolved.conf
resolvectl status
```

```
# /etc/hosts format:
127.0.0.1       localhost
127.0.1.1       myhostname
192.168.1.50    fileserver fileserver.local
```

---

## 4. Network Interfaces

### Viewing Interfaces

```bash
# Modern: ip command (iproute2)
ip addr                      # Show all interfaces and IPs
ip addr show eth0            # Show specific interface
ip link                      # Show interface link status
ip link show eth0
ip route                     # Show routing table
ip route show default        # Show default gateway

# Legacy: ifconfig (net-tools, still useful)
ifconfig                     # Show active interfaces
ifconfig -a                  # Show ALL interfaces (including down)
ifconfig eth0

# Quick: hostname
hostname -I                  # Show all IP addresses
hostname -f                  # Show FQDN
```

### Understanding ip addr Output

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86300sec preferred_lft 86300sec
    inet6 fe80::211:22ff:fe33:4455/64 scope link
       valid_lft forever preferred_lft forever

│  │                                       │   └── state: UP/DOWN
│  │                              └── MTU: maximum transmission unit
│  └── Interface name
└── Interface index
```

### Configuring Interfaces (Temporary)

```bash
# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Assign an IP address temporarily
sudo ip addr add 192.168.1.200/24 dev eth0

# Remove an IP address
sudo ip addr del 192.168.1.200/24 dev eth0

# Add a default route
sudo ip route add default via 192.168.1.1

# Set an IP with ifconfig (legacy)
sudo ifconfig eth0 192.168.1.200 netmask 255.255.255.0
```

---

## 5. Network Topology Visualization

```
                    ┌──────────────────────────────────────────┐
                    │              INTERNET                     │
                    └──────────────────┬───────────────────────┘
                                       │
                          ┌────────────┴────────────┐
                          │         ROUTER          │
                          │    (Default Gateway)    │
                          │    192.168.1.1          │
                          └───┬──────────┬──────────┘
                              │          │
              ┌───────────────┴─────┐    └───────────────────┐
              │      SWITCH         │                         │
              └──┬───────┬──────────┘                        │
                 │       │                                    │
     ┌───────────┴──┐  ┌─┴───────────┐            ┌──────────┴──────┐
     │   Desktop    │  │   Server    │            │   Laptop (WiFi) │
     │192.168.1.100 │  │192.168.1.50 │            │  192.168.1.150  │
     └──────────────┘  └─────────────┘            └─────────────────┘

     All devices share:
     Network: 192.168.1.0/24
     Gateway: 192.168.1.1
     DNS:     8.8.8.8, 1.1.1.1 (or router's IP)
```

---

## 6. Network Diagnostic Tools

### ping

```bash
# Basic connectivity test
ping google.com
ping 8.8.8.8

# Limit number of packets
ping -c 4 google.com

# Set interval between packets (seconds)
ping -i 0.5 google.com

# Set packet size
ping -s 1024 google.com

# Ping IPv6
ping6 ::1
ping -6 google.com

# Show timestamp
ping -D google.com

# Flood ping (requires root, use carefully)
sudo ping -f 192.168.1.1
```

### traceroute / tracepath

```bash
# Trace the path packets take
traceroute google.com
sudo apt install traceroute    # Install if needed

# Use ICMP instead of UDP
sudo traceroute -I google.com

# Use TCP
sudo traceroute -T -p 443 google.com

# No DNS resolution (faster)
traceroute -n google.com

# Simpler alternative (no root needed)
tracepath google.com
```

### ss and netstat — Socket Statistics

```bash
# ss (modern replacement for netstat)
ss                             # All sockets
ss -t                          # TCP sockets
ss -u                          # UDP sockets
ss -l                          # Listening sockets only
ss -p                          # Show process using socket
ss -n                          # No DNS/service name resolution
ss -4                          # IPv4 only
ss -6                          # IPv6 only

# Common combinations
ss -tlnp                       # TCP listening sockets with PIDs
ss -tunlp                      # TCP+UDP listening with PIDs
ss -tnp state established      # Established TCP with PIDs

# Filter by port
ss -tlnp sport = :22
ss -tlnp sport = :80 or sport = :443

# netstat (legacy, install net-tools)
sudo apt install net-tools
netstat -tlnp                  # TCP listening
netstat -an                    # All connections, numeric
netstat -rn                    # Routing table
netstat -i                     # Interface statistics
netstat -s                     # Protocol statistics
```

### DNS Tools

```bash
# nslookup (basic DNS lookup)
nslookup google.com
nslookup google.com 8.8.8.8   # Use specific DNS server
nslookup -type=MX gmail.com   # MX records (mail servers)
nslookup -type=NS google.com  # Name servers

# dig (more detailed DNS tool)
dig google.com
dig google.com A               # A record (IPv4)
dig google.com AAAA            # IPv6 address
dig google.com MX              # Mail exchanger
dig google.com NS              # Name servers
dig google.com TXT             # TXT records (SPF, DKIM, etc.)
dig google.com ANY             # All records

# Short output
dig +short google.com

# Use specific DNS server
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com

# Reverse DNS lookup (IP to hostname)
dig -x 8.8.8.8
dig +short -x 8.8.8.8

# Trace DNS resolution chain
dig +trace google.com

# host (simpler alternative)
host google.com
host -t MX gmail.com
host 8.8.8.8                   # Reverse lookup
```

---

## 7. curl and wget

### curl

```bash
# Download a file
curl -O https://example.com/file.zip

# Download and save with custom name
curl -o myfile.zip https://example.com/file.zip

# Follow redirects
curl -L https://example.com/redirect

# Show response headers
curl -I https://example.com
curl -v https://example.com    # Verbose (headers + body)

# Send GET request and show response
curl https://api.example.com/data

# Send POST request with JSON
curl -X POST https://api.example.com/data \
     -H "Content-Type: application/json" \
     -d '{"key": "value"}'

# Send POST with form data
curl -X POST https://example.com/form \
     -d "username=alice&password=secret"

# Add custom headers
curl -H "Authorization: Bearer TOKEN" https://api.example.com

# Download with progress bar
curl --progress-bar -O https://example.com/bigfile.iso

# Limit download speed
curl --limit-rate 1M -O https://example.com/file.iso

# Check HTTP status code
curl -s -o /dev/null -w "%{http_code}" https://example.com

# Follow redirects and show final URL
curl -Ls -o /dev/null -w "%{url_effective}" https://example.com
```

### wget

```bash
# Download a file
wget https://example.com/file.zip

# Download to specific directory
wget -P /tmp https://example.com/file.zip

# Resume an interrupted download
wget -c https://example.com/bigfile.iso

# Download in background
wget -b https://example.com/bigfile.iso

# Quiet mode (no output)
wget -q https://example.com/file.zip

# Download with authentication
wget --user=alice --password=secret https://example.com/file.zip

# Mirror a website
wget --mirror --convert-links --no-parent https://example.com/

# Download multiple URLs from a file
wget -i urls.txt

# Retry on failure
wget --tries=5 https://example.com/file.zip
```

---

## 8. SSH — Secure Shell

SSH provides encrypted remote terminal access, file transfers, and tunneling.

### Connecting

```bash
# Basic connection
ssh alice@192.168.1.50
ssh alice@server.example.com

# Specify port (default: 22)
ssh -p 2222 alice@server.example.com

# Connect with specific key
ssh -i ~/.ssh/id_ed25519 alice@server.example.com

# Run a command without interactive session
ssh alice@server "ls -la /home/alice"
ssh alice@server "sudo systemctl status nginx"

# Enable X11 forwarding (GUI apps over SSH)
ssh -X alice@server

# SSH with verbose output (debugging)
ssh -v alice@server
ssh -vvv alice@server   # More verbose
```

### SSH Key Management

```bash
# Generate an SSH key pair (ed25519 is modern and secure)
ssh-keygen -t ed25519 -C "alice@laptop"
# Creates: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)

# Generate RSA key (4096-bit for compatibility)
ssh-keygen -t rsa -b 4096 -C "alice@laptop"

# Generate key with custom filename
ssh-keygen -t ed25519 -f ~/.ssh/server_key -C "server access key"

# Copy public key to remote server (enables passwordless login)
ssh-copy-id alice@server.example.com
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server.example.com
ssh-copy-id -p 2222 alice@server.example.com

# Manual method (if ssh-copy-id unavailable)
cat ~/.ssh/id_ed25519.pub | ssh alice@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# View your public key
cat ~/.ssh/id_ed25519.pub

# Add key to ssh-agent (avoids passphrase prompts during session)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l    # List loaded keys
```

### SSH Config File (~/.ssh/config)

```bash
# Create/edit SSH client config
nano ~/.ssh/config
chmod 600 ~/.ssh/config

# Example ~/.ssh/config:
Host webserver
    HostName 192.168.1.50
    User alice
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host jumphost
    HostName bastion.example.com
    User alice
    Port 2222

Host internal-server
    HostName 10.0.0.100
    User bob
    ProxyJump jumphost

# With config, connect simply as:
ssh webserver
ssh internal-server   # Automatically tunnels through jumphost
```

### File Transfer with scp and sftp

```bash
# scp: Secure Copy
# Copy local file to remote
scp file.txt alice@server:/home/alice/
scp -P 2222 file.txt alice@server:/tmp/

# Copy remote file to local
scp alice@server:/home/alice/file.txt /local/path/

# Copy directory recursively
scp -r /local/dir alice@server:/home/alice/

# sftp: Interactive secure FTP
sftp alice@server
# SFTP commands:
# ls, cd, pwd (remote)
# lls, lcd, lpwd (local)
# get file.txt (download)
# put file.txt (upload)
# mget *.txt (multiple download)
# mput *.txt (multiple upload)
# mkdir, rm, rmdir
# exit
```

---

## 9. Network Manager

Modern Ubuntu/Debian systems use **NetworkManager** for network configuration.

### nmcli — Command Line Interface

```bash
# Show connection status
nmcli
nmcli general status

# List all connections
nmcli connection show
nmcli con show

# Show active connections
nmcli con show --active

# Show device status
nmcli device status
nmcli dev status

# Connect to a Wi-Fi network
nmcli dev wifi list
nmcli dev wifi connect "MyNetwork" password "mypassword"

# Disconnect from a network
nmcli con down "MyNetwork"

# Connect/reconnect to saved connection
nmcli con up "MyNetwork"

# Show IP info for a connection
nmcli con show "Wired connection 1"

# Add a static IP connection
nmcli con add type ethernet \
  con-name "static-eth0" \
  ifname eth0 \
  ipv4.method manual \
  ipv4.addresses "192.168.1.100/24" \
  ipv4.gateway "192.168.1.1" \
  ipv4.dns "8.8.8.8,1.1.1.1"

# Set DNS servers on existing connection
nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8 1.1.1.1"

# Reload connection
nmcli con reload
nmcli con up "Wired connection 1"

# Enable/disable networking
nmcli networking off
nmcli networking on

# Enable/disable Wi-Fi
nmcli radio wifi off
nmcli radio wifi on
```

### nmtui — Text User Interface

```bash
# Launch the curses-based TUI
nmtui
# Navigate with arrow keys and Tab
# Options: Edit a connection, Activate a connection, Set system hostname
```

---

## 10. Firewall with ufw

**ufw (Uncomplicated Firewall)** provides a simplified interface to `iptables`.

```bash
# Check firewall status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# Enable/disable firewall
sudo ufw enable
sudo ufw disable

# Default policies (deny all incoming, allow all outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow specific services by name
sudo ufw allow ssh          # Port 22
sudo ufw allow http         # Port 80
sudo ufw allow https        # Port 443

# Allow/deny specific ports
sudo ufw allow 8080
sudo ufw allow 8080/tcp
sudo ufw deny 25/tcp

# Allow a port range
sudo ufw allow 6000:6007/tcp

# Allow from a specific IP
sudo ufw allow from 192.168.1.100
sudo ufw allow from 192.168.1.100 to any port 22

# Allow from a subnet
sudo ufw allow from 192.168.1.0/24

# Deny a specific IP
sudo ufw deny from 203.0.113.100

# Delete a rule by number
sudo ufw delete 3

# Delete a rule by specification
sudo ufw delete allow 8080

# Reset all rules
sudo ufw reset

# Application profiles
sudo ufw app list
sudo ufw allow "Nginx Full"
sudo ufw allow "OpenSSH"
```

---

## 11. Common Ports and Services

| Port | Protocol | Service                    |
|------|----------|----------------------------|
| 20   | TCP      | FTP data                   |
| 21   | TCP      | FTP control                |
| 22   | TCP      | SSH                        |
| 23   | TCP      | Telnet (unencrypted, avoid)|
| 25   | TCP      | SMTP (outbound email)      |
| 53   | TCP/UDP  | DNS                        |
| 67/68| UDP      | DHCP server/client         |
| 80   | TCP      | HTTP                       |
| 110  | TCP      | POP3 (email retrieval)     |
| 143  | TCP      | IMAP (email retrieval)     |
| 443  | TCP      | HTTPS                      |
| 465  | TCP      | SMTPS (email over TLS)     |
| 993  | TCP      | IMAPS                      |
| 3306 | TCP      | MySQL/MariaDB              |
| 5432 | TCP      | PostgreSQL                 |
| 6379 | TCP      | Redis                      |
| 8080 | TCP      | HTTP alternative           |
| 8443 | TCP      | HTTPS alternative          |
| 27017| TCP      | MongoDB                    |

```bash
# Find what's listening on a port
ss -tlnp | grep :80
sudo lsof -i :80

# Check if a port is open on a remote host
nc -zv server.example.com 22
nc -zv server.example.com 80 443

# Port scan with nmap (install first)
sudo apt install nmap
nmap -p 22,80,443 192.168.1.50
nmap -sV 192.168.1.50            # Service version detection
```

---

## Common Mistakes

1. **SSH key permissions too open** — `~/.ssh/` must be `700`, `~/.ssh/authorized_keys` must be `600`. SSH refuses to use keys if permissions are too permissive.
2. **Forgetting `nmcli con up` after changes** — Modifying a connection with `nmcli con mod` doesn't apply changes until you reconnect.
3. **Confusing `ip addr` (temporary) vs NetworkManager (persistent)** — Changes made with `ip addr add` are lost on reboot. Use `nmcli` or edit config files for persistence.
4. **Blocking SSH before enabling ufw** — Run `sudo ufw allow ssh` BEFORE `sudo ufw enable`, or you'll lock yourself out of remote sessions.
5. **Not using `dig +trace`** — When debugging DNS issues, `+trace` reveals exactly which server is returning the wrong answer.
6. **Using `ping` as the only connectivity test** — ICMP can be blocked by firewalls. Use `curl` or `nc` to test application-layer connectivity.
7. **Storing SSH private keys without passphrases on shared systems** — Always use a passphrase on private keys, and use `ssh-agent` to avoid repeated prompts.

---

## Pro Tips

- **`mtr`** — Combines `ping` and `traceroute` in real-time: `sudo apt install mtr && mtr google.com`. Shows per-hop packet loss and latency continuously.
- **SSH multiplexing** — Reuse SSH connections for speed: add `ControlMaster auto` / `ControlPath ~/.ssh/cm-%r@%h:%p` / `ControlPersist 10m` to `~/.ssh/config`.
- **`ss -s`** — Quick socket summary (total TCP, UDP, listen counts). Great for a fast system health check.
- **`nc` (netcat) as a simple port test**: `nc -zv hostname port` — tests TCP connectivity without needing `nmap`.
- **`curl -w "@format_file" URL`** — You can create a custom format file to measure DNS resolution time, TCP connect time, TLS handshake time, etc.
- **`/etc/hosts` for local dev** — Add `127.0.0.1 myapp.local` to test local web apps with real hostnames.
- **`resolvectl query domain.com`** — On systemd systems, shows which DNS server answered and the TTL.

---

## Practice Exercises

**Exercise 1 — Interface Inspection**  
Run `ip addr` and `ip route`. Identify your primary network interface, its IP address, subnet mask (in CIDR and decimal), and default gateway. What is your system's MAC address?

**Exercise 2 — DNS Investigation**  
Use `dig` to find the A records, MX records, and NS records for a domain (e.g., `github.com`). Then use `dig +trace github.com` to see the full resolution chain.

**Exercise 3 — Connection Monitoring**  
Run `ss -tlnp` and identify all listening services. Then run `ss -tnp state established` to see active connections. What processes are currently holding open connections?

**Exercise 4 — SSH Key Setup**  
Generate a new ED25519 SSH key pair with a passphrase. Add an entry to `~/.ssh/config` for a local server (or another machine you have access to). Test the connection.

**Exercise 5 — curl API Testing**  
Use `curl` to make a GET request to `https://httpbin.org/get`. Then make a POST request to `https://httpbin.org/post` with JSON data `{"name": "linux", "lesson": 10}`. Inspect the responses.

**Exercise 6 — Firewall Configuration**  
Check if `ufw` is installed and enabled. Set default policies to deny incoming and allow outgoing. Allow SSH and HTTP. View the numbered rules. Delete the HTTP rule by number. Verify with `sudo ufw status`.

**Exercise 7 — Network Diagnosis**  
Run `ping -c 4 8.8.8.8` (tests IP connectivity), then `ping -c 4 google.com` (tests DNS + IP), then `traceroute google.com` (tests routing path). What do the differences tell you about where issues might be?

**Exercise 8 — Port Investigation**  
Use `ss -tlnp` to find what service is listening on port 53 (DNS). Use `nc -zv localhost 22` to test if SSH port is open. Use `curl -I http://localhost` to check if a web server is running.

---

## Key Takeaways

- The **OSI model** (7 layers) and **TCP/IP model** (4 layers) describe how network communication is structured
- IPv4 addresses are 32-bit; CIDR notation (`/24`) indicates the network prefix length
- Private ranges (`10.x`, `172.16-31.x`, `192.168.x`) are not routed on the public internet
- **`ip addr`**, **`ip route`**, and **`nmcli`** are the modern tools for interface and connection management
- **`ss -tlnp`** is the go-to command for listing open ports and their associated processes
- **`dig`** and **`nslookup`** query DNS; `dig +trace` shows the full resolution chain
- SSH key authentication (ed25519) is more secure than password auth; always protect private keys with a passphrase
- **`ufw`** provides an easy interface for firewall rules — always allow SSH before enabling!
- Always configure firewall rules **before** enabling the firewall to avoid locking yourself out

---

## Next Lesson Preview

**Lesson 11 — Text Processing and Shell Scripting**  
You've built a solid foundation in system administration. Next, we'll harness the power of the command line for automation: `grep`, `sed`, `awk`, pipes, redirects, and writing your first real Bash scripts with loops, conditionals, and functions.
