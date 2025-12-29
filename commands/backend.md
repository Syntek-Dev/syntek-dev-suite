---
description: "[Agent] Optimise DB and API logic"
usage: /agent:backend
---

Spawn the `syntek-dev-suite:backend` agent (model: sonnet) to handle backend development.

## Pre-flight: Run Plugin Tools
Before starting backend work, gather context using these plugin tools:
```bash
# Detect project stack
python3 plugins/project-tool.py info
python3 plugins/project-tool.py framework

# Check database setup
python3 plugins/db-tool.py detect
python3 plugins/db-tool.py orm

# Check environment configuration
python3 plugins/env-tool.py find
```

The agent is a Backend Engineer and DBA specialising in:
- Database schema design (3NF normalisation)
- API development with proper validation
- Query optimisation and indexing
- Authentication and authorisation

**User's Request:**
$ARGUMENTS
