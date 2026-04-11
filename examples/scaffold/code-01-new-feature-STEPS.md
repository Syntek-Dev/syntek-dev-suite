# New Feature — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] User story exists in `project-management/src/STORIES/`
- [ ] Story has been assigned to a sprint in `project-management/src/SPRINTS/`
- [ ] Branch created following `how-to/workflows/02-git-workflow/`
- [ ] No blocking items in `/GAPS.md`

---

## Steps

Follow these steps in order. Do not skip or reorder without explicit reason.

### Step 1 — Architectural Plan

Generate a plan document before writing any code.

```
/syntek-dev-suite:plan [feature name or user story reference]
```

Review the generated plan. Confirm scope, DB schema changes, and API surface before proceeding. Save the plan to `project-management/src/PLANS/PLAN-{FEATURE}.md`.

### Step 2 — Generate User Stories (if not already done)

If user stories have not been created for this feature:

```
/syntek-dev-suite:stories [feature description]
```

Save stories to `project-management/src/STORIES/`.

### Step 3 — Sprint Assignment (if not already done)

If stories are not yet assigned to a sprint:

```
/syntek-dev-suite:sprint
```

### Step 4 — Write Failing Tests First

Before implementing, write the failing tests that define the acceptance criteria.

```
/syntek-dev-suite:test-writer [feature description] --mode failing-first
```

Verify all new tests are red before proceeding to Step 5.

### Step 5 — Implement Backend

```
/syntek-dev-suite:backend [feature description]
```

The backend agent reads `.claude/ARCHITECTURE-PATTERNS.md`, `.claude/API-DESIGN.md`, and `.claude/SECURITY.md` before writing any code. Do not proceed until the agent confirms completion.

### Step 6 — Implement Frontend (if applicable)

```
/syntek-dev-suite:frontend [feature description]
```

The frontend agent reads `.claude/ACCESSIBILITY.md` and `.claude/CODING-PRINCIPLES.md`. Skip this step if the feature is API-only.

### Step 7 — Verify Tests Are Green

Run the project test suite. All tests written in Step 4 must be green before proceeding.

```bash
# Run tests using the project's test command (see how-to/workflows/02-git-workflow/)
```

If any test is red, return to Step 5 or Step 6 and fix the implementation. Do not proceed with failing tests.

### Step 8 — Code Review

```
/syntek-dev-suite:review
```

Address all issues flagged by the review agent before proceeding. Re-run review if significant changes were made.

### Step 9 — Documentation

If the feature introduces a new API endpoint, background job, or significant module:

```
/syntek-dev-suite:docs [scope of documentation]
```

### Step 10 — Commit

```
/syntek-dev-suite:git
```

The git agent calls `/syntek-dev-suite:version bump minor` (or `patch` for small additions) before committing.

---

## Error Handling

If a step fails:

1. Do not proceed to the next step
2. Diagnose the root cause — run `/syntek-dev-suite:debug` if needed
3. Fix the issue, then re-run the failed step from the beginning
4. If blocked for more than 30 minutes, log a bug in `project-management/src/BUGS/` using the bug-report workflow

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
