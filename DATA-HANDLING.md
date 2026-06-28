# OpenTenebris — Data Handling Policy

**Last updated: June 2026**

---

## Our Commitment

OpenTenebris is operated by WinterGate Intelligence Collective. We collect only what is necessary to operate the platform and protect our community.

---

## 1. What We Collect

### Account Information
| Data | Required? | Purpose | Retention |
|---|---|---|---|
| Username | Yes | Account identification, display name | Until account deletion |
| Email address | **No** | Password recovery (optional) | Until account deletion |
| Password hash | Yes | Authentication (Werkzeug scrypt hash) | Until account deletion |

Email is **completely optional**. You can register with no email, a fake email, or a real one — no verification is performed. We recommend providing a valid email solely so you can reset your password if you ever lose access.

### Communications Data
| Data | Purpose | Retention |
|---|---|---|
| Message content | Service operation — delivering messages to recipients | 30–150 days (see below) |
| Message metadata (timestamps, channel) | Message ordering, threading | 30–150 days |
| Call metadata (duration, quality) | Call history display | 90 days |

### Security Data
| Data | Purpose | Retention |
|---|---|---|
| IP address | Abuse prevention, rate limiting, blacklist enforcement | 24 hours (rate-limit logs) |
| Failed login attempts | Brute force detection | 24 hours |
| Security events | Threat detection | 7 days (info), 90 days (critical) |

### Profile Data (Optional)
| Data | Purpose |
|---|---|
| Avatar, banner, bio, status | Profile display — until changed or account deleted |

---

## 2. What We Do NOT Collect

- ❌ No tracking cookies or browser fingerprinting
- ❌ No behavioral analytics or user profiling
- ❌ No advertising data or ad targeting
- ❌ No third-party data sharing or sale
- ❌ No payment information (no billing system exists)
- ❌ No message content scanning for ads
- ❌ No social graph analysis

---

## 3. Message Archiving

Messages older than a dynamic threshold are compressed to cold storage:

| Platform Size | Archive After |
|---|---|
| ≤ 100 users | 30 days |
| 100–1000 users | 90 days |
| > 1000 users | 150 days |

Archived message content is replaced in the active database with `[Archived]`. Archives are stored as compressed files on the server.

---

## 4. Account Deletion

When you delete your account:
- Messages you authored are removed from active tables
- Username is anonymized to `deleted_user_<hash>`
- Email, password hash, bio, avatar, and banner are cleared
- Auth tokens revoked immediately
- Server memberships and friend connections removed

---

## 5. Data Access & Sharing

- Platform operators access security logs for moderation
- **No data is sold or shared with third parties**
- Law enforcement requests through proper legal channels are the only exception
- Archived data may be synced to a private GitHub repository for redundancy (accessible only to operators)

---

## 6. Email & Password Reset

- Email is optional at registration — no verification email is sent
- If you provide an email, you can reset your password via a token-based reset link
- Reset tokens expire after 1 hour
- Password reset is rate-limited by the email provider's quota

---

## 7. Contact

For questions about this policy, contact the platform operator through WinterGate IC.

---

*OpenTenebris — Where Shadows Connect.*
