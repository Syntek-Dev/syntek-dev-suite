# Changelog Template

## Overview

Standard changelog template following the [Keep a Changelog](https://keepachangelog.com/) format. Every project MUST have a `CHANGELOG.md` in the root directory.

## Metadata

| Property            | Value                                         |
| ------------------- | --------------------------------------------- |
| **Example Version** | 1.0.0                                         |
| **Last Updated**    | 2025-01                                       |
| **Stacks**          | All (TALL, Django, React, Mobile, Shared-Lib) |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Initial Changelog Template](#initial-changelog-template)
- [Changelog Update Rules](#changelog-update-rules)
  - [Example Entry](#example-entry)

---

## Initial Changelog Template

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project setup
- Development environment configuration
- CI/CD pipeline configuration

### Changed
- (No changes yet)

### Deprecated
- (No deprecations yet)

### Removed
- (No removals yet)

### Fixed
- (No fixes yet)

### Security
- (No security updates yet)
```

---

## Changelog Update Rules

1. Update changelog BEFORE merging to main/staging
2. Use present tense ("Add feature" not "Added feature")
3. Group by type: Added, Changed, Deprecated, Removed, Fixed, Security
4. Include ticket/issue references when applicable
5. Keep entries concise but descriptive

### Example Entry

```markdown
## [1.2.0] - 2025-01-15

### Added
- Add user profile page with avatar upload (#123)
- Add email notification preferences

### Changed
- Update login flow to include MFA step

### Fixed
- Fix password reset token expiry (#125)

### Security
- Update dependencies to address CVE-2025-XXXX
```