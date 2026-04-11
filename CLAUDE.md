# Claude Dev Team

**Last Updated**: 11/04/2026
**Version**: 1.12.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Required Reference Documents

All agents MUST read the following documents before writing code, tests, or reviews for any project initialised with Syntek Dev Suite. These files are copied to `.claude/` in every project by `/init`.

| Document | Purpose |
|----------|---------|
| `CODING-PRINCIPLES.md` | Software design principles (SOLID, CUPID, GRASP, DDD, Unix Philosophy, Twelve-Factor), coding instincts (Pike, Torvalds), everyday rules (DRY, KISS, YAGNI, Beck's Four Rules), naming conventions, error handling, logging, git workflow, and code review checklist |
| `TESTING.md` | Testing matrix per layer (Python/Django, TypeScript/React, React Native, GraphQL), database isolation, migration testing, factories, property-based testing, coverage thresholds, mocking philosophy, snapshot testing, boundary testing, accessibility testing, performance testing, flaky test policy, and CI integration |
| `SECURITY.md` | Secrets management, authentication (argon2id/scrypt/bcrypt hierarchy), cryptography standards (approved and banned algorithms), transport security, API and GraphQL security, file upload security, browser storage policy, container security, OWASP Top 10:2025 mitigations, Row Level Security (RLS) requirements, supply chain security, data classification, security logging, and incident response |
| `ACCESSIBILITY.md` | WCAG 2.2 AA compliance, semantic HTML, ARIA patterns, keyboard navigation, focus management, forms, colour contrast, images and media, motion, React component patterns, React Native mobile accessibility, server-rendered accessibility (Django/Laravel), testing, and checklist |
| `API-DESIGN.md` | REST conventions (URL structure, HTTP methods, status codes, pagination, filtering), GraphQL conventions (schema, queries, mutations, error unions), error response format, authentication and authorisation, rate limiting, versioning, webhooks, API documentation, and client-side consumption patterns |
| `ARCHITECTURE-PATTERNS.md` | Service layer, middleware and request pipeline, frontend state management, routing conventions, background job patterns, email and notification patterns, file processing pipelines, and project structure (Django, Laravel, React) |
| `DATA-STRUCTURES.md` | Fundamental structures, domain modelling (value objects, aggregates, enums), database schema design (PostgreSQL and MariaDB/MySQL), normalisation, indexes, migrations, soft deletes, multi-tenancy, anti-patterns, and refactoring guidance |
| `PERFORMANCE.md` | Database query optimisation (N+1, EXPLAIN, indexing), caching (hierarchy, application, HTTP, invalidation), frontend performance (bundle size, code splitting, rendering strategy, React patterns), image optimisation, background jobs and queues, connection pooling, mobile performance, monitoring, load testing, and checklist |
| `DEVELOPMENT.md` | Development workflow, environment setup, container commands, and common tasks |
| `SEO-CHECKLIST.md` | SEO & AI discoverability checklist — Beginner through Advanced (including GEO and llms.txt) |

Templates for these files live in `examples/setup/` and are automatically copied to new projects by the `/init` command.

> **Coding Principles:** The `CODING-PRINCIPLES.md` file is the definitive reference for all code quality and design decisions. It is organised in layers — Pike and Torvalds guide low-level coding instincts; SOLID and GRASP guide class and module design; CUPID acts as a quality check across all principles; Package Principles guide monorepo and library architecture; DDD guides system boundaries and naming; the Unix Philosophy and Twelve-Factor App guide deployment and composition; DRY, KISS, YAGNI, and Beck's Four Rules guide everyday decisions. All agents MUST consult this document before writing, reviewing, or refactoring code. The Code Review Agent (`/syntek-dev-suite:review`) and Refactor Agent (`/syntek-dev-suite:refactor`) use it as their primary reference.

> **SEO Practices:** The `SEO-CHECKLIST.md` file is the definitive reference for all SEO and AI discoverability work. The SEO agent (`/syntek-dev-suite:seo`) reads this checklist before every implementation. See `examples/seo/SEO.md` for framework-specific code examples.

---

## Localisation Settings

All agents MUST read and apply these settings when generating content, code, documentation, or any user-facing output.

### Language
- **Written Language:** British English (UK English)
- Use British spelling conventions:
  - `colour` not `color`
  - `organisation` not `organization`
  - `analyse` not `analyze`
  - `behaviour` not `behavior`
  - `centre` not `center`
  - `licence` (noun) / `license` (verb)
  - `defence` not `defense`
  - `travelled` not `traveled`

### Date and Time Format
- **Date Format:** `DD/MM/YYYY` (e.g., `25/12/2025`)
- **Alternative Date Format:** `DD Month YYYY` (e.g., `25 December 2025`)
- **Time Format:** 24-hour clock (e.g., `14:30` not `2:30 PM`)
- **DateTime Format:** `DD/MM/YYYY HH:MM` (e.g., `25/12/2025 14:30`)
- **ISO DateTime:** Use when required for APIs/databases: `YYYY-MM-DDTHH:MM:SS`
- **Timezone:** `Europe/London` (GMT/BST)

### Currency
- **Primary Currency:** GBP (British Pound Sterling)
- **Currency Symbol:** `£`
- **Format:** `£1,234.56`
- **Decimal Separator:** `.` (period)
- **Thousands Separator:** `,` (comma)

### Measurement Units
- Use metric system where applicable (metres, kilometres, kilograms)
- Use UK conventions for paper sizes (A4, A5)

### Browser Configuration

**Environment Variables:** Browser paths are configured via environment variables for cross-platform compatibility.

| Variable | Purpose | Fallback |
|----------|---------|----------|
| `CHROME_PATH` | Primary Chrome binary path | Auto-detected |
| `CHROME_BINARY` | Alias for Chrome binary | `$CHROME_PATH` |
| `DUSK_CHROME_BINARY` | Laravel Dusk Chrome path | `$CHROME_PATH` |
| `PUPPETEER_EXECUTABLE_PATH` | Puppeteer Chrome path | `$CHROME_PATH` |
| `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` | Playwright Chrome path | `$CHROME_PATH` |

**Chrome Detection:** Run `./plugins/chrome-tool.py detect` to auto-detect Chrome on any OS.

```bash
# Detect Chrome and generate env file
./plugins/chrome-tool.py write

# Output: Creates .env.chrome with all browser variables
```

**CRITICAL:** When launching browsers for testing, debugging, or E2E tests, ALWAYS use Chrome via the environment variable. Do NOT use Firefox or other browsers unless explicitly requested.

**Usage Examples:**
```bash
# Launch Chrome for manual testing (using env var)
$CHROME_PATH http://localhost:3000

# Or use the detected command
google-chrome http://localhost:3000

# Launch Chrome with DevTools for debugging
$CHROME_PATH --auto-open-devtools-for-tabs http://localhost:3000

# Headless Chrome for automated tests
$CHROME_PATH --headless --disable-gpu --no-sandbox http://localhost:3000

# Chrome with remote debugging enabled
$CHROME_PATH --remote-debugging-port=9222 http://localhost:3000
```

**E2E Test Configuration:**
| Framework | Chrome Configuration |
|-----------|---------------------|
| Playwright | `channel: 'chrome'` or `executablePath: process.env.CHROME_PATH` |
| Cypress | `browser: 'chrome'` in `cypress.config.js` |
| Puppeteer | `executablePath: process.env.PUPPETEER_EXECUTABLE_PATH` |
| Selenium | `ChromeOptions()` with `os.environ.get('CHROME_PATH')` |
| Laravel Dusk | Uses `DUSK_CHROME_BINARY` env var automatically |

### Claude Code Chrome Integration

Claude Code integrates with the **Claude in Chrome** browser extension for browser automation from the terminal.

**Prerequisites:**
- Google Chrome browser
- Claude in Chrome extension (v1.0.36+) from Chrome Web Store
- Claude Code CLI (v2.0.73+)
- Paid Claude plan (Pro, Team, or Enterprise)

**Setup:**
```bash
# Update Claude Code
claude update

# Start Claude Code with Chrome enabled
claude --chrome

# Check connection status
/chrome
```

**Enable Chrome by Default:**
Run `/chrome` and select "Enable by default" to auto-enable Chrome integration on startup.

**Capabilities:**
- Live debugging (console errors, DOM state)
- Design verification against mocks
- Web app testing (form validation, user flows)
- Authenticated access (Google Docs, Gmail, Notion, etc.)
- Data extraction from web pages
- Task automation and session recording

**Example Usage:**
```bash
# Test local web app
I just updated the login form. Open localhost:3000, try invalid data, check error messages.

# Extract data
Go to the product page and extract name, price, availability as CSV.
```

**Troubleshooting:**
| Issue | Solution |
|-------|----------|
| Extension not detected | Verify v1.0.36+, restart Chrome, run `/chrome` → "Reconnect" |
| Browser not responding | Check for modal dialogs, create new tab, restart extension |

---

## Plugin Tools

The `plugins/` directory contains Python utilities that agents can use to gather information about the development environment. These tools return structured JSON output.

### Available Plugins

| Plugin | Purpose | Usage |
|--------|---------|-------|
| `ddev-tool.py` | DDEV project status and configuration | `./plugins/ddev-tool.py status` |
| `docker-tool.py` | Docker containers, compose, images, networks | `./plugins/docker-tool.py status` |
| `git-tool.py` | Git repository status, branches, remotes | `./plugins/git-tool.py status` |
| `env-tool.py` | Environment file discovery and validation | `./plugins/env-tool.py find` |
| `db-tool.py` | Database type and ORM detection | `./plugins/db-tool.py detect` |
| `project-tool.py` | Language, framework, and structure detection | `./plugins/project-tool.py info` |
| `log-tool.py` | Log file discovery and analysis | `./plugins/log-tool.py find` |
| `quality-tool.py` | Code quality checks and linting | `./plugins/quality-tool.py status` |
| `chrome-tool.py` | Cross-platform Chrome detection and configuration | `./plugins/chrome-tool.py detect` |
| `pm-tool.py` | PM tool detection (ClickUp, Linear, Jira, etc.) | `./plugins/pm-tool.py detect` |

### When Agents Should Use Plugins

Agents should run these plugins to gather context before making decisions:

- **Setup Agent:** Run `project-tool.py`, `env-tool.py`, `docker-tool.py`, `ddev-tool.py`, `chrome-tool.py`
- **CI/CD Agent:** Run `git-tool.py`, `docker-tool.py`, `ddev-tool.py`
- **Database Agent:** Run `db-tool.py`, `env-tool.py`
- **Backend Agent:** Run `project-tool.py`, `db-tool.py`, `env-tool.py`
- **Logging Agent:** Run `log-tool.py`, `project-tool.py`
- **Debugger Agent:** Run `log-tool.py`, `env-tool.py`
- **Version Agent:** Run `git-tool.py`, `project-tool.py`
- **Git Agent:** Run `git-tool.py` then call Version Agent before commits
- **PM Agent:** Run `pm-tool.py`, `project-tool.py`, `env-tool.py`

### Plugin Usage Example

```bash
# Get project information
./plugins/project-tool.py info

# Check database configuration
./plugins/db-tool.py detect

# Find environment files and validate
./plugins/env-tool.py find
./plugins/env-tool.py validate .env .env.example
```

---

## Documentation Standards

### Markdown File Naming

**ALL documentation `.md` files MUST use CAPITALISED filename with lowercase `.md` extension** (e.g., `README.md`, `CONTRIBUTING.md`, `API-REFERENCE.md`).

**ALL markdown documentation files MUST include a Table of Contents** immediately after the main heading.

### Markdown Metadata Headers

**CRITICAL:** All `.md` files MUST include a standardised metadata header after the main title:

```markdown
# Document Title

**Last Updated**: DD/MM/YYYY
**Version**: X.Y.Z
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---
```

The Version Agent (`/version`) automatically updates `Version` and `Last Updated` when running `/version bump` or `/version headers`. If editing manually, update `Last Updated` to today's date.

### Markdown Format

All markdown files use GitHub Flavoured Markdown (GFM). Always specify the language for fenced code blocks. The **Markdown All in One** VS Code extension (`yzhang.markdown-all-in-one`) is recommended — see [docs/GUIDES/MARKDOWN-ALL-IN-ONE.md](docs/GUIDES/MARKDOWN-ALL-IN-ONE.md) for details.

### Documentation Folder Structure

```
docs/
├── API/
├── SETUP/
├── ARCHITECTURE/
├── GUIDES/
├── TESTS/
├── QA/
├── PLANS/
├── DEVOPS/
├── DATABASE/
└── README.md
```

The root-level `README.md` MUST provide a project overview, quick start instructions, and link to documentation in the `docs/` folder.

### Section README Files

**CRITICAL:** Every significant folder (3+ files, business logic, module boundaries, configuration) MUST contain a `README.md` explaining its purpose and providing a directory tree. See `examples/setup/SECTION-README-TEMPLATE.md` for the template.

- **Setup Agent (`/setup`)**: Creates initial Section READMEs when scaffolding
- **Doc Writer Agent (`/docs`)**: Creates and updates Section READMEs
- **All Agents**: Read the folder's README.md first for context before working

---

## Database and Environment Configuration

Database naming, test database isolation, and framework-specific settings file structures are documented in `DEVELOPMENT.md`. Key rules:

- Every project MUST have separate databases per environment (suffixed `_dev`, `_test`, `_staging`, `_production`)
- All tests MUST use a dedicated test database, never the development database
- All projects MUST have environment-specific settings files per framework convention

---

## Version Management System

The Version Agent (`/version`) manages all version-related files. Every project MUST maintain: version files, `VERSION-HISTORY.md`, `CHANGELOG.md` ([Keep a Changelog](https://keepachangelog.com/) format), and `RELEASES.md`. See `examples/version/` for templates.

| Command | Description |
|---------|-------------|
| `/version bump major` | Increment major version (breaking changes) |
| `/version bump minor` | Increment minor version (new features) |
| `/version bump patch` | Increment patch version (bug fixes) |
| `/version headers` | Update metadata headers in all .md files |
| `/version init` | Initialise version files for new project |
| `/version status` | Show current version and pending changes |

**CRITICAL:** The Git Agent MUST call `/version bump <type>` before creating commits. The Git Agent analyses staged changes, determines the increment type, the Version Agent updates all version files, then the Git Agent creates the commit.

### Changelog Update Rules

1. All changes MUST be logged before merging to main/staging branches
2. Group changes by type (Added, Changed, Fixed, etc.)
3. Include issue/ticket references where applicable
4. Use imperative mood ("Add feature" not "Added feature")
5. Keep entries concise but descriptive
6. **Date format:** DD/MM/YYYY (per localisation settings)

---

## Multi-Repository Architecture

### GraphQL for Inter-Service Communication
**CRITICAL:** In multi-repository setups, all services MUST communicate via GraphQL APIs.

### Requirements
1. **API Gateway:** Use a GraphQL gateway/federation for cross-service queries
2. **Schema Registry:** Maintain a shared schema repository or registry
3. **Introspection:** Enable schema introspection in development/staging
4. **Documentation:** Auto-generate API documentation from GraphQL schemas

### Recommended Stack
| Component | Options |
|-----------|---------|
| Gateway | Apollo Federation, GraphQL Mesh, Hasura |
| Client | Apollo Client, urql, graphql-request |
| Schema | GraphQL Code Generator for types |

### Multi-Repo Structure
```
organisation/
├── api-gateway/          # GraphQL gateway/federation
├── service-users/        # User service with GraphQL endpoint
├── service-orders/       # Orders service with GraphQL endpoint
├── service-products/     # Products service with GraphQL endpoint
├── shared-schemas/       # Shared GraphQL types and fragments
└── frontend-web/         # Web frontend consuming GraphQL
```

### GraphQL Best Practices
1. Use DataLoader for N+1 query prevention
2. Implement proper error handling with GraphQL errors
3. Add query complexity analysis and depth limiting
4. Use persisted queries in production
5. Implement field-level authorisation

---

## Code Context and Navigation

### Finding Code Context
When investigating bugs, implementing features, or understanding the codebase:
1. **Read Section READMEs first** - Check for `README.md` in the folder you're working with to understand its structure and purpose
2. **Check the `docs/` folder** - Code context documentation can be found here to help guide where to look for fixes
3. Look for relevant documentation in subfolders like `docs/ARCHITECTURE/`, `docs/API/`, or `docs/GUIDES/`
4. Use the documentation to understand the structure before diving into source code

### Efficient File Reading
When reading source files:
1. **Read the folder's README.md first** - This provides an overview of the folder's contents and structure
2. **Read the summary at the top of each file** - Most files contain a summary, description, or module docstring at the top explaining the file's purpose
3. Use this summary to determine if the file is relevant before reading the entire contents
4. This approach saves time and context when navigating large codebases

### Agent Context Loading Order
All agents should follow this order when loading context:
1. **Project `CLAUDE.md`** - Global settings and conventions
2. **Folder `README.md`** - Section-specific structure and purpose
3. **File headers/docstrings** - Individual file purpose
4. **Full file content** - Only when necessary

---

## Project Stack

*(To be filled in per project)*

- **Language:**
- **Framework:**
- **Database:**
- **Container:**

---

## Code Conventions

*(To be defined per project)*
