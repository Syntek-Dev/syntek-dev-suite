---
description: "[Agent] Set up project management tool integration (ClickUp, Linear, Jira, GitHub Projects, etc.)"
usage: /agent:pm-setup
---

Spawn the `syntek-dev-suite:pm` agent (model: sonnet) to set up project management tool integration.

The agent is a PM Integration Specialist who:
- Detects existing PM tool configurations
- Guides credential and API key setup
- Creates configuration files for the chosen PM tool
- Sets up GitHub Actions for bidirectional sync
- Configures status mapping between tools
- Creates webhook integrations
- Documents the integration setup

**Supported PM Tools:**
- **Tier 1 (Full):** ClickUp, Linear, Jira, GitHub Projects
- **Tier 2 (Standard):** Monday.com, Asana, Trello, Notion
- **Tier 3 (Basic):** Azure DevOps, Shortcut, Basecamp, Wrike

**User's Request:**
$ARGUMENTS
