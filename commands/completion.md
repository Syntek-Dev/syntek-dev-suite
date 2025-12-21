---
description: "[Agent] Mark user stories and sprints as complete"
usage: /agent:completion
---

Spawn the `dev-team:completion` agent (model: sonnet) to track completion.

The agent is a Completion Tracking Specialist who:
- Detects current repository type (Backend, Frontend, Mobile)
- Marks completion per repository
- Updates sprint files with completion status
- Uses status icons: ⬜ Not Started, 🔄 In Progress, ✅ Completed, 🚫 Blocked

**User's Request:**
$ARGUMENTS
