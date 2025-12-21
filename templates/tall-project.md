# TALL Stack Project Template (All-in-One)

Use this template to set up a self-contained TALL (Tailwind, Alpine.js, Laravel, Livewire) project with Claude Code configuration.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Project CLAUDE.md](#project-claudemd)
- [Directory Structure](#directory-structure)
- [Environment-Specific Command Files](#environment-specific-command-files)

---

## Architecture Overview

This is an **all-in-one** stack - backend, frontend, and database are integrated in a single project. Unlike the multi-repo architecture (`stack-django` + `stack-react` + `stack-mobile` + `stack-shared-lib`), this stack is fully self-contained.

```
┌─────────────────────────────────────────────────────────────────┐
│                        THIS PROJECT                              │
│                   TALL Stack (All-in-One)                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Frontend Layer                                            │  │
│  │  - Livewire 3 (reactive components)                       │  │
│  │  - Alpine.js (lightweight JS interactions)                 │  │
│  │  - Tailwind CSS (utility-first styling)                   │  │
│  │  - Blade templates                                        │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Backend Layer                                             │  │
│  │  - Laravel 11.x                                           │  │
│  │  - Eloquent ORM                                           │  │
│  │  - Services & Actions                                     │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Database Layer                                            │  │
│  │  - MariaDB / MySQL                                        │  │
│  │  - DDEV Container                                         │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**When to use TALL Stack:**
- Traditional server-rendered web applications
- Admin panels and dashboards
- Content management systems
- Projects that don't need mobile apps

**When to use Multi-Repo Architecture:**
- Projects needing both web AND mobile apps
- Headless CMS with multiple frontends
- API-first architecture

---

## Project CLAUDE.md

Create this file at `.claude/CLAUDE.md` or `CLAUDE.md` in the project root:

```markdown
# Project: [Insert Project Name]

## Stack Overview

| Component | Technology |
|-----------|------------|
| **Type** | TALL Stack (All-in-One) |
| **Architecture** | Self-contained - Backend + Frontend + Database |
| **Language** | PHP 8.3 |
| **Framework** | Laravel 11.x |
| **Frontend** | Livewire 3, Alpine.js, TailwindCSS |
| **Build Tool** | Vite |
| **Testing** | Pest PHP |
| **Container** | DDEV |
| **Database** | MariaDB 10.11 / MySQL 8 |

**Note:** This is NOT a headless API. For headless architecture, use `stack-django` + `stack-react` + `stack-mobile`.

---

## Skill Targets

- **Stack Skill:** `stack-tall`
- **Global Skill:** `global-workflow`

---

## Environment

| Setting | Value |
|---------|-------|
| **Local URL** | https://[project-name].ddev.site |
| **Database (Dev)** | [project-name]_dev |
| **Database (Test)** | [project-name]_test |
| **Database (Staging)** | [project-name]_staging |
| **Database (Production)** | [project-name]_production |
| **Locale** | en_GB |
| **Timezone** | Europe/London |
| **Currency** | GBP (£) |

---

## Key Locations

| Directory | Purpose |
|-----------|---------|
| `app/Livewire/` | Livewire components |
| `app/Models/` | Eloquent models |
| `app/Services/` | Business logic services |
| `app/Actions/` | Single-purpose action classes |
| `app/Http/Controllers/` | HTTP controllers (minimal) |
| `app/Http/Requests/` | Form request validation |
| `resources/views/components/` | Blade components |
| `resources/views/livewire/` | Livewire component views |
| `resources/views/layouts/` | Layout templates |
| `resources/js/alpine/` | Complex Alpine.js components |
| `custom-packages/` | Local composer packages |
| `tests/Feature/` | Feature/integration tests |
| `tests/Unit/` | Unit tests |

---

## Development Commands

```bash
# Environment
ddev start                           # Start development environment
ddev stop                            # Stop environment
ddev restart                         # Restart environment
ddev describe                        # Show environment info

# Artisan
ddev artisan <command>               # Run artisan commands
ddev artisan migrate                 # Run migrations
ddev artisan migrate:fresh --seed    # Fresh migrate with seeding
ddev artisan tinker                  # Interactive shell
ddev artisan make:livewire <Name>    # Create Livewire component
ddev artisan make:model <Name> -mf   # Create model with migration and factory

# Package Management
ddev composer <command>              # Run composer commands
ddev exec npm <command>              # Run npm commands
ddev exec npm run dev                # Start Vite dev server
ddev exec npm run build              # Build for production

# Testing
ddev exec php artisan test           # Run all tests
ddev exec php artisan test --filter=TestName  # Run specific test
ddev exec ./vendor/bin/pest          # Run Pest directly
ddev exec ./vendor/bin/pest --coverage  # Run with coverage

# Database
ddev import-db < dump.sql            # Import database
ddev export-db > dump.sql            # Export database
ddev mysql                           # MySQL shell
```

---

## Code Conventions

### Livewire Components
- Use `wire:navigate` for SPA-like navigation
- Use `#[Rule]` attributes for validation
- Keep components focused (single responsibility)
- Use `#[Computed]` for derived data

### Blade Templates
- Use `<x-components>` exclusively
- **Never** use `@include` - extract to components instead
- Use slots for flexible component content
- Follow naming: `<x-module.component-name>`

### Alpine.js
- Use `x-data` in separate JS files for complex logic
- Keep inline `x-data` for simple toggle states only
- Use `$wire` for Livewire integration
- Prefer `x-cloak` to prevent flash of unstyled content

### Testing (Pest)
```php
it('can create a user', function () {
    // Arrange
    $data = ['name' => 'Test User', 'email' => 'test@example.com'];

    // Act
    $response = post('/users', $data);

    // Assert
    $response->assertStatus(201);
    expect(User::count())->toBe(1);
});
```

---

## Database Schema

*(Document key tables and relationships here)*

---

## API Endpoints

*(Document API routes if applicable)*
```

---

## Settings File

Create this file at `.claude/settings.local.json`:

```json
{
  "language": "php",
  "framework": "laravel",
  "locale": "en_GB",
  "timezone": "Europe/London",
  "permissions": {
    "allow": [
      "Read(**)",
      "Edit(**)",
      "Write(**)",
      "Bash(ddev:*)",
      "Bash(composer:*)",
      "Bash(npm:*)",
      "Bash(git:*)",
      "Bash(php:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo:*)"
    ]
  },
  "environment": {
    "containerType": "ddev",
    "envFiles": [
      ".env",
      ".env.dev",
      ".env.test",
      ".env.staging",
      ".env.production"
    ]
  },
  "testing": {
    "framework": "pest",
    "command": "ddev exec php artisan test",
    "coverageCommand": "ddev exec ./vendor/bin/pest --coverage"
  },
  "linting": {
    "php": "ddev exec ./vendor/bin/pint",
    "js": "ddev exec npm run lint"
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
description: Start the DDEV development environment
usage: /dev
---

Start the TALL stack development environment using DDEV.

**Run:** `./dev.sh`

This script will:
1. Start DDEV containers
2. Install Composer dependencies
3. Install npm dependencies
4. Run migrations
5. Start Vite dev server

**Access:** Check the output for your site URL (typically https://[project-name].ddev.site)

$ARGUMENTS
```

### .claude/commands/test.md

```markdown
---
description: Run the Pest test suite
usage: /test
---

Run the project's Pest test suite using the test database.

**Run:** `./test.sh`

This script will run tests with the testing environment configuration.

**Additional Commands:**
- With filter: `ddev exec php artisan test --filter=$ARGUMENTS`
- With coverage: `ddev exec ./vendor/bin/pest --coverage`

$ARGUMENTS
```

### .claude/commands/staging.md

```markdown
---
description: Build and prepare for staging deployment
usage: /staging
---

Build and prepare the application for staging deployment.

**Run:** `./staging.sh`

This script will:
1. Run the test suite first
2. Build frontend assets
3. Cache configuration, routes, and views

$ARGUMENTS
```

### .claude/commands/production.md

```markdown
---
description: Build and prepare for production deployment
usage: /production
---

Build and prepare the application for production deployment.

**Run:** `./production.sh`

This script will:
1. Prompt for confirmation (safety check)
2. Run the test suite first
3. Build frontend assets
4. Cache and optimise configuration

$ARGUMENTS
```

### .claude/commands/build.md

```markdown
---
description: Build assets for production
usage: /build
---

Build frontend assets for production deployment.

**Run:** `./production.sh` (recommended for full build)

Or manually:
1. Run `ddev exec npm run build`
2. Run `ddev artisan config:cache`
3. Run `ddev artisan route:cache`
4. Run `ddev artisan view:cache`

$ARGUMENTS
```

### .claude/commands/migrate.md

```markdown
---
description: Run database migrations
usage: /migrate
---

Run Laravel database migrations.

**Commands:**
- Standard: `ddev artisan migrate`
- Fresh with seed: `ddev artisan migrate:fresh --seed`
- Rollback: `ddev artisan migrate:rollback`

$ARGUMENTS
```

### .claude/commands/make-component.md

```markdown
---
description: Create a new Livewire component
usage: /make-component <ComponentName>
---

Create a new Livewire component with the TALL stack conventions.

**Command:** `ddev artisan make:livewire $ARGUMENTS`

This creates:
- `app/Livewire/$ARGUMENTS.php` - Component class
- `resources/views/livewire/$ARGUMENTS.blade.php` - Component view

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
│       ├── build.md
│       ├── migrate.md
│       └── make-component.md
├── .ddev/
│   ├── config.yaml
│   ├── commands/
│   │   └── host/
│   │       ├── dev             # DDEV shortcut to ./dev.sh
│   │       ├── test            # DDEV shortcut to ./test.sh
│   │       ├── staging         # DDEV shortcut to ./staging.sh
│   │       └── production      # DDEV shortcut to ./production.sh
│   └── docker-compose.*.yaml
├── app/
│   ├── Actions/
│   ├── Http/
│   │   ├── Controllers/
│   │   └── Requests/
│   ├── Livewire/
│   ├── Models/
│   └── Services/
├── resources/
│   ├── css/
│   ├── js/
│   │   └── alpine/
│   └── views/
│       ├── components/
│       ├── layouts/
│       └── livewire/
├── tests/
│   ├── Feature/
│   └── Unit/
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
├── .env.dev.example
├── .env.test.example
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

**CRITICAL:** All TALL stack projects using DDEV MUST have environment-specific command files for managing different environments.

### Required Root-Level Scripts

| File | Purpose | Environment |
|------|---------|-------------|
| `dev.sh` | Start development environment | Development |
| `test.sh` | Run test suite with test database | Testing |
| `staging.sh` | Deploy to staging environment | Staging |
| `production.sh` | Deploy to production environment | Production |

### dev.sh

```bash
#!/bin/bash
set -e

echo "🚀 Starting development environment..."

ddev start
ddev composer install
ddev exec npm install
ddev artisan migrate

echo "✅ Development environment ready!"
echo "🌐 Access your site at: $(ddev describe -j | jq -r '.raw.primary_url')"

# Start Vite in the background
ddev exec npm run dev &
```

### test.sh

```bash
#!/bin/bash
set -e

echo "🧪 Running test suite..."

# Run tests with test environment
ddev exec php artisan test --env=testing

echo "✅ Tests complete!"
```

### staging.sh

```bash
#!/bin/bash
set -e

echo "🚀 Deploying to staging..."

# Run tests first
./test.sh

# Build assets
ddev exec npm run build

# Cache configuration
ddev artisan config:cache
ddev artisan route:cache
ddev artisan view:cache

echo "✅ Staging build ready!"
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

# Build assets
ddev exec npm run build

# Cache configuration
ddev artisan config:cache
ddev artisan route:cache
ddev artisan view:cache
ddev artisan optimize

echo "✅ Production build ready!"
```

### DDEV Custom Commands

For DDEV projects, also create custom commands in `.ddev/commands/`:

#### .ddev/commands/host/dev

```bash
#!/bin/bash
## Description: Start development environment
## Usage: dev
## Example: ddev dev

./dev.sh
```

#### .ddev/commands/host/test

```bash
#!/bin/bash
## Description: Run test suite with test database
## Usage: test
## Example: ddev test

./test.sh
```

#### .ddev/commands/host/staging

```bash
#!/bin/bash
## Description: Build and prepare for staging deployment
## Usage: staging
## Example: ddev staging

./staging.sh
```

#### .ddev/commands/host/production

```bash
#!/bin/bash
## Description: Build and prepare for production deployment
## Usage: production
## Example: ddev production

./production.sh
```

### Command File Permissions

**CRITICAL:** All shell scripts MUST be executable:

```bash
chmod +x dev.sh test.sh staging.sh production.sh
chmod +x .ddev/commands/host/*
```
