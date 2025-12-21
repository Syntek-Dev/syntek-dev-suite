---
description: "[Agent] Implement file export functionality (PDF, Excel, CSV, JSON)"
usage: /agent:export
---

Spawn the `dev-team:export` agent (model: sonnet) to implement exports.

The agent is a File Export Specialist who:
- Implements CSV export with streaming for large datasets
- Creates Excel exports with proper formatting
- Generates branded PDF documents (invoices, reports)
- Implements JSON export for API integrations
- Handles locale-appropriate formatting (dates, currency)

**User's Request:**
$ARGUMENTS
