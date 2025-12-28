# Version Management Examples

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Directory Tree](#directory-tree)
- [Files](#files)
- [Usage](#usage)
  - [Initialise Version Management](#initialise-version-management)
  - [Bump Version](#bump-version)
  - [Update Headers Only](#update-headers-only)
- [Related Sections](#related-sections)

---

## Overview

This folder contains templates and examples for the Version Agent (`/version`). The Version Agent manages:

- **Semantic versioning** across all version files
- **VERSION-HISTORY.md** - Technical changelog for developers
- **CHANGELOG.md** - Brief developer summary (Keep a Changelog format)
- **RELEASES.md** - User-facing release notes
- **Markdown metadata headers** - Version and date tracking in all `.md` files

---

## Directory Tree

```
version/
├── README.md
├── VERSION-HISTORY-TEMPLATE.md
├── RELEASES-TEMPLATE.md
├── MARKDOWN-HEADERS.md
└── GIT-INTEGRATION.md
```

---

## Files

| File                          | Purpose                                             |
| ----------------------------- | --------------------------------------------------- |
| `README.md`                   | This file - folder overview                         |
| `VERSION-HISTORY-TEMPLATE.md` | Template and examples for technical changelog       |
| `RELEASES-TEMPLATE.md`        | Template and examples for user-facing release notes |
| `MARKDOWN-HEADERS.md`         | Guide for markdown metadata headers                 |
| `GIT-INTEGRATION.md`          | How Version Agent integrates with Git Agent         |

---

## Usage

### Initialise Version Management

```bash
/version init
```

This creates:
- `VERSION-HISTORY.md` in project root
- `RELEASES.md` in project root
- Adds metadata headers to all `.md` files

### Bump Version

```bash
# Patch release (bug fixes)
/version bump patch

# Minor release (new features)
/version bump minor

# Major release (breaking changes)
/version bump major
```

### Update Headers Only

```bash
/version headers
```

Updates the `Version` and `Last Updated` fields in all markdown files.

---

## Related Sections

- [../git/](../git/) - Git workflow examples
- [../setup/](../setup/) - Project setup templates including CHANGELOG-TEMPLATE.md
- [../../agents/version.md](../../agents/version.md) - Version agent definition
