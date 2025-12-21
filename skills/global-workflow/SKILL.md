# Global Workflow & Standards

This skill defines project-wide standards. All agents MUST apply these settings.

---

## Table of Contents

- [1. Localisation](#1-localisation)
- [2. Repository Setup](#2-repository-setup)
- [3. Branch Strategy](#3-branch-strategy)
- [4. Git Protocol](#4-git-protocol)
- [5. Pull Request Management](#5-pull-request-management)
- [6. Semantic Versioning](#6-semantic-versioning)
- [7. Documentation Standards](#7-documentation-standards)
- [8. Code Comment Standards](#8-code-comment-standards)
- [9. Self-Learning System](#9-self-learning-system)

---

## 1. Localisation

| Setting | Value |
|---------|-------|
| **Locale** | British English (en-GB) |
| **Spelling** | Use 's' not 'z' (*optimise*, *organise*, *behaviour*) |
| **Vocabulary** | *postcode*, *CV*, *holiday*, *mobile* |
| **Currency** | GBP (£) with format `£1,234.56` |
| **Date Format** | DD/MM/YYYY |
| **Time Format** | 24-hour clock (HH:MM) |
| **Timezone** | Europe/London |

**Code Syntax Exception:** Keep US English for reserved words (CSS `color`, PHP `function`), but use GB English for variable names (`$colourPalette`) and user-facing copy.

---

## 2. Repository Setup

### SSH Cloning Requirement

**CRITICAL:** All developers MUST clone repositories using SSH, not HTTPS.

```bash
# CORRECT - Use SSH
git clone git@github.com:organisation/repository.git

# INCORRECT - Do NOT use HTTPS
git clone https://github.com/organisation/repository.git
```

**Why SSH:**
- Enables proper commit author tracking
- Simplifies authentication (no password prompts)
- Required for signed commits
- Better security with key-based authentication

### Setting Up SSH

1. Generate SSH key (if not already done):
   ```bash
   ssh-keygen -t ed25519 -C "your.email@example.com"
   ```

2. Add SSH key to ssh-agent:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```

3. Add public key to GitHub:
   - Copy: `cat ~/.ssh/id_ed25519.pub`
   - Add at: GitHub → Settings → SSH and GPG keys → New SSH key

4. Test connection:
   ```bash
   ssh -T git@github.com
   ```

### Converting HTTPS to SSH

If a repository was cloned with HTTPS, convert it:
```bash
git remote set-url origin git@github.com:organisation/repository.git
```

---

## 3. Branch Strategy

### Required Core Branches

Every project MUST have these protected branches:

| Branch | Purpose | Protected | Deploy Target |
|--------|---------|-----------|---------------|
| `main` or `master` | Production-ready code | Yes | Production |
| `staging` | Client review and acceptance | Yes | Staging |
| `dev` | Integration and final testing | Yes | Development |
| `testing` | QA and automated testing | Yes | Testing |

### User Story Branch Naming

All feature work uses user story branches:

```
us<number>/<short-description>
```

**Examples:**
- `us001/user-authentication`
- `us015/payment-integration`
- `us042/api-rate-limiting`

**Rules:**
- User story number is zero-padded to 3 digits (us001, us042, us100)
- Description uses kebab-case (lowercase with hyphens)
- Description should be 2-4 words maximum

### Hotfix Branch Naming

For critical production fixes:

```
hotfix/<issue-number>-<short-description>
```

**Examples:**
- `hotfix/123-login-crash`
- `hotfix/456-payment-timeout`

### Branch Flow

```
us###/feature → testing → dev → staging → main
```

| From | To | Condition | On Rejection |
|------|-----|-----------|--------------|
| `us###/feature` | `testing` | Developer tests pass | Fix in feature branch |
| `testing` | `dev` | QA tests pass | Fix and re-submit |
| `dev` | `staging` | Integration tests pass | Fix and re-submit |
| `staging` | `main` | **Client accepts** | Back to `us###/feature` |

---

## 4. Git Protocol

### Commit Message Template

```
<type>(<scope>): <Description> - <Summarise>

<Body - What was changed and why>

Files Changed:
- <app-name/folder/file>

Still to do:
- <Task 1>
- <Task 2>

Version: <old-version> → <new-version>
```

### Commit Types

| Type | Use For |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code restructure, no feature change |
| `test` | Adding/updating tests |
| `chore` | Build, config, dependencies |

### Commit Rules

1. **Imperative mood:** "Add feature" not "Added feature"
2. **Version update:** Include version change in commit
3. **Changelog first:** Update CHANGELOG.md before committing
4. **No filler:** Output ONLY the commit message code block

---

## 5. Pull Request Management

### Using GitHub CLI (gh)

**CRITICAL:** All PR operations MUST use the `gh` CLI tool for consistency and automation.

#### Creating PRs

```bash
# Create PR from current branch to testing
gh pr create --base testing --title "[US001] feat(auth): Add login" --body "..."

# Create PR with template
gh pr create --base testing --title "[US001] feat(auth): Add login" --body-file .github/PULL_REQUEST_TEMPLATE.md
```

#### Reviewing PRs

```bash
# List open PRs
gh pr list

# View PR details
gh pr view <number>

# View PR diff
gh pr diff <number>

# Add review comment
gh pr review <number> --comment --body "Comment text"

# Approve PR
gh pr review <number> --approve --body "Approved: All tests pass"

# Request changes
gh pr review <number> --request-changes --body "Needs fixes: ..."
```

#### Managing PRs

```bash
# Merge PR (after approval)
gh pr merge <number> --merge

# Close PR without merging (rejected)
gh pr close <number> --comment "Rejected: Reason..."

# Reopen PR
gh pr reopen <number>

# Check PR status
gh pr status
```

### PR Title Format

```
[<branch-type>] <type>(<scope>): <Description>
```

**Examples:**
- `[US001] feat(auth): Add user login functionality`
- `[HOTFIX] fix(payment): Resolve checkout timeout`
- `[TESTING→DEV] feat(auth): User authentication module`

### PR Body Template

```markdown
## Summary

- <Bullet point 1: What this PR does>
- <Bullet point 2: Key changes>
- <Bullet point 3: Any notable decisions>

## Changes

### Added
- <New feature or file>

### Changed
- <Modified functionality>

### Fixed
- <Bug fix>

## Version

**Previous:** `X.Y.Z`
**New:** `X.Y.Z`
**Increment Type:** MAJOR | MINOR | PATCH

## Commits Included

| Commit | Type | Description |
|--------|------|-------------|
| abc123 | feat | Add login form |
| def456 | fix | Correct validation |

## Test Plan

- [ ] <How to test change 1>
- [ ] <How to test change 2>
- [ ] Unit tests pass
- [ ] Integration tests pass

## Checklist

- [ ] Code follows project style guide
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version files updated
- [ ] No secrets or credentials in code

## Related Issues

Closes #<issue-number>
```

### Client Acceptance/Rejection (Staging → Main)

#### Accepting Changes

```bash
# Approve and merge to main
gh pr review <number> --approve --body "Client accepted: Ready for production"
gh pr merge <number> --merge
```

#### Rejecting Changes

```bash
# Request changes with rejection reason
gh pr review <number> --request-changes --body "Client rejected: <detailed reason>"

# Close the PR
gh pr close <number> --comment "Rejected by client. Creating fix branch from original US branch."
```

After rejection:
1. Create new branch from original `us###/feature`
2. Implement fixes
3. Re-submit through full flow: `testing` → `dev` → `staging`

---

## 6. Semantic Versioning

### Version Format

```
MAJOR.MINOR.PATCH
```

| Type | When to Increment | Example |
|------|-------------------|---------|
| **MAJOR** | Breaking changes | 1.0.0 → 2.0.0 |
| **MINOR** | New features (backwards compatible) | 1.0.0 → 1.1.0 |
| **PATCH** | Bug fixes (backwards compatible) | 1.0.0 → 1.0.1 |

### Pre-Commit Requirements

**CRITICAL:** Before EVERY commit:

1. Determine version increment (MAJOR/MINOR/PATCH)
2. Update all version files
3. Update CHANGELOG.md
4. Stage version and changelog files

---

## 7. Documentation Standards

| Standard | Value |
|----------|-------|
| **Format** | Markdown (`*.md`) |
| **Location** | `docs/` folder |
| **Filenames** | CAPITALISED with lowercase `.md` extension |
| **Required** | Table of Contents in all docs |

### Bug Fix Documentation

**CRITICAL:** All bug fixes (on user story branches or hotfix branches) MUST be documented in `docs/BUGS/`.

See the `/debug` agent for the required documentation format. Bug documentation MUST include:
- Root cause analysis
- How the fix resolves the issue
- Prevention recommendations
- Regression test requirements

---

## 8. Code Comment Standards

All agents writing code MUST follow these documentation rules:

### File Headers

Every file MUST have a header comment explaining its purpose.

### Docstrings

Every public function/method MUST have a docstring with:
- Description of what the function does
- Parameters with types
- Return value with type
- Exceptions thrown

### No Pronouns Rule

**CRITICAL:** Never use pronouns referring to code in comments.

| Do | Don't |
|-------|----------|
| `The function validates input` | `It validates the input` |
| `Returns the user object` | `Returns this` |
| `The service handles authentication` | `We handle auth here` |

---

## 9. Self-Learning System

### Feedback Collection

After completing tasks, the system prompts for feedback. All agents should be aware of this system.

### Feedback Criteria

Help users give quick, meaningful feedback with these guidelines:

| Rating | When to Use | Examples |
|--------|-------------|----------|
| **good** | Task completed successfully | Code works, follows patterns, well documented |
| **bad** | Task needs improvement | Missing requirements, wrong patterns, errors |
| **skip** | Can't evaluate yet | Need to test more, partial completion |

### Quick Decision Guide

**Rate "good" if ALL apply:**
- The code/output meets your request
- It follows project patterns and conventions
- No obvious errors or missing pieces
- Documentation is adequate

**Rate "bad" if ANY apply:**
- Output doesn't match what you asked for
- Code has errors or won't work
- Missing important security or validation
- Wrong file structure or naming
- Had to make significant changes after

**Rate "skip" if:**
- Haven't tested the output yet
- Task was cancelled or interrupted
- Not sure if it's correct

### Feedback Commands

```
/learning:feedback good                    # Quick positive
/learning:feedback bad                     # Quick negative
/learning:feedback bad Missing validation  # With comment
/learning:feedback skip                    # Skip this time
```

### Why Feedback Matters

- All feedback is stored in `docs/METRICS/` and committed to Git
- The entire team's feedback improves agent prompts
- Auto-optimisation uses feedback patterns to enhance agents
- New team members benefit from accumulated improvements

### Agent Responsibilities

When completing a task, agents should:
1. Ensure output meets the feedback criteria for "good"
2. If unsure about requirements, ask before implementing
3. Follow project patterns to avoid "bad" ratings
