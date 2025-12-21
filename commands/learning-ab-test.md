---
description: "[Learning] Manage A/B tests for agent prompts"
usage: /learning:ab-test <command> [agent] [options]
---

Manage A/B testing of agent prompt variants. Test different versions of agent prompts to see which performs better.

**Commands:**

- `/learning:ab-test list` - List all active A/B tests
- `/learning:ab-test status <agent>` - Get status and results for an agent's test
- `/learning:ab-test create <agent>` - Create a new A/B test (interactive)
- `/learning:ab-test conclude <agent> [winner]` - Conclude a test and optionally apply the winner

**How A/B Testing Works:**

1. **Create a test**: Define a variant with modified prompt instructions
2. **Run agents**: The system randomly selects baseline or variant for each run
3. **Collect feedback**: Team members provide feedback via `/learning:feedback`
4. **Analyse results**: Statistical significance is calculated automatically
5. **Conclude**: Apply the winning variant or archive the test

**Example Workflow:**

```
# List current tests
/learning:ab-test list

# Check status of backend agent test
/learning:ab-test status backend

# Conclude test and apply the winner
/learning:ab-test conclude backend enhanced-security
```

**Arguments:**
$ARGUMENTS

---

Based on the command, run the appropriate ab-test-tool.py command:

- For `list`: `./plugins/ab-test-tool.py list`
- For `status <agent>`: `./plugins/ab-test-tool.py status <agent>`
- For `create <agent>`: Guide the user through creating a variant
- For `conclude <agent> [winner]`: `./plugins/ab-test-tool.py conclude <agent> [winner]`

Display results in a clear, formatted manner.
