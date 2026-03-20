# Lesson 22: Intrusion Detection Systems

> **Security Advanced · Lesson 22** | Difficulty: ★★★★☆ Advanced | Time: ~130 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Distinguish between IDS (detection) and IPS (prevention) and choose the right tool
- Install and configure Snort and Suricata for network traffic inspection
- Set up Fail2ban to automatically block brute-force attackers
- Deploy OSSEC/Wazuh for host-based intrusion detection (HIDS)
- Perform structured log analysis with `grep`, `awk`, `journalctl`, and the ELK stack
- Write custom detection rules for known attack patterns

---

## Prerequisites

- Lesson 21 (Cryptography Fundamentals) or equivalent
- Basic networking (TCP/IP, protocols, `tcpdump`)
- Linux service management (`systemctl`, `journalctl`)
- Root or sudo access on a test system

---

## Key Concepts

### 1. IDS vs. IPS vs. SIEM

| Type | Function | Placement | Action |
|------|----------|-----------|--------|
| **NIDS** (Network IDS) | Inspects network traffic | Span/tap port | Alert only |
| **NIPS** (Network IPS) | Inspects + blocks traffic | Inline | Alert + block |
| **HIDS** (Host IDS) | Monitors files, processes, logs | Agent on host | Alert + optional block |
| **SIEM** | Correlates events from many sources | Centralized | Alert + report |

**Detection methods**:
- **Signature-based** – matches known attack patterns (fast, low FP, misses zero-days)
- **Anomaly-based** – learns normal behaviour and flags deviations (catches novel attacks, higher FP)
- **Heuristic/Behavioural** – rules derived from expert knowledge

### 2. Snort

Snort is the most widely deployed open-source NIDS/NIPS.

```bash
# Install Snort on Debian/Ubuntu
sudo apt update && sudo apt install -y snort

# Test configuration
sudo snort -T -c /etc/snort/snort.conf

# Run in IDS mode (alert to console) on eth0
sudo snort -A console -q -i eth0 -c /etc/snort/snort.conf

# Run against a PCAP file (offline analysis)
sudo snort -r capture.pcap -c /etc/snort/snort.conf -l /var/log/snort/
```

**Writing a Snort rule**:

```
# Rule format:
# action proto src_ip src_port direction dst_ip dst_port (options)

# Detect ICMP ping (for testing)
alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)

# Detect SSH brute-force (>10 attempts in 60 seconds)
alert tcp any any -> $HOME_NET 22 (msg:"SSH Brute Force Attempt"; \
  flags:S; threshold:type threshold,track by_src,count 10,seconds 60; \
  sid:1000002; rev:1;)

# Detect HTTP SQL injection attempt
alert tcp any any -> $HTTP_SERVERS $HTTP_PORTS \
  (msg:"SQL Injection Attempt"; flow:to_server,established; \
  content:"union select"; nocase; http_uri; sid:1000003; rev:1;)
```

```bash
# Place custom rules in /etc/snort/rules/local.rules
echo 'alert icmp any any -> any any (msg:"ICMP Test"; sid:9000001; rev:1;)' \
  | sudo tee -a /etc/snort/rules/local.rules

# Verify rules load correctly
sudo snort -T -c /etc/snort/snort.conf
```

### 3. Suricata

Suricata is a modern, multi-threaded IDS/IPS/NSM that is largely Snort-rule compatible.

```bash
# Install Suricata
sudo apt install -y suricata

# Update rules (uses suricata-update, wrapper around Emerging Threats)
sudo suricata-update

# Run in IDS mode
sudo suricata -c /etc/suricata/suricata.yaml -i eth0

# Run in IPS mode (requires nfqueue)
sudo iptables -I FORWARD -j NFQUEUE --queue-num 0
sudo suricata -c /etc/suricata/suricata.yaml -q 0

# Test against a PCAP
sudo suricata -c /etc/suricata/suricata.yaml -r suspicious.pcap \
  -l /var/log/suricata/

# View the fast log (one alert per line)
tail -f /var/log/suricata/fast.log

# View structured JSON events
tail -f /var/log/suricata/eve.json | python3 -m json.tool | head -40
```

**Suricata YAML key settings** (`/etc/suricata/suricata.yaml`):

```yaml
# Threading – use all cores
threading:
  set-cpu-affinity: yes

# Enable AF-Packet for high-speed capture
af-packet:
  - interface: eth0
    threads: auto
    cluster-id: 99
    cluster-type: cluster_flow

# Log formats
outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: eve.json
      types:
        - alert
        - http
        - dns
        - tls
        - flow
```

### 4. Fail2ban

Fail2ban monitors log files and bans IPs that show malicious patterns.

```bash
# Install
sudo apt install -y fail2ban

# Create a local override (never edit jail.conf directly)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit jail.local
sudo nano /etc/fail2ban/jail.local
```

Key `jail.local` configuration:

```ini
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5
banaction = iptables-multiport
backend  = systemd

[sshd]
enabled  = true
port     = ssh
logpath  = %(sshd_log)s
maxretry = 3
bantime  = 24h

[nginx-http-auth]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log

[nginx-botsearch]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/access.log
maxretry = 2
```

```bash
# Start and enable Fail2ban
sudo systemctl enable --now fail2ban

# Check status of all jails
sudo fail2ban-client status

# Check a specific jail
sudo fail2ban-client status sshd

# Manually ban an IP
sudo fail2ban-client set sshd banip 192.168.1.100

# Manually unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Test a filter against a log file
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf
```

### 5. OSSEC / Wazuh (HIDS)

OSSEC is a host-based IDS; Wazuh is its popular fork with better tooling and a web UI.

```bash
# Install Wazuh agent (on monitored host)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor \
  | sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" \
  | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update && sudo apt install -y wazuh-agent

# Configure manager IP
sudo sed -i 's/MANAGER_IP/your.wazuh.manager/' /var/ossec/etc/ossec.conf
sudo systemctl enable --now wazuh-agent
```

**OSSEC/Wazuh capabilities**:
- File Integrity Monitoring (FIM) — detects file changes on critical paths
- Log analysis — parses syslog, auth.log, Apache, Nginx, Windows Event Logs
- Rootcheck — scans for known rootkit signatures
- Vulnerability detection — maps installed packages to CVEs
- Active response — can block IPs via iptables/firewalld

```bash
# Manually run an integrity check
sudo /var/ossec/bin/ossec-control check

# View OSSEC alerts
sudo tail -f /var/ossec/logs/alerts/alerts.log

# Check agent status
sudo /var/ossec/bin/agent_control -l
```

### 6. Log Analysis

Logs are the raw material for detection. Key sources:

| Log File | Contents |
|----------|----------|
| `/var/log/auth.log` | Authentication events (sudo, su, SSH) |
| `/var/log/syslog` | General system messages |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/nginx/access.log` | HTTP requests |
| `journalctl -u sshd` | SSH daemon via systemd |

```bash
# Find failed SSH logins
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head

# Find successful logins after failures (potential brute-force success)
grep "Accepted password\|Accepted publickey" /var/log/auth.log | tail -20

# Count 4xx HTTP errors by IP
awk '$9 ~ /^4/ {print $1}' /var/log/nginx/access.log \
  | sort | uniq -c | sort -rn | head -20

# Extract all IPs that triggered 403 Forbidden
grep " 403 " /var/log/nginx/access.log | awk '{print $1}' \
  | sort | uniq -c | sort -rn

# Monitor auth.log in real time
sudo journalctl -u ssh -f --output=short-iso

# Search for suspicious cron jobs added recently
find /etc/cron* /var/spool/cron -newer /etc/passwd -ls 2>/dev/null
```

---

## Practical Exercises

### Exercise 1 – Deploy Snort and Trigger an Alert

```bash
# Install and configure
sudo apt install -y snort

# Add a test rule
echo 'alert tcp any any -> any 4444 (msg:"Netcat Shell Detected"; sid:9999001; rev:1;)' \
  | sudo tee /etc/snort/rules/local.rules

# Run Snort in the background
sudo snort -A fast -q -i lo -c /etc/snort/snort.conf \
  -l /var/log/snort/ &

# Trigger the alert (in another terminal)
nc -zv 127.0.0.1 4444 2>/dev/null || true

# Check alerts
sudo cat /var/log/snort/alert
```

### Exercise 2 – Fail2ban Brute-Force Simulation

```bash
# Ensure Fail2ban is running
sudo systemctl start fail2ban

# Simulate failed SSH logins by generating fake log lines
for i in $(seq 1 6); do
  echo "$(date '+%b %d %H:%M:%S') $(hostname) sshd[$$]: Failed password for root \
    from 10.0.0.99 port 2222$i ssh2" | sudo tee -a /var/log/auth.log
done

# Wait for Fail2ban to process
sleep 5
sudo fail2ban-client status sshd

# Check if the IP is banned
sudo iptables -L f2b-sshd -n 2>/dev/null || sudo nft list ruleset 2>/dev/null | grep 10.0.0.99
```

### Exercise 3 – Log Analysis Pipeline

```bash
# Generate a sample Apache-format log for analysis
cat > /tmp/sample_access.log <<'EOF'
192.168.1.10 - - [10/Jun/2025:10:01:01 +0000] "GET /index.html HTTP/1.1" 200 1024
10.0.0.5 - - [10/Jun/2025:10:01:05 +0000] "GET /admin/../../../etc/passwd HTTP/1.1" 403 512
10.0.0.5 - - [10/Jun/2025:10:01:06 +0000] "GET /?id=1' OR '1'='1 HTTP/1.1" 400 256
192.168.1.20 - - [10/Jun/2025:10:01:10 +0000] "POST /login HTTP/1.1" 401 128
192.168.1.20 - - [10/Jun/2025:10:01:11 +0000] "POST /login HTTP/1.1" 401 128
192.168.1.20 - - [10/Jun/2025:10:01:12 +0000] "POST /login HTTP/1.1" 401 128
EOF

# Count requests per IP
echo "=== Requests per IP ==="
awk '{print $1}' /tmp/sample_access.log | sort | uniq -c | sort -rn

# Find suspicious patterns
echo "=== Path Traversal Attempts ==="
grep "\.\." /tmp/sample_access.log

echo "=== Possible SQL Injection ==="
grep -iE "(union|select|insert|drop|or .1.=.1)" /tmp/sample_access.log

echo "=== 40x Errors by IP ==="
awk '$9 ~ /^4/ {print $1, $9}' /tmp/sample_access.log | sort | uniq -c | sort -rn
```

### Exercise 4 – Custom Suricata Rule

```bash
# Write a rule detecting a custom C2 beacon pattern
sudo tee /etc/suricata/rules/custom.rules > /dev/null <<'EOF'
alert http $HOME_NET any -> $EXTERNAL_NET any \
  (msg:"Possible C2 Beacon - Suspicious User-Agent"; \
  flow:established,to_server; http.user_agent; content:"Mozilla/4.0"; \
  http.uri; content:"/update"; sid:8000001; rev:1;)

alert dns any any -> any any \
  (msg:"Long DNS Query - Possible DNS Tunneling"; \
  dns.query; isdataat:50,relative; sid:8000002; rev:1;)
EOF

# Add to suricata.yaml rule-files section and reload
sudo suricata-update --no-merge
sudo kill -USR2 $(pgrep suricata) 2>/dev/null || sudo systemctl restart suricata
```

---

## Common Pitfalls

| Pitfall | Impact | Mitigation |
|---------|--------|------------|
| Running IDS on the wrong interface | Misses all traffic | Use `ip link` to verify and specify correct interface |
| Not updating signatures | Misses new attacks | Schedule daily `suricata-update` / `pulledpork` runs |
| Tuning Fail2ban ban time too short | Attackers simply retry | Start with 24h bantime, escalate repeat offenders |
| Ignoring false positives | Alert fatigue | Tune rules; whitelist known-good IPs |
| Log rotation breaking HIDS | FIM misses changes | Configure logrotate to signal OSSEC after rotation |
| No central log collection | Logs deleted by attacker | Ship logs to remote syslog/SIEM immediately |
| Running Snort as root | Privilege escalation risk | Drop privileges with `-u snort -g snort` |

---

## Summary

- **Snort/Suricata** inspect network traffic; Suricata is multi-threaded and preferred for modern deployments
- **Fail2ban** provides automatic IP blocking based on log patterns — essential for any internet-facing SSH
- **OSSEC/Wazuh** monitors host-level events: file integrity, logs, and system calls
- **Log analysis** with `grep`/`awk` is a fundamental skill; ELK/Wazuh dashboards scale this up
- Combine NIDS + HIDS + centralised logging for defence-in-depth
- Tune aggressively to reduce alert fatigue — an ignored alert is as bad as no alert

---

## Further Reading

- [Snort 3 Documentation](https://www.snort.org/documents)
- [Suricata User Guide](https://docs.suricata.io/)
- [Fail2ban Manual](https://www.fail2ban.org/wiki/index.php/MANUAL_0_8)
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Emerging Threats Rules](https://rules.emergingthreats.net/)
- *The Practice of Network Security Monitoring* by Richard Bejtlich
