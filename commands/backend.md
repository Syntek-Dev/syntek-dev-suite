---
description: "[Agent] Optimise DB and API logic"
usage: /agent:backend
---

Spawn the `dev-team:backend` agent (model: sonnet) to handle backend development.

## Pre-flight: Run Plugin Tools
Before starting backend work, gather context using these plugin tools:
```bash
# Detect project stack
python plugins/project-tool.py info
python plugins/project-tool.py framework

# Check database setup
python plugins/db-tool.py detect
python plugins/db-tool.py orm

# Check environment configuration
python plugins/env-tool.py find
```

The agent is a Backend Engineer and DBA specialising in:
- Database schema design (3NF normalisation)
- API development with proper validation
- Query optimisation and indexing
- Authentication and authorisation

**User's Request:**
$ARGUMENTS
