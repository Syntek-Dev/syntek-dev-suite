# PII Middleware and Guards

## Metadata

| Property | Value |
|----------|-------|
| **Version** | 2.0.0 |
| **Last Updated** | December 2025 |
| **Status** | Stable |

## Framework Versions Tested

| Framework | Version | Tested Date |
|-----------|---------|-------------|
| Laravel | 12.x | 20/12/2025 |
| Django | 6.x | 20/12/2025 |
| Strawberry GraphQL | 0.220+ | 20/12/2025 |
| Next.js | 16.x | 20/12/2025 |
| PHP | 8.4 | 20/12/2025 |
| Python | 3.14 | 20/12/2025 |
| Node.js | 24.x LTS | 20/12/2025 |
| TypeScript | 5.9 | 20/12/2025 |

---

## Table of Contents

- [Overview](#overview)
- [Laravel (TALL Stack)](#laravel-tall-stack)
- [Django/Wagtail (Strawberry GraphQL)](#djangowagtail-strawberry-graphql)
- [Node.js/TypeScript (NestJS GraphQL)](#nodejstypescript-nestjs-graphql)


## Overview

Middleware and guards protect PII-containing endpoints by:
1. Checking user permissions before allowing access
2. Logging all access attempts for audit trail
3. Returning 403 Forbidden when permission denied

---

## Laravel (TALL Stack)

### HTTP Middleware

Laravel 12.x provides robust middleware support for protecting PII endpoints with permission checks and comprehensive audit logging.

```php
<?php
/**
 * RequirePiiAccess.php
 *
 * Middleware that restricts access to PII-containing endpoints.
 * Logs all access attempts for comprehensive audit trail.
 * Compatible with Laravel 12.x and PHP 8.4.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class RequirePiiAccess
{
    /**
     * Handles an incoming request to a PII endpoint.
     * Rejects requests from users without pii.access permission.
     *
     * @param Request $request The incoming HTTP request
     * @param Closure $next The next middleware in the pipeline
     * @return Response The HTTP response
     */
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        // Check if user is authenticated
        if (!$user) {
            Log::warning('PII access denied - unauthenticated request', [
                'path' => $request->path(),
                'ip' => $request->ip(),
                'user_agent' => $request->userAgent(),
            ]);

            return response()->json([
                'error' => 'Authentication required'
            ], 401);
        }

        // Check if user has PII access permission
        if (!$user->hasPermission('pii.access')) {
            Log::warning('PII access denied - insufficient permissions', [
                'user_id' => $user->id,
                'user_email' => $user->email,
                'path' => $request->path(),
                'method' => $request->method(),
                'ip' => $request->ip(),
                'user_agent' => $request->userAgent(),
            ]);

            return response()->json([
                'error' => 'Insufficient permissions to access PII'
            ], 403);
        }

        // Log successful PII access for audit trail
        Log::info('PII endpoint accessed successfully', [
            'user_id' => $user->id,
            'user_email' => $user->email,
            'path' => $request->path(),
            'method' => $request->method(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'timestamp' => now()->toIso8601String(),
        ]);

        return $next($request);
    }
}
```

### Route Protection

```php
<?php
/**
 * routes/api.php
 *
 * API routes with PII protection middleware.
 * All routes in this group require authentication and pii.access permission.
 */

use App\Http\Controllers\UserController;
use App\Http\Middleware\RequirePiiAccess;
use Illuminate\Support\Facades\Route;

// PII-protected routes
Route::middleware(['auth:sanctum', RequirePiiAccess::class])->group(function () {
    // Retrieve PII for a specific user
    Route::get('/users/{uuid}/pii', [UserController::class, 'getPii'])
        ->name('users.pii.show');

    // Update PII for a specific user
    Route::patch('/users/{uuid}/pii', [UserController::class, 'updatePii'])
        ->name('users.pii.update');

    // Export user data (GDPR compliance)
    Route::post('/users/{uuid}/export', [UserController::class, 'exportData'])
        ->name('users.export');

    // Bulk PII operations
    Route::post('/users/bulk-export', [UserController::class, 'bulkExport'])
        ->name('users.bulk-export');
});
```

### Middleware Registration

```php
<?php
/**
 * bootstrap/app.php
 *
 * Register the PII access middleware alias.
 * Laravel 12.x application bootstrap configuration.
 */

use App\Http\Middleware\RequirePiiAccess;
use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function ($middleware) {
        // Register middleware alias for use in routes
        $middleware->alias([
            'pii.access' => RequirePiiAccess::class,
        ]);
    })
    ->create();
```

---

## Django/Wagtail (Strawberry GraphQL)

```python
"""
schema/permissions/pii_permission.py

GraphQL permission class that restricts access to PII-containing fields.
Logs all access attempts for audit trail.
"""

import logging
from strawberry.permission import BasePermission
from strawberry.types import Info
from typing import Any

logger = logging.getLogger(__name__)


class RequirePiiAccess(BasePermission):
    """
    Permission class that restricts access to PII fields.
    Rejects requests from users without pii_access permission.
    """

    message = "Insufficient permissions to access PII"

    def has_permission(self, source: Any, info: Info, **kwargs) -> bool:
        """
        Checks if the requesting user has pii_access permission.

        Args:
            source: The parent object being resolved
            info: GraphQL resolver info containing request context
            **kwargs: Additional keyword arguments

        Returns:
            bool: True if user has permission, False otherwise
        """
        request = info.context.get('request')
        if not request or not request.user.is_authenticated:
            logger.warning('PII access denied - unauthenticated request')
            return False

        has_access = request.user.has_perm('accounts.pii_access')

        if not has_access:
            logger.warning('PII access denied', extra={
                'user_id': request.user.id,
                'path': request.path,
                'ip': self._get_client_ip(request),
            })
        else:
            logger.info('PII endpoint accessed', extra={
                'user_id': request.user.id,
                'path': request.path,
                'ip': self._get_client_ip(request),
            })

        return has_access

    def _get_client_ip(self, request) -> str:
        """Extracts the client IP address from the request."""
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            return x_forwarded_for.split(',')[0].strip()
        return request.META.get('REMOTE_ADDR', '')
```

### Usage

```python
import strawberry
from .permissions import RequirePiiAccess

@strawberry.type
class Query:
    @strawberry.field(permission_classes=[RequirePiiAccess])
    def user_pii(self, info: Info, user_id: str) -> UserPiiType:
        """Retrieves user PII data (requires pii_access permission)."""
        return get_user_pii(user_id)
```

---

## Node.js/TypeScript (NestJS GraphQL)

```typescript
/**
 * pii-access.guard.ts
 *
 * Guard that restricts access to PII-containing fields/queries.
 * Logs all access attempts for audit trail.
 */

import { Injectable, CanActivate, ExecutionContext, Logger } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

@Injectable()
export class PiiAccessGuard implements CanActivate {
  private readonly logger = new Logger(PiiAccessGuard.name);

  /**
   * Checks if the requesting user has pii.access permission.
   *
   * @param context - The execution context
   * @returns True if user has permission, false otherwise
   */
  canActivate(context: ExecutionContext): boolean {
    const ctx = GqlExecutionContext.create(context);
    const { user, req } = ctx.getContext();

    if (!user) {
      this.logger.warn('PII access denied - unauthenticated request');
      return false;
    }

    const hasAccess = user.permissions?.includes('pii.access') ?? false;

    if (!hasAccess) {
      this.logger.warn('PII access denied', {
        userId: user.id,
        path: req?.path,
        ip: req?.ip,
      });
    } else {
      this.logger.log('PII endpoint accessed', {
        userId: user.id,
        path: req?.path,
        ip: req?.ip,
      });
    }

    return hasAccess;
  }
}
```

### Usage

```typescript
import { UseGuards } from '@nestjs/common';
import { Resolver, Query } from '@nestjs/graphql';
import { PiiAccessGuard } from './guards/pii-access.guard';

@Resolver()
export class UserResolver {
  @Query(() => UserPiiType)
  @UseGuards(PiiAccessGuard)
  async userPii(@Args('userId') userId: string): Promise<UserPiiType> {
    return this.userService.getPii(userId);
  }
}
```
