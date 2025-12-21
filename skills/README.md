# Skills

## Overview

This folder contains stack-specific skills that agents use to apply the correct patterns, conventions, and best practices for each technology stack. Skills are loaded automatically based on the `Skill Target` defined in the project's `CLAUDE.md` file.

Each skill is a `SKILL.md` file within a named folder that provides detailed guidance for agents working with that stack.

**Version:** 1.0.0

---

## Table of Contents

- [Overview](#overview)
- [Directory Tree](#directory-tree)
- [Available Skills](#available-skills)
- [How Skills Work](#how-skills-work)
- [Usage](#usage)
- [Related Sections](#related-sections)

---

## Directory Tree

```
skills/
├── README.md                  # This file
├── global-workflow/
│   └── SKILL.md               # British English, dates, Git, currency
├── stack-django/
│   └── SKILL.md               # Django, Wagtail, PostgreSQL, GraphQL
├── stack-mobile/
│   └── SKILL.md               # React Native, Expo, NativeWind 4.x
├── stack-react/
│   └── SKILL.md               # React, Next.js, TypeScript, Tailwind 4.x
├── stack-shared-lib/
│   └── SKILL.md               # NPM packages for web and mobile
└── stack-tall/
    └── SKILL.md               # Laravel, Livewire, Alpine, Tailwind 4.x
```

---

## Available Skills

### Stack Skills

| Skill | Folder | Technologies |
|-------|--------|--------------|
| **TALL Stack** | `stack-tall/` | Laravel 12.x, Livewire 3.x, Alpine.js 3.x, Tailwind CSS 4.x, MariaDB 12 |
| **Django Stack** | `stack-django/` | Python 3.14, Django 6.x, Wagtail 7.x, PostgreSQL 18, Strawberry GraphQL |
| **React Stack** | `stack-react/` | TypeScript 5.9, React 19.x, Next.js 16.x, Tailwind CSS 4.x, Node.js 24 |
| **Mobile Stack** | `stack-mobile/` | TypeScript 5.9, React Native 0.83.x, Expo, NativeWind 4.x |
| **Shared Library** | `stack-shared-lib/` | TypeScript 5.9, NPM package for web and mobile |

### Global Skill

The `global-workflow` skill is **always applied** regardless of stack. It provides:

| Setting | Value |
|---------|-------|
| **Language** | British English spelling |
| **Date Format** | DD/MM/YYYY |
| **Time Format** | 24-hour clock (14:30) |
| **Timezone** | Europe/London (GMT/BST) |
| **Currency** | GBP (£1,234.56) |
| **Git** | Commit message standards |
| **Documentation** | File header formats |

---

## How Skills Work

1. **Project Configuration**
   - Your project's `.claude/CLAUDE.md` specifies a `Skill Target`
   - Example: `Skill Target: stack-tall`

2. **Agent Execution**
   - When an agent runs, it loads the specified stack skill
   - The global workflow skill is always applied additionally

3. **Pattern Application**
   - Agents use the skill to determine:
     - Which frameworks and versions to use
     - Which coding patterns to follow
     - Which testing frameworks apply
     - Which container commands to run

### Skill Loading Example

```markdown
## Skill Targets (in CLAUDE.md)

- **Stack Skill:** stack-tall
- **Global Skill:** global-workflow
```

When `/agent:backend` runs, it loads:
1. `skills/stack-tall/SKILL.md` - Laravel patterns
2. `skills/global-workflow/SKILL.md` - British English, dates, currency

---

## Usage

### Setting the Skill Target

In your project's `.claude/CLAUDE.md`:

```markdown
## Skill Targets

- **Stack Skill:** stack-tall
- **Global Skill:** global-workflow
```

### Available Skill Targets

| Project Type | Skill Target |
|--------------|--------------|
| Laravel + Livewire + Alpine + Tailwind | `stack-tall` |
| Django + Wagtail + PostgreSQL | `stack-django` |
| React + Next.js + TypeScript | `stack-react` |
| React Native + Expo | `stack-mobile` |
| NPM shared library | `stack-shared-lib` |

### Creating a Custom Skill

1. Create a new folder: `skills/stack-mystack/`
2. Add a `SKILL.md` file with stack-specific guidance
3. Update your `CLAUDE.md` to use `Skill Target: stack-mystack`

---

## Related Sections

- [../agents/](../agents/) - Agents that apply these skills
- [../templates/](../templates/) - Project templates that set skill targets
- [../examples/](../examples/) - Code examples for each stack
- [../CLAUDE.md](../CLAUDE.md) - Global plugin settings
