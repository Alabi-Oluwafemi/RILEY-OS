# Riley’s Digest — Security Design & Deployment Checklist

## Security layers

### Layer 1: Identity
- Passwords are hashed with Node.js `scryptSync` using unique salts.
- Password length is enforced at 12+ characters and new accounts can be forced to change the temporary password.
- Sessions use 256-bit random opaque tokens; only SHA-256 token hashes are stored in SQLite.
- Session cookies are `HttpOnly`, `SameSite=Lax`, `Secure` in production and expire after a configurable TTL.
- Password changes revoke other active sessions.
- Disabled users have their sessions revoked.

### Layer 2: Access control
- Server-side RBAC uses explicit permissions rather than role-name checks scattered across pages.
- Every business API route calls `requirePermission()`.
- Customers are additionally restricted by `client_id` for projects, invoices and service requests.
- The UI only hides links/buttons as a convenience; authorization remains server-side.

### Layer 3: Request protection
- State-changing requests require a double-submit CSRF token (`rd_csrf` cookie + `x-rd-csrf` header).
- Mutations check the `Origin` header against same-origin or `APP_ALLOWED_ORIGINS`.
- API body size is capped at 100 KB in the shared guard.
- Login and authenticated API calls have rate limits.

### Layer 4: Data protection
- SQL is parameterized through better-sqlite3 prepared statements.
- SQLite enables foreign keys, WAL mode, busy timeout and secure delete.
- Audit records avoid password/token material and capture security-relevant metadata.
- Secrets should never be placed in normal CRM/content tables.

### Layer 5: Browser/network hardening
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy`
- COOP/CORP headers
- HSTS
- CSP baseline

## Least-privilege matrix

| Capability | Owner | Employee | Developer | Security Analyst | Customer |
|---|:---:|:---:|:---:|:---:|:---:|
| Business dashboard | ✓ | ✓ | ✓ | ✓ | ✓ (own only) |
| Client/CRM data | Full | Read/create/update | — | — | — |
| Leads | Full | Read/create/update | — | — | — |
| Projects/tasks | Full | Read/create/update | — | — | Own projects read |
| Editorial content | Full | Read/create/update | — | — | — |
| Social media | Full | Read/create/update | — | — | — |
| Inventory | Full | Read | — | — | — |
| Invoices | Full | — | — | — | Own invoices read |
| Services catalogue | Full | Read | Read | — | Read |
| Security workspace | Full | — | Read | Read/manage | — |
| Audit logs | Full | — | Read | Read | — |
| User/team administration | Full | — | — | — | — |
| Service requests | Full | Read/update | — | — | Create/read own |

## Before public production deployment

1. Move from local SQLite to managed PostgreSQL with encrypted storage.
2. Replace in-process rate limiting with Redis/Upstash so limits work across instances.
3. Add MFA/WebAuthn for Owner, Security Analyst and Developer accounts.
4. Add centralized logs/SIEM, alerting and retention controls.
5. Add private object storage and signed URLs for documents/uploads.
6. Add dependency updates, npm audit/Dependabot-equivalent, secret scanning, SAST and DAST to CI.
7. Tighten CSP with nonces and remove `unsafe-inline` / `unsafe-eval` after deployment testing.
8. Configure `TRUST_PROXY=true` only when the reverse proxy is trusted and overwrites client IP headers.
9. Add backup encryption, restore tests and disaster-recovery procedures.
10. Add formal authorization/security tests covering every role and endpoint.

This is a security-oriented application starter, not a substitute for a formal penetration test, secure code review, or compliance assessment.
