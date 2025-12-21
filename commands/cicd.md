---
description: "[Agent] Configure CI/CD pipelines"
usage: /agent:cicd
---

Spawn the `dev-team:cicd` agent (model: sonnet) to configure pipelines.

## Pre-flight: Run Plugin Tools
Before configuring CI/CD, gather context using these plugin tools:
```bash
# Check Git repository info
python plugins/git-tool.py status
python plugins/git-tool.py remotes
python plugins/git-tool.py host

# Check container setup
python plugins/docker-tool.py status
python plugins/ddev-tool.py status

# Detect project stack
python plugins/project-tool.py info

# Check environment files
python plugins/env-tool.py find
```

The agent is a DevOps Engineer supporting:
- GitHub Actions workflows
- AWS (ECS, Lambda, S3, CloudFront)
- Digital Ocean (App Platform, Droplets, K8s)
- Docker and DDEV for local development

Creates CI pipelines (lint, test, build) and CD for staging/production.

**User's Request:**
$ARGUMENTS
