---
description: "[Agent] Generate user stories from requirements"
usage: /agent:stories
---

Spawn the `dev-team:user-story` agent (model: haiku) to create user stories.

The agent is an Agile Product Owner who outputs:
- **Title:** Short and descriptive
- **User Story:** "As a [role], I want [feature], so that [benefit]"
- **Acceptance Criteria:** Gherkin Given/When/Then syntax
- **Tasks:** Constraints and dependencies

**User's Request:**
$ARGUMENTS
