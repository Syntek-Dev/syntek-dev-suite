# Security Audit — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Scope of the audit is defined (whole project, specific module, or pre-release check)
- [ ] `.claude/SECURITY.md` has been read and is current
- [ ] All recent changes are committed — audit should run against a clean working tree
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Security Hardening Review

Run a comprehensive security review across the defined scope.

```
/syntek-dev-suite:security [audit scope]
```

The security agent reads `.claude/SECURITY.md` and checks:
- Authentication and session management
- Authorisation and access control (including RLS)
- Cryptography (approved algorithms, no banned algorithms)
- Input validation and injection prevention
- Transport security (HTTPS, headers, CORS)
- Secrets management (no hardcoded secrets, proper env var usage)
- File upload security (if applicable)
- Container security
- Supply chain security (dependency versions)

### Step 2 — Hostile QA Security Pass

```
/syntek-dev-suite:qa-tester [audit scope] --focus security
```

The QA tester deliberately attempts to:
- Bypass authentication
- Access resources belonging to other users (IDOR)
- Inject payloads into all input fields
- Exploit privilege escalation paths
- Trigger information disclosure via error messages

### Step 3 — Dependency Audit

Check for known vulnerabilities in project dependencies.

```bash
# For Node.js projects:
npm audit

# For Python projects:
pip-audit

# For Rust projects:
cargo audit
```

Address any HIGH or CRITICAL severity vulnerabilities before proceeding.

### Step 4 — OWASP Top 10:2025 Checklist

Work through the full OWASP checklist in `.claude/SECURITY.md` for the audit scope:

- [ ] A01: Broken Access Control — all endpoints require authorisation
- [ ] A02: Cryptographic Failures — approved algorithms only, PII encrypted at rest
- [ ] A03: Injection — all inputs validated, parameterised queries used
- [ ] A04: Insecure Design — threat modelling done, security by design
- [ ] A05: Security Misconfiguration — no default credentials, minimal exposed services
- [ ] A06: Vulnerable and Outdated Components — dependency audit clean
- [ ] A07: Identification and Authentication Failures — MFA available, session managed correctly
- [ ] A08: Software and Data Integrity Failures — CI/CD secured, dependencies verified
- [ ] A09: Security Logging and Monitoring Failures — security events logged and monitored
- [ ] A10: Server-Side Request Forgery — external URLs validated if applicable

### Step 5 — Document Audit Findings

Create a security audit report in `project-management/src/SECURITY/`:

```
project-management/src/SECURITY/SECURITY-AUDIT-{DATE}-{SCOPE}.md
```

Include:
- **Scope**: What was audited
- **Date**: Audit date
- **Conducted by**: Agent-assisted audit with [auditor name]
- **Findings**: Table of findings by severity (Critical / High / Medium / Low / Info)
- **Mitigations applied**: What was fixed during the audit
- **Remaining risk**: Any accepted risk with justification
- **Sign-off**: Date and reviewer

### Step 6 — Apply Critical and High Fixes

Address all Critical and High severity findings before the audit is considered complete. Use `code/workflows/03-security-hardening/` for each fix.

### Step 7 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If a Critical vulnerability is found in production:
1. Do not commit the audit document publicly until the vulnerability is patched
2. Follow the incident response process in `.claude/SECURITY.md`
3. Patch first, document second

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
