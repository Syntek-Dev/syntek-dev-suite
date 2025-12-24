# Syntek Dev Suite - Plugin Usage Guide

**Version:** 1.1.0
**Plugin:** syntek-dev-suite
**Maintained by:** Syntek Developers

---

## Table of Contents

- [Syntek Dev Suite - Plugin Usage Guide](#syntek-dev-suite---plugin-usage-guide)
  - [Table of Contents](#table-of-contents)
  - [Quick Start](#quick-start)
    - [Basic Usage](#basic-usage)
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
    - [Version Commands](#version-commands)
    - [Version Files Managed](#version-files-managed)
    - [Markdown Metadata Headers](#markdown-metadata-headers)
  - [Skills Reference](#skills-reference)
    - [Stack Skills](#stack-skills)
    - [Global Skill](#global-skill)
  - [Self-Learning and A/B Testing](#self-learning-and-ab-testing)
    - [How It Works](#how-it-works)
    - [A/B Testing](#ab-testing)
    - [Giving Feedback](#giving-feedback)
    - [Project-Specific Learning](#project-specific-learning)
    - [Learning Best Practices](#learning-best-practices)
  - [Markdown All in One Extension](#markdown-all-in-one-extension)
    - [Key Features](#key-features)
    - [Keyboard Shortcuts](#keyboard-shortcuts)
    - [Table of Contents](#table-of-contents-1)
  - [Best Practices](#best-practices)
    - [1. Always Start with `/syntek-dev-suite:plan`](#1-always-start-with-syntek-dev-suiteplan)
    - [2. Use `/syntek-dev-suite:qa-tester` Before Merging](#2-use-syntek-dev-suiteqa-tester-before-merging)
    - [3. Keep CLAUDE.md Updated](#3-keep-claudemd-updated)
    - [4. Let Agents Read Files](#4-let-agents-read-files)
    - [5. Use the Right Model](#5-use-the-right-model)
    - [6. Chain Commands Logically](#6-chain-commands-logically)
    - [7. Commit After Each Step](#7-commit-after-each-step)
    - [8. Give Feedback](#8-give-feedback)
  - [Environment Commands](#environment-commands)
  - [Browser Configuration](#browser-configuration)
    - [Browser Environment Variable](#browser-environment-variable)
    - [Launching Chrome](#launching-chrome)
    - [Claude Code Chrome Integration](#claude-code-chrome-integration)
    - [E2E Test Configuration](#e2e-test-configuration)
  - [Getting Help](#getting-help)


---

## Quick Start

The Syntek Dev Suite provides specialised AI agents for full-stack development. Each agent has domain expertise and understands your project's stack through the `CLAUDE.md` file.

### Basic Usage

```bash
# Plan a new feature
/syntek-dev-suite:plan Add user authentication with social login

# Implement backend
/syntek-dev-suite:backend Create the User model and auth endpoints

# Implement frontend
/syntek-dev-suite:frontend Create the login form component

# Write tests
/syntek-dev-suite:test-writer Write tests for the auth flow

# Review code
/syntek-dev-suite:qa-tester Review the auth implementation

# Give feedback to improve the agent
/syntek-dev-suite:learning-feedback good
```

---

## Complete Development Workflow

This section walks through a complete development cycle from repository setup to PR submission.

### Phase 1: Repository Setup

```bash
# 1. Clone or create your repository
git clone git@github.com:your-org/your-project.git
cd your-project

# 2. Initialise the plugin for your project
/syntek-dev-suite:init

# 3. Edit .claude/CLAUDE.md with your project details
# Replace all [Insert...] placeholders

# 4. Create core Git branches
/syntek-dev-suite:git Create main, develop, staging, and production branches with appropriate protection rules
git commit -m "chore: initialise project structure"
```

### Phase 2: User Stories and Sprint Planning

```bash
# 1. Generate user stories from requirements
/syntek-dev-suite:stories Based on this PRD, create user stories for user authentication feature
git commit -m "docs: add user authentication stories"

# 2. Organise stories into sprints
/syntek-dev-suite:sprint Organise the authentication stories into Sprint 1, prioritising login and registration
git commit -m "docs: plan Sprint 1 with authentication stories"

# 3. Create a branch for the first user story
git checkout -b feature/US-001-user-login
```

### Phase 3: Planning the User Story

```bash
# 1. Plan the implementation
/syntek-dev-suite:plan Plan the implementation for US-001: User Login with email/password
git commit -m "docs: add implementation plan for US-001"
```

### Phase 4: Test-Driven Development

```bash
# 1. Write tests first (TDD)
/syntek-dev-suite:test-writer Write tests for the LoginController and AuthService
git commit -m "test: add login controller and auth service tests"

# 2. Generate documentation for the feature
/syntek-dev-suite:docs Document the authentication API endpoints
git commit -m "docs: add authentication API documentation"
```

### Phase 5: Implementation

```bash
# 1. Implement backend code
/syntek-dev-suite:backend Implement the LoginController to pass the tests
git commit -m "feat: implement login controller"

/syntek-dev-suite:backend Implement the AuthService for password validation
git commit -m "feat: implement auth service"

# 2. Implement frontend code
/syntek-dev-suite:frontend Create the login form component
git commit -m "feat: add login form component"

/syntek-dev-suite:frontend Add form validation and error handling
git commit -m "feat: add login form validation"

# 3. Add logging
/syntek-dev-suite:logging Add audit logging for login attempts
git commit -m "feat: add login audit logging"

# 4. Fix any syntax or linting issues
/syntek-dev-suite:syntax Fix linting errors in the auth module
git commit -m "style: fix linting errors in auth module"
```

### Phase 6: Quality Assurance

```bash
# 1. Run QA testing
/syntek-dev-suite:qa-tester Review the login implementation for security and edge cases
git commit -m "fix: address QA feedback on login security"

# 2. Perform manual tests and document results
# Edit docs/QA/MANUAL-TESTS.md with test results
git commit -m "docs: add manual test results for login"

# 3. Fix any bugs discovered
/syntek-dev-suite:debug Fix the session timeout issue discovered in testing
git commit -m "fix: correct session timeout handling"
```

### Phase 7: Review and Refactoring

```bash
# 1. Code review
/syntek-dev-suite:review Review the authentication module for SOLID principles and security
git commit -m "refactor: apply code review suggestions"

# 2. Refactor if needed
/syntek-dev-suite:refactor Extract password validation logic into separate service
git commit -m "refactor: extract password validation service"

# 3. Update documentation
/syntek-dev-suite:docs Update the auth documentation with the new service structure
git commit -m "docs: update auth documentation"

# 4. Generate support articles if user-facing
/syntek-dev-suite:support-articles Write help article for "How to log in to your account"
git commit -m "docs: add login help article"
```

### Phase 8: Completion and PR

```bash
# 1. Mark the story as complete
/syntek-dev-suite:completion Mark US-001 as complete with implementation notes
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

## Command Reference

All Syntek Dev Suite commands use the `/syntek-dev-suite:` prefix:

```bash
/syntek-dev-suite:<command> [arguments]
```

---

## Agent Commands

### Planning & Architecture

| Command | Model | Description |
|---------|-------|-------------|
| `/syntek-dev-suite:plan` | Opus | Create architectural plans, break down features |
| `/syntek-dev-suite:stories` | Haiku | Generate user stories from requirements |
| `/syntek-dev-suite:sprint` | Sonnet | Organise stories into balanced sprints |
| `/syntek-dev-suite:completion` | Sonnet | Track story and sprint completion |

### Development

| Command | Model | Description |
|---------|-------|-------------|
| `/syntek-dev-suite:setup` | Sonnet | Project initialisation and configuration |
| `/syntek-dev-suite:backend` | Sonnet | Backend development, APIs, database |
| `/syntek-dev-suite:frontend` | Sonnet | UI/UX, components, accessibility |
| `/syntek-dev-suite:database` | Sonnet | Database design, migrations, optimisation |
| `/syntek-dev-suite:auth` | Sonnet | Authentication, MFA, session management |

### Quality & Testing

| Command | Model | Description |
|---------|-------|-------------|
| `/syntek-dev-suite:test-writer` | Sonnet | TDD test suites and stubs |
| `/syntek-dev-suite:qa-tester` | Sonnet | Hostile QA, security, edge cases |
| `/syntek-dev-suite:review` | Sonnet | Code review, SOLID, security |
| `/syntek-dev-suite:debug` | Opus | Root cause analysis, debugging |

### Refactoring & Maintenance

| Command | Model | Description |
|---------|-------|-------------|
| `/syntek-dev-suite:refactor` | Sonnet | Code cleanup without changing logic |
| `/syntek-dev-suite:syntax` | Haiku | Fix syntax and linting errors |
| `/syntek-dev-suite:docs` | Haiku | Technical documentation |

### Infrastructure

| Command | Model | Description |
|---------|-------|-------------|
| `/syntek-dev-suite:cicd` | Sonnet | CI/CD pipelines, deployments |
| `/syntek-dev-suite:security` | Sonnet | Access control, headers, rate limiting |
| `/syntek-dev-suite:logging` | Sonnet | Logging, Sentry, audit trails |
| `/syntek-dev-suite:git` | Sonnet | Branch management, versioning |

### Specialised

| Command | Model | Description |
|---------|-------|-------------|
| `/syntek-dev-suite:gdpr` | Sonnet | GDPR compliance, data protection |
| `/syntek-dev-suite:seo` | Sonnet | SEO, meta tags, structured data |
| `/syntek-dev-suite:notifications` | Sonnet | Email, SMS, push notifications |
| `/syntek-dev-suite:export` | Sonnet | PDF, Excel, CSV, JSON exports |
| `/syntek-dev-suite:reporting` | Sonnet | Data queries, report services |
| `/syntek-dev-suite:data` | Sonnet | Data analysis, Python, SQL |
| `/syntek-dev-suite:support-articles` | Sonnet | Help documentation |

---

## Plugin Commands

| Command | Description |
|---------|-------------|
| `/syntek-dev-suite:init` | Initialise Syntek Dev Suite for a project |

---

## Learning Commands

The learning system helps agents improve over time based on your feedback.

| Command | Description |
|---------|-------------|
| `/syntek-dev-suite:learning-feedback good` | Mark the last run as successful |
| `/syntek-dev-suite:learning-feedback bad [comment]` | Mark as needing improvement with optional comment |
| `/syntek-dev-suite:learning-ab-test list` | List active A/B tests |
| `/syntek-dev-suite:learning-ab-test status <agent>` | Show test results for an agent |
| `/syntek-dev-suite:learning-optimise status` | Show optimisation system status |
| `/syntek-dev-suite:learning-optimise analyse <agent>` | Analyse an agent's performance |
| `/syntek-dev-suite:learning-optimise apply <id>` | Apply a pending optimisation |

---

## Version Management

The version agent manages semantic versioning, changelogs, and markdown headers across your project.

### Version Commands

| Command | Description |
|---------|-------------|
| `/syntek-dev-suite:version bump <type>` | Increment version (major, minor, patch) |
| `/syntek-dev-suite:version update` | Update all version files and documentation |
| `/syntek-dev-suite:version headers` | Update metadata headers in all .md files |
| `/syntek-dev-suite:version init` | Initialise version files for a new project |
| `/syntek-dev-suite:version status` | Show current version and pending changes |
| `/syntek-dev-suite:version history` | Show version history summary |

### Version Files Managed

| File | Purpose | Audience |
|------|---------|----------|
| **Version files** | Semantic version (package.json, etc.) | Build systems |
| **VERSION-HISTORY.md** | Technical change log with code details | Developers |
| **CHANGELOG.md** | Brief developer-focused summary | Developers |
| **RELEASES.md** | User-facing feature highlights | End users |

### Markdown Metadata Headers

All `.md` files include a metadata header maintained by the version agent:

```markdown
# Document Title

**Last Updated**: DD/MM/YYYY
**Version**: X.Y.Z
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---
```

---

## Skills Reference

Skills are loaded automatically based on your project's `Skill Target` in `CLAUDE.md`.

### Stack Skills

| Skill | Target | Applied To |
|-------|--------|------------|
| `stack-tall` | TALL Stack | Laravel, Livewire, Alpine, Tailwind |
| `stack-django` | Django Stack | Django, Wagtail, PostgreSQL, GraphQL |
| `stack-react` | React Stack | React, Next.js, TypeScript, Tailwind |
| `stack-mobile` | Mobile Stack | React Native, Expo, NativeWind |
| `stack-shared-lib` | Shared Library | NPM packages for web/mobile |

### Global Skill

The `global-workflow` skill is always loaded and provides:
- British English localisation
- Date format: DD/MM/YYYY
- Time format: 24-hour clock (14:30)
- Timezone: Europe/London
- Currency: GBP (£)
- Git commit message standards
- Documentation formatting rules
- Browser configuration (Chrome/Chrome Beta)

---

## Self-Learning and A/B Testing

The plugin includes a self-learning system that improves agent performance based on your feedback. Each project develops its own optimised prompts over time.

### How It Works

1. **Feedback Collection** - After each agent run, rate the output
2. **Metrics Recording** - Run duration, outcomes, and errors are tracked
3. **Pattern Analysis** - The system identifies what works and what doesn't
4. **A/B Testing** - Prompt variants are tested to find the best approach
5. **Prompt Optimisation** - Winning prompts are applied automatically

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

After each agent run, provide feedback to improve future runs:

```bash
# If the output was good
/syntek-dev-suite:learning-feedback good

# If the output needs improvement
/syntek-dev-suite:learning-feedback bad The output didn't follow the coding style

# If you can't evaluate yet
# (skip feedback - just don't run the command)
```

### Project-Specific Learning

- All feedback and metrics are stored in your project's `docs/METRICS/` folder
- Data is committed to Git, so the whole team benefits from improvements
- Each project develops its own optimised prompts over time
- No external API calls - learning uses Claude Code CLI directly
- The more feedback you provide, the better the agents become for your project

### Learning Best Practices

1. **Be consistent** - Always provide feedback after significant agent runs
2. **Be specific** - When marking as "bad", explain what was wrong
3. **Trust the process** - Improvements take time and multiple data points
4. **Check A/B tests** - Review test status periodically to see improvements

---

## Markdown All in One Extension

The Syntek Dev Suite is configured for optimal use with the **Markdown All in One** VS Code extension.

### Key Features

| Feature | Description |
|---------|-------------|
| Auto-updating TOCs | Table of Contents stays in sync with headings |
| Smart Lists | Auto-renumbering and intelligent indentation |
| Table Formatting | GFM tables auto-align on save |
| Task Lists | Toggle checkboxes with `Alt+C` |
| Math Support | Render LaTeX-style math expressions |

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+B` / `Cmd+B` | Toggle bold |
| `Ctrl+I` / `Cmd+I` | Toggle italic |
| `Tab` | Indent list item |
| `Shift+Tab` | Un-indent list item |
| `Alt+C` | Toggle task checkbox |

### Table of Contents

TOCs auto-update when you save. To exclude a heading from the TOC:

```markdown
## Internal Notes <!-- omit in toc -->
```

**Installation:** When opening a Syntek Dev Suite project in VS Code, you'll be prompted to install recommended extensions.

**Extension ID:** `yzhang.markdown-all-in-one`

For the complete guide, see [docs/GUIDES/MARKDOWN-ALL-IN-ONE.md](docs/GUIDES/MARKDOWN-ALL-IN-ONE.md).

---

## Best Practices

### 1. Always Start with `/syntek-dev-suite:plan`

For any non-trivial feature, get a roadmap before coding:

```bash
/syntek-dev-suite:plan Add password reset functionality
```

### 2. Use `/syntek-dev-suite:qa-tester` Before Merging

Catch issues early with hostile QA:

```bash
/syntek-dev-suite:qa-tester Review my changes before PR
```

### 3. Keep CLAUDE.md Updated

When you add dependencies or change frameworks, update `.claude/CLAUDE.md`:

```markdown
## Key Dependencies
- PDF: `barryvdh/laravel-dompdf`
- AI: `openai-php/laravel`  # Add new ones here
```

### 4. Let Agents Read Files

Don't paste code into commands. The agent will read files:

```bash
# Good
/syntek-dev-suite:backend Fix the UserController

# Less good
/syntek-dev-suite:backend Fix this code: [pasted code]
```

### 5. Use the Right Model

- **Opus** (complex): `/syntek-dev-suite:plan`, `/syntek-dev-suite:debug`, `/syntek-dev-suite:data`
- **Sonnet** (balanced): `/syntek-dev-suite:backend`, `/syntek-dev-suite:frontend`, `/syntek-dev-suite:qa-tester`
- **Haiku** (fast): `/syntek-dev-suite:docs`, `/syntek-dev-suite:syntax`, `/syntek-dev-suite:stories`

### 6. Chain Commands Logically

Follow the natural development flow:

```
plan → stories → sprint → backend → frontend → test → qa → review → complete
```

### 7. Commit After Each Step

Small, focused commits after each agent interaction:

```bash
/syntek-dev-suite:backend Create the User model
git commit -m "feat: add User model"

/syntek-dev-suite:test-writer Write tests for User model
git commit -m "test: add User model tests"
```

### 8. Give Feedback

Help agents improve by providing feedback:

```bash
/syntek-dev-suite:learning-feedback good   # When output is correct
/syntek-dev-suite:learning-feedback bad    # When output needs improvement
```

---

## Environment Commands

Each project has environment-specific scripts:

| Script | Purpose | Command |
|--------|---------|---------|
| `./dev.sh` | Start development | `ddev start` or `docker-compose up` |
| `./test.sh` | Run tests | Test suite with test database |
| `./staging.sh` | Staging build | Build + cache for staging |
| `./production.sh` | Production build | Build + optimise for production |

---

## Browser Configuration

**CRITICAL:** Always use Chrome for testing, debugging, and E2E tests. Never use Firefox unless explicitly requested.

### Browser Environment Variable

| Variable | Purpose | Detection |
|----------|---------|-----------|
| `CHROME_PATH` | Primary Chrome binary path | `./plugins/chrome-tool.py detect` |

```bash
# Detect Chrome and generate .env.chrome
./plugins/chrome-tool.py write
```

### Launching Chrome

```bash
# Standard Chrome for manual testing
$CHROME_PATH http://localhost:3000

# Chrome with DevTools for debugging
$CHROME_PATH --auto-open-devtools-for-tabs http://localhost:3000

# Chrome with specific viewport for responsive testing
$CHROME_PATH --window-size=375,812 http://localhost:3000  # iPhone X
$CHROME_PATH --window-size=768,1024 http://localhost:3000  # iPad

# Headless Chrome for automated tests
$CHROME_PATH --headless --disable-gpu --no-sandbox http://localhost:3000

# Chrome with remote debugging enabled
$CHROME_PATH --remote-debugging-port=9222 http://localhost:3000
```

### Claude Code Chrome Integration

Use `claude --chrome` to enable browser automation from the terminal:

```bash
# Start Claude Code with Chrome enabled
claude --chrome

# Check connection status
/chrome

# Enable Chrome by default
# Run /chrome and select "Enable by default"
```

### E2E Test Configuration

| Framework | Chrome Configuration |
|-----------|---------------------|
| Playwright | `channel: 'chrome'` or `executablePath: process.env.CHROME_PATH` |
| Cypress | `browser: 'chrome'` in config or `--browser chrome` CLI flag |
| Puppeteer | `executablePath: process.env.PUPPETEER_EXECUTABLE_PATH` |
| Selenium | `options.binary_location = os.environ.get('CHROME_PATH')` |
| Laravel Dusk | Uses `DUSK_CHROME_BINARY` env var automatically |

---

## Getting Help

- **Plugin Issues:** https://github.com/syntek-developers/syntek-dev-suite/issues
- **Documentation:** See `examples/` folder in the plugin directory
- **Reset to Default:** Delete `.claude/` folder and run `/plugin:init` again

---

**Happy coding with Syntek Dev Suite!**