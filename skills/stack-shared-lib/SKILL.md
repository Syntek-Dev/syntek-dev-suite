# Stack: Shared NPM Package

This skill defines shared library configuration for React Web and React Native projects.

---

## Table of Contents

- [Stack: Shared NPM Package](#stack-shared-npm-package)
  - [Table of Contents](#table-of-contents)
  - [Architecture](#architecture)
  - [Commands](#commands)
  - [Critical Rules](#critical-rules)
    - [Platform Agnostic Code](#platform-agnostic-code)
    - [Styling for Shared UI](#styling-for-shared-ui)
  - [Package.json Configuration](#packagejson-configuration)
  - [Versioning (Semantic Versioning)](#versioning-semantic-versioning)
  - [Development Workflow](#development-workflow)
  - [File Structure](#file-structure)
  - [Testing](#testing)

---

## Architecture

| Layer | Technology |
|-------|------------|
| **Context** | Shared Logic/UI for React (Web) and React Native (Mobile) |
| **Environment** | Node.js (Host machine or Docker) |
| **Build Tool** | tsup / Rollup (ESM & CJS output) |
| **Language** | TypeScript (Strict mode) |
| **Testing** | Vitest or Jest |

---

## Commands

| Task | Command |
|------|---------|
| Build package | `npm run build` |
| Watch mode | `npm run watch` |
| Run tests | `npm test` |
| Link locally | `npm link` or `yalc push` |
| Lint | `npm run lint` |
| Publish | `npm publish` |

---

## Critical Rules

### Platform Agnostic Code

**CRITICAL:** Shared code must work on BOTH web and mobile.

| Avoid | Alternative |
|-------|-------------|
| `window`, `document` | Platform detection or conditional imports |
| `fs`, `path` (Node APIs) | Only in build tools, not runtime code |
| `localStorage` | Abstract to platform-specific adapters |
| `fetch` (browser-specific) | Use cross-platform HTTP client |

```tsx
// BAD - Will crash on React Native
const width = window.innerWidth;

// GOOD - Platform-safe
import { Platform, Dimensions } from 'react-native';

const width = Platform.OS === 'web'
  ? window.innerWidth
  : Dimensions.get('window').width;
```

### Styling for Shared UI

- Use **NativeWind** compatible patterns
- Or use primitive props (no Tailwind classes in shared code)
- Let consuming apps handle final styling

```tsx
// Option 1: Primitive props
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size: 'sm' | 'md' | 'lg';
}

// Option 2: Style prop
interface CardProps {
  style?: ViewStyle | CSSProperties;
}
```

---

## Package.json Configuration

**CRITICAL:** Ensure correct exports configuration.

```json
{
  "name": "@company/shared",
  "version": "1.0.0",
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs",
      "types": "./dist/index.d.ts"
    }
  },
  "files": ["dist"],
  "sideEffects": false
}
```

---

## Versioning (Semantic Versioning)

| Version Bump | When to Use |
|--------------|-------------|
| **Major** (X.0.0) | Breaking changes |
| **Minor** (0.X.0) | New features (backwards compatible) |
| **Patch** (0.0.X) | Bug fixes (backwards compatible) |

---

## Development Workflow

```
1. Make changes to shared lib
       ↓
2. Run `npm run build`
       ↓
3. Link to consuming app:
   - `npm link` (in shared lib)
   - `npm link @company/shared` (in consuming app)
   OR
   - `yalc push` (in shared lib)
   - `yalc add @company/shared` (in consuming app)
       ↓
4. Test in consuming app
       ↓
5. Bump version in package.json
       ↓
6. Commit and publish
```

---

## File Structure

```
packages/shared/
├── src/
│   ├── index.ts            # Main exports
│   ├── components/
│   │   ├── Button.tsx
│   │   └── Card.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   └── useFetch.ts
│   ├── utils/
│   │   ├── formatters.ts
│   │   └── validators.ts
│   └── types/
│       └── index.ts
├── dist/                   # Built output (gitignored)
├── package.json
├── tsconfig.json
└── tsup.config.ts          # Or rollup.config.js

docs/
└── METRICS/            # Self-learning system (see global-workflow skill)
    ├── README.md
    ├── config.json
    ├── runs/
    ├── feedback/
    └── optimisations/
```

---

## Testing

```tsx
import { describe, it, expect } from 'vitest';
import { formatCurrency } from './formatters';

describe('formatCurrency', () => {
  it('formats GBP correctly', () => {
    expect(formatCurrency(1234.56, 'GBP')).toBe('£1,234.56');
  });

  it('handles zero', () => {
    expect(formatCurrency(0, 'GBP')).toBe('£0.00');
  });
});
```
