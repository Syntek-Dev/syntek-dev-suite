# API Design — Steps

**Last Updated**: {DATE}
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)

---

## Prerequisites

Before starting, confirm:

- [ ] Requirements are defined (user stories or ADR)
- [ ] `.claude/API-DESIGN.md` has been read
- [ ] Database schema design is agreed (or complete the database-migration workflow first)
- [ ] Branch created following `how-to/workflows/02-git-workflow/`
- [ ] No blocking items in `/GAPS.md`

---

## Steps

### Step 1 — Architectural Plan

Generate an API design plan before writing any code.

```
/syntek-dev-suite:plan [API description — endpoints, resources, or GraphQL operations]
```

The plan must address:
- URL structure and HTTP methods (REST) or operation names and types (GraphQL)
- Request and response schemas
- Authentication and authorisation requirements
- Pagination and filtering strategy
- Rate limiting requirements
- Error response format (per `.claude/API-DESIGN.md`)

Save the plan to `project-management/src/PLANS/PLAN-API-{RESOURCE}.md`.

### Step 2 — Write API Contract Tests

Write tests that define the expected API behaviour before implementation.

```
/syntek-dev-suite:test-writer [API description] --mode contract-tests
```

Tests must cover:
- Happy path responses (correct status codes and response shape)
- Authentication failures (401/403)
- Validation errors (400 with error detail)
- Not found (404)
- Rate limiting (429, if applicable)

### Step 3 — Implement API Layer

```
/syntek-dev-suite:backend [API description]
```

The backend agent reads `.claude/API-DESIGN.md`, `.claude/ARCHITECTURE-PATTERNS.md`, and `.claude/SECURITY.md`. Implementation must follow:
- Service layer pattern (business logic in services, not views/controllers)
- Proper serialisation and validation
- Field-level authorisation checks
- Consistent error response format

### Step 4 — Verify Contract Tests Pass

Run the test suite. All contract tests written in Step 2 must be green.

If any test fails, return to Step 3 and fix the implementation.

### Step 5 — Security Review

```
/syntek-dev-suite:security [API description]
```

Specifically check:
- All endpoints require authentication (unless explicitly public)
- Authorisation is enforced at the service layer, not just the view
- Input validation covers all fields
- Sensitive data is not exposed in responses or error messages

### Step 6 — Documentation

Generate API documentation for the new endpoints.

```
/syntek-dev-suite:docs [API description]
```

For REST APIs: update or create the API reference in `code/docs/`.
For GraphQL: ensure schema introspection is enabled and types are documented.

### Step 7 — Code Review

```
/syntek-dev-suite:review
```

### Step 8 — Commit

```
/syntek-dev-suite:git
```

---

## Error Handling

If the plan (Step 1) reveals a breaking change to an existing API:
1. Consult `.claude/API-DESIGN.md` versioning section
2. Determine if a new version is required or if a non-breaking extension is possible
3. Update the ADR in `project-management/src/PLANS/` if the approach changes

---

## Completion

Run through `CHECKLIST.md` before marking this workflow complete.
