# Testing Guide

**Last Updated**: 15/03/2026
**Version**: 1.8.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Testing Matrix](#testing-matrix)
- [Running Tests](#running-tests)
- [Python / Django](#python--django)
- [TypeScript / React (Web)](#typescript--react-web)
- [React Native / Mobile](#react-native--mobile)
- [GraphQL](#graphql)
- [Database Isolation](#database-isolation)
- [Migration Testing](#migration-testing)
- [Test Data and Factories](#test-data-and-factories)
- [Property-Based Testing with Hypothesis](#property-based-testing-with-hypothesis)
- [Coverage Thresholds and Enforcement](#coverage-thresholds-and-enforcement)
- [Test Naming Conventions](#test-naming-conventions)
- [Mocking Philosophy](#mocking-philosophy)
- [Snapshot Testing](#snapshot-testing)
- [Error Path and Boundary Testing](#error-path-and-boundary-testing)
- [Accessibility Testing](#accessibility-testing)
- [Performance and Load Testing](#performance-and-load-testing)
- [Flaky Test Policy](#flaky-test-policy)
- [CI Integration](#ci-integration)
- [Per-Package Testing Files](#per-package-testing-files)
- [Rules and Principles](#rules-and-principles)

---

## Overview

This repo uses different testing frameworks per layer. All layers follow Arrange-Act-Assert and the testing pyramid: many unit, some integration, few E2E.

---

## Testing Matrix

| Layer                  | Unit / Integration              | E2E / Browser | Framework                                          |
|------------------------|---------------------------------|---------------|-----------------------------------------------------|
| Python / Django        | pytest + factory_boy + hypothesis | -             | pytest-django, testcontainers-python                |
| GraphQL (Python)       | pytest                          | -             | pytest-django + strawberry test client              |
| Web (React/TS)         | Vitest + RTL + MSW              | Playwright    | vitest, @testing-library/react, msw                 |
| GraphQL (TS resolvers) | Vitest + MSW                    | -             | vitest, msw                                         |
| Mobile (RN)            | Jest + RNTL                     | Maestro       | jest, @testing-library/react-native                 |
| Postgres               | pytest transactional fixtures   | -             | testcontainers-python                               |

---

## Running Tests

### Full suite (all layers)

```bash
syntek-dev test
```

### Per layer

```bash
# Python
pytest packages/backend/syntek-auth/tests/

# TypeScript (all packages via Turborepo)
pnpm test

# Single package
pnpm --filter @syntek/ui-auth test

# Markdown linting
pnpm lint:md
```

---

## Python / Django

**Tools:** pytest-django, factory_boy, pytest-cov, testcontainers-python

Each backend module has its own `tests/` directory and a minimal `tests/settings.py` for Django configuration during testing. There is no project-level `manage.py` - tests run via pytest directly.

```bash
# Run with coverage
pytest packages/backend/syntek-auth/ --cov=syntek_auth --cov-report=html

# Run only unit tests
pytest packages/backend/syntek-auth/ -m unit

# Run only integration tests (spins up Postgres via testcontainers)
pytest packages/backend/syntek-auth/ -m integration
```

### Module test settings

Each backend module provides `tests/settings.py`:

```python
# packages/backend/syntek-auth/tests/settings.py
SECRET_KEY = "test-secret-key"  # noqa: S105
INSTALLED_APPS = [
    "django.contrib.contenttypes",
    "django.contrib.auth",
    "syntek_auth",
]
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "syntek_test",
        "USER": "postgres",
        "PASSWORD": "postgres",
        "HOST": "localhost",
        "PORT": "5432",
    }
}
```

### Postgres via testcontainers

Use testcontainers-python for integration tests that need a real PostgreSQL 18.3 instance - no external database setup required:

```python
# packages/backend/syntek-auth/tests/conftest.py
import pytest
from testcontainers.postgres import PostgresContainer


@pytest.fixture(scope="session")
def postgres_container():
    with PostgresContainer("postgres:18.3") as pg:
        yield pg
```

### factory_boy example

```python
# packages/backend/syntek-auth/tests/factories.py
import factory
from django.contrib.auth import get_user_model


class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = get_user_model()

    email = factory.Sequence(lambda n: f"user{n}@example.com")
    password = factory.PostGenerationMethodCall("set_password", "secret-password-123")
    is_active = True
```

---

## TypeScript / React (Web)

**Tools:** Vitest, React Testing Library, MSW, Playwright, Cypress

Each `packages/web/*` package uses Vitest for unit and integration tests, and Playwright or Cypress for E2E tests. Tests live alongside source files.

```bash
# Unit + integration (watch mode)
pnpm --filter @syntek/ui test --watch

# Coverage
pnpm --filter @syntek/ui-auth test --coverage

# E2E (Playwright)
pnpm --filter @syntek/ui-auth test:e2e
```

### Vitest component test example

```tsx
// packages/web/ui-auth/src/LoginForm.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";

import { LoginForm } from "./LoginForm";

describe("LoginForm", () => {
  it("calls onSubmit with email and password when form is submitted", async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    fireEvent.change(screen.getByLabelText("Email"), {
      target: { value: "user@example.com" },
    });
    fireEvent.change(screen.getByLabelText("Password"), {
      target: { value: "secret123" },
    });
    fireEvent.click(screen.getByRole("button", { name: /sign in/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: "user@example.com",
      password: "secret123",
    });
  });
});
```

### MSW for GraphQL mocking

```typescript
// packages/web/ui-auth/tests/msw/handlers.ts
import { graphql, HttpResponse } from "msw";

export const handlers = [
  graphql.mutation("Login", () => {
    return HttpResponse.json({
      data: { login: { token: "test-token", user: { id: "1" } } },
    });
  }),
];
```

---

## React Native / Mobile

**Tools:** Jest, React Native Testing Library (RNTL), Maestro

```bash
# Unit + integration
pnpm --filter @syntek/mobile-auth test

# Watch
pnpm --filter @syntek/mobile-auth test --watch

# Maestro E2E (requires device/emulator)
maestro test mobile/mobile-auth/.maestro/
```

### RNTL component test example

```tsx
// mobile/mobile-auth/src/BiometricPrompt.test.tsx
import { render, fireEvent } from "@testing-library/react-native";
import { describe, it, expect, vi } from "vitest";

import { BiometricPrompt } from "./BiometricPrompt";

describe("BiometricPrompt", () => {
  it("calls onAuthenticate when the prompt button is pressed", () => {
    const onAuthenticate = vi.fn();
    const { getByText } = render(<BiometricPrompt onAuthenticate={onAuthenticate} />);

    fireEvent.press(getByText("Use Face ID"));

    expect(onAuthenticate).toHaveBeenCalledOnce();
  });
});
```

---

## GraphQL

### Python (Strawberry) - use pytest

```python
# packages/backend/syntek-auth/tests/test_schema.py
import pytest
from strawberry.test import Client

from syntek_auth.schema import schema


@pytest.mark.django_db
def test_login_mutation_returns_token(user_factory):
    user = user_factory(email="test@example.com")
    client = Client(schema)

    result = client.execute(
        """
        mutation Login($email: String!, $password: String!) {
          login(email: $email, password: $password) {
            token
          }
        }
        """,
        variables={"email": "test@example.com", "password": "secret-password-123"},
    )

    assert result.errors is None
    assert result.data["login"]["token"] is not None
```

### TypeScript resolvers - use Vitest with direct unit tests + MSW

Direct resolver unit tests do not need a network:

```typescript
// packages/web/api-client/src/resolvers/auth.test.ts
import { describe, it, expect, vi } from "vitest";
import { loginResolver } from "./auth";

describe("loginResolver", () => {
  it("returns user and token on valid credentials", async () => {
    const mockContext = { dataSources: { authApi: { login: vi.fn().mockResolvedValue({ token: "tok_1" }) } } };
    const result = await loginResolver(null, { email: "a@b.com", password: "pw" }, mockContext);
    expect(result.token).toBe("tok_1");
  });
});
```

Use MSW to mock the GraphQL endpoint in component-level tests.

---

## Database Isolation

- **Python integration tests:** use `@pytest.mark.django_db` with transaction rollback (default in pytest-django). Each test starts from a clean state.
- **Testcontainers:** spin up an ephemeral PostgreSQL 18.3 container per session for integration tests. Never point tests at the dev database.
- **TypeScript:** mock the data layer via MSW or `vi.mock()`. No real DB in unit tests.

---

## Migration Testing

The coding principles require that every new database migration has a test verifying it runs and rolls back cleanly. Use testcontainers to run migrations against a real PostgreSQL instance without touching the development database.

### Pattern: forward and rollback verification

```python
# packages/backend/syntek-auth/tests/test_migrations.py
import subprocess

import pytest
from testcontainers.postgres import PostgresContainer


@pytest.fixture(scope="module")
def migration_db():
    with PostgresContainer("postgres:18.3") as pg:
        yield {
            "host": pg.get_container_host_ip(),
            "port": pg.get_exposed_port(5432),
            "user": "test",
            "password": "test",
            "name": "test",
        }


@pytest.mark.integration
class TestMigrations:
    def test_migrate_forward(self, migration_db):
        """All migrations apply cleanly to an empty database."""
        result = subprocess.run(
            [
                "python", "-m", "django", "migrate",
                "--settings=tests.settings",
                "--database=default",
                "--run-syncdb",
            ],
            env=_build_env(migration_db),
            capture_output=True,
            text=True,
        )
        assert result.returncode == 0, f"Migration failed:\n{result.stderr}"

    def test_migrate_rollback(self, migration_db):
        """Migrations roll back to zero without errors."""
        # Apply all migrations first
        subprocess.run(
            ["python", "-m", "django", "migrate", "--settings=tests.settings"],
            env=_build_env(migration_db),
            capture_output=True,
        )
        # Roll back to zero
        result = subprocess.run(
            [
                "python", "-m", "django", "migrate",
                "syntek_auth", "zero",
                "--settings=tests.settings",
            ],
            env=_build_env(migration_db),
            capture_output=True,
            text=True,
        )
        assert result.returncode == 0, f"Rollback failed:\n{result.stderr}"

    def test_no_pending_migrations(self, migration_db):
        """No model changes exist that haven't been captured in a migration."""
        result = subprocess.run(
            [
                "python", "-m", "django", "makemigrations",
                "--check", "--dry-run",
                "--settings=tests.settings",
            ],
            env=_build_env(migration_db),
            capture_output=True,
            text=True,
        )
        assert result.returncode == 0, f"Pending migrations detected:\n{result.stdout}"


def _build_env(db_config: dict) -> dict:
    import os
    env = os.environ.copy()
    env["DATABASE_URL"] = (
        f"postgres://{db_config['user']}:{db_config['password']}"
        f"@{db_config['host']}:{db_config['port']}/{db_config['name']}"
    )
    return env
```

### Rules

- Migration tests run in CI on every PR that touches a migration file or model definition.
- Data migrations (RunPython) must have both a forward and reverse function. Migrations with `reverse_code=migrations.RunPython.noop` are only acceptable if the forward operation is additive (adding a column, populating a new field) and data loss on rollback is documented in the migration's docstring.
- Never test migrations against sqlite. Always use testcontainers with the same PostgreSQL version as production (18.3).

---

## Test Data and Factories

- **Python:** use factory_boy (`DjangoModelFactory`) for all model fixtures. Never build model instances inline across tests.
- **TypeScript:** use plain builder functions in `tests/helpers/builders.ts`.

### TypeScript builder example

```typescript
// packages/web/ui-auth/tests/helpers/builders.ts
interface UserBuilder {
  id: string;
  email: string;
  name: string;
  role: "admin" | "member" | "viewer";
}

let sequence = 0;

export function buildUser(overrides: Partial<UserBuilder> = {}): UserBuilder {
  sequence += 1;
  return {
    id: `user_${sequence}`,
    email: `user${sequence}@example.com`,
    name: `Test User ${sequence}`,
    role: "member",
    ...overrides,
  };
}
```

Builders return plain objects with sensible defaults. Override only the fields relevant to the test. This keeps tests focused on what matters and resistant to unrelated changes.

---

## Property-Based Testing with Hypothesis

Use hypothesis for any function that must hold across a wide range of inputs - especially cryptographic functions, validators, and data transformations.

Install: `uv pip install hypothesis` (included in `install.sh`).

```python
# packages/backend/syntek-crypto-bridge/tests/test_crypto.py
from hypothesis import given, settings
from hypothesis import strategies as st

from syntek_crypto_bridge import encrypt_field, decrypt_field


@given(plaintext=st.text(min_size=1, max_size=500))
@settings(max_examples=200)
def test_encrypt_decrypt_round_trip(plaintext: str) -> None:
    """AES-256-GCM round-trip: decrypt(encrypt(x)) == x for any input."""
    key = b"a" * 32
    ciphertext = encrypt_field(plaintext, key)
    assert decrypt_field(ciphertext, key) == plaintext


@given(
    value=st.one_of(st.text(), st.integers(), st.floats(allow_nan=False)),
    length=st.integers(min_value=1, max_value=64),
)
def test_password_validator_never_raises(value: object, length: int) -> None:
    """Password validator must not raise; it returns True or False."""
    from syntek_auth.validators import meets_minimum_length
    result = meets_minimum_length(str(value), min_length=length)
    assert isinstance(result, bool)
```

### Where to use hypothesis

- Cryptographic functions (e.g., field-level encryption bridges) - round-trip, tamper detection.
- Input validators - must never raise; must return a bool or raise a specific exception.
- Data transformation functions - idempotency, associativity.
- HMAC / signature functions - different inputs produce different outputs.

### Where NOT to use hypothesis

- Tests that require database state (use factory_boy + pytest fixtures instead).
- E2E or integration tests (too slow for property-based iteration).

---

## Coverage Thresholds and Enforcement

Coverage is measured per layer and enforced in CI. A PR that drops coverage below the threshold is blocked from merging.

| Layer              | Minimum Line Coverage | Tool                         |
|--------------------|----------------------|------------------------------|
| Python / Django    | 80%                  | pytest-cov                   |
| TypeScript (Web)   | 75%                  | Vitest (`--coverage`)        |
| React Native       | 70%                  | Jest (`--coverage`)          |

### Configuration

**Python** - add to `pyproject.toml` or `pytest.ini` per package:

```ini
[tool.pytest.ini_options]
addopts = "--cov=syntek_auth --cov-fail-under=80"
```

**TypeScript** - add to the package's `vitest.config.ts`:

```typescript
export default defineConfig({
  test: {
    coverage: {
      provider: "v8",
      thresholds: {
        lines: 75,
        branches: 70,
        functions: 75,
        statements: 75,
      },
    },
  },
});
```

### Rules

- Coverage thresholds are enforced in CI. A build that falls below the threshold fails.
- Coverage measures the floor, not the goal. 80% coverage with thoughtless tests is worse than 60% coverage with meaningful assertions. Write tests that verify behaviour, not tests that exercise lines.
- Exclude generated code, migration files, and configuration from coverage reports. In Python, add `omit = ["*/migrations/*", "*/tests/*"]` to the coverage configuration.
- When adding a new module, set up coverage from the first PR. Do not defer coverage configuration.

---

## Test Naming Conventions

Consistent test names make it possible to understand what failed from CI output alone, without reading the test body.

### Python

Follow the pattern `test_<unit>_<scenario>_<expected_result>`:

```python
# Good
def test_login_with_expired_token_returns_401():
def test_encrypt_field_with_empty_string_raises_value_error():
def test_user_factory_creates_active_user_by_default():

# Bad
def test_login():
def test_login_2():
def test_it_works():
```

### TypeScript / React / React Native

Use descriptive `it()` or `test()` strings that read as sentences:

```typescript
// Good
it("returns a 401 error when the token is expired")
it("renders the login form with email and password fields")
it("disables the submit button while the request is in flight")

// Bad
it("works")
it("handles error")
it("test login")
```

### General rules

- A test name should be understandable to someone who has never read the source code.
- If a test name is too long, the function under test may be doing too much.
- Never use sequential numbering (`test_1`, `test_2`). Numbers communicate nothing.

---

## Mocking Philosophy

Mocks are a tool for isolating the unit under test from external systems. Used well, they make tests fast and deterministic. Used badly, they create tests that pass despite broken code.

### What to mock

- **Network boundaries.** HTTP calls, GraphQL endpoints, WebSocket connections. Use MSW for TypeScript, `responses` or `httpx_mock` for Python.
- **Filesystem access.** Use `tmp_path` fixtures (pytest) or in-memory implementations.
- **Time and dates.** Freeze time with `freezegun` (Python) or `vi.useFakeTimers()` (Vitest). Never call `datetime.now()` or `Date.now()` directly in business logic - accept a clock dependency.
- **Third-party services.** Stripe, email providers, cloud storage. Wrap them behind an interface (see Dependency Inversion in CODING-PRINCIPLES.md) and provide a test double.
- **Randomness.** Seed random number generators or inject them as dependencies for any logic that depends on random values.

### What NOT to mock

- **The thing you are testing.** If you mock the function under test, the test proves nothing. This sounds obvious but happens when a test mocks an internal method of the class it is testing.
- **Internal modules in the same package.** If `UserService` calls `PasswordHasher` and both are in `syntek-auth`, test them together. Mocking `PasswordHasher` tests the wiring, not the behaviour.
- **Data structures and value objects.** Never mock a plain object, dataclass, or TypeScript interface. Construct real instances using factories or builders.
- **The database in integration tests.** Integration tests exist to verify real database behaviour. Use testcontainers instead.

### Anti-patterns to avoid

- **Mock-heavy tests that test nothing.** If a test mocks every dependency and only asserts that mocks were called with expected arguments, it is testing the implementation, not the behaviour. These tests break on every refactor and catch no bugs.
- **Partial mocks / spy overuse.** Spying on a method of the object under test couples the test to internal structure. Prefer testing through the public interface.
- **Mocking what you don't own without a wrapper.** If you mock `stripe.PaymentIntent.create` directly in 30 test files, every Stripe SDK update breaks 30 tests. Wrap Stripe behind a `PaymentGateway` interface, mock the interface, and have one integration test that verifies the real Stripe wrapper.

### MSW as the default for TypeScript

MSW intercepts at the network level, meaning the entire client-side code path (fetch calls, error handling, retries, caching) runs for real. This is preferable to mocking `fetch` or `axios` directly because it tests more of the stack with less coupling to implementation details.

```typescript
// Setup in vitest.setup.ts
import { setupServer } from "msw/node";
import { handlers } from "./msw/handlers";

export const server = setupServer(...handlers);

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Set `onUnhandledRequest: "error"` so that any unmocked network call fails the test immediately rather than silently hitting a real endpoint.

---

## Snapshot Testing

Snapshot tests capture the serialised output of a component or function and compare it against a stored reference. They are useful for detecting unintended changes but dangerous when used carelessly.

### When to use snapshots

- **Small, stable component output.** A `Badge` or `Alert` component with limited props is a good candidate. The snapshot is small enough to review meaningfully.
- **Serialised data structures.** API response shapes, GraphQL schema output, or configuration objects where the exact structure matters and changes should be deliberate.
- **Error message formatting.** Snapshot the formatted output of custom error classes to catch unintended changes to error messages.

### When NOT to use snapshots

- **Large component trees.** A snapshot of an entire page or form with dozens of elements is unreadable. Reviewers will blindly approve updates. Use targeted assertions instead.
- **Frequently changing components.** If a component's output changes every sprint, the snapshot adds noise without value. Use behavioural assertions (RTL queries, event handlers).
- **As a substitute for assertions.** A snapshot that passes does not mean the component is correct - it means it has not changed. If you cannot explain what the snapshot is protecting, delete it.

### Rules

- Every snapshot file is committed to version control.
- When a snapshot update is required, the PR description must explain why the output changed.
- If a snapshot test is updated more than three times in a quarter without catching a real bug, replace it with targeted assertions.
- Never snapshot timestamps, random IDs, or other non-deterministic values. Stabilise the output first (freeze time, seed IDs).

### Example: small component snapshot

```tsx
// packages/web/ui-core/src/Badge.test.tsx
import { render } from "@testing-library/react";
import { expect, it } from "vitest";

import { Badge } from "./Badge";

it("renders the default badge", () => {
  const { container } = render(<Badge label="Active" variant="success" />);
  expect(container.firstChild).toMatchSnapshot();
});
```

---

## Error Path and Boundary Testing

The rules state that security-critical paths require negative tests. This section extends that principle to all code: every public function should have tests for the inputs most likely to cause failures.

### Boundary categories to test

**Null and undefined.** What happens when a required argument is `null`, `undefined`, or `None`? The function should either reject it with a clear error or handle it explicitly - never silently produce wrong output.

**Empty collections.** Empty lists, empty strings, empty dictionaries. Functions that aggregate, filter, or transform collections must handle the empty case without raising unexpected exceptions.

**Single-element collections.** Off-by-one errors often appear when a collection has exactly one item. Test the single-element case alongside empty and many-element cases.

**Maximum and minimum lengths.** If a field has a max length (database column, API validation), test at the boundary: one below the limit, exactly at the limit, and one above. The same applies to numeric ranges.

**Unicode and special characters.** Test with emoji, CJK characters, right-to-left text, zero-width joiners, and strings that look like code (`<script>`, `'; DROP TABLE`). This is especially important for any function that handles user input, file names, or search queries.

**Type coercion edge cases.** In TypeScript, test with `0`, `""`, `false`, `NaN`, and `null` - values that are falsy but may be valid inputs. In Python, test with `0`, `0.0`, `""`, `[]`, `{}`, and `False`.

**Concurrent access.** For any operation that reads and writes shared state (database rows, cache entries, file locks), test what happens when two operations run simultaneously. Use `pytest-asyncio` or thread-based tests in Python; use `Promise.all` in TypeScript.

### Example: boundary tests for a validator

```python
# packages/backend/syntek-auth/tests/test_validators.py
import pytest

from syntek_auth.validators import validate_display_name


class TestValidateDisplayName:
    def test_rejects_empty_string(self):
        with pytest.raises(ValueError, match="must not be empty"):
            validate_display_name("")

    def test_rejects_none(self):
        with pytest.raises(TypeError):
            validate_display_name(None)

    def test_accepts_single_character(self):
        assert validate_display_name("A") == "A"

    def test_accepts_exactly_at_max_length(self):
        name = "a" * 100
        assert validate_display_name(name) == name

    def test_rejects_one_over_max_length(self):
        with pytest.raises(ValueError, match="exceeds maximum length"):
            validate_display_name("a" * 101)

    def test_handles_unicode_emoji(self):
        assert validate_display_name("Sam \U0001f415") == "Sam \U0001f415"

    def test_strips_leading_and_trailing_whitespace(self):
        assert validate_display_name("  Sam  ") == "Sam"

    def test_rejects_string_that_is_only_whitespace(self):
        with pytest.raises(ValueError, match="must not be empty"):
            validate_display_name("   ")
```

### Rule of thumb

For every public function, write at least one test for the happy path, one for a rejected input, and one for a boundary condition. If the function accepts a collection, test empty, one, and many. If it accepts a string, test empty, whitespace-only, and a string at the length limit.

---

## Accessibility Testing

Accessibility is not optional. Every user-facing component must be usable with assistive technology. Testing catches regressions before they reach users.

### React Testing Library as the baseline

RTL encourages accessible queries by default. Prefer queries in this order:

1. `getByRole` - the most accessible query; mirrors how screen readers navigate.
2. `getByLabelText` - for form controls.
3. `getByPlaceholderText` - only when a label is genuinely absent (which is itself an accessibility issue).
4. `getByText` - for non-interactive content.
5. `getByTestId` - last resort only. If you need `data-testid` to find an element, it may be missing accessible markup.

If a component cannot be found with `getByRole`, that is a signal that the component has an accessibility problem, not that the test needs a different query.

### axe-core integration

Use `vitest-axe` (Vitest) or `jest-axe` (Jest) to run automated WCAG 2.1 AA checks on rendered components.

```bash
pnpm add -D vitest-axe
```

```tsx
// packages/web/ui-auth/src/LoginForm.a11y.test.tsx
import { render } from "@testing-library/react";
import { axe, toHaveNoViolations } from "vitest-axe";
import { expect, it } from "vitest";

import { LoginForm } from "./LoginForm";

expect.extend(toHaveNoViolations);

it("has no accessibility violations", async () => {
  const { container } = render(<LoginForm onSubmit={() => {}} />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

### Playwright accessibility checks

Run axe in E2E tests to catch violations that only appear with real browser rendering, CSS, and dynamic content.

```typescript
// packages/web/ui-auth/tests/e2e/login.a11y.spec.ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("login page has no accessibility violations", async ({ page }) => {
  await page.goto("/login");

  const results = await new AxeBuilder({ page })
    .withTags(["wcag2a", "wcag2aa"])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

### Rules

- Every new component PR includes at least one `axe` assertion.
- Playwright E2E tests include an accessibility scan for every page-level test.
- `getByTestId` is not permitted unless the element genuinely has no accessible role, label, or text - and the reason is documented in a code comment.
- Colour contrast, focus management, and keyboard navigation are verified in the manual testing checklist (see `MANUAL-TESTING.md`).

---

## Performance and Load Testing

Performance testing is in scope for any service that handles multi-tenant traffic or processes concurrent requests. It is not required for every PR but must be run before major releases and after significant architectural changes.

### Tools

| Purpose              | Tool    | Language |
|----------------------|---------|----------|
| HTTP load testing    | Locust  | Python   |
| Scripted load tests  | k6      | JavaScript |
| Database benchmarks  | pgbench | SQL      |

### When to run performance tests

- Before any release that changes database queries, caching, or serialisation in a high-traffic path.
- After adding a new tenant to the multi-tenant platform.
- When introducing a new backing service (cache layer, message queue, search index).
- When a production incident is traced to a performance regression.

### Locust example

```python
# tests/performance/locustfile.py
from locust import HttpUser, task, between


class AuthFlowUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def login(self):
        self.client.post("/api/auth/login", json={
            "email": "loadtest@example.com",
            "password": "test-password-123",
        })

    @task(3)
    def get_profile(self):
        self.client.get("/api/users/me", headers={
            "Authorization": "Bearer <test-token>",
        })
```

```bash
# Run locally against staging
locust -f tests/performance/locustfile.py --host=https://staging.example.com
```

### Rules

- Performance tests never run against production databases with real user data.
- Use a dedicated staging environment or a testcontainers-based local setup with seeded data.
- Performance test results (response times, error rates, throughput) are recorded in the release notes when they influence a release decision.
- Establish baselines for critical endpoints. A regression of more than 20% in p95 response time requires investigation before release.

---

## Flaky Test Policy

A flaky test is any test that passes and fails without a code change. Flaky tests erode trust in the test suite and train developers to ignore failures. They are treated as bugs.

### Response process

1. **Detect.** CI flags any test that fails on a retry but passed on the previous commit. Developers who encounter a flaky test in local development report it immediately.
2. **Quarantine.** The flaky test is moved to a quarantine marker (`@pytest.mark.quarantine` in Python, `.skip("quarantine: TICKET-123")` in Vitest/Jest) within 24 hours. Quarantined tests still run in CI but do not block the build.
3. **Fix.** The flaky test must be fixed or deleted within 5 working days. The fix is tracked with a ticket.
4. **Restore.** Once fixed, the quarantine marker is removed and the test re-enters the main suite.

### Common causes and fixes

| Cause                         | Fix                                                           |
|-------------------------------|---------------------------------------------------------------|
| Test depends on real time     | Freeze time (`freezegun`, `vi.useFakeTimers()`)              |
| Test depends on execution order | Ensure each test sets up and tears down its own state        |
| Test depends on network       | Mock all network calls (MSW, `responses`)                    |
| Test depends on random values | Seed the random number generator or inject it                |
| Race condition in async code  | Use proper async assertions (`waitFor`, `eventually`)        |
| Shared mutable state          | Isolate state per test (fresh container, fresh factory data) |

### Rules

- A flaky test that has been quarantined for more than 5 working days without a fix is deleted. The coverage gap is documented and a new, stable test is written.
- Never "fix" a flaky test by adding retries, sleeps, or increased timeouts. These mask the root cause.
- If the same test flakes more than twice after being "fixed", escalate - the underlying design likely has a concurrency or state isolation problem.

---

## CI Integration

All tests run in CI on every push and pull request. The CI pipeline is the single source of truth for whether code is ready to merge.

### Pipeline stages

| Stage              | What runs                                           | Trigger            | Blocks merge |
|--------------------|-----------------------------------------------------|--------------------|--------------|
| Lint               | Ruff (Python), ESLint (TS), markdownlint            | Every push         | Yes          |
| Unit tests         | pytest (unit), Vitest, Jest                         | Every push         | Yes          |
| Integration tests  | pytest (integration) with testcontainers            | Every push         | Yes          |
| Coverage check     | pytest-cov, Vitest coverage, Jest coverage          | Every push         | Yes          |
| Accessibility      | vitest-axe, Playwright axe                          | Every push         | Yes          |
| E2E tests          | Playwright, Maestro                                 | PR only            | Yes          |
| Performance tests  | Locust / k6 against staging                         | Release branch only | No (advisory) |

### Testcontainers in CI

Testcontainers requires Docker-in-Docker or a Docker socket mount. The CI runner must have Docker available. Configure the testcontainers connection in the CI environment:

```yaml
# .github/workflows/test.yml (relevant section)
services:
  docker:
    image: docker:dind
    options: --privileged

env:
  TESTCONTAINERS_RYUK_DISABLED: "true"  # Not needed in ephemeral CI
  DOCKER_HOST: "unix:///var/run/docker.sock"
```

### Playwright in CI

Playwright requires browser binaries. Install them in a setup step and cache them:

```yaml
- name: Install Playwright browsers
  run: pnpm exec playwright install --with-deps chromium
```

Playwright E2E tests run against a locally started dev server in CI, not against staging. This keeps E2E tests fast and reproducible.

### Maestro in CI

Maestro E2E tests for React Native require an emulator. Run these on a dedicated CI runner with Android emulator support, or gate them to a nightly/weekly schedule if emulator setup is too slow for every PR.

### Rules

- Every CI stage must complete in under 15 minutes. If a stage consistently exceeds this, investigate parallelisation or test splitting.
- CI failures block merge. No exceptions, no manual overrides except by a maintainer with a documented reason.
- The CI configuration is version-controlled alongside the code. Changes to CI require the same review process as code changes.
- Secrets used in CI (API keys for staging, Docker credentials) are stored in the CI platform's secrets manager, never in the repository.

---

## Per-Package Testing Files

Every package in this repo carries two testing files:

### TEST-STATUS.md

Tracks the automated test suite: what tests exist, what each one verifies, and whether it currently passes. Updated after each test run by the contributor or CI.

**Location:**

- `packages/backend/syntek-{name}/TEST-STATUS.md`
- `packages/web/{name}/TEST-STATUS.md`
- `mobile/{name}/TEST-STATUS.md`

**Template:** `docs/GUIDES/templates/TEST-STATUS.md`

### docs/MANUAL-TESTING.md

Step-by-step guide for a human tester to verify the package works correctly. Covers happy paths, error paths, security edge cases, and a regression checklist.

**Location:**

- `packages/backend/syntek-{name}/docs/MANUAL-TESTING.md`
- `packages/web/{name}/docs/MANUAL-TESTING.md`
- `mobile/{name}/docs/MANUAL-TESTING.md`

**Template:** `docs/GUIDES/templates/MANUAL-TESTING.md`

**Convention:** Both files are created when a new module is scaffolded (`/add-module`). `TEST-STATUS.md` is kept up to date as tests are written. `MANUAL-TESTING.md` is written alongside the first implementation PR and updated whenever behaviour changes.

---

## Rules and Principles

1. Every new public function has at least one unit test.
2. Every GraphQL mutation/query has an integration test covering the happy path, an auth failure, and an invalid input case.
3. Tests are deterministic - no real time, random values, or live network calls.
4. Tests are independent - each test sets up its own state.
5. Follow Arrange-Act-Assert in every test.
6. Test behaviour, not implementation.
7. Unit tests complete in under 100ms each.
8. Security-critical paths (auth, encryption, RBAC) have negative tests that verify rejection of invalid or malicious input.
9. Test code is held to the same standard as production code.
10. Every public function has at least one boundary test (see Error Path and Boundary Testing).
11. Mock at system boundaries, not within the unit under test (see Mocking Philosophy).
12. Flaky tests are quarantined within 24 hours and fixed within 5 working days (see Flaky Test Policy).
13. Coverage thresholds are enforced in CI and never bypassed (see Coverage Thresholds).
14. Accessibility assertions are required on every user-facing component (see Accessibility Testing).
