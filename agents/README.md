# Agents

## Table of Contents

- [Agents](#agents)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Directory Tree](#directory-tree)
  - [Agents by Category](#agents-by-category)
    - [Planning \& Architecture](#planning--architecture)
    - [Development](#development)
    - [Quality \& Testing](#quality--testing)
    - [Refactoring \& Maintenance](#refactoring--maintenance)
    - [Infrastructure](#infrastructure)
    - [Specialised](#specialised)
  - [Usage](#usage)
    - [How Agents Work](#how-agents-work)
  - [Related Sections](#related-sections)

---

## Overview

This folder contains the 28 specialised AI agent definitions for the Syntek Dev Suite. Each agent is a markdown file containing instructions, context, and behavioural guidelines that Claude uses when the agent is invoked.

Agents are invoked via their corresponding command in the `commands/` folder (e.g., `/agent:backend` invokes the backend agent).

**Version:** 1.0.0

---

## Directory Tree

```
agents/
├── README.md              # This file
├── authentication.md      # Authentication and MFA specialist
├── backend.md             # Backend development agent
├── cicd.md                # CI/CD and deployment agent
├── code-reviewer.md       # Code review agent
├── completion.md          # Sprint/story completion tracker
├── data-scientist.md      # Data analysis agent
├── database.md            # Database design and migrations
├── debugger.md            # Root cause analysis debugging
├── doc-writer.md          # Technical documentation
├── export.md              # PDF, Excel, CSV export agent
├── frontend.md            # Frontend and UI/UX agent
├── gdpr.md                # GDPR compliance agent
├── git.md                 # Git workflow and branching
├── logging.md             # Logging and audit trails
├── notifications.md       # Email, SMS, push notifications
├── optimiser.md           # Performance optimisation
├── planner.md             # System architect and planner
├── qa-tester.md           # Quality assurance tester
├── refactor.md            # Code refactoring agent
├── reporting.md           # Report generation agent
├── security.md            # Security and access control
├── seo.md                 # SEO and meta tags agent
├── setup.md               # Project setup and initialisation
├── sprint.md              # Sprint organisation agent
├── support-articles.md    # Help documentation writer
├── syntax.md              # Syntax and linting fixer
├── test-writer.md         # TDD test suite writer
└── user-story.md          # User story generator
```

---

## Agents by Category

### Planning & Architecture

| Agent | File | Model | Description |
|-------|------|-------|-------------|
| Planner | `planner.md` | Opus | System architect, creates implementation plans |
| User Story | `user-story.md` | Haiku | Generates user stories from requirements |
| Sprint | `sprint.md` | Sonnet | Organises stories into balanced sprints |
| Completion | `completion.md` | Sonnet | Tracks story and sprint completion |

### Development

| Agent | File | Model | Description |
|-------|------|-------|-------------|
| Setup | `setup.md` | Sonnet | Project initialisation and configuration |
| Backend | `backend.md` | Sonnet | Backend development, APIs, database |
| Frontend | `frontend.md` | Sonnet | UI/UX, components, accessibility |
| Database | `database.md` | Sonnet | Database design, migrations, optimisation |
| Authentication | `authentication.md` | Sonnet | Authentication, MFA, session management |

### Quality & Testing

| Agent | File | Model | Description |
|-------|------|-------|-------------|
| Test Writer | `test-writer.md` | Sonnet | TDD test suites and stubs |
| QA Tester | `qa-tester.md` | Sonnet | Hostile QA, security, edge cases |
| Code Reviewer | `code-reviewer.md` | Sonnet | Code review, SOLID, security |
| Debugger | `debugger.md` | Opus | Root cause analysis, debugging |

### Refactoring & Maintenance

| Agent | File | Model | Description |
|-------|------|-------|-------------|
| Refactor | `refactor.md` | Sonnet | Code cleanup without changing logic |
| Syntax | `syntax.md` | Haiku | Fix syntax and linting errors |
| Doc Writer | `doc-writer.md` | Haiku | Technical documentation |
| Optimiser | `optimiser.md` | Sonnet | Performance optimisation |

### Infrastructure

| Agent | File | Model | Description |
|-------|------|-------|-------------|
| CI/CD | `cicd.md` | Sonnet | CI/CD pipelines, deployments |
| Security | `security.md` | Sonnet | Access control, headers, rate limiting |
| Logging | `logging.md` | Sonnet | Logging, Sentry, audit trails |
| Git | `git.md` | Sonnet | Branch management, versioning |

### Specialised

| Agent | File | Model | Description |
|-------|------|-------|-------------|
| GDPR | `gdpr.md` | Sonnet | GDPR compliance, data protection |
| SEO | `seo.md` | Sonnet | SEO, meta tags, structured data |
| Notifications | `notifications.md` | Sonnet | Email, SMS, push notifications |
| Export | `export.md` | Sonnet | PDF, Excel, CSV, JSON exports |
| Reporting | `reporting.md` | Sonnet | Data queries, report services |
| Data Scientist | `data-scientist.md` | Sonnet | Data analysis, Python, SQL |
| Support Articles | `support-articles.md` | Sonnet | Help documentation |

---

## Usage

Agents are invoked through their corresponding command:

```bash
# Invoke the backend agent
/agent:backend Create the User model and API endpoints

# Invoke the planner agent
/agent:plan Design a user dashboard feature

# Invoke the QA tester agent
/agent:qa-tester Review the authentication flow
```

### How Agents Work

1. User invokes a command (e.g., `/agent:backend`)
2. Claude reads the command file from `commands/backend.md`
3. Command spawns the agent using `agents/backend.md`
4. Agent reads the project's `.claude/CLAUDE.md` for stack context
5. Agent applies the appropriate skill from `skills/`
6. Agent performs the requested task

---

## Related Sections

- [../commands/](../commands/) - Slash commands that invoke these agents
- [../skills/](../skills/) - Stack-specific knowledge applied by agents
- [../examples/](../examples/) - Code examples agents reference
- [../templates/](../templates/) - Project templates for CLAUDE.md
