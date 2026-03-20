# 23 — Cryptography Fundamentals

> **Security Advanced · Module 3** | Difficulty: ★★★★☆ Advanced | Time: ~120 min

---

## Learning Objectives

- Understand symmetric and asymmetric encryption
- Use OpenSSL for encryption and certificate management
- Understand TLS/SSL and certificate chains
- Crack common weak cryptographic implementations
- Work with GPG for file encryption
- Understand hashing and digital signatures
- Identify cryptographic vulnerabilities

---

## 1. Cryptography Fundamentals

```
SYMMETRIC ENCRYPTION          ASYMMETRIC ENCRYPTION
┌─────────────────────┐       ┌──────────────────────────────┐
│ Same key for        │       │ Public key  → encrypt/verify │
│ encrypt/decrypt     │       │ Private key → decrypt/sign   │
│                     │       │                              │
│ Fast: AES, ChaCha20 │       │ Slow: RSA, ECC, Ed25519      │
│ Key sharing problem │       │ Solves key distribution      │
└─────────────────────┘       └──────────────────────────────┘

HYBRID APPROACH (TLS, PGP):
1. Generate random symmetric key
2. Encrypt data with symmetric key (fast)
3. Encrypt symmetric key with recipient's public key
4. Send both encrypted data + encrypted key
```

### Encryption Algorithms

| Algorithm | Type | Key Size | Status |
|-----------|------|----------|--------|
| AES-256-GCM | Symmetric | 256-bit | ✅ Secure |
| ChaCha20-Poly1305 | Symmetric | 256-bit | ✅ Secure |
| RSA | Asymmetric | 2048+ bit | ✅ Secure (2048+) |
| ECC (P-256, P-384) | Asymmetric | 256+ bit | ✅ Secure |
| Ed25519 | Asymmetric | 256-bit | ✅ Secure |
| DES | Symmetric | 56-bit | ❌ Broken |
| 3DES | Symmetric | 112-bit | ⚠️ Deprecated |
| RC4 | Stream | Variable | ❌ Broken |
| MD5 | Hash | 128-bit | ❌ Broken for passwords |
| SHA-1 | Hash | 160-bit | ❌ Broken for signatures |
| SHA-256/512 | Hash | 256/512-bit | ✅ Secure |
| bcrypt/scrypt/Argon2 | KDF | Variable | ✅ Secure for passwords |

---

## 2. OpenSSL

OpenSSL is the Swiss Army knife of cryptography on Linux.

### Symmetric Encryption

```bash
# Encrypt a file
openssl enc -aes-256-cbc -salt -in plaintext.txt -out encrypted.txt
# Enter passphrase when prompted

# Decrypt
openssl enc -d -aes-256-cbc -in encrypted.txt -out plaintext.txt

# Modern (use GCM mode for authenticated encryption)
openssl enc -aes-256-gcm -salt -in plaintext.txt -out encrypted.txt -pbkdf2

# Without prompting (scripting)
openssl enc -aes-256-cbc -salt -k "SecretPassphrase" -in file.txt -out file.enc
```

### Asymmetric Encryption

```bash
# Generate RSA key pair
openssl genrsa -out private.pem 4096          # private key
openssl rsa -in private.pem -pubout -out public.pem  # extract public key

# Generate Ed25519 key pair (modern, faster)
openssl genpkey -algorithm ed25519 -out ed25519_private.pem
openssl pkey -in ed25519_private.pem -pubout -out ed25519_public.pem

# Encrypt with public key
openssl rsautl -encrypt -inkey public.pem -pubin -in message.txt -out message.enc
# Decrypt with private key
openssl rsautl -decrypt -inkey private.pem -in message.enc -out message.txt

# View key information
openssl pkey -in private.pem -text -noout
openssl pkey -in public.pem -pubin -text -noout
```

### Hashing

```bash
# Hash a file
openssl dgst -sha256 file.txt
openssl dgst -sha512 file.txt
openssl dgst -md5 file.txt        # DON'T use for security

# Hash of a string
echo -n "password" | openssl dgst -sha256

# Or use system tools
sha256sum file.txt
sha512sum file.txt
md5sum file.txt

# HMAC
echo -n "message" | openssl dgst -sha256 -hmac "secret_key"
```

### Digital Signatures

```bash
# Sign a file
openssl dgst -sha256 -sign private.pem -out signature.sig document.txt

# Verify signature
openssl dgst -sha256 -verify public.pem -signature signature.sig document.txt
# Output: Verified OK (or Verification Failure)

# Sign with pkeyutl (for Ed25519)
openssl pkeyutl -sign -inkey ed25519_private.pem -in document.txt -out sig.bin
openssl pkeyutl -verify -inkey ed25519_public.pem -pubin -in document.txt -sigfile sig.bin
```

### SSL/TLS Certificates

```bash
# Generate a self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Generate CSR (Certificate Signing Request) for CA
openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes
# Fill in: Country, State, Organization, Common Name (domain)

# View certificate
openssl x509 -in cert.pem -text -noout

# Check certificate expiry
openssl x509 -in cert.pem -noout -dates

# Check remote server certificate
openssl s_client -connect google.com:443 -servername google.com
echo | openssl s_client -connect google.com:443 | openssl x509 -text -noout

# Verify certificate chain
openssl verify -CAfile ca.pem server.pem

# Create Certificate Authority (CA)
# 1. Create CA key and self-signed cert
openssl genrsa -out ca.key 4096
openssl req -x509 -new -key ca.key -sha256 -days 3650 -out ca.crt -subj "/CN=My CA"

# 2. Sign CSR with CA
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out server.crt -days 365 -sha256
```

---

## 3. GPG — GNU Privacy Guard

GPG implements OpenPGP for email and file encryption.

```bash
# Generate GPG key pair
gpg --gen-key
gpg --full-gen-key    # with more options

# List keys
gpg --list-keys                    # public keys
gpg --list-secret-keys             # private keys

# Export keys
gpg --export -a "Alice" > alice_public.gpg      # public key
gpg --export-secret-keys -a "Alice" > alice_private.gpg  # private key (keep secure!)

# Import a key
gpg --import alice_public.gpg

# Encrypt a file for a recipient
gpg --encrypt --recipient alice@example.com document.txt
# Creates: document.txt.gpg

# Encrypt + sign
gpg --encrypt --sign --recipient alice@example.com document.txt

# Decrypt
gpg --decrypt document.txt.gpg

# Sign without encrypting
gpg --sign document.txt          # binary signature
gpg --clearsign document.txt     # clear text signature
gpg --detach-sign document.txt   # separate .sig file

# Verify signature
gpg --verify document.txt.sig document.txt

# Trust levels
gpg --edit-key alice@example.com
# (in gpg prompt) trust → select level → save
```

---

## 4. TLS/SSL Analysis

### Testing TLS Configuration

```bash
# Comprehensive TLS scanner
sudo apt install sslscan
sslscan google.com

# testssl.sh
wget https://testssl.sh/testssl.sh
chmod +x testssl.sh
./testssl.sh google.com

# nmap TLS scripts
nmap --script ssl-enum-ciphers -p 443 google.com
nmap --script ssl-cert -p 443 google.com
nmap --script ssl-dh-params -p 443 google.com

# Check for weak ciphers
openssl s_client -connect target.com:443 -cipher RC4-SHA   # RC4 (broken)
openssl s_client -connect target.com:443 -cipher DES-CBC3-SHA  # 3DES (deprecated)
```

### Common TLS Vulnerabilities

```bash
# Heartbleed (CVE-2014-0160) — read server memory
nmap --script ssl-heartbleed target.com

# POODLE (SSLv3) — check if SSLv3 is enabled
nmap --script sslv2 target.com
openssl s_client -connect target.com:443 -ssl3   # should fail

# BEAST — CBC cipher suites in TLS 1.0
# Check TLS version support
nmap --script ssl-enum-ciphers target.com | grep -i "tls 1.0\|ssl"

# SWEET32 — 3DES block cipher birthday attack
nmap --script ssl-enum-ciphers target.com | grep -i "3des\|des-cbc"
```

---

## 5. Password Cracking

### Hash Identification

```bash
# Identify hash type
hash-identifier "5f4dcc3b5aa765d61d8327deb882cf99"  # MD5
hashid '$6$salt$hash'      # SHA-512 Linux

# Common hash patterns:
# $1$ = MD5 crypt
# $5$ = SHA-256 crypt
# $6$ = SHA-512 crypt (Linux /etc/shadow)
# $y$ = yescrypt (Ubuntu 22.04+)
# $2b$ = bcrypt
# 32 hex chars = MD5
# 40 hex chars = SHA-1
# 64 hex chars = SHA-256
```

### John the Ripper

```bash
sudo apt install john

# Wordlist attack
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Specify format
john --format=raw-md5 --wordlist=wordlist.txt hashes.txt
john --format=sha512crypt --wordlist=wordlist.txt shadow.txt

# Brute force
john --incremental hashes.txt

# Rules-based (mangling)
john --wordlist=wordlist.txt --rules hashes.txt

# Show cracked passwords
john --show hashes.txt

# Restore interrupted session
john --restore

# Combine shadow and passwd for cracking
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt
```

### Hashcat

```bash
sudo apt install hashcat

# Basic wordlist attack
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt   # MD5
hashcat -m 1000 hash.txt wordlist.txt                     # NTLM
hashcat -m 1800 hash.txt wordlist.txt                     # SHA-512crypt
hashcat -m 3200 hash.txt wordlist.txt                     # bcrypt
hashcat -m 2500 capture.hccapx wordlist.txt               # WPA2

# Attack modes
hashcat -m 0 -a 0 hash.txt wordlist.txt         # wordlist (a=0)
hashcat -m 0 -a 1 hash.txt wordlist1 wordlist2  # combination (a=1)
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a        # brute-force (a=3)
hashcat -m 0 -a 6 hash.txt wordlist.txt ?d?d?d  # hybrid (a=6)

# Rules
hashcat -m 0 -r /usr/share/hashcat/rules/best64.rule hash.txt wordlist.txt

# Charsets for brute force
# ?l = lowercase (a-z)
# ?u = uppercase (A-Z)
# ?d = digits (0-9)
# ?s = special chars
# ?a = all printable
# ?b = all bytes

# Show results
hashcat -m 0 hash.txt --show
```

---

## 6. Steganography

```bash
# steghide — hide data in images
sudo apt install steghide

# Embed message
steghide embed -cf image.jpg -sf secret.txt   # will prompt for passphrase

# Extract message
steghide extract -sf image.jpg

# Info about an image
steghide info image.jpg

# Check for hidden data without passphrase
steghide info image.jpg  # will attempt without password

# stegseek (fast passphrase cracking for steghide)
sudo apt install stegseek
stegseek image.jpg /usr/share/wordlists/rockyou.txt

# binwalk — find embedded files
sudo apt install binwalk
binwalk image.jpg          # scan for embedded files
binwalk -e image.jpg       # extract embedded files
```

---

## Practice Exercises

### Exercise 23.1 — OpenSSL Encryption

```bash
# 1. Create a test message
echo "Top Secret: The password is: Tr0ub4dor&3" > /tmp/secret.txt

# 2. Encrypt with AES-256-CBC
openssl enc -aes-256-cbc -salt -pbkdf2 -in /tmp/secret.txt -out /tmp/secret.enc

# 3. Decrypt and verify
openssl enc -d -aes-256-cbc -pbkdf2 -in /tmp/secret.enc -out /tmp/decrypted.txt
diff /tmp/secret.txt /tmp/decrypted.txt   # should be no differences

# 4. Generate RSA key pair and sign a file
openssl genrsa -out /tmp/mykey.pem 2048
openssl rsa -in /tmp/mykey.pem -pubout -out /tmp/mypub.pem
openssl dgst -sha256 -sign /tmp/mykey.pem -out /tmp/sig.bin /tmp/secret.txt
openssl dgst -sha256 -verify /tmp/mypub.pem -signature /tmp/sig.bin /tmp/secret.txt
```

### Exercise 23.2 — Hash Cracking

```bash
# 1. Create some MD5 hashes
echo -n "password123" | md5sum | cut -d' ' -f1 > /tmp/crack_me.txt
echo -n "letmein" | md5sum | cut -d' ' -f1 >> /tmp/crack_me.txt
echo -n "qwerty" | md5sum | cut -d' ' -f1 >> /tmp/crack_me.txt

# 2. Crack with John
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /tmp/crack_me.txt
john --show --format=raw-md5 /tmp/crack_me.txt

# 3. Crack with hashcat
hashcat -m 0 -a 0 /tmp/crack_me.txt /usr/share/wordlists/rockyou.txt
hashcat -m 0 /tmp/crack_me.txt --show
```

---

## Key Takeaways

- **AES-256-GCM** for symmetric; **RSA-4096** or **Ed25519** for asymmetric
- OpenSSL handles encryption, PKI, TLS analysis, and more
- **GPG** = OpenPGP implementation for file/email encryption
- **sslscan** and **testssl.sh** test TLS configuration quality
- Broken: **MD5, SHA-1, DES, RC4, SSLv3, TLS 1.0**
- **John the Ripper**: flexible and rule-based cracking
- **Hashcat**: GPU-accelerated, fastest option for offline cracking
- Steganography hides data in files; **stegseek** cracks steghide passphrases

---

➡️ [24 — Network Security and Traffic Analysis](24_Network_Security_Traffic_Analysis.md)

*Security Advanced · Lesson 3 of 8 | [Course Index](../INDEX.md)*
