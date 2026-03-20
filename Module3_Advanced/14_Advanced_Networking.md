# 14 — Advanced Networking

> **Module 3 · Lesson 4** | Difficulty: ★★★★☆ Advanced | Time: ~90 min

---

## Learning Objectives

- Configure advanced networking features
- Set up VLANs, bridges, and bonds
- Configure VPN (OpenVPN/WireGuard)
- Implement network namespaces
- Use advanced nftables/iptables rules
- Configure proxy and port forwarding
- Understand network performance tuning

---

## 1. Network Bonding and Bridging

### Network Bonding (Link Aggregation)

```bash
sudo apt install ifenslave

# Create bond configuration
cat << 'EOF' | sudo tee /etc/netplan/02-bond.yaml
network:
  version: 2
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: active-backup      # or: balance-rr, 802.3ad, etc.
        primary: eth0
        mii-monitor-interval: 100
      dhcp4: true
EOF

sudo netplan apply

# Bonding modes:
# mode=0 (balance-rr)     — round-robin, load balancing
# mode=1 (active-backup)  — failover only
# mode=2 (balance-xor)    — XOR load balancing
# mode=4 (802.3ad)        — LACP dynamic link aggregation
# mode=6 (balance-alb)    — adaptive load balancing
```

### Network Bridging

```bash
# Create a bridge (for virtual machines)
cat << 'EOF' | sudo tee /etc/netplan/03-bridge.yaml
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
ip addr show br0
```

---

## 2. VLANs

```bash
sudo apt install vlan

# Create VLAN interface
sudo ip link add link eth0 name eth0.100 type vlan id 100
sudo ip addr add 192.168.100.1/24 dev eth0.100
sudo ip link set eth0.100 up

# Persistent VLAN configuration
cat << 'EOF' | sudo tee /etc/netplan/04-vlan.yaml
network:
  version: 2
  vlans:
    vlan10:
      id: 10
      link: eth0
      addresses:
        - 10.0.10.1/24
    vlan20:
      id: 20
      link: eth0
      addresses:
        - 10.0.20.1/24
EOF

sudo netplan apply
```

---

## 3. VPN Configuration

### WireGuard (Modern, Simple, Fast)

```bash
sudo apt install wireguard

# Generate key pairs
wg genkey | tee /etc/wireguard/private.key | wg pubkey > /etc/wireguard/public.key
chmod 600 /etc/wireguard/private.key

PRIVATE_KEY=$(cat /etc/wireguard/private.key)
PUBLIC_KEY=$(cat /etc/wireguard/public.key)

# Server configuration
cat << EOF | sudo tee /etc/wireguard/wg0.conf
[Interface]
PrivateKey = $PRIVATE_KEY
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client 1
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32
EOF

sudo systemctl enable --now wg-quick@wg0
sudo wg show
```

### OpenVPN

```bash
sudo apt install openvpn easy-rsa

# Initialize PKI
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-dh
./easyrsa gen-req server nopass
./easyrsa sign-req server server
openvpn --genkey secret ta.key

# Client configuration
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

---

## 4. Port Forwarding and NAT

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
# Permanent: echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.d/99-net.conf

# NAT (masquerade outgoing traffic)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding (forward external port to internal service)
# Forward port 8080 → internal server 192.168.1.100:80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT \
    --to-destination 192.168.1.100:80

# SSH tunneling
# Forward local port 8080 to remote port 80
ssh -L 8080:remotehost:80 user@jumphost

# Reverse tunnel (expose local service through remote server)
ssh -R 9090:localhost:22 user@public-server
# Now: ssh -p 9090 user@public-server → connects to your local machine

# SOCKS5 proxy via SSH
ssh -D 1080 user@server    # creates SOCKS5 proxy on localhost:1080
```

---

## 5. Network Namespaces

Network namespaces provide isolated network stacks (used by Docker/containers).

```bash
# Create namespace
sudo ip netns add myns

# List namespaces
ip netns list

# Run command in namespace
sudo ip netns exec myns ip addr
sudo ip netns exec myns bash

# Create veth pair (virtual ethernet cable)
sudo ip link add veth0 type veth peer name veth1

# Move one end into namespace
sudo ip link set veth1 netns myns

# Configure interfaces
sudo ip addr add 10.1.1.1/24 dev veth0
sudo ip link set veth0 up
sudo ip netns exec myns ip addr add 10.1.1.2/24 dev veth1
sudo ip netns exec myns ip link set veth1 up

# Test communication
ping 10.1.1.2                               # from host
sudo ip netns exec myns ping 10.1.1.1       # from namespace

# Clean up
sudo ip netns delete myns
```

---

## 6. Network Troubleshooting Toolkit

```bash
# Comprehensive troubleshooting steps:

# Step 1: Check interface is up
ip link show
ip addr show

# Step 2: Check IP configuration
ip addr
ip route

# Step 3: Test local network
ping -c 4 <gateway>

# Step 4: Test DNS
dig google.com
cat /etc/resolv.conf

# Step 5: Test internet
ping -c 4 8.8.8.8
curl -v http://google.com

# Step 6: Check listening services
ss -tlnp

# Step 7: Capture traffic
sudo tcpdump -i eth0 -w /tmp/capture.pcap
tcpdump -r /tmp/capture.pcap

# Check MTU issues
ping -M do -s 1472 8.8.8.8          # test with maximum frame size
# If fails, MTU is the issue

# Check firewall
sudo iptables -L -n -v
sudo ufw status verbose

# Check routing
traceroute 8.8.8.8
mtr --report 8.8.8.8
```

---

## 7. Network Performance Tuning

```bash
# Kernel network parameters
cat << 'EOF' | sudo tee /etc/sysctl.d/99-network-tuning.conf
# Increase TCP buffer sizes
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864

# Enable TCP fast open
net.ipv4.tcp_fastopen = 3

# Increase connection backlog
net.core.somaxconn = 65535

# Enable TCP timestamps
net.ipv4.tcp_timestamps = 1

# Reduce TIME_WAIT
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_tw_reuse = 1
EOF

sudo sysctl -p /etc/sysctl.d/99-network-tuning.conf

# Test bandwidth
sudo apt install iperf3
iperf3 -s                    # server
iperf3 -c server_ip -t 30   # client (30-second test)
```

---

## Practice Exercises

### Exercise 14.1 — SSH Tunneling

```bash
# Set up an SSH tunnel to access a remote web service
# 1. Connect to a server with SSH
# 2. Forward local port 8888 to remote port 80
ssh -L 8888:localhost:80 user@server

# 3. Access in a browser: http://localhost:8888
# 4. Also test with curl:
curl http://localhost:8888
```

### Exercise 14.2 — Network Namespace

```bash
# Create an isolated network namespace for testing
sudo ip netns add testns

# Create veth pair
sudo ip link add veth-host type veth peer name veth-ns

# Configure
sudo ip link set veth-ns netns testns
sudo ip addr add 172.16.0.1/30 dev veth-host
sudo ip link set veth-host up
sudo ip netns exec testns ip addr add 172.16.0.2/30 dev veth-ns
sudo ip netns exec testns ip link set veth-ns up
sudo ip netns exec testns ip link set lo up

# Verify isolation
sudo ip netns exec testns ip addr
sudo ip netns exec testns ping -c 2 172.16.0.1

# Clean up
sudo ip netns delete testns
sudo ip link delete veth-host
```

---

## Key Takeaways

- **Bonding** combines NICs for redundancy or bandwidth
- **Bridges** connect network interfaces at Layer 2 (used for VMs)
- **WireGuard** is the modern VPN choice: simple, fast, and secure
- **SSH tunneling** provides encrypted port forwarding without a full VPN
- **Network namespaces** are the foundation of container networking
- **IP forwarding + NAT** allows a Linux box to act as a router
- **tcpdump** and **Wireshark** are essential for packet-level troubleshooting

---

➡️ [15 — Security Hardening](15_Security_Hardening.md)

*Module 3 · Lesson 4 of 5 | [Course Index](../INDEX.md)*
