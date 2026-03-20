# 14. Advanced Networking

> **Difficulty:** Advanced | **Time Estimate:** 4–5 hours | **Prerequisites:** Lessons 1–13

---

## Learning Objectives

By the end of this lesson, you will be able to:

- Configure network interfaces with the `ip` command suite
- Understand TCP/IP fundamentals including the three-way handshake
- Build packet-filtering rules with `iptables`, `nftables`, and UFW
- Capture and analyse network traffic with `tcpdump`
- Configure DNS and DHCP servers
- Set up SSH tunnels and port forwarding
- Test network performance with `iperf3`

---

## 1. The ip Command Suite

The `ip` command (from the `iproute2` package) replaces legacy tools like `ifconfig`, `route`, and `arp`.

### Interface Management

```bash
# Show all interfaces
ip link show
ip link show eth0         # Specific interface

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Set MTU
sudo ip link set eth0 mtu 9000   # Jumbo frames

# Show interface statistics
ip -s link show eth0
```

### Address Management

```bash
# Show IP addresses
ip addr show
ip addr show eth0             # Specific interface
ip -4 addr show               # IPv4 only
ip -6 addr show               # IPv6 only

# Add/remove IP address
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Add secondary address
sudo ip addr add 10.0.0.5/8 dev eth0 label eth0:1

# Flush all addresses from interface
sudo ip addr flush dev eth0
```

### Routing

```bash
# View routing table
ip route show
ip route show table main
ip route show table all   # All routing tables

# Add/delete routes
sudo ip route add 192.168.2.0/24 via 192.168.1.1
sudo ip route add 192.168.2.0/24 dev eth1        # Direct route
sudo ip route del 192.168.2.0/24

# Default gateway
sudo ip route add default via 192.168.1.1
sudo ip route del default via 192.168.1.1
sudo ip route replace default via 192.168.1.254   # Change gateway

# Find route to a destination
ip route get 8.8.8.8

# Flush route cache
sudo ip route flush cache
```

---

## 2. Static vs Dynamic IP Configuration

### Netplan (Ubuntu 18.04+)

```bash
# View current config
cat /etc/netplan/*.yaml

# Static IP example
cat << 'EOF' | sudo tee /etc/netplan/01-static.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
        search: [example.com]
EOF

sudo netplan generate   # Validate config
sudo netplan apply      # Apply changes

# DHCP example
cat << 'EOF' | sudo tee /etc/netplan/01-dhcp.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false
EOF
```

### NetworkManager (Desktop/RHEL)

```bash
nmcli device status                        # Device overview
nmcli connection show                      # All connections
nmcli connection show --active             # Active connections

# Configure static IP
nmcli connection modify "Wired connection 1" \
    ipv4.method manual \
    ipv4.addresses "192.168.1.100/24" \
    ipv4.gateway "192.168.1.1" \
    ipv4.dns "8.8.8.8,1.1.1.1"

nmcli connection up "Wired connection 1"

# Switch to DHCP
nmcli connection modify "Wired connection 1" ipv4.method auto
```

---

## 3. Network Bonding and Bridging

```bash
# --- Bonding (Link Aggregation) ---
# Load bonding module
sudo modprobe bonding

# Create bond using netplan
cat << 'EOF' | sudo tee /etc/netplan/02-bond.yaml
network:
  version: 2
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: active-backup    # Options: balance-rr, active-backup, 802.3ad
        mii-monitor-interval: 100
      dhcp4: true
EOF

# --- Bridging (for VMs/containers) ---
cat << 'EOF' | sudo tee /etc/netplan/02-bridge.yaml
network:
  version: 2
  bridges:
    br0:
      interfaces: [eth0]
      dhcp4: true
      parameters:
        stp: false
        forward-delay: 0
EOF
sudo netplan apply
```

---

## 4. TCP/IP Deep Dive

### Three-Way Handshake

```
Client                            Server
  │                                  │
  │──────── SYN (seq=x) ────────────►│  Client initiates
  │                                  │
  │◄─── SYN-ACK (seq=y, ack=x+1) ───│  Server acknowledges + syncs
  │                                  │
  │──────── ACK (ack=y+1) ──────────►│  Client acknowledges
  │                                  │
  │◄═══════ DATA TRANSFER ══════════►│  Established
  │                                  │
  │──────── FIN ────────────────────►│  Client initiates close
  │◄──────── FIN-ACK ────────────────│
  │──────── ACK ────────────────────►│  Connection closed
```

### TCP States

```bash
# View socket states
ss -tnp                    # TCP connections with process info
ss -tnlp                   # Listening sockets
ss -tnp state established  # Only established
ss -s                      # Summary statistics
ss -tnp 'dport = :80'      # Connections to port 80

# Netstat (older alternative)
netstat -tnlp              # Listening ports
netstat -an | grep ESTABLISHED | wc -l   # Count established connections
```

---

## 5. iptables

iptables organises rules into **tables** → **chains** → **rules**.

```
Tables:   filter  nat  mangle  raw  security
Chains:   INPUT  OUTPUT  FORWARD  PREROUTING  POSTROUTING

Packet flow:
  Incoming ──► PREROUTING ──► [routing] ──► INPUT ──► local process
                                       └──► FORWARD ──► POSTROUTING ──► outgoing
  Local process ──► OUTPUT ──► POSTROUTING ──► outgoing
```

```bash
# === Viewing Rules ===
sudo iptables -L -v -n          # filter table, verbose, numeric
sudo iptables -L -v -n --line-numbers   # Show rule numbers
sudo iptables -t nat -L -v -n   # NAT table

# === Basic Rules ===
# Allow established/related traffic
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow SSH from specific subnet
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT

# Drop all other INPUT
sudo iptables -A INPUT -j DROP

# === Managing Rules ===
sudo iptables -D INPUT 3                     # Delete rule #3
sudo iptables -I INPUT 1 -p icmp -j ACCEPT  # Insert at position 1
sudo iptables -F INPUT                       # Flush INPUT chain
sudo iptables -Z                             # Zero packet counters

# === NAT (Masquerade / Port Forwarding) ===
# Internet sharing (NAT masquerade)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1

# Port forwarding: external :8080 → internal 192.168.1.10:80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT \
    --to-destination 192.168.1.10:80

# === Persistence ===
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

---

## 6. nftables (Modern Replacement)

```bash
# List all rules
sudo nft list ruleset

# Create a basic firewall
sudo nft add table inet filter
sudo nft add chain inet filter input \
    { type filter hook input priority 0\; policy drop\; }
sudo nft add chain inet filter output \
    { type filter hook output priority 0\; policy accept\; }
sudo nft add chain inet filter forward \
    { type filter hook forward priority 0\; policy drop\; }

# Add rules
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport { 80, 443 } accept

# Save and load
sudo nft list ruleset > /etc/nftables.conf
sudo systemctl enable --now nftables
```

---

## 7. UFW (Uncomplicated Firewall)

```bash
# Status and enable
sudo ufw status verbose
sudo ufw enable
sudo ufw disable
sudo ufw reset               # Remove all rules

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow/deny by port
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443
sudo ufw allow 8080:8090/tcp          # Port range
sudo ufw deny 3306                    # Block MySQL from outside

# Allow by service name
sudo ufw allow OpenSSH
sudo ufw allow "Nginx Full"
sudo ufw app list                     # Available app profiles

# Allow from specific source
sudo ufw allow from 192.168.1.0/24
sudo ufw allow from 10.0.0.5 to any port 22

# Delete rules
sudo ufw status numbered
sudo ufw delete 3                     # By number
sudo ufw delete allow 80/tcp          # By spec

# Logging
sudo ufw logging on
sudo ufw logging medium               # off, low, medium, high, full
tail -f /var/log/ufw.log

# Rate limiting (SSH brute force protection)
sudo ufw limit ssh
```

---

## 8. VPN Concepts

### WireGuard (Modern VPN)

```bash
# Install WireGuard
sudo apt install wireguard

# Generate key pair
wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
chmod 600 /etc/wireguard/privatekey

# Server config: /etc/wireguard/wg0.conf
cat << 'EOF' | sudo tee /etc/wireguard/wg0.conf
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32
EOF

sudo systemctl enable --now wg-quick@wg0
sudo wg show
```

---

## 9. Network Namespaces

```bash
# Create network namespace (isolated network stack)
sudo ip netns add myns

# List namespaces
ip netns list

# Run command in namespace
sudo ip netns exec myns ip link show
sudo ip netns exec myns ping 8.8.8.8   # No connectivity yet

# Create veth pair and connect to namespace
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns myns

# Configure addresses
sudo ip addr add 10.1.0.1/24 dev veth0
sudo ip link set veth0 up
sudo ip netns exec myns ip addr add 10.1.0.2/24 dev veth1
sudo ip netns exec myns ip link set veth1 up
sudo ip netns exec myns ip link set lo up

# Test connectivity
ping -c 2 10.1.0.2

# Delete namespace
sudo ip netns del myns
```

---

## 10. Packet Analysis

### tcpdump

```bash
# Capture all traffic on eth0
sudo tcpdump -i eth0

# Capture with more detail (-v) and no hostname resolution (-n)
sudo tcpdump -i eth0 -nn -v

# Capture to file for Wireshark analysis
sudo tcpdump -i eth0 -w /tmp/capture.pcap

# Read capture file
sudo tcpdump -r /tmp/capture.pcap

# Filters (BPF syntax)
sudo tcpdump -i eth0 port 80                      # HTTP only
sudo tcpdump -i eth0 host 192.168.1.50            # Specific host
sudo tcpdump -i eth0 src 192.168.1.50             # From source
sudo tcpdump -i eth0 'tcp and port 443'           # HTTPS
sudo tcpdump -i eth0 'not port 22'                # Exclude SSH
sudo tcpdump -i eth0 'icmp'                       # Ping only
sudo tcpdump -i eth0 -nn -A port 80 | head -50   # Show ASCII content
sudo tcpdump -i eth0 -c 100 -w capture.pcap      # Capture 100 packets

# SYN packets only (new connections)
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'
```

---

## 11. Network Performance Testing

```bash
# iperf3 — bandwidth testing
# Server side
iperf3 -s               # Listen on port 5201

# Client side
iperf3 -c 192.168.1.100           # TCP test (default)
iperf3 -c 192.168.1.100 -t 30     # 30-second test
iperf3 -c 192.168.1.100 -u -b 1G  # UDP test at 1 Gbps
iperf3 -c 192.168.1.100 -P 4      # 4 parallel streams
iperf3 -c 192.168.1.100 -R        # Reverse (server sends, client receives)
iperf3 -c 192.168.1.100 --json    # JSON output

# Other tools
nmap -sn 192.168.1.0/24           # Ping sweep / host discovery
nmap -sV -p 1-1000 192.168.1.100  # Version scan top 1000 ports

mtr 8.8.8.8                       # Continuous traceroute
traceroute 8.8.8.8                # One-time path trace
ping -c 4 -s 1400 8.8.8.8        # Ping with large packets (MTU check)
```

---

## 12. DNS Server Basics (BIND)

```bash
# Install BIND
sudo apt install bind9 bind9-dnsutils

# Main config files
/etc/bind/named.conf                # Master config
/etc/bind/named.conf.options        # Global options
/etc/bind/named.conf.local          # Zone definitions

# Add a forward zone
cat << 'EOF' | sudo tee -a /etc/bind/named.conf.local
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
};
EOF

# Create zone file
sudo mkdir -p /etc/bind/zones
cat << 'EOF' | sudo tee /etc/bind/zones/db.example.com
$TTL    604800
@   IN  SOA ns1.example.com. admin.example.com. (
            2024010101  ; Serial
            3600        ; Refresh
            900         ; Retry
            604800      ; Expire
            86400 )     ; Negative Cache TTL

@   IN  NS  ns1.example.com.
ns1 IN  A   192.168.1.10
@   IN  A   192.168.1.10
www IN  A   192.168.1.10
mail IN  A  192.168.1.20
@   IN  MX  10 mail.example.com.
EOF

# Check and reload
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/zones/db.example.com
sudo systemctl restart bind9

# Test DNS
dig @192.168.1.10 www.example.com
nslookup www.example.com 192.168.1.10
```

---

## 13. DHCP Server

```bash
# Install isc-dhcp-server
sudo apt install isc-dhcp-server

# Configure: /etc/dhcp/dhcpd.conf
cat << 'EOF' | sudo tee /etc/dhcp/dhcpd.conf
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 1.1.1.1;
    option domain-name "example.com";
}

# Static binding (reserve IP for MAC)
host myprinter {
    hardware ethernet aa:bb:cc:dd:ee:ff;
    fixed-address 192.168.1.25;
}
EOF

sudo systemctl enable --now isc-dhcp-server
# View leases
cat /var/lib/dhcp/dhcpd.leases
```

---

## 14. SSH Tunneling and Port Forwarding

```bash
# === Local Port Forwarding ===
# Access remote service as if it were local
# ssh -L [local_port]:[remote_host]:[remote_port] [ssh_server]
ssh -L 8080:internal-server:80 bastion.example.com
# Now: http://localhost:8080 → bastion → internal-server:80

# Keep-alive tunnel (no command, just tunnel)
ssh -fNL 5432:db.internal:5432 user@bastion
# -f: background  -N: no command  -L: local forward

# === Remote Port Forwarding ===
# Expose local service to remote server
# ssh -R [remote_port]:[local_host]:[local_port] [ssh_server]
ssh -R 2222:localhost:22 user@public-server.com
# Now: someone ssh to public-server:2222 → reaches your localhost:22

# === Dynamic Port Forwarding (SOCKS proxy) ===
ssh -D 1080 -fN user@proxy-server.com
# Configure browser/app to use SOCKS5 proxy at localhost:1080

# === Jump Hosts (ProxyJump) ===
ssh -J user@bastion user@internal-server
# Or in ~/.ssh/config:
cat << 'EOF' >> ~/.ssh/config
Host internal
    HostName internal-server
    User admin
    ProxyJump bastion.example.com
EOF

# X11 Forwarding (run GUI apps over SSH)
ssh -X user@remote-server
# Then run: firefox &
```

---

## ASCII Diagram: Packet Journey Through iptables

```
 Incoming packet
       │
       ▼
  PREROUTING (nat)          ← DNAT happens here
       │
       ▼
  [Routing decision]
    /         \
   /           \
  ▼             ▼
INPUT         FORWARD (filter)
(filter)         │
  │              ▼
  ▼           POSTROUTING (nat)  ← SNAT/MASQUERADE here
Local process    │
  │              ▼
  └──► OUTPUT → Network
      (filter)
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| `iptables -F` flushes all rules | Locks you out if default is DROP | Set default to ACCEPT before flushing |
| Forgetting `ip_forward=1` for routing | NAT masquerade does nothing | `sysctl -w net.ipv4.ip_forward=1` |
| UFW rules in wrong order | Later ALLOW can't override earlier DENY | UFW uses first-match; order matters |
| Not saving iptables rules | Lost on reboot | Use `iptables-persistent` or `nftables` service |
| Incorrect subnet mask in dhcpd.conf | Clients in wrong range | Always verify with `ipcalc` |
| Running tcpdump without `-n` | DNS lookups slow down capture | Always use `-nn` in production |

---

## Pro Tips

- **`ss -s`** gives a quick summary of socket statistics — great for spotting connection floods
- **`tcpdump -i any`** captures on all interfaces — useful when you're unsure which interface to watch
- **`nmap -sV --script vuln`** runs vulnerability scripts against open ports — know your exposure
- **WireGuard** is ~4x faster and simpler to configure than OpenVPN; use it for new VPN deployments
- **SSH `ControlMaster`** config reuses existing connections, eliminating re-authentication overhead
- For persistent `ip` routes/addresses, use Netplan or NetworkManager — `ip` commands are lost on reboot

---

## Practice Exercises

1. **Static IP** — Configure a static IP address, gateway, and DNS on a network interface using Netplan. Verify with `ip addr`, `ip route`, and `ping 8.8.8.8`.

2. **iptables Firewall** — Write an iptables rule set that allows SSH, HTTP, HTTPS, and established traffic, and drops everything else. Make it persistent.

3. **UFW Profile** — Create a UFW application profile for a custom app on port 9090. Enable rate limiting on SSH. Export and inspect the running rules.

4. **tcpdump Analysis** — Capture 200 packets of HTTP traffic, save to a `.pcap` file, then use `tcpdump -r` to filter only packets containing "GET" requests.

5. **SSH Tunnel** — Set up a local port forward that routes `localhost:3307` to a remote MySQL server on port 3306 through an SSH bastion host.

6. **iperf3 Benchmark** — Run an iperf3 server, test bandwidth with 1, 2, and 4 parallel streams, and compare the results in JSON format.

7. **Network Namespace** — Create a network namespace, a veth pair, configure IP addresses on both ends, and verify ping works between them.

8. **DNS Resolution** — Install BIND, configure a local forward zone for `lab.local`, add A records for 3 hosts, and test with `dig`.

---

## Key Takeaways

- The `ip` command suite is the modern, comprehensive replacement for `ifconfig`, `route`, and `arp`
- **iptables** processes rules in order — first match wins; always allow ESTABLISHED,RELATED traffic first
- **UFW** is a user-friendly iptables frontend; use `ufw limit` for SSH brute-force protection
- The **three-way handshake** (SYN → SYN-ACK → ACK) establishes every TCP connection
- **SSH tunneling** (local, remote, dynamic) is a powerful and often underutilised feature
- `tcpdump` with BPF filters is the fastest way to capture and analyse specific traffic types
- Network changes via `ip` commands are **ephemeral** — use Netplan/NM for persistence

---

## Next Lesson Preview

**Lesson 15: Security Hardening** — We'll harden SSH, configure fail2ban to block brute-force attacks, implement AppArmor mandatory access controls, set up file integrity monitoring with AIDE, manage SSL/TLS certificates, use GPG for encryption, and run comprehensive security audits with Lynis and rkhunter.
