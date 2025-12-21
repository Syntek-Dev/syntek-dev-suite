# Syntax & Linting Examples

## Overview

Linting and code style configurations for consistent code quality across all stacks.

---

## Table of Contents

- [Overview](#overview)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [React/Next.js Stack](#reactnextjs-stack)
- [React Native Stack](#react-native-stack)

## TALL Stack (Laravel 12)

### Laravel Pint

```php
<?php
// pint.json

{
    "preset": "laravel",
    "rules": {
        "array_syntax": { "syntax": "short" },
        "binary_operator_spaces": { "default": "single_space" },
        "blank_line_after_namespace": true,
        "blank_line_after_opening_tag": true,
        "blank_line_before_statement": {
            "statements": ["return", "throw", "try"]
        },
        "braces": { "position_after_control_structures": "same" },
        "class_attributes_separation": {
            "elements": { "method": "one", "property": "one" }
        },
        "concat_space": { "spacing": "one" },
        "declare_strict_types": true,
        "final_class": true,
        "method_argument_space": { "on_multiline": "ensure_fully_multiline" },
        "no_unused_imports": true,
        "ordered_imports": { "sort_algorithm": "alpha" },
        "single_quote": true,
        "trailing_comma_in_multiline": { "elements": ["arrays", "arguments"] }
    },
    "exclude": [
        "bootstrap",
        "storage",
        "vendor"
    ]
}
```

### PHPStan

```yaml
# phpstan.neon

includes:
    - vendor/larastan/larastan/extension.neon

parameters:
    level: 9
    paths:
        - app
        - config
        - database
        - routes
        - tests

    excludePaths:
        - app/Console/Kernel.php
        - app/Exceptions/Handler.php

    ignoreErrors:
        - '#Unsafe usage of new static#'

    checkMissingIterableValueType: false
    checkGenericClassInNonGenericObjectType: false

    universalObjectCratesClasses:
        - Illuminate\Http\Request
```

---

## Django/Wagtail Stack

### Ruff Configuration

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
    "B",      # flake8-bugbear
    "C4",     # flake8-comprehensions
    "UP",     # pyupgrade
    "DJ",     # flake8-django
    "S",      # flake8-bandit
    "T20",    # flake8-print
    "SIM",    # flake8-simplify
    "RUF",    # Ruff-specific
]

ignore = [
    "E501",   # line too long
    "S101",   # assert usage
]

[tool.ruff.lint.isort]
known-first-party = ["apps", "config"]

[tool.ruff.format]
quote-style = "single"
indent-style = "space"
```

### Mypy Configuration

```toml
# pyproject.toml

[tool.mypy]
python_version = "3.14"
plugins = ["mypy_django_plugin.main"]
strict = true
warn_return_any = true
warn_unused_configs = true
exclude = ["migrations", "tests"]

[tool.django-stubs]
django_settings_module = "config.settings.development"

[[tool.mypy.overrides]]
module = "*.migrations.*"
ignore_errors = true
```

---

## React/Next.js Stack

### ESLint Configuration

```javascript
// eslint.config.mjs

import { FlatCompat } from '@eslint/eslintrc';
import js from '@eslint/js';
import typescript from '@typescript-eslint/eslint-plugin';
import typescriptParser from '@typescript-eslint/parser';

const compat = new FlatCompat();

export default [
  js.configs.recommended,
  ...compat.extends('next/core-web-vitals'),
  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parser: typescriptParser,
      parserOptions: {
        project: './tsconfig.json',
      },
    },
    plugins: {
      '@typescript-eslint': typescript,
    },
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'off',
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/prefer-nullish-coalescing': 'error',
      '@typescript-eslint/strict-boolean-expressions': 'error',

      'react/jsx-curly-brace-presence': ['error', { props: 'never', children: 'never' }],
      'react/self-closing-comp': 'error',

      'import/order': [
        'error',
        {
          groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
          'newlines-between': 'always',
          alphabetize: { order: 'asc' },
        },
      ],
    },
  },
];
```

### Prettier Configuration

```json
// .prettierrc

{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

### TypeScript Configuration

```json
// tsconfig.json

{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

---

## React Native Stack

### ESLint Configuration

```javascript
// eslint.config.mjs

import { FlatCompat } from '@eslint/eslintrc';
import js from '@eslint/js';
import reactNative from 'eslint-plugin-react-native';

const compat = new FlatCompat();

export default [
  js.configs.recommended,
  ...compat.extends('expo'),
  {
    files: ['**/*.{ts,tsx}'],
    plugins: {
      'react-native': reactNative,
    },
    rules: {
      'react-native/no-unused-styles': 'error',
      'react-native/split-platform-components': 'error',
      'react-native/no-inline-styles': 'warn',
      'react-native/no-color-literals': 'warn',
      'react-native/no-raw-text': ['error', { skip: ['Text'] }],

      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/no-explicit-any': 'error',

      'import/order': [
        'error',
        {
          groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
          'newlines-between': 'always',
        },
      ],
    },
  },
];
```

### TypeScript Configuration

```json
// tsconfig.json

{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```
