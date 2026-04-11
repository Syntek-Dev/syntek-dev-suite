# Sprint Planning — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Product backlog is populated with user stories in `project-management/src/STORIES/`
- [ ] Stories have MoSCoW priority labels (Must / Should / Could / Won't)
- [ ] Team capacity for the sprint is known
- [ ] Previous sprint has been marked complete (or is explicitly carried over)
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Review Backlog

Read the stories in `project-management/src/STORIES/` and identify candidates for the upcoming sprint. Consider:
- Must-have items that are overdue
- Dependencies between stories (some stories block others)
- Technical debt that must be addressed before new features
- Carryover items from the previous sprint

### Step 2 — Run Sprint Planning Agent

```
/syntek-dev-suite:sprint
```

The sprint agent reads the available stories and:
- Applies MoSCoW prioritisation
- Balances Must / Should / Could items for the sprint
- Flags dependencies and sequencing requirements
- Produces a sprint plan document

### Step 3 — Review Sprint Balance

Check the sprint agent output for balance:

- [ ] At least 60% of sprint capacity is Must-have items
- [ ] No more than 20% is Could-have items
- [ ] Dependencies are respected (blocking stories come first)
- [ ] Capacity is not overcommitted

Adjust the sprint scope if needed and re-run the agent.

### Step 4 — Create Sprint Document

Save the sprint plan to `project-management/src/SPRINTS/`:

```
project-management/src/SPRINTS/SPRINT-{NN}-{YYYY-MM-DD}.md
```

Include:
- Sprint number and dates
- Goal / theme
- Stories included with MoSCoW priority
- Capacity allocation
- Definition of done for this sprint

### Step 5 — Update Stories

For each story included in the sprint, update its status to `In Sprint` and add the sprint reference.

### Step 6 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If there are too many Must-have items for the sprint capacity:
1. Escalate to the product owner — some Must-have items may need to be re-prioritised
2. Do not reduce quality (skip testing, skip review) to fit more stories in
3. Carry excess Must-have items to the top of the next sprint backlog

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
