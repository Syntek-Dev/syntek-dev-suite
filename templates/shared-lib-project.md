# Shared Design System & Components Template

Use this template to set up a shared NPM package that provides **components, typography, fonts, colours, and design tokens** for both React Web (`stack-react`) and React Native Mobile (`stack-mobile`) projects.

**Template Repository:** `Syntek-Studio/ui_design_template`

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Template Repository](#template-repository)
- [Project CLAUDE.md](#project-claudemd)
- [Directory Structure](#directory-structure)
- [Environment-Specific Command Files](#environment-specific-command-files)
- [Package.json Configuration](#packagejson-configuration)
- [Tokens Structure](#tokens-structure)
- [Development Workflow](#development-workflow)
- [GitHub Template Repository Updates](#github-template-repository-updates)

---

## Architecture Overview

This is the **shared design system** that provides:
- **UI Components** - Button, Card, Modal, Input, etc. (separate web/mobile implementations)
- **Typography** - Heading, Text, Label components
- **Colour Palette** - Design tokens for brand colours
- **Fonts** - Pre-configured font families
- **Design Tokens** - Spacing, breakpoints, shadows

```
┌─────────────────────────────────────────────────────────────────┐
│                        THIS PROJECT                              │
│                   Shared Design System                           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  src/                                                      │  │
│  │  ├── web/components/      → React Web components          │  │
│  │  ├── mobile/components/   → React Native components       │  │
│  │  ├── tokens/              → Shared design tokens          │  │
│  │  ├── tailwind.css         → Tailwind v4 styles            │  │
│  │  └── index.ts             → Main exports                  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              │ imports
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│    React Web            │     │    React Native Mobile   │
│   (stack-react)         │     │    (stack-mobile)        │
│                         │     │                          │
│  import { Button }      │     │  import { Mobile }       │
│  from '@scope/ui'       │     │  from '@scope/ui'        │
│                         │     │  <Mobile.Button />       │
└─────────────────────────┘     └──────────────────────────┘
```

**Key Architectural Decisions:**
- **Separate web/mobile components:** Web uses `onClick` + `<button>`, Mobile uses `onPress` + `<Pressable>`
- **Shared tokens:** Colours, spacing, typography scales shared across platforms
- **Tailwind CSS v4 + NativeWind v4:** Modern styling with cross-platform compatibility
- **Storybook:** Visual component development and documentation

---

## Template Repository

**GitHub:** `Syntek-Studio/ui_design_template`

### Creating a New Project from Template

```bash
# Option 1: GitHub template (recommended)
gh repo create @scope/project-name --template Syntek-Studio/ui_design_template --private

# Option 2: Clone and re-initialise
git clone git@github.com:Syntek-Studio/ui_design_template.git project-name
cd project-name
rm -rf .git
git init
git remote add origin git@github.com:Your-Org/project-name.git
```

### Post-Clone Setup

After cloning from template, the setup agent will:
1. **Detect the stack** from README.md (`@syntek/ui` → `stack-shared-lib`)
2. **Ask for project name** (e.g., `@company/design-system`)
3. **Update package.json** with new name and scope
4. **Update .claude/CLAUDE.md** with project-specific details
5. **Verify git remote** matches project name

---

## Project CLAUDE.md

Create this file at `.claude/CLAUDE.md` in the project root:

```markdown
# Project: [Insert Package Name]

## Stack Overview

| Component | Technology |
|-----------|------------|
| **Type** | Shared Design System (NPM Package) |
| **Purpose** | Components, typography, fonts, colours for web and mobile |
| **Language** | TypeScript 5.x (Strict mode) |
| **Build Tool** | tsup (ESM & CJS output) |
| **Styling** | Tailwind CSS v4 + NativeWind v4 |
| **Storybook** | Storybook 8.x (web + mobile) |
| **Testing** | Vitest |
| **Environment** | Node.js (NO Docker) |

---

## Skill Targets

- **Stack Skill:** `stack-shared-lib`
- **Global Skill:** `global-workflow`

---

## Consumer Projects

| Project | Stack | Usage |
|---------|-------|-------|
| React Web | `stack-react` | `import { Button } from '@scope/ui'` |
| React Native Mobile | `stack-mobile` | `import { Mobile } from '@scope/ui'` then `<Mobile.Button />` |

Both consumers use the same design tokens to maintain visual consistency.

---

## Key Locations

| Directory | Purpose |
|-----------|---------|
| `src/index.ts` | Main exports |
| `src/web/components/` | React Web components (onClick, button) |
| `src/mobile/components/` | React Native components (onPress, Pressable) |
| `src/tokens/` | Shared design tokens (colours, spacing, typography) |
| `src/tokens/colours.ts` | Brand colour palette |
| `src/tokens/spacing.ts` | Spacing scale (4, 8, 12, 16, etc.) |
| `src/tokens/typography.ts` | Font sizes, weights, line heights |
| `src/tokens/breakpoints.ts` | Responsive breakpoints |
| `src/tokens/shadows.ts` | Shadow definitions |
| `src/tailwind.css` | Tailwind v4 styles |
| `.storybook-web/` | Web Storybook configuration |
| `.storybook-mobile/` | Mobile Storybook configuration |
| `dist/` | Built output (gitignored) |

---

## Development Commands

```bash
# Build
npm run build                        # Build package with tsup
npm run dev                          # Watch mode (rebuild on change)

# Type Checking
npm run type-check                   # TypeScript check

# Linting
npm run lint                         # ESLint
npm run lint:fix                     # Fix issues

# Storybook - Web
npm run storybook:web                # Start web Storybook (port 6006)
npm run storybook:web:build          # Build static web Storybook

# Storybook - Mobile
npm run storybook:mobile             # Start mobile Storybook (runs in RN app)
npm run storybook:mobile:build       # Build mobile Storybook

# Local Development
npm link                             # Link package globally
npm link @scope/ui                   # Link in consuming app
# OR
yalc push                            # Push to local registry
yalc add @scope/ui                   # Add in consuming app

# Publishing
npm version patch                    # Bump patch version
npm version minor                    # Bump minor version
npm version major                    # Bump major version
npm publish                          # Publish to npm
```

---

## Component Architecture

### Separate Web and Mobile Components

Components are separated by platform due to fundamental API differences:

```
src/
├── web/components/
│   └── Button/
│       ├── Button.tsx           # Uses onClick, <button>
│       ├── Button.stories.tsx   # Web Storybook stories
│       └── index.ts             # Re-export
├── mobile/components/
│   └── Button/
│       ├── Button.tsx           # Uses onPress, <Pressable>
│       ├── Button.stories.tsx   # Mobile Storybook stories
│       └── index.ts             # Re-export
└── tokens/                      # Shared across platforms
    ├── colours.ts
    ├── spacing.ts
    └── typography.ts
```

### Import Patterns

```tsx
// Web (default exports)
import { Button, Card } from '@scope/ui';

// Mobile (namespaced exports)
import { Mobile } from '@scope/ui';
<Mobile.Button />
<Mobile.Card />
```

### Platform Differences

| Feature | Web | Mobile |
|---------|-----|--------|
| Event handler | `onClick` | `onPress` |
| Button element | `<button>` | `<Pressable>` |
| Hover states | `:hover` CSS | Not available |
| Text wrapper | Optional | Required `<Text>` |

---

## Styling

### Tailwind CSS v4 + NativeWind v4

This project uses the latest Tailwind CSS v4 with NativeWind v4 for React Native compatibility.

**PostCSS Configuration:**
```javascript
// postcss.config.mjs
export default {
  plugins: {
    '@tailwindcss/postcss': {}
  }
}
```

**Path Aliases:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

---

## Storybook

### Web Storybook

Located in `.storybook-web/`, configured for React with Webpack 5.

```bash
npm run storybook:web         # http://localhost:6006
```

### Mobile Storybook

Located in `.storybook-mobile/`, runs inside your React Native app.

```bash
npm run storybook:mobile      # Launches in RN simulator
```

**Note:** Mobile Storybook requires additional setup in the consuming React Native app. See [Storybook for React Native](https://storybook.js.org/docs/react/get-started/install#react-native) for configuration.

---

## Code Conventions

### Component Structure

Each component folder contains:
- `ComponentName.tsx` - Main component
- `ComponentName.stories.tsx` - Storybook stories
- `index.ts` - Re-export

### Commit Convention

Use Conventional Commits:
```
type(scope): description

feat(button): add loading state
fix(card): correct border radius
docs(readme): update installation guide
```

### Branch Strategy

```
feature/name → testing → dev → staging → main
```

### PR Titles

Format: `user-story-number/feature-name`

---

## Versioning

| Version Bump | When to Use |
|--------------|-------------|
| **Major** (X.0.0) | Breaking changes to component APIs |
| **Minor** (0.X.0) | New components or features |
| **Patch** (0.0.X) | Bug fixes, style tweaks |
```

---

## Settings File

Create this file at `.claude/settings.local.json`:

```json
{
  "language": "typescript",
  "framework": "shared-library",
  "stackType": "stack-shared-lib",
  "locale": "en_GB",
  "timezone": "Europe/London",
  "permissions": {
    "allow": [
      "Read(**)",
      "Edit(**)",
      "Write(**)",
      "Bash(npm:*)",
      "Bash(npx:*)",
      "Bash(yalc:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo:*)",
      "Bash(npm publish:*)"
    ]
  },
  "environment": {
    "containerType": "none",
    "platform": "node",
    "envFiles": []
  },
  "testing": {
    "framework": "vitest",
    "command": "npm test",
    "coverageCommand": "npm test -- --coverage"
  },
  "linting": {
    "eslint": "npm run lint",
    "eslintFix": "npm run lint:fix",
    "typecheck": "npm run type-check"
  },
  "build": {
    "command": "npm run build",
    "watchCommand": "npm run dev",
    "outputDir": "dist"
  },
  "storybook": {
    "web": {
      "command": "npm run storybook:web",
      "buildCommand": "npm run storybook:web:build",
      "port": 6006
    },
    "mobile": {
      "command": "npm run storybook:mobile",
      "buildCommand": "npm run storybook:mobile:build"
    }
  }
}
```

---

## Commands

Create these files in `.claude/commands/`:

**CRITICAL:** Commands should reference the root-level environment scripts (`dev.sh`, `test.sh`, `staging.sh`, `production.sh`) to ensure consistency and avoid duplication.

### .claude/commands/dev.md

```markdown
---
description: Start development mode with watch
usage: /dev
---

Start the shared library in development mode with file watching.

**Run:** `./dev.sh`

This script will:
1. Install dependencies if needed
2. Start tsup in watch mode

In consuming app, run `npm link @scope/ui` or `yalc add @scope/ui`

**Workflow:**
```
Make changes → Auto-rebuild → Test in consuming app
```

$ARGUMENTS
```

### .claude/commands/test.md

```markdown
---
description: Run the test suite
usage: /test
---

Run the project's test suite.

**Run:** `./test.sh`

This script will:
1. Run the test suite
2. Run type checking
3. Run linting

**Additional Commands:**
- Watch mode: `npm test -- --watch`
- With coverage: `npm test -- --coverage`
- Single file: `npm test -- $ARGUMENTS`

$ARGUMENTS
```

### .claude/commands/staging.md

```markdown
---
description: Build for staging (prerelease)
usage: /staging
---

Build and prepare a staging/prerelease version.

**Run:** `./staging.sh`

This script will:
1. Run the test suite first
2. Build the package
3. Bump prerelease version

After running, use `npm publish --tag staging` to publish the prerelease.

$ARGUMENTS
```

### .claude/commands/production.md

```markdown
---
description: Build for production
usage: /production
---

Build and prepare for production release.

**Run:** `./production.sh`

This script will:
1. Prompt for confirmation (safety check)
2. Run the test suite first
3. Build the package

After running, use `npm version <patch|minor|major>` then `npm publish` to release.

$ARGUMENTS
```

### .claude/commands/storybook.md

```markdown
---
description: Start Storybook for component development
usage: /storybook [web|mobile]
---

Start Storybook for visual component development.

**Commands:**
- Web Storybook: `npm run storybook:web` (http://localhost:6006)
- Mobile Storybook: `npm run storybook:mobile` (runs in RN simulator)

**Build Commands:**
- Build web: `npm run storybook:web:build`
- Build mobile: `npm run storybook:mobile:build`

$ARGUMENTS
```

### .claude/commands/build.md

```markdown
---
description: Build the package for distribution
usage: /build
---

Build the shared package for distribution.

**Run:** `./production.sh` (recommended for full build with tests)

Or manually:
1. Run `npm run build`
2. Output is generated in `dist/` directory
3. Verify exports with `npm pack --dry-run`

**Output:**
- `dist/index.mjs` - ESM module
- `dist/index.js` - CommonJS module
- `dist/index.d.ts` - TypeScript declarations

$ARGUMENTS
```

### .claude/commands/link.md

```markdown
---
description: Link package for local development
usage: /link
---

Link the package for local development in consuming apps.

**Using npm link:**
```bash
# In this package:
npm link

# In consuming app:
npm link @scope/ui
```

**Using yalc (recommended):**
```bash
# In this package:
yalc push

# In consuming app:
yalc add @scope/ui
```

**After changes:**
```bash
# Rebuild and push
npm run build && yalc push
```

$ARGUMENTS
```

### .claude/commands/version.md

```markdown
---
description: Bump package version
usage: /version <patch|minor|major>
---

Bump the package version following semantic versioning.

**Commands:**
- Patch (bug fixes): `npm version patch`
- Minor (new features): `npm version minor`
- Major (breaking changes): `npm version major`

**Workflow:**
1. Bump version: `npm version $ARGUMENTS`
2. Update CHANGELOG.md
3. Commit and tag: `git push && git push --tags`
4. Publish: `npm publish`

$ARGUMENTS
```

### .claude/commands/lint.md

```markdown
---
description: Run linting and type checks
usage: /lint
---

Run all code quality checks.

**Commands:**
- ESLint: `npm run lint`
- Fix ESLint: `npm run lint:fix`
- TypeScript: `npm run type-check`

**All checks:**
```bash
npm run lint && npm run type-check
```

$ARGUMENTS
```

### .claude/commands/make-component.md

```markdown
---
description: Create a new component for web and mobile
usage: /make-component <ComponentName>
---

Create a new component with both web and mobile implementations.

**Creates:**
```
src/web/components/$ARGUMENTS/
├── $ARGUMENTS.tsx
├── $ARGUMENTS.stories.tsx
└── index.ts

src/mobile/components/$ARGUMENTS/
├── $ARGUMENTS.tsx
├── $ARGUMENTS.stories.tsx
└── index.ts
```

**Remember to:**
1. Export from `src/web/components/index.ts`
2. Export from `src/mobile/components/index.ts`
3. Add to main exports in `src/index.ts`

**Platform Differences:**
- Web: Use `onClick`, `<button>`, CSS `:hover`
- Mobile: Use `onPress`, `<Pressable>`, no hover states

$ARGUMENTS
```

### .claude/commands/make-token.md

```markdown
---
description: Create a new design token file
usage: /make-token <tokenName>
---

Create a new shared design token file.

**Creates:**
- `src/tokens/$ARGUMENTS.ts` - Token definitions

**Token Types:**
- `colours.ts` - Brand colour palette
- `spacing.ts` - Spacing scale (4, 8, 12, 16, etc.)
- `typography.ts` - Font sizes, weights, line heights
- `breakpoints.ts` - Responsive breakpoints
- `shadows.ts` - Shadow definitions
- `borders.ts` - Border radii, widths

**Remember to:**
1. Export from `src/tokens/index.ts`
2. Add to main exports in `src/index.ts`

$ARGUMENTS
```

---

## Directory Structure

```
[project-root]/
├── .claude/
│   ├── CLAUDE.md
│   ├── settings.local.json
│   └── commands/
│       ├── dev.md              # References ./dev.sh
│       ├── test.md             # References ./test.sh
│       ├── staging.md          # References ./staging.sh
│       ├── production.md       # References ./production.sh
│       ├── storybook.md
│       ├── build.md
│       ├── link.md
│       ├── version.md
│       ├── lint.md
│       ├── make-component.md
│       └── make-token.md
├── .storybook-web/                 # Web Storybook configuration
│   ├── main.ts
│   └── preview.ts
├── .storybook-mobile/              # Mobile Storybook configuration
│   ├── main.ts
│   └── preview.ts
├── src/
│   ├── index.ts                    # Main exports
│   ├── tailwind.css                # Tailwind v4 styles
│   ├── web/
│   │   └── components/
│   │       ├── index.ts
│   │       ├── Button/
│   │       │   ├── Button.tsx
│   │       │   ├── Button.stories.tsx
│   │       │   └── index.ts
│   │       ├── Card/
│   │       ├── Modal/
│   │       └── Input/
│   ├── mobile/
│   │   └── components/
│   │       ├── index.ts
│   │       ├── Button/
│   │       │   ├── Button.tsx
│   │       │   ├── Button.stories.tsx
│   │       │   └── index.ts
│   │       ├── Card/
│   │       ├── Modal/
│   │       └── Input/
│   └── tokens/                     # Shared design tokens
│       ├── index.ts
│       ├── colours.ts              # Brand colour palette
│       ├── spacing.ts              # Spacing scale (4, 8, 12, 16, etc.)
│       ├── typography.ts           # Font sizes, weights, line heights
│       ├── breakpoints.ts          # Responsive breakpoints
│       ├── shadows.ts              # Shadow definitions
│       └── borders.ts              # Border radii, widths
├── dist/                           # Built output (gitignored)
│   ├── index.mjs
│   ├── index.js
│   └── index.d.ts
├── docs/
│   ├── API/
│   ├── ARCHITECTURE/
│   ├── BUGS/
│   ├── DEVOPS/
│   ├── GUIDES/
│   ├── METRICS/               # Self-learning system data
│   │   ├── README.md
│   │   ├── config.json
│   │   ├── runs/
│   │   ├── feedback/
│   │   ├── aggregates/
│   │   ├── variants/
│   │   └── optimisations/
│   ├── PLANS/
│   ├── QA/
│   ├── STORIES/
│   └── TESTS/
├── package.json
├── package-lock.json
├── tsconfig.json
├── tsup.config.ts
├── postcss.config.mjs
├── .babelrc
├── eslint.config.js
├── .gitignore
├── CHANGELOG.md
├── README.md
├── dev.sh
├── test.sh
├── staging.sh
└── production.sh
```

---

## Environment-Specific Command Files

**CRITICAL:** All shared library projects MUST have environment-specific command files for managing different environments.

**Note:** Shared library projects run in Node.js (NOT in Docker), and focus on building/publishing rather than deployment.

### Required Root-Level Scripts

| File | Purpose | Environment |
|------|---------|-------------|
| `dev.sh` | Start development with watch mode | Development |
| `test.sh` | Run test suite | Testing |
| `staging.sh` | Build and publish prerelease version | Staging |
| `production.sh` | Build and publish production version | Production |

### dev.sh

```bash
#!/bin/bash
set -e

echo "🚀 Starting development environment..."

# Install dependencies if needed
if [ ! -d "node_modules" ]; then
    npm install
fi

# Start in watch mode
echo "✅ Starting watch mode..."
echo "📖 Run 'npm run storybook:web' in another terminal for component development"
npm run dev
```

### test.sh

```bash
#!/bin/bash
set -e

echo "🧪 Running test suite..."

npm test

echo "🔍 Running type check..."
npm run type-check

echo "🧹 Running linter..."
npm run lint

echo "✅ All checks complete!"
```

### staging.sh

```bash
#!/bin/bash
set -e

echo "🚀 Building for staging (prerelease)..."

# Run tests first
./test.sh

# Build the package
npm run build

# Bump prerelease version
npm version prerelease --preid=staging

echo "✅ Staging build complete!"
echo "📦 Run 'npm publish --tag staging' to publish prerelease"
```

### production.sh

```bash
#!/bin/bash
set -e

echo "🚀 Building for production..."

read -p "⚠️  Are you sure you want to build for PRODUCTION? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "❌ Build cancelled."
    exit 1
fi

# Run tests first
./test.sh

# Build the package
npm run build

echo "✅ Production build complete!"
echo "📦 Run 'npm version <patch|minor|major>' then 'npm publish' to release"
```

### Command File Permissions

**CRITICAL:** All shell scripts MUST be executable:

```bash
chmod +x dev.sh test.sh staging.sh production.sh
```

---

## Package.json Configuration

**CRITICAL:** Ensure correct exports configuration. This matches the `Syntek-Studio/ui_design_template` structure.

```json
{
  "name": "@scope/ui",
  "version": "0.1.0",
  "private": false,
  "description": "A shared UI component library for React Web and React Native applications. Built with TypeScript, Tailwind CSS 4, and Nativewind 4.",
  "type": "module",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "require": "./dist/index.js",
      "import": "./dist/index.mjs"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch",
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "storybook:web": "storybook dev -p 6006 -c .storybook-web",
    "storybook:web:build": "storybook build -c .storybook-web -o storybook-static-web",
    "storybook:mobile": "storybook dev -p 6007 -c .storybook-mobile",
    "storybook:mobile:build": "storybook build -c .storybook-mobile -o storybook-static-mobile"
  },
  "peerDependencies": {
    "nativewind": "^4.0.0",
    "react": "^18 || ^19",
    "react-dom": "^18 || ^19",
    "react-native": ">=0.70.0",
    "react-native-reanimated": ">=3.0.0",
    "react-native-safe-area-context": ">=4.0.0",
    "react-native-web": ">=0.18.0"
  },
  "keywords": ["ui", "components", "react", "react-native", "tailwind", "nativewind", "design-system"],
  "author": "",
  "license": "ISC"
}
```

---

## Tokens Structure

The `src/tokens/` directory contains shared design tokens used by both web and mobile components.

### src/tokens/colours.ts

```typescript
/**
 * Brand colour palette
 * Use semantic names for colours
 */
export const colours = {
  // Brand
  primary: {
    50: '#eff6ff',
    100: '#dbeafe',
    200: '#bfdbfe',
    300: '#93c5fd',
    400: '#60a5fa',
    500: '#3b82f6',  // Default
    600: '#2563eb',
    700: '#1d4ed8',
    800: '#1e40af',
    900: '#1e3a8a',
  },
  secondary: {
    // ... secondary palette
  },

  // Semantic
  success: '#22c55e',
  warning: '#f59e0b',
  error: '#ef4444',
  info: '#3b82f6',

  // Neutral
  white: '#ffffff',
  black: '#000000',
  grey: {
    50: '#f9fafb',
    100: '#f3f4f6',
    200: '#e5e7eb',
    300: '#d1d5db',
    400: '#9ca3af',
    500: '#6b7280',
    600: '#4b5563',
    700: '#374151',
    800: '#1f2937',
    900: '#111827',
  },
} as const;

export type Colours = typeof colours;
```

### src/tokens/spacing.ts

```typescript
/**
 * Spacing scale (in pixels)
 * Based on 4px grid system
 */
export const spacing = {
  0: 0,
  px: 1,
  0.5: 2,
  1: 4,
  1.5: 6,
  2: 8,
  2.5: 10,
  3: 12,
  3.5: 14,
  4: 16,
  5: 20,
  6: 24,
  7: 28,
  8: 32,
  9: 36,
  10: 40,
  11: 44,
  12: 48,
  14: 56,
  16: 64,
  20: 80,
  24: 96,
  28: 112,
  32: 128,
  36: 144,
  40: 160,
  44: 176,
  48: 192,
  52: 208,
  56: 224,
  60: 240,
  64: 256,
  72: 288,
  80: 320,
  96: 384,
} as const;

export type Spacing = typeof spacing;
```

### src/tokens/typography.ts

```typescript
/**
 * Typography scale
 */
export const typography = {
  fontFamily: {
    sans: ['Inter', 'system-ui', 'sans-serif'],
    serif: ['Georgia', 'serif'],
    mono: ['Fira Code', 'monospace'],
  },
  fontSize: {
    xs: { size: 12, lineHeight: 16 },
    sm: { size: 14, lineHeight: 20 },
    base: { size: 16, lineHeight: 24 },
    lg: { size: 18, lineHeight: 28 },
    xl: { size: 20, lineHeight: 28 },
    '2xl': { size: 24, lineHeight: 32 },
    '3xl': { size: 30, lineHeight: 36 },
    '4xl': { size: 36, lineHeight: 40 },
    '5xl': { size: 48, lineHeight: 1 },
    '6xl': { size: 60, lineHeight: 1 },
  },
  fontWeight: {
    thin: '100',
    extralight: '200',
    light: '300',
    normal: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
    extrabold: '800',
    black: '900',
  },
} as const;

export type Typography = typeof typography;
```

### src/tokens/breakpoints.ts

```typescript
/**
 * Responsive breakpoints (in pixels)
 */
export const breakpoints = {
  xs: 0,
  sm: 640,
  md: 768,
  lg: 1024,
  xl: 1280,
  '2xl': 1536,
} as const;

export type Breakpoints = typeof breakpoints;
```

### src/tokens/shadows.ts

```typescript
/**
 * Shadow definitions
 */
export const shadows = {
  none: 'none',
  sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
  base: '0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)',
  md: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
  lg: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
  xl: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
  '2xl': '0 25px 50px -12px rgb(0 0 0 / 0.25)',
  inner: 'inset 0 2px 4px 0 rgb(0 0 0 / 0.05)',
} as const;

export type Shadows = typeof shadows;
```

### src/tokens/borders.ts

```typescript
/**
 * Border definitions
 */
export const borders = {
  radius: {
    none: 0,
    sm: 2,
    base: 4,
    md: 6,
    lg: 8,
    xl: 12,
    '2xl': 16,
    '3xl': 24,
    full: 9999,
  },
  width: {
    0: 0,
    1: 1,
    2: 2,
    4: 4,
    8: 8,
  },
} as const;

export type Borders = typeof borders;
```

### src/tokens/index.ts

```typescript
export { colours, type Colours } from './colours';
export { spacing, type Spacing } from './spacing';
export { typography, type Typography } from './typography';
export { breakpoints, type Breakpoints } from './breakpoints';
export { shadows, type Shadows } from './shadows';
export { borders, type Borders } from './borders';
```

---

## Development Workflow

```
1. Make changes to shared lib
       ↓
2. Run `npm run dev` (watch mode) or `npm run build`
       ↓
3. View in Storybook:
   - `npm run storybook:web` (web components)
   - `npm run storybook:mobile` (mobile components)
       ↓
4. Link to consuming app:
   - `npm link` (in shared lib)
   - `npm link @scope/ui` (in consuming app)
   OR
   - `yalc push` (in shared lib)
   - `yalc add @scope/ui` (in consuming app)
       ↓
5. Test in consuming app
       ↓
6. Bump version in package.json
       ↓
7. Commit and publish
```

---

## GitHub Template Repository Updates

When the `Syntek-Studio/ui_design_template` repo is updated, ensure the following files exist in `.claude/`:

### Required Files

| File | Purpose |
|------|---------|
| `.claude/CLAUDE.md` | Project context for Claude Code |
| `.claude/settings.local.json` | Claude Code settings |
| `.claude/commands/dev.md` | Development command |
| `.claude/commands/storybook.md` | Storybook command |
| `.claude/commands/test.md` | Test command |
| `.claude/commands/build.md` | Build command |
| `.claude/commands/link.md` | Link command |
| `.claude/commands/version.md` | Version command |
| `.claude/commands/lint.md` | Lint command |
| `.claude/commands/make-component.md` | Component generator |
| `.claude/commands/make-token.md` | Token generator |

### Required in src/tokens/

| File | Purpose |
|------|---------|
| `src/tokens/index.ts` | Token exports |
| `src/tokens/colours.ts` | Brand colour palette |
| `src/tokens/spacing.ts` | Spacing scale |
| `src/tokens/typography.ts` | Font sizes, weights, line heights |
| `src/tokens/breakpoints.ts` | Responsive breakpoints |
| `src/tokens/shadows.ts` | Shadow definitions |
| `src/tokens/borders.ts` | Border radii, widths |
