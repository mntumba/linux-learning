# Lesson 23: Firewall and Network Security

> **Security Advanced · Lesson 23** | Difficulty: ★★★★☆ Advanced | Time: ~140 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Write complex iptables rules and understand the Netfilter hook model
- Configure nftables as a modern, unified firewall solution
- Manage firewalld zones and services for dynamic firewall management
- Deploy a WireGuard VPN and understand its cryptographic design
- Configure an OpenVPN server with certificate-based authentication
- Design a DMZ architecture to isolate public-facing services

---

## Prerequisites

- Lesson 21 (Cryptography Fundamentals) and Lesson 22 (IDS) or equivalent
- Solid networking knowledge: routing, NAT, TCP/IP, iptables basics
- Root access on a test system (VM or container recommended)

---

## Key Concepts

### 1. Netfilter and the iptables Architecture

iptables is a userspace tool that configures the Linux kernel's **Netfilter** packet filtering framework.

```
Incoming packet
     │
     ▼
[PREROUTING] ──► Route decision ──► [INPUT] ──► Local process
                       │
                       ▼
                  [FORWARD] ──► [POSTROUTING] ──► Out interface
                                      ▲
                               Local process ──► [OUTPUT]
```

**Tables and their purposes**:

| Table | Purpose | Chains |
|-------|---------|--------|
| `filter` | Allow/deny packets (default) | INPUT, FORWARD, OUTPUT |
| `nat` | Address/port translation | PREROUTING, POSTROUTING, OUTPUT |
| `mangle` | Modify packet headers (QoS, TTL) | All 5 chains |
| `raw` | Bypass connection tracking | PREROUTING, OUTPUT |
| `security` | SELinux MAC labels | INPUT, FORWARD, OUTPUT |

### 2. iptables Deep Dive

```bash
# View all rules with line numbers and packet/byte counters
sudo iptables -L -v -n --line-numbers
sudo iptables -t nat -L -v -n --line-numbers

# Default DROP policy (whitelist approach – most secure)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow established/related traffic (stateful inspection)
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback interface
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow SSH from a specific subnet only
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -m conntrack \
  --ctstate NEW -j ACCEPT

# Allow HTTP and HTTPS from anywhere
sudo iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT

# Rate-limit new connections (anti-DDoS)
sudo iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW \
  -m limit --limit 60/min --limit-burst 20 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j DROP

# Log and drop invalid packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID \
  -j LOG --log-prefix "INVALID: " --log-level 4
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# NAT – masquerade outbound traffic (for a router/gateway)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Port forwarding (DNAT) – redirect external port 80 to internal server
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
  -j DNAT --to-destination 192.168.1.10:80
sudo iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 80 \
  -m conntrack --ctstate NEW -j ACCEPT

# Save rules (Debian/Ubuntu)
sudo apt install -y iptables-persistent
sudo netfilter-persistent save

# Save rules (RHEL/CentOS)
sudo service iptables save
```

**Advanced iptables techniques**:

```bash
# Block a country using ipset (much faster than many individual rules)
sudo apt install -y ipset
sudo ipset create blocked_countries hash:net
sudo ipset add blocked_countries 1.0.0.0/8    # Example range

sudo iptables -A INPUT -m set --match-set blocked_countries src -j DROP

# String matching – block HTTP requests containing "wget" user-agent
sudo iptables -A INPUT -p tcp --dport 80 \
  -m string --string "User-Agent: Wget" --algo bm -j DROP

# Recent module – block SSH scanners
sudo iptables -A INPUT -p tcp --dport 22 -m recent --name SSH --set
sudo iptables -A INPUT -p tcp --dport 22 -m recent --name SSH \
  --rcheck --seconds 60 --hitcount 4 -j LOG --log-prefix "SSH-SCAN: "
sudo iptables -A INPUT -p tcp --dport 22 -m recent --name SSH \
  --rcheck --seconds 60 --hitcount 4 -j DROP
```

### 3. nftables – The Modern Replacement

nftables replaces iptables/ip6tables/arptables/ebtables with a unified framework.

```bash
# Install
sudo apt install -y nftables
sudo systemctl enable --now nftables

# View current ruleset
sudo nft list ruleset

# Flush all rules
sudo nft flush ruleset
```

A complete nftables configuration (`/etc/nftables.conf`):

```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    # Reusable sets
    set trusted_hosts {
        type ipv4_addr
        flags interval
        elements = { 192.168.1.0/24, 10.0.0.0/8 }
    }

    chain input {
        type filter hook input priority 0; policy drop;

        # Loopback
        iif lo accept

        # Connection tracking
        ct state established,related accept
        ct state invalid drop

        # ICMP (allow ping, but rate-limit)
        ip protocol icmp icmp type echo-request limit rate 10/second accept
        ip protocol icmp accept

        # SSH – only from trusted hosts
        tcp dport 22 ip saddr @trusted_hosts ct state new accept

        # Web traffic
        tcp dport { 80, 443 } ct state new accept

        # Log and drop everything else
        log prefix "nft-drop: " flags all counter drop
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}

table ip nat {
    chain prerouting {
        type nat hook prerouting priority -100;
        # DNAT example
        iif eth0 tcp dport 8080 dnat to 192.168.1.10:80
    }

    chain postrouting {
        type nat hook postrouting priority 100;
        oif eth0 masquerade
    }
}
```

```bash
# Apply the configuration
sudo nft -f /etc/nftables.conf

# Add a rule dynamically
sudo nft add rule inet filter input tcp dport 8443 ct state new accept

# Delete a rule (get handle first)
sudo nft list ruleset -a
sudo nft delete rule inet filter input handle 12
```

### 4. firewalld Zones

firewalld provides a dynamic, zone-based firewall management daemon. It is the default on RHEL/CentOS/Fedora.

```bash
# Install
sudo apt install -y firewalld
sudo systemctl enable --now firewalld

# List zones and their interfaces
sudo firewall-cmd --list-all-zones

# Get and change the default zone
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --set-default-zone=drop

# Assign an interface to a zone
sudo firewall-cmd --zone=public --change-interface=eth0 --permanent

# Add/remove services
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --zone=public --remove-service=http --permanent

# Add/remove specific ports
sudo firewall-cmd --zone=public --add-port=8443/tcp --permanent

# Rich rules (more expressive)
sudo firewall-cmd --zone=public --add-rich-rule=\
  'rule family="ipv4" source address="10.0.0.0/8" service name="ssh" accept' --permanent

# Rate limiting with rich rules
sudo firewall-cmd --zone=public --add-rich-rule=\
  'rule service name="http" accept limit value="100/m"' --permanent

# Reload to apply permanent changes
sudo firewall-cmd --reload

# Masquerade (NAT)
sudo firewall-cmd --zone=public --add-masquerade --permanent

# Port forwarding
sudo firewall-cmd --zone=public \
  --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.1.10 --permanent
```

### 5. WireGuard VPN

WireGuard is a modern, cryptographically opinionated VPN protocol built into the Linux kernel (5.6+).

**Design principles**: Uses Curve25519 (key exchange), ChaCha20-Poly1305 (encryption), BLAKE2s (hashing), SipHash24 (hashtable keys), HKDF (key derivation).

```bash
# Install
sudo apt install -y wireguard

# Generate server key pair
wg genkey | sudo tee /etc/wireguard/server_private.key | \
  wg pubkey | sudo tee /etc/wireguard/server_public.key
sudo chmod 600 /etc/wireguard/server_private.key

# Generate client key pair
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

**Server configuration** (`/etc/wireguard/wg0.conf`):

```ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address    = 10.10.0.1/24
ListenPort = 51820
# Enable IP forwarding and NAT for client internet access
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey  = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.10.0.2/32
```

**Client configuration**:

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address    = 10.10.0.2/24
DNS        = 1.1.1.1

[Peer]
PublicKey  = <SERVER_PUBLIC_KEY>
Endpoint   = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0    # Route all traffic through VPN
PersistentKeepalive = 25
```

```bash
# Bring up the VPN
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

# Show VPN status
sudo wg show

# Add a peer without restarting
sudo wg set wg0 peer <PEER_PUBLIC_KEY> allowed-ips 10.10.0.3/32
```

### 6. OpenVPN

OpenVPN is a mature, certificate-based VPN using SSL/TLS.

```bash
# Install Easy-RSA for PKI management
sudo apt install -y openvpn easy-rsa

# Initialize PKI
make-cadir ~/openvpn-ca && cd ~/openvpn-ca
./easyrsa init-pki
./easyrsa build-ca nopass          # Create CA
./easyrsa gen-req server nopass    # Server cert request
./easyrsa sign-req server server   # Sign server cert
./easyrsa gen-dh                   # Diffie-Hellman params
openvpn --genkey secret ta.key     # TLS auth key

# Generate a client certificate
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

**Minimal OpenVPN server config** (`/etc/openvpn/server.conf`):

```
port 1194
proto udp
dev tun
ca   /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key  /etc/openvpn/easy-rsa/pki/private/server.key
dh   /etc/openvpn/easy-rsa/pki/dh.pem
tls-auth /etc/openvpn/ta.key 0
cipher AES-256-GCM
auth SHA256
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
user nobody
group nogroup
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
```

### 7. DMZ Architecture

A DMZ (demilitarized zone) isolates public-facing services from the internal network.

```
Internet
   │
[Firewall – Outer]  ← Only 80/443/25 allowed inbound
   │
  DMZ: Web, Mail, DNS servers
   │
[Firewall – Inner]  ← Only specific app traffic allowed inbound
   │
 Internal: App servers, Databases, Admin hosts
```

```bash
# Example iptables rules for a dual-homed DMZ host
# eth0 = external, eth1 = DMZ, eth2 = internal

# Allow internet → DMZ (web only)
sudo iptables -A FORWARD -i eth0 -o eth1 -p tcp --dport 443 \
  -m conntrack --ctstate NEW -j ACCEPT

# Allow DMZ → internal (app server only, specific port)
sudo iptables -A FORWARD -i eth1 -o eth2 -p tcp -d 10.0.0.5 --dport 8080 \
  -m conntrack --ctstate NEW -j ACCEPT

# Block DMZ → internal (everything else)
sudo iptables -A FORWARD -i eth1 -o eth2 -j LOG --log-prefix "DMZ-BLOCKED: "
sudo iptables -A FORWARD -i eth1 -o eth2 -j DROP
```

---

## Practical Exercises

### Exercise 1 – Stateful iptables Ruleset

```bash
# Flush and build a complete ruleset from scratch
sudo iptables -F; sudo iptables -X; sudo iptables -Z
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
sudo iptables -A INPUT -p icmp -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m limit --limit 3/min -j ACCEPT
sudo iptables -A INPUT -j LOG --log-prefix "DROPPED: "

# Verify
sudo iptables -L -v -n
```

### Exercise 2 – nftables Equivalent Ruleset

```bash
sudo nft flush ruleset
sudo tee /tmp/test.nft <<'EOF'
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo accept
        ct state established,related accept
        ct state invalid drop
        ip protocol icmp accept
        tcp dport 22 ct state new limit rate 3/minute accept
        log prefix "nft-drop: " counter drop
    }
    chain output { type filter hook output priority 0; policy accept; }
    chain forward { type filter hook forward priority 0; policy drop; }
}
EOF
sudo nft -f /tmp/test.nft
sudo nft list ruleset
```

### Exercise 3 – WireGuard Local Test Tunnel

```bash
# Create a loopback WireGuard tunnel for testing (no actual server needed)
sudo ip link add dev wg-test type wireguard
sudo ip addr add 10.99.0.1/24 dev wg-test

# Generate keys
wg genkey > /tmp/wg_priv && wg pubkey < /tmp/wg_priv > /tmp/wg_pub
sudo wg set wg-test private-key /tmp/wg_priv
sudo ip link set wg-test up

# Inspect
sudo wg show wg-test
sudo ip addr show wg-test

# Cleanup
sudo ip link del wg-test
rm -f /tmp/wg_priv /tmp/wg_pub
```

---

## Common Pitfalls

| Pitfall | Risk | Fix |
|---------|------|-----|
| Setting INPUT to DROP before allowing SSH | Lock yourself out | Add SSH ACCEPT rule before changing policy |
| Forgetting `-m conntrack` on stateless rules | Breaks established sessions | Always add `ESTABLISHED,RELATED` rule first |
| iptables rules lost after reboot | No persistent protection | Use `iptables-persistent` / `nftables.service` |
| firewalld and iptables running together | Rule conflicts | Choose one; disable the other |
| WireGuard server missing IP forwarding | Clients can't reach internet | `sysctl -w net.ipv4.ip_forward=1` + persist in `/etc/sysctl.d/` |
| OpenVPN using weak cipher (CBC mode) | Vulnerable to padding oracles | Use `AES-256-GCM` and `auth SHA256` |
| DMZ server allowed to reach all internal hosts | Compromise = full breach | Allowlist only required internal IPs/ports |

---

## Summary

- **iptables** remains widely used; understand tables, chains, and stateful connection tracking
- **nftables** is the future: unified syntax, better performance, atomic rule updates
- **firewalld** provides a user-friendly abstraction suitable for dynamic environments
- **WireGuard** is the modern VPN choice: simple, fast, auditable, kernel-native
- **OpenVPN** offers mature PKI-based auth and wider client platform support
- A **DMZ** limits blast radius when a public-facing server is compromised
- Always test firewall rules before applying them permanently to production systems

---

## Further Reading

- [Netfilter/iptables Project](https://www.netfilter.org/)
- [nftables Wiki](https://wiki.nftables.org/)
- [firewalld Documentation](https://firewalld.org/documentation/)
- [WireGuard Whitepaper](https://www.wireguard.com/papers/wireguard.pdf)
- [OpenVPN Hardening Guide](https://community.openvpn.net/openvpn/wiki/Hardening)
- *Linux Firewalls* by Steve Suehring (No Starch Press)
