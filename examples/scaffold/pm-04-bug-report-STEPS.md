# Bug Report — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Bug can be reproduced reliably (at least 2 out of 3 attempts)
- [ ] Environment where the bug occurs is known (dev / staging / production)
- [ ] Branch created following `how-to/workflows/02-git-workflow/` (use `fix/` prefix)
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Hostile QA Reproduction

Use the QA tester to reproduce and characterise the bug.

```
/syntek-dev-suite:qa-tester [bug description — what happens vs what should happen]
```

The QA tester will:
- Attempt to reproduce the bug
- Identify the conditions that trigger it
- Check if related functionality is also affected
- Determine severity (critical / high / medium / low)

### Step 2 — Deep Debug

```
/syntek-dev-suite:debug [bug description and reproduction steps]
```

The debugger agent will:
- Trace the execution path
- Identify the root cause (not just the symptom)
- Propose a minimal fix
- Check for similar patterns elsewhere that may have the same bug

### Step 3 — Create Bug Record

Create a bug record in `project-management/src/BUGS/`:

```
project-management/src/BUGS/BUG-{NNN}-{short-description}.md
```

Include:
- **Summary**: One-line description of the bug
- **Environment**: Where the bug occurs
- **Steps to reproduce**: Numbered, specific steps
- **Expected behaviour**: What should happen
- **Actual behaviour**: What actually happens
- **Root cause**: From the debug analysis
- **Severity**: Critical / High / Medium / Low
- **Fix approach**: Brief description of the fix

### Step 4 — Implement Fix

Make the minimal fix identified in Step 2. Do not refactor or add features in the same commit.

```
/syntek-dev-suite:backend [fix description]
```

or

```
/syntek-dev-suite:frontend [fix description]
```

### Step 5 — Write Regression Test

Write a test that would have caught this bug.

```
/syntek-dev-suite:test-writer [bug description] --mode regression
```

The regression test must fail before the fix is applied and pass after. This prevents the bug from recurring.

### Step 6 — Verify All Tests Pass

Run the full test suite. All tests must be green, including the new regression test.

### Step 7 — Code Review

```
/syntek-dev-suite:review
```

### Step 8 — Commit

```
/syntek-dev-suite:git
```

Commit message format: `fix: [short description of what was fixed]`

---

## Error Handling

If the root cause cannot be identified after 30 minutes of debugging:
1. Escalate to a more experienced team member
2. Increase logging around the suspect area temporarily
3. Check recent commits — `git log --oneline -20` — for anything that may have introduced the bug

If the fix is more complex than expected:
1. Create a user story for the proper fix
2. Apply a temporary workaround if severity is high
3. Document the workaround clearly in the bug record

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
