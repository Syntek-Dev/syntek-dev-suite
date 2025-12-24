# .gitignore Templates

## Overview

Standard ignore file templates for different project types. These templates cover common patterns across all supported stacks.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 1.0.0 |
| **Last Updated** | 2025-01 |
| **Stacks** | All (TALL, Django, React, Mobile, Shared-Lib) |

---

## Table of Contents

- [.gitignore Templates](#gitignore-templates)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [Universal .gitignore](#universal-gitignore)
  - [.dockerignore](#dockerignore)
  - [.gitattributes](#gitattributes)

---

## Universal .gitignore

```gitignore
# Environment files
.env
.env.dev
.env.staging
.env.production
.env.local
.env.*.local

# Dependencies
node_modules/
vendor/
__pycache__/
*.pyc
.venv/
venv/

# Build outputs
dist/
build/
.next/
out/

# IDE
.idea/
.vscode/
*.swp
*.swo
.DS_Store

# Logs
*.log
logs/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Testing
coverage/
.coverage
htmlcov/
.pytest_cache/

# DDEV
.ddev/.gitignore

# Docker
.docker/

# Secrets
*.pem
*.key
secrets/
```

---

## .dockerignore

```dockerignore
# Git
.git
.gitignore

# Documentation
README.md
docs/
*.md

# Dependencies (reinstalled in container)
node_modules/
vendor/
__pycache__/
.venv/
venv/

# Environment files
.env
.env.*
!.env.*.example

# Build artifacts
dist/
build/
.next/
out/

# IDE
.idea/
.vscode/

# Testing
coverage/
.coverage
htmlcov/
.pytest_cache/
tests/

# CI/CD
.github/
.gitlab-ci.yml

# DDEV
.ddev/

# Logs
*.log
logs/
```

---

## .gitattributes

```gitattributes
# .gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf

# Binary files
*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.ico binary
*.pdf binary
*.woff binary
*.woff2 binary
*.ttf binary
*.eot binary

# Lock files - don't diff
package-lock.json -diff
yarn.lock -diff
composer.lock -diff
pnpm-lock.yaml -diff

# LFS (if using large files)
# *.psd filter=lfs diff=lfs merge=lfs -text
```