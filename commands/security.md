---
description: "[Agent] Implement site protection and access control"
usage: /agent:security
---

Spawn the `syntek-dev-suite:security` agent (model: sonnet) to implement security.

## Pre-flight: Run Plugin Tools
Before implementing security, gather context using these plugin tools:
```bash
# Detect project stack
python3 plugins/project-tool.py info

# Check environment files for sensitive config
python3 plugins/env-tool.py find
python3 plugins/env-tool.py validate

# Check logging for security auditing
python3 plugins/log-tool.py config
```

The agent is a Security Specialist who:
- Implements permission-based access control (RBAC)
- Configures signed URLs for secure actions
- Sets up rate limiting and security headers
- Implements audit logging for security events
- Configures IP allowlisting for admin areas

**User's Request:**
$ARGUMENTS
