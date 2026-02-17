# Changelog

**Last Updated**: 17/02/2026
**Version**: 1.5.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added

- Nothing yet

---

## [1.5.0] - 17/02/2026

### Added

- Coding principles reference document (`examples/setup/CODING-PRINCIPLES.md`) featuring Rob Pike's 5 Rules of Programming and Linus Torvalds' 6 coding rules
- Coding Principles section to all project templates (TALL, Django, React, Mobile, Shared Library) embedded CLAUDE.md content
- Coding Principles section to base CLAUDE.md template (`examples/setup/CLAUDE-MD-TEMPLATE.md`)
- Step in `/init` command to copy `CODING-PRINCIPLES.md` to new projects during initialisation
- Updated folder structure and output summary in `commands/init.md` to reflect CODING-PRINCIPLES inclusion

---

## [1.4.0] - 09/01/2026

### Added

- Plugin copy functionality in `/init` command - copies `syntek-dev-suite/plugins/*.py` to `.claude/plugins/` during project initialisation
- New Step 3.5 in init process for explicit plugin tool copying with `chmod +x` to make them executable
- Plugin Tools section in CLAUDE.md template with full plugin documentation and usage examples
- `.claude/plugins/` folder to standard project structure

### Changed

- Updated `commands/init.md` with plugin folder structure and copy instructions
- Updated `examples/setup/CLAUDE-MD-TEMPLATE.md` with Plugin Tools section and available plugins table
- Pre-flight commands now reference source syntek-dev-suite path correctly
- Added note clarifying plugins are available at `.claude/plugins/` after initialisation

### Documentation

- Added complete list of 12 available plugins with descriptions to CLAUDE.md template
- Added example bash commands for plugin usage in project CLAUDE.md files

---

## [1.3.1] - 29/12/2025

### Fixed
- Fixed hardcoded `/home/sam-dev/claude-dev-team` paths that prevented plugin from working when installed in different locations
- Updated `plugins/ab-test-tool.py` and `plugins/optimiser-tool.py` to use dynamic path detection via `Path(__file__).parent.parent`
- Changed all `python` commands to `python3` for cross-platform compatibility (Linux, macOS, Windows)
- Updated all agent files (30+) to use relative paths (`./skills/`, `./plugins/`, `./examples/`)
- Updated all command files to use `python3` for plugin tool execution
- Updated documentation to reference GitHub repository URL instead of hardcoded local paths

### Changed
- Plugin tools now use relative paths (`./plugins/`) instead of absolute paths in `config.json`
- All skill and example references now use relative paths for portability

---

## [1.3.0] - 28/12/2025

### Added
- Project Management (PM) Agent for issue tracking and project management integration
- PM tool plugin (`pm-tool.py`) for detecting ClickUp, Linear, Jira, GitHub Projects, Monday.com, Asana, Trello, Notion, Azure DevOps, and Shortcut
- PM Agent setup command (`/agent:pm-setup`) for configuring PM tool integration
- Updated all section README.md files across the repository for improved navigation

### Changed
- Updated CLAUDE.md with PM tool documentation and agent responsibilities
- Agent count increased from 29 to 30 specialised agents
- Plugin count now at 14 tools for comprehensive environment detection

---

## [1.2.0] - 24/12/2025

### Added
- Version Agent for semantic versioning and markdown metadata management
- Version command (`/version`) with bump, init, headers, and status subcommands
- Markdown metadata headers standard (Last Updated, Version, Maintained By, Language, Timezone)
- Markdown All in One VS Code extension integration for enhanced editing
- GitHub Flavoured Markdown (GFM) documentation standards
- VS Code recommended extensions configuration (`.vscode/extensions.json`)
- VS Code workspace settings for markdown formatting (`.vscode/settings.json`)
- Version examples: VERSION-HISTORY-TEMPLATE, RELEASES-TEMPLATE, GIT-INTEGRATION, MARKDOWN-HEADERS
- Implementation plan for Markdown All in One integration

### Changed
- Updated CLAUDE.md with version management documentation and markdown standards
- Enhanced git agent to call Version Agent before commits
- Updated README.md structure and content
- Improved setup templates with metadata header support
- Enhanced global workflow skill with version management patterns

---

## [1.1.0] - 24/12/2025

### Added
- Cross-platform Chrome detection with `chrome-tool.py` plugin
- Claude Code Chrome integration documentation in CLAUDE.md
- Browser configuration via environment variables (`CHROME_PATH`, `DUSK_CHROME_BINARY`, etc.)
- Chrome-based E2E testing examples for Playwright, Cypress, Puppeteer, Selenium, and Laravel Dusk
- Live debugging capabilities using Claude in Chrome extension
- Browser automation documentation for web app testing

### Changed
- Updated debugger agent with Chrome-based debugging workflow
- Updated QA tester agent with browser-based testing capabilities
- Updated test writer agent with Chrome E2E test examples
- Updated setup agent to run `chrome-tool.py` during initialisation
- Enhanced all example documentation with Table of Contents
- Improved global workflow skill with browser configuration patterns
- Updated all stack skills with Chrome testing patterns
- Refreshed all project templates with browser environment setup

### Fixed
- Standardised documentation structure across 78 files

---

## [1.0.0] - 21/12/2025

### Added
- Complete plugin architecture with 28 specialised agents
- Full command system with `/agent:`, `/plugin:`, and `/learning:` prefixes
- Self-learning system with A/B testing and prompt optimisation
- 80+ versioned code examples across all supported stacks
- 5 stack skills: TALL, Django, React, Mobile, Shared Library
- Global workflow skill for British English localisation
- 12 Python plugin tools for environment detection
- 5 project templates for quick initialisation
- Section README files for all folders
- Comprehensive root README with installation guide

### Changed
- Updated all Tailwind CSS references to 4.x
- Updated all NativeWind references to 4.x
- Standardised version notation across all examples

---

## [0.9.0] - 15/12/2025

### Added
- Support articles agent for help documentation
- Data scientist agent for Python/SQL analysis
- Reporting agent for data queries
- Export agent for PDF, Excel, CSV generation

### Changed
- Improved QA tester agent with security scanning
- Enhanced code reviewer with SOLID principles checks
- Updated debugger agent to use Opus model

### Fixed
- Timezone handling in logging examples
- Git hook examples for pre-commit validation

---

## [0.8.0] - 01/12/2025

### Added
- Self-learning system foundation
- Metrics tool for agent run tracking
- Feedback tool for user ratings
- A/B test tool for prompt variants
- Optimiser tool for prompt improvements
- Learning commands (`/learning:feedback`, `/learning:ab-test`, `/learning:optimise`)

### Changed
- All agents now record metrics after execution
- Feedback prompts appear after agent completion

---

## [0.7.0] - 15/11/2025

### Added
- GDPR compliance agent
- SEO agent with meta tag generation
- Notifications agent for email, SMS, push
- Authentication examples: Passkeys, Social Login

### Changed
- Improved security agent with rate limiting
- Enhanced backend agent with PII handling

### Fixed
- Session management examples for Laravel 12

---

## [0.6.0] - 01/11/2025

### Added
- Refactor agent for code cleanup
- Syntax agent for linting fixes
- Optimiser agent for performance
- Code reviewer agent with checklist

### Changed
- Test writer agent now generates stubs first
- QA tester agent includes accessibility checks

---

## [0.5.0] - 15/10/2025

### Added
- CI/CD agent with GitHub Actions support
- AWS deployment examples
- Digital Ocean deployment examples
- Docker multi-stage build examples

### Changed
- Setup agent now detects container platform
- Git agent improved with branching strategies

### Fixed
- DDEV configuration for PHP 8.4

---

## [0.4.0] - 01/10/2025

### Added
- Logging agent with structured logging
- Debugger agent using Opus model
- Git agent for workflow management
- Git workflow examples with pre-commit hooks

### Changed
- Backend agent now includes audit logging
- Frontend agent includes error boundaries

---

## [0.3.0] - 15/09/2025

### Added
- Test writer agent with TDD approach
- QA tester agent for hostile testing
- Testing examples for all stacks
- Code review checklist examples

### Changed
- All agents now read project CLAUDE.md first
- Improved stack detection in setup agent

### Fixed
- Django migration examples for PostgreSQL 18

---

## [0.2.0] - 01/09/2025

### Added
- Stack skills system (TALL, Django, React, Mobile, Shared Library)
- Global workflow skill for localisation
- Project templates for all stacks
- Database migration examples

### Changed
- Agents now apply skills automatically
- Commands use consistent prefixes

### Fixed
- TypeScript version compatibility in examples

---

## [0.1.0] - 15/08/2025

### Added
- Plugin architecture foundation
- Core agents: Planner, Backend, Frontend, Database
- Sprint and user story agents
- Completion tracking agent
- Setup agent for project initialisation

### Changed
- Migrated from single prompt to agent system

---

## [0.0.1] - 01/08/2025

### Added
- Initial project structure
- Plugin configuration files
- Basic command system
- Example folder structure

---

## [0.0.0] - 15/07/2025

### Added
- Project inception
- Initial design documents
- Technology stack decisions
- [[Unreleased]](#)
- [[1.1.0] - 24/12/2025](#)
- [[1.0.0] - 21/12/2025](#)
- [[0.9.0] - 15/12/2025](#)
- [[0.8.0] - 01/12/2025](#)
- [[0.7.0] - 15/11/2025](#)
- [[0.6.0] - 01/11/2025](#)
- [[0.5.0] - 15/10/2025](#)
- [[0.4.0] - 01/10/2025](#)
- [[0.3.0] - 15/09/2025](#)
- [[0.2.0] - 01/09/2025](#)
- [[0.1.0] - 15/08/2025](#)
- [[0.0.1] - 01/08/2025](#)
- [[0.0.0] - 15/07/2025](#)
