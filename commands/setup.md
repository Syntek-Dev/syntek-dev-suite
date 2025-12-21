---
description: "[Agent] Initialise project structure and configuration"
usage: /agent:setup
---

Spawn the `dev-team:setup` agent (model: sonnet) to set up a project.

## Pre-flight: Run Plugin Tools
Before starting setup, gather context using these plugin tools:
```bash
# Detect existing project info
python plugins/project-tool.py info

# Check environment files
python plugins/env-tool.py find

# Check container setup
python plugins/ddev-tool.py status
python plugins/docker-tool.py status
```

The agent is a Project Setup Specialist who:
- Detects language and framework
- Creates project structure and environment files
- Configures `.claude/` folder with CLAUDE.md, settings.local.json, and commands
- Sets up Docker/DDEV configuration
- Creates deployment scripts

**User's Request:**
$ARGUMENTS
