# Django/Wagtail Headless Backend Template

Use this template to set up a headless Django/Wagtail backend with PostgreSQL that exposes a GraphQL API for React web and React Native mobile frontends.

**Template Repository:** `Syntek-Studio/backend_template`

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Architecture Overview](#architecture-overview)
- [Template Repository](#template-repository)
  - [Creating a New Project from Template](#creating-a-new-project-from-template)
  - [Post-Clone Setup](#post-clone-setup)
- [Project CLAUDE.md](#project-claudemd)
- [GraphQL API](#graphql-api)
  - [Endpoint](#endpoint)
  - [Authentication](#authentication)
  - [Example Query](#example-query)
  - [Example Mutation](#example-mutation)
- [Code Conventions](#code-conventions)
  - [Django/Views](#djangoviews)
  - [Wagtail (Headless)](#wagtail-headless)
  - [Type Hinting](#type-hinting)
- [Database Schema](#database-schema)
- [Commands](#commands)
  - [.claude/commands/dev.md](#claudecommandsdevmd)
  - [.claude/commands/test.md](#claudecommandstestmd)
  - [.claude/commands/staging.md](#claudecommandsstagingmd)
  - [.claude/commands/production.md](#claudecommandsproductionmd)
  - [.claude/commands/migrate.md](#claudecommandsmigratemd)
  - [.claude/commands/schema.md](#claudecommandsschemamd)
  - [.claude/commands/make-resolver.md](#claudecommandsmake-resolvermd)
- [Directory Structure](#directory-structure)
- [GraphQL Dependencies](#graphql-dependencies)
- [CORS Configuration](#cors-configuration)
- [Environment-Specific Command Files](#environment-specific-command-files)
  - [Required Root-Level Scripts](#required-root-level-scripts)
  - [dev.sh](#devsh)
  - [test.sh](#testsh)
  - [staging.sh](#stagingsh)
  - [production.sh](#productionsh)
  - [Command File Permissions](#command-file-permissions)


---

## Architecture Overview

This is a **headless backend** that serves as the API layer for:
- **React Web** (`stack-react`) - Web frontend consuming GraphQL API
- **React Native Mobile** (`stack-mobile`) - Mobile app consuming GraphQL API

```
┌─────────────────┐         ┌─────────────────┐
│  React Web      │         │  React Native   │
│  (stack-react)  │         │  (stack-mobile) │
└────────┬────────┘         └────────┬────────┘
         │                           │
         │ GraphQL API               │ GraphQL API
         ▼                           ▼
┌─────────────────────────────────────────────────┐
│              THIS PROJECT                        │
│  Django/Wagtail Headless Backend                │
│  ┌───────────────────────────────────────────┐  │
│  │  GraphQL API (Strawberry/Graphene)        │  │
│  │  - Queries, Mutations, Subscriptions      │  │
│  │  - Authentication (JWT)                    │  │
│  │  - File uploads                           │  │
│  └───────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────┐  │
│  │  PostgreSQL Database                       │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

---

## Template Repository

**GitHub:** `Syntek-Studio/backend_template`

### Creating a New Project from Template

```bash
# Option 1: GitHub template (recommended)
gh repo create @scope/project-api --template Syntek-Studio/backend_template --private

# Option 2: Clone and re-initialise
git clone git@github.com:Syntek-Studio/backend_template.git project-api
cd project-api
rm -rf .git
git init
git remote add origin git@github.com:Your-Org/project-api.git
```

### Post-Clone Setup

After cloning from template, the setup agent will:
1. **Detect the stack** from README.md (`Django` + `Wagtail` + `GraphQL` → `stack-django`)
2. **Ask for project name** (e.g., `project-api`)
3. **Update configuration files** with new project name
4. **Update .claude/CLAUDE.md** with project-specific details
5. **Verify git remote** matches project name

---

## Project CLAUDE.md

Create this file at `.claude/CLAUDE.md` or `CLAUDE.md` in the project root:

```markdown
# Project: [Insert Project Name] - Backend

## Stack Overview

| Component     | Technology                       |
| ------------- | -------------------------------- |
| **Type**      | Headless Backend (GraphQL API)   |
| **Language**  | Python 3.12                      |
| **Framework** | Django 5.x / Wagtail 6.x         |
| **API**       | GraphQL (Strawberry or Graphene) |
| **Database**  | PostgreSQL 15                    |
| **Server**    | Gunicorn / Nginx                 |
| **Testing**   | pytest, pytest-django            |
| **Container** | Docker Compose                   |

---

## Skill Targets

- **Stack Skill:** `stack-django`
- **Global Skill:** `global-workflow`

---

## Architecture

This is a **headless backend** serving GraphQL API to:
- `stack-react` - React web frontend
- `stack-mobile` - React Native mobile app

Both frontends consume the same GraphQL API and share styling from `stack-shared-lib`.

---

## Environment

| Setting                   | Value                                     |
| ------------------------- | ----------------------------------------- |
| **Local URL**             | http://localhost:8000                     |
| **GraphQL Endpoint**      | http://localhost:8000/graphql/            |
| **GraphQL Playground**    | http://localhost:8000/graphql/ (dev only) |
| **Admin URL**             | /admin/                                   |
| **Wagtail Admin**         | /cms/                                     |
| **Database (Dev)**        | [project-name]_dev                        |
| **Database (Test)**       | [project-name]_test                       |
| **Database (Staging)**    | [project-name]_staging                    |
| **Database (Production)** | [project-name]_production                 |
| **Locale**                | en_GB                                     |
| **Timezone**              | Europe/London                             |
| **Currency**              | GBP (£)                                   |

---

## Key Locations

| Directory                | Purpose                                    |
| ------------------------ | ------------------------------------------ |
| `apps/core/`             | Shared functionality and utilities         |
| `apps/core/schema.py`    | Root GraphQL schema                        |
| `apps/users/`            | User management and authentication         |
| `apps/users/schema.py`   | User-related GraphQL types/queries         |
| `apps/content/`          | Wagtail content (headless pages, snippets) |
| `apps/content/schema.py` | Content GraphQL types/queries              |
| `apps/api/`              | GraphQL configuration and middleware       |
| `config/settings/`       | Environment-specific settings              |
| `tests/`                 | Test files                                 |

---

## Development Commands

```bash
# Environment
docker compose up -d                 # Start containers (detached)
docker compose down                  # Stop containers
docker compose logs -f web           # View web logs
docker compose ps                    # List running containers

# Django Management
docker compose exec web python manage.py <command>
docker compose exec web python manage.py migrate
docker compose exec web python manage.py makemigrations
docker compose exec web python manage.py createsuperuser
docker compose exec web python manage.py shell

# Testing
docker compose exec web pytest                    # Run all tests
docker compose exec web pytest --cov=apps        # With coverage

# GraphQL
# Access GraphQL Playground at http://localhost:8000/graphql/
# Schema introspection enabled in development
```

---

## GraphQL API

### Endpoint
- **URL:** `/graphql/`
- **Method:** POST (queries/mutations), GET (playground in dev)

### Authentication
- JWT tokens via `Authorization: Bearer <token>` header
- Token refresh endpoint available

### Example Query
```graphql
query GetCurrentUser {
  me {
    id
    email
    firstName
    lastName
  }
}
```

### Example Mutation
```graphql
mutation Login($email: String!, $password: String!) {
  tokenAuth(email: $email, password: $password) {
    token
    refreshToken
    user {
      id
      email
    }
  }
}
```

---

## Code Conventions

### Django/Views
- **No traditional views** - All data exposed via GraphQL
- **CRITICAL:** Always use **Class-Based Views (CBVs)** instead of Function-Based Views (FBVs) when views are required (e.g., webhooks, health checks, admin customisations)
- Business logic lives in **Services**, not resolvers
- Keep GraphQL resolvers thin - delegate to services

### Wagtail (Headless)
- Use Wagtail for content management only
- Expose content via GraphQL, not traditional templates
- Use `StreamField` for flexible content structures

### Type Hinting
**CRITICAL:** All Python code must use strict type hints.

```python
from typing import Optional
from strawberry import type, field

@type
class UserType:
    id: str
    email: str
    first_name: str
    last_name: str

    @field
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"
```

---

## Database Schema

*(Document key tables and relationships here)*
```

---

## Settings File

Create this file at `.claude/settings.local.json`:

```json
{
  "language": "python",
  "framework": "django",
  "projectType": "headless-backend",
  "api": "graphql",
  "locale": "en_GB",
  "timezone": "Europe/London",
  "permissions": {
    "allow": [
      "Read(**)",
      "Edit(**)",
      "Write(**)",
      "Bash(docker:*)",
      "Bash(docker-compose:*)",
      "Bash(python:*)",
      "Bash(pip:*)",
      "Bash(pytest:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo:*)"
    ]
  },
  "environment": {
    "containerType": "docker-compose",
    "envFiles": [
      ".env",
      ".env.dev",
      ".env.test",
      ".env.staging",
      ".env.production"
    ]
  },
  "testing": {
    "framework": "pytest",
    "command": "docker compose exec web pytest",
    "coverageCommand": "docker compose exec web pytest --cov=apps"
  },
  "linting": {
    "python": "docker compose exec web ruff check .",
    "format": "docker compose exec web black ."
  },
  "graphql": {
    "endpoint": "/graphql/",
    "playground": true,
    "introspection": true
  },
  "consumers": [
    "stack-react",
    "stack-mobile"
  ]
}
```

---

## Commands

Create these files in `.claude/commands/`:

**CRITICAL:** Commands should reference the root-level environment scripts (`dev.sh`, `test.sh`, `staging.sh`, `production.sh`) to ensure consistency and avoid duplication.

### .claude/commands/dev.md

```markdown
---
description: Start the Docker development environment
usage: /dev
---

Start the Django headless backend development environment.

**Run:** `./dev.sh`

This script will:
1. Start Docker containers with development configuration
2. Wait for the database to be ready
3. Run any pending migrations
4. Display access URLs for GraphQL Playground and Django Admin

**Access:**
- GraphQL Playground: http://localhost:8000/graphql/
- Django Admin: http://localhost:8000/admin/
- Wagtail CMS: http://localhost:8000/cms/

$ARGUMENTS
```

### .claude/commands/test.md

```markdown
---
description: Run the pytest test suite
usage: /test
---

Run the project's pytest test suite using the test database.

**Run:** `./test.sh`

This script will:
1. Set the DJANGO_ENV to testing
2. Run pytest with the test database configuration

**Additional Commands:**
- With filter: `docker compose exec web pytest -k "$ARGUMENTS"`
- With coverage: `docker compose exec web pytest --cov=apps`
- GraphQL tests: `docker compose exec web pytest tests/graphql/`

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
2. Build Docker images with staging configuration
3. Deploy to staging environment

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
3. Build Docker images with production configuration
4. Deploy to production environment

$ARGUMENTS
```

### .claude/commands/migrate.md

```markdown
---
description: Run database migrations
usage: /migrate
---

Run Django database migrations.

**Commands:**
- Make migrations: `docker compose exec web python manage.py makemigrations`
- Run migrations: `docker compose exec web python manage.py migrate`
- Show migrations: `docker compose exec web python manage.py showmigrations`

$ARGUMENTS
```

### .claude/commands/schema.md

```markdown
---
description: Generate or export GraphQL schema
usage: /schema
---

Work with the GraphQL schema.

**Commands:**
- Export schema: `docker compose exec web python manage.py graphql_schema --out schema.graphql`
- View in playground: http://localhost:8000/graphql/

**Schema Location:** `apps/core/schema.py` (root schema)

$ARGUMENTS
```

### .claude/commands/make-resolver.md

```markdown
---
description: Create a new GraphQL resolver
usage: /make-resolver <app_name> <resolver_name>
---

Create a new GraphQL resolver with the project conventions.

**Creates:**
- Query/Mutation in `apps/$ARGUMENTS/schema.py`
- Service in `apps/$ARGUMENTS/services.py`
- Types in `apps/$ARGUMENTS/types.py`

**Remember to:**
1. Register in root schema (`apps/core/schema.py`)
2. Add tests in `tests/graphql/`

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
│       ├── migrate.md
│       ├── schema.md
│       └── make-resolver.md
├── apps/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── middleware.py      # GraphQL middleware
│   │   └── urls.py            # GraphQL URL config
│   ├── core/
│   │   ├── __init__.py
│   │   ├── schema.py          # Root GraphQL schema
│   │   ├── models.py
│   │   ├── services.py
│   │   └── utils.py
│   ├── users/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── schema.py          # User GraphQL types/queries
│   │   ├── types.py           # User GraphQL types
│   │   ├── mutations.py       # User mutations
│   │   └── services.py
│   └── content/
│       ├── __init__.py
│       ├── models.py          # Wagtail page models
│       ├── schema.py          # Content GraphQL types
│       ├── blocks.py          # StreamField blocks
│       └── services.py
├── config/
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── testing.py
│   │   ├── staging.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── tests/
│   ├── conftest.py
│   ├── graphql/               # GraphQL-specific tests
│   │   ├── test_queries.py
│   │   └── test_mutations.py
│   ├── test_models.py
│   └── test_services.py
├── docs/
│   ├── API/
│   │   └── GRAPHQL-SCHEMA.md  # GraphQL schema documentation
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
├── docker-compose.staging.yml
├── docker-compose.production.yml
├── Dockerfile
├── Dockerfile.production
├── requirements.txt
├── requirements-dev.txt
├── pyproject.toml
├── schema.graphql             # Exported GraphQL schema
├── .env.dev.example
├── .env.test.example
├── .env.staging.example
├── .env.production.example
├── CHANGELOG.md
├── README.md
├── manage.py
├── dev.sh
├── staging.sh
└── production.sh
```

---

## GraphQL Dependencies

Add these to `requirements.txt`:

```
# GraphQL
strawberry-graphql[django]>=0.220.0
# OR for Graphene:
# graphene-django>=3.0.0

# Authentication
django-graphql-jwt>=0.4.0
PyJWT>=2.8.0

# CORS (for frontend access)
django-cors-headers>=4.3.0
```

---

## CORS Configuration

Configure CORS for frontend access in `config/settings/base.py`:

```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",      # React dev server
    "http://localhost:8081",      # React Native (Expo)
]

CORS_ALLOW_CREDENTIALS = True
```

---

## Environment-Specific Command Files

**CRITICAL:** All Django projects using Docker MUST have environment-specific command files for managing different environments.

### Required Root-Level Scripts

| File            | Purpose                           | Environment |
| --------------- | --------------------------------- | ----------- |
| `dev.sh`        | Start development environment     | Development |
| `test.sh`       | Run test suite with test database | Testing     |
| `staging.sh`    | Deploy to staging environment     | Staging     |
| `production.sh` | Deploy to production environment  | Production  |

### dev.sh

```bash
#!/bin/bash
set -e

echo "🚀 Starting development environment..."

docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d

echo "⏳ Waiting for database..."
sleep 5

docker-compose exec web python manage.py migrate

echo "✅ Development environment ready!"
echo "🌐 GraphQL Playground: http://localhost:8000/graphql/"
echo "🔧 Django Admin: http://localhost:8000/admin/"
```

### test.sh

```bash
#!/bin/bash
set -e

echo "🧪 Running test suite..."

# Use test database configuration
export DJANGO_ENV=testing

docker-compose -f docker-compose.yml -f docker-compose.test.yml run --rm web pytest

echo "✅ Tests complete!"
```

### staging.sh

```bash
#!/bin/bash
set -e

echo "🚀 Deploying to staging..."

# Run tests first
./test.sh

# Build and deploy
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

# Build and deploy
docker-compose -f docker-compose.yml -f docker-compose.production.yml build
docker-compose -f docker-compose.yml -f docker-compose.production.yml up -d

echo "✅ Production deployment complete!"
```

### Command File Permissions

**CRITICAL:** All shell scripts MUST be executable:

```bash
chmod +x dev.sh test.sh staging.sh production.sh
```
