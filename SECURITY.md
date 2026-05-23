# Security Policy

Datean Technology takes the security of our software and infrastructure seriously.
We appreciate the responsible disclosure of vulnerabilities by the security community.

---

## Supported Versions

We actively maintain and patch **the latest production deployment only**.

| Version / Branch | Supported |
|------------------|-----------|
| Latest `env/prod` deployment | ✅ Actively supported |
| Previous production releases | ❌ Not supported |
| `env/test` and feature branches | ❌ Not supported (pre-release) |

If you have found a vulnerability in an older release, please check whether it still
exists in the current production version before reporting.

---

## Reporting a Vulnerability

> ⚠️ **Do NOT open a public GitHub issue for security vulnerabilities.**
> Public disclosure before a fix is available puts all users at risk.

Please report vulnerabilities by emailing:

📧 **mercy.akinkuotu@hotmail.com**

Use the subject line: `[SECURITY] <brief description>`

If the content is sensitive, request our PGP public key in a preliminary email and we
will provide it for encrypted communication.

---

## What to Include in Your Report

To help us triage and reproduce the issue quickly, please provide as much of the
following as possible:

| Field | Details |
|-------|---------|
| **Affected service** | Repository name and/or URL |
| **Vulnerability type** | e.g. XSS, SSRF, broken auth, exposed credentials, injection |
| **Severity estimate** | Critical / High / Medium / Low (CVSS if available) |
| **Steps to reproduce** | Clear, numbered steps from a clean starting state |
| **Proof of concept** | Code snippet, screenshot, or HTTP request/response |
| **Observed impact** | What an attacker could achieve |
| **Suggested fix** | Optional — include if you have one |
| **Your contact details** | So we can follow up and credit you (if desired) |

The more detail you provide, the faster we can act.

---

## Response Timeline

| Milestone | Target |
|-----------|--------|
| **Acknowledgement** | Within 48 hours of receipt |
| **Initial assessment** | Within 3 business days |
| **Fix plan communicated to reporter** | Within 7 calendar days |
| **Patch released** | Depends on severity — see below |

### Severity-based patch targets

| Severity | Target patch window |
|----------|---------------------|
| Critical (CVSS ≥ 9.0) | 24–48 hours |
| High (CVSS 7.0–8.9) | 7 days |
| Medium (CVSS 4.0–6.9) | 30 days |
| Low (CVSS < 4.0) | Next scheduled release |

We will keep you informed at each milestone. If we anticipate missing a target, we
will communicate this proactively.

---

## Disclosure Policy

We follow **coordinated disclosure**:

1. Reporter notifies us privately.
2. We confirm receipt within 48 hours.
3. We work on a fix and agree a disclosure date with the reporter.
4. Fix is deployed to production.
5. Reporter and Datean Technology publish disclosures simultaneously (or reporter
   publishes after the fix is confirmed live).

We ask that you:
- Allow us a reasonable window to fix the issue before any public disclosure.
- Avoid accessing, modifying, or deleting data that does not belong to you during testing.
- Not perform denial-of-service testing or any action that degrades service for other users.

In return, we commit to:
- Treating your report with urgency and respect.
- Not pursuing legal action against researchers acting in good faith.
- Crediting you in our release notes or security advisory (if you wish).

---

## Out of Scope

The following are generally outside the scope of this policy:

- Vulnerabilities in third-party services we depend on (report these directly to the vendor)
- Social engineering attacks against Datean Technology employees
- Physical attacks against our offices or infrastructure
- Findings from automated scanners submitted without a proof of concept
- Issues affecting only unsupported versions

---

*Last reviewed: May 2026*
```

---
