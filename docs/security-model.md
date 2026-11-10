# Security model

Closeout is a high-value target: it holds unfixed vulnerabilities for many companies and can send requests to their systems. This is a summary of the main threats and the controls behind each one.

| Threat | Controls |
|---|---|
| **One firm reading another firm's data** | Row-Level Security on every tenant table, enforced per transaction and failing closed. Other tenants' records return *not found*. See [tenancy](tenancy-and-data-protection.md). |
| **Retest engine used for SSRF or as an attack proxy** | Client-acknowledged scope and testing window; DNS resolved once and pinned; private and metadata ranges blocked; no redirects; per-host rate limiting; per-firm kill switch; one published egress IP. See [retest engine](retest-engine.md). |
| **Worker compromised by a hostile response** | Restricted database role with column-level grants; isolated network segment with no route to the API; regex timeouts; size and time caps. |
| **Theft of stored credentials** | Per-engagement data keys, AES-256-GCM bound to each record; decrypted only in the worker; the API exposes header names only; evidence redacted. |
| **Account takeover** | Mandatory TOTP for firm staff, replay-protected, with single-use recovery codes; per-IP rate limiting and per-account lockout; scrypt password hashing; idle and absolute session timeouts; a password change revokes other sessions. |
| **Session theft and CSRF** | Random 256-bit session tokens stored only as hashes; `httpOnly`, `SameSite`, `__Host-` prefixed secure cookies; Origin checks on every cookie-authenticated write. |
| **XSS** | Framework escaping; a per-request nonce Content Security Policy; target responses rendered as text only. |
| **Malicious uploads** | Size caps; XML with DOCTYPE or entity declarations rejected (no XXE, no entity bombs); schema-validated input. |
| **Billing tampering** | Webhook signatures checked with a short replay window; each event applied once; out-of-order events can't roll back state; unknown prices never grant a plan; card data never touches Closeout's servers. |
| **Leaked API token** | Shown once and stored as a hash; scoped to the remediation workflow only (no account, team, billing or token endpoints); fixed expiry; revocable; creation triggers an email alert and an audit entry. |
| **Email as a leak or phishing vector** | No vulnerability details in emails; links only to Closeout's own origin; the mailer has no database or key access. |
| **False "fixed" attestations** | Ambiguous responses are inconclusive, never fixed; a human confirms every closure; append-only audit log. |
| **Supply chain** | Lockfile and dependency audit gate, automated updates, secret scanning, SAST, container and IaC scanning; minimal images. |

## Audit log

Security-relevant actions (sign-ins, failed sign-ins, scope acknowledgements, retests, closures, token creation, integration changes) are written to an append-only audit log, recording who did what, to which record, and from which IP address.

## Reporting a vulnerability

See the [security policy](https://getcloseout.tech/security) or `/.well-known/security.txt`. Good-faith research within the policy won't be pursued legally.
