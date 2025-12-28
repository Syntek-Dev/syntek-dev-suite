# Git Workflows Examples

## Overview

Standardised Git workflows and automation for consistent version control across all technology stacks.

## Table of Contents

- [Overview](#overview)
- [Table of Contents](#table-of-contents)
- [Core Branches](#core-branches)
- [Branch Naming Conventions](#branch-naming-conventions)
  - [User Story Branches](#user-story-branches)
- [Branch Flow](#branch-flow)
  - [Standard Flow (User Story → Production)](#standard-flow-user-story--production)
  - [Flow Rules](#flow-rules)
- [Commit Message Standards](#commit-message-standards)
- [Pull Request Templates](#pull-request-templates)
  - [PR Title Format](#pr-title-format)
  - [User Story → Testing](#user-story--testing)
  - [Testing → Dev](#testing--dev)
  - [Dev → Staging](#dev--staging)
  - [Staging → Main (Client Accepted)](#staging--main-client-accepted)
- [Git Hooks](#git-hooks)
  - [Pre-commit Hooks](#pre-commit-hooks)
  - [Commit Message Hooks](#commit-message-hooks)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [React/Next.js Stack](#reactnextjs-stack)
- [React Native Stack](#react-native-stack)


## Core Branches

Every project MUST have these protected branches:

| Branch             | Purpose                       | Protected | Deploy Target |
| ------------------ | ----------------------------- | --------- | ------------- |
| `main` or `master` | Production-ready code         | Yes       | Production    |
| `staging`          | Client review and acceptance  | Yes       | Staging       |
| `dev`              | Integration and final testing | Yes       | Development   |
| `testing`          | QA and automated testing      | Yes       | Testing       |

## Branch Naming Conventions

### User Story Branches

All feature work uses user story branches:

```
us<number>/<short-description>
```

| Type       | Pattern                               | Example                     |
| ---------- | ------------------------------------- | --------------------------- |
| User Story | `us<number>/<description>`            | `us001/user-authentication` |
| Hotfix     | `hotfix/<issue-number>-<description>` | `hotfix/123-login-crash`    |

**Rules:**
- User story number is zero-padded to 3 digits (us001, us042, us100)
- Description uses kebab-case (lowercase with hyphens)
- Description should be 2-4 words maximum
- No special characters except hyphens

## Branch Flow

### Standard Flow (User Story → Production)

```
us001/feature
    ↓ PR (tested by developer)
testing
    ↓ PR (QA verified)
dev
    ↓ PR (final integration testing)
staging
    ↓ PR (client acceptance)
main/master
```

### Flow Rules

| From            | To        | Condition              | Action on Rejection                   |
| --------------- | --------- | ---------------------- | ------------------------------------- |
| `us###/feature` | `testing` | Developer tests pass   | Fix in feature branch, re-submit      |
| `testing`       | `dev`     | QA tests pass          | Create new PR from testing            |
| `dev`           | `staging` | Integration tests pass | Create new PR from dev                |
| `staging`       | `main`    | **Client accepts**     | If rejected → back to `us###/feature` |

## Commit Message Standards

```
<type>(<scope>): <Description> - <Summarise>

<Body - What was changed and why>

Files Changed:
- <app-name/folder/file>

Still to do:
- <Task 1>
- <Task 2>

Version: <old-version> → <new-version>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`

**Example:**
```
feat(auth): Add two-factor authentication - Enhance login security

Implement TOTP-based two-factor authentication for user accounts.
Users can now enable 2FA from their account settings.

The implementation uses the Google Authenticator compatible TOTP
algorithm with 30-second time windows.

Files Changed:
- app/Services/TwoFactorService.php
- app/Http/Controllers/Auth/TwoFactorController.php
- resources/views/auth/two-factor.blade.php
- database/migrations/2025_01_15_add_two_factor_columns.php

Still to do:
- Add backup codes generation
- Add SMS fallback option

Version: 1.2.0 → 1.3.0
```

---

## Pull Request Templates

### PR Title Format

```
[<branch-type>] <type>(<scope>): <Description>
```

**Examples:**
- `[US001] feat(auth): Add user login functionality`
- `[HOTFIX] fix(payment): Resolve timeout on checkout`
- `[TESTING→DEV] feat(auth): User authentication module`

### User Story → Testing

```markdown
## Summary

<Brief description of the user story implementation>

## User Story

**US###:** <User story title>

## Acceptance Criteria

- [ ] <Criteria 1>
- [ ] <Criteria 2>
- [ ] <Criteria 3>

## Developer Testing

- [ ] All unit tests pass
- [ ] Manual testing completed
- [ ] Edge cases considered

## Ready for QA

This PR is ready for QA testing on the `testing` branch.
```

### Testing → Dev

```markdown
## Summary

<Changes that passed QA testing>

## QA Results

- [ ] Functional testing passed
- [ ] Regression testing passed
- [ ] Performance acceptable
- [ ] No critical bugs found

## Test Evidence

<Link to test reports or summary of testing performed>

## Ready for Integration

This PR is ready for integration testing on the `dev` branch.
```

### Dev → Staging

```markdown
## Summary

<Changes ready for client review>

## Integration Testing

- [ ] All features work together
- [ ] No conflicts with existing features
- [ ] Performance benchmarks met

## Release Notes (for client)

### New Features
- <Feature 1>
- <Feature 2>

### Bug Fixes
- <Fix 1>

## Ready for Client Review

This PR is ready for client acceptance testing on `staging`.
```

### Staging → Main (Client Accepted)

```markdown
## Summary

<Production-ready changes>

## Client Acceptance

- [x] Client has reviewed changes
- [x] Client has approved for production
- [ ] OR Client has rejected (do not merge, see rejection notes)

## Production Checklist

- [ ] Database migrations reviewed
- [ ] Environment variables documented
- [ ] Rollback plan prepared
- [ ] Monitoring alerts configured

## Version

**Releasing:** `X.Y.Z`

## Deploy Instructions

<Any special deployment steps>
```

---

## Git Hooks

### Pre-commit Hooks

```yaml
# .pre-commit-config.yaml

repos:
  # General
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
        args: ['--maxkb=1000']
      - id: check-merge-conflict
      - id: detect-private-key
      - id: no-commit-to-branch
        args: ['--branch', 'main', '--branch', 'production']

  # Secrets detection
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

### Commit Message Hooks

```bash
#!/bin/bash
# .git/hooks/commit-msg

commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

# Conventional commit pattern
pattern="^(feat|fix|docs|style|refactor|test|chore|perf|ci)(\(.+\))?: .{1,72}"

if ! echo "$commit_msg" | grep -qE "$pattern"; then
    echo "ERROR: Commit message does not follow conventional commit format."
    echo ""
    echo "Expected format: <type>(<scope>): <subject>"
    echo "Types: feat, fix, docs, style, refactor, test, chore, perf, ci"
    echo ""
    echo "Example: feat(auth): add passkey authentication"
    exit 1
fi

# Check for ticket reference in feature/bugfix branches
branch_name=$(git symbolic-ref --short HEAD 2>/dev/null)
if [[ "$branch_name" =~ ^(feature|bugfix)/ ]]; then
    if ! echo "$commit_msg" | grep -qiE "(closes|fixes|resolves|refs) #[0-9]+|[A-Z]+-[0-9]+"; then
        echo "WARNING: Feature/bugfix commits should reference a ticket."
        echo "Consider adding: Closes #123 or JIRA-456"
    fi
fi

exit 0
```

---

## TALL Stack (Laravel 12)

```yaml
# .pre-commit-config.yaml (Laravel additions)

repos:
  # PHP
  - repo: https://github.com/digitalpulp/pre-commit-php
    rev: 1.4.0
    hooks:
      - id: php-lint
      - id: php-cs-fixer
        args: ['--config=.php-cs-fixer.php']

  # Local hooks
  - repo: local
    hooks:
      - id: pint
        name: Laravel Pint
        entry: ./vendor/bin/pint --test
        language: system
        types: [php]
        pass_filenames: false

      - id: phpstan
        name: PHPStan
        entry: ./vendor/bin/phpstan analyse --memory-limit=512M
        language: system
        types: [php]
        pass_filenames: false

      - id: pest
        name: Pest Tests
        entry: ./vendor/bin/pest --parallel
        language: system
        pass_filenames: false
        stages: [pre-push]
```

```php
<?php
// .php-cs-fixer.php

use PhpCsFixer\Config;
use PhpCsFixer\Finder;

$finder = Finder::create()
    ->in([
        __DIR__ . '/app',
        __DIR__ . '/config',
        __DIR__ . '/database',
        __DIR__ . '/routes',
        __DIR__ . '/tests',
    ])
    ->name('*.php')
    ->notName('*.blade.php')
    ->ignoreDotFiles(true)
    ->ignoreVCS(true);

return (new Config())
    ->setFinder($finder)
    ->setRules([
        '@PSR12' => true,
        '@PHP84Migration' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'no_unused_imports' => true,
        'not_operator_with_successor_space' => true,
        'trailing_comma_in_multiline' => true,
        'phpdoc_scalar' => true,
        'unary_operator_spaces' => true,
        'binary_operator_spaces' => true,
        'blank_line_before_statement' => [
            'statements' => ['break', 'continue', 'declare', 'return', 'throw', 'try'],
        ],
        'phpdoc_single_line_var_spacing' => true,
        'phpdoc_var_without_name' => true,
        'class_attributes_separation' => [
            'elements' => [
                'const' => 'one',
                'method' => 'one',
                'property' => 'one',
                'trait_import' => 'none',
            ],
        ],
        'method_argument_space' => [
            'on_multiline' => 'ensure_fully_multiline',
            'keep_multiple_spaces_after_comma' => true,
        ],
        'single_trait_insert_per_statement' => true,
    ])
    ->setRiskyAllowed(true)
    ->setUsingCache(true);
```

```bash
#!/bin/bash
# scripts/git-hooks/pre-push

echo "Running pre-push checks..."

# Run PHPStan
echo "Running PHPStan..."
./vendor/bin/phpstan analyse --memory-limit=512M
if [ $? -ne 0 ]; then
    echo "PHPStan failed. Push aborted."
    exit 1
fi

# Run Pest tests
echo "Running tests..."
./vendor/bin/pest --parallel --stop-on-failure
if [ $? -ne 0 ]; then
    echo "Tests failed. Push aborted."
    exit 1
fi

echo "All checks passed!"
exit 0
```

---

## Django/Wagtail Stack

```yaml
# .pre-commit-config.yaml (Django additions)

repos:
  # Python
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: ['--fix']
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.13.0
    hooks:
      - id: mypy
        additional_dependencies:
          - django-stubs[compatible-mypy]
          - djangorestframework-stubs
          - types-requests

  # Django specific
  - repo: local
    hooks:
      - id: django-check
        name: Django System Check
        entry: python manage.py check --fail-level WARNING
        language: system
        pass_filenames: false

      - id: django-migrations
        name: Check Migrations
        entry: python manage.py makemigrations --check --dry-run
        language: system
        pass_filenames: false

      - id: pytest
        name: Pytest
        entry: pytest --tb=short -q
        language: system
        pass_filenames: false
        stages: [pre-push]
```

```toml
# pyproject.toml

[tool.ruff]
target-version = "py314"
line-length = 120
exclude = [
    ".git",
    ".venv",
    "__pycache__",
    "migrations",
    "node_modules",
]

[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "W",      # pycodestyle warnings
    "F",      # pyflakes
    "I",      # isort
    "C4",     # flake8-comprehensions
    "B",      # flake8-bugbear
    "UP",     # pyupgrade
    "DJ",     # flake8-django
    "S",      # flake8-bandit (security)
    "T20",    # flake8-print
]
ignore = [
    "E501",   # line too long (handled by formatter)
    "S101",   # assert usage (needed for tests)
]

[tool.ruff.lint.isort]
known-first-party = ["apps", "config"]
section-order = ["future", "standard-library", "django", "third-party", "first-party", "local-folder"]

[tool.ruff.lint.isort.sections]
django = ["django", "wagtail"]

[tool.mypy]
python_version = "3.14"
plugins = ["mypy_django_plugin.main"]
strict = true
warn_return_any = true
warn_unused_configs = true
exclude = ["migrations", "tests"]

[tool.django-stubs]
django_settings_module = "config.settings.development"

[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings.testing"
python_files = ["test_*.py", "*_test.py"]
addopts = "-v --tb=short --strict-markers"
markers = [
    "slow: marks tests as slow",
    "integration: marks integration tests",
]
```

```python
# scripts/pre_push_check.py

#!/usr/bin/env python
"""Pre-push validation script for Django projects."""

import subprocess
import sys
from pathlib import Path


def run_command(cmd: list[str], description: str) -> bool:
    """Run a command and return success status."""
    print(f"\n{'=' * 60}")
    print(f"Running: {description}")
    print('=' * 60)

    result = subprocess.run(cmd, capture_output=False)
    return result.returncode == 0


def main() -> int:
    """Run all pre-push checks."""
    checks = [
        (['python', 'manage.py', 'check', '--fail-level', 'WARNING'], 'Django System Check'),
        (['python', 'manage.py', 'makemigrations', '--check', '--dry-run'], 'Migration Check'),
        (['ruff', 'check', '.'], 'Ruff Linting'),
        (['mypy', '.'], 'Type Checking'),
        (['pytest', '--tb=short', '-q', '--exitfirst'], 'Unit Tests'),
    ]

    failed = []

    for cmd, description in checks:
        if not run_command(cmd, description):
            failed.append(description)

    if failed:
        print(f"\n{'=' * 60}")
        print("FAILED CHECKS:")
        for check in failed:
            print(f"  - {check}")
        print('=' * 60)
        return 1

    print(f"\n{'=' * 60}")
    print("All checks passed!")
    print('=' * 60)
    return 0


if __name__ == '__main__':
    sys.exit(main())
```

---

## React/Next.js Stack

```yaml
# .pre-commit-config.yaml (Next.js additions)

repos:
  # JavaScript/TypeScript
  - repo: local
    hooks:
      - id: eslint
        name: ESLint
        entry: npx eslint --fix
        language: system
        types: [javascript, jsx, ts, tsx]

      - id: prettier
        name: Prettier
        entry: npx prettier --write
        language: system
        types: [javascript, jsx, ts, tsx, json, css, scss, md]

      - id: tsc
        name: TypeScript Check
        entry: npx tsc --noEmit
        language: system
        pass_filenames: false

      - id: vitest
        name: Vitest
        entry: npx vitest run --passWithNoTests
        language: system
        pass_filenames: false
        stages: [pre-push]

      - id: build
        name: Build Check
        entry: npm run build
        language: system
        pass_filenames: false
        stages: [pre-push]
```

```javascript
// lint-staged.config.js

export default {
  '*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
  ],
  '*.{json,md,mdx,css,scss}': [
    'prettier --write',
  ],
  '*.ts?(x)': () => 'tsc --noEmit',
};
```

```json
// .husky/pre-commit

{
  "scripts": {
    "prepare": "husky"
  }
}
```

```bash
#!/bin/sh
# .husky/pre-commit

npx lint-staged
```

```bash
#!/bin/sh
# .husky/pre-push

echo "Running pre-push checks..."

# Type check
echo "Type checking..."
npx tsc --noEmit
if [ $? -ne 0 ]; then
    echo "Type check failed. Push aborted."
    exit 1
fi

# Run tests
echo "Running tests..."
npm run test -- --run --passWithNoTests
if [ $? -ne 0 ]; then
    echo "Tests failed. Push aborted."
    exit 1
fi

# Build check
echo "Checking build..."
npm run build
if [ $? -ne 0 ]; then
    echo "Build failed. Push aborted."
    exit 1
fi

echo "All checks passed!"
exit 0
```

```typescript
// scripts/validate-branch.ts

import { execSync } from 'child_process';

const BRANCH_PATTERN = /^(feature|bugfix|hotfix|release|chore)\/[a-z0-9-]+$/;
const PROTECTED_BRANCHES = ['main', 'production', 'staging'];

function getCurrentBranch(): string {
  return execSync('git symbolic-ref --short HEAD', { encoding: 'utf-8' }).trim();
}

function validateBranch(): void {
  const branch = getCurrentBranch();

  if (PROTECTED_BRANCHES.includes(branch)) {
    console.error(`ERROR: Direct commits to '${branch}' are not allowed.`);
    console.error('Please create a feature branch and submit a pull request.');
    process.exit(1);
  }

  if (!BRANCH_PATTERN.test(branch)) {
    console.warn(`WARNING: Branch name '${branch}' does not follow naming conventions.`);
    console.warn('Expected format: <type>/<description>');
    console.warn('Types: feature, bugfix, hotfix, release, chore');
  }
}

validateBranch();
```

---

## React Native Stack

```yaml
# .pre-commit-config.yaml (React Native additions)

repos:
  - repo: local
    hooks:
      - id: eslint
        name: ESLint
        entry: npx eslint --fix
        language: system
        types: [javascript, jsx, ts, tsx]

      - id: prettier
        name: Prettier
        entry: npx prettier --write
        language: system
        types: [javascript, jsx, ts, tsx, json]

      - id: tsc
        name: TypeScript Check
        entry: npx tsc --noEmit
        language: system
        pass_filenames: false

      - id: jest
        name: Jest Tests
        entry: npx jest --passWithNoTests --bail
        language: system
        pass_filenames: false
        stages: [pre-push]
```

```javascript
// lint-staged.config.js

export default {
  '*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
  ],
  '*.{json,md}': [
    'prettier --write',
  ],
  '*.ts?(x)': () => 'tsc --noEmit',
};
```

```bash
#!/bin/sh
# .husky/pre-commit

npx lint-staged
```

```bash
#!/bin/sh
# .husky/pre-push

echo "Running pre-push checks..."

# Type check
echo "Type checking..."
npx tsc --noEmit
if [ $? -ne 0 ]; then
    echo "Type check failed. Push aborted."
    exit 1
fi

# Run tests
echo "Running tests..."
npx jest --passWithNoTests --bail
if [ $? -ne 0 ]; then
    echo "Tests failed. Push aborted."
    exit 1
fi

# Expo doctor check
echo "Running Expo doctor..."
npx expo-doctor
if [ $? -ne 0 ]; then
    echo "Expo doctor found issues. Push aborted."
    exit 1
fi

echo "All checks passed!"
exit 0
```

```typescript
// scripts/pre-build-check.ts

import { execSync } from 'child_process';

interface CheckResult {
  name: string;
  passed: boolean;
  message: string;
}

const checks: Array<{ name: string; command: string }> = [
  { name: 'TypeScript', command: 'npx tsc --noEmit' },
  { name: 'ESLint', command: 'npx eslint . --max-warnings 0' },
  { name: 'Jest', command: 'npx jest --passWithNoTests' },
  { name: 'Expo Doctor', command: 'npx expo-doctor' },
];

async function runChecks(): Promise<void> {
  const results: CheckResult[] = [];

  for (const check of checks) {
    console.log(`\nRunning ${check.name}...`);

    try {
      execSync(check.command, { stdio: 'inherit' });
      results.push({ name: check.name, passed: true, message: 'Passed' });
    } catch {
      results.push({ name: check.name, passed: false, message: 'Failed' });
    }
  }

  console.log('\n=== Check Summary ===');
  results.forEach((result) => {
    const icon = result.passed ? '\u2713' : '\u2717';
    console.log(`${icon} ${result.name}: ${result.message}`);
  });

  const failed = results.filter((r) => !r.passed);
  if (failed.length > 0) {
    console.error(`\n${failed.length} check(s) failed. Please fix before building.`);
    process.exit(1);
  }

  console.log('\nAll checks passed! Ready to build.');
}

runChecks();
```

```json
// package.json (scripts section)

{
  "scripts": {
    "prepare": "husky",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "typecheck": "tsc --noEmit",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "pre-build": "ts-node scripts/pre-build-check.ts",
    "validate": "npm run typecheck && npm run lint && npm run test"
  }
}
```
