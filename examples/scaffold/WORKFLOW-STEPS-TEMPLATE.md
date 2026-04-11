# {WORKFLOW_NAME} — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

> **Note:** This file was generated from the generic fallback template. The steps below are placeholders and must be customised for your project. This workflow has been logged in `/GAPS.md` as needing specific steps.

---

## Prerequisites

Before starting, confirm:

- [ ] All dependencies listed in `CONTEXT.md` are met
- [ ] You are on the correct branch (see `how-to/workflows/02-git-workflow/`)
- [ ] No blocking items in `/GAPS.md` for this workflow

---

## Steps

Follow these steps in order. Do not skip or reorder steps without explicit instruction.

### Step 1 — [Customise: First Action]

> Replace this placeholder with the specific first action for this workflow.

```bash
# Add any shell commands here
```

### Step 2 — [Customise: Main Action]

> Replace this placeholder with the primary work for this workflow.

Invoke the relevant agent:

```
/syntek-dev-suite:[agent-name] [arguments]
```

### Step 3 — [Customise: Verification]

> Replace this placeholder with how to verify the work is correct.

### Step 4 — Commit

Run the git workflow to commit changes:

```
/syntek-dev-suite:git
```

---

## Error Handling

If a step fails:

1. Do not proceed to the next step
2. Diagnose the root cause before retrying
3. If blocked, log the issue in `project-management/src/BUGS/`

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
