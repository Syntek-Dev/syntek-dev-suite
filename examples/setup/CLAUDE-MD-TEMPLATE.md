# CLAUDE.md Template

## Overview

Template for the `.claude/CLAUDE.md` file that provides project context for Claude Code. This file should be created in every project.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 1.0.0 |
| **Last Updated** | 2025-01 |
| **Stacks** | All (TALL, Django, React, Mobile, Shared-Lib) |

---

## Table of Contents

- [CLAUDE.md Template](#claudemd-template)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [Template](#template)
  - [Settings File Template](#settings-file-template)
    - [.claude/settings.local.json](#claudesettingslocaljson)

---

## Template

```markdown
# [Project Name]

## Stack
- **Language:** [e.g., PHP 8.3, TypeScript 5.x, Python 3.12]
- **Framework:** [e.g., Laravel 11, Next.js 14, Django 5]
- **Database:** [e.g., MySQL 8, PostgreSQL 15]
- **Container:** [e.g., DDEV, Docker, Docker Compose]
- **Locale:** [e.g., en_GB]
- **Timezone:** [e.g., Europe/London]
- **Browser:** [e.g., Google Chrome, Chrome Beta for debugging]

## Project Structure
\`\`\`
[project root]/
├── .claude/              # Claude Code configuration
│   ├── CLAUDE.md         # This file
│   ├── settings.local.json
│   └── commands/         # Custom Claude commands
├── docs/                 # Documentation
│   ├── TESTS/           # Test specifications
│   ├── QA/              # QA reports
│   ├── PLANS/           # Implementation plans
│   └── DEVOPS/          # DevOps documentation
├── src/                  # Source code (or app/, lib/, etc.)
├── tests/                # Test files
└── [framework-specific directories]
\`\`\`

## Development Commands
\`\`\`bash
# Start development environment
./dev.sh

# Run tests
[test command for framework]

# Build for production
[build command for framework]
\`\`\`

## Code Conventions
- [List project-specific conventions]
- [Naming conventions]
- [File organisation rules]

## Environment Variables
See `.env.*.example` files for required configuration.

## API Endpoints (if applicable)
[Document key API endpoints or link to API docs]

## Database Schema (if applicable)
[Document key tables/models or link to schema docs]
```

---

## Settings File Template

### .claude/settings.local.json

```json
{
  "language": "[detected/specified language]",
  "locale": "[user-specified locale]",
  "timezone": "[user-specified timezone]",
  "permissions": {
    "allow": [
      "Read(**)",
      "Edit(**)",
      "Write(**)",
      "Bash(npm:*)",
      "Bash(composer:*)",
      "Bash(git:*)",
      "Bash(ddev:*)"
    ],
    "deny": []
  },
  "environment": {
    "envFiles": [
      ".env.dev",
      ".env.staging",
      ".env.production"
    ]
  }
}
```