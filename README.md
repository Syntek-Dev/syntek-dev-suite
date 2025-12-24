# Syntek Dev Suite

**Full-Stack AI Agent Suite for Claude Code**

Version: 1.1.0 | Maintained by: Syntek Developers

---

## Table of Contents

- [Syntek Dev Suite](#syntek-dev-suite)
  - [Table of Contents](#table-of-contents)
  - [Adding to Claude Code CLI](#adding-to-claude-code-cli)
    - [Option 1: Global Installation (All Projects)](#option-1-global-installation-all-projects)
    - [Option 2: Project-Specific Installation](#option-2-project-specific-installation)
    - [Verify Installation](#verify-installation)
  - [Overview](#overview)
    - [Supported Stacks](#supported-stacks)
    - [Key Features](#key-features)
  - [Complete Development Workflow](#complete-development-workflow)
    - [Phase 1: Repository Setup](#phase-1-repository-setup)
    - [Phase 2: User Stories and Sprint Planning](#phase-2-user-stories-and-sprint-planning)
    - [Phase 3: Planning the User Story](#phase-3-planning-the-user-story)
    - [Phase 4: Test-Driven Development](#phase-4-test-driven-development)
    - [Phase 5: Implementation](#phase-5-implementation)
    - [Phase 6: Quality Assurance](#phase-6-quality-assurance)
    - [Phase 7: Review and Refactoring](#phase-7-review-and-refactoring)
    - [Phase 8: Completion and PR](#phase-8-completion-and-pr)
    - [Workflow Summary](#workflow-summary)
  - [Quick Start](#quick-start)
    - [1. Initialise for Your Project](#1-initialise-for-your-project)
    - [2. Start Developing](#2-start-developing)
  - [Installation](#installation)
    - [Prerequisites](#prerequisites)
    - [Manual Setup (Alternative to /plugin:init)](#manual-setup-alternative-to-plugininit)
  - [Command Reference](#command-reference)
    - [Command Types](#command-types)
    - [Plugin Commands](#plugin-commands)
    - [Agent Commands](#agent-commands)
      - [Planning \& Architecture](#planning--architecture)
      - [Development](#development)
      - [Quality \& Testing](#quality--testing)
      - [Refactoring \& Maintenance](#refactoring--maintenance)
      - [Infrastructure](#infrastructure)
      - [Specialised](#specialised)
  - [Skills System](#skills-system)
    - [How Skills Work](#how-skills-work)
    - [Available Skills](#available-skills)
  - [Templates](#templates)
  - [Self-Learning System](#self-learning-system)
    - [How It Works](#how-it-works)
    - [A/B Testing](#ab-testing)
    - [Giving Feedback](#giving-feedback)
    - [Learning Commands](#learning-commands)
    - [Project-Specific Learning](#project-specific-learning)
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
/plugin:init
```

If the command is recognised, the plugin is installed correctly.

---

## Overview

Syntek Dev Suite is a comprehensive plugin for Claude Code that provides specialised AI agents for full-stack software development. Instead of generic AI assistance, you "hire" domain experts who understand your specific technology stack.

### Supported Stacks

| Stack | Technologies | Container |
|-------|--------------|-----------|
| **TALL** | Laravel 12, Livewire 3, Alpine.js, Tailwind CSS 4, MariaDB | DDEV |
| **Django** | Python 3.14, Django 6, Wagtail 7, PostgreSQL 18, GraphQL | Docker Compose |
| **React** | TypeScript 5.9, React 19, Next.js 16, Tailwind CSS 4, Node 24 | Docker |
| **Mobile** | TypeScript 5.9, React Native 0.83, Expo, NativeWind 4.x | Docker |
| **Shared Library** | TypeScript 5.9, NPM package for web and mobile | Docker |

### Key Features

- **28 Specialised Agents** - Each with domain expertise and stack awareness
- **Automatic Stack Detection** - Agents read your `CLAUDE.md` to understand your project
- **Container Support** - DDEV for PHP, Docker for everything else
- **Code Examples** - 80+ versioned examples for common patterns
- **British English** - Localised for UK spelling, date formats, and currency
- **Self-Learning System** - Project-specific prompt improvements via A/B testing

---

## Complete Development Workflow

This section walks through a complete development cycle using the Syntek Dev Suite, from project setup to PR submission.

### Phase 1: Repository Setup

```bash
# 1. Clone or create your repository
git clone git@github.com:your-org/your-project.git
cd your-project

# 2. Initialise the plugin for your project
/plugin:init

# 3. Edit .claude/CLAUDE.md with your project details
# Replace all [Insert...] placeholders

# 4. Create core Git branches
/agent:git Create main, develop, staging, and production branches with appropriate protection rules
git commit -m "chore: initialise project structure"
```

### Phase 2: User Stories and Sprint Planning

```bash
# 1. Generate user stories from requirements
/agent:stories Based on this PRD, create user stories for user authentication feature
git commit -m "docs: add user authentication stories"

# 2. Organise stories into sprints
/agent:sprint Organise the authentication stories into Sprint 1, prioritising login and registration
git commit -m "docs: plan Sprint 1 with authentication stories"

# 3. Create a branch for the first user story
git checkout -b feature/US-001-user-login
```

### Phase 3: Planning the User Story

```bash
# 1. Plan the implementation
/agent:plan Plan the implementation for US-001: User Login with email/password
git commit -m "docs: add implementation plan for US-001"
```

### Phase 4: Test-Driven Development

```bash
# 1. Write tests first (TDD)
/agent:test-writer Write tests for the LoginController and AuthService
git commit -m "test: add login controller and auth service tests"

# 2. Generate documentation for the feature
/agent:docs Document the authentication API endpoints
git commit -m "docs: add authentication API documentation"
```

### Phase 5: Implementation

```bash
# 1. Implement backend code
/agent:backend Implement the LoginController to pass the tests
git commit -m "feat: implement login controller"

/agent:backend Implement the AuthService for password validation
git commit -m "feat: implement auth service"

# 2. Implement frontend code
/agent:frontend Create the login form component
git commit -m "feat: add login form component"

/agent:frontend Add form validation and error handling
git commit -m "feat: add login form validation"

# 3. Add logging
/agent:logging Add audit logging for login attempts
git commit -m "feat: add login audit logging"

# 4. Fix any syntax or linting issues
/agent:syntax Fix linting errors in the auth module
git commit -m "style: fix linting errors in auth module"
```

### Phase 6: Quality Assurance

```bash
# 1. Run QA testing
/agent:qa-tester Review the login implementation for security and edge cases
git commit -m "fix: address QA feedback on login security"

# 2. Perform manual tests and document results
# Edit docs/QA/MANUAL-TESTS.md with test results
git commit -m "docs: add manual test results for login"

# 3. Fix any bugs discovered
/agent:debug Fix the session timeout issue discovered in testing
git commit -m "fix: correct session timeout handling"
```

### Phase 7: Review and Refactoring

```bash
# 1. Code review
/agent:review Review the authentication module for SOLID principles and security
git commit -m "refactor: apply code review suggestions"

# 2. Refactor if needed
/agent:refactor Extract password validation logic into separate service
git commit -m "refactor: extract password validation service"

# 3. Update documentation
/agent:docs Update the auth documentation with the new service structure
git commit -m "docs: update auth documentation"

# 4. Generate support articles if user-facing
/agent:support-articles Write help article for "How to log in to your account"
git commit -m "docs: add login help article"
```

### Phase 8: Completion and PR

```bash
# 1. Mark the story as complete
/agent:completion Mark US-001 as complete with implementation notes
git commit -m "docs: mark US-001 as complete"

# 2. Push and create PR
git push -u origin feature/US-001-user-login

# Create the pull request
gh pr create --title "feat: US-001 User Login" --body "## Summary
- Implements email/password login
- Adds session management
- Includes audit logging

## Test Plan
- [x] Unit tests pass
- [x] Integration tests pass
- [x] Manual QA completed

## Stories
- Closes US-001"
```

### Workflow Summary

```
Repository Setup → User Stories → Sprint Planning → Create Branch
       ↓
   Planning → Tests (TDD) → Documentation → Implementation
       ↓
   Logging → Syntax Check → QA Testing → Manual Tests
       ↓
   Bug Fixes → Debugging → Code Review → Refactoring
       ↓
   Documentation Update → Support Articles → Completion → PR
```

---

## Quick Start

### 1. Initialise for Your Project

```bash
# Navigate to your project root
cd ~/my-project

# Run the init command
/plugin:init
```

This will:
- Detect your project stack
- Create `.claude/CLAUDE.md` from the appropriate template
- Set up container configuration (DDEV or Docker)
- Copy the Syntek Guide to `.claude/SYNTEK-GUIDE.md`

### 2. Start Developing

```bash
# Plan a feature
/agent:plan Add user authentication with social login

# Implement backend
/agent:backend Create User model and auth endpoints

# Implement frontend
/agent:frontend Create login form component

# Write tests
/agent:test-writer Write tests for auth flow

# Review code
/agent:qa-tester Review the auth implementation
```

---

## Installation

### Prerequisites

- Claude Code CLI installed
- Docker or DDEV (depending on your stack)
- Git

### Manual Setup (Alternative to /plugin:init)

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

### Command Types

| Prefix | Type | Description |
|--------|------|-------------|
| `/agent:` | Agent | Spawns a specialised AI agent |
| `/plugin:` | Plugin | Plugin management commands |
| `/learning:` | Learning | Self-learning system commands |

### Plugin Commands

| Command | Description |
|---------|-------------|
| `/plugin:init` | Initialise Syntek Dev Suite for a project |

### Agent Commands

#### Planning & Architecture

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:plan` | Opus | Create architectural plans, break down features |
| `/agent:stories` | Haiku | Generate user stories from requirements |
| `/agent:sprint` | Sonnet | Organise stories into balanced sprints |
| `/agent:completion` | Sonnet | Track story and sprint completion |

#### Development

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:setup` | Sonnet | Project initialisation and configuration |
| `/agent:backend` | Sonnet | Backend development, APIs, database |
| `/agent:frontend` | Sonnet | UI/UX, components, accessibility |
| `/agent:database` | Sonnet | Database design, migrations, optimisation |
| `/agent:auth` | Sonnet | Authentication, MFA, session management |

#### Quality & Testing

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:test-writer` | Sonnet | TDD test suites and stubs |
| `/agent:qa-tester` | Sonnet | Hostile QA, security, edge cases |
| `/agent:review` | Sonnet | Code review, SOLID, security |
| `/agent:debug` | Opus | Root cause analysis, debugging |

#### Refactoring & Maintenance

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:refactor` | Sonnet | Code cleanup without changing logic |
| `/agent:syntax` | Haiku | Fix syntax and linting errors |
| `/agent:docs` | Haiku | Technical documentation |

#### Infrastructure

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:cicd` | Sonnet | CI/CD pipelines, deployments |
| `/agent:security` | Sonnet | Access control, headers, rate limiting |
| `/agent:logging` | Sonnet | Logging, Sentry, audit trails |
| `/agent:git` | Sonnet | Branch management, versioning |

#### Specialised

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:gdpr` | Sonnet | GDPR compliance, data protection |
| `/agent:seo` | Sonnet | SEO, meta tags, structured data |
| `/agent:notifications` | Sonnet | Email, SMS, push notifications |
| `/agent:export` | Sonnet | PDF, Excel, CSV, JSON exports |
| `/agent:reporting` | Sonnet | Data queries, report services |
| `/agent:data` | Sonnet | Data analysis, Python, SQL |
| `/agent:support-articles` | Sonnet | Help documentation |

---

## Skills System

Skills provide stack-specific knowledge that agents use automatically.

### How Skills Work

1. Your project's `CLAUDE.md` specifies a `Skill Target` (e.g., `stack-tall`)
2. When an agent runs, it loads the stack skill and global workflow skill
3. The agent applies stack-specific patterns and conventions

### Available Skills

| Skill | Target | Applied To |
|-------|--------|------------|
| `stack-tall` | TALL Stack | Laravel, Livewire, Alpine, Tailwind |
| `stack-django` | Django | Django, Wagtail, PostgreSQL, GraphQL |
| `stack-react` | React | React, Next.js, TypeScript, Tailwind |
| `stack-mobile` | Mobile | React Native, Expo, NativeWind |
| `stack-shared-lib` | Shared Library | NPM packages for web/mobile |
| `global-workflow` | All | British English, Git, dates, currency |

---

## Templates

| Template | File | Container |
|----------|------|-----------|
| TALL | `templates/tall-project.md` | DDEV |
| Django | `templates/django-project.md` | Docker Compose |
| React | `templates/react-project.md` | Docker |
| Mobile | `templates/mobile-project.md` | Docker |
| Shared Lib | `templates/shared-lib-project.md` | Docker |

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
/learning:ab-test list

# Check test status for an agent
/learning:ab-test status backend

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
/learning:feedback good

# If the output needs improvement
/learning:feedback bad The output didn't follow the coding style
```

### Learning Commands

| Command | Description |
|---------|-------------|
| `/learning:feedback good` | Mark the last run as successful |
| `/learning:feedback bad [comment]` | Mark as needing improvement |
| `/learning:ab-test list` | List active A/B tests |
| `/learning:ab-test status <agent>` | Show test results for an agent |
| `/learning:optimise status` | Show optimisation system status |
| `/learning:optimise analyse <agent>` | Analyse an agent's performance |
| `/learning:optimise apply <id>` | Apply a pending optimisation |

### Project-Specific Learning

- All feedback and metrics are stored in your project's `docs/METRICS/` folder
- Data is committed to Git, so the whole team benefits from improvements
- Each project develops its own optimised prompts over time
- No external API calls - learning uses Claude Code CLI directly

---

## Plugin Architecture

### Directory Structure

```
syntek-dev-suite/
├── .claude-plugin/          # Plugin configuration
├── agents/                  # Agent definitions (28 agents)
├── commands/                # Slash commands
├── skills/                  # Stack-specific skills
├── templates/               # Project templates
├── examples/                # Code examples (80+)
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

1. **Always start with `/agent:plan`** - Get a roadmap before coding
2. **Use `/agent:qa-tester` before merging** - Catch issues early
3. **Keep CLAUDE.md updated** - Add new dependencies and constraints
4. **Let agents read files** - Don't paste code, reference files
5. **Give feedback** - Use `/learning:feedback` to improve agents over time
6. **Commit regularly** - Small, focused commits after each step

---

## Contributing

### Adding a New Agent

1. Create `agents/my-agent.md` with agent instructions
2. Create `commands/my-agent.md` with command definition
3. Test with `/agent:my-agent Test this new agent`

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
