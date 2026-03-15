# CLAUDE.md Template

## Overview

Template for the `.claude/CLAUDE.md` file that provides project context for Claude Code. This file should be created in every project.

## Metadata

| Property            | Value                                         |
| ------------------- | --------------------------------------------- |
| **Example Version** | 1.9.0                                         |
| **Last Updated**    | 15/03/2026                                    |
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

All code in this project follows the principles in `.claude/CODING-PRINCIPLES.md`, organised in layers:

- **Low-level coding instincts:** Rob Pike's 5 Rules and Linus Torvalds' Coding Rules — measure before optimising, simple data structures over fancy algorithms, data models drive logic, short focused functions, eliminate special cases, favour stability over cleverness
- **Class and module design:** SOLID Principles and GRASP Patterns — single responsibility, dependency inversion, information expert, high cohesion, low coupling
- **Quality check:** CUPID Properties — composable, Unix philosophy, predictable, idiomatic, domain-based
- **System boundaries and naming:** Domain-Driven Design — ubiquitous language, bounded contexts, aggregates, domain events, anti-corruption layers
- **Package and monorepo architecture:** Package Principles — reuse-release equivalence, common closure, acyclic dependencies, stable dependencies
- **Deployment and composition:** The Unix Philosophy and The Twelve-Factor App — do one thing well, config in environment, stateless processes, dev/prod parity
- **Everyday decisions:** DRY (Rule of Three), KISS, YAGNI, and Kent Beck's Four Rules of Simple Design — passes tests, reveals intention, no duplication, fewest elements

Additional practical sections cover the 750-line file limit, error handling, naming conventions, testing requirements, comments and documentation, security, dependencies, git workflow, logging, and a code review checklist.

See `.claude/CODING-PRINCIPLES.md` for the full rules.

## Required Reference Documents

All agents MUST read these documents before writing code, tests, or performing reviews:

| Document | Purpose |
|----------|---------|
| `.claude/CODING-PRINCIPLES.md` | Software design principles (SOLID, CUPID, GRASP, DDD, Unix Philosophy, Twelve-Factor), coding instincts (Pike, Torvalds), everyday rules (DRY, KISS, YAGNI, Beck's Four Rules), naming conventions, error handling, logging, git workflow, and code review checklist |
| `.claude/TESTING.md` | Testing matrix per layer (Python/Django, TypeScript/React, React Native, GraphQL), database isolation, migration testing, factories, property-based testing, coverage thresholds, mocking philosophy, snapshot testing, boundary testing, accessibility testing, performance testing, flaky test policy, and CI integration |
| `.claude/SECURITY.md` | Secrets management, authentication (argon2id/scrypt/bcrypt hierarchy), cryptography standards (approved and banned algorithms), transport security, API and GraphQL security, file upload and browser storage policies, container security, OWASP Top 10:2025 mitigations, supply chain security, data classification, security logging, and incident response |
| `.claude/ACCESSIBILITY.md` | WCAG 2.2 AA compliance, semantic HTML, ARIA patterns, keyboard navigation, focus management, forms, colour contrast, images and media, motion, React component patterns, React Native mobile accessibility, server-rendered accessibility (Django/Laravel), testing, and checklist |
| `.claude/API-DESIGN.md` | REST conventions (URL structure, HTTP methods, status codes, pagination, filtering), GraphQL conventions (schema, queries, mutations, error unions), error response format, authentication and authorisation, rate limiting, versioning, webhooks, API documentation, and client-side consumption patterns |
| `.claude/ARCHITECTURE-PATTERNS.md` | Service layer, middleware and request pipeline, frontend state management, routing conventions, background job patterns, email and notification patterns, file processing pipelines, and project structure (Django, Laravel, React) |
| `.claude/DATA-STRUCTURES.md` | Fundamental structures, domain modelling (value objects, aggregates, enums), database schema design (PostgreSQL and MariaDB/MySQL), normalisation, indexes, migrations, soft deletes, multi-tenancy, anti-patterns, and refactoring guidance |
| `.claude/PERFORMANCE.md` | Database query optimisation (N+1, EXPLAIN, indexing), caching (hierarchy, application, HTTP, invalidation), frontend performance (bundle size, code splitting, rendering strategy, React patterns), image optimisation, background jobs and queues, connection pooling, mobile performance, monitoring, load testing, and checklist |
| `.claude/DEVELOPMENT.md` | Development workflow, environment setup, and common tasks |
| `.claude/SEO-CHECKLIST.md` | SEO & AI discoverability checklist — Beginner through Advanced (including GEO and llms.txt) |

## Project Structure
\`\`\`
[project root]/
├── .claude/                      # Claude Code configuration
│   ├── CLAUDE.md                 # This file
│   ├── CODING-PRINCIPLES.md      # Coding standards and principles
│   ├── TESTING.md                # Testing guide for this stack
│   ├── SECURITY.md               # Security patterns and compliance
│   ├── ACCESSIBILITY.md          # WCAG 2.2 AA and ARIA patterns
│   ├── API-DESIGN.md             # REST and GraphQL conventions
│   ├── ARCHITECTURE-PATTERNS.md  # Service layer and project structure
│   ├── DATA-STRUCTURES.md        # Domain modelling and schema design
│   ├── PERFORMANCE.md            # Query optimisation and caching
│   ├── DEVELOPMENT.md            # Development workflow and tasks
│   ├── SEO-CHECKLIST.md          # SEO & AI discoverability checklist
│   ├── settings.local.json
│   ├── plugins/                  # Python plugin tools
│   │   └── *.py
│   └── commands/                 # Custom Claude commands
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