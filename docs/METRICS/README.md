# Self-Learning Metrics

## Overview

This folder contains agent performance metrics, user feedback, and prompt optimisations.
Data is committed to Git so the entire team benefits from learned improvements.

---

## Table of Contents

- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [How It Works](#how-it-works)
- [Commands](#commands)
- [Configuration](#configuration)

---

## Directory Structure

```
docs/METRICS/
├── README.md              # This file
├── config.json            # System configuration
├── runs/                  # Individual agent run records
│   └── YYYY-MM/           # Organised by month
├── feedback/              # User feedback records
│   └── YYYY-MM/
├── aggregates/            # Daily/weekly summaries
│   ├── daily/
│   └── weekly/
├── variants/              # A/B test prompt variants
│   └── {agent-name}/
├── optimisations/         # LLM-generated improvements
│   ├── pending/
│   ├── applied/
│   └── rejected/
└── templates/             # Analysis prompt templates
```

---

## How It Works

1. **Metrics Collection**: Each agent run is recorded with duration, outcome, and quality metrics
2. **Feedback**: Developers provide thumbs up/down via `/learning:feedback good|bad`
3. **A/B Testing**: Prompt variants are tested and statistically compared
4. **Auto-Optimisation**: LLM analyses patterns and automatically improves prompts (enabled by default)

---

## Commands

| Command | Purpose |
|---------|---------|
| `/learning:feedback good\|bad [comment]` | Rate the last agent response |
| `/learning:metrics [agent]` | View performance metrics |
| `/learning:ab-test create\|status\|conclude` | Manage A/B tests |
| `/learning:optimise review\|apply\|reject` | Review and apply improvements |

---

## Configuration

Edit `config.json` to customise behaviour:

| Setting | Default | Description |
|---------|---------|-------------|
| `enabled` | `true` | Enable/disable the entire learning system |
| `feedback_required` | `true` | Always prompt for feedback after agent runs |
| `auto_optimisation_enabled` | `true` | Automatically apply high-confidence improvements |
| `min_runs_for_analysis` | `50` | Minimum runs before analysing an agent |
| `retention_days` | `90` | How long to keep run records |

To disable auto-optimisation for a project, set `auto_optimisation_enabled: false` in config.json or answer "No" when prompted during `/plugin:init` or `/agent:setup`.
