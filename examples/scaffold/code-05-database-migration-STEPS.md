# Database Migration — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Schema change is planned and agreed (ADR or plan document exists)
- [ ] `.claude/DATA-STRUCTURES.md` has been read
- [ ] Backup of the development database exists (or DB is ephemeral/seeded)
- [ ] Branch created following `how-to/workflows/02-git-workflow/`
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Generate Migration

```
/syntek-dev-suite:database [migration description — table name, columns, changes]
```

The database agent reads `.claude/DATA-STRUCTURES.md`, `.claude/SECURITY.md`, and `.claude/ARCHITECTURE-PATTERNS.md`. It will:
- Generate the migration file following the project's ORM conventions
- Apply Row Level Security (RLS) policies for any new user-scoped or tenant-scoped tables
- Add appropriate indexes based on expected query patterns
- Use reversible migrations wherever possible

**Critical requirements the agent enforces:**
- RLS must be applied in the same migration as the table creation (`0001_initial.py` pattern)
- Sensitive fields (email, phone, PII) must use field-level encryption per `.claude/SECURITY.md`
- Every new table with user or tenant scope must have a RLS policy — no exceptions

### Step 2 — Review Migration File

Before applying, manually review the generated migration:

- [ ] Migration is reversible (has both up and down operations, or is additive-only)
- [ ] RLS policies are correct for the intended access pattern
- [ ] Indexes are appropriate — not over-indexed, not under-indexed
- [ ] No data loss risk for existing rows
- [ ] Column types match the domain model in `.claude/DATA-STRUCTURES.md`

### Step 3 — Apply to Development Database

Apply the migration to the development database and verify it runs without errors.

```bash
# Apply migration (command varies by stack — see how-to/docs/DEVELOPMENT.md)
```

If the migration fails, fix the migration file and retry from Step 1.

### Step 4 — Write Migration Tests

```
/syntek-dev-suite:test-writer [migration description] --mode migration-tests
```

Tests must cover:
- Migration applies cleanly to an empty database
- Migration applies cleanly to a database with existing data
- RLS policies are enforced correctly (rows from one user/tenant are not visible to another)
- Migration is reversible (if applicable)

### Step 5 — Verify Tests Pass

Run the migration tests against an isolated test database. All tests must be green before proceeding.

### Step 6 — Code Review

```
/syntek-dev-suite:review
```

Pay particular attention to:
- RLS policy correctness
- Migration reversibility
- Index strategy

### Step 7 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If the migration causes data loss in development:
1. Revert the migration: run the down operation or restore from backup
2. Revise the migration to handle existing data safely (e.g., add DEFAULT for new NOT NULL columns)
3. Re-run from Step 1

If RLS tests fail:
1. Review the RLS policy in the migration file
2. Check that the policy covers both SELECT and write operations
3. Consult `.claude/SECURITY.md` RLS section for examples

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
