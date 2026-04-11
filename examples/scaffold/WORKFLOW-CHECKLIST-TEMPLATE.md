# {WORKFLOW_NAME} — Checklist

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Pre-Conditions

Confirm all of these before starting:

- [ ] Branch is up to date with `dev` (or `main` for hotfixes)
- [ ] No uncommitted changes from a previous workflow
- [ ] All dependencies listed in `CONTEXT.md` are satisfied
- [ ] `/GAPS.md` has been checked — no blocking gaps for this workflow

---

## Execution Checklist

Tick each item as you complete it. Do not mark the workflow complete until all items are checked.

- [ ] All steps in `STEPS.md` followed in order
- [ ] No steps skipped or reordered without explicit reason
- [ ] All referenced agents invoked and completed successfully
- [ ] Output artefacts created in the correct locations
- [ ] No new linting errors introduced (`/syntek-dev-suite:syntax` clean)
- [ ] Tests pass (if applicable to this workflow)

---

## Definition of Done

The workflow is complete when ALL of the following are true:

- [ ] The primary objective stated in `CONTEXT.md` has been achieved
- [ ] All output artefacts are committed and pushed
- [ ] `/syntek-dev-suite:git` has been run and commit message is accurate
- [ ] Any new gaps discovered during this workflow are logged in `/GAPS.md`
- [ ] The relevant story or ticket has been updated (if applicable)

---

## Notes

Add any workflow-specific completion criteria below. Customise this section for your project's definition of done.

- [ ] [Customise: project-specific completion criterion]
- [ ] [Customise: project-specific completion criterion]
