---
description: "[Plugin] Initialise Syntek Dev Suite for a project"
usage: /plugin:init
---

# Syntek Dev Suite Initialisation

This command initialises the Syntek Dev Suite plugin for your project.

## What This Command Does

1. **Detects your project stack** (TALL, Django, React, React Native, Shared Library)
2. **Creates the `.claude/` folder structure** with:
   - `CLAUDE.md` - Project context file from the appropriate template
   - `settings.local.json` - Claude Code settings
   - `SYNTEK-GUIDE.md` - Complete plugin usage guide
3. **Sets up container configuration** (Docker or DDEV based on stack)
4. **Creates environment files** from templates

## Pre-flight: Run Plugin Tools

Before initialisation, gather project context:
```bash
# Detect existing project info
python3 plugins/project-tool.py info
python3 plugins/project-tool.py framework
python3 plugins/project-tool.py container

# Check for existing .claude folder
ls -la .claude/ 2>/dev/null || echo "No .claude folder found"

# Check for existing containers
python3 plugins/ddev-tool.py status
python3 plugins/docker-tool.py status
```

## Initialisation Process

### Step 1: Stack Detection

Detect the project stack by checking:
- `composer.json` + `artisan` → TALL Stack (Laravel)
- `manage.py` + `settings.py` → Django Stack
- `package.json` + `next.config.js` → React Stack
- `app.json` + React Native → Mobile Stack
- `package.json` + library structure → Shared Library

If stack cannot be auto-detected, ask the user:

```
Which stack are you using?

1. **TALL Stack** - Laravel + Livewire + Alpine.js + Tailwind (DDEV)
2. **Django** - Django/Wagtail + PostgreSQL (Docker)
3. **React Web** - React/Next.js + TypeScript (Docker)
4. **React Native Mobile** - Expo + NativeWind (Docker)
5. **Shared Library** - NPM package for web/mobile (Docker)
```

### Step 2: Project Naming

Ask for the project name:

```
What would you like to call this project?

This name will be used for:
- Package name (e.g., @syntek/project-name)
- Container names
- Database names
- Documentation titles
```

### Step 3: Create .claude Folder Structure

Create the following structure:

```
.claude/
├── CLAUDE.md              # Project context (from template)
├── settings.local.json    # Claude Code settings
├── SYNTEK-GUIDE.md        # Plugin usage guide
└── commands/              # Project-specific commands
    ├── dev.md
    ├── test.md
    ├── staging.md
    └── production.md
```

### Step 4: Copy Template Files

Based on detected stack, copy from the plugin's templates directory:
- `templates/tall-project.md` → `.claude/CLAUDE.md`
- `templates/django-project.md` → `.claude/CLAUDE.md`
- `templates/react-project.md` → `.claude/CLAUDE.md`
- `templates/mobile-project.md` → `.claude/CLAUDE.md`
- `templates/shared-lib-project.md` → `.claude/CLAUDE.md`

### Step 5: Create Plugin Usage Guide

Create `.claude/SYNTEK-GUIDE.md` with the complete plugin documentation (see examples/setup/SYNTEK-GUIDE-TEMPLATE.md).

### Step 6: Container Setup

Based on the stack, set up containers:

| Stack | Container | Action |
|-------|-----------|--------|
| TALL | DDEV | Create `.ddev/config.yaml` |
| Django | Docker Compose | Create `docker-compose.yml` |
| React | Docker | Create `Dockerfile` |
| Mobile | Docker | Create `Dockerfile` |
| Shared Library | Docker | Create `Dockerfile` |

Ask user:
```
Would you like to set up the development container now?

1. **Yes** - Create container configuration and start
2. **No** - Skip container setup (configure later)
```

### Step 7: Environment Files

Create environment file templates:
- `.env.dev.example`
- `.env.test.example`
- `.env.staging.example`
- `.env.production.example`

### Step 8: Self-Learning Setup

Create the `docs/METRICS/` folder structure for the self-learning system:

```
docs/METRICS/
├── README.md              # Folder documentation
├── config.json            # System configuration
├── runs/                  # Agent run records
├── feedback/              # User feedback
├── aggregates/            # Daily/weekly summaries
│   ├── daily/
│   └── weekly/
├── variants/              # A/B test prompt variants
├── optimisations/         # LLM-generated improvements
│   ├── pending/
│   ├── applied/
│   └── rejected/
└── templates/             # Analysis prompt templates
```

**Ask the user about auto-optimisation:**

```
This project will use the self-learning system to improve agent performance.

Auto-optimisation is ENABLED by default. This means:
- Agent prompts improve based on team feedback
- All developers contribute to improvements
- High-confidence changes are applied automatically

Would you like to:
1. **Keep enabled** (recommended) - Agents improve automatically
2. **Disable auto-apply** - Review all improvements manually
3. **Disable learning** - No metrics or feedback collection
```

**Create `docs/METRICS/config.json` based on response:**

Option 1 (default):
```json
{
  "enabled": true,
  "auto_optimisation_enabled": true,
  "min_runs_for_analysis": 50
}
```

Option 2:
```json
{
  "enabled": true,
  "auto_optimisation_enabled": false,
  "min_runs_for_analysis": 50
}
```

Option 3:
```json
{
  "enabled": false
}
```

### Step 9: Output Summary

After initialisation, output:

```markdown
## Syntek Dev Suite Initialised

### Project Configuration
- **Project Name:** [project-name]
- **Stack:** [detected-stack]
- **Container:** [DDEV/Docker]
- **Database:** [database-type]

### Files Created
| File | Purpose |
|------|---------|
| `.claude/CLAUDE.md` | Project context for Claude agents |
| `.claude/settings.local.json` | Claude Code settings |
| `.claude/SYNTEK-GUIDE.md` | Plugin usage guide |
| `.claude/commands/` | Project-specific commands |
| `docs/METRICS/` | Self-learning system data |

### Next Steps
1. Review and customise `.claude/CLAUDE.md`
2. Run `/agent:setup` to complete project setup
3. Use `/agent:plan` to plan your first feature
4. See `.claude/SYNTEK-GUIDE.md` for full command reference

### Quick Reference
- **Agents:** `/agent:backend`, `/agent:frontend`, `/agent:plan`, etc.
- **Templates:** `/template:tall`, `/template:django`, etc.
- **Skills:** Loaded automatically based on stack
```

## User's Request

$ARGUMENTS
