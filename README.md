Riley’s Digest Secure Business Hub
A Next.js 16 + React 19 business management app for Riley’s Digest, rebuilt around defense-in-depth and least privilege.
Security architecture
Authentication — email/password login, scrypt password hashing, opaque random server-side sessions, HttpOnly/SameSite cookies, inactivity/session expiry.
Least-privilege RBAC — OWNER, EMPLOYEE, DEVELOPER, SECURITY_ANALYST, CUSTOMER roles with explicit permission lists in `src/lib/permissions.ts`.
Row-level customer scoping — CUSTOMER users can only query projects, invoices and service requests attached to their `client_id`.
CSRF + same-origin validation — state-changing API requests require a CSRF header tied to a SameSite cookie and a same-origin/allowlist check.
Rate limiting — login attempts and authenticated API calls are rate limited. The starter uses an in-process limiter; production deployments should use Redis/Upstash or equivalent.
Secure database access — parameterized SQLite queries, foreign-key enforcement, WAL mode, busy timeout and secure delete.
Audit logging — authentication, authorization failures, user changes, invoices, inventory movements, content/social changes and service requests can be recorded.
Security headers — frame protection, MIME sniffing protection, referrer policy, permissions policy, COOP/CORP and CSP are configured in `next.config.ts`.
Input validation — shared length/type/format checks for common API fields. Production can add a schema library such as Zod once dependencies are available.
Password lifecycle — newly created users can receive a temporary password and are required to change it at first login.
Role boundaries
Role	Default access
OWNER	Full business, security, finance and user administration
EMPLOYEE	Clients, leads, projects/tasks, content, social, inventory read, requests
DEVELOPER	System/service visibility and audit visibility; no finance or inventory write
SECURITY_ANALYST	Security and audit capabilities; no finance or business-content administration
CUSTOMER	Own projects, own invoices, own service requests and service catalogue
Permission enforcement is server-side. The sidebar is only a convenience layer; hiding a menu item is not the security control.
Setup
Requires Node.js 20.9+.
```bash
cp .env.example .env.local
# edit BOOTSTRAP_OWNER_EMAIL and BOOTSTRAP_OWNER_PASSWORD
npm install
npm run db:seed
npm run dev
```
Open `http://localhost:3000/login`.
After signing in as OWNER, use Team & Roles to create employees, developers, security analysts and customers. Generated temporary passwords are shown once and new accounts must change their password at first login.
Important production hardening
This repo is a secure starter, not a final compliance-certified system. Before public deployment, add:
Managed PostgreSQL with encrypted storage/backups instead of local SQLite.
Redis-backed distributed rate limiting.
MFA / WebAuthn or TOTP for staff and especially owner/security roles.
A dedicated secret manager for cybersecurity credentials and API tokens.
Object storage with private buckets, malware scanning and signed URLs for uploads.
Centralized logging/SIEM and alerting for anomalous auth and permission events.
CSP nonces (rather than `unsafe-inline` / `unsafe-eval`) and a reviewed production security header policy.
Automated dependency scanning, SAST, secret scanning, DAST and CI protections.
Database encryption/key-management appropriate to the hosting environment.
Formal backup restore tests and incident-response procedures.
Do not put real passwords, private keys, API tokens, banking credentials or client secrets into the ordinary business tables.
