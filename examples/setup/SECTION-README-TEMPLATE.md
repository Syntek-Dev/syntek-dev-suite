# Section README Template

## Overview

Every significant folder in a codebase should contain a `README.md` that explains what that section does and provides a tree layout of its contents. This allows agents and developers to quickly understand the purpose and structure of any folder.

---

## Table of Contents

- [Section README Template](#section-readme-template)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Template Structure](#template-structure)
  - [When to Create Section READMEs](#when-to-create-section-readmes)
  - [Examples](#examples)
    - [API Folder README](#api-folder-readme)
    - [Components Folder README](#components-folder-readme)
    - [Services Folder README](#services-folder-readme)
    - [Utils Folder README](#utils-folder-readme)
    - [Config Folder README](#config-folder-readme)
    - [Tests Folder README](#tests-folder-readme)
    - [Docs Folder README](#docs-folder-readme)
  - [Tree Generation](#tree-generation)
    - [Tree Characters](#tree-characters)
    - [Generating Trees Automatically](#generating-trees-automatically)


---

## Purpose

Section READMEs serve to:

1. **Provide Context** - Agents can read the README first to understand what a folder contains
2. **Document Structure** - The tree layout shows what files and subfolders exist
3. **Explain Relationships** - How files in this folder relate to each other
4. **Guide Navigation** - Help developers find what they need quickly
5. **Reduce Onboarding Time** - New team members can understand the codebase structure

---

## Template Structure

Every Section README MUST follow this structure:

```markdown
# [Folder Name]

## Overview

Brief description of what this folder contains and its purpose in the project.

---

## Directory Tree

\`\`\`
folder-name/
├── README.md              # This file
├── subfolder/
│   ├── file1.ext
│   └── file2.ext
├── file3.ext
└── file4.ext
\`\`\`

---

## Files

| File/Folder | Purpose |
|-------------|---------|
| `subfolder/` | Description of subfolder |
| `file3.ext` | Description of file |
| `file4.ext` | Description of file |

---

## Usage

How to use the code/content in this folder. Include examples if applicable.

---

## Related Sections

- [../other-folder/](../other-folder/) - How this relates to other-folder
- [../another/](../another/) - How this relates to another folder
```

---

## When to Create Section READMEs

Create a README.md in a folder when:

1. **The folder has 3+ files** - Worth documenting structure
2. **The folder contains business logic** - Services, models, controllers
3. **The folder is a module boundary** - Components, features, domains
4. **The folder contains configuration** - Config, settings, environment
5. **New folders are created** - Part of the setup/scaffolding process

Do NOT create READMEs for:
- Vendor/node_modules folders
- Build output folders (dist, build, .next)
- Cache folders (.cache, __pycache__)
- IDE/editor folders (.vscode, .idea) unless project-specific

---

## Examples

### API Folder README

```markdown
# API

## Overview

This folder contains all API route handlers and endpoint definitions. The API follows RESTful conventions and is organised by resource.

---

## Directory Tree

\`\`\`
api/
├── README.md
├── routes/
│   ├── users.ts
│   ├── orders.ts
│   └── products.ts
├── middleware/
│   ├── auth.ts
│   └── validation.ts
└── index.ts
\`\`\`

---

## Endpoints

| Route File | Base Path | Description |
|------------|-----------|-------------|
| `users.ts` | `/api/users` | User CRUD operations |
| `orders.ts` | `/api/orders` | Order management |
| `products.ts` | `/api/products` | Product catalogue |

---

## Adding New Endpoints

1. Create a new route file in `routes/`
2. Register routes in `index.ts`
3. Add middleware as needed

---

## Related Sections

- [../services/](../services/) - Business logic used by endpoints
- [../models/](../models/) - Data models for request/response
- [../middleware/](../middleware/) - Shared middleware
```

---

### Components Folder README

```markdown
# Components

## Overview

Reusable React components organised by category. All components follow the project's design system and accessibility standards.

---

## Directory Tree

\`\`\`
components/
├── README.md
├── ui/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── index.ts
│   ├── Input/
│   └── Modal/
├── layout/
│   ├── Header/
│   ├── Footer/
│   └── Sidebar/
└── forms/
    ├── LoginForm/
    └── RegisterForm/
\`\`\`

---

## Component Categories

| Category | Purpose | Examples |
|----------|---------|----------|
| `ui/` | Base UI elements | Button, Input, Modal |
| `layout/` | Page structure | Header, Footer, Sidebar |
| `forms/` | Form components | LoginForm, RegisterForm |

---

## Adding New Components

1. Create folder with component name
2. Add component file, test, and index export
3. Follow existing patterns in the category

---

## Related Sections

- [../styles/](../styles/) - Shared styles and themes
- [../hooks/](../hooks/) - Custom hooks used by components
- [../utils/](../utils/) - Utility functions
```

---

### Services Folder README

```markdown
# Services

## Overview

Business logic layer containing service classes that handle data operations, external API calls, and complex business rules. Services are injected into controllers/handlers.

---

## Directory Tree

\`\`\`
services/
├── README.md
├── UserService.ts
├── OrderService.ts
├── PaymentService.ts
├── EmailService.ts
└── __tests__/
    ├── UserService.test.ts
    └── OrderService.test.ts
\`\`\`

---

## Service Files

| Service | Responsibility |
|---------|----------------|
| `UserService.ts` | User registration, authentication, profile management |
| `OrderService.ts` | Order creation, status updates, history |
| `PaymentService.ts` | Payment processing, refunds |
| `EmailService.ts` | Email sending, template rendering |

---

## Usage Patterns

Services are instantiated via dependency injection:

\`\`\`typescript
const userService = new UserService(database);
const user = await userService.findById(userId);
\`\`\`

---

## Related Sections

- [../repositories/](../repositories/) - Data access layer
- [../models/](../models/) - Data models
- [../api/](../api/) - API routes that use services
```

---

### Utils Folder README

```markdown
# Utils

## Overview

Shared utility functions used across the application. All utilities are pure functions with no side effects.

---

## Directory Tree

\`\`\`
utils/
├── README.md
├── date.ts
├── string.ts
├── validation.ts
├── formatters.ts
└── __tests__/
    ├── date.test.ts
    └── string.test.ts
\`\`\`

---

## Utility Files

| File | Contents |
|------|----------|
| `date.ts` | Date formatting, parsing, comparison |
| `string.ts` | String manipulation, slugify, truncate |
| `validation.ts` | Input validation helpers |
| `formatters.ts` | Currency, number, phone formatting |

---

## Related Sections

- [../types/](../types/) - TypeScript type definitions
- [../constants/](../constants/) - Constant values
```

---

### Config Folder README

```markdown
# Config

## Overview

Application configuration organised by environment. Uses environment variables for secrets and sensitive values.

---

## Directory Tree

\`\`\`
config/
├── README.md
├── index.ts           # Config loader
├── base.ts            # Shared settings
├── development.ts     # Development settings
├── testing.ts         # Test settings
├── staging.ts         # Staging settings
└── production.ts      # Production settings
\`\`\`

---

## Configuration Files

| File | Purpose |
|------|---------|
| `index.ts` | Exports merged config based on NODE_ENV |
| `base.ts` | Settings shared across all environments |
| `development.ts` | Local development settings |
| `testing.ts` | Test runner settings (test database) |
| `staging.ts` | Staging environment settings |
| `production.ts` | Production environment settings |

---

## Environment Loading

\`\`\`typescript
import config from './config';

// Automatically loads correct config based on NODE_ENV
console.log(config.database.host);
\`\`\`

---

## Related Sections

- [../.env.example](../.env.example) - Environment variable template
- [../docs/SETUP/](../docs/SETUP/) - Setup documentation
```

---

### Tests Folder README

```markdown
# Tests

## Overview

Test files organised by type. Uses Jest as the test runner with React Testing Library for component tests.

---

## Directory Tree

\`\`\`
tests/
├── README.md
├── unit/
│   ├── services/
│   └── utils/
├── integration/
│   ├── api/
│   └── database/
├── e2e/
│   ├── auth.spec.ts
│   └── checkout.spec.ts
├── fixtures/
│   └── users.json
└── setup.ts
\`\`\`

---

## Test Categories

| Category | Purpose | Runner |
|----------|---------|--------|
| `unit/` | Isolated function tests | Jest |
| `integration/` | Service + database tests | Jest |
| `e2e/` | Full user journey tests | Playwright |
| `fixtures/` | Test data | - |

---

## Running Tests

\`\`\`bash
# Run all unit tests
npm run test:unit

# Run integration tests
npm run test:integration

# Run E2E tests
npm run test:e2e

# Run all tests
npm test
\`\`\`

---

## Related Sections

- [../src/](../src/) - Source code being tested
- [../docs/TESTS/](../docs/TESTS/) - Test documentation
```

---

### Docs Folder README

```markdown
# Documentation

## Overview

Project documentation organised by category. All markdown files use CAPITALISED filenames.

---

## Directory Tree

\`\`\`
docs/
├── README.md
├── API/
│   ├── ENDPOINTS.md
│   └── AUTHENTICATION.md
├── SETUP/
│   ├── DEVELOPMENT.md
│   └── PRODUCTION.md
├── ARCHITECTURE/
│   ├── OVERVIEW.md
│   └── DATABASE.md
├── DATABASE/
│   ├── MIGRATIONS/
│   └── SCHEMA-2025-01-15.sql
└── GUIDES/
    ├── CONTRIBUTING.md
    └── TESTING.md
\`\`\`

---

## Documentation Categories

| Folder | Contents |
|--------|----------|
| `API/` | API endpoint reference and authentication |
| `SETUP/` | Development and production setup guides |
| `ARCHITECTURE/` | System architecture and design decisions |
| `DATABASE/` | Schema snapshots and migration history |
| `GUIDES/` | Contributing, testing, and deployment guides |

---

## Related Sections

- [../README.md](../README.md) - Project root README
- [../CHANGELOG.md](../CHANGELOG.md) - Version history
```

---

## Tree Generation

When generating the directory tree for a README, use this format:

```
folder-name/
├── file1.ext           # Comment about file
├── subfolder/
│   ├── nested1.ext
│   └── nested2.ext
├── file2.ext
└── last-item.ext       # └ for last item, ├ for others
```

### Tree Characters

| Character | Usage |
|-----------|-------|
| `├──` | Item with more siblings below |
| `└──` | Last item in current level |
| `│` | Vertical line for nested levels |
| `   ` | Indent (3 spaces) for alignment |

### Generating Trees Automatically

Use the `tree` command with appropriate flags:

```bash
# Basic tree (2 levels deep)
tree -L 2 folder-name/

# With file size and hidden files
tree -L 2 -ah folder-name/

# Excluding node_modules and build folders
tree -L 3 -I 'node_modules|dist|build' folder-name/
```

Then manually add the README.md reference and purpose comments.