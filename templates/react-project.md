# React Web Frontend Template

Use this template to set up a React web frontend that consumes a GraphQL API from the Django backend and uses the shared library for styling/components.

**Template Repository:** `Syntek-Studio/frontend_template`

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Architecture Overview](#architecture-overview)
- [Template Repository](#template-repository)
  - [Creating a New Project from Template](#creating-a-new-project-from-template)
  - [Post-Clone Setup](#post-clone-setup)
- [Project CLAUDE.md](#project-claudemd)
- [GraphQL Integration](#graphql-integration)
  - [Apollo Client Setup](#apollo-client-setup)
  - [Code Generation](#code-generation)
- [Shared Library Usage](#shared-library-usage)
- [Code Conventions](#code-conventions)
  - [Components](#components)
  - [Styling](#styling)
- [Commands](#commands)
  - [.claude/commands/dev.md](#claudecommandsdevmd)
  - [.claude/commands/test.md](#claudecommandstestmd)
  - [.claude/commands/staging.md](#claudecommandsstagingmd)
  - [.claude/commands/production.md](#claudecommandsproductionmd)
  - [.claude/commands/codegen.md](#claudecommandscodegenmd)
  - [.claude/commands/build.md](#claudecommandsbuildmd)
  - [.claude/commands/link-shared.md](#claudecommandslink-sharedmd)
  - [test.sh](#testsh)
  - [staging.sh](#stagingsh)
  - [production.sh](#productionsh)
  - [Command File Permissions](#command-file-permissions)
- [GraphQL Code Generator Configuration](#graphql-code-generator-configuration)
- [Environment Variables](#environment-variables)
- [Package Dependencies](#package-dependencies)


---

## Architecture Overview

This is a **web frontend** that:
- **Consumes** GraphQL API from `stack-django` (Django/Wagtail backend)
- **Imports** components, typography, fonts, and colours from `stack-shared-lib`

```
┌─────────────────────────────────────────────────────────────────┐
│                        THIS PROJECT                              │
│                     React Web Frontend                           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  UI Layer                                                  │  │
│  │  - React 18+ with TypeScript                              │  │
│  │  - Tailwind CSS v4                                        │  │
│  │  - Components from @company/shared-lib                    │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Data Layer                                                │  │
│  │  - Apollo Client / urql                                   │  │
│  │  - GraphQL queries and mutations                          │  │
│  │  - Generated TypeScript types from schema                 │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              │ GraphQL API
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Django Backend                               │
│                    (stack-django)                                │
│  - GraphQL API at /graphql/                                     │
│  - JWT Authentication                                           │
│  - PostgreSQL Database                                          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     Shared Library                               │
│                   (stack-shared-lib)                             │
│  - Components (Button, Card, Modal, etc.)                       │
│  - Typography system                                            │
│  - Colour palette and design tokens                             │
│  - Fonts                                                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Template Repository

**GitHub:** `Syntek-Studio/frontend_template`

### Creating a New Project from Template

```bash
# Option 1: GitHub template (recommended)
gh repo create @scope/project-web --template Syntek-Studio/frontend_template --private

# Option 2: Clone and re-initialise
git clone git@github.com:Syntek-Studio/frontend_template.git project-web
cd project-web
rm -rf .git
git init
git remote add origin git@github.com:Your-Org/project-web.git
```

### Post-Clone Setup

After cloning from template, the setup agent will:
1. **Detect the stack** from README.md (`React Web` + `GraphQL` → `stack-react`)
2. **Ask for project name** (e.g., `@company/project-web`)
3. **Update package.json** with new name and scope
4. **Update .claude/CLAUDE.md** with project-specific details
5. **Verify git remote** matches project name

---

## Project CLAUDE.md

Create this file at `.claude/CLAUDE.md` or `CLAUDE.md` in the project root:

```markdown
# Project: [Insert Project Name] - Web

## Stack Overview

| Component           | Technology                                 |
| ------------------- | ------------------------------------------ |
| **Type**            | React Web Frontend (GraphQL Consumer)      |
| **Language**        | TypeScript 5.x                             |
| **Framework**       | React 18+                                  |
| **Build Tool**      | Vite                                       |
| **Styling**         | Tailwind CSS v4 + shared-lib design tokens |
| **GraphQL Client**  | Apollo Client / urql                       |
| **Code Generation** | GraphQL Code Generator                     |
| **Testing**         | Vitest, React Testing Library              |
| **Container**       | Docker                                     |

---

## Skill Targets

- **Stack Skill:** `stack-react`
- **Global Skill:** `global-workflow`

---

## Architecture

This frontend:
- **Consumes** GraphQL API from `stack-django` backend
- **Imports** styling and components from `stack-shared-lib`
- Shares the same design system with `stack-mobile` (React Native)

---

## Environment

| Setting                      | Value                                 |
| ---------------------------- | ------------------------------------- |
| **Local URL**                | http://localhost:3000                 |
| **Backend API (Dev)**        | http://localhost:8000/graphql/        |
| **Backend API (Staging)**    | https://staging-api.[domain]/graphql/ |
| **Backend API (Production)** | https://api.[domain]/graphql/         |
| **Locale**                   | en_GB                                 |
| **Timezone**                 | Europe/London                         |
| **Currency**                 | GBP (£)                               |

---

## Key Locations

| Directory                | Purpose                               |
| ------------------------ | ------------------------------------- |
| `src/components/`        | App-specific components               |
| `src/pages/`             | Route page components                 |
| `src/graphql/`           | GraphQL queries, mutations, fragments |
| `src/graphql/generated/` | Auto-generated types and hooks        |
| `src/hooks/`             | Custom React hooks                    |
| `src/contexts/`          | React Context providers               |
| `src/services/`          | Non-GraphQL services                  |
| `src/utils/`             | Utility functions                     |
| `src/types/`             | Shared TypeScript types               |

---

## Development Commands

```bash
# Environment (Docker)
docker compose up                    # Start with logs
docker compose up -d                 # Start detached
docker compose down                  # Stop containers

# Development
docker compose run --rm app npm run dev           # Dev server
docker compose run --rm app npm run build         # Production build

# GraphQL
docker compose run --rm app npm run codegen       # Generate types from schema
docker compose run --rm app npm run codegen:watch # Watch mode

# Testing
docker compose run --rm app npm test              # Run tests
docker compose run --rm app npm run test:coverage # With coverage

# Linting
docker compose run --rm app npm run lint          # ESLint
docker compose run --rm app npm run format        # Prettier
```

---

## GraphQL Integration

### Apollo Client Setup
```typescript
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: import.meta.env.VITE_GRAPHQL_ENDPOINT,
});

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

export const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
});
```

### Code Generation
Run `npm run codegen` to generate TypeScript types from the backend schema.

---

## Shared Library Usage

Import components and styles from the shared library:

```typescript
import { Button, Card, Typography } from '@company/shared-lib';
import { colours, spacing } from '@company/shared-lib/tokens';
```

The shared library provides:
- **Components:** Button, Card, Modal, Input, etc.
- **Typography:** Heading, Text, Label components
- **Design Tokens:** Colours, spacing, breakpoints
- **Fonts:** Pre-configured font families

---

## Code Conventions

### Components
- **Functional components only** with TypeScript interfaces
- Use shared-lib components where available
- App-specific components extend shared-lib patterns

### Styling
- Use Tailwind CSS v4 utilities
- Design tokens from shared-lib for consistency
- Avoid hardcoded colours - use token references

---

*(Add project-specific conventions here)*
```

---

## Settings File

Create this file at `.claude/settings.local.json`:

```json
{
  "language": "typescript",
  "framework": "react",
  "projectType": "graphql-consumer",
  "locale": "en_GB",
  "timezone": "Europe/London",
  "permissions": {
    "allow": [
      "Read(**)",
      "Edit(**)",
      "Write(**)",
      "Bash(docker:*)",
      "Bash(docker-compose:*)",
      "Bash(npm:*)",
      "Bash(npx:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo:*)"
    ]
  },
  "environment": {
    "containerType": "docker",
    "envFiles": [
      ".env",
      ".env.local",
      ".env.development",
      ".env.staging",
      ".env.production"
    ]
  },
  "testing": {
    "framework": "vitest",
    "command": "docker compose run --rm app npm test",
    "coverageCommand": "docker compose run --rm app npm run test:coverage"
  },
  "linting": {
    "eslint": "docker compose run --rm app npm run lint",
    "prettier": "docker compose run --rm app npm run format"
  },
  "graphql": {
    "backend": "stack-django",
    "endpoint": "VITE_GRAPHQL_ENDPOINT",
    "codegen": "npm run codegen"
  },
  "dependencies": {
    "sharedLib": "@company/shared-lib",
    "backend": "stack-django"
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
description: Start the React development server
usage: /dev
---

Start the React web development environment.

**Run:** `./dev.sh`

**Prerequisites:**
- Ensure Django backend is running at http://localhost:8000
- Ensure shared-lib is linked: `npm link @company/shared-lib`

This script will:
1. Start Docker containers with development configuration
2. Access the app at http://localhost:3000
3. GraphQL requests go to Django backend

$ARGUMENTS
```

### .claude/commands/test.md

```markdown
---
description: Run the Vitest test suite
usage: /test
---

Run the project's Vitest test suite.

**Run:** `./test.sh`

This script will run the full test suite.

**Additional Commands:**
- Watch mode: `docker compose run --rm app npm test -- --watch`
- With coverage: `docker compose run --rm app npm run test:coverage`

$ARGUMENTS
```

### .claude/commands/staging.md

```markdown
---
description: Deploy to staging environment
usage: /staging
---

Build and deploy to the staging environment.

**Run:** `./staging.sh`

This script will:
1. Run the test suite first
2. Generate GraphQL types
3. Build and deploy with staging configuration

$ARGUMENTS
```

### .claude/commands/production.md

```markdown
---
description: Deploy to production environment
usage: /production
---

Build and deploy to the production environment.

**Run:** `./production.sh`

This script will:
1. Prompt for confirmation (safety check)
2. Run the test suite first
3. Generate GraphQL types
4. Build and deploy with production configuration

$ARGUMENTS
```

### .claude/commands/codegen.md

```markdown
---
description: Generate TypeScript types from GraphQL schema
usage: /codegen
---

Generate TypeScript types and hooks from the Django backend's GraphQL schema.

**Commands:**
- One-time: `docker compose run --rm app npm run codegen`
- Watch mode: `docker compose run --rm app npm run codegen:watch`

**Prerequisites:**
- Django backend must be running with introspection enabled
- Schema endpoint: http://localhost:8000/graphql/

**Output:** `src/graphql/generated/`

$ARGUMENTS
```

### .claude/commands/build.md

```markdown
---
description: Build for production
usage: /build
---

Build the React application for production.

**Run:** `./production.sh` (recommended for full build)

Or manually:
1. Run `docker compose run --rm app npm run codegen` to ensure types are up to date
2. Run `docker compose run --rm app npm run build`
3. Output is in `dist/` directory

$ARGUMENTS
```

### .claude/commands/link-shared.md

```markdown
---
description: Link the shared library for local development
usage: /link-shared
---

Link the shared library for local development.

**Using npm link:**
```bash
# In shared-lib project:
npm link

# In this project:
npm link @company/shared-lib
```

**Using yalc (recommended):**
```bash
# In shared-lib project:
yalc push

# In this project:
yalc add @company/shared-lib
```

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
│       ├── codegen.md
│       ├── build.md
│       └── link-shared.md
├── src/
│   ├── components/                 # App-specific components
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   └── Footer.tsx
│   │   └── features/
│   │       └── UserProfile.tsx
│   ├── pages/
│   │   ├── HomePage.tsx
│   │   └── ProfilePage.tsx
│   ├── graphql/
│   │   ├── queries/
│   │   │   ├── user.graphql
│   │   │   └── content.graphql
│   │   ├── mutations/
│   │   │   └── auth.graphql
│   │   ├── fragments/
│   │   │   └── userFields.graphql
│   │   └── generated/              # Auto-generated by codegen
│   │       ├── types.ts
│   │       └── hooks.ts
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   └── useCurrentUser.ts
│   ├── contexts/
│   │   └── AuthContext.tsx
│   ├── services/
│   │   └── apollo.ts               # Apollo Client setup
│   ├── utils/
│   │   └── formatters.ts
│   ├── types/
│   │   └── index.ts
│   ├── styles/
│   │   └── globals.css
│   ├── App.tsx
│   └── main.tsx
├── public/
├── tests/
│   └── setup.ts
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
├── docker-compose.yml
├── docker-compose.dev.yml
├── Dockerfile
├── package.json
├── tsconfig.json
├── vite.config.ts
├── vitest.config.ts
├── tailwind.config.js
├── codegen.ts                      # GraphQL Code Generator config
├── .env.example
├── .env.development.example
├── .env.staging.example
├── .env.production.example
├── CHANGELOG.md
├── README.md
├── dev.sh
├── test.sh
├── staging.sh
└── production.sh
```

---

## Environment-Specific Command Files

**CRITICAL:** All React projects using Docker MUST have environment-specific command files for managing different environments.

### Required Root-Level Scripts

| File            | Purpose                          | Environment |
| --------------- | -------------------------------- | ----------- |
| `dev.sh`        | Start development environment    | Development |
| `test.sh`       | Run test suite                   | Testing     |
| `staging.sh`    | Deploy to staging environment    | Staging     |
| `production.sh` | Deploy to production environment | Production  |

### dev.sh

```bash
#!/bin/bash
set -e

echo "🚀 Starting development environment..."

docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d

echo "✅ Development environment ready!"
echo "🌐 Access your app at: http://localhost:3000"
echo "📡 Backend GraphQL: http://localhost:8000/graphql/"
```

### test.sh

```bash
#!/bin/bash
set -e

echo "🧪 Running test suite..."

docker-compose -f docker-compose.yml -f docker-compose.test.yml run --rm app npm test

echo "✅ Tests complete!"
```

### staging.sh

```bash
#!/bin/bash
set -e

echo "🚀 Deploying to staging..."

# Run tests first
./test.sh

# Generate GraphQL types
docker-compose run --rm app npm run codegen

# Build for staging
docker-compose -f docker-compose.yml -f docker-compose.staging.yml build
docker-compose -f docker-compose.yml -f docker-compose.staging.yml up -d

echo "✅ Staging deployment complete!"
```

### production.sh

```bash
#!/bin/bash
set -e

echo "🚀 Deploying to production..."

read -p "⚠️  Are you sure you want to deploy to PRODUCTION? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "❌ Deployment cancelled."
    exit 1
fi

# Run tests first
./test.sh

# Generate GraphQL types
docker-compose run --rm app npm run codegen

# Build for production
docker-compose -f docker-compose.yml -f docker-compose.production.yml build
docker-compose -f docker-compose.yml -f docker-compose.production.yml up -d

echo "✅ Production deployment complete!"
```

### Command File Permissions

**CRITICAL:** All shell scripts MUST be executable:

```bash
chmod +x dev.sh test.sh staging.sh production.sh
```

---

## GraphQL Code Generator Configuration

Create `codegen.ts`:

```typescript
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: process.env.VITE_GRAPHQL_ENDPOINT || 'http://localhost:8000/graphql/',
  documents: ['src/graphql/**/*.graphql'],
  generates: {
    'src/graphql/generated/types.ts': {
      plugins: ['typescript'],
    },
    'src/graphql/generated/hooks.ts': {
      preset: 'client',
      plugins: ['typescript-operations', 'typescript-react-apollo'],
      config: {
        withHooks: true,
        withComponent: false,
      },
    },
  },
};

export default config;
```

---

## Environment Variables

```env
# .env.development.example
VITE_GRAPHQL_ENDPOINT=http://localhost:8000/graphql/
VITE_APP_NAME="[Project Name] (Development)"

# .env.staging.example
VITE_GRAPHQL_ENDPOINT=https://staging-api.[domain]/graphql/
VITE_APP_NAME="[Project Name] (Staging)"

# .env.production.example
VITE_GRAPHQL_ENDPOINT=https://api.[domain]/graphql/
VITE_APP_NAME="[Project Name]"
```

---

## Package Dependencies

Key dependencies to add:

```json
{
  "dependencies": {
    "@apollo/client": "^3.8.0",
    "@company/shared-lib": "^1.0.0",
    "graphql": "^16.8.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0"
  },
  "devDependencies": {
    "@graphql-codegen/cli": "^5.0.0",
    "@graphql-codegen/client-preset": "^4.1.0",
    "@graphql-codegen/typescript": "^4.0.0",
    "@graphql-codegen/typescript-operations": "^4.0.0",
    "@graphql-codegen/typescript-react-apollo": "^4.0.0",
    "tailwindcss": "^4.0.0"
  }
}
```
