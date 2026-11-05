# Threat Model and Secure Configuration

**Summary:** This is a threat model of a small notes web app ("NoteApp") where users log in and save private notes. It includes a trust-boundary diagram, a STRIDE threat list, risks ranked by likelihood and impact, and a hardening checklist with verification steps.

## 1. Diagram: users, data, trust boundaries, external dependencies

```
[Browser/User] --HTTPS--> [Web Server/App] --> [Database]
   (untrusted)    |         (trusted)            (notes, users, password hashes)
                  |             |
          Trust boundary 1      +--> [Email service] (external)
                                +--> [CDN / npm packages] (external)
```

**Assets:** user credentials, session tokens, private notes, admin access, database backups

**Trust boundaries:** Internet to app; app to database; app to third-party services

## 2. Threats (STRIDE)

| Category | Threat |
|---|---|
| Spoofing | Credential stuffing, stolen session cookie |
| Tampering | SQL injection, changing another user's note ID (IDOR) |
| Repudiation | No audit logs for logins or admin actions |
| Information disclosure | Verbose error messages, exposed DB backups, notes readable via IDOR |
| Denial of service | No rate limiting on login or search |
| Elevation of privilege | Missing access checks on admin routes |

## 3. Risk ranking (Likelihood x Impact, scale 1-5)

| Rank | Risk | L | I | Score | Level |
|---|---|---|---|---|---|
| 1 | IDOR on notes | 5 | 5 | 25 | High |
| 2 | Credential stuffing / weak login | 5 | 4 | 20 | High |
| 3 | SQL injection | 3 | 5 | 15 | High |
| 4 | Vulnerable dependencies | 4 | 3 | 12 | Medium |
| 5 | Missing audit logs | 4 | 2 | 8 | Medium |
| 6 | Login denial of service | 3 | 2 | 6 | Low |

## 4. Hardening checklist with verification steps

- [ ] Enforce an ownership check on every note query. **Verify:** log in as User A and request User B's note ID; expect 403/404.
- [ ] Rate-limit login, add lockout and MFA. **Verify:** 10 failed logins trigger a block.
- [ ] Use parameterized queries only. **Verify:** input `' OR 1=1--` returns no extra data.
- [ ] Set cookies to `HttpOnly; Secure; SameSite=Lax`, use HTTPS only, enable HSTS. **Verify:** check response headers in browser dev tools.
- [ ] Hash passwords with bcrypt or argon2. **Verify:** database shows no plaintext passwords.
- [ ] Run `npm audit` or `pip-audit` and patch. **Verify:** zero high/critical findings.
- [ ] Log logins and admin actions, hide stack traces. **Verify:** a forced error shows a generic message.
- [ ] Encrypt backups and restrict database network access. **Verify:** the database is not reachable from the public internet.
