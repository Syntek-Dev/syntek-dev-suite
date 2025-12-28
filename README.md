# Syntek Dev Suite

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Adding to Claude Code CLI](#adding-to-claude-code-cli)
  - [Option 1: Global Installation (All Projects)](#option-1-global-installation-all-projects)
  - [Option 2: Project-Specific Installation](#option-2-project-specific-installation)
  - [Verify Installation](#verify-installation)
- [Overview](#overview)
  - [Supported Stacks](#supported-stacks)
  - [Key Features](#key-features)
- [Complete Development Workflow](#complete-development-workflow)
  - [Phase 1: Project Initialisation](#phase-1-project-initialisation)
  - [Phase 2: Project Planning](#phase-2-project-planning)
  - [Phase 3: User Stories and Sprint Planning](#phase-3-user-stories-and-sprint-planning)
  - [Phase 4: Pre-Development Review](#phase-4-pre-development-review)
  - [Phase 5: PM Tool Integration (Optional)](#phase-5-pm-tool-integration-optional)
  - [Phase 6: Git Branch Setup](#phase-6-git-branch-setup)
  - [Phase 7: User Story Planning](#phase-7-user-story-planning)
  - [Phase 8: Test-Driven Development](#phase-8-test-driven-development)
  - [Phase 9: Coding](#phase-9-coding)
  - [Phase 10: Quality Assurance](#phase-10-quality-assurance)
  - [Phase 11: Final Verification](#phase-11-final-verification)
  - [Phase 12: Documentation](#phase-12-documentation)
  - [Phase 13: Completion and PR](#phase-13-completion-and-pr)
  - [Workflow Summary](#workflow-summary)
- [Quick Start](#quick-start)
  - [1. Initialise for Your Project](#1-initialise-for-your-project)
  - [2. Start Developing](#2-start-developing)
- [Installation](#installation)
  - [Prerequisites](#prerequisites)
  - [Manual Setup (Alternative to /syntek-dev-suite:init)](#manual-setup-alternative-to-syntek-dev-suiteinit)
- [Command Reference](#command-reference)
  - [Agent Commands](#agent-commands)
    - [Planning \& Architecture](#planning--architecture)
    - [Development](#development)
    - [Quality \& Testing](#quality--testing)
    - [Refactoring \& Maintenance](#refactoring--maintenance)
    - [Infrastructure](#infrastructure)
    - [Specialised](#specialised)
  - [Plugin Commands](#plugin-commands)
  - [Learning Commands](#learning-commands)
  - [Version Management](#version-management)
- [Skills System](#skills-system)
  - [How Skills Work](#how-skills-work)
  - [Available Skills](#available-skills)
- [Templates](#templates)
- [Self-Learning System](#self-learning-system)
  - [How It Works](#how-it-works)
  - [A/B Testing](#ab-testing)
  - [Giving Feedback](#giving-feedback)
  - [Project-Specific Learning](#project-specific-learning)
- [Markdown All in One Extension](#markdown-all-in-one-extension)
  - [Key Features](#key-features-1)
  - [Installation](#installation-1)
- [Plugin Architecture](#plugin-architecture)
  - [Directory Structure](#directory-structure)
- [Configuration](#configuration)
  - [Project Configuration (`.claude/CLAUDE.md`)](#project-configuration-claudeclaudemd)
- [Troubleshooting](#troubleshooting)
  - ["Agent doesn't understand my stack"](#agent-doesnt-understand-my-stack)
  - ["Container tools not working"](#container-tools-not-working)
- [Best Practices](#best-practices)
- [Contributing](#contributing)
  - [Adding a New Agent](#adding-a-new-agent)
- [Documentation](#documentation)
- [Support](#support)


## Adding to Claude Code CLI

To install the Syntek Dev Suite plugin, add it to your Claude Code settings:

### Option 1: Global Installation (All Projects)

Add to your global settings file `~/.claude/settings.json`:

```json
{
  "plugins": [
    "/path/to/claude-dev-team"
  ]
}
```

### Option 2: Project-Specific Installation

Add to your project's `.claude/settings.local.json`:

```json
{
  "plugins": [
    "/path/to/claude-dev-team"
  ]
}
```

### Verify Installation

After adding the plugin, restart Claude Code and run:

```bash
/syntek-dev-suite:init
```

If the command is recognised, the plugin is installed correctly.

---

## Overview

Syntek Dev Suite is a comprehensive plugin for Claude Code that provides specialised AI agents for full-stack software development. Instead of generic AI assistance, you "hire" domain experts who understand your specific technology stack.

### Supported Stacks

| Stack              | Technologies                                                  | Container      |
| ------------------ | ------------------------------------------------------------- | -------------- |
| **TALL**           | Laravel 12, Livewire 3, Alpine.js, Tailwind CSS 4, MariaDB    | DDEV           |
| **Django**         | Python 3.14, Django 6, Wagtail 7, PostgreSQL 18, GraphQL      | Docker Compose |
| **React**          | TypeScript 5.9, React 19, Next.js 16, Tailwind CSS 4, Node 24 | Docker         |
| **Mobile**         | TypeScript 5.9, React Native 0.83, Expo, NativeWind 4.x       | Docker         |
| **Shared Library** | TypeScript 5.9, NPM package for web and mobile                | Docker         |

### Key Features

- **30 Specialised Agents** - Each with domain expertise and stack awareness
- **Automatic Stack Detection** - Agents read your `CLAUDE.md` to understand your project
- **Container Support** - DDEV for PHP, Docker for everything else
- **Code Examples** - 60+ versioned examples for common patterns
- **British English** - Localised for UK spelling, date formats, and currency
- **Self-Learning System** - Project-specific prompt improvements via A/B testing

---

## Complete Development Workflow

This section walks through a complete development cycle using the Syntek Dev Suite, from project setup to PR submission.

> **Note:** All commands use the `/syntek-dev-suite:` prefix.

### Phase 1: Project Initialisation

```bash
# 1. Initialise Claude Code to work with Syntek Dev Suite
/syntek-dev-suite:init

# 2. Set up all project files using the setup agent
/syntek-dev-suite:setup

# 3. Set up dot files (.gitignore, .editorconfig, etc.)
/syntek-dev-suite:setup .files

# 4. Commit initialisation work
/syntek-dev-suite:git
```

### Phase 2: Project Planning

```bash
# 5. Plan the project architecture
/syntek-dev-suite:plan X project that will [describe what the project does]
# Example: /syntek-dev-suite:plan E-commerce project that will allow users to browse products and checkout

# 6. Commit the plan
/syntek-dev-suite:git
```

### Phase 3: User Stories and Sprint Planning

```bash
# 7. Generate user stories from the plan
/syntek-dev-suite:stories

# 8. Commit user stories
/syntek-dev-suite:git

# 9. Organise user stories into sprints
/syntek-dev-suite:sprint

# 10. Commit sprints
/syntek-dev-suite:git
```

### Phase 4: Pre-Development Review

```bash
# 11. Review all setup before coding begins
/syntek-dev-suite:review
# Reviews: init, plan, .files, stories, and sprints

# 12. Commit any review changes
/syntek-dev-suite:git
```

### Phase 5: PM Tool Integration (Optional)

```bash
# 13. Link with project management software
/syntek-dev-suite:pm-setup
# Creates integration with ClickUp, Linear, Jira, etc.
# Sets up git workflow to sync user stories and sprints

# 14. Commit PM setup
/syntek-dev-suite:git
```

### Phase 6: Git Branch Setup

```bash
# 15. Push setup to main and create branches
/syntek-dev-suite:git
# Pushes to main branch
# Creates staging, dev, and testing branches from main via PR

# 16. Create user story branch from dev
/syntek-dev-suite:git
# Creates us001/name branch for the first user story from dev branch
```

### Phase 7: User Story Planning

```bash
# 17. Plan the specific user story
/syntek-dev-suite:plan user story X
# Example: /syntek-dev-suite:plan user story 1

# 18. Commit the user story plan
/syntek-dev-suite:git
```

### Phase 8: Test-Driven Development

```bash
# 19. Write TDD/BDD tests (tests should fail initially)
/syntek-dev-suite:test-writer
# Writes tests scoped ONLY to the user story
# Creates minimal code stubs to ensure tests fail

# 20. Commit tests
/syntek-dev-suite:git
```

### Phase 9: Coding

```bash
# 21. Implement the user story using relevant agents
# Commit after EACH piece of work for proper version control

/syntek-dev-suite:backend    # For API, database, server logic
/syntek-dev-suite:git        # Commit backend work

/syntek-dev-suite:frontend   # For UI components, styling
/syntek-dev-suite:git        # Commit frontend work

/syntek-dev-suite:database   # For migrations, schemas
/syntek-dev-suite:git        # Commit database work

/syntek-dev-suite:data       # For data analysis, Python/SQL
/syntek-dev-suite:git        # Commit data work

# Use whichever agents are relevant to the user story
# Goal: Pass tests and satisfy acceptance criteria
# IMPORTANT: Always commit after each agent completes their work
```

### Phase 10: Quality Assurance

```bash
# 22. Run QA to ensure tests pass
/syntek-dev-suite:qa-tester
/syntek-dev-suite:git        # Commit QA fixes

# 23. Generate bug report and fix issues
/syntek-dev-suite:debug
/syntek-dev-suite:git        # Commit bug fixes

# 24. Review the code
/syntek-dev-suite:review
/syntek-dev-suite:git        # Commit review changes

# 25. Refactor if needed
/syntek-dev-suite:refactor
/syntek-dev-suite:git        # Commit refactoring

# 26. Fix syntax and linting issues
/syntek-dev-suite:syntax
/syntek-dev-suite:git        # Commit syntax fixes

# 27. Security audit
/syntek-dev-suite:security
/syntek-dev-suite:git        # Commit security fixes
```

### Phase 11: Final Verification

```bash
# 28. Final QA check - ensure tests still pass
/syntek-dev-suite:qa-tester
/syntek-dev-suite:git        # Commit any final QA fixes

# 29. Final checks cycle (commit after each if changes made)
/syntek-dev-suite:debug      # Final bug check
/syntek-dev-suite:git

/syntek-dev-suite:review     # Final code review
/syntek-dev-suite:git

/syntek-dev-suite:refactor   # Final refactoring pass
/syntek-dev-suite:git

/syntek-dev-suite:syntax     # Final syntax check
/syntek-dev-suite:git

/syntek-dev-suite:security   # Final security check
/syntek-dev-suite:git
```

### Phase 12: Documentation

```bash
# 30. Write documentation
/syntek-dev-suite:docs
/syntek-dev-suite:git        # Commit documentation

# 31. Update version files
/syntek-dev-suite:version
/syntek-dev-suite:git        # Commit version updates
```

### Phase 13: Completion and PR

```bash
# 32. Mark work as complete
/syntek-dev-suite:completion
/syntek-dev-suite:git        # Commit completion status

# 33. Create PR to testing branch for review
/syntek-dev-suite:git
# Creates PR from user story branch to testing for review
```

### Workflow Summary

```
SETUP PHASE
───────────────────────────────────────────────────────────
init → setup → setup .files → git
plan project → git
stories → git → sprint → git
review → git

PM & GIT SETUP
───────────────────────────────────────────────────────────
pm-setup → git
git (main + branches) → git (user story branch)

USER STORY PLANNING
───────────────────────────────────────────────────────────
plan user story → git
test-writer → git

IMPLEMENTATION (commit after each agent)
───────────────────────────────────────────────────────────
backend → git → frontend → git → database → git → data → git

QUALITY ASSURANCE (commit after each step)
───────────────────────────────────────────────────────────
qa-tester → git → debug → git → review → git → refactor → git → syntax → git → security → git

FINAL VERIFICATION (commit after each step)
───────────────────────────────────────────────────────────
qa-tester → git → debug → git → review → git → refactor → git → syntax → git → security → git

DOCUMENTATION & COMPLETION
───────────────────────────────────────────────────────────
docs → git → version → git → completion → git → git (PR to testing)
```

---

## Quick Start

### 1. Initialise for Your Project

```bash
# Navigate to your project root
cd ~/my-project

# Run the init command
/syntek-dev-suite:init
```

This will:
- Detect your project stack
- Create `.claude/CLAUDE.md` from the appropriate template
- Set up container configuration (DDEV or Docker)
- Copy the Syntek Guide to `.claude/SYNTEK-GUIDE.md`

### 2. Start Developing

```bash
# Plan a feature
/syntek-dev-suite:plan Add user authentication with social login

# Implement backend
/syntek-dev-suite:backend Create User model and auth endpoints

# Implement frontend
/syntek-dev-suite:frontend Create login form component

# Write tests
/syntek-dev-suite:test-writer Write tests for auth flow

# Review code
/syntek-dev-suite:qa-tester Review the auth implementation
```

---

## Installation

### Prerequisites

- Claude Code CLI installed
- Docker or DDEV (depending on your stack)
- Git

### Manual Setup (Alternative to /syntek-dev-suite:init)

1. **Navigate to your project root:**

```bash
cd ~/my-new-project
```

2. **Copy the correct template:**

**For TALL Stack (Laravel + Livewire + Alpine + Tailwind):**
```bash
mkdir -p .claude
cp /path/to/claude-dev-team/templates/tall-project.md ./.claude/CLAUDE.md
cp /path/to/claude-dev-team/examples/setup/SYNTEK-GUIDE-TEMPLATE.md ./.claude/SYNTEK-GUIDE.md
```

**For Django/Wagtail:**
```bash
mkdir -p .claude
cp /path/to/claude-dev-team/templates/django-project.md ./.claude/CLAUDE.md
cp /path/to/claude-dev-team/examples/setup/SYNTEK-GUIDE-TEMPLATE.md ./.claude/SYNTEK-GUIDE.md
```

**For React Web (SPA):**
```bash
mkdir -p .claude
cp /path/to/claude-dev-team/templates/react-project.md ./.claude/CLAUDE.md
cp /path/to/claude-dev-team/examples/setup/SYNTEK-GUIDE-TEMPLATE.md ./.claude/SYNTEK-GUIDE.md
```

**For React Native Mobile:**
```bash
mkdir -p .claude
cp /path/to/claude-dev-team/templates/mobile-project.md ./.claude/CLAUDE.md
cp /path/to/claude-dev-team/examples/setup/SYNTEK-GUIDE-TEMPLATE.md ./.claude/SYNTEK-GUIDE.md
```

**For Shared NPM Package:**
```bash
mkdir -p .claude
cp /path/to/claude-dev-team/templates/shared-lib-project.md ./.claude/CLAUDE.md
cp /path/to/claude-dev-team/examples/setup/SYNTEK-GUIDE-TEMPLATE.md ./.claude/SYNTEK-GUIDE.md
```

3. **Edit CLAUDE.md:**

Open `.claude/CLAUDE.md` and replace all `[Insert Project Name]` placeholders with your project details.

---

## Command Reference

All Syntek Dev Suite commands use the `/syntek-dev-suite:` prefix:

```bash
/syntek-dev-suite:<command> [arguments]
```

### Agent Commands

#### Planning & Architecture

| Command                        | Model  | Description                                     |
| ------------------------------ | ------ | ----------------------------------------------- |
| `/syntek-dev-suite:plan`       | Opus   | Create architectural plans, break down features |
| `/syntek-dev-suite:stories`    | Haiku  | Generate user stories from requirements         |
| `/syntek-dev-suite:sprint`     | Sonnet | Organise stories into balanced sprints          |
| `/syntek-dev-suite:completion` | Sonnet | Track story and sprint completion               |

#### Development

| Command                      | Model  | Description                               |
| ---------------------------- | ------ | ----------------------------------------- |
| `/syntek-dev-suite:setup`    | Sonnet | Project initialisation and configuration  |
| `/syntek-dev-suite:backend`  | Sonnet | Backend development, APIs, database       |
| `/syntek-dev-suite:frontend` | Sonnet | UI/UX, components, accessibility          |
| `/syntek-dev-suite:database` | Sonnet | Database design, migrations, optimisation |
| `/syntek-dev-suite:auth`     | Sonnet | Authentication, MFA, session management   |

#### Quality & Testing

| Command                         | Model  | Description                      |
| ------------------------------- | ------ | -------------------------------- |
| `/syntek-dev-suite:test-writer` | Sonnet | TDD test suites and stubs        |
| `/syntek-dev-suite:qa-tester`   | Sonnet | Hostile QA, security, edge cases |
| `/syntek-dev-suite:review`      | Sonnet | Code review, SOLID, security     |
| `/syntek-dev-suite:debug`       | Opus   | Root cause analysis, debugging   |

#### Refactoring & Maintenance

| Command                      | Model  | Description                         |
| ---------------------------- | ------ | ----------------------------------- |
| `/syntek-dev-suite:refactor` | Sonnet | Code cleanup without changing logic |
| `/syntek-dev-suite:syntax`   | Haiku  | Fix syntax and linting errors       |
| `/syntek-dev-suite:docs`     | Haiku  | Technical documentation             |

#### Infrastructure

| Command                      | Model  | Description                            |
| ---------------------------- | ------ | -------------------------------------- |
| `/syntek-dev-suite:cicd`     | Sonnet | CI/CD pipelines, deployments           |
| `/syntek-dev-suite:security` | Sonnet | Access control, headers, rate limiting |
| `/syntek-dev-suite:logging`  | Sonnet | Logging, Sentry, audit trails          |
| `/syntek-dev-suite:git`      | Sonnet | Branch management, versioning          |

#### Specialised

| Command                              | Model  | Description                       |
| ------------------------------------ | ------ | --------------------------------- |
| `/syntek-dev-suite:gdpr`             | Sonnet | GDPR compliance, data protection  |
| `/syntek-dev-suite:seo`              | Sonnet | SEO, meta tags, structured data   |
| `/syntek-dev-suite:notifications`    | Sonnet | Email, SMS, push notifications    |
| `/syntek-dev-suite:export`           | Sonnet | PDF, Excel, CSV, JSON exports     |
| `/syntek-dev-suite:reporting`        | Sonnet | Data queries, report services     |
| `/syntek-dev-suite:data`             | Sonnet | Data analysis, Python, SQL        |
| `/syntek-dev-suite:support-articles` | Sonnet | Help documentation                |
| `/syntek-dev-suite:pm-setup`         | Sonnet | PM tool setup and integration     |
| `/syntek-dev-suite:version`          | Sonnet | Version management and changelogs |

### Plugin Commands

| Command                  | Description                               |
| ------------------------ | ----------------------------------------- |
| `/syntek-dev-suite:init` | Initialise Syntek Dev Suite for a project |

### Learning Commands

| Command                                               | Description                     |
| ----------------------------------------------------- | ------------------------------- |
| `/syntek-dev-suite:learning-feedback good`            | Mark the last run as successful |
| `/syntek-dev-suite:learning-feedback bad [comment]`   | Mark as needing improvement     |
| `/syntek-dev-suite:learning-ab-test list`             | List active A/B tests           |
| `/syntek-dev-suite:learning-ab-test status <agent>`   | Show test results for an agent  |
| `/syntek-dev-suite:learning-optimise status`          | Show optimisation system status |
| `/syntek-dev-suite:learning-optimise analyse <agent>` | Analyse an agent's performance  |

### Version Management

| Command                                 | Description                                |
| --------------------------------------- | ------------------------------------------ |
| `/syntek-dev-suite:version bump <type>` | Increment version (major, minor, patch)    |
| `/syntek-dev-suite:version update`      | Update all version files and documentation |
| `/syntek-dev-suite:version headers`     | Update metadata headers in all .md files   |
| `/syntek-dev-suite:version init`        | Initialise version files for a new project |
| `/syntek-dev-suite:version status`      | Show current version and pending changes   |

---

## Skills System

Skills provide stack-specific knowledge that agents use automatically.

### How Skills Work

1. Your project's `CLAUDE.md` specifies a `Skill Target` (e.g., `stack-tall`)
2. When an agent runs, it loads the stack skill and global workflow skill
3. The agent applies stack-specific patterns and conventions

### Available Skills

| Skill              | Target         | Applied To                            |
| ------------------ | -------------- | ------------------------------------- |
| `stack-tall`       | TALL Stack     | Laravel, Livewire, Alpine, Tailwind   |
| `stack-django`     | Django         | Django, Wagtail, PostgreSQL, GraphQL  |
| `stack-react`      | React          | React, Next.js, TypeScript, Tailwind  |
| `stack-mobile`     | Mobile         | React Native, Expo, NativeWind        |
| `stack-shared-lib` | Shared Library | NPM packages for web/mobile           |
| `global-workflow`  | All            | British English, Git, dates, currency |

---

## Templates

| Template   | File                              | Container      |
| ---------- | --------------------------------- | -------------- |
| TALL       | `templates/tall-project.md`       | DDEV           |
| Django     | `templates/django-project.md`     | Docker Compose |
| React      | `templates/react-project.md`      | Docker         |
| Mobile     | `templates/mobile-project.md`     | Docker         |
| Shared Lib | `templates/shared-lib-project.md` | Docker         |

---

## Self-Learning System

The plugin includes a self-learning system that improves agent performance based on your feedback. Learning data is stored per-project in `docs/METRICS/` and committed to Git.

### How It Works

1. **Feedback Collection** - After each agent run, rate the output
2. **Metrics Recording** - Run duration, outcomes, and errors are tracked
3. **Pattern Analysis** - The system identifies what works and what doesn't
4. **Prompt Optimisation** - Agent prompts are improved based on feedback
5. **A/B Testing** - Prompt variants are tested to find the best approach

### A/B Testing

Each agent can run A/B tests on prompt variants to discover what works best for your specific project:

```bash
# List active A/B tests
/syntek-dev-suite:learning-ab-test list

# Check test status for an agent
/syntek-dev-suite:learning-ab-test status backend

# The system automatically:
# - Randomly assigns variants to runs
# - Tracks success/failure rates
# - Identifies statistically significant winners
# - Applies winning prompts automatically
```

### Giving Feedback

After each agent run:

```bash
# If the output was good
/syntek-dev-suite:learning-feedback good

# If the output needs improvement
/syntek-dev-suite:learning-feedback bad The output didn't follow the coding style
```

### Project-Specific Learning

- All feedback and metrics are stored in your project's `docs/METRICS/` folder
- Data is committed to Git, so the whole team benefits from improvements
- Each project develops its own optimised prompts over time
- No external API calls - learning uses Claude Code CLI directly

---

## Markdown All in One Extension

The Syntek Dev Suite is configured for optimal use with the **Markdown All in One** VS Code extension.

### Key Features

| Feature            | Description                                   |
| ------------------ | --------------------------------------------- |
| Auto-updating TOCs | Table of Contents stays in sync with headings |
| Smart Lists        | Auto-renumbering and intelligent indentation  |
| Table Formatting   | GFM tables auto-align on save                 |
| Task Lists         | Toggle checkboxes with `Alt+C`                |
| Math Support       | Render LaTeX-style math expressions           |

### Installation

When opening a Syntek Dev Suite project in VS Code, you'll be prompted to install recommended extensions. Alternatively, install manually:

```bash
code --install-extension yzhang.markdown-all-in-one
```

For the complete guide, see [docs/GUIDES/MARKDOWN-ALL-IN-ONE.md](docs/GUIDES/MARKDOWN-ALL-IN-ONE.md).

---

## Plugin Architecture

### Directory Structure

```
syntek-dev-suite/
├── .claude-plugin/          # Plugin configuration
├── agents/                  # Agent definitions (30 agents)
├── commands/                # Slash commands
├── skills/                  # Stack-specific skills
├── templates/               # Project templates
├── examples/                # Code examples (60+)
├── plugins/                 # Python tools
└── CHANGELOG.md             # Version history
```

---

## Configuration

### Project Configuration (`.claude/CLAUDE.md`)

Each project has a `CLAUDE.md` that tells agents:

```markdown
## Stack Overview
- **Type:** TALL Stack
- **Language:** PHP 8.4
- **Framework:** Laravel 12.x

## Skill Targets
- **Stack Skill:** stack-tall
- **Global Skill:** global-workflow

## Environment
- **Container:** DDEV
- **Database:** MariaDB
- **Locale:** en_GB
- **Timezone:** Europe/London
```

---

## Troubleshooting

### "Agent doesn't understand my stack"

Ensure `.claude/CLAUDE.md` exists with correct `Skill Target`:

```markdown
## Skill Targets
- **Stack Skill:** stack-tall
```

### "Container tools not working"

```bash
chmod +x /path/to/claude-dev-team/plugins/*.py
```

---

## Best Practices

1. **Always start with `/syntek-dev-suite:plan`** - Get a roadmap before coding
2. **Use `/syntek-dev-suite:qa-tester` before merging** - Catch issues early
3. **Keep CLAUDE.md updated** - Add new dependencies and constraints
4. **Let agents read files** - Don't paste code, reference files
5. **Give feedback** - Use `/syntek-dev-suite:learning-feedback` to improve agents over time
6. **Commit regularly** - Small, focused commits after each step

---

## Contributing

### Adding a New Agent

1. Create `agents/my-agent.md` with agent instructions
2. Create `commands/my-agent.md` with command definition
3. Test with `/syntek-dev-suite:my-agent Test this new agent`

---

## Documentation

For detailed documentation, see:

- [agents/](agents/) - Agent definitions and capabilities
- [commands/](commands/) - Available slash commands
- [skills/](skills/) - Stack-specific skills
- [templates/](templates/) - Project templates
- [examples/](examples/) - Code examples by domain
- [plugins/](plugins/) - Python utility tools
- [CHANGELOG.md](CHANGELOG.md) - Version history

---

## Support

- **Issues:** https://github.com/syntek-developers/syntek-dev-suite/issues
- **Documentation:** See `examples/` folder
- **Guide:** After init, see `.claude/SYNTEK-GUIDE.md`

---

**Happy coding with Syntek Dev Suite!**
