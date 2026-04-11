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
   - `plugins/*.py` - add the custom plugins from `syntek-dev-suite/plugins/*.py` and allow them to be executable using `chmod +x .claude/plugins/*.py`.
3. **Sets up container configuration** (Docker or DDEV based on stack)
4. **Creates environment files** from templates

## Step 0: Locate Plugin Directory

Before anything else, locate the syntek-dev-suite installation directory dynamically using the Glob tool. Do **not** assume a fixed path — the plugin may be installed anywhere on the device.

Search for `plugins/project-tool.py` using Glob in this order:
1. `~/.claude/plugins/syntek-dev-suite/plugins/project-tool.py`
2. `~/.claude/plugins/**/plugins/project-tool.py`
3. `~/Repos/**/syntek-dev-suite/plugins/project-tool.py`
4. `~/**/syntek-dev-suite/plugins/project-tool.py`

Once found, derive `SYNTEK_DIR` as the directory **two levels above** `project-tool.py` — that is, the syntek-dev-suite root containing `plugins/`, `examples/`, `templates/`, and `config.json`.

Use `SYNTEK_DIR` as the base for **all** plugin commands and file copies throughout this workflow.

## Pre-flight: Run Plugin Tools

Gather project context using the discovered plugin path:

```bash
# Detect existing project info
python3 $SYNTEK_DIR/plugins/project-tool.py info
python3 $SYNTEK_DIR/plugins/project-tool.py framework
python3 $SYNTEK_DIR/plugins/project-tool.py container

# Check for existing .claude folder
ls -la .claude/ 2>/dev/null || echo "No .claude folder found"

# Check for existing containers
python3 $SYNTEK_DIR/plugins/ddev-tool.py status
python3 $SYNTEK_DIR/plugins/docker-tool.py status
```

> **Note:** After initialisation, the plugins will be available at `.claude/plugins/` within your project. All subsequent agent commands use `.claude/plugins/` — only this init step needs `SYNTEK_DIR`.

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
├── CLAUDE.md                  # Project context (from template)
├── CODING-PRINCIPLES.md       # Rob Pike + Linus Torvalds coding rules
├── TESTING.md                 # Testing guide for this project's stack
├── SECURITY.md                # Security patterns and compliance checklist
├── ACCESSIBILITY.md           # WCAG 2.2 AA compliance and ARIA patterns
├── API-DESIGN.md              # REST and GraphQL API conventions
├── ARCHITECTURE-PATTERNS.md   # Service layer, middleware, and project structure
├── DATA-STRUCTURES.md         # Domain modelling and database schema design
├── PERFORMANCE.md             # Query optimisation, caching, and frontend performance
├── DEVELOPMENT.md             # Development workflow and common tasks
├── SEO-CHECKLIST.md           # SEO & AI discoverability checklist (Beginner → Advanced)
├── settings.local.json        # Claude Code settings
├── SYNTEK-GUIDE.md            # Plugin usage guide
├── plugins/                   # Custom Python plugins (copied from syntek-dev-suite)
│   └── *.py
└── commands/                  # Project-specific commands
    ├── dev.md
    ├── test.md
    ├── staging.md
    └── production.md
```

### Step 3.5: Copy Plugin Tools

Copy the Python plugin tools from the syntek-dev-suite to the project using the `SYNTEK_DIR` discovered in Step 0:

```bash
# Create plugins directory
mkdir -p .claude/plugins/

# Copy all Python plugins from syntek-dev-suite
cp $SYNTEK_DIR/plugins/*.py .claude/plugins/

# Make them executable
chmod +x .claude/plugins/*.py
```

This gives the project its own copy of the tools for:

- Project-specific customisation if needed
- Portability (project works even without syntek-dev-suite installed)
- Version stability

### Step 4: Copy Template Files

Based on detected stack, copy from the plugin's templates directory using `SYNTEK_DIR`:
- `$SYNTEK_DIR/templates/tall-project.md` → `.claude/CLAUDE.md`
- `$SYNTEK_DIR/templates/django-project.md` → `.claude/CLAUDE.md`
- `$SYNTEK_DIR/templates/react-project.md` → `.claude/CLAUDE.md`
- `$SYNTEK_DIR/templates/mobile-project.md` → `.claude/CLAUDE.md`
- `$SYNTEK_DIR/templates/shared-lib-project.md` → `.claude/CLAUDE.md`

Also copy all required reference files using `SYNTEK_DIR`:
- `examples/setup/CODING-PRINCIPLES.md` → `.claude/CODING-PRINCIPLES.md`
- `examples/setup/TESTING.md` → `.claude/TESTING.md`
- `examples/setup/SECURITY.md` → `.claude/SECURITY.md`
- `examples/setup/ACCESSIBILITY.md` → `.claude/ACCESSIBILITY.md`
- `examples/setup/API-DESIGN.md` → `.claude/API-DESIGN.md`
- `examples/setup/ARCHITECTURE-PATTERNS.md` → `.claude/ARCHITECTURE-PATTERNS.md`
- `examples/setup/DATA-STRUCTURES.md` → `.claude/DATA-STRUCTURES.md`
- `examples/setup/PERFORMANCE.md` → `.claude/PERFORMANCE.md`
- `examples/setup/DEVELOPMENT.md` → `.claude/DEVELOPMENT.md`
- `examples/setup/SEO-CHECKLIST.md` → `.claude/SEO-CHECKLIST.md`

```bash
cp $SYNTEK_DIR/examples/setup/CODING-PRINCIPLES.md .claude/CODING-PRINCIPLES.md
cp $SYNTEK_DIR/examples/setup/TESTING.md .claude/TESTING.md
cp $SYNTEK_DIR/examples/setup/SECURITY.md .claude/SECURITY.md
cp $SYNTEK_DIR/examples/setup/ACCESSIBILITY.md .claude/ACCESSIBILITY.md
cp $SYNTEK_DIR/examples/setup/API-DESIGN.md .claude/API-DESIGN.md
cp $SYNTEK_DIR/examples/setup/ARCHITECTURE-PATTERNS.md .claude/ARCHITECTURE-PATTERNS.md
cp $SYNTEK_DIR/examples/setup/DATA-STRUCTURES.md .claude/DATA-STRUCTURES.md
cp $SYNTEK_DIR/examples/setup/PERFORMANCE.md .claude/PERFORMANCE.md
cp $SYNTEK_DIR/examples/setup/DEVELOPMENT.md .claude/DEVELOPMENT.md
cp $SYNTEK_DIR/examples/setup/SEO-CHECKLIST.md .claude/SEO-CHECKLIST.md
```

These files contain the full project standards that all agents read before working. Every project `.claude/CLAUDE.md` must reference all ten files.

### Step 5: Create Plugin Usage Guide

Create `.claude/SYNTEK-GUIDE.md` with the complete plugin documentation (source: `$SYNTEK_DIR/examples/setup/SYNTEK-GUIDE-TEMPLATE.md`).

### Step 6: Container Setup

Based on the stack, set up containers:

| Stack          | Container      | Action                      |
| -------------- | -------------- | --------------------------- |
| TALL           | DDEV           | Create `.ddev/config.yaml`  |
| Django         | Docker Compose | Create `docker-compose.yml` |
| React          | Docker         | Create `Dockerfile`         |
| Mobile         | Docker         | Create `Dockerfile`         |
| Shared Library | Docker         | Create `Dockerfile`         |

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
| File                          | Purpose                           |
| ----------------------------- | --------------------------------- |
| `.claude/CLAUDE.md`              | Project context for Claude agents  |
| `.claude/CODING-PRINCIPLES.md`     | Rob Pike + Linus Torvalds rules        |
| `.claude/TESTING.md`               | Testing guide for this stack           |
| `.claude/SECURITY.md`              | Security patterns and checklist        |
| `.claude/ACCESSIBILITY.md`         | WCAG 2.2 AA and ARIA patterns          |
| `.claude/API-DESIGN.md`            | REST and GraphQL conventions           |
| `.claude/ARCHITECTURE-PATTERNS.md` | Service layer and project structure    |
| `.claude/DATA-STRUCTURES.md`       | Domain modelling and schema design     |
| `.claude/PERFORMANCE.md`           | Query optimisation and caching         |
| `.claude/DEVELOPMENT.md`           | Development workflow and tasks         |
| `.claude/SEO-CHECKLIST.md`         | SEO & AI discoverability checklist     |
| `.claude/settings.local.json`    | Claude Code settings               |
| `.claude/SYNTEK-GUIDE.md`        | Plugin usage guide                 |
| `.claude/commands/`              | Project-specific commands          |
| `.claude/plugins/*.py`           | Custom agent plugins               |
| `docs/METRICS/`                  | Self-learning system data          |

### Next Steps
1. Review and customise `.claude/CLAUDE.md`
2. Run `/agent:setup` to complete project setup
3. Use `/agent:plan` to plan your first feature
4. See `.claude/SYNTEK-GUIDE.md` for full command reference

### Quick Reference
- **Agents:** `/syntek-dev-suite:backend`, `/syntek-dev-suite:frontend`, `/syntek-dev-suite:plan`, etc.
- **Templates:** `/syntek-dev-suite:tall`, `/syntek-dev-suite:django`, etc.
- **Skills:** Loaded automatically based on stack
```

### Step 10: Offer Workflow Scaffolding

After displaying the Step 9 summary, ask the user:

```
Would you like to scaffold the three-layer workflow structure now?

This adds:
- `code/`, `how-to/`, and `project-management/` domain folders
- Numbered workflow folders with CONTEXT.md, STEPS.md, and CHECKLIST.md
- An updated `.claude/CLAUDE.md` with routing logic, MCP server registration, and model selection rules
- `GAPS.md` at the project root for any missing workflow files

1. **Yes** → Run `/syntek-dev-suite:scaffold new`
2. **No**  → Skip (run `/syntek-dev-suite:scaffold` manually later)
```

If the user selects **Yes**, spawn the scaffold agent:

```
/syntek-dev-suite:scaffold new
```

The scaffold agent reads the `.claude/CLAUDE.md` just created by init to extract the project name and stack — no additional input is needed.

---

## User's Request

$ARGUMENTS
