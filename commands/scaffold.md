---
description: "[Agent] Scaffold a multi-layer workflow structure for new or existing projects"
usage: /syntek-dev-suite:scaffold [new|migrate]
---

# Scaffold — Multi-Layer Project Structure

This command generates a standardised three-layer workflow structure for your project.

## What It Creates

**Layer 1 — `.claude/CLAUDE.md` (updated)**
- Layer routing table (which domain to consult for which task)
- MCP server registration (code-review-graph, context7, docfork, claude-in-chrome)
- Model selection rules (Haiku / Sonnet / Opus by task type)
- GAPS.md rule and routing rule

**Layer 2 — CONTEXT.md in every folder**
- Every significant folder gets a CONTEXT.md explaining its purpose and contents

**Layer 3 — Three domain folders**
- `code/` — 5 coding workflows (new feature, TDD, security hardening, API design, DB migration)
- `how-to/` — 3 operational workflows (setup, git workflow, versioning)
- `project-management/` — 6 PM workflows (ADR, sprint planning, story writing, bug report, code review, security audit)

**Project root**
- `GAPS.md` — logs any workflow folders missing STEPS.md or CHECKLIST.md

## Modes

| Mode | When to Use |
|------|------------|
| `new` | Fresh project that has just been initialised with `/syntek-dev-suite:init` |
| `migrate` | Existing project — adds missing domains without overwriting what exists |
| *(no argument)* | Auto-detects: uses `migrate` if any domain folder already exists, otherwise `new` |

## Usage

```
/syntek-dev-suite:scaffold
/syntek-dev-suite:scaffold new
/syntek-dev-suite:scaffold migrate
```

## Prerequisites

- Project must be initialised with `/syntek-dev-suite:init` first
- `.claude/CLAUDE.md` must exist (the scaffold agent reads it for project name and stack)

## Notes

- A backup of the existing `.claude/CLAUDE.md` is saved to `.claude/CLAUDE.md.pre-scaffold` before any changes
- The scaffold does not touch your source code
- Run `/syntek-dev-suite:git` after scaffolding to commit the new structure

$ARGUMENTS
