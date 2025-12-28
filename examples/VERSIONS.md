# Framework Versions

## Overview

This file tracks the framework and language versions used in our code examples. Agents should reference this file and compare against project versions before providing code examples.

**Last Updated:** 24/12/2025

## Supported Stacks

This project supports four primary stacks plus a shared UI library. Agents should detect which stack is in use before providing examples.

| Stack                  | Backend                                   | Frontend                              | Database        | Styling                           |
| ---------------------- | ----------------------------------------- | ------------------------------------- | --------------- | --------------------------------- |
| **TALL**               | Laravel 12.x + PHP 8.4                    | Alpine.js 3.x + Livewire 3.x          | MariaDB 12.x    | Tailwind CSS 4.x                  |
| **Django/Wagtail**     | Django 6.x + Python 3.14 + Wagtail 7.x    | GraphQL (Strawberry 16.x)             | PostgreSQL 18.x | Tailwind CSS 4.x                  |
| **React/TS (Next.js)** | Node.js 24.x + Next.js 16.x + GraphQL API | React 19.x + TypeScript 5.9           | PostgreSQL 18.x | Tailwind CSS 4.x                  |
| **React Native**       | GraphQL API consumer                      | React Native 0.83.x + TypeScript 5.9  | N/A             | NativeWind 4.x (stable)           |
| **Shared-Lib**         | N/A (NPM package)                         | TypeScript 5.9 + tsup + Storybook 8.x | N/A             | Tailwind CSS 4.x + NativeWind 4.x |

---

## Table of Contents

- [Overview](#overview)
- [Supported Stacks](#supported-stacks)
- [Table of Contents](#table-of-contents)
- [Current Versions Used in Examples](#current-versions-used-in-examples)
  - [Languages](#languages)
  - [Backend Frameworks](#backend-frameworks)
  - [Frontend Frameworks](#frontend-frameworks)
  - [CSS/Styling](#cssstyling)
  - [Databases](#databases)
  - [ORMs and Query Builders](#orms-and-query-builders)
- [Tailwind CSS Version Detection](#tailwind-css-version-detection)
  - [Tailwind 4.x (Current Default)](#tailwind-4x-current-default)
  - [Tailwind v3 (Legacy)](#tailwind-v3-legacy)
  - [Version Detection Workflow](#version-detection-workflow)
- [How Agents Should Use This File](#how-agents-should-use-this-file)
  - [1. Read Project Version Files](#1-read-project-version-files)
  - [2. Detect Tailwind Version](#2-detect-tailwind-version)
  - [3. Search Online for Latest Versions](#3-search-online-for-latest-versions)
  - [4. Compare and Report](#4-compare-and-report)
  - [5. Adapt Examples](#5-adapt-examples)
- [Version Checking Workflow](#version-checking-workflow)
- [Update Policy](#update-policy)
- [Sources](#sources)

---

## Current Versions Used in Examples

### Languages

| Language   | Example Version | EOL Date | Notes          |
| ---------- | --------------- | -------- | -------------- |
| PHP        | 8.4             | Nov 2028 | Active support |
| Python     | 3.14            | Oct 2029 | Active support |
| TypeScript | 5.9             | N/A      | Evergreen      |
| Node.js    | 24 LTS          | Apr 2028 | Active LTS     |

### Backend Frameworks

| Framework          | Example Version | Min Secure Version | Notes                                        |
| ------------------ | --------------- | ------------------ | -------------------------------------------- |
| Laravel            | 12.x            | 11.x               | Released Feb 2025                            |
| Django             | 6.x             | 5.2 LTS            | Released Dec 2025, supports Python 3.12-3.14 |
| Wagtail            | 7.x LTS         | 6.4                | LTS with preliminary Django 6 support        |
| Next.js            | 16.x            | 15.x               | Released Dec 2025, ships with React 19.2     |
| Strawberry GraphQL | 0.260+          | 0.220+             | Python GraphQL library                       |

### Frontend Frameworks

| Framework    | Example Version | Notes                             |
| ------------ | --------------- | --------------------------------- |
| React        | 19.x            | Stable, shipped with Next.js 16   |
| React Native | 0.83.x          | Released Dec 2025 with React 19.2 |
| Alpine.js    | 3.x             | Stable                            |
| Livewire     | 3.x             | With Laravel 11+                  |
| Storybook    | 8.x             | Component documentation           |

### CSS/Styling

| Framework    | Example Version | Notes                                         |
| ------------ | --------------- | --------------------------------------------- |
| Tailwind CSS | 4.x             | **Current default** - CSS-first configuration |
| NativeWind   | 4.x (stable)    | For React Native (v5 is pre-release)          |

### Databases

| Database   | Example Version | Stack            | Notes             |
| ---------- | --------------- | ---------------- | ----------------- |
| MariaDB    | 12.x            | TALL             | GA Aug 2025       |
| PostgreSQL | 18.x            | Django, React/TS | Released Sep 2025 |

### ORMs and Query Builders

| Tool       | Example Version | Framework | Notes                        |
| ---------- | --------------- | --------- | ---------------------------- |
| Eloquent   | 12.x            | Laravel   | Bundled with Laravel         |
| Django ORM | 6.x             | Django    | Bundled with Django          |
| Prisma     | 6.x             | Node.js   | Check for updates frequently |
| TypeORM    | 0.3.x           | Node.js   | Check for security updates   |

---

## Tailwind CSS Version Detection

**CRITICAL:** Agents MUST detect the Tailwind version before providing CSS examples.

### Tailwind 4.x (Current Default)

**Detection:** Look for `@theme` directive in CSS files (e.g., `app.css`, `globals.css`)

```css
/* Tailwind 4.x - CSS-first configuration */
@import "tailwindcss";

@theme {
  --color-brand-primary: #3b82f6;
  --color-brand-secondary: #1e40af;
  --font-family-display: "Inter", sans-serif;
}
```

**Characteristics:**
- Configuration in CSS using `@theme { }` blocks
- No `tailwind.config.js` required (optional for plugins)
- CSS custom properties for design tokens
- Import via `@import "tailwindcss";`

### Tailwind v3 (Legacy)

**Detection:** Look for `tailwind.config.js` or `tailwind.config.ts`

```javascript
// Tailwind v3 - JavaScript configuration
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          primary: '#3b82f6',
          secondary: '#1e40af',
        }
      }
    }
  }
}
```

**Characteristics:**
- Configuration in JavaScript/TypeScript
- `tailwind.config.js` or `tailwind.config.ts` required
- Theme extension via `theme.extend`
- Directives: `@tailwind base;`, `@tailwind components;`, `@tailwind utilities;`

### Version Detection Workflow

```
1. Check for CSS files with @theme directive → Tailwind 4.x
2. Check for tailwind.config.js/ts → Tailwind v3
3. Check package.json for tailwindcss version
4. Default to v4 if creating new configuration
```

---

## How Agents Should Use This File

### 1. Read Project Version Files

Check these files to determine the project's actual versions:

| File               | Stack         | What to Check                              |
| ------------------ | ------------- | ------------------------------------------ |
| `composer.json`    | TALL/Laravel  | `require.php`, `require.laravel/framework` |
| `requirements.txt` | Django        | `Django==`, `strawberry-graphql==`         |
| `pyproject.toml`   | Django        | `[project.dependencies]` section           |
| `package.json`     | Node.js/React | `dependencies`, `devDependencies`          |

### 2. Detect Tailwind Version

```
1. Search for @theme in CSS files → Tailwind 4.x
2. Check for tailwind.config.js/ts → Tailwind v3
3. Read package.json tailwindcss version
```

### 3. Search Online for Latest Versions

Use WebSearch to check:
- "[framework] latest stable version 2025"
- "[framework] security vulnerabilities 2025"
- "[framework] changelog" for breaking changes

### 4. Compare and Report

Report any significant differences to the user:

```
Your project uses Laravel 11.x (examples: 12.x, latest: 12.x, secure: 11.x+)
- Examples will work but some syntax may differ
- Consider upgrading to Laravel 12 for latest features

Your project uses Tailwind v3 (examples: 4.x)
- Will adapt examples to use tailwind.config.js pattern
```

### 5. Adapt Examples

When project version differs from example version:
- Note syntax differences in your response
- Adapt code snippets to match project version
- Warn about deprecated methods if applicable
- Convert Tailwind 4.x examples to v3 syntax if needed

---

## Version Checking Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Read project files (composer.json, package.json, etc.)  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Detect Tailwind version (check for @theme or config.js) │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Read examples/VERSIONS.md for example versions          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. WebSearch for latest stable + secure versions           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Compare: Project vs Examples vs Latest                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Report differences and adapt examples as needed         │
└─────────────────────────────────────────────────────────────┘
```

---

## Update Policy

Examples in this repository are updated:
- When a new major version of a framework is released
- When security vulnerabilities affect example code
- When syntax changes break compatibility

To request an update, note in your conversation that examples need updating for version X.

---

## Sources

Version information gathered from:
- [Laravel 12 Release Notes](https://laravel.com/docs/12.x/releases)
- [Django 6.0 Release Notes](https://docs.djangoproject.com/en/dev/releases/6.0/)
- [Wagtail 7.2 Release Notes](https://docs.wagtail.org/en/stable/releases/7.2.html)
- [Next.js 16 Blog](https://nextjs.org/blog/next-16)
- [React Native 0.83 Blog](https://reactnative.dev/blog/2025/12/10/react-native-0.83)
- [PostgreSQL 18 Release](https://www.postgresql.org/about/news/postgresql-18-released-3142/)
- [MariaDB 12 Release Notes](https://mariadb.com/docs/release-notes/community-server/release-notes-mariadb-12.0-rolling-releases/what-is-mariadb-120)
- [NativeWind v5 Installation](https://www.nativewind.dev/v5/getting-started/installation)
