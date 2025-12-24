# Markdown Metadata Headers Guide

**Last Updated**: 24/12/2025
**Version**: 1.2.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Header Format](#header-format)
- [Field Descriptions](#field-descriptions)
- [Placement Rules](#placement-rules)
- [Update Workflow](#update-workflow)
- [Examples](#examples)

---

## Overview

All markdown files in the project must include a standardised metadata header. This header:

- Tracks when the document was last updated
- Shows which version the document corresponds to
- Identifies who maintains the document
- Confirms language and timezone settings

The Version Agent automatically updates these headers during version bumps.

---

## Header Format

```markdown
# Document Title

**Last Updated**: DD/MM/YYYY
**Version**: X.Y.Z
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents
...
```

---

## Field Descriptions

| Field | Format | Description |
|-------|--------|-------------|
| **Last Updated** | DD/MM/YYYY | Date the file content was last modified |
| **Version** | X.Y.Z | Project version when file was last updated |
| **Maintained By** | Text | Team or individual responsible for the document |
| **Language** | en_GB | Language and locale for the document |
| **Timezone** | Europe/London | Default timezone for dates/times in the document |

---

## Placement Rules

### Correct Placement

The header must appear:
1. Immediately after the main H1 title
2. Before any other content
3. Followed by a horizontal rule (`---`)
4. Then the Table of Contents (if applicable)

### Example: Correct

```markdown
# API Documentation

**Last Updated**: 24/12/2025
**Version**: 1.2.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Endpoints](#endpoints)

---

## Overview
...
```

### Example: Incorrect

```markdown
# API Documentation

## Table of Contents        <!-- WRONG: TOC before header -->

**Last Updated**: 24/12/2025
**Version**: 1.2.0
...
```

```markdown
# API Documentation

## Overview                 <!-- WRONG: Content before header -->

This document describes...

**Last Updated**: 24/12/2025
**Version**: 1.2.0
...
```

---

## Update Workflow

### When Version Agent Updates Headers

The Version Agent updates **both** the `Version` and `Last Updated` fields whenever:

1. **Version bump** (`/version bump major|minor|patch`)
   - Updates `Version` to the new version number
   - Updates `Last Updated` to the current date

2. **Header update** (`/version headers`)
   - Updates `Last Updated` to the current date
   - Updates `Version` to the current project version

3. **Initialisation** (`/version init`)
   - Adds headers to files missing them
   - Sets `Version` to current project version
   - Sets `Last Updated` to current date

### What Gets Updated

| Command | Version Field | Last Updated Field |
|---------|---------------|-------------------|
| `/version bump` | ✅ New version | ✅ Current date |
| `/version headers` | ✅ Current version | ✅ Current date |
| `/version init` | ✅ Current version | ✅ Current date |

### Manual Updates

If you manually edit a markdown file without running a version command:
- Update the `Last Updated` date to today's date
- Keep the `Version` as-is unless bumping manually

---

## Examples

### README.md

```markdown
# Project Name

**Last Updated**: 24/12/2025
**Version**: 2.1.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [About](#about)
- [Installation](#installation)
- [Usage](#usage)

---

## About

This project provides...
```

### API Documentation

```markdown
# API Reference

**Last Updated**: 24/12/2025
**Version**: 2.1.0
**Maintained By**: Backend Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Authentication](#authentication)
- [Endpoints](#endpoints)
- [Error Handling](#error-handling)

---

## Authentication

All API requests require...
```

### Contributing Guide

```markdown
# Contributing Guide

**Last Updated**: 24/12/2025
**Version**: 2.1.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Getting Started](#getting-started)
- [Pull Request Process](#pull-request-process)
- [Code Style](#code-style)

---

## Getting Started

Thank you for considering contributing...
```

### Nested Documentation

```markdown
# Database Schema

**Last Updated**: 24/12/2025
**Version**: 2.1.0
**Maintained By**: Database Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Tables](#tables)
- [Relationships](#relationships)

---

## Overview

This document describes the database schema...
```

---

## Files to Skip

The Version Agent skips these directories when updating headers:

- `node_modules/`
- `vendor/`
- `.git/`
- `build/`
- `dist/`
- `.next/`
- `__pycache__/`

---

## Adding Headers to Existing Files

When initialising version management on an existing project:

1. Run `/version init`
2. The Version Agent will:
   - Scan all `.md` files
   - Add headers to files missing them
   - Place headers after the main title
   - Set `Version` and `Last Updated` to current values

3. Review the changes before committing

---

## Customising the Maintained By Field

For large projects, you can customise the "Maintained By" field:

| Document Type | Maintained By |
|---------------|---------------|
| README.md | Development Team |
| API docs | Backend Team |
| Frontend docs | Frontend Team |
| Database docs | Database Team |
| DevOps docs | Platform Team |
| Security docs | Security Team |

Update this field manually when creating new documentation.

---

## Validation

To check if all files have valid headers, the Version Agent runs:

```bash
/version status
```

This shows:
- Files with valid headers
- Files missing headers
- Files with outdated version numbers
- Files with outdated dates
