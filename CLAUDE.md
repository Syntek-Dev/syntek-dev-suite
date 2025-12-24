# Claude Dev Team

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
| `metrics-tool.py` | Self-learning metrics recording and querying | `./plugins/metrics-tool.py status` |
| `feedback-tool.py` | User feedback collection and analysis | `./plugins/feedback-tool.py status` |
| `quality-tool.py` | Code quality checks and linting | `./plugins/quality-tool.py status` |
| `ab-test-tool.py` | A/B testing for agent prompt variants | `./plugins/ab-test-tool.py list` |
| `optimiser-tool.py` | Agent performance analysis and optimisation | `./plugins/optimiser-tool.py status` |
| `chrome-tool.py` | Cross-platform Chrome detection and configuration | `./plugins/chrome-tool.py detect` |

### When Agents Should Use Plugins

Agents should run these plugins to gather context before making decisions:

- **Setup Agent:** Run `project-tool.py`, `env-tool.py`, `docker-tool.py`, `ddev-tool.py`, `chrome-tool.py`
- **CI/CD Agent:** Run `git-tool.py`, `docker-tool.py`, `ddev-tool.py`
- **Database Agent:** Run `db-tool.py`, `env-tool.py`
- **Backend Agent:** Run `project-tool.py`, `db-tool.py`, `env-tool.py`
- **Logging Agent:** Run `log-tool.py`, `project-tool.py`
- **Debugger Agent:** Run `log-tool.py`, `env-tool.py`
- **Optimiser Agent:** Run `metrics-tool.py`, `feedback-tool.py`, `optimiser-tool.py`

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
**ALL documentation `.md` files MUST use CAPITALISED filename with lowercase `.md` extension.**

Examples:
- `README.md`
- `CONTRIBUTING.md`
- `API-REFERENCE.md`
- `GETTING-STARTED.md`
- `CHANGELOG.md`

**CRITICAL:** The filename is CAPITALISED, the `.md` extension is lowercase.

**ALL markdown documentation files MUST include a Table of Contents at the top of the document**, immediately after the main heading.

Format:
```markdown
# Document Title

## Section 1
...
```

### Documentation Folder Structure
All detailed documentation should be stored in the `docs/` folder with appropriate subfolders:

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
├── METRICS/              # Self-learning system data
│   ├── README.md
│   ├── config.json
│   ├── runs/
│   ├── feedback/
│   ├── aggregates/
│   ├── variants/
│   └── optimisations/
└── README.md
```

### Root README.md
The root-level `README.md` MUST:
1. Provide a project overview
2. Include quick start instructions
3. **Reference and link to documentation in the `docs/` folder**
4. Include a documentation section that lists available docs

Example:
```markdown
## Documentation

For detailed documentation, see the [docs/](docs/) folder:

- [API Documentation](docs/API/)
- [Setup Guides](docs/SETUP/)
- [Architecture Overview](docs/ARCHITECTURE/)
- [Contributing Guide](docs/GUIDES/CONTRIBUTING.md)
```

### Section README Files (CRITICAL)

**CRITICAL:** Every significant folder in the codebase MUST contain a `README.md` that explains what that section does and provides a tree layout of its contents.

#### Purpose

Section READMEs allow agents and developers to:
1. **Quickly understand** what a folder contains without reading every file
2. **Navigate efficiently** by seeing the folder structure at a glance
3. **Gain context** about how files in the folder relate to each other
4. **Reduce onboarding time** for new team members

#### When to Create Section READMEs

Create a README.md in a folder when:
- **The folder has 3+ files** - Worth documenting structure
- **The folder contains business logic** - Services, models, controllers, repositories
- **The folder is a module boundary** - Components, features, domains, modules
- **The folder contains configuration** - Config, settings, environment
- **New folders are created** - Part of any scaffolding or feature work

Do NOT create READMEs for:
- Vendor/node_modules folders
- Build output folders (dist, build, .next)
- Cache folders (.cache, __pycache__)
- IDE/editor folders (.vscode, .idea) unless project-specific

#### Section README Template

For the complete template with examples, see `examples/setup/SECTION-README-TEMPLATE.md`

Every Section README MUST follow this structure:

```markdown
# [Folder Name]

## Overview

Brief description of what this folder contains and its purpose.

---

## Directory Tree

\`\`\`
folder-name/
├── README.md
├── subfolder/
│   └── file.ext
└── file.ext
\`\`\`

---

## Files

| File/Folder | Purpose |
|-------------|---------|
| `file.ext` | Description |

---

## Usage

How to use the code in this folder.

---

## Related Sections

- [../related/](../related/) - Relationship description
```

#### Required Section READMEs by Stack

| Stack | Required Folders |
|-------|------------------|
| **All Stacks** | `src/` or `app/`, `config/`, `tests/`, `docs/`, `scripts/` |
| **React/Next.js** | `components/`, `hooks/`, `services/`, `utils/`, `types/` |
| **Laravel/PHP** | `app/Http/Controllers/`, `app/Models/`, `app/Services/`, `database/migrations/` |
| **Django/Python** | `apps/`, `api/`, `services/`, `utils/` |

#### Agent Responsibilities

- **Setup Agent (`/setup`)**: Creates initial Section READMEs when scaffolding projects
- **Doc Writer Agent (`/docs`)**: Creates and updates Section READMEs for existing folders
- **All Agents**: Read the README.md in each folder first to gain context before working

---

## Database Configuration

### Environment-Specific Databases
**CRITICAL:** Every project MUST have separate databases for each environment:

| Environment | Database Suffix | Purpose |
|-------------|-----------------|---------|
| Development | `_dev` | Local development work |
| Testing | `_test` | Automated and manual tests |
| Staging | `_staging` | Pre-production testing |
| Production | `_production` | Live data |

### Test Database Isolation
**CRITICAL:** All tests MUST use a dedicated test database that is:
- Separate from the development database
- Automatically created/migrated before test runs
- Cleared or reset between test suites
- Never used for manual development

Configuration example:
```
# .env.dev
DB_DATABASE=myapp_dev

# .env.test
DB_DATABASE=myapp_test

# .env.staging
DB_DATABASE=myapp_staging

# .env.production
DB_DATABASE=myapp_production
```

---

## Environment-Specific Settings Files

### Framework Settings Structure
**CRITICAL:** All projects MUST have environment-specific settings files using the appropriate naming convention for the language/framework.

### Python/Django/Wagtail
```
config/
├── settings/
│   ├── __init__.py
│   ├── base.py          # Shared settings
│   ├── development.py   # Development settings
│   ├── testing.py       # Test settings (uses test database)
│   ├── staging.py       # Staging settings
│   └── production.py    # Production settings
```

### PHP/Laravel
```
config/
├── app.php              # Base config (reads from env)
├── database.php         # Database config
└── environments/        # Optional environment overrides
    ├── development.php
    ├── testing.php
    ├── staging.php
    └── production.php
```

### Node.js/TypeScript
```
config/
├── index.ts             # Config loader
├── base.ts              # Shared settings
├── development.ts       # Development settings
├── testing.ts           # Test settings
├── staging.ts           # Staging settings
└── production.ts        # Production settings
```

### React Native/Expo
```
config/
├── index.ts             # Config loader
├── base.ts              # Shared settings
├── development.ts       # Development settings
├── staging.ts           # Staging settings
└── production.ts        # Production settings
```

---

## Changelog System

### CHANGELOG.md Requirements
**CRITICAL:** Every project MUST maintain a `CHANGELOG.md` in the root directory following the [Keep a Changelog](https://keepachangelog.com/) format.

### Format
```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New features that have been added

### Changed
- Changes in existing functionality

### Deprecated
- Features that will be removed in future versions

### Removed
- Features that have been removed

### Fixed
- Bug fixes

### Security
- Security-related changes

## [1.0.0] - 2025-01-15

### Added
- Initial release
```

### Changelog Update Rules
1. All changes MUST be logged before merging to main/staging branches
2. Group changes by type (Added, Changed, Fixed, etc.)
3. Include issue/ticket references where applicable
4. Use present tense ("Add feature" not "Added feature")
5. Keep entries concise but descriptive

---

## Self-Learning System

The plugin includes a self-learning system that improves agent performance based on user feedback. Learning data is stored in `docs/METRICS/` and committed to Git, enabling team-wide improvements.

### Overview

The self-learning system:
- **Collects metrics** on agent runs (duration, outcomes, errors)
- **Gathers feedback** from developers after each agent run
- **Analyses patterns** to identify improvement opportunities
- **Proposes optimisations** to agent prompts based on data
- **Supports A/B testing** of prompt variants

### Folder Structure

```
docs/METRICS/
├── README.md              # System documentation
├── config.json            # Configuration settings
├── runs/                  # Agent run records
├── feedback/              # User feedback data
├── aggregates/            # Daily/weekly summaries
│   ├── daily/
│   └── weekly/
├── variants/              # A/B test prompt variants
└── optimisations/         # Improvement proposals
    ├── pending/
    ├── applied/
    └── rejected/
```

### Configuration

The `docs/METRICS/config.json` file controls system behaviour:

```json
{
  "enabled": true,
  "feedback_prompt_enabled": true,
  "feedback_required": true,
  "ab_testing_enabled": true,
  "auto_optimisation_enabled": true,
  "min_runs_for_analysis": 50,
  "retention_days": 90
}
```

| Setting | Description |
|---------|-------------|
| `enabled` | Master switch for the learning system |
| `feedback_prompt_enabled` | Show feedback prompt after agent runs |
| `feedback_required` | Whether feedback is required to continue |
| `ab_testing_enabled` | Enable A/B testing of prompt variants |
| `auto_optimisation_enabled` | Auto-apply high-confidence improvements |
| `min_runs_for_analysis` | Minimum runs before analysis is triggered |
| `retention_days` | How long to keep metrics data |

### Learning Commands

| Command | Description |
|---------|-------------|
| `/learning:feedback good` | Mark the last run as successful |
| `/learning:feedback bad` | Mark the last run as needing improvement |
| `/learning:feedback skip` | Skip feedback for this run |
| `/learning:ab-test list` | List active A/B tests |
| `/learning:ab-test status <agent>` | Show test results for an agent |
| `/learning:optimise status` | Show optimisation system status |
| `/learning:optimise analyse <agent>` | Analyse an agent's performance |
| `/learning:optimise apply <id>` | Apply a pending optimisation |

### Feedback Criteria

When giving feedback, consider:

| Rating | Criteria |
|--------|----------|
| **Good** | Task completed correctly, followed patterns, no fixes needed |
| **Bad** | Errors occurred, wrong approach, significant manual fixes needed |
| **Skip** | Task was abandoned, not enough context to judge |

### Agent Responsibilities

All agents should:
1. **Record metrics** at the start and end of runs
2. **Support feedback** collection after completion
3. **Follow optimised prompts** when available
4. **Report quality metrics** for code changes

The Optimiser Agent specifically:
1. Analyses feedback patterns across the team
2. Proposes targeted prompt improvements
3. Manages A/B tests for variant comparison
4. Applies auto-optimisations when confidence is high

### Data Privacy

- All learning data is stored locally in `docs/METRICS/`
- Data is committed to Git for team-wide sharing
- No external API calls are made for learning
- Sensitive information is never included in metrics

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
- [Localisation Settings](#)
- [Plugin Tools](#)
- [Documentation Standards](#)
- [Section 1](#)
- [Documentation](#)
- [Overview](#)
- [Directory Tree](#)
- [Files](#)
- [Usage](#)
- [Related Sections](#)
- [Database Configuration](#)
- [Environment-Specific Settings Files](#)
- [Changelog System](#)
- [[Unreleased]](#)
- [[1.0.0] - 2025-01-15](#)
- [Self-Learning System](#)
- [Multi-Repository Architecture](#)
- [Code Context and Navigation](#)
- [Project Stack](#)
- [Code Conventions](#)
