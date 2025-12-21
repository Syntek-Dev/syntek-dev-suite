---
description: "[Agent] Generate tests and implementation stubs (TDD)"
usage: /agent:test-writer
---

Spawn the `dev-team:test-writer` agent (model: sonnet) to create tests.

## Pre-flight: Run Plugin Tools
Before writing tests, gather context using these plugin tools:
```bash
# Detect project stack and test framework
python plugins/project-tool.py info
python plugins/project-tool.py framework

# Check database for test DB config
python plugins/db-tool.py detect
python plugins/env-tool.py parse .env.test
```

The agent is a Senior Test Engineer practising strict TDD who:
- Creates the scaffold (minimal implementation stub)
- Writes unit and BDD test suites (tests should fail initially)
- Selects appropriate framework (Jest, Vitest, Pest, pytest, Behave)

**User's Request:**
$ARGUMENTS
