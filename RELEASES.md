# Release Notes

**Last Updated**: 11/04/2026
**Version**: 1.11.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Latest Release](#latest-release)
  - [Version 1.11.0 - 11 April 2026](#version-1110---11-april-2026)
- [Previous Releases](#previous-releases)
  - [Version 1.10.0 - 05 April 2026](#version-1100---05-april-2026)
  - [Version 1.9.0 - 15 March 2026](#version-190---15-march-2026)
  - [Version 1.8.0 - 15 March 2026](#version-180---15-march-2026)
  - [Version 1.7.0 - 13 March 2026](#version-170---13-march-2026)
  - [Version 1.6.0 - 24 February 2026](#version-160---24-february-2026)
  - [Version 1.5.0 - 17 February 2026](#version-150---17-february-2026)
  - [Version 1.4.0 - 09 January 2026](#version-140---09-january-2026)
  - [Version 1.3.1 - 29 December 2025](#version-131---29-december-2025)
  - [Version 1.3.0 - 28 December 2025](#version-130---28-december-2025)
  - [Version 1.2.0 - 24 December 2025](#version-120---24-december-2025)
  - [Version 1.1.0 - 24 December 2025](#version-110---24-december-2025)
  - [Version 1.0.0 - 21 December 2025](#version-100---21-december-2025)
- [Need Help?](#need-help)


---

## Latest Release

### Version 1.11.0 - 11 April 2026

#### What's New

**Multi-Layer Project Scaffolding System**

Version 1.11.0 introduces `/syntek-dev-suite:scaffold` — a new command that generates a standardised three-layer workflow structure for any syntek-dev-suite project, whether brand new or already established.

The scaffolding creates:
- **Layer 1** — An updated `.claude/CLAUDE.md` with routing logic, MCP server registration, model selection rules (Haiku/Sonnet/Opus by task type), and the GAPS.md protocol
- **Layer 2** — `CONTEXT.md` in every significant folder, explaining purpose, contents, and constraints
- **Layer 3** — Three top-level domains (`code/`, `how-to/`, `project-management/`) each with `docs/`, `src/`, and `workflows/` sub-folders containing 14 numbered workflow directories

Each workflow folder contains `CONTEXT.md` (what, when, what it produces), `STEPS.md` (ordered steps referencing syntek-dev-suite agents), and `CHECKLIST.md` (completion criteria and definition of done).

**GAPS.md Protocol**

Any workflow folder missing `STEPS.md` or `CHECKLIST.md` is logged in `/GAPS.md` at the project root. Claude never silently generates missing files — it logs the gap and proceeds using `CONTEXT.md` alone.

**Integrated with `/init`**

The `/init` command now offers to run `/syntek-dev-suite:scaffold new` as Step 10, immediately after project initialisation.

#### Upgrade Notes

No breaking changes. All existing agents, commands, skills, and plugins are unchanged. The scaffold is purely additive.

---

## Previous Releases

### Version 1.10.0 - 05 April 2026

#### What's New

**Row Level Security Documentation and Requirements**

Version 1.10.0 adds comprehensive Row Level Security (RLS) documentation for PostgreSQL, along with updated requirements across the database, backend, data scientist, and security agents. This ensures that every agent working on a multi-tenant or data-sensitive project understands RLS policies and applies them consistently.

**New RLS Guide**

A new guide (`examples/database/rls/RLS.md`) covers everything you need to implement RLS correctly:

- How to enable and force RLS on tables (`ENABLE ROW LEVEL SECURITY`, `FORCE ROW LEVEL SECURITY`)
- Designing per-table policies for SELECT, INSERT, UPDATE, and DELETE
- Role separation — keeping application roles and superuser roles distinct
- Multi-tenant isolation patterns using `current_setting()` and `app.current_tenant_id`
- Understanding `BYPASSRLS` and why application roles must never hold it
- Auditing and testing your policies to confirm they enforce isolation

**Updated Agent Requirements**

Three agents have been updated with explicit RLS requirements:

- **Database agent** — must verify RLS is enabled on all multi-tenant tables and that policies exist before approving schema changes
- **Backend agent** — must confirm application roles do not carry `BYPASSRLS` and that service accounts hold minimum required privileges
- **Data scientist agent** — must confirm queries respect RLS policies and that analysis scripts do not connect as a superuser

**Updated Security Reference**

`examples/setup/SECURITY.md` now includes a dedicated Row Level Security section covering multi-tenant isolation patterns, policy auditing, common pitfalls, and a testing approach for verifying enforcement.

---

## Previous Releases

### Version 1.9.0 - 15 March 2026

#### Highlights

- All 29 agents now explicitly load their relevant reference documents before working
- `/init` copies all ten reference documents to `.claude/` (previously only five were copied)
- Every agent's startup sequence includes a dedicated step to load domain-relevant guides (Coding Principles, Security, Accessibility, API Design, Architecture Patterns, Data Structures, Performance, Testing, Development, SEO Checklist)

---

### Version 1.8.0 - 15 March 2026

#### What's New

**Five New Reference Guides for Every Agent and Developer**

Version 1.8.0 adds five new technical reference documents to Syntek Dev Suite. These join the existing set of guides that agents read before working on your project, giving them deeper knowledge of accessibility standards, API design conventions, architecture patterns, data structure choices, and performance optimisation.

Every new project initialised with `/init` will include these guides in its `.claude/` folder.

#### The Five New Guides

**Accessibility Guide**

A complete WCAG 2.2 reference that covers what you need to do — and why — to make your application accessible. Topics include:

- ARIA roles and how to use them correctly
- Keyboard navigation and focus management
- Colour contrast requirements with specific ratios
- Screen reader support for dynamic content
- Stack-specific patterns for Blade, Django templates, React, and React Native

**API Design Guide**

Clear, consistent rules for designing APIs that are predictable and easy to consume. Topics include:

- RESTful URL structure and naming conventions
- HTTP methods, status codes, and idempotency rules
- Standardised error response format
- Authentication header patterns
- Pagination and rate limiting conventions
- GraphQL naming and error handling

**Architecture Patterns Guide**

Practical guidance on structuring larger applications. Topics include:

- Layered architecture with clear dependency rules
- Domain-Driven Design (DDD) — bounded contexts, aggregates, value objects
- CQRS — separating reads and writes for complex domains
- Event-driven patterns and domain events
- Repository and service layer design
- Hexagonal architecture (ports and adapters)

**Data Structures Guide**

When to use which data structure, and how to handle data consistently at boundaries. Topics include:

- Choosing between arrays, maps, sets, and queues
- DTO patterns per stack (PHP typed classes, Python dataclasses, TypeScript interfaces)
- Value objects and immutable data
- JSON serialisation conventions and camelCase vs snake_case boundaries
- Collection handling — filtering, sorting, pagination

**Performance Guide**

Practical performance techniques with specific targets to hit. Topics include:

- Caching strategies — in-memory, HTTP headers, query caching
- Preventing N+1 queries with eager loading (`with()`, `select_related`, Prisma `include`)
- Database indexing — what to index and what not to
- Core Web Vitals targets: LCP < 2.5s, CLS < 0.1, INP < 200ms
- Asset optimisation — WebP/AVIF images, code splitting, tree shaking
- Stack-specific tips — Laravel Octane, Django caching, Next.js ISR, React Native FlashList

#### Why This Matters

Previously, agents relied on general knowledge to make decisions about accessibility, APIs, architecture, data handling, and performance. Now they have project-specific references that reflect your team's agreed conventions — the same ones every developer on the project works to.

This means:

- **Consistent APIs** — Every endpoint follows the same structure and error format
- **Accessible interfaces** — WCAG 2.2 compliance built in from the start
- **Maintainable architecture** — Agreed patterns for layers, domains, and service boundaries
- **Fewer performance surprises** — N+1 queries caught early, caching applied consistently
- **Shared data conventions** — DTOs and type patterns that work across the team

#### Expanded Existing Guides

The four existing reference guides have also been substantially expanded:

- **Coding Principles** — Additional sections with practical examples
- **Development Workflow** — Database configuration conventions and environment-specific settings structure
- **Security** — Additional OWASP patterns and per-stack security guidance
- **Testing** — Additional test patterns and tooling coverage

---

## Previous Releases

### Version 1.7.0 - 13 March 2026

#### What's New

**Your SEO Agent Now Knows About AI Search**

Version 1.7.0 adds a comprehensive SEO & AI Discoverability Checklist to every project initialised with Syntek Dev Suite. The checklist covers everything from setting up `robots.txt` and meta tags through to Generative Engine Optimisation (GEO) — making your site visible to both traditional search engines and AI-powered tools like ChatGPT, Claude, and Perplexity.

#### The New SEO Checklist

Every project now includes `SEO-CHECKLIST.md` in its `.claude/` folder. The checklist is structured in three tiers:

**Beginner** — The essentials every site needs:
- `robots.txt` with AI crawler permissions (GPTBot, ClaudeBot, PerplexityBot)
- XML sitemaps, canonical URLs, and meta descriptions
- Open Graph and Twitter Card tags for social sharing
- `llms.txt` — a new file that tells AI agents what your site is about

**Intermediate** — Building on the basics:
- JSON-LD structured data (Organisation, Article, Breadcrumbs)
- Core Web Vitals performance targets
- FAQ sections and question-format headings for AI readability
- "Last updated" timestamps so AI tools know your content is current

**Advanced** — For maximum visibility in AI search:
- Generative Engine Optimisation (GEO) strategies
- Direct answer boxes and entity disambiguation
- Structured citation-ready content
- Monitoring for AI overview appearances

#### AI Discoverability (What Is llms.txt?)

As AI tools like ChatGPT and Perplexity increasingly answer questions directly from web content, standard SEO is no longer enough. The `llms.txt` file — placed at your site root — is a plain Markdown summary that tells AI agents what your site does, who it is for, and which pages are most useful. Think of it like `robots.txt`, but for AI.

The SEO agent will now help you create both `llms.txt` and `llms-full.txt` as part of its standard implementation.

#### The SEO Agent Now Audits First

Before implementing anything, the SEO agent reads your project's `SEO-CHECKLIST.md` and audits which items are already in place. You will get a clear picture of what is done, what is missing, and what to prioritise — instead of starting from scratch every time.

---

## Previous Releases

### Version 1.6.0 - 24 February 2026

#### What's New

**Every New Project Now Comes With a Full Playbook**

When you initialise a new project with `/init`, you now get four ready-to-use reference guides in your `.claude/` folder — not just one. Every agent working on your project will read these before writing a single line of code.

The four guides are:

- **Coding Principles** — Proven rules from Rob Pike and Linus Torvalds, plus practical conventions for naming, error handling, logging, and code review
- **Testing Guide** — How to test your specific stack, which tools to use, TDD workflow, and what test coverage is expected
- **Security Guide** — OWASP-aligned security patterns, how to handle secrets, authentication best practices, and a pre-deployment security checklist
- **Development Workflow** — How to get a project running locally, container commands, branching workflow, and everyday development tasks

These guides work across all five supported stacks: TALL (Laravel), Django, React/Next.js, React Native, and Shared Libraries.

#### Why This Matters

Previously, agents had general knowledge of best practices but no project-specific reference to check against. Now every project ships with a shared baseline that agents and developers alike can refer to.

This means:

- **Consistent security** — Security patterns are documented and enforced from day one
- **Consistent testing** — Everyone knows which tools to use and what coverage is expected
- **Faster onboarding** — New developers (and new agents) have a single place to get up to speed
- **Fewer surprises** — Common decisions are already made and documented

#### Expanded Coding Principles

The Coding Principles guide has been significantly expanded. It now covers:

- How to handle errors properly (no silent failures)
- Language-specific naming conventions (PHP, Python, TypeScript)
- Security-minded coding habits
- When to add dependencies (and when not to)
- Git commit and branch conventions
- DRY, KISS, and YAGNI in practice
- What to log and what not to log
- A pre-PR code review checklist

---

## Previous Releases

### Version 1.5.0 - 17 February 2026

#### What's New

**Coding Principles Built Into Every Project**

Every new project you create with Syntek Dev Suite now ships with a built-in coding principles reference. When you run `/init`, the `CODING-PRINCIPLES.md` file is automatically included — giving your whole team a shared set of proven guidelines from day one.

The document covers:

- **Rob Pike's 5 Rules of Programming** - Timeless wisdom from one of Unix's creators, covering simplicity, performance thinking, and the importance of good data structures
- **Linus Torvalds' Coding Rules** - Battle-tested guidelines on function size, naming clarity, and why comments should explain *why* not *what*

These principles are also embedded into all project templates, so `CLAUDE.md` files in new projects include a Coding Principles section pointing developers to the reference document.

#### Why This Matters

Having a shared set of coding principles helps teams:

- Write simpler, more maintainable code
- Avoid premature optimisation
- Keep functions focused and readable
- Name things clearly and consistently
- Write comments that add real value

---

## Previous Releases

### Version 1.4.0 - 09 January 2026

#### What's New

**Portable Plugin Tools**

Your projects are now fully self-contained! When you initialise a new project with `/init`, all the Python plugin tools are automatically copied into your project's `.claude/plugins/` folder.

This means:

- **Work anywhere** - Your project works even without the Syntek Dev Suite installed globally
- **Share with your team** - Everyone gets the same plugin versions, no setup required
- **Customise freely** - Modify plugins for project-specific needs without affecting other projects
- **Version stability** - Your project keeps working even if the main plugin updates

#### How It Works

When you run `/init`, the system now:

1. Creates the `.claude/plugins/` directory in your project
2. Copies all 14 Python plugin tools
3. Makes them executable automatically

Your project's `CLAUDE.md` will include full documentation on how to use these plugins.

#### For Existing Projects

Want to add plugin tools to an existing project? Simply run:

```bash
mkdir -p .claude/plugins/
cp /path/to/syntek-dev-suite/plugins/*.py .claude/plugins/
chmod +x .claude/plugins/*.py
```

---

## Previous Releases

### Version 1.3.1 - 29 December 2025

#### What's Fixed

**Works Anywhere Now**

This bug fix release makes the plugin work properly no matter where you install it. All hardcoded paths have been replaced with dynamic detection.

- **Installation flexibility** - Install the plugin anywhere on your system
- **Cross-platform compatibility** - Better support for Linux and macOS
- **Portable configuration** - All file paths are now relative

---

### Version 1.3.0 - 28 December 2025

#### What's New

**Project Management Integration**

Connect your Claude Dev Team to your favourite project management tool. Whether you use ClickUp, Linear, Jira, GitHub Projects, or any of the other 10 supported platforms, you can now:

- Automatically sync your user stories and tasks with your PM tool
- Create and update issues directly from Claude Code
- Track sprint progress across your entire team
- Link commits to specific issues for better traceability
- Customise field mappings to match your workflow

Simply run `/syntek-dev-suite:pm-setup` to get started, and the system will detect your PM tool automatically.

**Improved Navigation**

We've refreshed all the documentation README files throughout the project to make it easier to:
- Find the right agent for your task
- Understand what each plugin does
- Navigate the codebase structure
- Discover examples and templates

#### Coming Soon

In our next release, we're working on:
- Enhanced reporting capabilities for sprint metrics
- Dashboard visualisations for agent performance
- Integration with more communication tools (Slack, Teams, Discord)

---

## Previous Releases

### Version 1.2.0 - 24 December 2025

#### Highlights

**Smart Version Management**

We've introduced a powerful Version Agent that handles all your versioning needs:

- **Automatic version bumping** - Just tell it what changed (feature, fix, or breaking change) and it updates all version files
- **Consistent documentation** - Maintains technical changelogs, developer summaries, and user-facing release notes
- **Metadata tracking** - Every documentation file now includes version info, last updated date, and language settings

Run `/version bump minor` to increment your version and update all documentation in one go.

**Enhanced Markdown Editing**

Documentation just got better with:
- Automatic table of contents generation
- Smart table formatting
- Consistent headers across all files
- British English spell checking
- VS Code extension integration for a smooth editing experience

Your documentation stays professional and consistent without extra effort.

---

### Version 1.1.0 - 24 December 2025

#### Highlights

**Browser Testing Made Easy**

Testing web applications is now seamless with integrated Chrome support:

- **One-click browser launch** - The system auto-detects Chrome on your machine
- **Live debugging** - See console errors and inspect the DOM directly from Claude Code
- **Automated testing** - Works with Playwright, Cypress, Puppeteer, and more
- **Visual verification** - Compare your running app against design mockups

Perfect for catching bugs early and ensuring your app looks great across all browsers.

---

### Version 1.0.0 - 21 December 2025

#### Highlights

**Complete Development Suite**

The first major release brings everything you need for full-stack development:

- **30 Specialised Agents** - From planning to deployment, there's an agent for every task
- **Self-Learning System** - The agents get smarter over time based on your feedback
- **Multi-Stack Support** - Laravel, Django, React, React Native, and more
- **80+ Code Examples** - Real-world examples for common development patterns

**Smart Environment Detection**

The suite automatically detects:
- Your programming language and framework
- Database type and ORM
- Docker and DDEV configurations
- Git repository settings
- Environment variables

No configuration files to write - it just works.

**Team Collaboration**

Built for teams with:
- Shared learning across the entire team
- Consistent British English documentation
- Standardised code patterns
- Section README files for easy onboarding

Get new team members productive on day one.

#### What Users Are Saying

"The self-learning system is brilliant - the agents actually improve the more we use them!"

"Finally, an AI dev tool that understands British English properly. Colour, not color!"

"The Section README files saved us hours during onboarding. New developers know exactly where to find things."

---

## Need Help?

Visit our documentation at `/docs` or run `/help` for a list of all available commands.

For issues and feature requests, please visit our [GitHub repository](https://github.com/Syntek-Studio/syntek-dev-suite).
