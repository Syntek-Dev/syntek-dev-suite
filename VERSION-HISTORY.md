# Version History

**Last Updated**: 09/01/2026
**Version**: 1.4.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Unreleased](#unreleased)
- [1.4.0 - 09/01/2026](#140---09012026)
- [1.3.1 - 29/12/2025](#131---29122025)
- [1.3.0 - 28/12/2025](#130---28122025)
- [1.2.0 - 24/12/2025](#120---24122025)
- [1.1.0 - 24/12/2025](#110---24122025)
- [1.0.0 - 21/12/2025](#100---21122025)

---

## [Unreleased]

### Technical Changes

- Nothing yet

---

## [1.4.0] - 09/01/2026

### Summary

Feature release adding plugin copy functionality to the `/init` command. Projects initialised with Syntek Dev Suite now receive a local copy of all Python plugin tools, making projects portable and self-contained.

### Files Changed

| File | Changes |
|------|---------|
| `commands/init.md` | Added Step 3.5 for plugin copying, updated folder structure, fixed pre-flight paths |
| `examples/setup/CLAUDE-MD-TEMPLATE.md` | Added Plugin Tools section with usage examples and plugin table |
| `VERSION` | Bumped from 1.3.1 to 1.4.0 |
| `CHANGELOG.md` | Added 1.4.0 release notes |
| `VERSION-HISTORY.md` | Added 1.4.0 technical details |
| `RELEASES.md` | Added 1.4.0 user-facing notes |

### New Features

#### Plugin Copy on Init

The `/init` command now copies all Python plugins to the target project:

```bash
# New Step 3.5 in init process
mkdir -p .claude/plugins/
cp /path/to/syntek-dev-suite/plugins/*.py .claude/plugins/
chmod +x .claude/plugins/*.py
```

**Benefits:**

- Projects are self-contained and portable
- Works without syntek-dev-suite being installed
- Allows project-specific plugin customisation
- Version stability - project keeps its plugin versions

#### CLAUDE.md Template Updates

The CLAUDE.md template now includes:

1. **Updated folder structure** with `.claude/plugins/` directory
2. **Plugin Tools section** with usage examples
3. **Available Plugins table** listing all 12 plugins

**New template sections:**

- Plugin Tools section with bash usage examples
- Available Plugins table with all 12 plugins listed
- Updated folder structure showing `.claude/plugins/` directory

### Technical Details

**Init Process Changes:**

| Step | Previous                | New                                               |
|------|-------------------------|---------------------------------------------------|
| 3    | Create folder structure | Create folder structure (now includes `plugins/`) |
| 3.5  | N/A                     | Copy plugin tools and make executable             |
| 4    | Copy templates          | Copy templates (unchanged)                        |

**Pre-flight Path Correction:**

The pre-flight commands now correctly reference the source syntek-dev-suite path:

```bash
# Old (incorrect - target path before init)
python3 .claude/plugins/project-tool.py info

# New (correct - source path during init)
python3 /path/to/syntek-dev-suite/plugins/project-tool.py info
```

**Plugins Included:**

All 14 Python plugins are copied:

- `project-tool.py` - Project and framework detection
- `db-tool.py` - Database detection
- `env-tool.py` - Environment file management
- `git-tool.py` - Git repository status
- `ddev-tool.py` - DDEV container status
- `docker-tool.py` - Docker container status
- `log-tool.py` - Log file discovery
- `metrics-tool.py` - Self-learning metrics
- `feedback-tool.py` - User feedback collection
- `quality-tool.py` - Code quality checks
- `chrome-tool.py` - Chrome browser detection
- `pm-tool.py` - PM tool detection
- `ab-test-tool.py` - A/B testing management
- `optimiser-tool.py` - Prompt optimisation

### Migration Notes

**For Existing Projects:**
Run the plugin copy commands manually to add plugins to existing projects:

```bash
mkdir -p .claude/plugins/
cp /path/to/syntek-dev-suite/plugins/*.py .claude/plugins/
chmod +x .claude/plugins/*.py
```

**For New Projects:**
Simply run `/init` and plugins will be copied automatically.

---

## [1.3.1] - 29/12/2025

### Summary
Critical bug fix release addressing path portability issues that prevented the plugin from working when installed in different locations. All hardcoded paths have been replaced with dynamic path detection, and Python command compatibility has been improved for cross-platform support.

### Bug Fixes

#### Path Portability

**Problem:** The plugin used hardcoded paths (`/home/sam-dev/claude-dev-team`) that prevented it from working when installed in different locations.

**Solution:** Replaced all hardcoded paths with dynamic path detection using `os.getcwd()` and `Path(__file__).parent.parent`.

| Issue | Fix | Benefit |
|-------|-----|---------|
| Plugin tools referenced absolute paths | Use `Path(__file__).parent.parent` for plugin directory | Works from any installation location |
| Agent files referenced `/home/sam-dev/claude-dev-team` | Use relative paths (`./skills/`, `./plugins/`, `./examples/`) | Portable across different systems |
| Config file used absolute paths | Use relative paths (`./plugins/`) | Configuration remains valid across installations |
| Documentation contained hardcoded paths | Reference GitHub repository URL | Users can find the project regardless of local path |

#### Python Command Compatibility

**Problem:** Commands used `python` which may not exist on systems where only `python3` is available (common on Linux/macOS).

**Solution:** Changed all Python command invocations to use `python3` explicitly.

| Change | Reason |
|--------|--------|
| `python` → `python3` in config.json | Ensures compatibility with systems that only have `python3` |
| `python` → `python3` in all agent files | Consistent command usage across all agents |
| `python` → `python3` in command files | Works on Linux, macOS, and Windows with Python 3 alias |

### Files Changed

| File | Changes |
|------|---------|
| `plugins/ab-test-tool.py` | Replaced `os.getcwd()` path assumptions with `Path(__file__).parent.parent` for metrics directory |
| `plugins/optimiser-tool.py` | Replaced `os.getcwd()` path assumptions with `Path(__file__).parent.parent` for metrics directory |
| `config.json` | Updated all `python` commands to `python3`, changed absolute paths to relative (`./plugins/`) |
| `agents/*.md` (30+ files) | Updated skill references from `/home/sam-dev/claude-dev-team` to `./skills/`, `./plugins/`, `./examples/` |
| `commands/*.md` (30+ files) | Updated plugin tool commands to use `python3` |
| `README.md` | Replaced hardcoded path references with GitHub repository URL |
| `templates/README.md` | Updated documentation to reference GitHub repository |
| `commands/init.md` | Updated init documentation with portable path examples |
| `.claude-plugin/README.md` | Added installation path guidance and GitHub repository link |

### Technical Details

**Dynamic Path Detection Pattern:**
```python
# Old (hardcoded)
metrics_dir = "/home/sam-dev/claude-dev-team/docs/METRICS"

# New (dynamic)
from pathlib import Path
plugin_root = Path(__file__).parent.parent
metrics_dir = plugin_root / "docs" / "METRICS"
```

**Relative Path Pattern:**
```markdown
# Old (absolute)
Read `/home/sam-dev/claude-dev-team/skills/stack-tall/SKILL.md`

# New (relative)
Read `./skills/stack-tall/SKILL.md`
```

**Python Command Pattern:**
```bash
# Old
python ./plugins/git-tool.py status

# New
python3 ./plugins/git-tool.py status
```

### Configuration Changes

| File | Key | Change |
|------|-----|--------|
| `config.json` | `tools.*.command` | Changed `python` to `python3` for all plugin tool commands |
| `config.json` | `tools.*.command` | Changed `/home/sam-dev/claude-dev-team/plugins/` to `./plugins/` |

### Testing Notes

This release has been tested with:
- Installation in different directory paths
- Python 3.10, 3.11, 3.12 on Linux
- Both `python` and `python3` command availability scenarios
- Relative path resolution from plugin root directory

### Migration Notes

**For Existing Users:**
No migration required. The changes are backwards compatible and will automatically work with your existing installation.

**For New Users:**
The plugin now works correctly regardless of installation location. Simply clone the repository to any directory and the plugin will function properly.

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
