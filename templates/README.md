# Templates

## Overview

This folder contains project templates that define the `CLAUDE.md` configuration for different technology stacks. Templates are used by the `/plugin:init` command or can be copied manually to initialise a new project.

Each template is a complete `CLAUDE.md` file with placeholders for project-specific information.

**Version:** 1.2.0

---

## Table of Contents

- [Overview](#overview)
- [Table of Contents](#table-of-contents)
- [Directory Tree](#directory-tree)
- [Available Templates](#available-templates)
- [What Templates Provide](#what-templates-provide)
  - [Stack Overview](#stack-overview)
  - [Skill Targets](#skill-targets)
  - [Environment](#environment)
  - [Development Standards](#development-standards)
  - [Placeholders](#placeholders)
- [Usage](#usage)
  - [Using /plugin:init (Recommended)](#using-plugininit-recommended)
  - [Manual Setup](#manual-setup)
  - [Template Selection Guide](#template-selection-guide)
- [Related Sections](#related-sections)

---

## Directory Tree

```
templates/
├── README.md                  # This file
├── django-project.md          # Django + Wagtail + PostgreSQL template
├── mobile-project.md          # React Native + Expo template
├── react-project.md           # React + Next.js + TypeScript template
├── shared-lib-project.md      # NPM shared library template
└── tall-project.md            # Laravel + Livewire + Tailwind template
```

---

## Available Templates

| Template           | File                    | Stack                                                     | Container      |
| ------------------ | ----------------------- | --------------------------------------------------------- | -------------- |
| **TALL Stack**     | `tall-project.md`       | Laravel 12.x, Livewire 3.x, Alpine.js, Tailwind 4.x       | DDEV           |
| **Django Stack**   | `django-project.md`     | Python 3.14, Django 6.x, Wagtail 7.x, PostgreSQL 18       | Docker Compose |
| **React Stack**    | `react-project.md`      | TypeScript 5.9, React 19.x, Next.js 16.x, Tailwind 4.x    | Docker         |
| **Mobile Stack**   | `mobile-project.md`     | TypeScript 5.9, React Native 0.83.x, Expo, NativeWind 4.x | Docker         |
| **Shared Library** | `shared-lib-project.md` | TypeScript 5.9, NPM package for web/mobile                | Docker         |

---

## What Templates Provide

Each template includes configuration for:

### Stack Overview
- Language and version
- Framework and version
- Database type
- Container platform

### Skill Targets
- Stack skill (e.g., `stack-tall`)
- Global skill (`global-workflow`)

### Environment
- Container commands
- Environment files
- Database configuration
- Locale and timezone

### Development Standards
- Coding conventions
- Testing frameworks
- Linting tools
- Documentation standards

### Placeholders

Templates contain placeholders that need to be replaced:

| Placeholder              | Replace With        |
| ------------------------ | ------------------- |
| `[Insert Project Name]`  | Your project name   |
| `[Insert Description]`   | Project description |
| `[Insert Database Name]` | Database name       |
| `[Insert Domain]`        | Development domain  |

---

## Usage

### Using /plugin:init (Recommended)

```bash
# Navigate to your project
cd ~/my-project

# Run init command
/plugin:init
```

The init command will:
1. Detect your project type
2. Copy the appropriate template
3. Set up container configuration
4. Copy the Syntek Guide

### Manual Setup

1. **Create the .claude directory:**

```bash
mkdir -p .claude
```

2. **Copy the appropriate template:**

```bash
# For TALL Stack
cp /path/to/claude-dev-team/templates/tall-project.md ./.claude/CLAUDE.md

# For Django
cp /path/to/claude-dev-team/templates/django-project.md ./.claude/CLAUDE.md

# For React
cp /path/to/claude-dev-team/templates/react-project.md ./.claude/CLAUDE.md

# For Mobile
cp /path/to/claude-dev-team/templates/mobile-project.md ./.claude/CLAUDE.md

# For Shared Library
cp /path/to/claude-dev-team/templates/shared-lib-project.md ./.claude/CLAUDE.md
```

3. **Copy the Syntek Guide:**

```bash
cp /path/to/claude-dev-team/examples/setup/SYNTEK-GUIDE-TEMPLATE.md ./.claude/SYNTEK-GUIDE.md
```

4. **Edit the template:**

Open `.claude/CLAUDE.md` and replace all placeholders with your project details.

### Template Selection Guide

| If Your Project Uses...               | Use Template            |
| ------------------------------------- | ----------------------- |
| Laravel, PHP, Livewire, DDEV          | `tall-project.md`       |
| Django, Python, Wagtail               | `django-project.md`     |
| React, Next.js, TypeScript            | `react-project.md`      |
| React Native, Expo                    | `mobile-project.md`     |
| NPM package shared between web/mobile | `shared-lib-project.md` |

---

## Related Sections

- [../skills/](../skills/) - Stack skills referenced by templates
- [../examples/setup/](../examples/setup/) - Setup examples and guides
- [../commands/init.md](../commands/init.md) - The init command that uses templates
- [../agents/setup.md](../agents/setup.md) - Setup agent that configures projects
