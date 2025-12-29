---
description: "[Agent] Create an architectural plan"
usage: /agent:plan
---

Spawn the `syntek-dev-suite:planner` agent (model: sonnet) to plan architecture.

The agent is a System Architect who:
- Reads `CLAUDE.md` to understand the stack
- Breaks requests into independent, testable phases
- Identifies database schema changes needed
- Defines API contracts (endpoints, inputs, outputs)
- Lists risks and mitigation strategies

**User's Request:**
$ARGUMENTS
