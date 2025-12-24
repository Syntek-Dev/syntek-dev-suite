# Environment File Templates

## Overview

Environment file templates for different deployment environments. Each project should have separate `.env.*.example` files committed to the repository with actual `.env.*` files gitignored.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 1.0.0 |
| **Last Updated** | 2025-01 |
| **Stacks** | All (TALL, Django, React, Mobile, Shared-Lib) |

**CRITICAL:** Never commit actual `.env.*` files. Only commit `.env.*.example` files.

---

## Table of Contents

- [Environment File Templates](#environment-file-templates)
  - [Overview](#overview)
  - [Metadata](#metadata)
  - [Table of Contents](#table-of-contents)
  - [Development Environment](#development-environment)
    - [.env.dev.example](#envdevexample)
  - [Test Environment](#test-environment)
    - [.env.test.example](#envtestexample)
  - [Staging Environment](#staging-environment)
    - [.env.staging.example](#envstagingexample)
  - [Production Environment](#production-environment)
    - [.env.production.example](#envproductionexample)
  - [Browser Environment Variables](#browser-environment-variables)
    - [Browser Binary Paths](#browser-binary-paths)
    - [Framework-Specific Variables](#framework-specific-variables)
      - [Laravel Dusk](#laravel-dusk)
      - [Playwright](#playwright)
      - [Cypress](#cypress)
      - [Puppeteer](#puppeteer)
      - [Selenium (Python)](#selenium-python)
    - [CI/CD Environment Variables](#cicd-environment-variables)
    - [Example GitHub Actions Configuration](#example-github-actions-configuration)


---

## Development Environment

### .env.dev.example

```env
# Application
APP_NAME="[Project Name] (Development)"
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost:3000
APP_LOCALE=[locale]
APP_TIMEZONE=[timezone]

# Database
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=[project]_dev
DB_USERNAME=root
DB_PASSWORD=secret

# Cache
CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_CONNECTION=sync

# Mail (use Mailpit/Mailtrap for dev)
MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null

# Logging
LOG_CHANNEL=stack
LOG_LEVEL=debug

# Browser (for E2E tests and debugging)
# CRITICAL: Always use Chrome - paths auto-detected by ./plugins/chrome-tool.py
# Run: ./plugins/chrome-tool.py write
CHROME_PATH=  # Auto-detected, fill in after running chrome-tool.py
BROWSER_BINARY=${CHROME_PATH}
DUSK_CHROME_BINARY=${CHROME_PATH}
CHROME_BINARY=${CHROME_PATH}
PUPPETEER_EXECUTABLE_PATH=${CHROME_PATH}

# External Services (use test/sandbox keys)
# STRIPE_KEY=
# AWS_ACCESS_KEY_ID=
# AWS_SECRET_ACCESS_KEY=
```

---

## Test Environment

### .env.test.example

```env
# Application
APP_NAME="[Project Name] (Testing)"
APP_ENV=testing
APP_DEBUG=true

# Database - MUST use _test suffix
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=[project]_test
DB_USERNAME=root
DB_PASSWORD=secret

# Testing optimisations
CACHE_DRIVER=array
SESSION_DRIVER=array
QUEUE_CONNECTION=sync
MAIL_MAILER=array

# Browser (for E2E tests)
# CRITICAL: Always use Chrome - paths auto-detected by ./plugins/chrome-tool.py
CHROME_PATH=  # Auto-detected, fill in after running chrome-tool.py
BROWSER_BINARY=${CHROME_PATH}
DUSK_CHROME_BINARY=${CHROME_PATH}
CHROME_BINARY=${CHROME_PATH}
PUPPETEER_EXECUTABLE_PATH=${CHROME_PATH}
# For headless testing in CI
CHROME_HEADLESS=true
```

---

## Staging Environment

### .env.staging.example

```env
# Application
APP_NAME="[Project Name] (Staging)"
APP_ENV=staging
APP_DEBUG=true
APP_URL=https://staging.[domain]
APP_LOCALE=[locale]
APP_TIMEZONE=[timezone]

# Database
DB_CONNECTION=mysql
DB_HOST=staging-db.[domain]
DB_PORT=3306
DB_DATABASE=[project]_staging
DB_USERNAME=staging_user
DB_PASSWORD=CHANGE_ME

# Cache
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

# Redis
REDIS_HOST=staging-redis.[domain]
REDIS_PORT=6379

# Mail
MAIL_MAILER=smtp
MAIL_HOST=smtp.[domain]
MAIL_PORT=587
MAIL_USERNAME=
MAIL_PASSWORD=

# Logging
LOG_CHANNEL=stack
LOG_LEVEL=debug

# External Services (use staging/sandbox keys)
# STRIPE_KEY=
# AWS_ACCESS_KEY_ID=
# AWS_SECRET_ACCESS_KEY=
# AWS_DEFAULT_REGION=us-east-1
```

---

## Production Environment

### .env.production.example

```env
# Application
APP_NAME="[Project Name]"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://[domain]
APP_LOCALE=[locale]
APP_TIMEZONE=[timezone]

# Database
DB_CONNECTION=mysql
DB_HOST=production-db.[domain]
DB_PORT=3306
DB_DATABASE=[project]_production
DB_USERNAME=production_user
DB_PASSWORD=CHANGE_ME

# Cache
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

# Redis
REDIS_HOST=production-redis.[domain]
REDIS_PORT=6379
REDIS_PASSWORD=CHANGE_ME

# Mail
MAIL_MAILER=smtp
MAIL_HOST=smtp.[domain]
MAIL_PORT=587
MAIL_USERNAME=
MAIL_PASSWORD=

# Logging
LOG_CHANNEL=stack
LOG_LEVEL=warning

# External Services (use production keys)
# STRIPE_KEY=
# AWS_ACCESS_KEY_ID=
# AWS_SECRET_ACCESS_KEY=
# AWS_DEFAULT_REGION=us-east-1
# SENTRY_DSN=
```

---

## Browser Environment Variables

**CRITICAL:** Always use Chrome for testing, debugging, and E2E tests. Never use Firefox unless explicitly requested.

### Chrome Detection

Use the `chrome-tool.py` plugin to auto-detect Chrome on any platform:

```bash
# Detect Chrome installation
./plugins/chrome-tool.py detect

# Generate .env.chrome with all browser variables
./plugins/chrome-tool.py write
```

### Browser Binary Paths

| Variable | Purpose | Auto-Detection |
|----------|---------|----------------|
| `CHROME_PATH` | **Primary** Chrome binary path | Auto-detected by `chrome-tool.py` |
| `BROWSER_BINARY` | General browser binary | `${CHROME_PATH}` |
| `CHROME_BINARY` | Chrome binary for Selenium | `${CHROME_PATH}` |
| `DUSK_CHROME_BINARY` | Laravel Dusk Chrome binary | `${CHROME_PATH}` |
| `PUPPETEER_EXECUTABLE_PATH` | Puppeteer Chrome binary | `${CHROME_PATH}` |
| `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` | Playwright Chrome binary | `${CHROME_PATH}` |
| `CHROME_HEADLESS` | Enable headless mode | `true` (CI) / `false` (local) |

### Framework-Specific Variables

#### Laravel Dusk
```env
# Uses CHROME_PATH automatically via alias
DUSK_CHROME_BINARY=${CHROME_PATH}
```

#### Playwright
```env
# Playwright uses channel configuration in playwright.config.ts
# Or use environment variable in config:
PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH=${CHROME_PATH}
```

#### Cypress
```env
# Cypress uses config or CLI flags
# Run with: npx cypress run --browser $CHROME_PATH
```

#### Puppeteer
```env
PUPPETEER_EXECUTABLE_PATH=${CHROME_PATH}
PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
```

#### Selenium (Python)
```env
# Python code reads CHROME_PATH:
# options.binary_location = os.environ.get('CHROME_PATH')
CHROME_PATH=  # Auto-detected
```

### CI/CD Environment Variables

For CI/CD pipelines (GitHub Actions, GitLab CI, etc.):

```env
# Enable headless mode
CHROME_HEADLESS=true

# Disable GPU (required for headless)
CHROME_NO_GPU=true

# Disable sandbox (required for Docker/CI)
CHROME_NO_SANDBOX=true

# Set display for headless (Linux)
DISPLAY=:99
```

### Example GitHub Actions Configuration

```yaml
env:
  CHROME_PATH: /usr/bin/google-chrome
  CHROME_BINARY: /usr/bin/google-chrome
  DUSK_CHROME_BINARY: /usr/bin/google-chrome
  PUPPETEER_EXECUTABLE_PATH: /usr/bin/google-chrome
  CHROME_HEADLESS: true
```