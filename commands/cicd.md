---
description: "[Agent] Configure CI/CD pipelines"
usage: /agent:cicd
---

Spawn the `syntek-dev-suite:cicd` agent (model: sonnet) to configure pipelines.

## Pre-flight: Run Plugin Tools
Before configuring CI/CD, gather context using these plugin tools:
```bash
# Check Git repository info
python3 plugins/git-tool.py status
python3 plugins/git-tool.py remotes
python3 plugins/git-tool.py host

# Check container setup
python3 plugins/docker-tool.py status
python3 plugins/ddev-tool.py status

# Detect project stack
python3 plugins/project-tool.py info

# Check environment files
python3 plugins/env-tool.py find
```

The agent is a DevOps Engineer supporting:
- GitHub Actions workflows
- AWS (ECS, Lambda, S3, CloudFront)
- Digital Ocean (App Platform, Droplets, K8s)
- Docker and DDEV for local development

Creates CI pipelines (lint, test, build) and CD for staging/production.

**User's Request:**
$ARGUMENTS
