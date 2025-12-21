---
description: "[Agent] Git workflow management - branches, commits, PRs, and versioning"
usage: /agent:git [command] [args]
---

Spawn the `dev-team:git` agent (model: sonnet) to manage git workflow.

## Pre-flight: Run Plugin Tools

Before performing git operations, gather context using these plugin tools:
```bash
# Check repository status
python plugins/git-tool.py status
python plugins/git-tool.py branches --all
python plugins/git-tool.py tags
python plugins/git-tool.py host
python plugins/git-tool.py commits 10
```

## Available Commands

| Command | Description | Example |
|---------|-------------|---------|
| `init` | Initialise branch structure (main, staging, dev, testing) | `/git init` |
| `branch` | Create a user story branch | `/git branch us001 user-login` |
| `commit` | Create commit with version and changelog update | `/git commit` |
| `pr` | Create pull request to target branch | `/git pr testing` |
| `status` | Show branch status and pending PRs | `/git status` |
| `version` | Show/update version (major, minor, patch) | `/git version minor` |
| `changelog` | Update changelog for current changes | `/git changelog` |
| `flow` | Show the branch flow diagram | `/git flow` |

## Branch Strategy

The agent manages a structured branch flow:

```
us###/feature → testing → dev → staging → main
```

| Branch | Purpose | Deploy Target |
|--------|---------|---------------|
| `main` | Production-ready code | Production |
| `staging` | Client review/acceptance | Staging |
| `dev` | Integration testing | Development |
| `testing` | QA verification | Testing |
| `us###/name` | Feature work | Local |

## User Story Branch Naming

All feature branches follow this format:
```
us<number>/<short-description>
```

Examples:
- `us001/user-authentication`
- `us015/payment-integration`
- `us042/api-rate-limiting`

## Commit Message Format

The agent uses the commit message template from global-workflow:

```
<type>(<scope>): <Description> - <Summarise>

<Body - What was changed and why>

Files Changed:
- <app-name/folder/file>

Still to do:
- <Task 1>

Version: <old> → <new>
```

## Pull Request Flow

| From | To | Condition |
|------|-----|-----------|
| `us###/feature` | `testing` | Developer tests pass |
| `testing` | `dev` | QA tests pass |
| `dev` | `staging` | Integration tests pass |
| `staging` | `main` | **Client accepts** |

### On Client Rejection

If the client rejects on staging:
1. Changes go back to original `us###/feature` branch
2. Fixes are made
3. Re-submit through full flow: testing → dev → staging

## Version Management

Before every commit, the agent:
1. Analyses changes for version impact
2. Determines increment type (MAJOR/MINOR/PATCH)
3. Updates all version files
4. Updates CHANGELOG.md
5. Includes version files in commit

### Version Increment Rules

| Type | When | Example |
|------|------|---------|
| MAJOR | Breaking changes | 1.0.0 → 2.0.0 |
| MINOR | New features | 1.0.0 → 1.1.0 |
| PATCH | Bug fixes | 1.0.0 → 1.0.1 |

## Usage Examples

### Initialise a New Project
```
/git init
```
Creates: main, staging, dev, testing branches with protection rules.

### Start a New User Story
```
/git branch us042 shopping-cart
```
Creates: `us042/shopping-cart` branch from `dev`.

### Commit Changes
```
/git commit
```
Prompts for:
- Commit type and scope
- Description
- Version increment type
Updates changelog and version files automatically.

### Create PR to Testing
```
/git pr testing
```
Creates PR from current user story branch to `testing`.

### Check Current Status
```
/git status
```
Shows: current branch, pending changes, open PRs, version info.

### Update Version Manually
```
/git version patch
```
Increments patch version and updates all version files.

### Show Branch Flow
```
/git flow
```
Displays the branch flow diagram and current position.

## Localisation

All dates use DD/MM/YYYY format and British English spelling in:
- Commit messages
- PR descriptions
- Changelog entries

**User's Request:**
$ARGUMENTS
