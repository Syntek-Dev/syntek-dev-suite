---
description: "[Learning] Provide feedback on the last agent response"
usage: /learning:feedback good|bad|skip [comment]
---

Record feedback for the most recent agent run. This helps the self-learning system understand what works well and what needs improvement.

**Usage:**
- `/learning:feedback good` - The response worked well
- `/learning:feedback bad` - The response needs improvement
- `/learning:feedback bad Missing security checks` - Add a comment explaining what was wrong
- `/learning:feedback skip` - Skip providing feedback this time

**How it works:**
1. Your feedback is linked to the last agent run
2. All feedback is stored in `docs/METRICS/feedback/` and committed to Git
3. The team's collective feedback helps improve agent prompts over time
4. When enough data is collected, the optimiser agent analyses patterns

**Arguments:**
$ARGUMENTS

---

Run the feedback tool to record the user's feedback:

```bash
./plugins/feedback-tool.py record $ARGUMENTS
```

After recording, display a confirmation message to the user.
