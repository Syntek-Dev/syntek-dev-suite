# CLAUDE.md Template

## Overview

Template for the `.claude/CLAUDE.md` file that provides project context for Claude Code. This file should be created in every project.

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

## Coding Principles

All code in this project follows Rob Pike's 5 Rules of Programming and Linus Torvalds' Coding Rules.

**Core rules:**
- Measure before optimising — no speed hacks without profiling
- Simple algorithms and simple data structures over fancy ones
- Data structures dominate: get the data model right and the logic becomes obvious
- Short, focused functions that do one thing
- Eliminate special cases rather than patching them with `if` statements
- Make it work first, then make it better
- Favour stability and readability over cleverness

See `.claude/CODING-PRINCIPLES.md` for the full rules.

## Required Reference Documents

All agents MUST read these documents before writing code, tests, or performing reviews:

| Document | Purpose |
|----------|---------|
| `.claude/CODING-PRINCIPLES.md` | Coding standards, naming conventions, error handling, git workflow |
| `.claude/TESTING.md` | Testing patterns, tooling, and requirements for this stack |
| `.claude/SECURITY.md` | Security requirements, OWASP mitigations, and deployment checklist |
| `.claude/DEVELOPMENT.md` | Development workflow, environment setup, and common tasks |

## Project Structure
\`\`\`
[project root]/
├── .claude/              # Claude Code configuration
│   ├── CLAUDE.md         # This file
│   ├── settings.local.json
│   ├── plugins/          # Python plugin tools
│   │   └── *.py
│   └── commands/         # Custom Claude commands
├── docs/                 # Documentation
│   ├── TESTS/           # Test specifications
│   ├── QA/              # QA reports
│   ├── PLANS/           # Implementation plans
│   ├── DEVOPS/          # DevOps documentation
│   └── METRICS/         # Self-learning system data
├── src/                  # Source code (or app/, lib/, etc.)
├── tests/                # Test files
└── [framework-specific directories]
\`\`\`

## Plugin Tools

All agents should use the Python plugin tools in `.claude/plugins/` to gather context:

\`\`\`bash
# Project and framework detection
python3 .claude/plugins/project-tool.py info
python3 .claude/plugins/project-tool.py framework

# Database configuration
python3 .claude/plugins/db-tool.py detect

# Environment files
python3 .claude/plugins/env-tool.py find

# Git status
python3 .claude/plugins/git-tool.py status

# Container status (DDEV or Docker)
python3 .claude/plugins/ddev-tool.py status
python3 .claude/plugins/docker-tool.py status

# Log files
python3 .claude/plugins/log-tool.py find
\`\`\`

### Available Plugins

| Plugin | Purpose |
|--------|---------|
| `project-tool.py` | Language, framework, and structure detection |
| `db-tool.py` | Database type and ORM detection |
| `env-tool.py` | Environment file discovery and validation |
| `git-tool.py` | Git repository status, branches, remotes |
| `ddev-tool.py` | DDEV project status and configuration |
| `docker-tool.py` | Docker containers, compose, images, networks |
| `log-tool.py` | Log file discovery and analysis |
| `metrics-tool.py` | Self-learning metrics recording and querying |
| `feedback-tool.py` | User feedback collection and analysis |
| `quality-tool.py` | Code quality checks and linting |
| `chrome-tool.py` | Cross-platform Chrome detection |
| `pm-tool.py` | PM tool detection (ClickUp, Linear, Jira, etc.) |

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