# Editor Configuration Templates

## Overview

Editor and linter configuration files for consistent code formatting across different IDEs and text editors.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 1.0.0 |
| **Last Updated** | 2025-01 |
| **Stacks** | All (TALL, Django, React, Mobile, Shared-Lib) |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [EditorConfig](#editorconfig)
- [Prettier](#prettier)
- [ESLint](#eslint)
- [Node Version](#node-version)
- [Python Configuration](#python-configuration)

---

## EditorConfig

### .editorconfig

```editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{js,jsx,ts,tsx,vue,json,yml,yaml}]
indent_size = 2

[*.{css,scss,less}]
indent_size = 2

[Makefile]
indent_style = tab
```

---

## Prettier

### .prettierrc

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

### .prettierignore

```
node_modules/
dist/
build/
coverage/
*.min.js
*.min.css
package-lock.json
yarn.lock
pnpm-lock.yaml
```

---

## ESLint

### .eslintignore

```
node_modules/
dist/
build/
coverage/
*.min.js
```

---

## Node Version

### .nvmrc

```
20
```

---

## Python Configuration

### .flake8

```ini
[flake8]
max-line-length = 100
exclude =
    .git,
    __pycache__,
    .venv,
    venv,
    migrations,
    build,
    dist
ignore = E203, E266, E501, W503
```