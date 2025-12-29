---
description: "[Agent] Initialise project structure and configuration"
usage: /agent:setup
---

Spawn the `syntek-dev-suite:setup` agent (model: sonnet) to set up a project.

## Pre-flight: Run Plugin Tools
Before starting setup, gather context using these plugin tools:
```bash
# Detect existing project info
python3 plugins/project-tool.py info

# Check environment files
python3 plugins/env-tool.py find

# Check container setup
python3 plugins/ddev-tool.py status
python3 plugins/docker-tool.py status
```

The agent is a Project Setup Specialist who:
- Detects language and framework
- Creates project structure and environment files
- Configures `.claude/` folder with CLAUDE.md, settings.local.json, and commands
- Sets up Docker/DDEV configuration
- Creates deployment scripts

**User's Request:**
$ARGUMENTS
