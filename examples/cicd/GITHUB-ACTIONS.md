# GitHub Actions Workflows

## Overview

GitHub Actions workflow templates for CI/CD pipelines across multiple technology stacks. Includes linting, testing, building, and deployment workflows for staging and production environments.

This document provides comprehensive examples for:
- **TALL Stack**: Laravel 12.x with PHP 8.4, PHPUnit, Pest, Laravel Pint
- **Django/Wagtail**: Django 6.x with Python 3.14, pytest, ruff, mypy
- **React/Next.js**: Next.js 16.x with Node.js 24.x, Jest, ESLint, TypeScript
- **React Native**: React Native 0.83.x with Expo EAS Build

## Metadata

| Property            | Value                                                                            |
| ------------------- | -------------------------------------------------------------------------------- |
| **Example Version** | 2.0.0                                                                            |
| **Last Updated**    | 2025-12                                                                          |
| **GitHub Actions**  | v4 actions                                                                       |
| **Stacks**          | TALL (Laravel 12.x), Django/Wagtail (6.x), Next.js (16.x), React Native (0.83.x) |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Standard CI Pipeline](#standard-ci-pipeline)
  - [.github/workflows/ci.yml](#githubworkflowsciyml)
- [Staging Deployment](#staging-deployment)
  - [.github/workflows/deploy-staging.yml](#githubworkflowsdeploy-stagingyml)
- [Production Deployment](#production-deployment)
  - [.github/workflows/deploy-production.yml](#githubworkflowsdeploy-productionyml)
- [DDEV CI Workflow](#ddev-ci-workflow)
  - [.github/workflows/ci-ddev.yml](#githubworkflowsci-ddevyml)
- [Required Secrets](#required-secrets)
  - [GitHub Repository Secrets](#github-repository-secrets)
  - [GitHub Environment Variables](#github-environment-variables)


## Standard CI Pipeline

### .github/workflows/ci.yml

```yaml
# CI Pipeline
# Runs linting, tests, and builds on push and pull requests.
# Uploads coverage reports to Codecov.

name: CI Pipeline

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main, staging]

env:
  NODE_VERSION: '20'  # or appropriate version

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: false

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
          retention-days: 7
```

---

## Staging Deployment

### .github/workflows/deploy-staging.yml

```yaml
# Staging Deployment
# Automatically deploys to staging environment on push to staging branch.
# Requires staging environment to be configured in GitHub.

name: Deploy to Staging

on:
  push:
    branches: [staging]

concurrency:
  group: staging-deployment
  cancel-in-progress: true

jobs:
  deploy:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build for staging
        run: npm run build
        env:
          NODE_ENV: staging
          VITE_API_URL: ${{ vars.API_URL }}

      - name: Deploy to staging server
        # Replace with platform-specific deployment step
        run: |
          echo "Deploying to staging..."
          # Add deployment commands here

      - name: Notify on success
        if: success()
        run: echo "Staging deployment successful"

      - name: Notify on failure
        if: failure()
        run: echo "Staging deployment failed"
```

---

## Production Deployment

### .github/workflows/deploy-production.yml

```yaml
# Production Deployment
# Deploys to production environment on push to main branch.
# Requires manual approval in GitHub environment settings.

name: Deploy to Production

on:
  push:
    branches: [main]
  workflow_dispatch:  # Allow manual trigger

concurrency:
  group: production-deployment
  cancel-in-progress: false  # Never cancel production deployments

jobs:
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    environment: production  # Requires approval if configured
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build for production
        run: npm run build
        env:
          NODE_ENV: production
          VITE_API_URL: ${{ vars.API_URL }}

      - name: Deploy to production server
        # Replace with platform-specific deployment step
        run: |
          echo "Deploying to production..."
          # Add deployment commands here

      - name: Create release tag
        if: success()
        run: |
          git tag -a "release-$(date +%Y%m%d-%H%M%S)" -m "Production release"
          git push origin --tags

      - name: Notify on success
        if: success()
        run: echo "Production deployment successful"

      - name: Notify on failure
        if: failure()
        run: echo "Production deployment failed"
```

---

## DDEV CI Workflow

### .github/workflows/ci-ddev.yml

```yaml
# CI with DDEV
# Uses DDEV for consistent development environment in CI.
# Suitable for PHP/Laravel/WordPress projects.

name: CI with DDEV

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main, staging]

jobs:
  test:
    name: Test with DDEV
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install DDEV
        run: |
          curl -fsSL https://raw.githubusercontent.com/ddev/ddev/master/scripts/install_ddev.sh | bash

      - name: Start DDEV
        run: ddev start

      - name: Install Composer dependencies
        run: ddev composer install

      - name: Install NPM dependencies
        run: ddev npm install

      - name: Run database migrations
        run: ddev exec php artisan migrate --seed

      - name: Run PHP tests
        run: ddev exec php artisan test

      - name: Run JavaScript tests
        run: ddev npm run test

      - name: Stop DDEV
        if: always()
        run: ddev stop
```

---

## Required Secrets

### GitHub Repository Secrets

| Secret Name                 | Description          | Required For       |
| --------------------------- | -------------------- | ------------------ |
| `CODECOV_TOKEN`             | Codecov upload token | Coverage reporting |
| `AWS_ACCESS_KEY_ID`         | AWS access key       | AWS deployments    |
| `AWS_SECRET_ACCESS_KEY`     | AWS secret key       | AWS deployments    |
| `DIGITALOCEAN_ACCESS_TOKEN` | DO API token         | DO deployments     |
| `DROPLET_SSH_KEY`           | SSH private key      | Server deployments |
| `DROPLET_HOST`              | Server hostname      | Server deployments |
| `DROPLET_USER`              | SSH username         | Server deployments |

### GitHub Environment Variables

| Variable Name    | Description         | Environment         |
| ---------------- | ------------------- | ------------------- |
| `API_URL`        | Backend API URL     | staging, production |
| `AWS_REGION`     | AWS region          | staging, production |
| `S3_BUCKET`      | S3 bucket name      | staging, production |
| `ECR_REPOSITORY` | ECR repository name | staging, production |
| `ECS_SERVICE`    | ECS service name    | staging, production |
| `ECS_CLUSTER`    | ECS cluster name    | staging, production |
