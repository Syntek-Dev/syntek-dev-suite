# Code Examples

**Version:** 1.0.0

## Overview

This directory contains minimal, versioned code examples for common patterns across our supported technology stacks:

| Stack | Languages/Frameworks |
|-------|---------------------|
| **TALL** | Laravel 12.x, PHP 8.4, Alpine.js 3, Livewire 3, Tailwind CSS 4, MariaDB 12 |
| **Django** | Python 3.14, Django 6.x, Wagtail 7.x, Strawberry GraphQL, PostgreSQL 18 |
| **React** | TypeScript 5.9, React 19.x, Next.js 16.x, Tailwind CSS 4, Node.js 24 |
| **Mobile** | TypeScript 5.9, React Native 0.83.x, Expo, NativeWind 4.x |

## Table of Contents

- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [How to Use These Examples](#how-to-use-these-examples)
- [Version Information](#version-information)
- [Examples by Domain](#examples-by-domain)

---

## Directory Structure

```
examples/
├── README.md                 # This file
├── VERSIONS.md               # Framework version tracking
├── CHANGELOG.md              # Example version history
│
├── setup/
│   ├── README-TEMPLATE.md
│   ├── GITIGNORE.md
│   ├── SHELL-SCRIPTS.md
│   ├── ENV-FILES.md
│   ├── CLAUDE-MD-TEMPLATE.md
│   ├── EDITOR-CONFIG.md
│   ├── CHANGELOG-TEMPLATE.md
│   └── SECTION-README-TEMPLATE.md   # NEW
│
├── backend/
│   ├── pii/
│   │   ├── RESPONSE-TRANSFORMERS.md
│   │   ├── MIDDLEWARE-GUARDS.md
│   │   └── STORAGE-SERVICES.md
│   └── rate-limiting/
│       └── RATE-LIMITING.md
│
├── database/
│   ├── migrations/
│   │   ├── LARAVEL.md
│   │   ├── DJANGO.md
│   │   ├── PRISMA.md
│   │   └── TYPEORM.md
│   ├── pii/
│   │   └── TABLE-DESIGN.md
│   └── sql/
│       └── SYNTAX-REFERENCE.md
│
├── authentication/
│   ├── PASSWORD-VALIDATION.md
│   ├── MFA.md
│   ├── SESSION-MANAGEMENT.md
│   ├── IP-SECURITY.md
│   ├── SOCIAL-LOGIN.md         # NEW
│   └── PASSKEYS.md             # NEW
│
├── security/
│   ├── SIGNED-URLS.md
│   ├── IRREVERSIBLE-URLS.md
│   ├── RBAC.md
│   ├── SECURITY-HEADERS.md
│   └── PII-ACCESS.md
│
├── cicd/
│   ├── GITHUB-ACTIONS.md
│   ├── DDEV-CONFIG.md
│   ├── AWS-DEPLOYMENT.md
│   ├── DIGITAL-OCEAN.md
│   └── DOCKER.md
│
├── frontend/
│   └── PII-MASKING.md
│
├── gdpr/
│   ├── PII-STORAGE.md
│   ├── DATA-EXPORT.md
│   ├── ANONYMISATION.md
│   └── LEGAL-TEMPLATES.md
│
├── notifications/
│   ├── EMAIL-TEMPLATES.md
│   ├── PUSH-NOTIFICATIONS.md
│   ├── PII-MASKING.md
│   └── EMAIL-PROVIDERS.md      # NEW
│
├── export/
│   └── CSV-FORMATTER.md
│
├── reporting/
│   └── REPORT-SERVICES.md
│
├── logging/                    # NEW
│   └── LOGGING.md
│
├── git/                        # NEW
│   └── GIT-WORKFLOWS.md
│
├── debugger/                   # NEW
│   └── DEBUGGING.md
│
├── test-writer/                # NEW
│   └── TESTING.md
│
├── code-reviewer/              # NEW
│   └── CODE-REVIEW.md
│
├── qa-tester/                  # NEW
│   └── QA-TESTING.md
│
├── refactor/                   # NEW
│   └── REFACTORING.md
│
├── seo/                        # NEW
│   └── SEO.md
│
├── syntax/                     # NEW
│   └── SYNTAX-LINTING.md
│
└── data-scientist/             # NEW
    └── DATA-ANALYSIS.md
```

## How to Use These Examples

### For Agents

Agents should follow this workflow when providing code examples:

1. **Check project versions** - Read `composer.json`, `package.json`, `requirements.txt`, or `pyproject.toml`
2. **Check latest secure versions online** - Search for current stable and security releases
3. **Compare with example versions** - Read `VERSIONS.md` for what examples are tested against
4. **Adapt examples** - Modify syntax if project uses different version than examples
5. **Reference the example file** - Point user to the full example for context

### For Developers

1. Browse the relevant domain folder for your use case
2. Check the metadata table at the top of each file for version compatibility
3. Copy and adapt the example for your specific framework version
4. Refer to `VERSIONS.md` to see if your framework version matches

## Version Information

See [VERSIONS.md](VERSIONS.md) for:
- Current framework versions used in examples
- Latest stable versions available
- Security-related version information
- Update recommendations

## Examples by Domain

### Setup

| Example | Description | Path |
|---------|-------------|------|
| README Template | Standard project README structure | [setup/README-TEMPLATE.md](setup/README-TEMPLATE.md) |
| Section README Template | Folder-level README structure with tree layout | [setup/SECTION-README-TEMPLATE.md](setup/SECTION-README-TEMPLATE.md) |
| Git Ignore Files | .gitignore, .dockerignore, .gitattributes | [setup/GITIGNORE.md](setup/GITIGNORE.md) |
| Shell Scripts | dev.sh, test.sh, staging.sh, production.sh | [setup/SHELL-SCRIPTS.md](setup/SHELL-SCRIPTS.md) |
| Environment Files | .env templates for all environments | [setup/ENV-FILES.md](setup/ENV-FILES.md) |
| CLAUDE.md Template | Project context file template | [setup/CLAUDE-MD-TEMPLATE.md](setup/CLAUDE-MD-TEMPLATE.md) |
| Editor Config | .editorconfig, .prettierrc, ESLint | [setup/EDITOR-CONFIG.md](setup/EDITOR-CONFIG.md) |
| Changelog Template | CHANGELOG.md following Keep a Changelog | [setup/CHANGELOG-TEMPLATE.md](setup/CHANGELOG-TEMPLATE.md) |

### Backend

| Example | Description | Path |
|---------|-------------|------|
| Response Transformers | Filter PII in API responses | [backend/pii/RESPONSE-TRANSFORMERS.md](backend/pii/RESPONSE-TRANSFORMERS.md) |
| Middleware Guards | Protect PII endpoints | [backend/pii/MIDDLEWARE-GUARDS.md](backend/pii/MIDDLEWARE-GUARDS.md) |
| Storage Services | Encrypt/hash PII for storage | [backend/pii/STORAGE-SERVICES.md](backend/pii/STORAGE-SERVICES.md) |
| Rate Limiting | Throttle sensitive endpoints | [backend/rate-limiting/RATE-LIMITING.md](backend/rate-limiting/RATE-LIMITING.md) |

### Database

| Example | Description | Path |
|---------|-------------|------|
| Laravel Migrations | Laravel migration patterns | [database/migrations/LARAVEL.md](database/migrations/LARAVEL.md) |
| Django Models | Django model and migration patterns | [database/migrations/DJANGO.md](database/migrations/DJANGO.md) |
| Prisma Schema | Prisma schema definitions | [database/migrations/PRISMA.md](database/migrations/PRISMA.md) |
| TypeORM Entities | TypeORM entity and migration patterns | [database/migrations/TYPEORM.md](database/migrations/TYPEORM.md) |
| PII Table Design | Secure PII storage schema | [database/pii/TABLE-DESIGN.md](database/pii/TABLE-DESIGN.md) |
| SQL Syntax Reference | Cross-engine SQL syntax differences | [database/sql/SYNTAX-REFERENCE.md](database/sql/SYNTAX-REFERENCE.md) |

### Authentication

| Example | Description | Path |
|---------|-------------|------|
| Password Validation | Strong password rules with breach detection | [authentication/PASSWORD-VALIDATION.md](authentication/PASSWORD-VALIDATION.md) |
| MFA | TOTP and SMS OTP multi-factor authentication | [authentication/MFA.md](authentication/MFA.md) |
| Session Management | Secure sessions, logout, password reset | [authentication/SESSION-MANAGEMENT.md](authentication/SESSION-MANAGEMENT.md) |
| IP Security | Secure IP capture for auth events | [authentication/IP-SECURITY.md](authentication/IP-SECURITY.md) |
| Social Login | OAuth2 with Google, Facebook, GitHub, Twitter/X, Instagram | [authentication/SOCIAL-LOGIN.md](authentication/SOCIAL-LOGIN.md) |
| Passkeys | WebAuthn/FIDO2 passwordless authentication | [authentication/PASSKEYS.md](authentication/PASSKEYS.md) |

### Security

| Example | Description | Path |
|---------|-------------|------|
| Signed URLs | Signed URLs, admin paths, token-based access | [security/SIGNED-URLS.md](security/SIGNED-URLS.md) |
| Irreversible URLs | UUIDs, Hashids, single-use tokens | [security/IRREVERSIBLE-URLS.md](security/IRREVERSIBLE-URLS.md) |
| RBAC | Role-based access control, permissions, policies | [security/RBAC.md](security/RBAC.md) |
| Security Headers | HTTP headers, rate limiting, IP allowlisting | [security/SECURITY-HEADERS.md](security/SECURITY-HEADERS.md) |
| PII Access | Permission-gated PII access, audit logging | [security/PII-ACCESS.md](security/PII-ACCESS.md) |

### CI/CD

| Example | Description | Path |
|---------|-------------|------|
| GitHub Actions | CI pipelines, staging/production deployments | [cicd/GITHUB-ACTIONS.md](cicd/GITHUB-ACTIONS.md) |
| DDEV Config | DDEV project setup, custom services, commands | [cicd/DDEV-CONFIG.md](cicd/DDEV-CONFIG.md) |
| AWS Deployment | ECS, S3/CloudFront, Lambda workflows | [cicd/AWS-DEPLOYMENT.md](cicd/AWS-DEPLOYMENT.md) |
| Digital Ocean | App Platform, Droplet, Kubernetes | [cicd/DIGITAL-OCEAN.md](cicd/DIGITAL-OCEAN.md) |
| Docker | Multi-stage Dockerfiles, Docker Compose | [cicd/DOCKER.md](cicd/DOCKER.md) |

### Frontend

| Example | Description | Path |
|---------|-------------|------|
| PII Masking | Masked display components for PII | [frontend/PII-MASKING.md](frontend/PII-MASKING.md) |

### GDPR

| Example | Description | Path |
|---------|-------------|------|
| PII Storage | Encrypted PII storage patterns | [gdpr/PII-STORAGE.md](gdpr/PII-STORAGE.md) |
| Data Export | User data export (DSAR) | [gdpr/DATA-EXPORT.md](gdpr/DATA-EXPORT.md) |
| Anonymisation | Data anonymisation and cookie consent | [gdpr/ANONYMISATION.md](gdpr/ANONYMISATION.md) |
| Legal Templates | Privacy Policy and T&Cs markdown templates | [gdpr/LEGAL-TEMPLATES.md](gdpr/LEGAL-TEMPLATES.md) |

### Notifications

| Example | Description | Path |
|---------|-------------|------|
| Email Templates | Reusable email layouts | [notifications/EMAIL-TEMPLATES.md](notifications/EMAIL-TEMPLATES.md) |
| Push Notifications | Mobile push notification setup | [notifications/PUSH-NOTIFICATIONS.md](notifications/PUSH-NOTIFICATIONS.md) |
| PII Masking | Mask PII in notifications | [notifications/PII-MASKING.md](notifications/PII-MASKING.md) |
| Email Providers | Postmark and Mailchimp Transactional | [notifications/EMAIL-PROVIDERS.md](notifications/EMAIL-PROVIDERS.md) |

### Export

| Example | Description | Path |
|---------|-------------|------|
| CSV Formatter | CSV export with streaming | [export/CSV-FORMATTER.md](export/CSV-FORMATTER.md) |

### Reporting

| Example | Description | Path |
|---------|-------------|------|
| Report Services | Base report service patterns | [reporting/REPORT-SERVICES.md](reporting/REPORT-SERVICES.md) |

### Logging

| Example | Description | Path |
|---------|-------------|------|
| Logging | Structured logging, log channels, audit trails | [logging/LOGGING.md](logging/LOGGING.md) |

### Git

| Example | Description | Path |
|---------|-------------|------|
| Git Workflows | Branch naming, commit messages, pre-commit hooks | [git/GIT-WORKFLOWS.md](git/GIT-WORKFLOWS.md) |

### Debugger

| Example | Description | Path |
|---------|-------------|------|
| Debugging | Xdebug, Django Debug Toolbar, React DevTools, Flipper | [debugger/DEBUGGING.md](debugger/DEBUGGING.md) |

### Test Writer

| Example | Description | Path |
|---------|-------------|------|
| Testing | Unit tests, integration tests, E2E tests | [test-writer/TESTING.md](test-writer/TESTING.md) |

### Code Reviewer

| Example | Description | Path |
|---------|-------------|------|
| Code Review | Review checklists, before/after examples | [code-reviewer/CODE-REVIEW.md](code-reviewer/CODE-REVIEW.md) |

### QA Tester

| Example | Description | Path |
|---------|-------------|------|
| QA Testing | Functional, security, accessibility testing | [qa-tester/QA-TESTING.md](qa-tester/QA-TESTING.md) |

### Refactor

| Example | Description | Path |
|---------|-------------|------|
| Refactoring | Extract service, composition, state machine patterns | [refactor/REFACTORING.md](refactor/REFACTORING.md) |

### SEO

| Example | Description | Path |
|---------|-------------|------|
| SEO | Meta tags, sitemaps, structured data, ASO | [seo/SEO.md](seo/SEO.md) |

### Syntax

| Example | Description | Path |
|---------|-------------|------|
| Syntax & Linting | ESLint, Prettier, PHPStan, Ruff, Mypy configs | [syntax/SYNTAX-LINTING.md](syntax/SYNTAX-LINTING.md) |

### Data Scientist

| Example | Description | Path |
|---------|-------------|------|
| Data Analysis | Analytics services, charts, Pandas integration | [data-scientist/DATA-ANALYSIS.md](data-scientist/DATA-ANALYSIS.md) |
