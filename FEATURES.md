# OpenTenebris — Complete Feature Reference

**Last updated: June 2026**

---

## Table of Contents
1. [Core Communication](#1-core-communication)
2. [Server Management](#2-server-management)
3. [Channel System](#3-channel-system)
4. [Roles & Permissions](#4-roles--permissions)
5. [User Profiles & Appearance](#5-user-profiles--appearance)
6. [Member List & Display](#6-member-list--display)
7. [Verification System](#7-verification-system)
8. [Security & Hardening](#8-security--hardening)
9. [Platform Administration](#9-platform-administration)
10. [Privacy Controls](#10-privacy-controls)
11. [Infrastructure](#11-infrastructure)
12. [Drag & Drop](#12-drag--drop)
13. [Context Menus](#13-context-menus)
14. [WebRTC Voice/Video](#14-webrtc-voicevideo)
15. [Session Memory](#15-session-memory)
16. [Audit & Logging](#16-audit--logging)
17. [Data Handling](#17-data-handling)

---

## 1. Core Communication

### Servers
- Create community servers with custom name, icon, banner, background image, and description
- Server discovery via platform categories
- Joinable via invite links (with optional expiry) or direct platform join
- Tags system for server categorization and discovery
- Server-level blacklist with exact, regex, and similarity matching

### Text Channels
- Unlimited text channels per server
- Permission controls per channel: `can_read` (everyone, members, admin) and `can_send` (everyone, admin, none for read-only)
- Channel types: text, announcement, announcement_sync, voice
- Channel categories with collapsible groups
- Real-time messaging via WebSocket (FastAPI on port 8000)
- Message polling fallback (every 3 seconds) when WebSocket disconnects
- Auto-scroll to bottom on new messages
- Typing indicators broadcast via WebSocket
- Message history with pagination (50 per page, load-more button)

### Direct Messages
- Private 1-on-1 conversations
- Block user (prevents all future DMs)
- Hide conversation (removes from list without deleting)
- Read receipts (mark as read on open)
- Privacy controls: who can DM you (everyone, friends only, nobody)
- Message editing and deletion

### Group Chats
- Create group conversations with name and description
- Invite users via invite links or direct add
- Owner + admin role hierarchy within groups
- Remove members (owner/admin only)
- Member count display
- Real-time messaging via WebSocket

### Voice Channels
- Dedicated voice channel type in channel list
- Voice icon indicator on channel items
- Double-click to join a voice channel triggers a WebRTC call
- WebRTC integration with TURN server support

### Real-Time System
- WebSocket connection per active channel (auto-connects, auto-reconnects with 5s backoff)
- Room-based message delivery
- Presence updates (online, idle, dnd, offline)
- Typing indicator with 3-second timeout
- Graceful fallback to polling on WebSocket failure

---

## 2. Server Management

### Create & Delete
- Create server with name and optional icon
- Delete server permanently (owner only, requires double confirmation)
- Leave server (members and admins; owner must delete or transfer)

### Settings
- **General tab**: Server name, tags, icon URL, description, banner, background image
- **Platform category** (platform servers only): server type selection
- **Announcement channels**: server announcements, platform sync, global sync
- **Roles tab**: Create/edit/delete roles, display separate toggle, hoist role selection
- **Blacklist tab**: Username blacklist with exact match and regex modes
- **Verify tab**: Verification level selector, channel pickers, role assignment, auto-build

### Invites
- Generate invite links via API
- Copy invite link to clipboard
- Invite links work for direct join
- Server owners can create infinite-use or single-use invites

### Profile & Appearance
- Custom server icon (URL-based)
- Server banner image
- Server background image
- Server description
- Tags for categorization

---

## 3. Channel System

### Channel Types
| Type | Icon | Behavior |
|---|---|---|
| Text | `#` | Standard text messaging |
| Announcement | 📢 | Announcement channel with sync capability |
| Announcement Sync | 🔄 | Synchronized with platform/global announcements |
| Voice | 🎤 | Triggers WebRTC call on double-click |

### Channel Categories
- Group channels under named categories
- Collapsible categories (state persisted in localStorage)
- Category-level permission indicators
- Empty category hint for admins ("+ Add a channel")
- Rename and delete categories

### Channel Permissions
- `can_read`: `everyone` (default), `members`, `admin`
- `can_send`: `everyone` (default), `admin`, `none` (read-only)
- Visual indicators: 🔒 (admin-only), 🔒 (members-only), 🚫 (read-only), ✏️ (admin-only send)
- Permission icons shown inline in channel list

### Batch Operations
- Channel reorder: `PUT /api/channels/reorder` — accepts ordered array of channel IDs
- Category reorder: `PUT /api/categories/reorder` — accepts ordered array of category IDs

### Channel CRUD
- Create channel with type, name, category, and permissions
- Rename channel in-place
- Delete channel (channel and all its messages)
- Edit channel permissions after creation

---

## 4. Roles & Permissions

### Role System
- Create roles with name, color (hex), and optional display-separate toggle
- Roles ordered by position (higher position = higher priority)
- Color inheritance: highest-position role with a non-default color determines member color
- Edit role name, color, display-separate setting
- Delete role (unassigns from all members)
- Auto-incrementing position on creation

### Display-Separate Roles
- Toggle `display_separate` on any role
- Members with display-separate roles appear in their own section in the member list
- Section header shows `ROLE NAME — count`
- Members without a display-separate role appear in "EVERYONE ELSE" section
- Each member shows their `top_role.name` as a tag next to their username

### Hoist Role (Top Role Display)
- Server owner/admins can set a `hoist_role_id` on the server
- If set and the user has that role, it becomes their displayed top role
- Otherwise, the highest-position role is used
- Gives explicit control over which role appears in the member list

### Role Assignment
- Users assigned roles via `user_role_assignments` table
- Role-on-verify: automatically assign a role when user passes verification
- Role permissions (future: granular permission bits per role)

---

## 5. User Profiles & Appearance

### Profile Color
- Users can set a custom hex profile color (e.g., `#ff6600`)
- Color stored on `users.color` column
- Profile color appears as the username color in:
  - Server member list
  - Direct message conversation list
  - Direct message message headers
  - Group chat member list
  - Group chat message headers
- Profile color picker in Settings → Appearance with:
  - Color input (native color picker)
  - Hex text input with validation (`/^#[0-9a-fA-F]{6}$/`)
  - Real-time sync between picker and text input
  - Save with PUT to `/api/settings/profile-color`

### Server Color Override
- Server owner/admins can set a color override on `server_settings`
- Modes:
  - **disabled**: No override (default)
  - **force**: All members without a role color use this color
  - **random**: Each member gets a deterministic HSL color derived from their user ID hash
- Override applies when member has no role colors
- Profile color used as fallback when user has no roles and server override is disabled

### Avatar & Banner
- Custom avatar URL
- Server-level icon, banner, background
- Avatar letter fallback with deterministic background color (`getAvatarColor()`)

### Status
- Online, Idle, Do Not Disturb, Offline
- Status dot indicator in member list and DM conversations
- Status updates broadcast via WebSocket presence events

### Tags
- Custom text tags per user (e.g., "Admin", "Mod", "VIP")
- Tags displayed next to username in messages and member lists
- Managed via admin endpoints

---

## 6. Member List & Display

### Enriched Members
- `GET /api/servers/<id>/members/enriched` returns:
  - `id`, `username`, `avatar`, `status`
  - `color` (computed top role color, server override, or profile color)
  - `profile_color` (raw from users table, null if user has roles)
  - `tag` (user's custom tag)
  - `role` (owner/admin/member)
  - `joined_at`
  - `server_roles` (all assigned roles with id, name, color, display_separate)
  - `has_security_icon` (for platform security badges)
  - `display_separate_roles` (roles with display_separate=true)
  - `top_role` (hoist role or highest-position role)
  - `verified` (verification status 0/1)

### Display Sections
1. **Display-separate role sections** — each role gets its own header and member list
2. **Online section** — members with online/idle/dnd status
3. **Offline section** — members with offline status
4. **Everyone Else section** — members without a display-separate role (if any exist)

### Color Display
- Username color is deterministic based on:
  1. Server color override (force mode) — highest priority
  2. Role color (highest-position non-default role)
  3. Profile color (when no roles assigned)
  4. `hashColor()` fallback (deterministic from user_id string)

---

## 7. Verification System

### Overview
Three-level server verification system to gate access behind identity confirmation.

### Level 1 — Code via DM
1. User joins server and is marked unverified (`verified = 0`)
2. User sees only the waiting and verify channels
3. User clicks "Request Code" or sends `/verify` in verify channel
4. Server generates an 8-character hex code (uses `secrets.token_hex(4)`)
5. Code is stored in `verification_codes` with 10-minute expiry
6. User enters the code in the verify channel
7. API validates code against `verification_codes` table
8. On success: `members.verified = 1`, optional role-on-verify assigned, full access granted
9. Invalid codes: logged as abuse in audit log, message removed

### Level 2 — Puzzle + Code via DM
1. User must first solve a puzzle
2. Puzzle types (random selection):
   - Color sequences ("Enter the 3rd color in: Red, Blue, Green, Yellow")
   - Reverse order ("Write these numbers backwards: 8-3-1-9-4")
   - Math ("What is 7 + 14?")
   - Pattern completion ("What letter comes next? A, C, E, G, ?")
3. Puzzle answer SHA-256 obfuscated with random salt
4. Puzzle has 5-minute expiry
5. After solving: a verification code is generated (same as Level 1 flow)
6. Code must be submitted in verify channel or via DM

### Level 3 — Application Review
1. Server owners define custom application questions
2. Questions support: type (text), required flag, max length, position ordering
3. Users submit answers as JSON object mapping question_id → answer
4. Validated server-side: required questions enforced, max length enforced
5. Application enters `pending` status
6. Server admins/owners review in settings → Verify tab → Pending Applications
7. Each application shows: username, timestamp, all question-answer pairs
8. **Approve**: sets `members.verified = 1`, generates application verification code
9. **Deny**: sets status to denied with optional review note
10. Users cannot re-submit while pending or already approved

### Auto-Build System
- One-click "Auto-Build Verify System" creates:
  - **VERIFICATION category** at position 0
  - **waiting-room** text channel (read: everyone, send: none)
  - **verify** text channel (read: everyone, send: everyone)
- Automatically saves channel IDs to verification settings
- Sets `auto_build = 1` flag

### Verify Channel Interception
- Frontend `sendMessage()` checks if active channel is the verify channel
- If yes, sends content to `/api/servers/<id>/verify/check-message` instead of normal message endpoint
- Backend validates code, marks verified, sets `members.verified = 1`
- Frontend shows system message: ✅ Verification successful! or ❌ Invalid code...
- On success: auto-reloads channels after 1 second to reveal access

### Waiting Room
- When joining a server with verification enabled, `join_server` sets `members.verified = 0`
- Frontend `renderChannels()` checks `currentMember.verified`
- Unverified users only see VERIFICATION category channels (verify + waiting-room)
- Yellow warning banner displayed: "⚠️ Verification Required — You need to verify before accessing other channels"
- On verification completion: `members.verified = 1`, full channel list revealed

### Verification Settings API
| Endpoint | Method | Purpose |
|---|---|---|
| `/api/servers/<id>/verify/settings` | GET | Get verification settings + questions |
| `/api/servers/<id>/verify/settings` | PUT | Update level, channels, role, message |
| `/api/servers/<id>/verify/questions` | POST | Add application question |
| `/api/servers/<id>/verify/questions/<qid>` | DELETE | Remove question |
| `/api/servers/<id>/verify/request-code` | POST | Request verification code (Level 1) |
| `/api/servers/<id>/verify/submit-code` | POST | Submit verification code |
| `/api/servers/<id>/verify/puzzle` | GET | Get random puzzle (Level 2) |
| `/api/servers/<id>/verify/submit-puzzle` | POST | Submit puzzle answer |
| `/api/servers/<id>/verify/check-message` | POST | Verify code via channel message |
| `/api/servers/<id>/verify/auto-build` | POST | Create verify channels automatically |
| `/api/servers/<id>/verify/applications` | GET | List pending applications (Level 3) |
| `/api/servers/<id>/verify/applications/<id>/review` | POST | Approve/deny application (Level 3) |
| `/api/servers/<id>/apply` | POST | Submit application (Level 3) |

### Database Tables
- `server_verify_settings` — level, channel IDs, welcome message, role on verify
- `server_application_questions` — questions with type, max_length, required, position
- `server_applications` — user applications with answers, status, reviewed_by
- `verification_codes` — codes with type (verify/puzzle/application), expiry, used_at

---

## 8. Security & Hardening

### Network-Level Hardening
All applied via iptables with systemd services:

#### IP Blacklist (ipset)
- Single ipset `blacklist` hash table (max 65536 entries)
- Single iptables rule: `-m set --match-set blacklist src -j DROP`
- Atomic reloads via `ipset swap`
- Blacklist file at `/etc/opentenebris/blacklist.txt`
- Auto-sync every 2 minutes via systemd timer
- No per-IP iptables rules — prevents rule table overflow

#### Ping Cloaking
- Maximum 1 ICMP echo reply per 5 seconds per source IP
- Excess pings silently dropped (appears as timeout/host-unreachable)
- Uses `iptables -m recent --name pingtrack` with `--rcheck --seconds 5`

#### Probe & Scan Protection
- Invalid TCP flag combinations blocked:
  - NULL scan (no flags)
  - XMAS scan (FIN+URG+PSH)
  - SYN+FIN combination
  - SYN+RST combination
- Backend file access pattern blocking (iptables `-m string` on ports 80/443):
  - `.db`, `.py`, `.env`, `.sqlite`, `.git`, `config.json`
  - `.backup`, `.bak`, `.sql`, `docker-compose`, `Dockerfile`
  - `proc/self`, `etc/passwd`, server-status, `.env`

#### CPU & Memory Limits
- opencode process: 25% CPU cap via `cpulimit`
- cgroup v2 memory limit: 512MB max, 256MB high watermark

### Application-Level Security

#### Authentication
- Token-based auth with SHA-256 hashed tokens
- Token stored in `auth_tokens` table with user association
- Bearer token in `Authorization` header
- Token required for all API endpoints via `@token_required` decorator

#### Rate Limiting
| Action | Limit | Window |
|---|---|---|
| General API | 500 | 60 seconds |
| Login attempts | 20 | 5 minutes |
| Registrations | 10 | 1 hour / IP |
| Friend requests | 50 | 5 minutes |
| Messages | 200 | 60 seconds |
| Reports | 20 | 1 hour |

#### Threat Detection (Auto-Defender)
Runs every 60 seconds, assigns threat scores:

| Event | Base Score |
|---|---|
| Failed login | 5 |
| Failed registration | 10 |
| Rate limit exceeded | 15 |
| SQL injection attempt | 50 |
| XSS attempt | 50 |
| Path traversal attempt | 40 |

Response thresholds: monitoring at 20+, 7-day block at 40+, 30-day block at 70+, 365-day emergency at 100+.

#### Detection Capabilities
- Brute force: >10 failed logins in 15 minutes
- Rapid requests: >50 in 5 minutes
- Injection: SQLi, XSS, command injection, path traversal, template injection, prototype pollution
- Spam: >3 URLs per message, repeated identical content

#### Blacklist System (3 Layers)
1. **Server-level username patterns** — exact, regex, Levenshtein similarity (80% threshold)
2. **Server Blacklist v2** — enhanced with critical patterns, permission-gated
3. **Platform-wide blacklist** — exact + critical (45%+ similarity) with auto-purge workflow

#### Auto-Purge
Users matching critical platform patterns enter 5-day monitoring. If still flagged after 5 days: account anonymized, tokens revoked, IPs blacklisted.

### Infrastructure Security
- TLS 1.3 via Caddy reverse proxy
- Security headers: X-Content-Type-Options, X-Frame-Options, CSP, Permissions-Policy
- Input sanitization (HTML stripping, control character removal)
- Request size limit: 10MB
- JSON content-type enforcement
- Fail2Ban jails for SSH and API abuse
- Passwords: Werkzeug scrypt-based hashing
- Auth tokens: SHA-256 hashed with server secret before DB storage
- SQLite WAL mode for write integrity

---

## 9. Platform Administration

### Platform Permissions
Boolean permission system with optional expiry:
- `view_audit_logs`
- `manage_users`
- `manage_servers`
- `view_reports`
- `manage_reports`
- `view_ip_capture`
- `manage_security`

Permissions assigned to users with configurable expiry dates.

### Audit Logs
Comprehensive audit trail tracking:
- User actions (login, registration, profile changes)
- Administrative actions (server create/delete, role changes, bans)
- Security events (threat detections, rate limit triggers)
- Verification events (code requests, puzzle solves, application reviews)
- Report resolution
- IP address capture for security events

Endpoints:
- `GET /api/audit/logs` — paginated audit log listing
- `GET /api/audit/logs/user/<id>` — per-user audit trail
- `GET /api/audit/logs/server/<id>` — per-server audit trail
- `GET /api/audit/actions` — distinct action types
- `GET /api/audit/stats` — summary statistics
- `GET /api/audit/search` — search and filter audit logs
- `GET /api/audit/export` — export audit data

### Platform Servers
- Official servers with pre-configured types
- Joinable by any user when `joinable = true`
- Category system for server discovery

### Reporting System
- User reporting with abuse categories
- Moderator review workflow
- Report resolution tracking
- Abuse archive for resolved reports

### Security Admin
- IP capture data for security events
- Blacklist management
- Monitor dashboard
- Security event review

---

## 10. Privacy Controls

### Email Policy
- Email is **completely optional** during registration
- No email verification required
- Email only used for password recovery
- Users can register with any email or none

### Profile Privacy
- Email visibility: everyone, friends only, nobody
- Friend request permissions: everyone, friends of friends, nobody
- DM permissions: everyone, friends only, nobody
- Block list: block users from DMing you
- Hide conversations from DM list

### Data Collection
- No tracking cookies or browser fingerprinting
- No behavioral analytics or user profiling
- No advertising data or ad targeting
- No third-party data sharing or sale
- No message content scanning for ads
- No social graph analysis

### Account Deletion
- Full account deletion with data removal
- Messages anonymized to `deleted_user_<hash>`
- Email, password hash, bio, avatar, banner cleared
- Auth tokens revoked immediately
- Server memberships and friend connections removed
- Data export before deletion (JSON download)

---

## 11. Infrastructure

### Architecture
```
Caddy (reverse proxy, TLS) ──▶ Flask API (Waitress, :5050)
                                  ──▶ SQLite (WAL mode)
                                  ──▶ Cold storage (compressed archives)
                              ──▶ FastAPI WebSocket (:8000)
```

### Stack
| Component | Technology | Port |
|---|---|---|
| Reverse Proxy | Caddy (TLS 1.3) | 443 |
| REST API | Flask + Waitress | 5050 |
| WebSocket | FastAPI (Uvicorn) | 8000 |
| Database | SQLite (WAL mode) | — |
| Cache | In-memory | — |

### Database
- SQLite with WAL mode for concurrent read performance
- Tables: 45+ tables covering users, servers, channels, messages, roles, permissions, security, verification
- WAL checkpoint at 1000-page intervals

### Caddy Configuration
- Automatic TLS certificate management
- Security headers automatically applied
- Reverse proxy to Flask API and WebSocket server
- Static file serving for frontend assets

---

## 12. Drag & Drop

### Channel Reordering
- Drag channels within and between categories
- Green line indicator (`#00ff88`, 3px, with box-shadow) shows exact insertion point
- Indicator positioned based on mouse Y relative to channel item midpoints
- `window._dropBefore` and `window._dropAfter` track precise drop position
- Drop above all categories works (uncategorized area)
- Batch reorder API called on drop

### Category Reordering
- Categories are draggable
- Drop indicator shows insertion point
- Category-level reorder via API

### Visual Feedback
- Dragged item: `opacity: 0.4`
- Drop indicator: absolute positioned, bright green, animated
- All channel items and labels have `draggable="true"`

---

## 13. Context Menus

### Channel Area Context Menu
- **Create Channel** — opens channel creation modal
- **Create Category** — creates new category with inline name prompt

### Channel Item Context Menu
- **Close** — closes the context menu
- **Rename Channel** — prompts for new name, sends PUT to rename endpoint
- **Copy Channel ID** — copies ID to clipboard
- **Edit Permissions** — opens permission editor modal
- **Delete Channel** — confirms and deletes channel
- **Create Channel in <Category>** — opens channel creation modal pre-set with category

### Category Context Menu
- **Create Channel** — opens channel creation modal pre-set with category
- **Rename Category** — prompts for new name
- **Delete Category** — confirms and deletes category (channels inside become uncategorized)

### Server Context Menu
- **Server Settings** — opens settings modal
- **Invite People** — opens invite modal
- **Leave Server** — confirms and leaves (members/admins only)
- **Delete Server** — double-confirm and delete (owner only)
- **Copy Server ID** — copies ID to clipboard

### Server Bar Context Menu
- **Server Settings** (owners/admins only)
- **Invite People**
- **Mark as Read**
- **Leave Server**
- **Delete Server** (owners only)
- **Copy ID**

All context menus:
- Positioned at cursor with boundary clamping
- Dark glass background with blur
- Close on click outside or Escape
- Close on any scroll event

---

## 14. WebRTC Voice/Video

### Features
- Voice calls between users
- Video calls (camera sharing)
- Screen sharing (display media)
- Quality adjustment: high (720p 30fps), medium (480p 24fps), low (240p 15fps)
- Mute/unmute microphone
- Toggle video on/off during call
- Toggle screen share on/off
- Call status display
- End call cleanup

### Backend
- `GET /api/call/token` — returns ICE servers, TURN credentials with HMAC-SHA1 authentication
- `POST /api/call/signal` — SDP offer/answer relay between peers
- TURN server credentials with username and HMAC-based credential

### Protocol
- WebRTC with STUN/TURN for NAT traversal
- SDP exchange via signaling endpoint
- PeerConnection with audio/video/data channels

---

## 15. Session Memory

### Save
- On `beforeunload`: compresses active state (serverId, channelId, servers array, collapsed categories) to base64 JSON
- Stored in `localStorage.session_memory`
- Falls back to minimal state (IDs only) if full state exceeds localStorage quota

### Restore
- On page load: reads and decompresses session memory
- Restores collapsed category states from memory
- Navigates to saved server if still valid
- Selects saved channel if still valid
- Restores full channel list state

---

## 16. Audit & Logging

### Event Types Tracked
- User authentication (login, registration, logout)
- Profile changes (password change, email change)
- Server events (create, delete, settings changes)
- Channel events (create, delete, rename, reorder)
- Role events (create, edit, delete, assignment)
- Verification events (code requested, puzzle solved, application submitted/reviewed)
- Security events (threat detected, rate limit exceeded, blacklist match)
- Report events (submitted, resolved)
- Platform administration (permission changes, security actions)

### Audit Log Fields
- `user_id` — actor
- `action` — action type string
- `target_type` — user/server/channel/role/etc
- `target_id` — ID of affected resource
- `details` — human-readable description
- `ip_address` — actor's IP (if IP capture enabled)
- `created_at` — timestamp

### IP Capture
- IPs captured for security events and audit actions
- Only users with `view_ip_capture` platform permission can see IPs in audit logs
- Owner toggle enables/disables IP visibility per user
- Stored in `blacklist_ip_capture` table

---

## 17. Data Handling

### Retention
| Data Type | Retention |
|---|---|
| Active messages | 30–150 days (by server size) |
| Archived messages | Cold storage (compressed) |
| Rate limit logs | 24 hours |
| Security events | 7 days (info), 90 days (critical) |
| IP addresses | 24 hours (rate-limit), 90 days (security) |
| Call metadata | 90 days |
| Account data | Until account deletion |

### Archiving
- Messages beyond threshold compressed to cold storage
- Active message replaced with `[Archived]`
- Archive stored as compressed files on server
- Threshold: 30 days (≤100 users), 90 days (100–1000), 150 days (1000+)

### Email
- Optional at registration
- No verification required
- Used only for password recovery
- Rate-limited by email provider quota
- Reset tokens expire after 1 hour

---

*OpenTenebris — Where Shadows Connect.*
