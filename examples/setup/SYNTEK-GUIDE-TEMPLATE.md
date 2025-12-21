# Syntek Dev Suite - Plugin Usage Guide

**Version:** 1.0.0
**Plugin:** syntek-dev-suite
**Maintained by:** Syntek Developers

---

## Table of Contents

- [Quick Start](#quick-start)
- [Complete Development Workflow](#complete-development-workflow)
- [Command Types](#command-types)
- [Agent Commands](#agent-commands)
- [Plugin Commands](#plugin-commands)
- [Learning Commands](#learning-commands)
- [Skills Reference](#skills-reference)
- [Self-Learning and A/B Testing](#self-learning-and-ab-testing)
- [Best Practices](#best-practices)
- [Environment Commands](#environment-commands)
- [Getting Help](#getting-help)

---

## Quick Start

The Syntek Dev Suite provides specialised AI agents for full-stack development. Each agent has domain expertise and understands your project's stack through the `CLAUDE.md` file.

### Basic Usage

```bash
# Plan a new feature
/agent:plan Add user authentication with social login

# Implement backend
/agent:backend Create the User model and auth endpoints

# Implement frontend
/agent:frontend Create the login form component

# Write tests
/agent:test-writer Write tests for the auth flow

# Review code
/agent:qa-tester Review the auth implementation

# Give feedback to improve the agent
/learning:feedback good
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

## Command Types

All Syntek commands are prefixed with their type for clarity:

| Prefix | Type | Description |
|--------|------|-------------|
| `/agent:` | Agent | Spawns a specialised AI agent |
| `/plugin:` | Plugin | Plugin management commands |
| `/learning:` | Learning | Self-learning system commands |

---

## Agent Commands

### Planning & Architecture

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:plan` | Opus | Create architectural plans, break down features |
| `/agent:stories` | Haiku | Generate user stories from requirements |
| `/agent:sprint` | Sonnet | Organise stories into balanced sprints |
| `/agent:completion` | Sonnet | Track story and sprint completion |

### Development

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:setup` | Sonnet | Project initialisation and configuration |
| `/agent:backend` | Sonnet | Backend development, APIs, database |
| `/agent:frontend` | Sonnet | UI/UX, components, accessibility |
| `/agent:database` | Sonnet | Database design, migrations, optimisation |
| `/agent:auth` | Sonnet | Authentication, MFA, session management |

### Quality & Testing

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:test-writer` | Sonnet | TDD test suites and stubs |
| `/agent:qa-tester` | Sonnet | Hostile QA, security, edge cases |
| `/agent:review` | Sonnet | Code review, SOLID, security |
| `/agent:debug` | Opus | Root cause analysis, debugging |

### Refactoring & Maintenance

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:refactor` | Sonnet | Code cleanup without changing logic |
| `/agent:syntax` | Haiku | Fix syntax and linting errors |
| `/agent:docs` | Haiku | Technical documentation |

### Infrastructure

| Command | Model | Description |
|---------|-------|-------------|
| `/agent:cicd` | Sonnet | CI/CD pipelines, deployments |
| `/agent:security` | Sonnet | Access control, headers, rate limiting |
| `/agent:logging` | Sonnet | Logging, Sentry, audit trails |
| `/agent:git` | Sonnet | Branch management, versioning |

### Specialised

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

## Plugin Commands

| Command | Description |
|---------|-------------|
| `/plugin:init` | Initialise Syntek Dev Suite for a project |

---

## Learning Commands

The learning system helps agents improve over time based on your feedback.

| Command | Description |
|---------|-------------|
| `/learning:feedback good` | Mark the last run as successful |
| `/learning:feedback bad [comment]` | Mark as needing improvement with optional comment |
| `/learning:ab-test list` | List active A/B tests |
| `/learning:ab-test status <agent>` | Show test results for an agent |
| `/learning:optimise status` | Show optimisation system status |
| `/learning:optimise analyse <agent>` | Analyse an agent's performance |
| `/learning:optimise apply <id>` | Apply a pending optimisation |

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

After each agent run, provide feedback to improve future runs:

```bash
# If the output was good
/learning:feedback good

# If the output needs improvement
/learning:feedback bad The output didn't follow the coding style

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

## Best Practices

### 1. Always Start with `/agent:plan`

For any non-trivial feature, get a roadmap before coding:

```bash
/agent:plan Add password reset functionality
```

### 2. Use `/agent:qa-tester` Before Merging

Catch issues early with hostile QA:

```bash
/agent:qa-tester Review my changes before PR
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
/agent:backend Fix the UserController

# Less good
/agent:backend Fix this code: [pasted code]
```

### 5. Use the Right Model

- **Opus** (complex): `/agent:plan`, `/agent:debug`, `/agent:data`
- **Sonnet** (balanced): `/agent:backend`, `/agent:frontend`, `/agent:qa-tester`
- **Haiku** (fast): `/agent:docs`, `/agent:syntax`, `/agent:stories`

### 6. Chain Commands Logically

Follow the natural development flow:

```
plan → stories → sprint → backend → frontend → test → qa → review → complete
```

### 7. Commit After Each Step

Small, focused commits after each agent interaction:

```bash
/agent:backend Create the User model
git commit -m "feat: add User model"

/agent:test-writer Write tests for User model
git commit -m "test: add User model tests"
```

### 8. Give Feedback

Help agents improve by providing feedback:

```bash
/learning:feedback good   # When output is correct
/learning:feedback bad    # When output needs improvement
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

## Getting Help

- **Plugin Issues:** https://github.com/syntek-developers/syntek-dev-suite/issues
- **Documentation:** See `examples/` folder in the plugin directory
- **Reset to Default:** Delete `.claude/` folder and run `/plugin:init` again

---

**Happy coding with Syntek Dev Suite!**