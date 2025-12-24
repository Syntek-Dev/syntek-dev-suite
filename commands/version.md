---
description: "[Agent] Version management - semantic versioning, changelogs, and markdown headers"
usage: /agent:version [command] [args]
---

Spawn the `dev-team:version` agent (model: sonnet) to manage version files and documentation.

## Pre-flight: Run Plugin Tools

Before performing version operations, gather context:
```bash
# Check repository and current version
python plugins/git-tool.py status
python plugins/git-tool.py tags
python plugins/project-tool.py info
```

## Available Commands

| Command | Description | Example |
|---------|-------------|---------|
| `bump` | Increment version (major, minor, patch) | `/version bump minor` |
| `update` | Update all version files and docs | `/version update` |
| `headers` | Update metadata headers in all .md files | `/version headers` |
| `init` | Initialise version files for new project | `/version init` |
| `status` | Show current version and pending changes | `/version status` |
| `history` | Show version history summary | `/version history` |

## Version Files Managed

| File | Purpose | Audience |
|------|---------|----------|
| **Version files** | Semantic version number (package.json, etc.) | Build systems |
| **VERSION-HISTORY.md** | Technical change log with code details | Developers |
| **CHANGELOG.md** | Brief developer-focused summary | Developers |
| **RELEASES.md** | User-facing feature highlights | End users |

## Markdown Metadata Headers

All `.md` files include a metadata header that the version agent maintains:

```markdown
# Document Title

**Last Updated**: DD/MM/YYYY
**Version**: X.Y.Z
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London
```

## Semantic Versioning

| Type | When | Example |
|------|------|---------|
| MAJOR | Breaking changes | 1.0.0 → 2.0.0 |
| MINOR | New features | 1.0.0 → 1.1.0 |
| PATCH | Bug fixes | 1.0.0 → 1.0.1 |

## Usage Examples

### Bump Version
```
/version bump minor
```
Increments the minor version (e.g., 1.2.0 → 1.3.0), updates all version files, VERSION-HISTORY.md, CHANGELOG.md, RELEASES.md, and markdown headers.

### Initialise Version Management
```
/version init
```
Creates VERSION-HISTORY.md, RELEASES.md, and adds metadata headers to all .md files.

### Update Headers Only
```
/version headers
```
Updates the `Version` and `Last Updated` fields in all markdown file headers.

### Check Current Status
```
/version status
```
Shows current version, pending changes, and files that need updating.

## Integration with Git

The Git Agent automatically calls the Version Agent before commits:

1. `/git commit` is run
2. Git Agent determines version increment type
3. Git Agent calls `/version bump <type>`
4. Version Agent updates all files
5. Git Agent creates the commit

## Localisation

All dates use DD/MM/YYYY format and British English spelling.

**User's Request:**
$ARGUMENTS
