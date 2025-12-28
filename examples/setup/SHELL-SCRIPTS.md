# Environment Shell Scripts

## Overview

Environment-specific shell scripts for managing development, testing, staging, and production environments. All projects MUST have these scripts at the root level.

## Metadata

| Property            | Value                                         |
| ------------------- | --------------------------------------------- |
| **Example Version** | 1.0.0                                         |
| **Last Updated**    | 2025-01                                       |
| **Stacks**          | All (TALL, Django, React, Mobile, Shared-Lib) |

**CRITICAL:** All shell scripts MUST be executable:
```bash
chmod +x dev.sh test.sh staging.sh production.sh
```

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [dev.sh](#devsh)
- [test.sh](#testsh)
- [staging.sh](#stagingsh)
- [production.sh](#productionsh)
- [Shared Library Scripts](#shared-library-scripts)
  - [dev.sh (Shared Lib)](#devsh-shared-lib)
  - [test.sh (Shared Lib)](#testsh-shared-lib)
  - [staging.sh (Shared Lib)](#stagingsh-shared-lib)
  - [production.sh (Shared Lib)](#productionsh-shared-lib)

---

## dev.sh

```bash
#!/bin/bash
set -e

echo "Starting development environment..."

# Detect container type
if [ -f ".ddev/config.yaml" ]; then
    echo "Using DDEV..."
    ddev start
    ddev composer install 2>/dev/null || true
    ddev npm install 2>/dev/null || true
    echo "Development environment ready!"
    echo "Access your site at: $(ddev describe -j | jq -r '.raw.primary_url')"
elif [ -f "docker-compose.yml" ]; then
    echo "Using Docker Compose..."
    docker-compose up -d
    echo "Development environment ready!"
elif [ -f "Dockerfile" ]; then
    echo "Using Docker..."
    docker build -t app-dev .
    docker run -d -p 3000:3000 --env-file .env.dev app-dev
    echo "Development environment ready!"
else
    echo "No container configuration found. Running locally..."
    if [ -f "package.json" ]; then
        npm install && npm run dev
    elif [ -f "composer.json" ]; then
        composer install && php artisan serve
    elif [ -f "requirements.txt" ]; then
        pip install -r requirements.txt && python manage.py runserver
    fi
fi
```

---

## test.sh

```bash
#!/bin/bash
set -e

echo "Running test suite..."

# Load test environment
if [ -f ".env.test" ]; then
    export $(cat .env.test | grep -v '^#' | xargs)
fi

# Detect container type and run tests
if [ -f ".ddev/config.yaml" ]; then
    echo "Using DDEV..."
    ddev exec php artisan test --env=testing
elif [ -f "docker-compose.yml" ]; then
    echo "Using Docker Compose..."
    docker-compose -f docker-compose.yml -f docker-compose.test.yml run --rm app npm test
elif [ -f "package.json" ]; then
    npm test
elif [ -f "composer.json" ]; then
    ./vendor/bin/pest
elif [ -f "requirements.txt" ]; then
    pytest
fi

echo "Tests complete!"
```

---

## staging.sh

```bash
#!/bin/bash
set -e

echo "Deploying to staging..."

# Load staging environment
if [ -f ".env.staging" ]; then
    export $(cat .env.staging | grep -v '^#' | xargs)
fi

# Run pre-deployment checks
echo "Running pre-deployment checks..."

# Run tests
if [ -f "package.json" ]; then
    npm test
elif [ -f "composer.json" ]; then
    ./vendor/bin/pest --stop-on-failure
elif [ -f "requirements.txt" ]; then
    pytest
fi

# Build
echo "Building for staging..."
if [ -f "package.json" ]; then
    npm run build
elif [ -f "composer.json" ]; then
    php artisan config:cache
    php artisan route:cache
    php artisan view:cache
fi

# Deploy (customise based on your deployment target)
echo "Deploying..."
# git push origin staging
# or: ./deploy-staging-custom.sh

echo "Staging deployment complete!"
```

---

## production.sh

```bash
#!/bin/bash
set -e

echo "Deploying to production..."

# Safety check
read -p "Are you sure you want to deploy to PRODUCTION? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "Deployment cancelled."
    exit 1
fi

# Load production environment
if [ -f ".env.production" ]; then
    export $(cat .env.production | grep -v '^#' | xargs)
fi

# Run pre-deployment checks
echo "Running pre-deployment checks..."

# Run full test suite
if [ -f "package.json" ]; then
    npm test
elif [ -f "composer.json" ]; then
    ./vendor/bin/pest
elif [ -f "requirements.txt" ]; then
    pytest
fi

# Build for production
echo "Building for production..."
if [ -f "package.json" ]; then
    npm run build
elif [ -f "composer.json" ]; then
    php artisan config:cache
    php artisan route:cache
    php artisan view:cache
    php artisan optimize
fi

# Deploy (customise based on your deployment target)
echo "Deploying..."
# git push origin main
# or: ./deploy-production-custom.sh

echo "Production deployment complete!"
```

---

## Shared Library Scripts

For NPM package projects (stack-shared-lib), use these simplified scripts that focus on building/publishing rather than deployment.

### dev.sh (Shared Lib)

```bash
#!/bin/bash
set -e

echo "Starting development environment..."

# Install dependencies if needed
if [ ! -d "node_modules" ]; then
    npm install
fi

# Start in watch mode
echo "Starting watch mode..."
echo "Run 'npm run storybook:web' in another terminal for component development"
npm run dev
```

### test.sh (Shared Lib)

```bash
#!/bin/bash
set -e

echo "Running test suite..."

npm test

echo "Running type check..."
npm run type-check

echo "Running linter..."
npm run lint

echo "All checks complete!"
```

### staging.sh (Shared Lib)

```bash
#!/bin/bash
set -e

echo "Building for staging (prerelease)..."

# Run tests first
./test.sh

# Build the package
npm run build

# Bump prerelease version
npm version prerelease --preid=staging

echo "Staging build complete!"
echo "Run 'npm publish --tag staging' to publish prerelease"
```

### production.sh (Shared Lib)

```bash
#!/bin/bash
set -e

echo "Building for production..."

read -p "Are you sure you want to build for PRODUCTION? (yes/no): " confirm
if [ "$confirm" != "yes" ]; then
    echo "Build cancelled."
    exit 1
fi

# Run tests first
./test.sh

# Build the package
npm run build

echo "Production build complete!"
echo "Run 'npm version <patch|minor|major>' then 'npm publish' to release"
```