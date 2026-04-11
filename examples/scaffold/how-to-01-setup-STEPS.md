# Project Setup — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Repository has been cloned
- [ ] syntek-dev-suite plugin is installed (`~/.claude/plugins/syntek-dev-suite/`)
- [ ] Required tools are installed (Node.js, Python, Docker — see `how-to/docs/DEVELOPMENT.md`)
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Run Syntek Dev Suite Initialisation

If the project has not already been initialised with syntek-dev-suite:

```
/syntek-dev-suite:init
```

This creates `.claude/` with all reference documents, plugins, and stack-specific configuration.

### Step 2 — Complete Project Setup

```
/syntek-dev-suite:setup [project description and stack]
```

The setup agent reads `.claude/DEVELOPMENT.md` and will:
- Verify required tools are installed
- Create or validate environment files (`.env.dev`, `.env.test`)
- Set up the development container (Docker or DDEV)
- Run initial database migrations
- Seed development data (if a seed script exists)

### Step 3 — Verify Development Environment

Confirm the environment is working:

```bash
# Start the development server (command varies by stack — see how-to/docs/DEVELOPMENT.md)
```

- [ ] Development server starts without errors
- [ ] Application is accessible on the expected local port
- [ ] Database connection is working
- [ ] Test suite runs cleanly

### Step 4 — Configure Environment Variables

Review `.env.dev.example` and create `.env.dev` with real development values:

```bash
cp .env.dev.example .env.dev
# Edit .env.dev with your local values
```

Never commit `.env.dev` or any file containing real secrets. See `.claude/SECURITY.md` for secrets management guidance.

### Step 5 — Confirm Git Configuration

Follow `how-to/workflows/02-git-workflow/` to confirm branch strategy, commit message format, and pre-commit hooks are working.

---

## Error Handling

If the container fails to start:
1. Check Docker is running (`docker info`)
2. Check for port conflicts (`lsof -i :[port]`)
3. Review container logs for the specific error

If migrations fail:
1. Verify the database container is running
2. Verify environment variables point to the correct database
3. Run the database migration workflow: `code/workflows/05-database-migration/`

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
