# Version History

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Unreleased](#unreleased)
- [1.3.0 - 28/12/2025](#130---28122025)
- [1.2.0 - 24/12/2025](#120---24122025)
- [1.1.0 - 24/12/2025](#110---24122025)
- [1.0.0 - 21/12/2025](#100---21122025)

---

## [Unreleased]

### Technical Changes
- Nothing yet

---

## [1.3.0] - 28/12/2025

### Summary
Added Project Management Agent and plugin for integration with popular PM tools. This release expands the agent suite to 30 specialised agents and 14 plugin tools, enabling seamless integration with issue tracking and project management platforms.

### Files Added

| File | Purpose |
|------|---------|
| `agents/pm.md` | Project Management Agent for issue tracking and PM tool integration |
| `plugins/pm-tool.py` | Plugin for detecting and integrating with PM tools |

### Files Changed

| File | Changes |
|------|---------|
| `CLAUDE.md` | Added `pm-tool.py` to plugin list, added PM Agent to agent responsibilities |
| `.claude-plugin/plugin.json` | Version bumped from 1.2.0 to 1.3.0 |
| `CHANGELOG.md` | Added version 1.3.0 release notes and metadata header |
| `agents/README.md` | Updated agent count to 30 |
| `plugins/README.md` | Updated plugin count to 14 |

### New Capabilities

**PM Tool Support:**
- ClickUp API integration
- Linear GraphQL API integration
- Jira REST API integration
- GitHub Projects API integration
- Monday.com API integration
- Asana API integration
- Trello API integration
- Notion API integration
- Azure DevOps REST API integration
- Shortcut API integration

**PM Agent Features:**
- Auto-detection of PM tool configuration via environment variables
- Issue creation and updates
- Sprint management
- Project board synchronisation
- Task assignment and tracking
- Custom field mapping
- Webhook configuration

### Configuration Changes

| Environment Variable | Purpose | Example |
|---------------------|---------|---------|
| `CLICKUP_API_TOKEN` | ClickUp authentication | `pk_123456789` |
| `LINEAR_API_KEY` | Linear authentication | `lin_api_123456789` |
| `JIRA_API_TOKEN` | Jira authentication | `ATATT3xFfGF0...` |
| `GITHUB_TOKEN` | GitHub Projects authentication | `ghp_123456789` |
| `MONDAY_API_TOKEN` | Monday.com authentication | `eyJhbGciOiJIUzI1NiJ9...` |
| `ASANA_ACCESS_TOKEN` | Asana authentication | `1/1234567890:abcdef...` |
| `TRELLO_API_KEY` | Trello authentication | `0123456789abcdef...` |
| `NOTION_API_KEY` | Notion authentication | `secret_123456789` |
| `AZURE_DEVOPS_PAT` | Azure DevOps authentication | `abcdefghijklmnop...` |
| `SHORTCUT_API_TOKEN` | Shortcut authentication | `sc_123456789` |

### Performance Notes
- PM tool detection runs asynchronously to avoid blocking
- API rate limiting implemented for all PM tool integrations
- Caching of PM tool metadata to reduce API calls

### Documentation Updates
- All README.md files updated with current version 1.3.0
- Section README files enhanced with improved structure
- CLAUDE.md updated with PM Agent usage patterns

---

## [1.2.0] - 24/12/2025

### Summary
Introduced Version Agent for comprehensive version management, including semantic versioning, markdown metadata headers, and documentation standards with Markdown All in One VS Code extension integration.

### Files Added

| File | Purpose |
|------|---------|
| `agents/version.md` | Version Agent for semantic versioning and markdown metadata |
| `examples/version/VERSION-HISTORY-TEMPLATE.md` | Template for technical version history |
| `examples/version/RELEASES-TEMPLATE.md` | Template for user-facing release notes |
| `examples/version/GIT-INTEGRATION.md` | Git workflow integration examples |
| `examples/version/MARKDOWN-HEADERS.md` | Markdown metadata header examples |
| `.vscode/extensions.json` | Recommended VS Code extensions |
| `.vscode/settings.json` | VS Code workspace settings for markdown |
| `docs/GUIDES/MARKDOWN-ALL-IN-ONE.md` | Markdown All in One feature guide |
| `docs/PLANS/MARKDOWN-ALL-IN-ONE-IMPLEMENTATION.md` | Implementation plan |

### Files Changed

| File | Changes |
|------|---------|
| `CLAUDE.md` | Added version management system documentation and markdown standards |
| `README.md` | Updated structure and content with version management info |
| `skills/global-workflow/SKILL.md` | Enhanced with version management patterns |

### New Features

**Version Management:**
- Semantic versioning (MAJOR.MINOR.PATCH)
- Automated version bumping (`/version bump <type>`)
- VERSION-HISTORY.md technical changelog
- RELEASES.md user-facing notes
- Markdown metadata header updates

**Markdown Standards:**
- Standardised metadata headers for all .md files
- Table of Contents requirement
- GitHub Flavoured Markdown (GFM) support
- Markdown All in One VS Code extension integration

**Metadata Header Fields:**
- Last Updated (DD/MM/YYYY)
- Version (X.Y.Z)
- Maintained By
- Language (British English en_GB)
- Timezone (Europe/London)

### Configuration Changes

| File | Key | Change |
|------|-----|--------|
| `.vscode/extensions.json` | recommendations | Added `yzhang.markdown-all-in-one` |
| `.vscode/settings.json` | `markdown.extension.*` | Configured TOC settings and table formatting |

---

## [1.1.0] - 24/12/2025

### Summary
Added cross-platform Chrome detection and Claude Code Chrome integration for browser automation and debugging.

### Files Added

| File | Purpose |
|------|---------|
| `plugins/chrome-tool.py` | Cross-platform Chrome detection plugin |

### Files Changed

| File | Changes |
|------|---------|
| `CLAUDE.md` | Added browser configuration and Chrome integration documentation |
| `agents/debugger.md` | Added Chrome-based debugging workflow |
| `agents/qa-tester.md` | Added browser-based testing capabilities |
| `agents/test-writer.md` | Added Chrome E2E test examples |
| `agents/setup.md` | Added `chrome-tool.py` execution during setup |

### New Features

**Chrome Detection:**
- Auto-detect Chrome installation across Windows, macOS, and Linux
- Generate `.env.chrome` with browser environment variables
- Support for Chrome, Chromium, Edge, and Brave

**Environment Variables:**
- `CHROME_PATH` - Primary Chrome binary path
- `CHROME_BINARY` - Alias for Chrome binary
- `DUSK_CHROME_BINARY` - Laravel Dusk Chrome path
- `PUPPETEER_EXECUTABLE_PATH` - Puppeteer Chrome path
- `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` - Playwright Chrome path

**Browser Capabilities:**
- Live debugging with console errors and DOM state
- Design verification against mockups
- Web app testing with form validation
- Authenticated access to web services
- Data extraction from web pages

---

## [1.0.0] - 21/12/2025

### Summary
Initial public release with complete plugin architecture, 28 specialised agents, self-learning system, and comprehensive stack support.

### Major Components

**Agent System:**
- 28 specialised agents for full-stack development
- Command system with `/agent:`, `/plugin:`, and `/learning:` prefixes
- Auto-detection of stack and environment

**Self-Learning System:**
- Metrics recording for agent runs
- User feedback collection
- A/B testing for prompt variants
- Automated prompt optimisation

**Stack Support:**
- TALL Stack (Tailwind 4, Alpine, Laravel 12, Livewire 3)
- Django Stack (Django 5.1, Wagtail 6.3, PostgreSQL 18)
- React Stack (Next.js 15, React 19, TypeScript 5)
- Mobile Stack (React Native 0.76, Expo 52, NativeWind 4)
- Shared Library Stack

**Plugin Tools:**
- 12 Python tools for environment detection
- DDEV, Docker, Git, Database, Environment, Logging
- Metrics, Feedback, Quality, A/B Testing, Optimiser

### Dependencies

| Component | Version |
|-----------|---------|
| Python | 3.10+ |
| Claude Code CLI | 2.0.73+ |
| Git | 2.x |
| VS Code | 1.85+ (recommended) |

### Performance Notes
- All plugin tools execute in < 100ms
- Markdown header updates process ~100 files/second
- Self-learning metrics stored in Git for team sharing
