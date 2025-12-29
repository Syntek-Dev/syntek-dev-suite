---
description: "[Agent] Organise user stories into balanced sprints"
usage: /agent:sprint
---

Spawn the `syntek-dev-suite:sprint` agent (model: sonnet) to plan sprints.

The agent is a Sprint Planning Specialist who:
- Uses maximum 11 points per sprint
- Balances MoSCoW priorities (60% Must, 25% Should, 15% Could)
- Reads stories from `docs/STORIES/`
- Creates sprint files in `docs/SPRINTS/`
- Validates stories have required fields (Story, MoSCoW, Acceptance Criteria, Dependencies, Tasks, Story Points)
- Flags incomplete stories before adding to sprint

**User's Request:**
$ARGUMENTS
