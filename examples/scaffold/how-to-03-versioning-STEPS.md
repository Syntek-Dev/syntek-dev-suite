# Versioning — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] All changes to be included in this version are committed
- [ ] You know the type of change (breaking / feature / fix)
- [ ] No blocking items in `/GAPS.md`

---

## Semantic Versioning Rules

| Change Type | Version Bump | When to Use |
|-------------|-------------|-------------|
| Breaking change | `major` | API contract broken, migration required for users |
| New feature | `minor` | Backward-compatible new functionality |
| Bug fix or patch | `patch` | Backward-compatible bug fix, documentation, minor tweak |

When in doubt, use `minor` for features and `patch` for fixes.

---

## Steps

### Step 1 — Determine Bump Type

Review the staged or recently committed changes and determine the appropriate version bump:

- **Breaking change** (`major`): Removed or renamed API, changed function signatures, DB migration that breaks existing data
- **New feature** (`minor`): New endpoint, new component, new command, new setting
- **Bug fix** (`patch`): Fixed incorrect behaviour, updated documentation, corrected typo

### Step 2 — Bump Version

```
/syntek-dev-suite:version bump [major|minor|patch]
```

The version agent updates:
- `VERSION` file (semver string)
- `CHANGELOG.md` (new unreleased section)
- `RELEASES.md`
- `VERSION-HISTORY.md`
- Metadata headers in all `.md` files

### Step 3 — Review CHANGELOG Entry

Open `CHANGELOG.md` and review the generated entry. Ensure it:
- Uses imperative mood ("Add", "Fix", "Remove")
- Is concise but descriptive
- References any relevant story or ticket numbers
- Is categorised correctly (Added / Changed / Fixed / Deprecated / Removed / Security)

Edit if needed — the agent's description is a starting point, not final copy.

### Step 4 — Commit Version Files

```
/syntek-dev-suite:git
```

The commit message should follow the format: `chore: bump version to X.Y.Z`

### Step 5 — Tag the Release (for releases to staging or production)

If this version is being released:

```bash
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push origin vX.Y.Z
```

---

## Error Handling

If the version agent cannot determine the current version:
1. Check that `VERSION` file exists at the project root
2. If missing, run `/syntek-dev-suite:version init` to initialise version files

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
