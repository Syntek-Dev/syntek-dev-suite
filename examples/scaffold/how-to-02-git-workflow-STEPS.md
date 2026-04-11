# Git Workflow — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] You know the story or ticket reference for the work (e.g., `us042`)
- [ ] Development environment is running and tests pass on `dev` branch
- [ ] No blocking items in `/GAPS.md`

---

## Branch Strategy

| Branch | Purpose | Merges Into |
|--------|---------|------------|
| `us###/feature-name` | Feature development | `testing` |
| `fix/description` | Bug fixes | `dev` |
| `hotfix/description` | Production hotfixes | `main` + `dev` |
| `testing` | QA and integration testing | `dev` |
| `dev` | Active development integration | `staging` |
| `staging` | Pre-production validation | `main` |
| `main` | Production | — |

---

## Steps

### Step 1 — Create Feature Branch

```
/syntek-dev-suite:git --action create-branch --story [story-reference]
```

Branch name format: `us###/short-description` (e.g., `us042/user-profile-page`)

Or manually:

```bash
git checkout dev
git pull origin dev
git checkout -b us042/short-description
```

### Step 2 — Work on the Feature

Complete the relevant workflow in `code/workflows/`. Commit frequently using Step 3.

### Step 3 — Commit Changes

```
/syntek-dev-suite:git
```

The git agent:
1. Reviews staged changes
2. Calls `/syntek-dev-suite:version bump [type]` to update version files
3. Creates a commit with a descriptive message
4. Runs pre-commit hooks (linting, formatting)

**Commit message format:** Imperative mood, present tense — "Add user profile page" not "Added user profile page"

### Step 4 — Push Branch

```bash
git push -u origin us042/short-description
```

### Step 5 — Create Pull Request

```
/syntek-dev-suite:git --action create-pr
```

PR checklist before opening:
- [ ] All tests are green
- [ ] `/syntek-dev-suite:review` has been run
- [ ] No merge conflicts with `dev`
- [ ] PR description explains the change and links to the story

### Step 6 — Address Review Feedback

If changes are requested:
1. Make changes on the feature branch
2. Run `/syntek-dev-suite:review` again
3. Commit and push
4. Re-request review

---

## Error Handling

If pre-commit hooks fail:
1. Read the hook output carefully — do not bypass with `--no-verify`
2. Fix the underlying issue (linting error, formatting, test failure)
3. Stage the fixed files and retry the commit

If there are merge conflicts:
1. Resolve conflicts locally — do not use "accept all theirs" blindly
2. Run the full test suite after resolving
3. Commit the resolution

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
