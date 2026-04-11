# Security Hardening — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Scope is defined: which module, endpoint, or feature is being hardened
- [ ] `.claude/SECURITY.md` has been read and is current
- [ ] Branch created following `how-to/workflows/02-git-workflow/`
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Security Implementation Review

Analyse the target scope for security vulnerabilities and apply hardening.

```
/syntek-dev-suite:security [scope description]
```

The security agent reads `.claude/SECURITY.md` and checks against OWASP Top 10:2025. It will:
- Review authentication and authorisation patterns
- Check for injection vulnerabilities (SQL, command, XSS)
- Verify Row Level Security (RLS) is applied to all user-scoped tables
- Check secrets management and transport security
- Review file upload handling (if applicable)

### Step 2 — Hostile QA Review

Run a hostile QA pass specifically targeting security edge cases.

```
/syntek-dev-suite:qa-tester [scope description] --focus security
```

The QA tester will attempt to find:
- Privilege escalation paths
- IDOR (Insecure Direct Object Reference) vulnerabilities
- Unvalidated inputs
- Session management weaknesses
- Data exposure via error messages or logs

### Step 3 — OWASP Checklist Review

Open `.claude/SECURITY.md` and work through the OWASP Top 10:2025 checklist manually for the specific scope. Confirm each item is addressed:

- [ ] A01: Broken Access Control
- [ ] A02: Cryptographic Failures
- [ ] A03: Injection
- [ ] A04: Insecure Design
- [ ] A05: Security Misconfiguration
- [ ] A06: Vulnerable and Outdated Components
- [ ] A07: Identification and Authentication Failures
- [ ] A08: Software and Data Integrity Failures
- [ ] A09: Security Logging and Monitoring Failures
- [ ] A10: Server-Side Request Forgery (SSRF)

### Step 4 — Document Findings

Create a security audit record in `project-management/src/SECURITY/`:

```
project-management/src/SECURITY/SECURITY-AUDIT-{DATE}-{SCOPE}.md
```

Include: scope, findings, mitigations applied, remaining risk (if any), and sign-off.

### Step 5 — Test Security Controls

```
/syntek-dev-suite:test-writer [scope description] --mode security-tests
```

Ensure security controls have corresponding tests (especially for auth boundaries and RLS policies).

### Step 6 — Code Review

```
/syntek-dev-suite:review
```

### Step 7 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If a critical vulnerability is found:
1. Do not commit the vulnerable code
2. Log the finding in `project-management/src/SECURITY/` immediately
3. If the vulnerability exists in production, trigger the incident response process in `.claude/SECURITY.md`

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
