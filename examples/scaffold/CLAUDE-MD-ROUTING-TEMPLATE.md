# {PROJECT_NAME}

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London
**Stack**: {STACK}

---

## Layer Routing

Read the appropriate domain context before starting any task.

| Task Type | Domain | Read First |
|-----------|--------|-----------|
| Writing code, TDD, security, API design, DB migrations | `code/` | `code/CONTEXT.md` |
| Project setup, git workflow, versioning, onboarding | `how-to/` | `how-to/CONTEXT.md` |
| Stories, sprints, ADRs, bug reports, code reviews, audits | `project-management/` | `project-management/CONTEXT.md` |

**Routing rule:** Read a workflow's `CONTEXT.md` for decision-making without executing it. Only enter `STEPS.md` when the workflow is explicitly triggered.

---

## MCP Servers

| Server | Scope | Purpose |
|--------|-------|---------|
| `code-review-graph` | Local | Semantic code navigation, impact radius, change detection |
| `context7` | Global | Up-to-date library and framework documentation |
| `docfork` | Global | Project-specific documentation lookup |
| `claude-in-chrome` | Global | Browser automation, UI testing, design verification |

---

## Model Selection

Use the most capable model appropriate to the task. Do not use a heavier model when a lighter one suffices.

| Task Type | Model | When to Use |
|-----------|-------|-------------|
| Simple edits, syntax fixes, formatting, linting | Haiku | Single-file changes, quick lookups, routine tasks |
| Feature implementation, code review, debugging, documentation | Sonnet | Most day-to-day development tasks |
| Complex architecture, security audits, multi-repo analysis, large context | Opus | High-stakes decisions, deep reasoning, cross-cutting concerns |

---

## GAPS.md Rule

If Claude encounters a workflow folder (any path matching `*/workflows/[0-9][0-9]-*/`) that is missing `STEPS.md` or `CHECKLIST.md`, it must:

1. Append an entry to `/GAPS.md` at the project root with the missing file path and a suggested description
2. Proceed using `CONTEXT.md` alone as best-effort guidance
3. Never generate and silently commit missing files without explicit user instruction

Check `/GAPS.md` at the start of any workflow to see if it has been recently updated with new gaps.

---

{EXISTING_STACK_SECTION}

---

{EXISTING_REFERENCE_DOCS_SECTION}

---

{EXISTING_CONVENTIONS_SECTION}
