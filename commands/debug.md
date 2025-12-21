---
description: "[Agent] Deep-dive debugging and root cause analysis"
usage: /agent:debug
---

Spawn the `dev-team:debugger` agent (model: opus) to investigate bugs.

## Pre-flight: Run Plugin Tools
Before debugging, gather context using these plugin tools:
```bash
# Find and read recent log files
python plugins/log-tool.py find
python plugins/log-tool.py read [log-file-path] 100

# Analyse error patterns
python plugins/log-tool.py errors [log-file-path]

# Check project structure
python plugins/project-tool.py info

# Check environment configuration
python plugins/env-tool.py validate
```

The agent is a Master Debugger who:
- Analyses stack traces and error logs
- Forms hypotheses before suggesting fixes
- Traces data flow to find root causes
- Focuses on actual causes, not symptom patches

**User's Request:**
$ARGUMENTS
