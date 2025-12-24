# Plugins

## Overview

This folder contains Python utility tools that agents use to gather information about the development environment. These tools return structured JSON output that agents use to make informed decisions.

All tools are executable Python scripts that can be run directly from the command line.

**Version:** 1.1.0

---

## Table of Contents

- [Plugins](#plugins)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Directory Tree](#directory-tree)
  - [Tools](#tools)
    - [Environment Detection](#environment-detection)
    - [Project Analysis](#project-analysis)
    - [Self-Learning System](#self-learning-system)
  - [Usage](#usage)
    - [Direct Execution](#direct-execution)
    - [Agent Usage](#agent-usage)
    - [JSON Output](#json-output)
  - [Related Sections](#related-sections)


## Directory Tree

```
plugins/
├── README.md           # This file
├── ab-test-tool.py     # A/B testing for prompt variants
├── chrome-tool.py      # Cross-platform Chrome detection and configuration
├── db-tool.py          # Database type and ORM detection
├── ddev-tool.py        # DDEV project status and configuration
├── docker-tool.py      # Docker containers, compose, images
├── env-tool.py         # Environment file discovery and validation
├── feedback-tool.py    # User feedback collection
├── git-tool.py         # Git repository status, branches, remotes
├── log-tool.py         # Log file discovery and analysis
├── metrics-tool.py     # Self-learning metrics recording
├── optimiser-tool.py   # Prompt analysis and optimisation
├── project-tool.py     # Language, framework, structure detection
└── quality-tool.py     # Code quality checks and linting
```

---

## Tools

### Environment Detection

| Tool | Command | Description |
|------|---------|-------------|
| `chrome-tool.py` | `./plugins/chrome-tool.py detect` | Cross-platform Chrome detection and configuration |
| `ddev-tool.py` | `./plugins/ddev-tool.py status` | Check DDEV container status and configuration |
| `docker-tool.py` | `./plugins/docker-tool.py status` | Check Docker containers, compose, images, networks |
| `env-tool.py` | `./plugins/env-tool.py find` | Discover and validate environment files |

### Project Analysis

| Tool | Command | Description |
|------|---------|-------------|
| `project-tool.py` | `./plugins/project-tool.py info` | Detect language, framework, and project structure |
| `db-tool.py` | `./plugins/db-tool.py detect` | Detect database type and ORM in use |
| `git-tool.py` | `./plugins/git-tool.py status` | Get Git repository status, branches, remotes |
| `log-tool.py` | `./plugins/log-tool.py find` | Find and analyse log files |
| `quality-tool.py` | `./plugins/quality-tool.py check` | Run code quality checks and linting |

### Self-Learning System

| Tool | Command | Description |
|------|---------|-------------|
| `metrics-tool.py` | `./plugins/metrics-tool.py record` | Record agent run metrics |
| `feedback-tool.py` | `./plugins/feedback-tool.py submit` | Submit user feedback on agent output |
| `ab-test-tool.py` | `./plugins/ab-test-tool.py list` | Manage A/B tests for prompt variants |
| `optimiser-tool.py` | `./plugins/optimiser-tool.py analyse` | Analyse and optimise agent prompts |

---

## Usage

### Direct Execution

Tools can be run directly from the command line:

```bash
# Get project information
./plugins/project-tool.py info

# Check database configuration
./plugins/db-tool.py detect

# Find environment files and validate
./plugins/env-tool.py find
./plugins/env-tool.py validate .env .env.example

# Check DDEV status
./plugins/ddev-tool.py status
```

### Agent Usage

Agents use these tools automatically to gather context:

| Agent | Tools Used |
|-------|------------|
| **Setup Agent** | `project-tool.py`, `env-tool.py`, `docker-tool.py`, `ddev-tool.py`, `chrome-tool.py` |
| **CI/CD Agent** | `git-tool.py`, `docker-tool.py`, `ddev-tool.py` |
| **Database Agent** | `db-tool.py`, `env-tool.py` |
| **Backend Agent** | `project-tool.py`, `db-tool.py`, `env-tool.py` |
| **Logging Agent** | `log-tool.py`, `project-tool.py` |
| **Debugger Agent** | `log-tool.py`, `env-tool.py`, `chrome-tool.py` |
| **QA Tester Agent** | `project-tool.py`, `chrome-tool.py` |
| **Test Writer Agent** | `project-tool.py`, `chrome-tool.py` |

### JSON Output

All tools return structured JSON for easy parsing:

```json
{
  "status": "success",
  "data": {
    "framework": "laravel",
    "version": "12.0",
    "database": "mariadb"
  }
}
```

---

## Related Sections

- [../agents/](../agents/) - Agents that use these tools
- [../commands/](../commands/) - Commands that invoke agents
- [../CLAUDE.md](../CLAUDE.md) - Plugin tool definitions in config
