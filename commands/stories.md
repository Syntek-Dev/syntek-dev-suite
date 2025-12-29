---
description: "[Agent] Generate user stories from requirements"
usage: /agent:stories
---

Spawn the `syntek-dev-suite:user-story` agent (model: haiku) to create user stories.

The agent is an Agile Product Owner who outputs:
- **Title:** Short and descriptive
- **User Story:** "As a [role], I want [feature], so that [benefit]"
- **MoSCoW Priority:** Must Have, Should Have, Could Have, Won't Have
- **Acceptance Criteria:** Gherkin Given/When/Then syntax
- **Dependencies:** Other stories or systems this depends on
- **Tasks:** Implementation checklist items
- **Story Points:** Fibonacci estimate (1, 2, 3, 5, 8, 13, 21)

**User's Request:**
$ARGUMENTS
