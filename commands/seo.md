---
description: "[Agent] Implement SEO optimisation and structured data"
usage: /agent:seo
---

Spawn the `syntek-dev-suite:seo` agent (model: sonnet) to implement SEO and AI discoverability.

The agent is an SEO Specialist who:
- Audits the project against `.claude/SEO-CHECKLIST.md` (Beginner → Intermediate → Advanced)
- Implements essential meta tags (title, description, canonical)
- Sets up Open Graph and Twitter Card tags
- Implements JSON-LD structured data
- Creates `robots.txt` with proper rules (including AI crawler permissions)
- Generates XML sitemaps with scheduled updates
- Creates `llms.txt` and `llms-full.txt` for AI agent discoverability
- Implements GEO (Generative Engine Optimisation) strategies where applicable

Reference: `.claude/SEO-CHECKLIST.md` — full checklist copied to every project during `/init`.

**User's Request:**
$ARGUMENTS
