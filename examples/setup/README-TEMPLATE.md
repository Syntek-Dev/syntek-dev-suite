# README Template

## Overview

Standard README.md template for project setup. Provides a consistent structure for project documentation.

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
- [Markdown Extension Compatibility](#markdown-extension-compatibility)
- [Template](#template)


---

## Markdown Extension Compatibility

This template is compatible with the **Markdown All in One** VS Code extension:

- **TOC Auto-Update**: The Table of Contents auto-updates on save
- **Table Formatting**: GFM tables auto-align on save
- **Task Lists**: Use `Alt+C` to toggle checkboxes

Install the extension for the best editing experience. See [docs/GUIDES/MARKDOWN-ALL-IN-ONE.md](../../docs/GUIDES/MARKDOWN-ALL-IN-ONE.md) for details.

---

## Template

```markdown
# [Project Name]

## Overview
[Brief description of the project]

## Quick Start

### Prerequisites
- [Container type] installed
- [Language] [version]

### Development Setup
\`\`\`bash
./dev.sh
\`\`\`

### Staging Deployment
\`\`\`bash
./staging.sh
\`\`\`

### Production Deployment
\`\`\`bash
./production.sh
\`\`\`

## Project Structure
| Directory            | Purpose                                |
| -------------------- | -------------------------------------- |
| `.claude/`           | Claude Code configuration and commands |
| `docs/`              | Project documentation                  |
| `docs/TESTS/`        | Test specifications and results        |
| `docs/TESTS/MANUAL/` | Manual testing guides                  |
| `docs/QA/`           | QA reports and test execution logs     |
| `docs/PLANS/`        | Implementation plans                   |
| `docs/DEVOPS/`       | CI/CD and deployment documentation     |

## Documentation
- [CLAUDE.md](.claude/CLAUDE.md) - Claude Code project context
- [docs/](docs/) - Full documentation

## Environment Setup
See `.env.*.example` files for required environment variables:
- `.env.dev.example` - Development environment
- `.env.staging.example` - Staging environment
- `.env.production.example` - Production environment

## Available Commands
See `.claude/commands/` for available Claude Code commands.
```