# OpenTenebris

**Where Shadows Connect**

A modern community platform for real-time messaging, servers, and community management.

---

## Features

### Communication
- **Servers** — Create community servers with banners, icons, and categories
- **Channels** — Text channels with permission controls
- **Direct Messages** — Private conversations with block management and privacy settings
- **Group Chats** — Group conversations with invite links and member management
- **Real-Time Messaging** — WebSocket-powered instant messaging with room-based delivery
- **Call History** — Call metadata tracking (duration, timestamps)

### Community Management
- **Custom Roles** — Create roles with custom colors and permission sets
- **Permission System** — Granular role-based permissions for channel and server management
- **Member Management** — Role assignment, moderation tools, audit logging
- **Server Invites** — Custom invite URLs with optional expiry

### Platform Features
- **Platform Servers** — Official community servers pre-configured and joinable by anyone
- **Reporting System** — User reporting with moderator review workflow
- **Friend System** — Friend requests with privacy controls
- **User Profiles** — Custom avatars, banners, bios, status indicators

---

## Privacy & Data Handling

OpenTenebris collects only what is necessary to operate.

### What We Collect
- **Account data**: username, email (optional, used only for password recovery)
- **Communications**: message content, message metadata
- **Security data**: IP addresses (abuse prevention and rate limiting only)

### What We Do NOT Collect
- No tracking cookies
- No analytics scripts
- No behavioral profiling
- No third-party data sharing
- No advertising data

### Email Policy
- Email is **completely optional** during registration
- You can register with no email or any email address — no verification required
- Email is only used if you ever need to reset your password
- We recommend providing a valid email so you can recover your account if you lose access

### Data Retention
- Messages archived after 30 days (platforms with ≤100 users), 90 days (100–1000 users), or 150 days (1000+ users)
- Archived messages are compressed to cold storage
- Rate-limit and security event records are purged hourly
- Account deletion removes or anonymizes all associated data

[Read the full Data Handling Policy →](DATA-HANDLING.md)

---

## Abuse Prevention

OpenTenebris uses a multi-layered security system to protect against abuse:

### Blacklist System (3 Layers)
1. **Server-level username patterns** — exact, regex, and similarity matching
2. **Server Blacklist v2** — Levenshtein distance matching with granular permissions
3. **Platform-wide blacklist** — exact and critical (45%+ similarity) patterns with auto-purge workflow

### Auto-Purge Monitoring
Users matching platform-level blacklist patterns enter a 5-day monitoring period. If still flagged after 5 days, the account is anonymized, tokens revoked, and associated IPs blocked.

### Rate Limiting
- 500 API requests/minute
- 20 login attempts / 5 minutes
- 10 registrations / hour / IP
- 50 friend requests / 5 minutes
- 200 messages / minute

[Read the full Security Policy →](SECURITY.md)

---

## Architecture

```
Caddy (reverse proxy, TLS) ──▶ Flask API (Waitress)
                                  ──▶ SQLite (WAL mode)
                                  ──▶ Cold storage (compressed archives)
                              ──▶ FastAPI WebSocket 
```

**Stack:** Python, Flask, FastAPI (WebSocket), SQLite, Caddy, Waitress

---

## Getting Started

OpenTenebris is hosted at **[opentenebris.org](https://opentenebris.org)**.

1. Create an account — username and password only, email optional
2. Browse or create servers
3. Join channels and start chatting

---

*Built by WinterGate Intelligence Collective — Where Shadows Connect.*
