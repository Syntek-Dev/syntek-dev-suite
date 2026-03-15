# Version History

**Last Updated**: 15/03/2026
**Version**: 1.8.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Unreleased](#unreleased)
- [1.8.0 - 15/03/2026](#180---15032026)
- [1.7.0 - 13/03/2026](#170---13032026)
- [1.6.0 - 24/02/2026](#160---24022026)
- [1.5.0 - 17/02/2026](#150---17022026)
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

## [1.8.0] - 15/03/2026

### Summary

Feature release adding five new reference documents to `examples/setup/` — Accessibility, API Design, Architecture Patterns, Data Structures, and Performance. These documents are registered in the Required Reference Documents section of `CLAUDE.md` and `CLAUDE-MD-TEMPLATE.md`, and are intended to be copied into new projects by the `/init` command. Existing reference documents (CODING-PRINCIPLES, DEVELOPMENT, SECURITY, TESTING) have been substantially expanded with additional content.

### Files Added

| File | Changes |
|------|---------|
| `examples/setup/ACCESSIBILITY.md` | New file: WCAG 2.2 compliance guide covering ARIA roles, keyboard navigation, colour contrast ratios, screen reader support, focus management, and stack-specific accessibility patterns (TALL/Blade, Django/templates, React, React Native) |
| `examples/setup/API-DESIGN.md` | New file: API design reference covering RESTful URL conventions, HTTP method semantics, versioning strategies, request/response patterns, authentication headers, standardised error response format, pagination, rate limiting, and GraphQL design principles |
| `examples/setup/ARCHITECTURE-PATTERNS.md` | New file: Architecture patterns guide covering layered architecture, Domain-Driven Design (DDD) with bounded contexts, CQRS, event-driven patterns, repository pattern, service layer design, and hexagonal architecture with per-stack implementation notes |
| `examples/setup/DATA-STRUCTURES.md` | New file: Data structures reference covering standard structure selection, serialisation formats (JSON, MessagePack), DTO patterns, value objects, collection handling, and stack-specific type conventions (PHP typed arrays, Python dataclasses/TypedDict, TypeScript interfaces) |
| `examples/setup/PERFORMANCE.md` | New file: Performance guide covering caching strategies (in-memory, HTTP cache headers, database query caching), N+1 query prevention via eager loading, lazy vs eager loading trade-offs, database indexing conventions, asset optimisation, Core Web Vitals targets (LCP < 2.5s, CLS < 0.1, INP < 200ms), and stack-specific performance patterns |

### Files Changed

| File | Changes |
|------|---------|
| `CLAUDE.md` | Added all five new reference documents to the Required Reference Documents table; updated version header to 1.8.0 |
| `examples/setup/CLAUDE-MD-TEMPLATE.md` | Added all five new reference documents to the Required Reference Documents section |
| `examples/setup/CODING-PRINCIPLES.md` | Expanded with additional coding principle sections; updated version header to 1.8.0 |
| `examples/setup/DEVELOPMENT.md` | Added database configuration conventions and environment-specific settings file structure; updated version header to 1.8.0 |
| `examples/setup/SECURITY.md` | Expanded with additional security patterns and per-stack guidance; updated version header to 1.8.0 |
| `examples/setup/TESTING.md` | Expanded with additional test patterns and tooling coverage; updated version header to 1.8.0 |
| `VERSION` | Bumped from 1.7.0 to 1.8.0 |
| `.claude-plugin/plugin.json` | Version bumped from 1.7.0 to 1.8.0 |
| `CHANGELOG.md` | Added 1.8.0 release notes |
| `VERSION-HISTORY.md` | Added 1.8.0 technical details (this file) |
| `RELEASES.md` | Added 1.8.0 user-facing release notes |

### New Reference Documents

#### Accessibility (`examples/setup/ACCESSIBILITY.md`)

A comprehensive WCAG 2.2 accessibility reference providing:

- **WCAG 2.2 Principles** — Perceivable, Operable, Understandable, Robust (POUR) with level targets (A, AA, AAA)
- **ARIA Roles and Attributes** — Landmark roles, widget roles, live regions, and correct attribute usage
- **Keyboard Navigation** — Focus order, keyboard traps, skip links, focus indicators
- **Colour Contrast** — Minimum contrast ratios (4.5:1 normal text, 3:1 large text/UI), tools for checking compliance
- **Screen Reader Support** — Alt text, form labels, heading hierarchy, announcements for dynamic content
- **Focus Management** — Programmatic focus for modals, drawers, route changes
- **Stack-Specific Patterns** — Blade/TALL, Django templates, React (`aria-*` props, `useRef`), React Native (`accessible`, `accessibilityLabel`)

#### API Design (`examples/setup/API-DESIGN.md`)

A RESTful and GraphQL API design reference providing:

- **URL Conventions** — Plural nouns, kebab-case, versioning prefix (`/api/v1/`), nested resources
- **HTTP Method Semantics** — GET, POST, PUT, PATCH, DELETE with idempotency notes
- **Response Status Codes** — Standard codes per operation type
- **Request/Response Patterns** — Consistent JSON envelope format, field naming (snake_case vs camelCase)
- **Authentication** — Bearer tokens, API key headers, refresh token patterns
- **Error Response Format** — Standardised error envelope (`error.code`, `error.message`, `error.details`)
- **Pagination** — Cursor-based and offset-based patterns with metadata
- **Rate Limiting** — Headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`)
- **GraphQL Design** — Query/mutation naming, input types, error handling, N+1 prevention via DataLoader

#### Architecture Patterns (`examples/setup/ARCHITECTURE-PATTERNS.md`)

An architectural guidance reference providing:

- **Layered Architecture** — Presentation, Application, Domain, Infrastructure layers with dependency rules
- **Domain-Driven Design (DDD)** — Bounded contexts, aggregates, entities, value objects, domain events, repositories
- **CQRS** — Command and Query separation, command handlers, query handlers, event sourcing introduction
- **Event-Driven Patterns** — Domain events, integration events, event bus, pub/sub patterns
- **Repository Pattern** — Interface contracts, concrete implementations, unit-of-work
- **Service Layer** — Application services vs domain services, orchestration vs domain logic
- **Hexagonal Architecture** — Ports and adapters, inbound/outbound ports, dependency inversion

#### Data Structures (`examples/setup/DATA-STRUCTURES.md`)

A data structure selection and usage reference providing:

- **Structure Selection Guide** — When to use arrays/lists, maps/dicts, sets, queues, stacks, trees
- **Serialisation Formats** — JSON conventions, camelCase vs snake_case boundaries, date serialisation (ISO 8601)
- **DTO Patterns** — Data Transfer Object conventions per stack (PHP typed classes, Python dataclasses, TypeScript interfaces)
- **Value Objects** — Immutable value types, equality by value, validation in constructor
- **Collection Handling** — Pagination cursors, filtering, sorting conventions
- **Stack-Specific Types** — PHP typed arrays and enums, Python TypedDict and dataclasses, TypeScript discriminated unions and generics

#### Performance (`examples/setup/PERFORMANCE.md`)

A performance optimisation reference providing:

- **Caching Strategies** — In-memory caching (Redis/Memcached), HTTP cache headers (`Cache-Control`, `ETag`), query result caching, fragment caching
- **N+1 Prevention** — Eager loading patterns per stack (Laravel `with()`, Django `select_related`/`prefetch_related`, Prisma `include`)
- **Lazy vs Eager Loading Trade-offs** — When each is appropriate, memory vs query count
- **Database Indexing** — Composite indexes, covering indexes, partial indexes, when not to index
- **Asset Optimisation** — Image formats (WebP, AVIF), lazy loading, code splitting, tree shaking
- **Core Web Vitals Targets** — LCP < 2.5s, CLS < 0.1, INP < 200ms with measurement tools
- **Stack-Specific Patterns** — Laravel Octane, Django caching framework, Next.js ISR/SSG, React Native Hermes and FlashList

### Version Header Updates

The following files had their metadata headers updated to 1.8.0:

- `CLAUDE.md`
- `CHANGELOG.md`
- `VERSION-HISTORY.md`
- `RELEASES.md`
- `examples/setup/CODING-PRINCIPLES.md`
- `examples/setup/DEVELOPMENT.md`
- `examples/setup/SECURITY.md`
- `examples/setup/TESTING.md`
- `examples/setup/ACCESSIBILITY.md`
- `examples/setup/API-DESIGN.md`
- `examples/setup/ARCHITECTURE-PATTERNS.md`
- `examples/setup/DATA-STRUCTURES.md`
- `examples/setup/PERFORMANCE.md`

Files intentionally NOT updated (no standard version header):
- `examples/setup/CLAUDE-MD-TEMPLATE.md`
- `commands/init.md`

---

## [1.7.0] - 13/03/2026

### Summary

Feature release adding a comprehensive SEO & AI Discoverability Checklist covering Beginner through Advanced tiers, including Generative Engine Optimisation (GEO), `llms.txt` creation, AI crawler permissions, and BLUF content strategy. The SEO agent is updated to audit projects against this checklist before implementation, and the `/init` command now copies the checklist to every new project's `.claude/` directory.

### Files Added

| File | Changes |
|------|---------|
| `examples/setup/SEO-CHECKLIST.md` | New file: full SEO and AI discoverability checklist covering Beginner (essential meta, robots.txt, llms.txt, sitemaps), Intermediate (structured data, performance, analytics), and Advanced (GEO, Core Web Vitals, multi-language, content quality) tiers |

### Files Changed

| File | Changes |
|------|---------|
| `agents/seo.md` | Added Step 2 to load `.claude/SEO-CHECKLIST.md` as audit baseline before implementation; added `llms.txt` / `llms-full.txt` to additional SEO files section; added AI Discoverability (GEO) section; updated description and introduction to reference AI discoverability; renumbered subsequent context-loading steps |
| `commands/seo.md` | Added checklist-first audit step; added `llms.txt` / `llms-full.txt` creation and GEO capabilities to the agent's feature list; updated description to include AI discoverability |
| `commands/init.md` | Added `SEO-CHECKLIST.md` to `.claude/` folder structure; added copy step `cp $SYNTEK_DIR/examples/setup/SEO-CHECKLIST.md .claude/SEO-CHECKLIST.md`; updated copy comment from "four" to "five" required reference files; added row to output summary table |
| `CLAUDE.md` | Added `SEO-CHECKLIST.md` to Required Reference Documents table; added SEO Practices callout block referencing the checklist and `examples/seo/SEO.md` |
| `VERSION` | Bumped from 1.6.0 to 1.7.0 |
| `CHANGELOG.md` | Added 1.7.0 release notes |
| `VERSION-HISTORY.md` | Added 1.7.0 technical details (this file) |
| `RELEASES.md` | Added 1.7.0 user-facing release notes |

### New Features

#### SEO & AI Discoverability Checklist (`examples/setup/SEO-CHECKLIST.md`)

A structured checklist covering three tiers of SEO and AI discoverability work:

**Beginner — Search Engine SEO**
- Essential root files: `robots.txt`, `sitemap.xml`, `favicon.ico`
- Meta tags: `<title>`, `<meta description>`, canonical URL
- Open Graph and Twitter Card tags
- Google Search Console verification

**Beginner — AI Discoverability**
- `llms.txt` — Markdown content summary for LLM agents, placed at site root
- `llms-full.txt` — Extended content version for AI agents
- AI crawler permissions in `robots.txt` (GPTBot, ClaudeBot, PerplexityBot, Google-Extended)
- BLUF content strategy — direct answers in the first paragraph

**Intermediate — Search Engine SEO**
- Structured data (JSON-LD): `WebSite`, `Organization`, `BreadcrumbList`, `Article`
- XML sitemaps with image and video extensions
- Core Web Vitals baseline (LCP < 2.5s, CLS < 0.1, INP < 200ms)
- `humans.txt` and `security.txt`

**Intermediate — AI Discoverability**
- Question-format headings and FAQ sections
- Comparison tables for decision content
- "Last updated" timestamps on content pages
- Server-side rendering — no critical content behind JavaScript

**Advanced — Search Engine SEO**
- International SEO: `hreflang` tags, geotargeting
- Advanced schema: `FAQPage`, `HowTo`, `Product`, `Review`, `Event`
- Video SEO with structured data
- Log file analysis and crawl budget optimisation

**Advanced — AI Discoverability (GEO)**
- Direct answer boxes and entity disambiguation
- Structured citation-ready content
- Knowledge graph entity creation
- Conversational keyword and prompt-style content targeting
- AI overview monitoring

#### Root Files Quick Reference

The checklist includes a quick reference table of all expected root files with their purpose:

| File | Purpose |
|------|---------|
| `robots.txt` | Crawler permissions including AI bots |
| `sitemap.xml` | Page index for search engines |
| `llms.txt` | Content summary for LLM agents |
| `llms-full.txt` | Extended content for AI agents |
| `humans.txt` | Team and technology credits |
| `security.txt` | Security contact information |
| `favicon.ico` | Browser tab icon |

#### SEO Agent Checklist-First Workflow

The SEO agent's context-loading sequence now includes reading the checklist as Step 2 (before loading stack skills), making it the audit baseline for every implementation:

```
1. Load CLAUDE.md and identify Skill Target
2. Read .claude/SEO-CHECKLIST.md → use as audit baseline
3. Load relevant stack skill (stack-tall, stack-django, stack-react)
4. Load global workflow skill
5. Run plugin tools
```

#### Init Command Changes

| Previous (1.6.0) | New (1.7.0) |
|-----------------|------------|
| Copy 4 reference files to `.claude/` | Copy 5 reference files to `.claude/` |
| CODING-PRINCIPLES, TESTING, SECURITY, DEVELOPMENT | + SEO-CHECKLIST |

### Version Header Updates

The following files had their metadata headers updated to 1.7.0:

- `CLAUDE.md`
- `CHANGELOG.md`
- `VERSION-HISTORY.md`
- `RELEASES.md`

---

## [1.6.0] - 24/02/2026

### Summary

Feature release adding three new reference documents — Testing, Security, and Development guides — plus a Required Reference Documents section to `CLAUDE.md` and templates. The `CODING-PRINCIPLES.md` has been substantially expanded, and the `/init` command now copies all four reference files into new projects automatically.

### Files Added

| File | Changes |
|------|---------|
| `examples/setup/TESTING.md` | New file: full testing guide for all 5 stacks (TALL, Django, React, Mobile, Shared Library) covering tooling, directory structure, TDD methodology, mock patterns, and test requirements |
| `examples/setup/SECURITY.md` | New file: web application security guide covering OWASP Top 10 mitigations, secrets management, authentication and authorisation patterns, input validation, CSRF, CORS, and stack-specific security practices |
| `examples/setup/DEVELOPMENT.md` | New file: development workflow guide covering environment setup, container commands (DDEV/Docker), branching workflow, environment variables, debugging, and common development tasks across all stacks |

### Files Changed

| File | Changes |
|------|---------|
| `examples/setup/CODING-PRINCIPLES.md` | Expanded from slim overview to full guide; added error handling section, naming conventions (per language), security coding principles, dependency management rules, git workflow conventions, DRY/KISS/YAGNI rules, logging standards, and a code review checklist |
| `examples/setup/CLAUDE-MD-TEMPLATE.md` | Added Required Reference Documents section listing all 4 files agents must read; ensures new projects carry the reference table in their `CLAUDE.md` |
| `commands/init.md` | Updated Step 3.5 (copy setup examples) to also copy `TESTING.md`, `SECURITY.md`, and `DEVELOPMENT.md` into `.claude/`; updated folder structure and output summary to reflect 4 reference files |
| `CLAUDE.md` | Added Required Reference Documents section at top with table listing all 4 reference files and their purposes |
| `VERSION` | Bumped from 1.5.0 to 1.6.0 |
| `CHANGELOG.md` | Added 1.6.0 release notes |
| `VERSION-HISTORY.md` | Added 1.6.0 technical details (this file) |
| `RELEASES.md` | Added 1.6.0 user-facing notes |

### New Features

#### Testing Guide (`examples/setup/TESTING.md`)

Comprehensive testing reference covering all five supported stacks:

- **Stack-specific tooling tables** — PHPUnit/Pest, pytest, Jest/Vitest, Detox per stack
- **Directory structure conventions** — Where to place unit, integration, and E2E tests
- **TDD methodology** — Red-Green-Refactor workflow with examples
- **Mock and stub patterns** — Per-stack examples (Mockery, unittest.mock, Jest mocks, MSW)
- **Test requirements** — Minimum coverage expectations, CI gate requirements
- **Test naming conventions** — `test_*`, `it_*`, `describe/it` per framework

#### Security Guide (`examples/setup/SECURITY.md`)

Web application security reference aligned with OWASP Top 10:

- **Secrets management** — Never commit secrets, use `.env`, vault patterns, rotation
- **Authentication and authorisation** — Token storage, session management, role checks
- **Input validation and sanitisation** — Per-stack injection prevention examples
- **CSRF protection** — Token patterns for TALL (Livewire), Django, and React (axios)
- **CORS configuration** — Allowed origins, headers, preflight handling
- **SQL injection prevention** — ORM usage, parameterised queries, raw query avoidance
- **XSS prevention** — Template escaping, CSP headers, DOMPurify usage
- **Stack-specific patterns** — Laravel middleware, Django security settings, Next.js headers
- **Dependency security** — `npm audit`, `pip-audit`, `composer audit` workflows
- **Deployment security checklist** — Pre-go-live security gates

#### Development Workflow Guide (`examples/setup/DEVELOPMENT.md`)

Day-to-day development reference for all stacks:

- **Prerequisites** — Required tools per stack with version guidance
- **Getting started** — Clone, env setup, container start, migration run
- **Container commands** — DDEV and Docker Compose cheat sheet
- **Branching workflow** — `us###/feature` → testing → dev → staging → main
- **Environment variables** — Required vars per stack, `.env` copy patterns
- **Debugging** — Stack-specific debug tips (Telescope, Django Debug Toolbar, React DevTools)
- **Database operations** — Migration, seeding, and reset commands
- **Common development tasks** — Frequently needed commands per stack

#### Expanded Coding Principles

`CODING-PRINCIPLES.md` expanded from ~80 lines to a comprehensive guide:

| Section Added | Content |
|---------------|---------|
| Error handling | Never swallow exceptions, always log with context, use typed exceptions |
| Naming conventions | Per-language rules (PHP PascalCase/camelCase, Python snake_case, TypeScript camelCase) |
| Security principles | Input validation, no secrets in code, principle of least privilege |
| Dependency management | Pin versions, audit regularly, minimise dependencies |
| Git conventions | Conventional Commits format, branch naming, commit scope |
| DRY/KISS/YAGNI | Practical guidelines with anti-pattern examples |
| Logging standards | Log levels, structured logging, what to log and what not to log |
| Code review checklist | Pre-PR checklist agents and developers must verify |

#### Required Reference Documents in CLAUDE.md

A new section at the top of the root `CLAUDE.md` and in `CLAUDE-MD-TEMPLATE.md` instructs all agents to read the four reference files before writing code:

```markdown
| Document | Purpose |
|----------|---------|
| CODING-PRINCIPLES.md | Coding standards, naming conventions, error handling |
| TESTING.md           | Testing patterns, tooling, TDD methodology           |
| SECURITY.md          | Security requirements, OWASP mitigations             |
| DEVELOPMENT.md       | Development workflow, environment setup              |
```

### Init Command Changes

Step 3.5 of the `/init` process now copies all four reference documents:

| Previous (1.5.0) | New (1.6.0) |
|-----------------|------------|
| Copy `CODING-PRINCIPLES.md` only | Copy `CODING-PRINCIPLES.md`, `TESTING.md`, `SECURITY.md`, `DEVELOPMENT.md` |
| 1 reference file in `.claude/` | 4 reference files in `.claude/` |

### Version Header Updates

The following files had their metadata headers updated to 1.6.0:

- `CLAUDE.md`
- `CHANGELOG.md`
- `VERSION-HISTORY.md`
- `RELEASES.md`
- `examples/setup/CODING-PRINCIPLES.md`
- `examples/setup/TESTING.md`
- `examples/setup/SECURITY.md`
- `examples/setup/DEVELOPMENT.md`

Files intentionally NOT updated (no standard version header):
- `commands/init.md`
- `examples/setup/CLAUDE-MD-TEMPLATE.md`

---

## [1.5.0] - 17/02/2026

### Summary

Feature release adding coding principles to the init workflow and all project templates. New projects initialised with `/init` now include `CODING-PRINCIPLES.md`, providing Rob Pike's 5 Rules of Programming and Linus Torvalds' 6 coding rules as a reference for all developers.

### Files Changed

| File | Changes |
|------|---------|
| `examples/setup/CODING-PRINCIPLES.md` | New file: full reference document with Rob Pike's 5 rules and Linus Torvalds' 6 coding rules |
| `commands/init.md` | Added step to copy `CODING-PRINCIPLES.md` during init; updated folder structure and output summary |
| `examples/setup/CLAUDE-MD-TEMPLATE.md` | Added Coding Principles section to base CLAUDE.md template |
| `templates/tall-project.md` | Added Coding Principles section to embedded CLAUDE.md content |
| `templates/django-project.md` | Added Coding Principles section to embedded CLAUDE.md content |
| `templates/react-project.md` | Added Coding Principles section to embedded CLAUDE.md content |
| `templates/mobile-project.md` | Added Coding Principles section to embedded CLAUDE.md content |
| `templates/shared-lib-project.md` | Added Coding Principles section to embedded CLAUDE.md content |
| `VERSION` | Bumped from 1.4.0 to 1.5.0 |
| `CHANGELOG.md` | Added 1.5.0 release notes |
| `VERSION-HISTORY.md` | Added 1.5.0 technical details |
| `RELEASES.md` | Added 1.5.0 user-facing notes |

### New Features

#### Coding Principles Reference Document

A new `examples/setup/CODING-PRINCIPLES.md` document has been added containing:

- **Rob Pike's 5 Rules of Programming** - Foundational rules on simplicity, performance optimisation, and data structures
- **Linus Torvalds' Coding Rules** - Practical style, naming, function complexity, and commenting guidelines

#### Project Template Integration

All five project templates now include a Coding Principles section in their embedded `CLAUDE.md` content. This ensures that every new project scaffolded with Syntek Dev Suite carries the coding principles forward automatically.

#### Init Command Update

The `/init` command now copies `CODING-PRINCIPLES.md` to new projects during initialisation, placing it alongside the other setup examples for immediate developer reference.

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
