# .claude-plugin

## Overview

This folder contains the Claude Code plugin configuration files that define the Syntek Dev Suite plugin metadata and marketplace registration. These files are read by Claude Code to identify and load the plugin.

**Version:** 1.2.0

---


## Table of Contents

- [Overview](#overview)
- [Table of Contents](#table-of-contents)
- [Directory Tree](#directory-tree)
- [Files](#files)
- [Usage](#usage)
  - [Plugin Installation](#plugin-installation)
- [Related Sections](#related-sections)

## Directory Tree

```
.claude-plugin/
├── README.md           # This file
├── plugin.json         # Plugin metadata and configuration
└── marketplace.json    # Marketplace registration details
```

---

## Files

| File               | Purpose                                                                             |
| ------------------ | ----------------------------------------------------------------------------------- |
| `plugin.json`      | Defines the plugin name, version, description, author, and keywords for Claude Code |
| `marketplace.json` | Registers the plugin with the Syntek marketplace and defines available plugins      |

---

## Usage

These files are automatically read by Claude Code when the plugin is installed. You typically do not need to modify them unless:

1. **Updating version:** Bump the version in `plugin.json` when releasing updates
2. **Adding keywords:** Add relevant keywords to improve discoverability
3. **Registering new plugins:** Add entries to `marketplace.json` for additional plugins

### Plugin Installation

To add this plugin to Claude Code, add the following to your `~/.claude/settings.json`:

```json
{
  "plugins": [
    "/path/to/claude-dev-team"
  ]
}
```

---

## Related Sections

- [../agents/](../agents/) - Agent definitions loaded by the plugin
- [../commands/](../commands/) - Slash commands registered by the plugin
- [../skills/](../skills/) - Stack-specific skills applied by agents
- [../templates/](../templates/) - Project templates for initialisation
- [Overview](#)
- [Directory Tree](#)
- [Files](#)
- [Usage](#)
- [Related Sections](#)
