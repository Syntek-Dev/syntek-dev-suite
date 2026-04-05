# Row Level Security (RLS)

**Last Updated**: 05/04/2026
**Version**: 1.0.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [When to Apply RLS](#when-to-apply-rls)
- [PostgreSQL — Native RLS](#postgresql--native-rls)
  - [Basic Policy Setup](#basic-policy-setup)
  - [Multi-Tenancy Policies](#multi-tenancy-policies)
  - [Role-Based Policies](#role-based-policies)
  - [Passing Context to PostgreSQL](#passing-context-to-postgresql)
  - [Supabase RLS](#supabase-rls)
- [SQL Server — Native RLS](#sql-server--native-rls)
- [MySQL / MariaDB — No Native RLS](#mysql--mariadb--no-native-rls)
- [SQLite — No Native RLS](#sqlite--no-native-rls)
- [Framework Integration](#framework-integration)
  - [Laravel / Eloquent](#laravel--eloquent)
  - [Django ORM](#django-orm)
  - [Prisma](#prisma)
  - [TypeORM](#typeorm)
- [Testing RLS Policies](#testing-rls-policies)
- [Checklist](#checklist)

---

## Overview

Row Level Security (RLS) is a database-engine feature that filters rows returned by — or affected by — a query based on the identity or attributes of the executing session. The filter runs inside the database, before results reach the application. This means a misconfigured application query cannot accidentally expose rows belonging to another user or tenant.

RLS is a **defence-in-depth layer**, not a replacement for application-level authorisation. Both must exist:

| Layer | What it does |
|-------|-------------|
| Application authorisation | Controls which endpoints and operations a user may invoke |
| RLS (database) | Ensures a query can only touch rows the current session is authorised to see or modify |

**CRITICAL:** RLS is mandatory on any table that stores user-scoped or tenant-scoped data. A missing RLS policy on such a table is a security finding.

---

## When to Apply RLS

Apply RLS policies to every table where:

- Rows are owned by a specific user (`user_id`, `owner_id`, `created_by`)
- Rows are scoped to a tenant (`tenant_id`, `organisation_id`, `account_id`)
- Rows carry a classification that restricts visibility (e.g., only admins may read draft records)
- The table stores PII or financial data that must not leak across account boundaries

Tables that are **global reference data** (currencies, countries, static configuration) do not require RLS.

---

## PostgreSQL — Native RLS

PostgreSQL has first-class RLS support. All user-scoped and tenant-scoped tables **MUST** use it.

### Basic Policy Setup

```sql
-- Enable RLS on the table (does not enforce anything yet)
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Force RLS even for the table owner
-- CRITICAL: Without this, the table owner bypasses all policies
ALTER TABLE orders FORCE ROW LEVEL SECURITY;

-- SELECT policy: a user may only read their own rows
CREATE POLICY orders_select_own
    ON orders
    FOR SELECT
    USING (user_id = current_setting('app.current_user_id')::uuid);

-- INSERT policy: rows must be created with the current user's id
CREATE POLICY orders_insert_own
    ON orders
    FOR INSERT
    WITH CHECK (user_id = current_setting('app.current_user_id')::uuid);

-- UPDATE policy: a user may only update their own rows
CREATE POLICY orders_update_own
    ON orders
    FOR UPDATE
    USING (user_id = current_setting('app.current_user_id')::uuid)
    WITH CHECK (user_id = current_setting('app.current_user_id')::uuid);

-- DELETE policy: a user may only delete their own rows
CREATE POLICY orders_delete_own
    ON orders
    FOR DELETE
    USING (user_id = current_setting('app.current_user_id')::uuid);
```

**Key rules:**
- `USING` filters rows for `SELECT`, `UPDATE`, `DELETE`.
- `WITH CHECK` validates rows being written for `INSERT`, `UPDATE`.
- Always set **both** on `UPDATE` policies so a user cannot move a row out of their own scope.
- `FORCE ROW LEVEL SECURITY` is **mandatory** — without it the table owner (often the migration user) bypasses all policies.

### Multi-Tenancy Policies

```sql
-- Tenant-scoped table
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
ALTER TABLE invoices FORCE ROW LEVEL SECURITY;

-- Any session variable can be used; use a namespaced key to avoid conflicts
CREATE POLICY invoices_tenant_isolation
    ON invoices
    FOR ALL
    USING (tenant_id = current_setting('app.current_tenant_id')::uuid)
    WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::uuid);

-- Admin bypass: a separate application role may read all tenants
CREATE POLICY invoices_admin_read_all
    ON invoices
    FOR SELECT
    TO app_admin_role
    USING (true);
```

### Role-Based Policies

```sql
-- Create application database roles (not database superusers)
CREATE ROLE app_user  NOLOGIN;
CREATE ROLE app_admin NOLOGIN;

-- Grant table access to roles
GRANT SELECT, INSERT, UPDATE, DELETE ON orders TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON orders TO app_admin;

-- app_user may only see their own rows
CREATE POLICY orders_user_scope
    ON orders
    FOR ALL
    TO app_user
    USING (user_id = current_setting('app.current_user_id')::uuid)
    WITH CHECK (user_id = current_setting('app.current_user_id')::uuid);

-- app_admin may see all rows
CREATE POLICY orders_admin_scope
    ON orders
    FOR ALL
    TO app_admin
    USING (true)
    WITH CHECK (true);
```

### Passing Context to PostgreSQL

The application must set session variables before executing queries. The approach depends on the connection method.

**Connection pool approach (PgBouncer / pgpool):**
Use `SET LOCAL` inside a transaction — it is reset automatically when the transaction ends.

```sql
BEGIN;
SET LOCAL app.current_user_id  = 'a1b2c3d4-e5f6-7890-abcd-ef1234567890';
SET LOCAL app.current_tenant_id = 'f1e2d3c4-b5a6-7890-fedc-ba0987654321';

-- All queries inside this transaction are now scoped automatically
SELECT * FROM orders;
UPDATE invoices SET status = 'paid' WHERE id = $1;

COMMIT;
```

**Direct connection approach:**
Use `SET` for the lifetime of the connection, reset on connection return.

```sql
SET app.current_user_id = 'a1b2c3d4-e5f6-7890-abcd-ef1234567890';
```

**Using `current_user` or `session_user`:**
When the application authenticates as a separate PostgreSQL role per user (uncommon but possible):

```sql
CREATE POLICY orders_by_pg_user
    ON orders
    FOR ALL
    USING (owner = current_user);
```

### Supabase RLS

Supabase uses PostgreSQL RLS with `auth.uid()` from the `auth` schema.

```sql
-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE profiles FORCE ROW LEVEL SECURITY;

-- Users may only read their own profile
CREATE POLICY profiles_select_own
    ON profiles
    FOR SELECT
    USING (id = auth.uid());

-- Users may only update their own profile
CREATE POLICY profiles_update_own
    ON profiles
    FOR UPDATE
    USING (id = auth.uid())
    WITH CHECK (id = auth.uid());

-- Users may only insert their own profile
CREATE POLICY profiles_insert_own
    ON profiles
    FOR INSERT
    WITH CHECK (id = auth.uid());

-- Tenant / organisation scoping
CREATE POLICY documents_tenant_scope
    ON documents
    FOR ALL
    USING (
        organisation_id IN (
            SELECT organisation_id
            FROM organisation_members
            WHERE user_id = auth.uid()
        )
    )
    WITH CHECK (
        organisation_id IN (
            SELECT organisation_id
            FROM organisation_members
            WHERE user_id = auth.uid()
        )
    );
```

**Supabase service role bypass:**
The `service_role` key bypasses RLS entirely. Never expose it to clients. Use only in trusted server-side code.

---

## SQL Server — Native RLS

SQL Server supports RLS via security predicates attached to a security policy.

```sql
-- Predicate function: returns 1 (allow) or 0 (deny)
-- The function must be schema-bound
CREATE FUNCTION dbo.fn_orders_user_predicate(@user_id UNIQUEIDENTIFIER)
RETURNS TABLE
WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_result
    WHERE @user_id = CAST(SESSION_CONTEXT(N'app_current_user_id') AS UNIQUEIDENTIFIER);
GO

-- Attach the predicate to the table as a filter policy
CREATE SECURITY POLICY dbo.orders_rls_policy
    ADD FILTER PREDICATE dbo.fn_orders_user_predicate(user_id) ON dbo.orders,
    ADD BLOCK PREDICATE  dbo.fn_orders_user_predicate(user_id) ON dbo.orders AFTER INSERT,
    ADD BLOCK PREDICATE  dbo.fn_orders_user_predicate(user_id) ON dbo.orders AFTER UPDATE
WITH (STATE = ON);
GO
```

**Setting session context (application side):**

```sql
-- Set before executing queries; cleared automatically at session end
EXEC sp_set_session_context N'app_current_user_id',
     N'a1b2c3d4-e5f6-7890-abcd-ef1234567890';
```

**Rules:**
- `FILTER PREDICATE` silently hides rows that do not match (equivalent to PostgreSQL `USING`).
- `BLOCK PREDICATE AFTER INSERT` / `AFTER UPDATE` prevent writes to rows outside the current scope (equivalent to PostgreSQL `WITH CHECK`).
- Use `read_only = 1` in `sp_set_session_context` so client code cannot override the context value.

---

## MySQL / MariaDB — No Native RLS

MySQL and MariaDB do not support native RLS. The required approach is a combination of:

1. **Database views** scoped to the current user/tenant
2. **Stored procedure wrappers** that enforce scoping
3. **Application-enforced query scoping** via ORM global scopes

**CRITICAL:** Because MySQL lacks native RLS, the application layer bears full responsibility for row isolation. Every ORM model that maps to a user-scoped or tenant-scoped table **MUST** apply a global scope. This is not optional.

### MySQL View-Based Scoping (reference pattern)

```sql
-- Underlying table (no native RLS)
CREATE TABLE orders (
    id          CHAR(36)     NOT NULL PRIMARY KEY,
    user_id     CHAR(36)     NOT NULL,
    tenant_id   CHAR(36)     NOT NULL,
    total       DECIMAL(10,2) NOT NULL,
    created_at  DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_orders_user_id   (user_id),
    INDEX idx_orders_tenant_id (tenant_id)
);

-- Scoped view — the application uses this view, not the raw table
-- The @current_user_id session variable must be set before any query
CREATE VIEW v_orders_user AS
    SELECT * FROM orders
    WHERE user_id = @current_user_id;

-- Stored procedure with explicit scoping
CREATE PROCEDURE get_user_orders(IN p_user_id CHAR(36))
BEGIN
    SELECT * FROM orders WHERE user_id = p_user_id;
END;
```

**Setting session variable (application side):**

```sql
SET @current_user_id = 'a1b2c3d4-e5f6-7890-abcd-ef1234567890';
```

**Important:** MySQL session variables are connection-scoped, not transaction-scoped. With connection pooling, always reset variables at the start of each request, not at the end.

---

## SQLite — No Native RLS

SQLite has no RLS support. All row isolation is application-enforced. Always filter by `user_id` or `tenant_id` in every query. Use parameterised queries exclusively.

```sql
-- Every query must include the user scope explicitly
SELECT * FROM orders WHERE user_id = ? AND id = ?;
UPDATE orders SET status = ? WHERE id = ? AND user_id = ?;
DELETE FROM orders WHERE id = ? AND user_id = ?;
```

---

## Framework Integration

### Laravel / Eloquent

Laravel global scopes enforce row isolation at the ORM level. **Every model on a user-scoped or tenant-scoped table must use one.**

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

/**
 * UserScope.php
 *
 * Global Eloquent scope that restricts all queries on a model to rows
 * belonging to the currently authenticated user. Applied automatically
 * by the HasUserScope trait on any model that uses it.
 */
class UserScope implements Scope
{
    /**
     * Applies the user ownership scope to the given Eloquent query builder.
     *
     * @param Builder $builder The Eloquent query builder instance
     * @param Model   $model   The model being queried
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where($model->getTable() . '.user_id', auth()->id());
    }
}
```

```php
<?php

namespace App\Models\Traits;

use App\Models\Scopes\UserScope;

/**
 * HasUserScope.php
 *
 * Trait that registers the UserScope global scope on any Eloquent model.
 * Add this trait to every model whose table stores per-user rows.
 */
trait HasUserScope
{
    protected static function bootHasUserScope(): void
    {
        static::addGlobalScope(new UserScope());
    }
}
```

```php
<?php

namespace App\Models;

use App\Models\Traits\HasUserScope;
use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    use HasUserScope;

    // All queries are now automatically scoped to the authenticated user
}
```

**For PostgreSQL with Laravel**, set the session variable before queries using a service provider or middleware:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

/**
 * SetPostgresRlsContext.php
 *
 * Middleware that sets PostgreSQL session variables required by RLS policies.
 * Must run after authentication middleware so that auth()->id() is available.
 */
class SetPostgresRlsContext
{
    /**
     * Sets the PostgreSQL RLS session context for the current authenticated user.
     *
     * @param Request $request The incoming HTTP request
     * @param Closure $next    The next middleware in the pipeline
     * @return mixed
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (auth()->check()) {
            DB::statement("SET LOCAL app.current_user_id = ?", [auth()->id()]);

            if (auth()->user()->tenant_id) {
                DB::statement(
                    "SET LOCAL app.current_tenant_id = ?",
                    [auth()->user()->tenant_id]
                );
            }
        }

        return $next($request);
    }
}
```

Register in `app/Http/Kernel.php` (Laravel 10) or `bootstrap/app.php` (Laravel 11+) **after** `\App\Http\Middleware\Authenticate::class`.

### Django ORM

Django does not support PostgreSQL RLS out of the box. Use a custom database backend or signals to set session variables per request.

```python
# middleware/rls.py

"""
rls.py

Django middleware that sets the PostgreSQL session variable required by RLS
policies before each request is processed. Runs after authentication
middleware to ensure the user is available.
"""

from django.db import connection


class SetPostgresRlsContextMiddleware:
    """
    Sets the PostgreSQL app.current_user_id session variable so that RLS
    policies can filter rows by the authenticated user.
    """

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        if request.user.is_authenticated:
            with connection.cursor() as cursor:
                cursor.execute(
                    "SET LOCAL app.current_user_id = %s",
                    [str(request.user.id)],
                )
                if hasattr(request.user, 'tenant_id') and request.user.tenant_id:
                    cursor.execute(
                        "SET LOCAL app.current_tenant_id = %s",
                        [str(request.user.tenant_id)],
                    )

        return self.get_response(request)
```

Add to `MIDDLEWARE` in `settings.py` **after** `django.contrib.auth.middleware.AuthenticationMiddleware`.

**Application-level queryset scoping (required for MySQL/MariaDB/SQLite):**

```python
# managers.py

"""
managers.py

Custom Django model managers that enforce row-level scoping at the
application layer for databases without native RLS support.
"""

from django.db import models


class UserScopedManager(models.Manager):
    """
    Model manager that automatically filters querysets to rows owned by
    the currently authenticated user. Requires the view layer to pass the
    user via get_queryset(request.user).
    """

    def for_user(self, user):
        """
        Returns a queryset filtered to rows belonging to the given user.

        Args:
            user: The authenticated User instance

        Returns:
            QuerySet filtered by user_id
        """
        return self.get_queryset().filter(user=user)


class TenantScopedManager(models.Manager):
    """
    Model manager that automatically filters querysets to rows belonging
    to the given tenant.
    """

    def for_tenant(self, tenant):
        """
        Returns a queryset filtered to rows belonging to the given tenant.

        Args:
            tenant: The authenticated tenant instance or tenant_id

        Returns:
            QuerySet filtered by tenant
        """
        return self.get_queryset().filter(tenant=tenant)
```

### Prisma

Prisma does not support native RLS natively. Use middleware to set PostgreSQL session variables.

```typescript
// lib/prisma.ts

/**
 * prisma.ts
 *
 * Prisma client factory that sets PostgreSQL RLS session variables before
 * executing queries. Creates a new client per request or reuses a singleton
 * depending on the deployment context.
 */

import { PrismaClient } from '@prisma/client';

/**
 * Creates a Prisma client that sets PostgreSQL RLS context variables
 * before every transaction.
 *
 * @param userId   - UUID of the authenticated user
 * @param tenantId - UUID of the authenticated tenant (optional)
 * @returns PrismaClient configured with RLS context
 */
export function createScopedPrismaClient(
  userId: string,
  tenantId?: string,
): PrismaClient {
  const prisma = new PrismaClient();

  // Prisma middleware runs before every query
  prisma.$use(async (params, next) => {
    return prisma.$transaction(async (tx) => {
      await tx.$executeRawUnsafe(
        `SET LOCAL app.current_user_id = '${userId}'`,
      );

      if (tenantId) {
        await tx.$executeRawUnsafe(
          `SET LOCAL app.current_tenant_id = '${tenantId}'`,
        );
      }

      return next(params);
    });
  });

  return prisma;
}
```

**Note:** For MySQL/MariaDB with Prisma, add a `where: { userId }` clause to every query via Prisma middleware or use extension patterns to enforce scoping.

### TypeORM

```typescript
// subscribers/RlsSubscriber.ts

/**
 * RlsSubscriber.ts
 *
 * TypeORM event subscriber that sets PostgreSQL RLS session variables
 * before every query connection is used. Registered globally via
 * the DataSource configuration.
 */

import {
  DataSource,
  EntitySubscriberInterface,
  EventSubscriber,
  QueryRunner,
} from 'typeorm';

@EventSubscriber()
export class RlsSubscriber implements EntitySubscriberInterface {
  /**
   * Sets the PostgreSQL RLS session variable before each query runner is used.
   *
   * @param queryRunner - The TypeORM query runner instance
   * @param userId      - UUID of the authenticated user
   * @param tenantId    - UUID of the authenticated tenant (optional)
   */
  static async setContext(
    queryRunner: QueryRunner,
    userId: string,
    tenantId?: string,
  ): Promise<void> {
    await queryRunner.query(
      `SET LOCAL app.current_user_id = '${userId}'`,
    );

    if (tenantId) {
      await queryRunner.query(
        `SET LOCAL app.current_tenant_id = '${tenantId}'`,
      );
    }
  }
}
```

---

## Testing RLS Policies

**CRITICAL:** RLS policies MUST be tested with at least these scenarios:

| Test Scenario | Expected Result |
|--------------|----------------|
| User A queries rows owned by User A | Returns User A's rows only |
| User A queries rows owned by User B | Returns empty set (not an error) |
| User A inserts a row with their own `user_id` | Succeeds |
| User A inserts a row with User B's `user_id` | Blocked by `WITH CHECK` policy |
| User A updates a row owned by User A | Succeeds |
| User A updates a row owned by User B | Blocked (0 rows affected, not an error) |
| User A deletes a row owned by User A | Succeeds |
| User A deletes a row owned by User B | Blocked (0 rows affected, not an error) |
| Admin role queries any row | Returns all rows (via admin policy) |
| No session context set | Returns empty set or raises `unrecognized configuration parameter` |
| `FORCE ROW LEVEL SECURITY` active with table owner session | Policies enforced even for owner |

**PostgreSQL test pattern:**

```sql
-- Test as a regular session with context set
BEGIN;
SET LOCAL app.current_user_id = 'user-a-uuid';

-- Should return only User A's rows
SELECT count(*) FROM orders;  -- expect: only User A's rows

-- Should be blocked
INSERT INTO orders (id, user_id, total)
VALUES (gen_random_uuid(), 'user-b-uuid', 99.99);  -- expect: policy violation

ROLLBACK;
```

---

## Checklist

Before marking any table or schema work complete, verify:

- [ ] RLS is enabled (`ENABLE ROW LEVEL SECURITY`) on all user-scoped and tenant-scoped tables
- [ ] `FORCE ROW LEVEL SECURITY` is set on all RLS-enabled tables
- [ ] `SELECT`, `INSERT`, `UPDATE`, and `DELETE` policies are defined (or a single `FOR ALL` policy with correct `USING` + `WITH CHECK`)
- [ ] `UPDATE` policies include both `USING` and `WITH CHECK`
- [ ] Session context variables are set by middleware before every query (PostgreSQL / SQL Server)
- [ ] Application-level ORM global scopes are applied on MySQL/MariaDB/SQLite tables (compensating control)
- [ ] Admin/service role bypass policies are intentional and documented
- [ ] Migration files enable RLS in the same migration that creates the table
- [ ] RLS policies are tested with cross-user access attempts that must return empty results, not errors
- [ ] The migration summary (`docs/DATABASE/MIGRATIONS/`) documents RLS policies applied
