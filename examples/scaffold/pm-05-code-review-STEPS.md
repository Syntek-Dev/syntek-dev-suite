# Code Review — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Pull request or diff is ready for review
- [ ] All tests are passing on the branch being reviewed
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Run Automated Review

```
/syntek-dev-suite:review
```

The code review agent reads `.claude/CODING-PRINCIPLES.md`, `.claude/SECURITY.md`, and `.claude/TESTING.md`. It checks for:

**Security:**
- Authentication and authorisation on all endpoints
- Input validation and sanitisation
- Secrets not committed or logged
- RLS applied where required
- OWASP Top 10 violations

**Code Quality:**
- SOLID and CUPID principles
- DRY (no unnecessary duplication)
- YAGNI (no speculative abstractions)
- Appropriate error handling
- Clear naming conventions
- No unused code

**Testing:**
- Adequate test coverage
- Tests are meaningful (not just structural)
- Edge cases are covered
- No test code in production paths

**Documentation:**
- Public APIs are documented
- Complex logic has explanatory comments
- README updated if behaviour changed

### Step 2 — Review Agent Findings

Go through each finding from the review agent:

- **Critical**: Must be fixed before merge — security vulnerabilities, data loss risk, broken tests
- **Major**: Should be fixed before merge — significant quality issues, missing tests
- **Minor**: Can be addressed in a follow-up — style preferences, minor improvements

### Step 3 — Address Critical and Major Findings

For each Critical and Major finding:
1. Make the necessary change
2. Run affected tests
3. Re-run `/syntek-dev-suite:review` to confirm the finding is resolved

Do not merge until all Critical and Major findings are resolved.

### Step 4 — Document Minor Findings (if not addressing now)

For Minor findings that will not be addressed in this PR, create follow-up tasks in `project-management/src/STORIES/` or your project management tool.

### Step 5 — Human Review Sign-off

Confirm that at least one team member (not the author) has reviewed the PR and approved it before merging.

### Step 6 — Merge and Commit Record

After merging, run:

```
/syntek-dev-suite:git
```

Record the review completion in `project-management/src/REVIEWS/` if your team maintains review history.

---

## Error Handling

If the review agent finds a large number of issues:
1. Do not attempt to fix everything in one pass — prioritise Critical first
2. Create a refactoring story for systemic quality issues
3. Consider whether the PR scope is too large and should be split

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
