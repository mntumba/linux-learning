# Appendix F: Networking Quick Reference

## TCP/IP Model

| Layer | Name | Protocols |
|-------|------|-----------|
| 4 | Application | HTTP, HTTPS, SSH, FTP, DNS, SMTP |
| 3 | Transport | TCP, UDP |
| 2 | Internet | IP, ICMP, ARP |
| 1 | Network Access | Ethernet, Wi-Fi |

## Common Ports

| Port | Protocol | Service |
|------|----------|---------|
| 20/21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67/68 | UDP | DHCP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5432 | TCP | PostgreSQL |
| 8080 | TCP | HTTP Alt |

## ip Command Cheatsheet

```bash
# Show interfaces
ip addr show
ip link show

# Add/remove IP
ip addr add 192.168.1.10/24 dev eth0
ip addr del 192.168.1.10/24 dev eth0

# Bring interface up/down
ip link set eth0 up
ip link set eth0 down

# Show routing table
ip route show

# Add default gateway
ip route add default via 192.168.1.1

# Add static route
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0

# Show ARP table
ip neigh show
```

## ss / netstat

```bash
# List all listening TCP sockets
ss -tlnp
netstat -tlnp

# List all connections
ss -anp
netstat -anp

# Show UDP listeners
ss -ulnp

# Filter by port
ss -tnp sport = :80
```

## tcpdump

```bash
# Capture on interface
tcpdump -i eth0

# Capture to file
tcpdump -i eth0 -w capture.pcap

# Read pcap file
tcpdump -r capture.pcap

# Filter by host
tcpdump -i eth0 host 192.168.1.1

# Filter by port
tcpdump -i eth0 port 80

# Filter by protocol
tcpdump -i eth0 tcp
tcpdump -i eth0 udp

# Verbose with no DNS resolution
tcpdump -i eth0 -nvv

# Capture HTTP GET requests
tcpdump -i eth0 -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

## nmap Cheatsheet

```bash
# Basic scan
nmap 192.168.1.1

# Scan range
nmap 192.168.1.0/24

# TCP SYN scan (stealth)
nmap -sS 192.168.1.1

# Service version detection
nmap -sV 192.168.1.1

# OS detection
nmap -O 192.168.1.1

# All common scripts + version + OS
nmap -A 192.168.1.1

# Scan specific ports
nmap -p 22,80,443 192.168.1.1

# Scan all ports
nmap -p- 192.168.1.1

# UDP scan
nmap -sU 192.168.1.1

# Aggressive scan
nmap -T4 -A -v 192.168.1.1

# Output to file
nmap -oN output.txt 192.168.1.1
nmap -oX output.xml 192.168.1.1
```

## DNS Tools

```bash
# Query DNS
dig example.com
dig example.com MX
dig @8.8.8.8 example.com

# Reverse lookup
dig -x 8.8.8.8

# Simple lookup
nslookup example.com

# Zone transfer attempt
dig axfr example.com @ns1.example.com
```

## curl / wget

```bash
# GET request
curl https://example.com
wget https://example.com

# POST with data
curl -X POST -d "user=admin&pass=secret" https://example.com/login

# With headers
curl -H "Authorization: Bearer TOKEN" https://api.example.com/data

# Follow redirects + save
curl -L -o file.zip https://example.com/file.zip

# Show headers only
curl -I https://example.com
```

## SSH

```bash
# Connect
ssh user@host
ssh -p 2222 user@host

# Key-based auth
ssh-keygen -t ed25519 -C "comment"
ssh-copy-id user@host

# Tunnel (local port forwarding)
ssh -L 8080:localhost:80 user@remote

# Tunnel (remote port forwarding)
ssh -R 9090:localhost:3000 user@remote

# SOCKS proxy
ssh -D 1080 user@remote

# Transfer files
scp file.txt user@host:/path/
rsync -avz /local/ user@host:/remote/
```
