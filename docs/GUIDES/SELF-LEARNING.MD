# Self-Learning System Guide

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Giving Feedback](#giving-feedback)
- [Viewing Metrics](#viewing-metrics)
- [A/B Testing](#ab-testing)
- [Optimisation](#optimisation)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

---

## Overview

The self-learning system helps agents improve over time based on team feedback. Every time you use an agent, the system:

1. **Records the run** - Duration, outcome, files changed
2. **Prompts for feedback** - Quick good/bad rating
3. **Analyses patterns** - What works, what doesn't
4. **Proposes improvements** - Auto-optimises prompts

All data is stored in `docs/METRICS/` and committed to Git, so the entire team benefits from improvements.

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        LEARNING CYCLE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. USE AGENT              2. GIVE FEEDBACK                     │
│  /agent:backend            /learning:feedback good              │
│        │                          │                              │
│        ▼                          ▼                              │
│  ┌──────────┐              ┌──────────────┐                     │
│  │ Run      │──────────────│ Feedback     │                     │
│  │ Recorded │              │ Recorded     │                     │
│  └──────────┘              └──────────────┘                     │
│        │                          │                              │
│        └──────────┬───────────────┘                              │
│                   ▼                                              │
│            ┌──────────────┐                                     │
│            │ Aggregated   │                                     │
│            │ in Metrics   │                                     │
│            └──────────────┘                                     │
│                   │                                              │
│                   ▼                                              │
│  3. ANALYSE               4. OPTIMISE                           │
│  /learning:optimise       /learning:optimise apply <id>        │
│  analyse backend                                                 │
│        │                          │                              │
│        ▼                          ▼                              │
│  ┌──────────────┐         ┌──────────────┐                      │
│  │ Patterns     │────────▶│ Improved     │                      │
│  │ Identified   │         │ Agent Prompt │                      │
│  └──────────────┘         └──────────────┘                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Getting Started

### Setup (Automatic)

When you run `/agent:setup` on a new project, you'll be asked:

```
Would you like to enable auto-optimisation?
1. Keep enabled (recommended) - Agents improve automatically
2. Disable auto-apply - Review all improvements manually
3. Disable learning - No metrics or feedback collection
```

The default is **enabled** - agents will automatically improve based on team feedback.

### Manual Setup

If setting up an existing project:

1. Create the metrics directory structure:
   ```bash
   mkdir -p docs/METRICS/{runs,feedback,aggregates,variants,optimisations}
   ```

2. Create `docs/METRICS/config.json`:
   ```json
   {
     "enabled": true,
     "auto_optimisation_enabled": true,
     "min_runs_for_analysis": 50
   }
   ```

3. Commit to Git so the team can access it.

---

## Giving Feedback

### After Every Agent Run

You'll see a prompt:

```
---
How was this response?
  /learning:feedback good  - Worked well
  /learning:feedback bad   - Needs improvement
---
```

### Quick Decision Guide

| Rate | When |
|------|------|
| **good** | Code works, follows patterns, well documented |
| **bad** | Missing requirements, wrong patterns, errors |
| **skip** | Can't evaluate yet, need to test more |

### Adding Comments

For "bad" ratings, add a comment to help the optimiser:

```
/learning:feedback bad Missing input validation on the form
/learning:feedback bad Didn't follow the existing service pattern
/learning:feedback bad Security: No CSRF protection added
```

### Feedback Commands

| Command | Description |
|---------|-------------|
| `/learning:feedback good` | Quick positive rating |
| `/learning:feedback bad` | Quick negative rating |
| `/learning:feedback bad <comment>` | Negative with explanation |
| `/learning:feedback skip` | Skip this rating |

---

## Viewing Metrics

### Check System Status

```
/learning:metrics status
```

Shows:
- Total runs recorded
- Feedback collection rate
- Auto-optimisation status

### View Agent Performance

```
/learning:metrics summary backend
/learning:metrics summary frontend --days 30
```

Shows:
- Total runs
- Completion rate
- Satisfaction rate
- Average duration

### Query Specific Data

```
# Recent runs
./plugins/metrics-tool.py query --days 7

# Agent-specific
./plugins/metrics-tool.py query --agent backend --days 14

# Feedback analysis
./plugins/feedback-tool.py analyse backend
```

---

## A/B Testing

Test different versions of agent prompts to see which performs better.

### Creating a Test

1. Create a variant file in `docs/METRICS/variants/<agent>/`:
   ```
   docs/METRICS/variants/backend/enhanced-security.md
   ```

2. Register the test:
   ```
   /learning:ab-test create backend --variant enhanced-security
   ```

### Checking Test Status

```
/learning:ab-test status backend
```

Shows:
- Runs per variant
- Satisfaction rates
- Statistical significance

### Concluding a Test

When you have enough data (usually 50+ runs per variant):

```
# Check if results are significant
/learning:ab-test status backend

# Apply the winner
/learning:ab-test conclude backend enhanced-security
```

---

## Optimisation

### Analysing an Agent

```
/learning:optimise analyse backend
```

This will:
1. Gather 14 days of metrics
2. Analyse feedback patterns
3. Identify improvement opportunities
4. Generate a proposal (if issues found)

### Reviewing Proposals

```
# List pending proposals
/learning:optimise list

# View a specific proposal
/learning:optimise review opt-20251221-backend
```

### Applying or Rejecting

```
# Apply a proposal
/learning:optimise apply opt-20251221-backend

# Reject with reason
/learning:optimise reject opt-20251221-backend Changes too aggressive
```

### Rolling Back

If an applied change causes problems:

```
/learning:optimise rollback backend
```

This restores the previous version from backup.

---

## Configuration

### Config File Location

`docs/METRICS/config.json`

### Options

| Setting | Default | Description |
|---------|---------|-------------|
| `enabled` | `true` | Enable/disable entire system |
| `feedback_prompt_enabled` | `true` | Show feedback prompt after runs |
| `feedback_required` | `true` | Always prompt for feedback |
| `ab_testing_enabled` | `true` | Allow A/B testing |
| `auto_optimisation_enabled` | `true` | Auto-apply high-confidence improvements |
| `min_runs_for_analysis` | `50` | Minimum runs before analysing |
| `retention_days` | `90` | How long to keep run data |

### Safety Thresholds

```json
{
  "safety_thresholds": {
    "min_satisfaction_rate": 0.60,
    "max_rerun_rate": 0.30,
    "confidence_required": 0.70
  }
}
```

- `min_satisfaction_rate`: Auto-rollback if satisfaction drops below 60%
- `confidence_required`: Only auto-apply with 70%+ confidence

---

## Troubleshooting

### Feedback Not Being Recorded

1. Check if the system is enabled:
   ```
   ./plugins/metrics-tool.py status
   ```

2. Verify `docs/METRICS/config.json` exists and has `enabled: true`

3. Check if the last run was recorded:
   ```
   ./plugins/metrics-tool.py last
   ```

### Optimisation Not Finding Issues

The system needs:
- At least 50 runs for an agent
- At least some negative feedback with comments
- Enough data to identify patterns

Run:
```
./plugins/optimiser-tool.py analyse backend
```

To see why analysis isn't ready.

### Rollback Not Working

Backups are stored in `docs/METRICS/optimisations/backups/`. Check if the backup exists:

```
ls docs/METRICS/optimisations/backups/
```

### Data Not Syncing Across Team

Ensure `docs/METRICS/` is:
1. Not in `.gitignore`
2. Being committed and pushed
3. Being pulled by other team members

---

## Best Practices

1. **Give feedback consistently** - Even quick good/bad ratings help
2. **Add comments for bad ratings** - Helps the optimiser understand issues
3. **Review proposals before auto-apply** - Check `docs/METRICS/optimisations/pending/`
4. **Monitor after changes** - Watch satisfaction rates after applying improvements
5. **Use A/B tests for risky changes** - Test variants before committing

---

## Commands Reference

| Command | Purpose |
|---------|---------|
| `/learning:feedback good\|bad\|skip [comment]` | Rate last response |
| `/learning:metrics [agent] [--days N]` | View metrics |
| `/learning:ab-test list` | List active tests |
| `/learning:ab-test status <agent>` | Check test results |
| `/learning:ab-test conclude <agent> [winner]` | End a test |
| `/learning:optimise status` | System status |
| `/learning:optimise analyse <agent>` | Analyse an agent |
| `/learning:optimise list` | List proposals |
| `/learning:optimise apply <id>` | Apply a proposal |
| `/learning:optimise reject <id> [reason]` | Reject a proposal |
| `/learning:optimise rollback <agent>` | Restore previous version |
