# Architecture Decision — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] The decision to be made is identified (technology choice, structural change, pattern adoption)
- [ ] Stakeholders who need to be consulted are identified
- [ ] Branch created following `how-to/workflows/02-git-workflow/`
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Generate Architectural Plan

```
/syntek-dev-suite:plan [decision description and context]
```

The planner agent will:
- Frame the decision with context and constraints
- Identify 2–3 viable options with trade-offs
- Recommend an approach with rationale
- Identify risks and mitigations
- Estimate implementation impact

### Step 2 — Create ADR Document

Based on the plan output, create an Architecture Decision Record (ADR) in `project-management/src/PLANS/`:

```
project-management/src/PLANS/ADR-{NNN}-{short-title}.md
```

ADR format:
- **Title**: Short descriptive title
- **Status**: Proposed / Accepted / Deprecated / Superseded
- **Context**: What problem or decision prompted this ADR
- **Decision**: What was decided
- **Rationale**: Why this option was chosen over alternatives
- **Consequences**: What becomes easier, harder, or changes as a result
- **Alternatives considered**: What was rejected and why

### Step 3 — Review and Validate

Share the ADR with relevant team members. Update the Status from `Proposed` to `Accepted` once consensus is reached.

If rejected, update Status to `Rejected` and record the reason.

### Step 4 — Update `.claude/CLAUDE.md` (if applicable)

If the ADR changes a fundamental project convention (stack, pattern, tool choice), update the relevant section of `.claude/CLAUDE.md` to reflect the new decision.

### Step 5 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If the plan reveals the decision is more complex than anticipated:
1. Break the ADR into smaller, more focused decisions
2. Address foundational decisions first
3. Create separate ADRs for each sub-decision

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
