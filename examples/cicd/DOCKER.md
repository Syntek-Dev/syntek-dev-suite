# Docker Configuration

## Overview

Docker configuration templates for containerised applications. Includes multi-stage builds for production and Docker Compose for local development across all supported stacks.

**Note:** React Native applications typically do not use Docker as they are mobile apps compiled for iOS/Android. For React Native, use the platform-specific build tools (Xcode, Android Studio) and Expo CLI.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Docker** | 27.x |
| **Docker Compose** | 3.8+ |
| **Stacks** | TALL (Laravel 12/PHP 8.4), Django 6/Python 3.14, Next.js 16/Node 24 |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [TALL Stack (Laravel/PHP)](#tall-stack-laravelphp)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [Docker Compose with Multiple Services](#docker-compose-with-multiple-services)


## TALL Stack (Laravel/PHP)

### Laravel Dockerfile

This multi-stage Dockerfile is optimised for Laravel 12.x with PHP 8.4 and production deployment.

```dockerfile
# Laravel 12.x / PHP 8.4 Dockerfile
# Multi-stage build for production deployment
# Stage 1: Composer dependencies
# Stage 2: Production runtime with optimisations

# ================================
# Composer Dependencies Stage
# ================================
FROM composer:latest AS composer-builder

WORKDIR /app

# Copy composer files for dependency resolution
COPY composer.json composer.lock ./

# Install Composer dependencies (production only, optimised autoloader)
RUN composer install \
    --no-dev \
    --no-scripts \
    --no-interaction \
    --prefer-dist \
    --optimize-autoloader

# ================================
# Production Runtime Stage
# ================================
FROM php:8.4-fpm-alpine AS production

# Install system dependencies required for PHP extensions and Laravel
RUN apk add --no-cache \
    git \
    curl \
    libpng-dev \
    oniguruma-dev \
    libxml2-dev \
    zip \
    unzip \
    libzip-dev \
    icu-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    mysql-client \
    nginx \
    supervisor

# Configure and install PHP extensions required by Laravel
# gd: Image manipulation
# pdo_mysql: Database connectivity
# mbstring: Multibyte string handling
# exif: Image metadata
# pcntl: Process control
# bcmath: Arbitrary precision mathematics
# zip: Archive handling
# intl: Internationalisation
# opcache: Performance optimisation
RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install \
        pdo_mysql \
        mbstring \
        exif \
        pcntl \
        bcmath \
        gd \
        zip \
        intl \
        opcache

# Install Redis extension for caching and queues
RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS && \
    pecl install redis-6.1.0 && \
    docker-php-ext-enable redis && \
    apk del .build-deps

# Install Composer from official image
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configure PHP for production with OPcache optimisations
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY docker/php/opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY docker/php/laravel.ini /usr/local/etc/php/conf.d/laravel.ini

# Set working directory
WORKDIR /var/www/html

# Copy application files with proper ownership
COPY --chown=www-data:www-data . .

# Copy vendor dependencies from composer-builder stage
COPY --from=composer-builder --chown=www-data:www-data /app/vendor ./vendor

# Run Laravel optimisations for production
RUN php artisan config:cache && \
    php artisan route:cache && \
    php artisan view:cache && \
    php artisan event:cache

# Set proper permissions for Laravel storage and cache
RUN chown -R www-data:www-data \
    /var/www/html/storage \
    /var/www/html/bootstrap/cache && \
    chmod -R 775 \
    /var/www/html/storage \
    /var/www/html/bootstrap/cache

# Configure Nginx for Laravel
COPY docker/nginx/laravel.conf /etc/nginx/http.d/default.conf

# Configure Supervisor to manage PHP-FPM and Nginx
COPY docker/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Switch to non-root user for security
USER www-data

# Expose HTTP port
EXPOSE 8080

# Health check endpoint
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Start Supervisor to manage services
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

### docker/php/opcache.ini

```ini
; OPcache configuration for Laravel production
; Improves performance by caching compiled PHP code in memory

opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000
opcache.revalidate_freq=0
opcache.validate_timestamps=0
opcache.save_comments=1
opcache.fast_shutdown=1
```

### docker/php/laravel.ini

```ini
; PHP configuration optimised for Laravel

; Memory limits
memory_limit=512M

; Upload limits
upload_max_filesize=100M
post_max_size=100M

; Execution time
max_execution_time=300

; Session configuration
session.cookie_httponly=1
session.cookie_secure=1
session.cookie_samesite=Strict

; Error handling (production)
display_errors=Off
log_errors=On
error_log=/var/log/php/error.log
```

### docker/nginx/laravel.conf

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    # Laravel front controller pattern
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP processing
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    # Deny access to hidden files
    location ~ /\.(?!well-known).* {
        deny all;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### docker/supervisor/supervisord.conf

```ini
[supervisord]
nodaemon=true
user=root
logfile=/var/log/supervisor/supervisord.log
pidfile=/var/run/supervisord.pid

[program:php-fpm]
command=/usr/local/sbin/php-fpm --nodaemonize
autostart=true
autorestart=true
priority=5
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:nginx]
command=/usr/sbin/nginx -g 'daemon off;'
autostart=true
autorestart=true
priority=10
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/worker.log
stopwaitsecs=3600
```

### Laravel Docker Compose

Docker Compose configuration for Laravel local development with MariaDB 12.x, Redis, and Mailpit.

```yaml
# Laravel TALL Stack Docker Compose
# Services: Laravel app, MariaDB, Redis, Mailpit
# Version: 3.8

version: '3.8'

services:
  # Laravel Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: laravel-app
    restart: unless-stopped
    working_dir: /var/www/html
    ports:
      - "8080:8080"
    env_file:
      - .env.dev
    environment:
      DB_CONNECTION: mysql
      DB_HOST: mariadb
      DB_PORT: 3306
      DB_DATABASE: laravel_dev
      REDIS_HOST: redis
      REDIS_PORT: 6379
      MAIL_HOST: mailpit
      MAIL_PORT: 1025
    volumes:
      - ./:/var/www/html
      - ./docker/php/laravel.ini:/usr/local/etc/php/conf.d/laravel.ini
    depends_on:
      mariadb:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - laravel-network

  # MariaDB 12.x Database
  mariadb:
    image: mariadb:12.0
    container_name: laravel-mariadb
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: laravel_dev
      MARIADB_USER: laravel
      MARIADB_PASSWORD: secret
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./docker/mariadb/my.cnf:/etc/mysql/conf.d/my.cnf:ro
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - laravel-network

  # Redis for caching and queues
  redis:
    image: redis:7-alpine
    container_name: laravel-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - laravel-network

  # Mailpit for email testing
  mailpit:
    image: axllent/mailpit:latest
    container_name: laravel-mailpit
    restart: unless-stopped
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI
    networks:
      - laravel-network

  # Laravel Scheduler (runs cron jobs)
  scheduler:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: laravel-scheduler
    restart: unless-stopped
    working_dir: /var/www/html
    env_file:
      - .env.dev
    volumes:
      - ./:/var/www/html
    depends_on:
      - mariadb
      - redis
    command: php artisan schedule:work
    networks:
      - laravel-network

volumes:
  mariadb_data:
    driver: local
  redis_data:
    driver: local

networks:
  laravel-network:
    driver: bridge
```

### docker/mariadb/my.cnf

```ini
[mysqld]
# General configuration
default_storage_engine=InnoDB
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# Performance optimisations
innodb_buffer_pool_size=1G
innodb_log_file_size=256M
innodb_flush_log_at_trx_commit=2
innodb_flush_method=O_DIRECT

# Query cache (disabled in MariaDB 10.5+)
query_cache_type=0
query_cache_size=0

# Connection settings
max_connections=200
```

---

## Django/Wagtail Stack

### Django Dockerfile

This multi-stage Dockerfile is optimised for Django 6.x with Python 3.14 and production deployment.

```dockerfile
# Django 6.x / Python 3.14 Dockerfile
# Multi-stage build for production deployment
# Stage 1: Python dependencies
# Stage 2: Production runtime with optimisations

# ================================
# Python Dependencies Stage
# ================================
FROM python:3.14-slim AS python-builder

# Set environment variables for Python optimisation
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

# Install system dependencies required for Python packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    libjpeg-dev \
    libwebp-dev \
    libpng-dev \
    libfreetype6-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements file for dependency installation
COPY requirements.txt requirements-prod.txt ./

# Install Python dependencies into a virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --upgrade pip setuptools wheel && \
    pip install -r requirements-prod.txt

# ================================
# Production Runtime Stage
# ================================
FROM python:3.14-slim AS production

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH" \
    DJANGO_SETTINGS_MODULE=config.settings.production

# Install runtime dependencies only (no build tools)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    libjpeg62-turbo \
    libwebp7 \
    libpng16-16 \
    libfreetype6 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user for security
RUN useradd -m -u 1000 django && \
    mkdir -p /app /app/staticfiles /app/media && \
    chown -R django:django /app

WORKDIR /app

# Copy virtual environment from builder stage
COPY --from=python-builder --chown=django:django /opt/venv /opt/venv

# Copy application code with proper ownership
COPY --chown=django:django . .

# Switch to non-root user
USER django

# Collect static files for production
RUN python manage.py collectstatic --noinput --clear

# Expose application port
EXPOSE 8000

# Health check endpoint
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8000/health/ || exit 1

# Start Gunicorn WSGI server
# --workers: Number of worker processes (recommended: 2-4 x CPU cores)
# --worker-class: Worker type (sync for CPU-bound, gevent for I/O-bound)
# --timeout: Worker timeout in seconds
# --bind: Address and port to bind
# --access-logfile: Access log location (- for stdout)
# --error-logfile: Error log location (- for stderr)
CMD ["gunicorn", \
     "--workers=4", \
     "--worker-class=gthread", \
     "--threads=2", \
     "--timeout=60", \
     "--bind=0.0.0.0:8000", \
     "--access-logfile=-", \
     "--error-logfile=-", \
     "--log-level=info", \
     "config.wsgi:application"]
```

### requirements-prod.txt

```txt
# Django 6.x Production Requirements
# Python 3.14

# Core Framework
Django>=6.0,<6.1
wagtail>=6.4,<6.5

# Database
psycopg[binary]>=3.2,<4.0
psycopg-pool>=3.2,<4.0

# WSGI Server
gunicorn>=22.0,<23.0

# Image Processing
Pillow>=11.0,<12.0
Willow>=1.8,<2.0

# Caching and Session
redis>=5.2,<6.0
django-redis>=5.4,<6.0

# Storage
boto3>=1.35,<2.0
django-storages>=1.14,<2.0

# Security
django-csp>=3.8,<4.0
django-cors-headers>=4.6,<5.0

# Monitoring and Performance
sentry-sdk>=2.19,<3.0
django-debug-toolbar>=4.4,<5.0  # Development only

# Utilities
python-dotenv>=1.0,<2.0
celery>=5.4,<6.0
django-celery-beat>=2.7,<3.0
```

### Django Docker Compose

Docker Compose configuration for Django/Wagtail local development with PostgreSQL 18.x, Redis, and Celery.

```yaml
# Django/Wagtail Stack Docker Compose
# Services: Django app, PostgreSQL, Redis, Celery, Mailpit
# Version: 3.8

version: '3.8'

services:
  # Django/Wagtail Application
  web:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: django-web
    restart: unless-stopped
    working_dir: /app
    command: >
      sh -c "python manage.py migrate &&
             python manage.py collectstatic --noinput &&
             gunicorn config.wsgi:application --bind 0.0.0.0:8000 --workers 4"
    ports:
      - "8000:8000"
    env_file:
      - .env.dev
    environment:
      DJANGO_SETTINGS_MODULE: config.settings.development
      DATABASE_URL: postgres://django:secret@postgres:5432/django_dev
      REDIS_URL: redis://redis:6379/0
      CELERY_BROKER_URL: redis://redis:6379/1
    volumes:
      - ./:/app
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - django-network

  # PostgreSQL 18.x Database
  postgres:
    image: postgres:18-alpine
    container_name: django-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: django_dev
      POSTGRES_USER: django
      POSTGRES_PASSWORD: secret
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=en_GB.UTF-8"
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/postgresql.conf:/etc/postgresql/postgresql.conf:ro
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U django -d django_dev"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - django-network

  # Redis for caching and Celery broker
  redis:
    image: redis:7-alpine
    container_name: django-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - django-network

  # Celery Worker for async tasks
  celery:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: django-celery
    restart: unless-stopped
    working_dir: /app
    command: celery -A config worker --loglevel=info --concurrency=4
    env_file:
      - .env.dev
    environment:
      DJANGO_SETTINGS_MODULE: config.settings.development
      DATABASE_URL: postgres://django:secret@postgres:5432/django_dev
      CELERY_BROKER_URL: redis://redis:6379/1
    volumes:
      - ./:/app
      - media_volume:/app/media
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - django-network

  # Celery Beat for scheduled tasks
  celery-beat:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: django-celery-beat
    restart: unless-stopped
    working_dir: /app
    command: celery -A config beat --loglevel=info --scheduler django_celery_beat.schedulers:DatabaseScheduler
    env_file:
      - .env.dev
    environment:
      DJANGO_SETTINGS_MODULE: config.settings.development
      DATABASE_URL: postgres://django:secret@postgres:5432/django_dev
      CELERY_BROKER_URL: redis://redis:6379/1
    volumes:
      - ./:/app
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - django-network

  # Mailpit for email testing
  mailpit:
    image: axllent/mailpit:latest
    container_name: django-mailpit
    restart: unless-stopped
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI
    networks:
      - django-network

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  static_volume:
    driver: local
  media_volume:
    driver: local

networks:
  django-network:
    driver: bridge
```

### docker/postgres/postgresql.conf

```conf
# PostgreSQL 18.x Configuration for Django/Wagtail
# Optimised for development and moderate production workloads

# Connection Settings
max_connections = 200
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 2621kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB

# Locale and Encoding
lc_messages = 'en_GB.UTF-8'
lc_monetary = 'en_GB.UTF-8'
lc_numeric = 'en_GB.UTF-8'
lc_time = 'en_GB.UTF-8'

# Logging
log_timezone = 'Europe/London'
timezone = 'Europe/London'
log_line_prefix = '%m [%p] %q%u@%d '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
```

---

## Docker Compose with Multiple Services

### docker-compose.yml (Full Stack)

```yaml
# Docker Compose for Full Stack Development
# Includes app, database, Redis, and worker

version: '3.8'

services:
  # Main application
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: development  # Use development stage
    ports:
      - "3000:3000"
    env_file:
      - .env.dev
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - app-network
    command: npm run dev

  # Background worker
  worker:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    env_file:
      - .env.dev
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - app-network
    command: npm run worker

  # PostgreSQL database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: app_dev
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis for caching and queues
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Mailpit for email testing
  mailpit:
    image: axllent/mailpit:latest
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI
    networks:
      - app-network

  # MinIO for S3-compatible storage
  minio:
    image: minio/minio:latest
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:
  minio_data:

networks:
  app-network:
    driver: bridge
```

### .dockerignore

```
# Git
.git
.gitignore

# Node
node_modules
npm-debug.log

# Build artifacts
dist
build
coverage

# Environment files (except examples)
.env
.env.*
!.env.example
!.env.*.example

# IDE
.idea
.vscode
*.swp

# OS
.DS_Store
Thumbs.db

# Docker
Dockerfile*
docker-compose*

# Documentation
*.md
docs/

# Tests
__tests__
*.test.js
*.spec.js
```
