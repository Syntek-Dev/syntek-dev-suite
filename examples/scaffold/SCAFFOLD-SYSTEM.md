# Scaffold System Reference

**Last Updated**: 11/04/2026
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Three-Layer Architecture](#three-layer-architecture)
- [Template Tokens](#template-tokens)
- [File Inventory](#file-inventory)
- [GAPS.md Protocol](#gapsmd-protocol)
- [Agent-to-Workflow Mapping](#agent-to-workflow-mapping)
- [Rules](#rules)

---

## Overview

The scaffold system generates a standardised three-layer project structure on top of an existing syntek-dev-suite initialised project. It does not touch source code — it creates the organisational layer that routes Claude to the right context and workflow for any given task.

The scaffold agent reads every template in this folder before generating a single file. It must not improvise content from memory. Templates provide the authoritative structure; the agent substitutes tokens and injects extracted sections.

---

## Three-Layer Architecture

### Layer 1 — `.claude/CLAUDE.md`

The single routing and orchestration file. Updated (not replaced without backup) by the scaffold. Contains:

- **Layer Routing table** — which domain folder to consult for which type of task
- **MCP Servers table** — available MCP servers and their scope
- **Model Selection table** — Haiku / Sonnet / Opus by task type (no version numbers)
- **GAPS.md rule** — how to handle missing workflow files
- **Routing rule** — when to read CONTEXT.md vs enter STEPS.md
- Preserved sections from the existing CLAUDE.md (Stack, Required Reference Documents, Plugin Tools, Code Conventions)

### Layer 2 — `CONTEXT.md` in Every Folder

Every significant folder contains a CONTEXT.md explaining:

- Purpose of the folder
- What files live inside it and why
- Constraints or conventions specific to this layer
- Which workflows are relevant to this folder's concerns
- What this folder is NOT for (cross-references to other domains)

### Layer 3 — Structured Domain Folders

Three top-level domain folders, each with identical internal structure:

```
{domain}/
├── CONTEXT.md               ← Layer 2: domain routing
├── docs/
│   └── CONTEXT.md           ← reference material for this domain
├── src/  (or scripts/)
│   └── CONTEXT.md           ← artefacts produced in this domain
└── workflows/
    ├── CONTEXT.md            ← routes to numbered workflows
    └── NN-workflow-name/
        ├── CONTEXT.md        ← what, when, what it produces, dependencies
        ├── STEPS.md          ← ordered steps Claude must follow (no improvisation)
        └── CHECKLIST.md      ← completion criteria and definition of done
```

**Domain inventory:**

| Domain | Purpose | Workflows |
|--------|---------|-----------|
| `code/` | Writing code, TDD, security, API design, DB migrations | 5 |
| `how-to/` | Setup, git workflow, versioning | 3 |
| `project-management/` | ADRs, sprints, stories, bug reports, reviews, audits | 6 |

---

## Template Tokens

All templates use exactly three substitution tokens:

| Token | Value | Source |
|-------|-------|--------|
| `{PROJECT_NAME}` | Human-readable project name | Extracted from `.claude/CLAUDE.md` |
| `{STACK}` | Stack identifier | Detected stack (TALL / Django / React / Mobile / Shared Lib) |
| `{DATE}` | Today's date | Format: DD/MM/YYYY |

Replace all occurrences of each token before writing the file. Do not leave unreplaced tokens in generated files.

---

## File Inventory

### Core Templates (always read first)

| File | Purpose | Used For |
|------|---------|---------|
| `SCAFFOLD-SYSTEM.md` | This file — agent orientation | Agent reads for context only |
| `CLAUDE-MD-ROUTING-TEMPLATE.md` | Routing overlay for `.claude/CLAUDE.md` | Generating new `.claude/CLAUDE.md` |
| `DOMAIN-CONTEXT-TEMPLATE.md` | Domain-level CONTEXT.md structure | `code/`, `how-to/`, `project-management/` |
| `LAYER-CONTEXT-TEMPLATE.md` | Sub-folder CONTEXT.md structure | `docs/`, `src/`, `scripts/`, `workflows/` |
| `WORKFLOW-CONTEXT-TEMPLATE.md` | Numbered workflow CONTEXT.md | Each `NN-workflow-name/CONTEXT.md` |
| `WORKFLOW-STEPS-TEMPLATE.md` | Generic fallback STEPS.md | When no specific steps file exists |
| `WORKFLOW-CHECKLIST-TEMPLATE.md` | Generic checklist structure | All `NN-workflow-name/CHECKLIST.md` |
| `GAPS-TEMPLATE.md` | GAPS.md starting structure | Project root `GAPS.md` |

### Per-Workflow STEPS Templates

These are the primary content files. The agent reads the matching file for each workflow and writes it verbatim (after token substitution) to the target workflow folder.

| File | Writes To |
|------|----------|
| `code-01-new-feature-STEPS.md` | `code/workflows/01-new-feature/STEPS.md` |
| `code-02-tdd-cycle-STEPS.md` | `code/workflows/02-tdd-cycle/STEPS.md` |
| `code-03-security-hardening-STEPS.md` | `code/workflows/03-security-hardening/STEPS.md` |
| `code-04-api-design-STEPS.md` | `code/workflows/04-api-design/STEPS.md` |
| `code-05-database-migration-STEPS.md` | `code/workflows/05-database-migration/STEPS.md` |
| `how-to-01-setup-STEPS.md` | `how-to/workflows/01-setup/STEPS.md` |
| `how-to-02-git-workflow-STEPS.md` | `how-to/workflows/02-git-workflow/STEPS.md` |
| `how-to-03-versioning-STEPS.md` | `how-to/workflows/03-versioning/STEPS.md` |
| `pm-01-architecture-decision-STEPS.md` | `project-management/workflows/01-architecture-decision/STEPS.md` |
| `pm-02-sprint-planning-STEPS.md` | `project-management/workflows/02-sprint-planning/STEPS.md` |
| `pm-03-story-writing-STEPS.md` | `project-management/workflows/03-story-writing/STEPS.md` |
| `pm-04-bug-report-STEPS.md` | `project-management/workflows/04-bug-report/STEPS.md` |
| `pm-05-code-review-STEPS.md` | `project-management/workflows/05-code-review/STEPS.md` |
| `pm-06-security-audit-STEPS.md` | `project-management/workflows/06-security-audit/STEPS.md` |

---

## GAPS.md Protocol

**Location:** Project root (`/GAPS.md` — visible alongside README.md and CHANGELOG.md)

**When to log a gap:**
- A workflow folder exists but is missing `STEPS.md`
- A workflow folder exists but is missing `CHECKLIST.md`
- The generic `WORKFLOW-STEPS-TEMPLATE.md` fallback was used (signals the file needs customisation)

**Format:**
```markdown
| code/workflows/01-new-feature/STEPS.md | STEPS.md | Step-by-step instructions for implementing a new feature using the plan → build → test → review cycle |
```

**Rules:**
1. Never silently generate missing files — always log to GAPS.md first
2. Before appending, check if the path already exists in GAPS.md (no duplicates)
3. If a gap has been resolved (file now exists), remove the resolved row
4. Proceed using `CONTEXT.md` alone when `STEPS.md` is missing

---

## Agent-to-Workflow Mapping

Workflow STEPS.md files reference syntek-dev-suite agents by their command name. This is the bridge between the workflow system and the agent system — improving an agent automatically improves any workflow that invokes it.

| Agent Command | Used In Workflows |
|--------------|------------------|
| `/syntek-dev-suite:plan` | new-feature, architecture-decision, api-design |
| `/syntek-dev-suite:stories` | new-feature, story-writing |
| `/syntek-dev-suite:sprint` | sprint-planning |
| `/syntek-dev-suite:backend` | new-feature, api-design |
| `/syntek-dev-suite:frontend` | new-feature |
| `/syntek-dev-suite:test-writer` | new-feature, tdd-cycle, api-design, database-migration |
| `/syntek-dev-suite:review` | new-feature, tdd-cycle, api-design, code-review |
| `/syntek-dev-suite:security` | security-hardening, security-audit |
| `/syntek-dev-suite:qa-tester` | security-hardening, bug-report, security-audit |
| `/syntek-dev-suite:debugger` | bug-report |
| `/syntek-dev-suite:database` | database-migration |
| `/syntek-dev-suite:docs` | api-design |
| `/syntek-dev-suite:git` | All workflows (final step) |
| `/syntek-dev-suite:version` | versioning, release |
| `/syntek-dev-suite:setup` | setup |
| `/syntek-dev-suite:completion` | sprint-planning |

---

## Rules

### Routing Rule
`.claude/CLAUDE.md` may load a workflow's `CONTEXT.md` for decision-making without executing it. Claude only enters `STEPS.md` when the workflow is **explicitly triggered** by the user or another workflow.

### Gap Handling Rule
If Claude encounters a workflow folder that contains only `CONTEXT.md` and is missing `STEPS.md` or `CHECKLIST.md`, it must:
1. Log the gap in `/GAPS.md` with the missing file path and a suggested description
2. Proceed using `CONTEXT.md` alone as best-effort guidance
3. Never generate and silently commit missing files without explicit user instruction

### Template Rule
The scaffold agent must read all templates from `$SYNTEK_DIR/examples/scaffold/` before generating any output. It must not improvise file content from memory or training data. Templates are the single source of truth for file structure.
