# 08 — Web Application Testing

## Target
Apache web server running on Metasploitable2 — `http://192.168.1.50`

---

## Tool 1 — Gobuster (Directory Enumeration)

```bash
gobuster dir -u http://192.168.1.50 -w /usr/share/wordlists/dirb/common.txt
```

### Results:

| Path | Status | Risk |
|------|--------|------|
| `phpMyAdmin/` | 301 | 🔴 High — database admin panel exposed |
| `phpinfo.php` | 200 | 🟡 Medium — full PHP config exposed |
| `twiki/` | 301 | 🔴 High — TWiki 2003, known RCE vulns |
| `dav/` | 301 | 🟠 High — WebDAV endpoint |
| `test/` | 301 | 🟡 Medium — unlabeled test directory |
| `cgi-bin/` | 403 | 🟢 Low — blocked |
| `.htaccess` | 403 | 🟢 Low — blocked |

---

## Tool 2 — Nikto (Automated Vulnerability Scan)

```bash
nikto -h http://192.168.1.50
```

Scan duration: 409 seconds · 8,242 requests · **30 findings**

### Key Findings:

**Outdated Software:**
```
Apache/2.2.8  (current: 2.4.66+) — End of life
PHP/5.2.4     (current: 8.5.1+)  — End of life, multiple CVEs
```

**Information Disclosure:**
- `/phpinfo.php` — full PHP configuration exposed publicly
- Directory indexing enabled on `/icons/`, `/doc/`, `/test/`
- PHP Easter egg query strings reveal version info
- ETag header leaks filesystem inode numbers (CVE-2003-1418)

**Exposed Admin Interface:**
- `/phpMyAdmin/` — database admin panel, browsable unauthenticated
- phpMyAdmin `ChangeLog` and `README` files fully accessible

**Missing Security Controls:**
- No `Content-Security-Policy` header
- No `Strict-Transport-Security` header
- No `X-Content-Type-Options` header
- No `Permissions-Policy` header
- HTTP `TRACE` method enabled (XST risk)

---

## Tool 3 — DVWA (Damn Vulnerable Web Application)

### Access:
```
URL: http://192.168.1.50/dvwa/login.php
Username: admin
Password: password
```
✅ **DVWA dashboard accessible with default credentials!**

### Vulnerability Categories in DVWA:

| Vulnerability | Description |
|--------------|-------------|
| SQL Injection | Manipulate database queries |
| XSS | Inject malicious scripts |
| Brute Force | Automated password guessing |
| Command Injection | Execute OS commands via web |
| File Upload | Upload malicious files |
| File Inclusion | Read sensitive files |
| CSRF | Cross-Site Request Forgery |

### Security Levels:
- **Low** → No protections (start here)
- **Medium** → Basic filters
- **High** → Strong protections
- **Impossible** → Fully secure (reference)

---

## Manual Follow-Up Findings

### phpinfo.php
Confirmed exact versions:
- PHP: 5.2.4-2ubuntu5.10 (build Jan 2010)
- System: Linux metasploitable 2.6.24-16-server

### phpMyAdmin
Login page loads but reports `mcrypt` extension error. Intended vulnerable config: **blank root password** (`root` / *empty*).

### TWiki
Confirmed via bundled README: **TWiki released Feb 2003** — multiple documented RCE vulnerabilities in `configure` and `rdiff` scripts.

---

## Summary

| Finding | Severity |
|---------|----------|
| Apache 2.2.8 — end-of-life | 🔴 High |
| PHP 5.2.4 — end-of-life | 🔴 High |
| phpMyAdmin exposed | 🔴 High |
| TWiki 2003 — known RCE vulns | 🔴 High |
| phpinfo.php exposed | 🟡 Medium |
| Directory indexing enabled | 🟡 Medium |
| Missing security headers | 🟢 Low |
| HTTP TRACE enabled | 🟢 Low |

---

## Remediation
1. Upgrade Apache and PHP to supported versions
2. Remove or restrict `phpinfo.php`
3. Restrict `phpMyAdmin` to trusted IPs only
4. Disable directory indexing: `Options -Indexes`
5. Add security headers (CSP, HSTS, X-Content-Type-Options)
6. Disable HTTP TRACE: `TraceEnable off`
7. Decommission or upgrade TWiki
