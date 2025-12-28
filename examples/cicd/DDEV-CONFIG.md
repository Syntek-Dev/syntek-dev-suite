# DDEV Configuration

## Overview

DDEV configuration templates for local development. DDEV provides consistent development environments for PHP, Laravel, WordPress, Drupal, and TYPO3 projects.

## Metadata

| Property            | Value                             |
| ------------------- | --------------------------------- |
| **Example Version** | 2.0.0                             |
| **Last Updated**    | 2025-12                           |
| **DDEV**            | 1.23.x                            |
| **PHP**             | 8.4                               |
| **Laravel**         | 12.x                              |
| **MariaDB**         | 12.x                              |
| **Stacks**          | TALL (Laravel), WordPress, Drupal |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Basic DDEV Configuration](#basic-ddev-configuration)
  - [.ddev/config.yaml (Laravel)](#ddevconfigyaml-laravel)
  - [.ddev/config.yaml (WordPress)](#ddevconfigyaml-wordpress)
- [DDEV with Redis](#ddev-with-redis)
  - [.ddev/docker-compose.redis.yaml](#ddevdocker-composeredisyaml)
  - [.ddev/docker-compose.mailpit.yaml](#ddevdocker-composemailpityaml)
- [DDEV Custom Commands](#ddev-custom-commands)
  - [.ddev/commands/host/deploy-staging](#ddevcommandshostdeploy-staging)
  - [.ddev/commands/host/fresh](#ddevcommandshostfresh)
  - [.ddev/commands/host/test](#ddevcommandshosttest)
  - [.ddev/commands/host/lint](#ddevcommandshostlint)
- [DDEV for Different Project Types](#ddev-for-different-project-types)
  - [Django Project](#django-project)
  - [React/Node Project](#reactnode-project)
  - [Drupal Project](#drupal-project)


## Basic DDEV Configuration

### .ddev/config.yaml (Laravel)

```yaml
# DDEV Configuration for Laravel 12.x
# Provides local development environment with PHP 8.4, MariaDB 12.x, and Node.js
# This configuration ensures consistency across development teams

name: my-laravel-project
type: laravel
docroot: public
php_version: "8.4"  # Laravel 12.x requires PHP 8.4+
webserver_type: nginx-fpm
database:
  type: mariadb
  version: "12"  # MariaDB 12.x for improved performance and compatibility
router_http_port: "80"  # HTTP port for local development
router_https_port: "443"  # HTTPS port with automatic SSL certificates
xdebug_enabled: false  # Disable Xdebug by default for performance; enable when debugging
additional_hostnames: []  # Add extra hostnames if needed (e.g., api.myproject.ddev.site)
additional_fqdns: []  # Add fully qualified domain names if required
use_dns_when_possible: true  # Use DNS resolution for better performance
composer_version: "2"  # Composer 2.x for faster dependency resolution
nodejs_version: "20"  # Node.js 20 LTS for frontend asset compilation

# Post-start hooks to set up the project automatically
# These commands run after DDEV starts to ensure the environment is ready
hooks:
  post-start:
    - exec: composer install  # Install PHP dependencies
    - exec: npm install  # Install Node.js dependencies
    - exec: php artisan key:generate --force  # Generate application key
    - exec: php artisan migrate --seed  # Run database migrations and seeders
```

### .ddev/config.yaml (WordPress)

```yaml
# DDEV Configuration for WordPress
# Provides local development environment with PHP and MariaDB

name: my-wordpress-project
type: wordpress
docroot: ""
php_version: "8.2"
webserver_type: nginx-fpm
database:
  type: mariadb
  version: "10.11"
router_http_port: "80"
router_https_port: "443"
xdebug_enabled: false
composer_version: "2"
nodejs_version: "20"

hooks:
  post-start:
    - exec: wp core install --url=https://my-wordpress-project.ddev.site --title="Development Site" --admin_user=admin --admin_password=admin --admin_email=admin@example.com --skip-email || true
```

---

## DDEV with Redis

### .ddev/docker-compose.redis.yaml

```yaml
# Redis service for DDEV
# Provides Redis caching and session storage

version: '3.6'
services:
  redis:
    container_name: ddev-${DDEV_SITENAME}-redis
    image: redis:7-alpine
    restart: "no"
    labels:
      com.ddev.site-name: ${DDEV_SITENAME}
      com.ddev.approot: $DDEV_APPROOT
    expose:
      - "6379"
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  redis-data:
```

### .ddev/docker-compose.mailpit.yaml

```yaml
# Mailpit for email testing in DDEV
# Catches all outgoing emails for development

version: '3.6'
services:
  mailpit:
    container_name: ddev-${DDEV_SITENAME}-mailpit
    image: axllent/mailpit:latest
    restart: "no"
    labels:
      com.ddev.site-name: ${DDEV_SITENAME}
      com.ddev.approot: $DDEV_APPROOT
    expose:
      - "1025"  # SMTP
      - "8025"  # Web UI
    environment:
      - MP_SMTP_AUTH_ACCEPT_ANY=true
      - MP_SMTP_AUTH_ALLOW_INSECURE=true

# Add to your .env:
# MAIL_MAILER=smtp
# MAIL_HOST=mailpit
# MAIL_PORT=1025
```

---

## DDEV Custom Commands

### .ddev/commands/host/deploy-staging

```bash
#!/bin/bash

## Description: Deploy to staging environment
## Usage: deploy-staging
## Example: ddev deploy-staging

set -e

echo "Deploying to staging..."

# Put the site into maintenance mode
ddev exec php artisan down

# Pull latest changes
ddev exec git pull origin staging

# Install dependencies (no dev dependencies for production)
ddev exec composer install --no-dev --optimize-autoloader

# Run database migrations
ddev exec php artisan migrate --force

# Clear and cache configurations
ddev exec php artisan config:cache
ddev exec php artisan route:cache
ddev exec php artisan view:cache

# Build frontend assets
ddev npm run build

# Bring the site back up
ddev exec php artisan up

echo "Deployment complete!"
```

### .ddev/commands/host/fresh

```bash
#!/bin/bash

## Description: Fresh install - reset database and seed
## Usage: fresh
## Example: ddev fresh

set -e

echo "Resetting database..."

# Drop all tables and re-run migrations with seeding
ddev exec php artisan migrate:fresh --seed

# Clear all caches
ddev exec php artisan cache:clear
ddev exec php artisan config:clear
ddev exec php artisan route:clear
ddev exec php artisan view:clear

echo "Fresh install complete!"
```

### .ddev/commands/host/test

```bash
#!/bin/bash

## Description: Run all tests
## Usage: test [filter]
## Example: ddev test
## Example: ddev test UserTest

set -e

FILTER=${1:-}

if [ -n "$FILTER" ]; then
    echo "Running tests matching: $FILTER"
    ddev exec php artisan test --filter="$FILTER"
else
    echo "Running all tests..."
    ddev exec php artisan test
fi
```

### .ddev/commands/host/lint

```bash
#!/bin/bash

## Description: Run linting and code style checks
## Usage: lint [--fix]
## Example: ddev lint
## Example: ddev lint --fix

set -e

FIX=${1:-}

echo "Running PHP CS Fixer..."
if [ "$FIX" = "--fix" ]; then
    ddev exec ./vendor/bin/pint
else
    ddev exec ./vendor/bin/pint --test
fi

echo "Running ESLint..."
if [ "$FIX" = "--fix" ]; then
    ddev npm run lint:fix
else
    ddev npm run lint
fi

echo "Linting complete!"
```

---

## DDEV for Different Project Types

### Django Project

```yaml
# .ddev/config.yaml for Django
name: my-django-project
type: python
docroot: ""
webserver_type: nginx-gunicorn
database:
  type: postgres
  version: "15"
router_http_port: "80"
router_https_port: "443"

hooks:
  post-start:
    - exec: pip install -r requirements.txt
    - exec: python manage.py migrate
    - exec: python manage.py collectstatic --noinput
```

### React/Node Project

```yaml
# .ddev/config.yaml for React/Node
name: my-react-project
type: nodejs
docroot: ""
webserver_type: nginx-fpm
nodejs_version: "20"
router_http_port: "80"
router_https_port: "443"

hooks:
  post-start:
    - exec: npm install
```

### Drupal Project

```yaml
# .ddev/config.yaml for Drupal
name: my-drupal-project
type: drupal
docroot: web
php_version: "8.3"
webserver_type: nginx-fpm
database:
  type: mariadb
  version: "10.11"
router_http_port: "80"
router_https_port: "443"
composer_version: "2"

hooks:
  post-start:
    - exec: composer install
    - exec: drush updb -y
    - exec: drush cr
```
