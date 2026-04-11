# TDD Cycle — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Scope of work is clearly defined (user story, bug ticket, or ADR)
- [ ] Branch created following `how-to/workflows/02-git-workflow/`
- [ ] Test environment is running and isolated from development DB
- [ ] No blocking items in `/GAPS.md`

---

## Steps

This workflow enforces strict Red → Green → Refactor discipline. Do not write implementation code before writing failing tests.

### Step 1 — Write Failing Tests (Red)

Write tests that define the exact behaviour to be implemented. Tests must fail before any implementation exists.

```
/syntek-dev-suite:test-writer [scope description] --mode failing-first
```

Acceptance criteria for this step:
- All new tests are red
- No implementation code has been written
- Test descriptions are clear and match the acceptance criteria in the user story

### Step 2 — Verify Red State

Run the full test suite and confirm:

- [ ] All new tests are failing (red)
- [ ] No existing tests have been broken
- [ ] Test output clearly shows what is missing

Do **not** proceed to Step 3 until the red state is confirmed.

### Step 3 — Implement (Green)

Write the minimum implementation required to make the tests pass. Do not add features not covered by the tests.

```
/syntek-dev-suite:backend [scope description]
```

Or for frontend work:

```
/syntek-dev-suite:frontend [scope description]
```

### Step 4 — Verify Green State

Run the full test suite and confirm:

- [ ] All new tests are passing (green)
- [ ] No existing tests have been broken
- [ ] No implementation stubs or TODO markers remain

Do **not** proceed to Step 5 until all tests are green.

### Step 5 — Refactor

With tests green, refactor the implementation for clarity, performance, and adherence to coding principles.

```
/syntek-dev-suite:refactor [scope description]
```

After refactoring, re-run tests. They must still be green. If any test turns red, fix the implementation before continuing.

### Step 6 — Coverage Check

```
/syntek-dev-suite:test-writer [scope description] --mode coverage-check
```

Coverage thresholds (from `.claude/TESTING.md`):
- Minimum 75% overall
- Minimum 90% for auth, permissions, tenancy, cryptography modules

If coverage is below threshold, return to Step 1 and add the missing tests.

### Step 7 — Code Review

```
/syntek-dev-suite:review
```

Address all issues before proceeding. Re-run review if significant changes were made.

### Step 8 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If tests turn red after refactoring (Step 5):
1. Revert the refactoring change that caused the regression
2. Re-run tests to confirm green
3. Approach the refactoring more carefully, one change at a time

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
