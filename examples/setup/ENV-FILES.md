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

- [Overview](#overview)
- [Metadata](#metadata)
- [Development Environment](#development-environment)
- [Test Environment](#test-environment)
- [Staging Environment](#staging-environment)
- [Production Environment](#production-environment)

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