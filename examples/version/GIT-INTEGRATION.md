# Version Agent Git Integration

**Last Updated**: 24/12/2025
**Version**: 1.2.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Integration Flow](#integration-flow)
- [Git Agent Responsibilities](#git-agent-responsibilities)
- [Version Agent Responsibilities](#version-agent-responsibilities)
- [Commit Workflow](#commit-workflow)
- [Examples](#examples)

---

## Overview

The Git Agent and Version Agent work together to ensure that every commit includes proper version updates. The Git Agent is responsible for determining the version increment type and delegating version updates to the Version Agent before creating the commit.

---

## Integration Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    /git commit                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  1. Git Agent analyses staged changes                           │
│     - Reviews modified files                                    │
│     - Determines change type (feature, fix, breaking)           │
│     - Decides version increment (MAJOR, MINOR, PATCH)           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. Git Agent calls Version Agent                               │
│     /version bump <type>                                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. Version Agent updates:                                      │
│     - All version files (package.json, etc.)                    │
│     - VERSION-HISTORY.md                                        │
│     - CHANGELOG.md                                              │
│     - RELEASES.md                                               │
│     - All .md file headers (Version + Last Updated)             │
│     - Stages all changes                                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. Git Agent creates commit                                    │
│     - Includes version in commit message                        │
│     - All version files staged                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Git Agent Responsibilities

The Git Agent (`/git`) handles:

| Task | Description |
|------|-------------|
| **Analyse changes** | Review staged files to understand the scope |
| **Determine increment** | Decide if MAJOR, MINOR, or PATCH based on changes |
| **Call Version Agent** | Delegate version updates to `/version bump` |
| **Create commit** | Write commit message with version info |
| **Create PRs** | Handle pull request creation |
| **Branch management** | Create, merge, and manage branches |

### Version Increment Decision Rules

| Change Type | Increment | Examples |
|-------------|-----------|----------|
| Breaking changes | MAJOR | API removed, schema incompatible |
| New features | MINOR | New endpoint, new UI component |
| Bug fixes | PATCH | Fix crash, correct calculation |
| Documentation only | PATCH | README update, comment changes |
| Dependencies | PATCH (usually) | Unless breaking changes |

---

## Version Agent Responsibilities

The Version Agent (`/version`) handles:

| Task | Description |
|------|-------------|
| **Update version files** | package.json, composer.json, etc. |
| **Update VERSION-HISTORY.md** | Technical changelog with file details |
| **Update CHANGELOG.md** | Developer summary in Keep a Changelog format |
| **Update RELEASES.md** | User-facing release notes |
| **Update .md headers** | Version and Last Updated in all markdown |
| **Stage changes** | Git add all modified version files |

### What Version Agent Does NOT Do

- ❌ Create commits (Git Agent handles this)
- ❌ Decide version increment type (Git Agent decides)
- ❌ Push to remote (Git Agent handles this)
- ❌ Create PRs (Git Agent handles this)

---

## Commit Workflow

### Step-by-Step Process

1. **User runs `/git commit`**

2. **Git Agent analyses changes:**
   ```
   Analysing staged changes...
   - app/Http/Controllers/UserController.php (new file)
   - app/Models/User.php (modified)
   - routes/api.php (modified)

   Change type: New feature (user management API)
   Recommended increment: MINOR
   ```

3. **Git Agent calls Version Agent:**
   ```
   Calling /version bump minor...
   ```

4. **Version Agent updates files:**
   ```
   Updating version files...
   - package.json: 1.2.0 → 1.3.0 ✓
   - composer.json: 1.2.0 → 1.3.0 ✓

   Updating VERSION-HISTORY.md... ✓
   Updating CHANGELOG.md... ✓
   Updating RELEASES.md... ✓

   Updating markdown headers...
   - Updated 24 .md files ✓

   Staging changes... ✓
   ```

5. **Git Agent creates commit:**
   ```
   Creating commit...

   feat(users): Add user management API - CRUD operations for users

   Added complete user management functionality including:
   - User CRUD operations
   - Role assignment
   - Permission checks

   Files Changed:
   - app/Http/Controllers/UserController.php
   - app/Models/User.php
   - routes/api.php

   Version: 1.2.0 → 1.3.0
   ```

---

## Examples

### Example 1: Feature Commit

```bash
# User runs
/git commit

# Git Agent output
Analysing staged changes...

Files staged:
- app/Services/PaymentService.php (new)
- app/Http/Controllers/PaymentController.php (new)
- routes/api.php (modified)
- database/migrations/2025_01_24_create_payments_table.php (new)

Change analysis:
- New PaymentService with Stripe integration
- New PaymentController with 4 endpoints
- New payments table migration

Recommended: MINOR version bump (new feature, backwards compatible)

Calling Version Agent...
/version bump minor

[Version Agent updates all files]

Creating commit with message:

feat(payments): Add Stripe payment integration - Process payments via Stripe API

Implemented complete payment processing:
- Stripe API integration in PaymentService
- Payment endpoints (create, confirm, refund, webhook)
- Payments database table

Files Changed:
- app/Services/PaymentService.php
- app/Http/Controllers/PaymentController.php
- routes/api.php
- database/migrations/2025_01_24_create_payments_table.php

Version: 1.3.0 → 1.4.0
```

### Example 2: Bug Fix Commit

```bash
# User runs
/git commit

# Git Agent output
Analysing staged changes...

Files staged:
- app/Services/ReportScheduler.php (modified)
- tests/Unit/ReportSchedulerTest.php (new)

Change analysis:
- Fixed timezone calculation in getNextRunTime()
- Added regression test

Recommended: PATCH version bump (bug fix)

Calling Version Agent...
/version bump patch

[Version Agent updates all files]

Creating commit with message:

fix(reports): Correct timezone handling in scheduled reports

Fixed timezone offset calculation that caused reports to run
at incorrect times for users outside UTC timezone.

Added regression test to prevent future issues.

Files Changed:
- app/Services/ReportScheduler.php
- tests/Unit/ReportSchedulerTest.php

Version: 1.4.0 → 1.4.1
```

### Example 3: Breaking Change Commit

```bash
# User runs
/git commit

# Git Agent output
Analysing staged changes...

Files staged:
- app/Http/Controllers/AuthController.php (modified)
- config/auth.php (modified)
- routes/api.php (modified)

Change analysis:
- AuthController rewritten for JWT
- Session-based auth removed
- API response format changed

⚠️ BREAKING CHANGES DETECTED:
- /api/auth/login response structure changed
- Session cookies no longer set
- Bearer token now required for all authenticated endpoints

Recommended: MAJOR version bump (breaking changes)

Proceed with MAJOR bump? [Y/n]

Calling Version Agent...
/version bump major

[Version Agent updates all files, including migration guide in VERSION-HISTORY.md]

Creating commit with message:

feat(auth)!: Migrate from sessions to JWT authentication

BREAKING CHANGE: Authentication now uses JWT tokens instead of sessions.

Migration required:
1. Update API calls to include Authorization header
2. Store JWT token client-side
3. Remove session cookie handling

Files Changed:
- app/Http/Controllers/AuthController.php
- config/auth.php
- routes/api.php

Version: 1.4.1 → 2.0.0
```

---

## Configuration

### Automatic Version Bumping

The Git Agent should automatically determine the version bump type unless:
- The user explicitly specifies: `/git commit --patch`
- Breaking changes detected: prompt user to confirm MAJOR
- Multiple change types: prompt user to choose

### Skip Version Bump

For commits that shouldn't bump version (rare cases):

```bash
/git commit --no-version
```

Use sparingly for:
- CI/CD configuration only
- Git ignore updates
- IDE settings

---

## Troubleshooting

### Version Files Out of Sync

If version files have different versions:
1. Run `/version status` to identify mismatches
2. Run `/version update` to synchronise all files
3. Run `/git commit` to commit the fix

### Missing Version History Entry

If VERSION-HISTORY.md is missing entries:
1. Run `/version history` to see gaps
2. Manually add entries or run `/version update`
3. Commit the updates

### Headers Not Updating

If markdown headers aren't updating:
1. Check if files are in excluded directories
2. Run `/version headers` to force update
3. Verify file has correct header format
