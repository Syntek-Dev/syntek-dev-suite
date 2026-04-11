# Story Writing — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Requirements exist in some form (brief, conversation notes, design, ticket)
- [ ] A product owner or stakeholder is available to review stories
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Generate User Stories

```
/syntek-dev-suite:stories [requirements description]
```

The stories agent converts requirements into structured user stories. It will:
- Write stories in "As a [role], I want [goal], so that [benefit]" format
- Add acceptance criteria for each story
- Assign a MoSCoW priority
- Estimate relative complexity (if requested)
- Flag technical dependencies

### Step 2 — Review Generated Stories

Review each story for:

- [ ] Clear role (who benefits)
- [ ] Specific goal (what they want to achieve)
- [ ] Meaningful benefit (why it matters)
- [ ] Testable acceptance criteria
- [ ] Appropriately scoped (fits within one sprint)

Stories that are too large should be split. Stories that are too small should be merged.

### Step 3 — Assign Story Numbers

Stories are numbered sequentially: `US-001`, `US-002`, etc. Check existing stories in `project-management/src/STORIES/` to find the next available number.

### Step 4 — Save Stories

Save each story to `project-management/src/STORIES/`:

```
project-management/src/STORIES/US-{NNN}-{short-title}.md
```

Use the user story template from `project-management/src/STORIES/` (if one exists) or the standard format from `.claude/ARCHITECTURE-PATTERNS.md`.

### Step 5 — Stakeholder Review

Share stories with the relevant stakeholder or product owner for review. Update based on feedback. Mark each story as `Ready for Sprint` once approved.

### Step 6 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If the requirements are too vague to write specific stories:
1. Return to the stakeholder for clarification before proceeding
2. Do not write stories with placeholder acceptance criteria — these will produce incorrect implementations
3. Document the ambiguity in `project-management/src/PLANS/` and request a requirements clarification meeting

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
