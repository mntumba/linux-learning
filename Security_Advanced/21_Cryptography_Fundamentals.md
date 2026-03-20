# Lesson 21: Cryptography Fundamentals

> **Security Advanced · Lesson 21** | Difficulty: ★★★★☆ Advanced | Time: ~120 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Explain the difference between symmetric and asymmetric cryptography and when to use each
- Encrypt, decrypt, sign, and verify files using GPG
- Understand the SSL/TLS handshake and certificate chain of trust
- Manage keys securely: generation, storage, rotation, and revocation
- Inspect and troubleshoot TLS connections with `openssl` and `curl`

---

## Prerequisites

- Comfort with the Linux command line (file permissions, pipes, redirection)
- Basic understanding of networking (TCP/IP, ports, DNS)
- Modules 1–20 of this course, or equivalent experience

---

## Key Concepts

### 1. Symmetric Cryptography

Symmetric algorithms use the **same key** for both encryption and decryption. They are fast and suitable for bulk data.

| Algorithm | Key Size | Notes |
|-----------|----------|-------|
| AES-128   | 128-bit  | Minimum recommended today |
| AES-256   | 256-bit  | Gold standard; FIPS-approved |
| ChaCha20  | 256-bit  | Preferred on constrained hardware |
| 3DES      | 112-bit  | Legacy; avoid in new designs |

**Key challenge**: How do two parties share the key securely? This is the *key distribution problem* that asymmetric crypto solves.

```bash
# Encrypt a file with AES-256-CBC using openssl
openssl enc -aes-256-cbc -salt -pbkdf2 -in secret.txt -out secret.enc

# Decrypt
openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out secret_decrypted.txt
```

### 2. Asymmetric Cryptography

Asymmetric (public-key) cryptography uses a mathematically linked **key pair**:

- **Public key** – shared freely; used to *encrypt* or *verify*
- **Private key** – kept secret; used to *decrypt* or *sign*

Common algorithms:

| Algorithm | Use Case | Notes |
|-----------|----------|-------|
| RSA-2048+ | Encryption, signatures | Widely supported |
| RSA-4096  | High-security signatures | Slower; key management overhead |
| ECDSA P-256 | Signatures (TLS, SSH) | Smaller keys, faster |
| Ed25519   | SSH, package signing | Modern, recommended |
| DH / ECDH | Key exchange | Never used alone for auth |

```bash
# Generate an RSA-4096 key pair
openssl genrsa -out private.pem 4096
openssl rsa -in private.pem -pubout -out public.pem

# Encrypt a small payload with the public key
openssl pkeyutl -encrypt -inkey public.pem -pubin -in secret.txt -out secret_rsa.enc

# Decrypt with the private key
openssl pkeyutl -decrypt -inkey private.pem -in secret_rsa.enc -out secret_rsa.dec
```

### 3. Cryptographic Hash Functions

Hashes produce a fixed-length *digest* from arbitrary input. They must be:
- **Pre-image resistant** – cannot reverse a hash to find input
- **Collision resistant** – infeasible to find two inputs with the same hash

```bash
# Compute hashes
sha256sum file.txt
sha512sum file.txt
b2sum file.txt          # BLAKE2, faster than SHA-2 with similar security

# Verify a downloaded file against its published checksum
echo "expected_hash  file.iso" | sha256sum --check
```

> ⚠️ **MD5 and SHA-1 are broken** for security use. Use SHA-256 or stronger.

### 4. GNU Privacy Guard (GPG)

GPG implements the OpenPGP standard (RFC 4880) and provides:
- Symmetric encryption (passphrase-based)
- Asymmetric encryption and signing
- A *web of trust* key model

```bash
# Generate a new keypair (interactive)
gpg --full-generate-key

# List keys
gpg --list-keys          # public keyring
gpg --list-secret-keys   # private keyring

# Export your public key (share with others)
gpg --export --armor alice@example.com > alice_pub.asc

# Import someone else's public key
gpg --import bob_pub.asc

# Encrypt a file for Bob (requires his public key imported)
gpg --encrypt --recipient bob@example.com --armor secret.txt

# Decrypt
gpg --decrypt secret.txt.asc

# Sign a file (detached signature)
gpg --detach-sign --armor document.pdf

# Verify a signature
gpg --verify document.pdf.asc document.pdf
```

**Key management best practices**:

```bash
# Set an expiry date when creating (recommended: 1–2 years)
# Then extend before expiry rather than revoking
gpg --edit-key alice@example.com
# > expire  (set new expiry)
# > save

# Generate a revocation certificate immediately after key creation
gpg --gen-revoke alice@example.com > alice_revoke.asc
# Store this offline securely!

# Upload your public key to a keyserver
gpg --keyserver hkps://keys.openpgp.org --send-keys <KEYID>
```

### 5. SSL/TLS Deep Dive

TLS (Transport Layer Security) secures communications over TCP. The handshake:

```
Client                          Server
  |--- ClientHello  ------------>|  (TLS version, cipher suites, random)
  |<-- ServerHello  -------------|  (chosen cipher, random, certificate)
  |<-- Certificate  -------------|  (server's X.509 cert)
  |<-- ServerHelloDone ----------|
  |--- ClientKeyExchange ------->|  (pre-master secret, or ECDHE params)
  |--- ChangeCipherSpec -------->|
  |--- Finished (encrypted) ---->|
  |<-- ChangeCipherSpec ----------|
  |<-- Finished (encrypted) -----|
  |=== Application Data =========|
```

**TLS 1.3** (RFC 8446) simplifies this to a 1-RTT handshake and removes weak cipher suites.

```bash
# Inspect a server's certificate chain
openssl s_client -connect example.com:443 -showcerts </dev/null 2>/dev/null

# Check certificate expiry
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -dates

# Test supported TLS versions and cipher suites
nmap --script ssl-enum-ciphers -p 443 example.com

# Generate a self-signed certificate for testing
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem \
  -days 365 -nodes -subj "/CN=localhost"

# Create a Certificate Signing Request (CSR) for a CA
openssl req -newkey rsa:2048 -keyout server.key -out server.csr -nodes \
  -subj "/C=US/ST=CA/O=MyOrg/CN=myserver.example.com"
```

### 6. Key Management Principles

| Principle | Description |
|-----------|-------------|
| **Least privilege** | Keys should have the minimum necessary permissions |
| **Rotation** | Rotate keys on a schedule or after suspected compromise |
| **Separation** | Never use the same key for signing and encryption |
| **Storage** | Use HSMs, TPMs, or secrets managers (Vault, AWS KMS) for production |
| **Revocation** | Have a tested revocation procedure before you need it |

```bash
# Store secrets using HashiCorp Vault (example workflow)
vault kv put secret/myapp/db password="s3cr3t"
vault kv get secret/myapp/db

# Use age (modern alternative to GPG for file encryption)
age-keygen -o key.txt
age -r $(grep 'public key' key.txt | awk '{print $NF}') secret.txt > secret.age
age -d -i key.txt secret.age
```

---

## Practical Exercises

### Exercise 1 – Symmetric Encryption with OpenSSL

```bash
# Create a test file
echo "This is a secret message" > plaintext.txt

# Encrypt using AES-256-GCM (authenticated encryption)
openssl enc -aes-256-cbc -salt -pbkdf2 -iter 100000 \
  -in plaintext.txt -out ciphertext.bin

# Verify the encrypted file is unreadable
xxd ciphertext.bin | head

# Decrypt and compare
openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 \
  -in ciphertext.bin -out decrypted.txt
diff plaintext.txt decrypted.txt && echo "Files match!"
```

### Exercise 2 – GPG Keypair and Signing

```bash
# Create a test keypair (batch mode for scripting)
cat > gpg_batch.txt <<'EOF'
%no-protection
Key-Type: EDDSA
Key-Curve: ed25519
Subkey-Type: ECDH
Subkey-Curve: cv25519
Name-Real: Test User
Name-Email: test@example.com
Expire-Date: 1y
%commit
EOF

gpg --batch --generate-key gpg_batch.txt

# Sign and verify a file
echo "Important document" > doc.txt
gpg --local-user test@example.com --detach-sign --armor doc.txt
gpg --verify doc.txt.asc doc.txt
```

### Exercise 3 – Inspect a Real TLS Certificate

```bash
# Fetch and parse a certificate
echo | openssl s_client -connect github.com:443 2>/dev/null \
  | openssl x509 -noout -text \
  | grep -A2 "Subject\|Issuer\|Not After\|Subject Alternative"

# Check certificate transparency logs
curl -s "https://crt.sh/?q=github.com&output=json" | python3 -m json.tool | head -50
```

### Exercise 4 – Hash Verification Workflow

```bash
# Simulate a secure software distribution check
echo "Simulated binary content" > myapp_v1.0.bin
sha256sum myapp_v1.0.bin > myapp_v1.0.bin.sha256

# Simulate receiving the file and verifying
sha256sum --check myapp_v1.0.bin.sha256 && echo "Integrity OK" || echo "INTEGRITY FAILURE"

# Simulate tampering
echo "tampered" >> myapp_v1.0.bin
sha256sum --check myapp_v1.0.bin.sha256
```

---

## Common Pitfalls

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Using ECB mode for AES | Patterns leak through encryption | Use CBC or GCM with random IV |
| Hardcoding keys in source code | Key exposure via git history | Use environment variables or secrets manager |
| Not validating certificate chains | Susceptible to MITM | Always verify against a trusted CA store |
| Reusing IVs/nonces | Catastrophic for stream ciphers | Generate a fresh random nonce per message |
| Storing passwords as hashes (SHA-256) | Crackable with rainbow tables | Use bcrypt, scrypt, or Argon2 with a salt |
| Skipping certificate revocation checks | Compromised certs remain trusted | Enable OCSP stapling and CRL checking |

---

## Summary

- **Symmetric crypto** (AES) is fast and used for bulk data; requires a shared secret
- **Asymmetric crypto** (RSA, ECDSA, Ed25519) solves key distribution; used for key exchange and signatures
- **GPG** is the go-to tool for file encryption and signing on Linux
- **TLS** combines both: asymmetric for handshake/auth, symmetric for the session
- Good key management — rotation, revocation, secure storage — is as important as algorithm choice
- Prefer modern algorithms: AES-256-GCM, ChaCha20-Poly1305, Ed25519, TLS 1.3

---

## Further Reading

- [Crypto 101](https://www.crypto101.io/) – free introductory cryptography textbook
- [RFC 8446](https://tools.ietf.org/html/rfc8446) – TLS 1.3 specification
- [GPG Best Practices](https://riseup.net/en/security/message-security/openpgp/best-practices)
- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)
- [age encryption tool](https://github.com/FiloSottile/age) – modern, simple file encryption
- *The Code Book* by Simon Singh – accessible history and theory of cryptography
