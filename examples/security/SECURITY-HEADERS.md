# Security Headers & Hardening

## Overview

Security hardening patterns including HTTP security headers, rate limiting, IP allowlisting, and audit logging. These patterns protect against common web vulnerabilities.

## Metadata

| Property            | Value                                             |
| ------------------- | ------------------------------------------------- |
| **Example Version** | 2.0.0                                             |
| **Last Updated**    | 2025-12                                           |
| **TALL Stack**      | Laravel 12.x / PHP 8.4 / MariaDB 12.x             |
| **Django/Wagtail**  | Django 6.x / Python 3.14 / PostgreSQL 18.x        |
| **React/Next.js**   | Next.js 16.x / React 19.x / TypeScript 5.9        |
| **React Native**    | React Native 0.83.x                               |
| **Stacks**          | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [TALL Stack (Laravel)](#tall-stack-laravel)
  - [Security Headers Middleware - Laravel](#security-headers-middleware---laravel)
  - [app/Http/Middleware/SecurityHeaders.php](#apphttpmiddlewaresecurityheadersphp)
- [Rate Limiting - Laravel](#rate-limiting---laravel)
  - [app/Providers/RouteServiceProvider.php](#appprovidersrouteserviceproviderphp)
  - [routes/web.php (Rate Limited Routes)](#routeswebphp-rate-limited-routes)
- [IP Allowlisting - Laravel](#ip-allowlisting---laravel)
  - [app/Http/Middleware/AdminIpAllowlist.php](#apphttpmiddlewareadminipallowlistphp)
- [Audit Logging - Laravel](#audit-logging---laravel)
  - [app/Services/AuditService.php](#appservicesauditservicephp)
  - [database/migrations/xxxx\_create\_audit\_logs\_table.php](#databasemigrationsxxxx_create_audit_logs_tablephp)
- [Django/Wagtail Stack](#djangowagtail-stack)
  - [Security Headers Middleware - Django](#security-headers-middleware---django)
  - [middleware/security\_headers.py](#middlewaresecurity_headerspy)
- [Rate Limiting - Django](#rate-limiting---django)
    - [middleware/rate\_limiting.py](#middlewarerate_limitingpy)
    - [settings/base.py (Rate Limiting Configuration)](#settingsbasepy-rate-limiting-configuration)
- [IP Allowlisting - Django](#ip-allowlisting---django)
    - [middleware/admin\_ip\_allowlist.py](#middlewareadmin_ip_allowlistpy)
    - [.env.example](#envexample)
- [Audit Logging - Django](#audit-logging---django)
    - [services/audit\_service.py](#servicesaudit_servicepy)
    - [apps/audit/models.py](#appsauditmodelspy)
    - [apps/audit/migrations/0001\_initial.py](#appsauditmigrations0001_initialpy)
- [React/Next.js Stack](#reactnextjs-stack)
  - [Security Headers Configuration - Next.js](#security-headers-configuration---nextjs)
    - [next.config.js](#nextconfigjs)
    - [Alternative: Middleware Approach (middleware.ts)](#alternative-middleware-approach-middlewarets)
- [Rate Limiting - Next.js](#rate-limiting---nextjs)
    - [lib/rate-limit.ts](#librate-limitts)
    - [app/api/auth/login/route.ts (Example Usage)](#appapiauthloginroutets-example-usage)
    - [package.json (Dependencies)](#packagejson-dependencies)
- [Audit Logging - Next.js](#audit-logging---nextjs)
    - [lib/audit-service.ts](#libaudit-servicets)
    - [prisma/schema.prisma (Audit Log Model)](#prismaschemaprisma-audit-log-model)
    - [app/api/example/route.ts (Usage Example)](#appapiexampleroutets-usage-example)
- [React Native Stack](#react-native-stack)
  - [Security Headers Note](#security-headers-note)


## TALL Stack (Laravel)

### Security Headers Middleware - Laravel

### app/Http/Middleware/SecurityHeaders.php

```php
<?php

/**
 * SecurityHeaders.php
 *
 * Middleware to add security headers to all responses.
 * Protects against XSS, clickjacking, and other attacks.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class SecurityHeaders
{
    /**
     * Adds security headers to the response.
     *
     * @param Request $request The HTTP request
     * @param Closure $next The next middleware
     * @return mixed
     */
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        return $response
            ->header('X-Content-Type-Options', 'nosniff')
            ->header('X-Frame-Options', 'SAMEORIGIN')
            ->header('X-XSS-Protection', '1; mode=block')
            ->header('Referrer-Policy', 'strict-origin-when-cross-origin')
            ->header('Permissions-Policy', 'geolocation=(), microphone=(), camera=()')
            ->header('Content-Security-Policy', $this->getCSP());
    }

    /**
     * Builds the Content-Security-Policy header value.
     * Customise based on application requirements.
     *
     * @return string The CSP header value
     */
    protected function getCSP(): string
    {
        return implode('; ', [
            "default-src 'self'",
            "script-src 'self' 'unsafe-inline' https://cdn.example.com",
            "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
            "img-src 'self' data: https:",
            "font-src 'self' https://fonts.gstatic.com",
            "connect-src 'self' https://api.example.com",
            "frame-ancestors 'self'",
            "form-action 'self'",
            "base-uri 'self'",
        ]);
    }
}
```

---

## Rate Limiting - Laravel

### app/Providers/RouteServiceProvider.php

```php
<?php

/**
 * Rate limiting configuration.
 *
 * Defines rate limits for different route groups.
 * Protects against brute force and DoS attacks.
 */

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class RouteServiceProvider extends ServiceProvider
{
    /**
     * Configures rate limiting for the application.
     */
    protected function configureRateLimiting(): void
    {
        // General API rate limit
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
        });

        // Strict limit for auth endpoints
        RateLimiter::for('auth', function (Request $request) {
            return Limit::perMinute(5)->by($request->ip());
        });

        // Very strict for password reset
        RateLimiter::for('password-reset', function (Request $request) {
            return Limit::perHour(3)->by($request->ip());
        });

        // Admin actions
        RateLimiter::for('admin', function (Request $request) {
            return Limit::perMinute(30)->by($request->user()?->id ?: $request->ip());
        });

        // Sensitive operations (exports, bulk actions)
        RateLimiter::for('sensitive', function (Request $request) {
            return Limit::perHour(10)->by($request->user()?->id ?: $request->ip());
        });
    }
}
```

### routes/web.php (Rate Limited Routes)

```php
<?php

use Illuminate\Support\Facades\Route;

// Apply rate limiting to route groups
Route::middleware(['throttle:auth'])->group(function () {
    Route::post('/login', [AuthController::class, 'login']);
    Route::post('/register', [AuthController::class, 'register']);
});

Route::middleware(['throttle:password-reset'])->group(function () {
    Route::post('/password/reset', [PasswordController::class, 'reset']);
});

Route::middleware(['auth', 'throttle:admin'])->prefix('admin')->group(function () {
    // Admin routes with stricter rate limiting
});
```

---

## IP Allowlisting - Laravel

### app/Http/Middleware/AdminIpAllowlist.php

```php
<?php

/**
 * AdminIpAllowlist.php
 *
 * Middleware to restrict admin access to specific IP addresses.
 * Returns 404 instead of 403 to hide admin existence.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class AdminIpAllowlist
{
    protected array $allowedIps;

    public function __construct()
    {
        $this->allowedIps = explode(',', env('ADMIN_ALLOWED_IPS', ''));
    }

    /**
     * Checks if the request IP is in the allowlist.
     * Returns 404 to hide admin existence from blocked IPs.
     *
     * @param Request $request The HTTP request
     * @param Closure $next The next middleware
     * @return mixed
     */
    public function handle(Request $request, Closure $next)
    {
        // Skip if allowlist is empty (disabled)
        if (empty(array_filter($this->allowedIps))) {
            return $next($request);
        }

        if (!in_array($request->ip(), $this->allowedIps)) {
            logger()->warning('Admin access blocked by IP allowlist', [
                'ip' => $request->ip(),
                'path' => $request->path(),
                'user_id' => $request->user()?->id,
            ]);

            abort(404); // Return 404 to hide admin existence
        }

        return $next($request);
    }
}
```

---

## Audit Logging - Laravel

### app/Services/AuditService.php

```php
<?php

/**
 * AuditService.php
 *
 * Service for logging security-relevant actions.
 * Provides audit trail for compliance and security monitoring.
 */

namespace App\Services;

use App\Models\AuditLog;

class AuditService
{
    /**
     * Logs an action with metadata.
     *
     * @param string $action The action type
     * @param string $resource The resource type
     * @param int|null $resourceId The resource ID
     * @param array $metadata Additional metadata
     */
    public function log(
        string $action,
        string $resource,
        ?int $resourceId = null,
        array $metadata = []
    ): void {
        $request = request();
        $user = $request->user();

        AuditLog::create([
            'user_id' => $user?->id,
            'action' => $action,
            'resource_type' => $resource,
            'resource_id' => $resourceId,
            'ip_address' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'metadata' => $metadata,
            'created_at' => now(),
        ]);
    }

    /**
     * Logs a security event.
     *
     * @param string $event The security event type
     * @param array $details Event details
     */
    public function logSecurityEvent(string $event, array $details = []): void
    {
        $this->log('security.' . $event, 'security', null, $details);
    }

    /**
     * Logs an access denied event.
     *
     * @param string $resource The resource that was denied
     * @param int|null $resourceId The resource ID
     */
    public function logAccessDenied(string $resource, ?int $resourceId = null): void
    {
        $this->logSecurityEvent('access_denied', [
            'resource' => $resource,
            'resource_id' => $resourceId,
            'path' => request()->path(),
        ]);
    }
}
```

### database/migrations/xxxx_create_audit_logs_table.php

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('audit_logs', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->nullable()->constrained()->nullOnDelete();
            $table->string('action', 100)->index();
            $table->string('resource_type', 100)->index();
            $table->unsignedBigInteger('resource_id')->nullable()->index();
            $table->string('ip_address', 45);
            $table->text('user_agent')->nullable();
            $table->json('metadata')->nullable();
            $table->timestamp('created_at')->useCurrent();

            // Composite index for common queries
            $table->index(['resource_type', 'resource_id', 'created_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('audit_logs');
    }
};
```

---

## Django/Wagtail Stack

### Security Headers Middleware - Django

### middleware/security_headers.py

```python
"""
security_headers.py

Middleware to add security headers to all responses.
Protects against XSS, clickjacking, and other attacks.
"""

from django.http import HttpResponse


class SecurityHeadersMiddleware:
    """
    Middleware that adds security headers to all responses.
    Configure CSP based on application requirements.
    """

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        return self.add_security_headers(response)

    def add_security_headers(self, response: HttpResponse) -> HttpResponse:
        """
        Adds security headers to the response.

        Args:
            response: The HTTP response

        Returns:
            The response with security headers added
        """
        response['X-Content-Type-Options'] = 'nosniff'
        response['X-Frame-Options'] = 'SAMEORIGIN'
        response['X-XSS-Protection'] = '1; mode=block'
        response['Referrer-Policy'] = 'strict-origin-when-cross-origin'
        response['Permissions-Policy'] = 'geolocation=(), microphone=(), camera=()'
        response['Content-Security-Policy'] = self.get_csp()

        return response

    def get_csp(self) -> str:
        """
        Builds the Content-Security-Policy header value.

        Returns:
            The CSP header value
        """
        directives = [
            "default-src 'self'",
            "script-src 'self' 'unsafe-inline' https://cdn.example.com",
            "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
            "img-src 'self' data: https:",
            "font-src 'self' https://fonts.gstatic.com",
            "connect-src 'self' https://api.example.com",
            "frame-ancestors 'self'",
            "form-action 'self'",
            "base-uri 'self'",
        ]

        return '; '.join(directives)
```

---

## Rate Limiting - Django

#### middleware/rate_limiting.py

```python
"""
rate_limiting.py

Middleware for rate limiting requests to prevent abuse.
Protects against brute force and DoS attacks.
"""

from django.core.cache import cache
from django.http import HttpResponse
from django.conf import settings
import hashlib


class RateLimitMiddleware:
    """
    Middleware that implements rate limiting based on IP address or user ID.
    Configure rate limits in Django settings.
    """

    def __init__(self, get_response):
        self.get_response = get_response

        # Default rate limits (can be overridden in settings)
        self.rate_limits = getattr(settings, 'RATE_LIMITS', {
            'default': {'requests': 60, 'period': 60},  # 60 requests per minute
            'auth': {'requests': 5, 'period': 60},  # 5 requests per minute
            'password_reset': {'requests': 3, 'period': 3600},  # 3 per hour
            'admin': {'requests': 30, 'period': 60},  # 30 requests per minute
            'sensitive': {'requests': 10, 'period': 3600},  # 10 per hour
        })

    def __call__(self, request):
        """
        Checks rate limits before processing the request.

        Args:
            request: The HTTP request

        Returns:
            HTTP response (429 if rate limited, otherwise normal response)
        """
        # Determine rate limit category
        limit_key = self._get_limit_key(request)
        rate_limit = self.rate_limits.get(limit_key, self.rate_limits['default'])

        # Check rate limit
        if not self._check_rate_limit(request, limit_key, rate_limit):
            return HttpResponse(
                'Rate limit exceeded. Please try again later.',
                status=429
            )

        return self.get_response(request)

    def _get_limit_key(self, request) -> str:
        """
        Determines which rate limit category to apply.

        Args:
            request: The HTTP request

        Returns:
            Rate limit category key
        """
        path = request.path.lower()

        if '/admin/' in path:
            return 'admin'
        elif any(endpoint in path for endpoint in ['/login/', '/register/']):
            return 'auth'
        elif '/password/reset/' in path:
            return 'password_reset'
        elif any(endpoint in path for endpoint in ['/export/', '/bulk/']):
            return 'sensitive'

        return 'default'

    def _check_rate_limit(self, request, limit_key: str, rate_limit: dict) -> bool:
        """
        Checks if the request is within rate limits.

        Args:
            request: The HTTP request
            limit_key: The rate limit category
            rate_limit: Dictionary with 'requests' and 'period' keys

        Returns:
            True if within limits, False if exceeded
        """
        # Create unique cache key based on IP or user
        identifier = request.user.id if request.user.is_authenticated else request.META.get('REMOTE_ADDR')
        cache_key = f'rate_limit:{limit_key}:{hashlib.md5(str(identifier).encode()).hexdigest()}'

        # Get current request count
        request_count = cache.get(cache_key, 0)

        if request_count >= rate_limit['requests']:
            return False

        # Increment request count
        cache.set(cache_key, request_count + 1, rate_limit['period'])
        return True
```

#### settings/base.py (Rate Limiting Configuration)

```python
"""
Rate limiting configuration for Django.
Define rate limits for different endpoint categories.
"""

RATE_LIMITS = {
    'default': {
        'requests': 60,
        'period': 60,  # seconds
    },
    'auth': {
        'requests': 5,
        'period': 60,
    },
    'password_reset': {
        'requests': 3,
        'period': 3600,  # 1 hour
    },
    'admin': {
        'requests': 30,
        'period': 60,
    },
    'sensitive': {
        'requests': 10,
        'period': 3600,  # 1 hour
    },
}

# Cache backend for rate limiting
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}

# Add rate limiting middleware
MIDDLEWARE = [
    # ... other middleware
    'middleware.rate_limiting.RateLimitMiddleware',
    # ... other middleware
]
```

---

## IP Allowlisting - Django

#### middleware/admin_ip_allowlist.py

```python
"""
admin_ip_allowlist.py

Middleware to restrict admin access to specific IP addresses.
Returns 404 instead of 403 to hide admin existence.
"""

from django.http import Http404
from django.conf import settings
import logging

logger = logging.getLogger(__name__)


class AdminIpAllowlistMiddleware:
    """
    Middleware that restricts admin access to allowed IP addresses.
    Returns 404 to hide admin existence from unauthorised IPs.
    """

    def __init__(self, get_response):
        self.get_response = get_response

        # Get allowed IPs from settings
        allowed_ips = getattr(settings, 'ADMIN_ALLOWED_IPS', '')
        self.allowed_ips = [ip.strip() for ip in allowed_ips.split(',') if ip.strip()]

    def __call__(self, request):
        """
        Checks if admin requests come from allowed IPs.

        Args:
            request: The HTTP request

        Returns:
            HTTP response (raises Http404 if blocked)
        """
        # Check if this is an admin request
        if request.path.startswith('/admin/'):
            # Skip if allowlist is empty (disabled)
            if not self.allowed_ips:
                return self.get_response(request)

            client_ip = self._get_client_ip(request)

            if client_ip not in self.allowed_ips:
                logger.warning(
                    'Admin access blocked by IP allowlist',
                    extra={
                        'ip': client_ip,
                        'path': request.path,
                        'user_id': request.user.id if request.user.is_authenticated else None,
                    }
                )

                # Return 404 to hide admin existence
                raise Http404()

        return self.get_response(request)

    def _get_client_ip(self, request) -> str:
        """
        Extracts the client IP address from the request.
        Handles X-Forwarded-For header for proxied requests.

        Args:
            request: The HTTP request

        Returns:
            Client IP address
        """
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0].strip()
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

#### .env.example

```bash
# Admin IP Allowlist (comma-separated)
# Leave empty to disable IP allowlisting
ADMIN_ALLOWED_IPS=192.168.1.100,10.0.0.50
```

---

## Audit Logging - Django

#### services/audit_service.py

```python
"""
audit_service.py

Service for logging security-relevant actions.
Provides audit trail for compliance and security monitoring.
"""

from django.conf import settings
from typing import Optional, Dict, Any
import logging

logger = logging.getLogger(__name__)


class AuditService:
    """
    Service for logging security-relevant actions with full context.
    Stores audit logs in database and optionally external logging service.
    """

    @staticmethod
    def log(
        action: str,
        resource: str,
        resource_id: Optional[int] = None,
        metadata: Optional[Dict[str, Any]] = None,
        request=None
    ) -> None:
        """
        Logs an action with metadata.

        Args:
            action: The action type (e.g., 'create', 'update', 'delete')
            resource: The resource type (e.g., 'user', 'order', 'product')
            resource_id: The resource ID if applicable
            metadata: Additional metadata dictionary
            request: The HTTP request object for context
        """
        from apps.audit.models import AuditLog

        user = None
        ip_address = None
        user_agent = None

        if request:
            user = request.user if request.user.is_authenticated else None
            ip_address = AuditService._get_client_ip(request)
            user_agent = request.META.get('HTTP_USER_AGENT', '')

        AuditLog.objects.create(
            user=user,
            action=action,
            resource_type=resource,
            resource_id=resource_id,
            ip_address=ip_address,
            user_agent=user_agent,
            metadata=metadata or {}
        )

        # Also log to standard logging for external services
        logger.info(
            f'Audit: {action} {resource}',
            extra={
                'user_id': user.id if user else None,
                'resource_type': resource,
                'resource_id': resource_id,
                'ip_address': ip_address,
                'metadata': metadata,
            }
        )

    @staticmethod
    def log_security_event(event: str, details: Optional[Dict[str, Any]] = None, request=None) -> None:
        """
        Logs a security event.

        Args:
            event: The security event type
            details: Event details dictionary
            request: The HTTP request object
        """
        AuditService.log(
            action=f'security.{event}',
            resource='security',
            metadata=details,
            request=request
        )

    @staticmethod
    def log_access_denied(resource: str, resource_id: Optional[int] = None, request=None) -> None:
        """
        Logs an access denied event.

        Args:
            resource: The resource that was denied
            resource_id: The resource ID if applicable
            request: The HTTP request object
        """
        metadata = {}
        if request:
            metadata['path'] = request.path
            metadata['method'] = request.method

        AuditService.log_security_event(
            event='access_denied',
            details={
                'resource': resource,
                'resource_id': resource_id,
                **metadata
            },
            request=request
        )

    @staticmethod
    def _get_client_ip(request) -> str:
        """
        Extracts the client IP address from the request.

        Args:
            request: The HTTP request

        Returns:
            Client IP address
        """
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0].strip()
        else:
            ip = request.META.get('REMOTE_ADDR', '')
        return ip
```

#### apps/audit/models.py

```python
"""
Audit log model for tracking security-relevant actions.
"""

from django.db import models
from django.contrib.auth import get_user_model
from django.utils import timezone

User = get_user_model()


class AuditLog(models.Model):
    """
    Model for storing audit logs of security-relevant actions.
    Provides full audit trail for compliance and security monitoring.
    """

    user = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='audit_logs',
        help_text='User who performed the action'
    )
    action = models.CharField(
        max_length=100,
        db_index=True,
        help_text='Action type (e.g., create, update, delete)'
    )
    resource_type = models.CharField(
        max_length=100,
        db_index=True,
        help_text='Type of resource affected'
    )
    resource_id = models.BigIntegerField(
        null=True,
        blank=True,
        db_index=True,
        help_text='ID of the affected resource'
    )
    ip_address = models.GenericIPAddressField(
        null=True,
        blank=True,
        help_text='IP address of the request'
    )
    user_agent = models.TextField(
        blank=True,
        help_text='User agent string'
    )
    metadata = models.JSONField(
        default=dict,
        blank=True,
        help_text='Additional metadata about the action'
    )
    created_at = models.DateTimeField(
        default=timezone.now,
        db_index=True,
        help_text='When the action occurred'
    )

    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['resource_type', 'resource_id', 'created_at']),
            models.Index(fields=['user', 'created_at']),
            models.Index(fields=['action', 'created_at']),
        ]
        verbose_name = 'Audit Log'
        verbose_name_plural = 'Audit Logs'

    def __str__(self):
        return f'{self.action} on {self.resource_type} at {self.created_at}'
```

#### apps/audit/migrations/0001_initial.py

```python
"""
Initial migration for audit logs table.
"""

from django.conf import settings
from django.db import migrations, models
import django.db.models.deletion
import django.utils.timezone


class Migration(migrations.Migration):

    initial = True

    dependencies = [
        migrations.swappable_dependency(settings.AUTH_USER_MODEL),
    ]

    operations = [
        migrations.CreateModel(
            name='AuditLog',
            fields=[
                ('id', models.BigAutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('action', models.CharField(db_index=True, help_text='Action type (e.g., create, update, delete)', max_length=100)),
                ('resource_type', models.CharField(db_index=True, help_text='Type of resource affected', max_length=100)),
                ('resource_id', models.BigIntegerField(blank=True, db_index=True, help_text='ID of the affected resource', null=True)),
                ('ip_address', models.GenericIPAddressField(blank=True, help_text='IP address of the request', null=True)),
                ('user_agent', models.TextField(blank=True, help_text='User agent string')),
                ('metadata', models.JSONField(blank=True, default=dict, help_text='Additional metadata about the action')),
                ('created_at', models.DateTimeField(db_index=True, default=django.utils.timezone.now, help_text='When the action occurred')),
                ('user', models.ForeignKey(blank=True, help_text='User who performed the action', null=True, on_delete=django.db.models.deletion.SET_NULL, related_name='audit_logs', to=settings.AUTH_USER_MODEL)),
            ],
            options={
                'verbose_name': 'Audit Log',
                'verbose_name_plural': 'Audit Logs',
                'ordering': ['-created_at'],
            },
        ),
        migrations.AddIndex(
            model_name='auditlog',
            index=models.Index(fields=['resource_type', 'resource_id', 'created_at'], name='audit_auditl_resourc_idx'),
        ),
        migrations.AddIndex(
            model_name='auditlog',
            index=models.Index(fields=['user', 'created_at'], name='audit_auditl_user_id_idx'),
        ),
        migrations.AddIndex(
            model_name='auditlog',
            index=models.Index(fields=['action', 'created_at'], name='audit_auditl_action_idx'),
        ),
    ]
```

---

## React/Next.js Stack

### Security Headers Configuration - Next.js

#### next.config.js

```javascript
/**
 * next.config.js
 *
 * Next.js configuration with security headers.
 * Protects against XSS, clickjacking, and other attacks.
 */

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Security headers configuration
  async headers() {
    return [
      {
        // Apply security headers to all routes
        source: '/:path*',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN',
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
          {
            key: 'Permissions-Policy',
            value: 'geolocation=(), microphone=(), camera=()',
          },
          {
            key: 'Content-Security-Policy',
            value: getContentSecurityPolicy(),
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=31536000; includeSubDomains',
          },
        ],
      },
    ];
  },

  // Other Next.js configuration
  reactStrictMode: true,
  swcMinify: true,
};

/**
 * Builds the Content-Security-Policy header value.
 * Customise based on application requirements.
 *
 * @returns {string} The CSP header value
 */
function getContentSecurityPolicy() {
  const directives = [
    "default-src 'self'",
    "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://cdn.example.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "img-src 'self' data: https: blob:",
    "font-src 'self' https://fonts.gstatic.com",
    "connect-src 'self' https://api.example.com",
    "frame-ancestors 'self'",
    "form-action 'self'",
    "base-uri 'self'",
    "object-src 'none'",
  ];

  return directives.join('; ');
}

module.exports = nextConfig;
```

#### Alternative: Middleware Approach (middleware.ts)

```typescript
/**
 * middleware.ts
 *
 * Next.js middleware for adding security headers.
 * Alternative to next.config.js headers configuration.
 */

import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

/**
 * Middleware function that adds security headers to all responses.
 *
 * @param request - The incoming request
 * @returns Response with security headers added
 */
export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Add security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'SAMEORIGIN');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
  response.headers.set('Content-Security-Policy', getContentSecurityPolicy());
  response.headers.set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');

  return response;
}

/**
 * Builds the Content-Security-Policy header value.
 *
 * @returns The CSP header value
 */
function getContentSecurityPolicy(): string {
  const directives = [
    "default-src 'self'",
    "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://cdn.example.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "img-src 'self' data: https: blob:",
    "font-src 'self' https://fonts.gstatic.com",
    "connect-src 'self' https://api.example.com",
    "frame-ancestors 'self'",
    "form-action 'self'",
    "base-uri 'self'",
    "object-src 'none'",
  ];

  return directives.join('; ');
}

// Configure which routes the middleware applies to
export const config = {
  matcher: [
    /*
     * Match all request paths except for:
     * - _next/static (static files)
     * - _next/image (image optimisation files)
     * - favicon.ico (favicon file)
     * - public folder
     */
    '/((?!_next/static|_next/image|favicon.ico|public/).*)',
  ],
};
```

---

## Rate Limiting - Next.js

#### lib/rate-limit.ts

```typescript
/**
 * rate-limit.ts
 *
 * Rate limiting utility using upstash/ratelimit.
 * Protects against brute force and DoS attacks.
 */

import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';
import { NextRequest } from 'next/server';

/**
 * Creates a rate limiter instance with specified limits.
 *
 * @param requests - Maximum number of requests
 * @param window - Time window (e.g., '1 m', '1 h')
 * @returns Ratelimit instance
 */
function createRateLimiter(requests: number, window: string) {
  return new Ratelimit({
    redis: Redis.fromEnv(),
    limiter: Ratelimit.slidingWindow(requests, window),
    analytics: true,
    prefix: '@ratelimit',
  });
}

// Rate limiters for different endpoint categories
export const rateLimiters = {
  // General API rate limit: 60 requests per minute
  api: createRateLimiter(60, '1 m'),

  // Strict limit for auth endpoints: 5 requests per minute
  auth: createRateLimiter(5, '1 m'),

  // Very strict for password reset: 3 requests per hour
  passwordReset: createRateLimiter(3, '1 h'),

  // Admin actions: 30 requests per minute
  admin: createRateLimiter(30, '1 m'),

  // Sensitive operations: 10 requests per hour
  sensitive: createRateLimiter(10, '1 h'),
};

/**
 * Checks rate limit for a given identifier and limiter type.
 *
 * @param identifier - Unique identifier (IP address or user ID)
 * @param limiterType - Type of rate limiter to use
 * @returns Object with success status and limit information
 */
export async function checkRateLimit(
  identifier: string,
  limiterType: keyof typeof rateLimiters = 'api'
) {
  const limiter = rateLimiters[limiterType];
  const { success, limit, reset, remaining } = await limiter.limit(identifier);

  return {
    success,
    limit,
    reset,
    remaining,
  };
}

/**
 * Gets the client IP address from the request.
 *
 * @param request - Next.js request object
 * @returns Client IP address
 */
export function getClientIp(request: NextRequest): string {
  const forwarded = request.headers.get('x-forwarded-for');
  const ip = forwarded ? forwarded.split(',')[0].trim() : request.ip || '127.0.0.1';
  return ip;
}
```

#### app/api/auth/login/route.ts (Example Usage)

```typescript
/**
 * Login API route with rate limiting.
 */

import { NextRequest, NextResponse } from 'next/server';
import { checkRateLimit, getClientIp } from '@/lib/rate-limit';

/**
 * Handles login requests with rate limiting protection.
 *
 * @param request - The incoming request
 * @returns Response with login result or rate limit error
 */
export async function POST(request: NextRequest) {
  const ip = getClientIp(request);

  // Check rate limit for auth endpoints
  const { success, reset } = await checkRateLimit(ip, 'auth');

  if (!success) {
    return NextResponse.json(
      {
        error: 'Too many requests. Please try again later.',
        resetAt: new Date(reset).toISOString(),
      },
      { status: 429 }
    );
  }

  // Process login...
  const body = await request.json();

  // Your login logic here

  return NextResponse.json({ success: true });
}
```

#### package.json (Dependencies)

```json
{
  "dependencies": {
    "@upstash/ratelimit": "^1.0.0",
    "@upstash/redis": "^1.28.0"
  }
}
```

---

## Audit Logging - Next.js

#### lib/audit-service.ts

```typescript
/**
 * audit-service.ts
 *
 * Service for logging security-relevant actions.
 * Provides audit trail for compliance and security monitoring.
 */

import { NextRequest } from 'next/server';

/**
 * Audit log entry interface.
 */
interface AuditLogEntry {
  userId?: string;
  action: string;
  resourceType: string;
  resourceId?: string | number;
  ipAddress?: string;
  userAgent?: string;
  metadata?: Record<string, any>;
  createdAt: Date;
}

/**
 * Service for logging security-relevant actions with full context.
 */
export class AuditService {
  /**
   * Logs an action with metadata.
   *
   * @param action - The action type
   * @param resourceType - The resource type
   * @param options - Additional options (resourceId, metadata, request)
   */
  static async log(
    action: string,
    resourceType: string,
    options: {
      resourceId?: string | number;
      metadata?: Record<string, any>;
      request?: NextRequest;
      userId?: string;
    } = {}
  ): Promise<void> {
    const { resourceId, metadata, request, userId } = options;

    const logEntry: AuditLogEntry = {
      userId,
      action,
      resourceType,
      resourceId,
      ipAddress: request ? this.getClientIp(request) : undefined,
      userAgent: request?.headers.get('user-agent') || undefined,
      metadata,
      createdAt: new Date(),
    };

    // Store in database (example using Prisma)
    try {
      const { PrismaClient } = await import('@prisma/client');
      const prisma = new PrismaClient();

      await prisma.auditLog.create({
        data: {
          userId: logEntry.userId,
          action: logEntry.action,
          resourceType: logEntry.resourceType,
          resourceId: logEntry.resourceId?.toString(),
          ipAddress: logEntry.ipAddress,
          userAgent: logEntry.userAgent,
          metadata: logEntry.metadata || {},
          createdAt: logEntry.createdAt,
        },
      });

      await prisma.$disconnect();
    } catch (error) {
      console.error('Failed to create audit log:', error);
    }

    // Also log to console/external service
    console.log(`[AUDIT] ${action} ${resourceType}`, {
      userId: logEntry.userId,
      resourceId: logEntry.resourceId,
      ipAddress: logEntry.ipAddress,
    });
  }

  /**
   * Logs a security event.
   *
   * @param event - The security event type
   * @param details - Event details
   * @param request - The request object
   */
  static async logSecurityEvent(
    event: string,
    details?: Record<string, any>,
    request?: NextRequest
  ): Promise<void> {
    await this.log(`security.${event}`, 'security', {
      metadata: details,
      request,
    });
  }

  /**
   * Logs an access denied event.
   *
   * @param resourceType - The resource that was denied
   * @param resourceId - The resource ID
   * @param request - The request object
   */
  static async logAccessDenied(
    resourceType: string,
    resourceId?: string | number,
    request?: NextRequest
  ): Promise<void> {
    const metadata: Record<string, any> = {};

    if (request) {
      metadata.path = request.nextUrl.pathname;
      metadata.method = request.method;
    }

    await this.logSecurityEvent(
      'access_denied',
      {
        resource: resourceType,
        resourceId,
        ...metadata,
      },
      request
    );
  }

  /**
   * Gets the client IP address from the request.
   *
   * @param request - Next.js request object
   * @returns Client IP address
   */
  private static getClientIp(request: NextRequest): string {
    const forwarded = request.headers.get('x-forwarded-for');
    const ip = forwarded ? forwarded.split(',')[0].trim() : request.ip || '127.0.0.1';
    return ip;
  }
}
```

#### prisma/schema.prisma (Audit Log Model)

```prisma
// Audit log model for Prisma ORM

model AuditLog {
  id           String   @id @default(cuid())
  userId       String?
  action       String   @db.VarChar(100)
  resourceType String   @db.VarChar(100)
  resourceId   String?
  ipAddress    String?  @db.VarChar(45)
  userAgent    String?  @db.Text
  metadata     Json     @default("{}")
  createdAt    DateTime @default(now())

  user User? @relation(fields: [userId], references: [id], onDelete: SetNull)

  @@index([action, createdAt])
  @@index([resourceType, resourceId, createdAt])
  @@index([userId, createdAt])
  @@map("audit_logs")
}
```

#### app/api/example/route.ts (Usage Example)

```typescript
/**
 * Example API route demonstrating audit logging.
 */

import { NextRequest, NextResponse } from 'next/server';
import { AuditService } from '@/lib/audit-service';

/**
 * Handles POST requests with audit logging.
 *
 * @param request - The incoming request
 * @returns Response with operation result
 */
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    // Perform operation...

    // Log the action
    await AuditService.log('create', 'resource', {
      resourceId: 'resource-123',
      userId: 'user-456',
      metadata: {
        details: 'Additional context about the action',
      },
      request,
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    // Log security event on error
    await AuditService.logSecurityEvent('operation_failed', {
      error: error instanceof Error ? error.message : 'Unknown error',
    }, request);

    return NextResponse.json(
      { error: 'Operation failed' },
      { status: 500 }
    );
  }
}
```

---

## React Native Stack

### Security Headers Note

**React Native does not handle HTTP security headers directly** as it is a client-side mobile application framework. Security headers are configured and enforced by the backend API servers that the React Native application communicates with.

For React Native applications:

1. **Backend API Security**: Ensure your backend APIs (Laravel, Django, Node.js/Next.js, etc.) implement the security headers shown in the sections above.

2. **HTTPS Enforcement**: Always use HTTPS for API communication. Configure this in your API client:

```typescript
/**
 * api-client.ts
 *
 * Secure API client configuration for React Native.
 * Enforces HTTPS and implements certificate pinning.
 */

import axios from 'axios';

const apiClient = axios.create({
  baseURL: process.env.EXPO_PUBLIC_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Ensure HTTPS is used
if (!apiClient.defaults.baseURL?.startsWith('https://')) {
  if (process.env.NODE_ENV === 'production') {
    throw new Error('API must use HTTPS in production');
  }
  console.warn('Warning: API is not using HTTPS');
}

export default apiClient;
```

3. **Certificate Pinning**: For production apps, implement SSL certificate pinning to prevent man-in-the-middle attacks:

```typescript
/**
 * Certificate pinning configuration (using expo-secure-store or react-native-ssl-pinning).
 * This ensures the app only trusts specific certificates.
 */

// Example using react-native-ssl-pinning
import { fetch } from 'react-native-ssl-pinning';

const secureRequest = async () => {
  const response = await fetch('https://api.example.com/data', {
    method: 'GET',
    timeoutInterval: 10000,
    sslPinning: {
      certs: ['cert1', 'cert2'], // Certificate names in your bundle
    },
  });

  return await response.json();
};
```

4. **Secure Storage**: Use platform-specific secure storage for sensitive data:

```typescript
/**
 * Secure storage helper for React Native.
 * Uses expo-secure-store or react-native-keychain.
 */

import * as SecureStore from 'expo-secure-store';

export const secureStorage = {
  /**
   * Stores a value securely.
   */
  async set(key: string, value: string): Promise<void> {
    await SecureStore.setItemAsync(key, value);
  },

  /**
   * Retrieves a value securely.
   */
  async get(key: string): Promise<string | null> {
    return await SecureStore.getItemAsync(key);
  },

  /**
   * Removes a value from secure storage.
   */
  async remove(key: string): Promise<void> {
    await SecureStore.deleteItemAsync(key);
  },
};
```

5. **Rate Limiting**: Implement client-side request throttling to complement backend rate limiting:

```typescript
/**
 * Client-side rate limiting utility.
 * Prevents excessive requests from the mobile app.
 */

class RequestThrottler {
  private requests: Map<string, number[]> = new Map();
  private readonly maxRequests: number;
  private readonly windowMs: number;

  constructor(maxRequests: number = 60, windowMs: number = 60000) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }

  /**
   * Checks if a request is allowed based on rate limits.
   */
  canMakeRequest(endpoint: string): boolean {
    const now = Date.now();
    const timestamps = this.requests.get(endpoint) || [];

    // Remove old timestamps outside the window
    const validTimestamps = timestamps.filter(t => now - t < this.windowMs);

    if (validTimestamps.length >= this.maxRequests) {
      return false;
    }

    validTimestamps.push(now);
    this.requests.set(endpoint, validTimestamps);
    return true;
  }
}

export const requestThrottler = new RequestThrottler(60, 60000);
```

**Key Points for React Native Security:**
- Security headers are enforced by the backend APIs, not the React Native app
- Focus on HTTPS enforcement, certificate pinning, and secure storage
- Implement client-side rate limiting to complement backend protections
- Use secure authentication mechanisms (OAuth 2.0, JWT with refresh tokens)
- Validate and sanitise all user input before sending to APIs
- Keep dependencies up to date to patch security vulnerabilities
