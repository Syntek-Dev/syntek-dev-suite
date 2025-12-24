# Rate Limiting for PII Endpoints

## Metadata

| Property | Value |
|----------|-------|
| **Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Status** | Stable |

## Framework Versions Tested

| Framework | Version | Tested Date |
|-----------|---------|-------------|
| Laravel | 12.x | 20/12/2025 |
| PHP | 8.4 | 20/12/2025 |
| MariaDB | 12.x | 20/12/2025 |
| Django | 6.x | 20/12/2025 |
| Python | 3.14 | 20/12/2025 |
| PostgreSQL | 18.x | 20/12/2025 |
| django-ratelimit | 4.x | 20/12/2025 |
| Next.js | 16.x | 20/12/2025 |
| Node.js | 24.x | 20/12/2025 |
| TypeScript | 5.9 | 20/12/2025 |
| React Native | 0.83.x | 20/12/2025 |

---

## Table of Contents

- [Rate Limiting for PII Endpoints](#rate-limiting-for-pii-endpoints)
  - [Metadata](#metadata)
  - [Framework Versions Tested](#framework-versions-tested)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [TALL Stack (Laravel)](#tall-stack-laravel)
    - [Installation](#installation)
    - [Rate Limiter Configuration](#rate-limiter-configuration)
    - [Route Configuration](#route-configuration)
    - [Custom Middleware (Optional)](#custom-middleware-optional)
    - [Controller Example](#controller-example)
  - [Django/Wagtail](#djangowagtail)
    - [Installation](#installation-1)
    - [Settings Configuration](#settings-configuration)
    - [Rate Limit Decorator](#rate-limit-decorator)
    - [GraphQL Schema Example (Strawberry)](#graphql-schema-example-strawberry)
    - [REST API Example (Django REST Framework)](#rest-api-example-django-rest-framework)
    - [Middleware (Optional)](#middleware-optional)
  - [Node.js/TypeScript (NestJS GraphQL)](#nodejstypescript-nestjs-graphql)
    - [PII Throttle Guard](#pii-throttle-guard)
    - [Module Configuration](#module-configuration)
    - [Usage](#usage)


## Overview

Rate limiting protects PII endpoints from:
1. Brute force attacks
2. Data scraping attempts
3. Resource exhaustion

**Recommended limits for PII endpoints:** 10 requests per minute per user/IP.

---

## TALL Stack (Laravel)

### Installation

```bash
# Laravel 12.x comes with rate limiting support built-in
# No additional packages required
```

### Rate Limiter Configuration

```php
<?php

/**
 * app/Providers/AppServiceProvider.php
 *
 * Rate limiter configuration for PII endpoints.
 * Configured in AppServiceProvider for Laravel 12.x.
 */

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * Initialises rate limiters for different endpoint types.
     *
     * @return void
     */
    public function boot(): void
    {
        // Standard PII endpoint rate limiter
        // 10 requests per minute per user or IP address
        RateLimiter::for('pii', function (Request $request) {
            return Limit::perMinute(10)
                ->by($request->user()?->id ?: $request->ip())
                ->response(function (Request $request, array $headers) {
                    return response()->json([
                        'message' => 'Rate limit exceeded for PII endpoint. Please try again later.',
                        'retry_after' => $headers['Retry-After'] ?? 60,
                    ], 429);
                });
        });

        // Stricter rate limiter for data export endpoints
        // 5 requests per hour per user
        RateLimiter::for('pii-export', function (Request $request) {
            return Limit::perHour(5)
                ->by($request->user()?->id ?: $request->ip())
                ->response(function (Request $request, array $headers) {
                    return response()->json([
                        'message' => 'Export rate limit exceeded. Maximum 5 exports per hour.',
                        'retry_after' => $headers['Retry-After'] ?? 3600,
                    ], 429);
                });
        });
    }
}
```

### Route Configuration

```php
<?php

/**
 * routes/api.php
 *
 * API routes with rate limiting middleware for PII endpoints.
 */

use App\Http\Controllers\Api\UserController;
use Illuminate\Support\Facades\Route;

// PII access routes with standard rate limiting
Route::middleware(['auth:sanctum', 'throttle:pii'])->group(function () {
    // Get user PII data
    Route::get('/users/{uuid}/pii', [UserController::class, 'getPii'])
        ->name('api.users.pii');

    // Update user PII data
    Route::patch('/users/{uuid}/pii', [UserController::class, 'updatePii'])
        ->name('api.users.update-pii');

    // Delete user PII data
    Route::delete('/users/{uuid}/pii', [UserController::class, 'deletePii'])
        ->name('api.users.delete-pii');
});

// PII export routes with stricter rate limiting
Route::middleware(['auth:sanctum', 'throttle:pii-export'])->group(function () {
    // Export user data (GDPR compliance)
    Route::post('/users/export', [UserController::class, 'exportData'])
        ->name('api.users.export');

    // Request data deletion (GDPR compliance)
    Route::post('/users/{uuid}/request-deletion', [UserController::class, 'requestDeletion'])
        ->name('api.users.request-deletion');
});
```

### Custom Middleware (Optional)

```php
<?php

/**
 * app/Http/Middleware/PiiRateLimitMiddleware.php
 *
 * Custom rate limiting middleware for PII endpoints with enhanced logging.
 * Provides additional tracking and monitoring for rate limit violations.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\RateLimiter;
use Symfony\Component\HttpFoundation\Response;

class PiiRateLimitMiddleware
{
    /**
     * Handle an incoming request with custom rate limiting logic.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     * @param  string  $limiterName  The name of the rate limiter to use
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function handle(Request $request, Closure $next, string $limiterName = 'pii'): Response
    {
        $key = $this->resolveRequestKey($request);

        // Check if rate limit is exceeded
        if (RateLimiter::tooManyAttempts($key, $this->maxAttempts($limiterName))) {
            // Log rate limit violation for security monitoring
            Log::warning('PII endpoint rate limit exceeded', [
                'user_id' => $request->user()?->id,
                'ip' => $request->ip(),
                'endpoint' => $request->path(),
                'limiter' => $limiterName,
            ]);

            return response()->json([
                'message' => 'Too many requests. Please try again later.',
                'retry_after' => RateLimiter::availableIn($key),
            ], 429);
        }

        // Increment the rate limiter
        RateLimiter::hit($key, $this->decayMinutes($limiterName) * 60);

        $response = $next($request);

        // Add rate limit headers to response
        $response->headers->add([
            'X-RateLimit-Limit' => $this->maxAttempts($limiterName),
            'X-RateLimit-Remaining' => RateLimiter::remaining($key, $this->maxAttempts($limiterName)),
        ]);

        return $response;
    }

    /**
     * Resolve the request key for rate limiting.
     *
     * Uses user ID if authenticated, otherwise uses IP address.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    protected function resolveRequestKey(Request $request): string
    {
        return $request->user()?->id
            ? 'pii:user:' . $request->user()->id
            : 'pii:ip:' . $request->ip();
    }

    /**
     * Get the maximum number of attempts for the limiter.
     *
     * @param  string  $limiterName
     * @return int
     */
    protected function maxAttempts(string $limiterName): int
    {
        return match ($limiterName) {
            'pii-export' => 5,
            'pii' => 10,
            default => 10,
        };
    }

    /**
     * Get the decay time in minutes for the limiter.
     *
     * @param  string  $limiterName
     * @return int
     */
    protected function decayMinutes(string $limiterName): int
    {
        return match ($limiterName) {
            'pii-export' => 60, // 1 hour
            'pii' => 1,         // 1 minute
            default => 1,
        };
    }
}
```

### Controller Example

```php
<?php

/**
 * app/Http/Controllers/Api/UserController.php
 *
 * User controller with PII access methods.
 * All methods are protected by rate limiting middleware.
 */

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\UserPiiResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;

class UserController extends Controller
{
    /**
     * Retrieve user PII data.
     *
     * Rate limited to 10 requests per minute.
     *
     * @param  string  $uuid  User UUID
     * @return \Illuminate\Http\JsonResponse
     */
    public function getPii(string $uuid): JsonResponse
    {
        $user = User::where('uuid', $uuid)->firstOrFail();

        // Authorisation check
        $this->authorize('viewPii', $user);

        // Log PII access for audit trail
        Log::info('PII data accessed', [
            'accessed_by' => auth()->id(),
            'target_user' => $user->id,
        ]);

        return response()->json([
            'data' => new UserPiiResource($user),
        ]);
    }

    /**
     * Export user data for GDPR compliance.
     *
     * Rate limited to 5 requests per hour.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function exportData(Request $request): JsonResponse
    {
        $user = $request->user();

        // Queue export job (async processing)
        \App\Jobs\ExportUserDataJob::dispatch($user);

        return response()->json([
            'message' => 'Data export requested. You will receive an email when ready.',
        ], 202);
    }
}
```

---

## Django/Wagtail

### Installation

```bash
# Install django-ratelimit for rate limiting support
pip install django-ratelimit==4.1.0

# Add to requirements.txt
echo "django-ratelimit==4.1.0" >> requirements.txt
```

### Settings Configuration

```python
"""
config/settings/base.py

Rate limiting configuration for Django/Wagtail projects.
"""

# Rate limiting cache backend
# Use Redis for production, in-memory for development
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
        'KEY_PREFIX': 'ratelimit',
    }
}

# Rate limit settings
RATELIMIT_ENABLE = True  # Set to False to disable rate limiting globally
RATELIMIT_USE_CACHE = 'default'
RATELIMIT_FAIL_OPEN = False  # Fail closed on cache errors for security

# PII endpoint rate limits
PII_RATE_LIMIT = '10/m'  # 10 requests per minute
PII_EXPORT_RATE_LIMIT = '5/h'  # 5 requests per hour
```

### Rate Limit Decorator

```python
"""
apps/core/decorators/rate_limit.py

Rate limiting decorator for GraphQL resolvers and REST API views.
Uses django-ratelimit for PII endpoint protection.
"""

import logging
from functools import wraps

from django.conf import settings
from django.core.cache import cache
from django_ratelimit.decorators import ratelimit
from graphql import GraphQLError

logger = logging.getLogger(__name__)


def graphql_ratelimit(key='user', rate='10/m', block=True, method='ALL'):
    """
    Rate limiting decorator for GraphQL resolvers.

    Applies rate limiting to GraphQL resolvers to prevent abuse
    of PII endpoints and other sensitive operations.

    Args:
        key: The key to use for rate limiting ('user', 'ip', or callable)
        rate: The rate limit (e.g., '10/m' for 10 per minute, '5/h' for 5 per hour)
        block: Whether to block requests that exceed the limit (default: True)
        method: HTTP method to limit (default: 'ALL')

    Returns:
        Decorated resolver function that enforces rate limiting

    Raises:
        GraphQLError: When rate limit is exceeded

    Example:
        @strawberry.field
        @graphql_ratelimit(key='user', rate='10/m')
        def user_pii(self, info: Info, user_id: str) -> UserPiiType:
            return get_user_pii(user_id)
    """
    def decorator(resolver_func):
        @wraps(resolver_func)
        def wrapper(self, info, *args, **kwargs):
            request = info.context.get('request')

            if not request:
                logger.warning('No request object found in GraphQL context')
                raise GraphQLError('Internal server error')

            # Determine rate limit key
            if key == 'user':
                if not request.user or not request.user.is_authenticated:
                    # Fall back to IP for unauthenticated requests
                    limit_key = f'ip:{request.META.get("REMOTE_ADDR")}'
                else:
                    limit_key = f'user:{request.user.id}'
            elif key == 'ip':
                limit_key = f'ip:{request.META.get("REMOTE_ADDR")}'
            elif callable(key):
                limit_key = key(request)
            else:
                limit_key = key

            # Apply rate limiting using django-ratelimit
            @ratelimit(key=lambda r, g: limit_key, rate=rate, block=block, method=method)
            def rate_limited_view(request):
                return True

            try:
                # Check rate limit
                result = rate_limited_view(request)

                # Check if request was rate limited
                if hasattr(request, 'limited') and request.limited:
                    logger.warning(
                        'GraphQL rate limit exceeded',
                        extra={
                            'user_id': request.user.id if request.user.is_authenticated else None,
                            'ip': request.META.get('REMOTE_ADDR'),
                            'resolver': resolver_func.__name__,
                            'rate': rate,
                        }
                    )
                    raise GraphQLError(
                        'Rate limit exceeded for this endpoint. Please try again later.',
                        extensions={
                            'code': 'RATE_LIMIT_EXCEEDED',
                            'rate': rate,
                        }
                    )

            except GraphQLError:
                raise
            except Exception as e:
                logger.error(f'Rate limiting error: {str(e)}')
                if not settings.RATELIMIT_FAIL_OPEN:
                    raise GraphQLError('Rate limiting error')

            return resolver_func(self, info, *args, **kwargs)
        return wrapper
    return decorator


def api_ratelimit(key='user', rate='10/m', block=True, method='ALL'):
    """
    Rate limiting decorator for Django REST Framework API views.

    Applies rate limiting to DRF views and viewsets to prevent abuse.

    Args:
        key: The key to use for rate limiting ('user', 'ip', or callable)
        rate: The rate limit (e.g., '10/m' for 10 per minute)
        block: Whether to block requests that exceed the limit
        method: HTTP method to limit (default: 'ALL')

    Returns:
        Decorated view function that enforces rate limiting

    Example:
        @api_ratelimit(key='user', rate='10/m')
        def get(self, request, *args, **kwargs):
            return Response(data)
    """
    def get_rate_key(group, request):
        """Resolve the rate limiting key."""
        if key == 'user':
            if request.user and request.user.is_authenticated:
                return f'user:{request.user.id}'
            return f'ip:{request.META.get("REMOTE_ADDR")}'
        elif key == 'ip':
            return f'ip:{request.META.get("REMOTE_ADDR")}'
        elif callable(key):
            return key(request)
        return key

    return ratelimit(key=get_rate_key, rate=rate, block=block, method=method)
```

### GraphQL Schema Example (Strawberry)

```python
"""
apps/users/schema.py

GraphQL schema with rate-limited PII queries.
"""

import logging
from typing import Optional

import strawberry
from strawberry.types import Info

from apps.core.decorators.rate_limit import graphql_ratelimit
from apps.core.permissions import RequirePiiAccess
from apps.users.models import User
from apps.users.services import UserPiiService

logger = logging.getLogger(__name__)


@strawberry.type
class UserPiiType:
    """User PII data type for GraphQL responses."""

    id: strawberry.ID
    email: str
    first_name: str
    last_name: str
    phone_number: Optional[str]
    date_of_birth: Optional[str]
    address: Optional[str]


@strawberry.type
class Query:
    """GraphQL queries for user PII data."""

    @strawberry.field(permission_classes=[RequirePiiAccess])
    @graphql_ratelimit(key='user', rate='10/m')
    def user_pii(self, info: Info, user_id: str) -> UserPiiType:
        """
        Retrieve user PII data.

        Rate limited to 10 requests per minute per user.
        Requires PII access permission.

        Args:
            info: GraphQL execution info containing request context
            user_id: UUID of the user to retrieve PII for

        Returns:
            UserPiiType containing PII data

        Raises:
            GraphQLError: If rate limit exceeded or unauthorised
        """
        request = info.context.get('request')

        # Log PII access for audit trail
        logger.info(
            'PII data accessed via GraphQL',
            extra={
                'accessed_by': request.user.id,
                'target_user': user_id,
                'ip': request.META.get('REMOTE_ADDR'),
            }
        )

        return UserPiiService.get_user_pii(user_id)


@strawberry.type
class Mutation:
    """GraphQL mutations for user PII operations."""

    @strawberry.mutation(permission_classes=[RequirePiiAccess])
    @graphql_ratelimit(key='user', rate='5/h')
    def export_user_data(self, info: Info, user_id: str) -> str:
        """
        Request user data export (GDPR compliance).

        Rate limited to 5 requests per hour per user.

        Args:
            info: GraphQL execution info
            user_id: UUID of the user to export data for

        Returns:
            Success message indicating export has been queued
        """
        request = info.context.get('request')

        logger.info(
            'User data export requested',
            extra={
                'requested_by': request.user.id,
                'target_user': user_id,
            }
        )

        # Queue async export task
        from apps.users.tasks import export_user_data_task
        export_user_data_task.delay(user_id, request.user.id)

        return 'Data export requested. You will receive an email when ready.'
```

### REST API Example (Django REST Framework)

```python
"""
apps/users/api/views.py

REST API views with rate limiting for PII endpoints.
"""

import logging

from django.utils.decorators import method_decorator
from rest_framework import status
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

from apps.core.decorators.rate_limit import api_ratelimit
from apps.core.permissions import HasPiiAccess
from apps.users.models import User
from apps.users.serializers import UserPiiSerializer

logger = logging.getLogger(__name__)


class UserPiiView(APIView):
    """
    API view for retrieving user PII data.

    Rate limited to 10 requests per minute per user.
    """

    permission_classes = [IsAuthenticated, HasPiiAccess]

    @method_decorator(api_ratelimit(key='user', rate='10/m', method='GET'))
    def get(self, request, user_id):
        """
        Retrieve user PII data.

        Args:
            request: Django request object
            user_id: UUID of the user

        Returns:
            Response with serialised PII data
        """
        try:
            user = User.objects.get(id=user_id)
        except User.DoesNotExist:
            return Response(
                {'error': 'User not found'},
                status=status.HTTP_404_NOT_FOUND
            )

        # Log PII access
        logger.info(
            'PII data accessed via REST API',
            extra={
                'accessed_by': request.user.id,
                'target_user': user_id,
                'ip': request.META.get('REMOTE_ADDR'),
            }
        )

        serializer = UserPiiSerializer(user)
        return Response(serializer.data)


class UserDataExportView(APIView):
    """
    API view for requesting user data export (GDPR compliance).

    Rate limited to 5 requests per hour per user.
    """

    permission_classes = [IsAuthenticated, HasPiiAccess]

    @method_decorator(api_ratelimit(key='user', rate='5/h', method='POST'))
    def post(self, request):
        """
        Request user data export.

        Returns:
            Response indicating export has been queued
        """
        user_id = request.user.id

        logger.info(
            'User data export requested via REST API',
            extra={'requested_by': user_id}
        )

        # Queue async export task
        from apps.users.tasks import export_user_data_task
        export_user_data_task.delay(user_id, user_id)

        return Response(
            {'message': 'Data export requested. You will receive an email when ready.'},
            status=status.HTTP_202_ACCEPTED
        )
```

### Middleware (Optional)

```python
"""
apps/core/middleware/rate_limit.py

Global rate limiting middleware for additional protection.
"""

import logging

from django.http import JsonResponse
from django_ratelimit.decorators import ratelimit

logger = logging.getLogger(__name__)


class GlobalRateLimitMiddleware:
    """
    Global rate limiting middleware.

    Applies a generous global rate limit to all requests
    as a baseline protection against DoS attacks.
    """

    def __init__(self, get_response):
        """
        Initialise the middleware.

        Args:
            get_response: Next middleware or view in the chain
        """
        self.get_response = get_response

    def __call__(self, request):
        """
        Process the request with global rate limiting.

        Args:
            request: Django request object

        Returns:
            Response from next middleware/view or 429 if rate limited
        """
        # Apply global rate limit (100 requests per minute per IP)
        @ratelimit(key='ip', rate='100/m', block=True)
        def check_rate_limit(request):
            return True

        try:
            check_rate_limit(request)

            # Check if limited
            if hasattr(request, 'limited') and request.limited:
                logger.warning(
                    'Global rate limit exceeded',
                    extra={
                        'ip': request.META.get('REMOTE_ADDR'),
                        'path': request.path,
                    }
                )
                return JsonResponse(
                    {'error': 'Too many requests. Please try again later.'},
                    status=429
                )

        except Exception as e:
            logger.error(f'Global rate limiting error: {str(e)}')

        return self.get_response(request)
```

---

## Node.js/TypeScript (NestJS GraphQL)

### PII Throttle Guard

```typescript
/**
 * pii-throttle.guard.ts
 *
 * Rate limiting guard for GraphQL PII endpoints.
 * Uses NestJS Throttler for rate limiting.
 */

import { Injectable, ExecutionContext } from '@nestjs/common';
import { ThrottlerGuard } from '@nestjs/throttler';
import { GqlExecutionContext } from '@nestjs/graphql';

@Injectable()
export class PiiThrottleGuard extends ThrottlerGuard {
  /**
   * Gets the request object from GraphQL context.
   *
   * @param context - The execution context
   * @returns The request and response objects
   */
  getRequestResponse(context: ExecutionContext) {
    const gqlCtx = GqlExecutionContext.create(context);
    const ctx = gqlCtx.getContext();
    return { req: ctx.req, res: ctx.res };
  }

  /**
   * Customises the throttle key to use user ID for authenticated requests.
   *
   * @param context - The execution context
   * @param suffix - The default suffix
   * @param name - The throttler name
   * @returns The throttle key string
   */
  protected async getTracker(
    context: ExecutionContext,
    suffix: string,
    name: string,
  ): Promise<string> {
    const gqlCtx = GqlExecutionContext.create(context);
    const { user, req } = gqlCtx.getContext();

    // Use user ID if authenticated, otherwise use IP
    const key = user?.id ?? req.ip;
    return `pii:${key}:${suffix}`;
  }
}
```

### Module Configuration

```typescript
// app.module.ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        name: 'pii',
        ttl: 60000, // 1 minute
        limit: 10,  // 10 requests per minute
      },
    ]),
  ],
})
export class AppModule {}
```

### Usage

```typescript
import { UseGuards } from '@nestjs/common';
import { Resolver, Query } from '@nestjs/graphql';
import { PiiAccessGuard } from './guards/pii-access.guard';
import { PiiThrottleGuard } from './guards/pii-throttle.guard';

@Resolver()
export class UserResolver {
  @Query(() => UserPiiType)
  @UseGuards(PiiAccessGuard, PiiThrottleGuard)
  async userPii(@Args('userId') userId: string): Promise<UserPiiType> {
    return this.userService.getPii(userId);
  }
}
```
