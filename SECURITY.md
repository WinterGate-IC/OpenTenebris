# OpenTenebris — Security & Abuse Prevention

**Last updated: June 2026**

---

## 1. Blacklist System (3 Layers)

### Layer 1: Server-Level Username Blacklist
Server owners and moderators can block username patterns within their servers:
- **Exact match** — block a specific username
- **Regex pattern** — block usernames matching a pattern
- **Similarity matching** — sequence matching at 80% threshold to catch leetspeak and variations

### Layer 2: Server Blacklist v2
Enhanced blacklisting with Levenshtein distance similarity scoring and critical pattern flags. Requires dedicated `blacklist`/`unblacklist` role permissions.

### Layer 3: Platform-Wide Blacklist
Managed by platform administrators:
- **Exact** — case-insensitive exact username match
- **Critical** — 45%+ similarity via Levenshtein distance, initiates the auto-purge workflow

---

## 2. Auto-Purge System

When a user matches a platform-level critical pattern:

1. **5-day monitoring period** — user continues to have access; every API request captures their IP
2. **After 5 days** if still flagged:
   - Account anonymized (username, email, password, bio, avatar cleared)
   - Auth tokens revoked
   - All captured IPs added to firewall blacklist
   - Audit trail created

---

## 3. Autonomous Threat Detection

The auto-defender runs every 60 seconds and assigns threat scores to IPs:

| Event | Base Score |
|---|---|
| Failed login | 5 |
| Failed registration | 10 |
| Rate limit exceeded | 15 |
| SQL injection attempt | 50 |
| XSS attempt | 50 |
| Path traversal attempt | 40 |

### Response Thresholds
| Score | Action | Duration |
|---|---|---|
| 20–39 | Monitoring only | 1 day |
| 40–69 | IP blocked | 7 days |
| 70–99 | IP blocked | 30 days |
| 100+ | IP blocked, emergency review | 365 days |

---

## 4. Detection Capabilities

- Brute force: >10 failed logins in 15 minutes
- Rapid requests: >50 requests in 5 minutes
- Injection detection: SQLi, XSS, command injection, path traversal, template injection, prototype pollution
- Spam: >3 URLs per message, repeated identical content

---

## 5. Rate Limiting

| Action | Limit | Window |
|---|---|---|
| General API requests | 500 | 60 seconds |
| Login attempts | 20 | 5 minutes |
| Registrations | 10 | 1 hour / IP |
| Friend requests | 50 | 5 minutes |
| Messages | 200 | 60 seconds |
| Reports | 20 | 1 hour |

---

## 6. Infrastructure Security

- TLS 1.3 via Caddy reverse proxy
- Security headers: X-Content-Type-Options, X-Frame-Options, CSP, Permissions-Policy
- Input sanitization (HTML stripping, control character removal)
- Request size limit: 10MB
- JSON content-type enforcement
- Fail2Ban: SSH and API abuse jails

---

## 7. Data Security

- Passwords hashed with Werkzeug's scrypt-based hashing
- Auth tokens SHA-256 hashed with server secret before storage
- SQLite WAL mode for write integrity
- Database accessible only by the application user (filesystem permissions)

---

## 8. Transparency

- All security events logged with timestamps and IPs
- Audit logs track administrative actions
- Regular automated cleanup of obsolete data
- Users can report abuse directly through the platform

---

*OpenTenebris — Where Shadows Connect.*
