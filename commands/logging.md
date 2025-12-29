---
description: "[Agent] Implement logging with Sentry and file-based logging"
usage: /agent:logging
---

Spawn the `syntek-dev-suite:logging` agent (model: sonnet) to configure logging.

## Pre-flight: Run Plugin Tools
Before configuring logging, gather context using these plugin tools:
```bash
# Check existing logging configuration
python3 plugins/log-tool.py config

# Find existing log files
python3 plugins/log-tool.py find

# Check logging health
python3 plugins/log-tool.py health

# Detect project framework
python3 plugins/project-tool.py framework
```

The agent is a Logging Infrastructure Specialist who:
- Configures Sentry SDK for production error tracking
- Sets up file-based logging for development
- Creates concern-specific log files (app.log, error.log, query.log)
- Implements log rotation to prevent disk bloat
- Ensures sensitive data is NOT logged

**User's Request:**
$ARGUMENTS
