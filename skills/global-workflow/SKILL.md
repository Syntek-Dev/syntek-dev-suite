# Global Workflow & Standards

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [1. Localisation](#1-localisation)
- [2. Repository Setup](#2-repository-setup)
  - [SSH Cloning Requirement](#ssh-cloning-requirement)
  - [Setting Up SSH](#setting-up-ssh)
  - [Converting HTTPS to SSH](#converting-https-to-ssh)
- [3. Branch Strategy](#3-branch-strategy)
  - [Required Core Branches](#required-core-branches)
  - [User Story Branch Naming](#user-story-branch-naming)
  - [Hotfix Branch Naming](#hotfix-branch-naming)
  - [Branch Flow](#branch-flow)
- [4. Git Protocol](#4-git-protocol)
  - [Commit Message Template](#commit-message-template)
  - [Commit Types](#commit-types)
  - [Commit Rules](#commit-rules)
- [5. Pull Request Management](#5-pull-request-management)
  - [Using GitHub CLI (gh)](#using-github-cli-gh)
    - [Creating PRs](#creating-prs)
    - [Reviewing PRs](#reviewing-prs)
    - [Managing PRs](#managing-prs)
  - [PR Title Format](#pr-title-format)
  - [PR Body Template](#pr-body-template)
  - [Client Acceptance/Rejection (Staging → Main)](#client-acceptancerejection-staging--main)
    - [Accepting Changes](#accepting-changes)
    - [Rejecting Changes](#rejecting-changes)
- [6. Semantic Versioning](#6-semantic-versioning)
  - [Version Format](#version-format)
  - [Pre-Commit Requirements](#pre-commit-requirements)
- [7. Documentation Standards](#7-documentation-standards)
  - [Markdown Generation Guidelines](#markdown-generation-guidelines)
    - [Table of Contents](#table-of-contents-1)
    - [Text Formatting](#text-formatting)
    - [Lists](#lists)
    - [Tables](#tables)
    - [Code Blocks](#code-blocks)
    - [Headings](#headings)
    - [Horizontal Rules](#horizontal-rules)
    - [Best Practices](#best-practices)
  - [Bug Fix Documentation](#bug-fix-documentation)
- [8. Code Comment Standards](#8-code-comment-standards)
  - [File Headers](#file-headers)
  - [Docstrings](#docstrings)
  - [No Pronouns Rule](#no-pronouns-rule)


---

## 1. Localisation

| Setting         | Value                                                 |
| --------------- | ----------------------------------------------------- |
| **Locale**      | British English (en-GB)                               |
| **Spelling**    | Use 's' not 'z' (*optimise*, *organise*, *behaviour*) |
| **Vocabulary**  | *postcode*, *CV*, *holiday*, *mobile*                 |
| **Currency**    | GBP (£) with format `£1,234.56`                       |
| **Date Format** | DD/MM/YYYY                                            |
| **Time Format** | 24-hour clock (HH:MM)                                 |
| **Timezone**    | Europe/London                                         |

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

| Branch             | Purpose                       | Protected | Deploy Target |
| ------------------ | ----------------------------- | --------- | ------------- |
| `main` or `master` | Production-ready code         | Yes       | Production    |
| `staging`          | Client review and acceptance  | Yes       | Staging       |
| `dev`              | Integration and final testing | Yes       | Development   |
| `testing`          | QA and automated testing      | Yes       | Testing       |

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

| From            | To        | Condition              | On Rejection            |
| --------------- | --------- | ---------------------- | ----------------------- |
| `us###/feature` | `testing` | Developer tests pass   | Fix in feature branch   |
| `testing`       | `dev`     | QA tests pass          | Fix and re-submit       |
| `dev`           | `staging` | Integration tests pass | Fix and re-submit       |
| `staging`       | `main`    | **Client accepts**     | Back to `us###/feature` |

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

| Type       | Use For                             |
| ---------- | ----------------------------------- |
| `feat`     | New feature                         |
| `fix`      | Bug fix                             |
| `docs`     | Documentation only                  |
| `style`    | Formatting, no logic change         |
| `refactor` | Code restructure, no feature change |
| `test`     | Adding/updating tests               |
| `chore`    | Build, config, dependencies         |

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

| Commit | Type | Description        |
| ------ | ---- | ------------------ |
| abc123 | feat | Add login form     |
| def456 | fix  | Correct validation |

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

| Type      | When to Increment                   | Example       |
| --------- | ----------------------------------- | ------------- |
| **MAJOR** | Breaking changes                    | 1.0.0 → 2.0.0 |
| **MINOR** | New features (backwards compatible) | 1.0.0 → 1.1.0 |
| **PATCH** | Bug fixes (backwards compatible)    | 1.0.0 → 1.0.1 |

### Pre-Commit Requirements

**CRITICAL:** Before EVERY commit:

1. Determine version increment (MAJOR/MINOR/PATCH)
2. Update all version files
3. Update CHANGELOG.md
4. Stage version and changelog files

---

## 7. Documentation Standards

| Standard      | Value                                            |
| ------------- | ------------------------------------------------ |
| **Format**    | Markdown (`*.md`)                                |
| **Location**  | `docs/` folder                                   |
| **Filenames** | CAPITALISED with lowercase `.md` extension       |
| **Required**  | Table of Contents in all docs                    |
| **Extension** | Markdown All in One (yzhang.markdown-all-in-one) |

### Markdown Generation Guidelines

When generating markdown documentation, follow these standards for compatibility with the Markdown All in One extension:

#### Table of Contents

1. **Placement**: After document title and introduction, before main content
2. **Format**: Unordered list with `-` marker
3. **Heading**: Use `## Table of Contents`
4. **Indentation**: 2 spaces per nesting level
5. **Links**: Use anchor link format `[Text](#heading-slug)`
6. **Slug Format**: Lowercase, hyphens for spaces, remove special chars

**Example:**
```markdown
## Table of Contents

- [Main Title](#main-title)
  - [Table of Contents](#table-of-contents)
  - [Section 1](#section-1)
    - [Subsection 1.1](#subsection-11)
```

#### Text Formatting

| Format            | Syntax     | Example          |
| ----------------- | ---------- | ---------------- |
| **Bold**          | `**text**` | `**important**`  |
| *Italic*          | `*text*`   | `*emphasise*`    |
| ~~Strikethrough~~ | `~~text~~` | `~~deprecated~~` |
| `Inline code`     | \`code\`   | \`function()\`   |

#### Lists

**Unordered Lists:**
- Use `-` marker consistently
- 2 spaces for indentation

```markdown
- Item 1
  - Nested item 1.1
  - Nested item 1.2
- Item 2
```

**Ordered Lists:**
- Use `1.` marker for all items (auto-renumbering)
- 3 spaces for indentation

```markdown
1. First step
   1. Sub-step 1.1
   1. Sub-step 1.2
1. Second step
```

**Task Lists:**
- Use for checklists and action items
- Format: `- [ ]` for incomplete, `- [x]` for complete

```markdown
- [ ] Task to do
- [x] Completed task
```

#### Tables

Use GitHub Flavoured Markdown table syntax:

```markdown
| Column 1 | Column 2 | Column 3 |
| -------- | -------- | -------- |
| Value 1  | Value 2  | Value 3  |
```

**Alignment:**
- Left: `|:---------|`
- Centre: `|:--------:|`
- Right: `|---------:|`

#### Code Blocks

**Fenced Code Blocks:**
Always specify language for syntax highlighting:

```markdown
\`\`\`typescript
const example = "code";
\`\`\`
```

**Common Languages:**
`python`, `typescript`, `javascript`, `bash`, `json`, `sql`, `php`, `dart`, `html`, `css`

#### Headings

**Hierarchy:**
- `# Title` - Document title (once per file)
- `## Section` - Main sections
- `### Subsection` - Subsections
- `####` and beyond - As needed (avoid excessive nesting)

**Excluding from TOC:**
Add `<!-- omit in toc -->` to exclude heading:
```markdown
## Internal Notes <!-- omit in toc -->
```

#### Horizontal Rules

Use `---` for horizontal rules between major sections.

#### Best Practices

1. **Consistency**: Use same syntax throughout (e.g., `**bold**` not `__bold__`)
2. **British English**: Follow CLAUDE.md localisation settings
3. **Blank Lines**: Add blank lines before/after headings, code blocks, tables
4. **Line Length**: Keep lines under 120 characters where practical
5. **TOC Inclusion**: Always include TOC section; extension handles updates for users

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

| Do                                   | Don't                    |
| ------------------------------------ | ------------------------ |
| `The function validates input`       | `It validates the input` |
| `Returns the user object`            | `Returns this`           |
| `The service handles authentication` | `We handle auth here`    |

