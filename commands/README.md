# Commands

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Directory Tree](#directory-tree)
- [Commands by Category](#commands-by-category)
  - [Plugin Commands](#plugin-commands)
  - [Agent Commands](#agent-commands)
    - [Planning \& Architecture](#planning--architecture)
    - [Development](#development)
    - [Quality \& Testing](#quality--testing)
    - [Refactoring \& Maintenance](#refactoring--maintenance)
    - [Infrastructure](#infrastructure)
    - [Specialised](#specialised)
  - [Learning Commands](#learning-commands)
- [Usage](#usage)
  - [Command Structure](#command-structure)
- [Related Sections](#related-sections)

---

## Overview

This folder contains all slash command definitions for the Syntek Dev Suite. Each command is a markdown file that defines how Claude should respond when the user invokes that command.

Commands follow a prefix convention:
- `/agent:` - Spawns a specialised AI agent
- `/plugin:` - Plugin management commands
- `/learning:` - Self-learning system commands

**Version:** 1.2.0

---

## Directory Tree

```
commands/
├── README.md              # This file
├── auth.md                # /agent:auth - Authentication agent
├── backend.md             # /agent:backend - Backend development
├── cicd.md                # /agent:cicd - CI/CD pipelines
├── completion.md          # /agent:completion - Sprint tracking
├── data.md                # /agent:data - Data analysis
├── database.md            # /agent:database - Database design
├── debug.md               # /agent:debug - Debugging agent
├── docs.md                # /agent:docs - Documentation
├── export.md              # /agent:export - Export generation
├── frontend.md            # /agent:frontend - Frontend development
├── gdpr.md                # /agent:gdpr - GDPR compliance
├── git.md                 # /agent:git - Git workflows
├── init.md                # /plugin:init - Project initialisation
├── learning-ab-test.md    # /learning:ab-test - A/B testing
├── learning-feedback.md   # /learning:feedback - User feedback
├── learning-optimise.md   # /learning:optimise - Prompt optimisation
├── logging.md             # /agent:logging - Logging setup
├── notifications.md       # /agent:notifications - Notifications
├── plan.md                # /agent:plan - System planning
├── qa-tester.md           # /agent:qa-tester - QA testing
├── refactor.md            # /agent:refactor - Code refactoring
├── reporting.md           # /agent:reporting - Report services
├── review.md              # /agent:review - Code review
├── security.md            # /agent:security - Security agent
├── seo.md                 # /agent:seo - SEO optimisation
├── setup.md               # /agent:setup - Project setup
├── sprint.md              # /agent:sprint - Sprint management
├── stories.md             # /agent:stories - User stories
├── support-articles.md    # /agent:support-articles - Help docs
├── syntax.md              # /agent:syntax - Syntax fixing
└── test-writer.md         # /agent:test-writer - Test writing
```

---

## Commands by Category

### Plugin Commands

| Command        | File      | Description                               |
| -------------- | --------- | ----------------------------------------- |
| `/plugin:init` | `init.md` | Initialise Syntek Dev Suite for a project |

### Agent Commands

#### Planning & Architecture

| Command             | File            | Model  | Description                |
| ------------------- | --------------- | ------ | -------------------------- |
| `/agent:plan`       | `plan.md`       | Opus   | Create architectural plans |
| `/agent:stories`    | `stories.md`    | Haiku  | Generate user stories      |
| `/agent:sprint`     | `sprint.md`     | Sonnet | Organise sprints           |
| `/agent:completion` | `completion.md` | Sonnet | Track completion           |

#### Development

| Command           | File          | Model  | Description          |
| ----------------- | ------------- | ------ | -------------------- |
| `/agent:setup`    | `setup.md`    | Sonnet | Project setup        |
| `/agent:backend`  | `backend.md`  | Sonnet | Backend development  |
| `/agent:frontend` | `frontend.md` | Sonnet | Frontend development |
| `/agent:database` | `database.md` | Sonnet | Database design      |
| `/agent:auth`     | `auth.md`     | Sonnet | Authentication       |

#### Quality & Testing

| Command              | File             | Model  | Description |
| -------------------- | ---------------- | ------ | ----------- |
| `/agent:test-writer` | `test-writer.md` | Sonnet | Write tests |
| `/agent:qa-tester`   | `qa-tester.md`   | Sonnet | QA testing  |
| `/agent:review`      | `review.md`      | Sonnet | Code review |
| `/agent:debug`       | `debug.md`       | Opus   | Debugging   |

#### Refactoring & Maintenance

| Command           | File          | Model  | Description   |
| ----------------- | ------------- | ------ | ------------- |
| `/agent:refactor` | `refactor.md` | Sonnet | Refactoring   |
| `/agent:syntax`   | `syntax.md`   | Haiku  | Syntax fixing |
| `/agent:docs`     | `docs.md`     | Haiku  | Documentation |

#### Infrastructure

| Command           | File          | Model  | Description     |
| ----------------- | ------------- | ------ | --------------- |
| `/agent:cicd`     | `cicd.md`     | Sonnet | CI/CD pipelines |
| `/agent:security` | `security.md` | Sonnet | Security        |
| `/agent:logging`  | `logging.md`  | Sonnet | Logging         |
| `/agent:git`      | `git.md`      | Sonnet | Git workflows   |

#### Specialised

| Command                   | File                  | Model  | Description       |
| ------------------------- | --------------------- | ------ | ----------------- |
| `/agent:gdpr`             | `gdpr.md`             | Sonnet | GDPR compliance   |
| `/agent:seo`              | `seo.md`              | Sonnet | SEO               |
| `/agent:notifications`    | `notifications.md`    | Sonnet | Notifications     |
| `/agent:export`           | `export.md`           | Sonnet | Export generation |
| `/agent:reporting`        | `reporting.md`        | Sonnet | Reporting         |
| `/agent:data`             | `data.md`             | Sonnet | Data analysis     |
| `/agent:support-articles` | `support-articles.md` | Sonnet | Help docs         |

### Learning Commands

| Command              | File                   | Description                     |
| -------------------- | ---------------------- | ------------------------------- |
| `/learning:feedback` | `learning-feedback.md` | Submit feedback on agent output |
| `/learning:ab-test`  | `learning-ab-test.md`  | Manage A/B tests for prompts    |
| `/learning:optimise` | `learning-optimise.md` | Analyse and optimise prompts    |

---

## Usage

Commands are invoked by typing the command followed by your request:

```bash
# Invoke a command
/agent:backend Create the User model

# With detailed context
/agent:plan Design a user dashboard with stats, orders, and settings widgets

# Learning feedback
/learning:feedback good
/learning:feedback bad The output didn't follow the coding style
```

### Command Structure

Each command file follows this structure:

```markdown
---
description: "[Agent] Short description"
usage: /agent:command-name
---

Instructions for Claude on how to handle this command...
```

---

## Related Sections

- [../agents/](../agents/) - Agent definitions invoked by these commands
- [../skills/](../skills/) - Stack skills applied during agent execution
- [../plugins/](../plugins/) - Python tools available to agents
- [../examples/](../examples/) - Code examples referenced by agents
