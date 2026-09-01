# 06 — Web Application Testing (DVWA)

## What is Web Application Testing?

Web application testing means finding and exploiting vulnerabilities in websites and web applications running on the target server.

**Analogy:**
- FTP/SMB attacks = attacking the **back office** of a building
- Web application testing = attacking through the **main entrance** (the website)

---

## Target Web Applications on Metasploitable2

Metasploitable2 comes with two intentionally vulnerable web applications:

| Application | Full Name | URL | Purpose |
|-------------|-----------|-----|---------|
| DVWA | Damn Vulnerable Web Application | http://192.168.1.50/dvwa | Beginner-friendly web hacking practice |
| Mutillidae | NOWASP Mutillidae | http://192.168.1.50/mutillidae | Advanced web vulnerability practice |

---

## Accessing the Web Application

### Step 1 — Fix Firefox Proxy Settings
If Firefox shows "The proxy server is refusing connections":

1. Open Firefox Settings (☰ → Settings)
2. Scroll to bottom → Network Settings
3. Change from "Manual proxy" → **"No proxy"**
4. Click OK

### Step 2 — Open DVWA
In Firefox address bar:
```
http://192.168.1.50/dvwa/login.php
```

### Step 3 — Login to DVWA
| Field | Value |
|-------|-------|
| Username | admin |
| Password | password |

✅ **DVWA dashboard accessible!**

---

## DVWA Vulnerability Categories

DVWA includes the following vulnerability types to practice:

| Vulnerability | Description |
|--------------|-------------|
| **SQL Injection** | Manipulate database queries through input fields |
| **XSS (Cross-Site Scripting)** | Inject malicious scripts into web pages |
| **Brute Force** | Guess passwords using automated tools |
| **Command Injection** | Execute OS commands through web forms |
| **File Upload** | Upload malicious files to the server |
| **File Inclusion** | Include remote/local files to read sensitive data |
| **CSRF** | Cross-Site Request Forgery attacks |
| **Insecure CAPTCHA** | Bypass CAPTCHA protections |

---

## DVWA Security Levels

DVWA has four difficulty levels — start with Low:

| Level | Description |
|-------|-------------|
| **Low** | No security controls — easiest to exploit |
| **Medium** | Basic protections — requires bypassing filters |
| **High** | Strong protections — needs advanced techniques |
| **Impossible** | Fully secure — used as reference for defense |

---

## Setting Security Level to Low

1. Login to DVWA
2. Click **"DVWA Security"** in left menu
3. Select **"Low"** from dropdown
4. Click **Submit**

---

## Next Steps in DVWA

### SQL Injection (Basic)
1. Click SQL Injection in left menu
2. Enter in User ID field:
   ```
   1' OR '1'='1
   ```
3. This bypasses authentication and dumps all users

### XSS (Reflected)
1. Click XSS (Reflected) in left menu
2. Enter in Name field:
   ```html
   <script>alert('XSS by Megha')</script>
   ```
3. A popup appears — XSS confirmed

### Brute Force
1. Click Brute Force in left menu
2. Use Burp Suite or Hydra to automate password guessing

---

## Remediation for Web Vulnerabilities

| Vulnerability | Fix |
|--------------|-----|
| SQL Injection | Use parameterized queries / prepared statements |
| XSS | Sanitize and encode all user input |
| Brute Force | Implement rate limiting + account lockout |
| Command Injection | Never pass user input to OS commands |
| File Upload | Whitelist allowed file types, scan uploads |
| CSRF | Use CSRF tokens on all forms |

---

## Key Finding

Metasploitable2's web server is running **Apache + PHP** with **no security controls** — making it a perfect practice target for all major OWASP Top 10 vulnerabilities.
