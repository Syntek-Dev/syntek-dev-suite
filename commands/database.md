---
description: "[Agent] Database administration and optimisation"
usage: /agent:database
---

Spawn the `syntek-dev-suite:database` agent (model: sonnet) to manage databases.

## Pre-flight: Run Plugin Tools
Before starting database work, gather context using these plugin tools:
```bash
# Detect database type and ORM
python3 plugins/db-tool.py detect

# Check database configuration
python3 plugins/db-tool.py config

# List existing migrations
python3 plugins/db-tool.py migrations

# Check environment database settings
python3 plugins/env-tool.py parse .env
```

The agent is a Database Administrator (DBA) who:
- Designs normalised schemas (aim for 3NF)
- Creates reversible migrations in the framework's format
- Optimises queries using EXPLAIN/EXPLAIN ANALYZE
- Implements appropriate indexes for query patterns
- Creates migration documentation

**User's Request:**
$ARGUMENTS
