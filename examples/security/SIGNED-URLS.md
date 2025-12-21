# Signed URLs & Token-Based Access

## Overview

Signed URLs and token-based access patterns for protecting sensitive routes. These patterns ensure that sensitive URLs cannot be guessed, enumerated, or reused. All examples use British English spelling and follow secure coding practices.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **Laravel** | 12.x |
| **PHP** | 8.4 |
| **MariaDB** | 12.x |
| **Django** | 6.x |
| **Python** | 3.14 |
| **PostgreSQL** | 18.x |
| **Strawberry GraphQL** | Latest |
| **Next.js** | 16.x |
| **React** | 19.x |
| **TypeScript** | 5.9 |
| **Prisma** | 6.x |
| **React Native** | 0.83.x |
| **NativeWind** | v4 |
| **Stacks** | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Signed URLs - Laravel (TALL Stack)](#signed-urls---laravel-tall-stack)
- [Randomised Admin Paths - Laravel](#randomised-admin-paths---laravel)
- [Token-Based Access - Laravel](#token-based-access---laravel)
- [Signed URLs - Django/Wagtail](#signed-urls---djangowagtail)
- [Token-Based Access - Django](#token-based-access---django)
- [Signed URLs - Next.js](#signed-urls---nextjs)
- [Token-Based Access - Next.js](#token-based-access---nextjs)
- [React Native - Consuming Signed URLs](#react-native---consuming-signed-urls)

## Signed URLs - Laravel (TALL Stack)

### app/Http/Controllers/SecureRouteController.php

```php
<?php

/**
 * SecureRouteController.php
 *
 * Generates signed URLs for sensitive actions.
 * Signed URLs include a cryptographic signature that validates the request.
 */

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;

class SecureRouteController extends Controller
{
    /**
     * Generates a signed URL for a sensitive action.
     * The URL expires after 1 hour and includes a cryptographic signature.
     *
     * @param User $user The user to generate the link for
     * @return string The signed URL
     */
    public function generateSecureLink(User $user): string
    {
        return URL::temporarySignedRoute(
            'admin.sensitive-action',
            now()->addHour(),
            ['user' => $user->id]
        );
    }

    /**
     * Handles the sensitive action after signature validation.
     * The 'signed' middleware validates the URL signature automatically.
     *
     * @param Request $request The validated request
     * @param User $user The user from route binding
     * @return \Illuminate\Http\JsonResponse
     */
    public function sensitiveAction(Request $request, User $user)
    {
        // Action is only reachable with valid signature
        return response()->json(['status' => 'Action completed']);
    }
}
```

### routes/web.php (Signed Routes)

```php
<?php

use App\Http\Controllers\SecureRouteController;
use Illuminate\Support\Facades\Route;

// Signed route - validates signature automatically
Route::get('/secure/{user}/action', [SecureRouteController::class, 'sensitiveAction'])
    ->name('admin.sensitive-action')
    ->middleware('signed');

// Generate signed URLs in controllers:
// $url = URL::temporarySignedRoute('admin.sensitive-action', now()->addHour(), ['user' => $user->id]);
```

---

## Randomised Admin Paths - Laravel

### config/admin.php

```php
<?php

/**
 * Admin path configuration.
 *
 * Stores the admin path prefix in environment variables to prevent
 * predictable admin URLs like /admin or /dashboard.
 */

return [
    // Store in env, not predictable
    // Generate with: bin2hex(random_bytes(8))
    'path_prefix' => env('ADMIN_PATH_PREFIX', 'management'),
];
```

### routes/web.php (Admin Routes)

```php
<?php

use App\Http\Controllers\AdminController;
use Illuminate\Support\Facades\Route;

// Admin routes with randomised path prefix
Route::prefix(config('admin.path_prefix'))
    ->middleware(['auth', 'role:admin'])
    ->group(function () {
        Route::get('/', [AdminController::class, 'dashboard'])->name('admin.dashboard');
        // ... additional admin routes
    });

// Example: if ADMIN_PATH_PREFIX=mgmt-a1b2c3d4
// Admin dashboard accessible at: /mgmt-a1b2c3d4/
```

---

## Token-Based Access - Laravel

### app/Http/Controllers/SecureAccessController.php

```php
<?php

/**
 * SecureAccessController.php
 *
 * Handles token-based access to protected resources.
 * Tokens are single-use and IP-bound for security.
 */

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Str;

class SecureAccessController extends Controller
{
    /**
     * Generates a secure access token for a resource.
     * Token is bound to the requesting user and IP address.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse Token response
     */
    public function generateAccessToken(Request $request)
    {
        $token = Str::random(64);

        cache()->put(
            "secure_access:{$token}",
            [
                'user_id' => $request->user()->id,
                'resource' => $request->resource,
                'permissions' => $request->permissions,
                'ip' => $request->ip(),
            ],
            now()->addMinutes(30)
        );

        return response()->json(['access_token' => $token]);
    }

    /**
     * Validates and consumes an access token to retrieve a resource.
     * Token is invalidated after first use (single-use).
     *
     * @param Request $request The HTTP request
     * @param string $token The access token
     * @return mixed The protected resource or 404
     */
    public function accessResource(Request $request, string $token)
    {
        $access = cache()->get("secure_access:{$token}");

        if (!$access) {
            abort(404); // Return 404, not 403, to hide existence
        }

        // Validate IP if required
        if ($access['ip'] !== $request->ip()) {
            cache()->forget("secure_access:{$token}");
            abort(404);
        }

        // Single-use: delete after access
        cache()->forget("secure_access:{$token}");

        return $this->loadResource($access['resource']);
    }
}
```

---

## Signed URLs - Django/Wagtail

### services/signed_urls.py

```python
"""
signed_urls.py

Generates and validates signed URLs for sensitive actions.
Uses Django's signing framework and itsdangerous for cryptographic signatures.
Compatible with Django 6.x and Python 3.14.
"""

from datetime import timedelta
from typing import Any, Optional

from django.core import signing
from django.urls import reverse
from django.utils import timezone
from itsdangerous import URLSafeTimedSerializer, BadSignature, SignatureExpired


class SignedUrlService:
    """
    Service for generating and validating signed URLs.
    Signed URLs include an expiry timestamp and cryptographic signature.
    Uses both Django's native signing and itsdangerous for flexibility.
    """

    def __init__(self, max_age_seconds: int = 3600):
        """
        Initialises the signed URL service.

        Args:
            max_age_seconds: URL validity period in seconds (default: 1 hour)
        """
        self.max_age = max_age_seconds

    def generate_signed_url(
        self,
        view_name: str,
        user_id: int,
        action: str,
        **kwargs: Any
    ) -> str:
        """
        Generates a signed URL for a sensitive action using Django signing.

        Args:
            view_name: The Django view name to generate URL for
            user_id: The user ID to embed in the signature
            action: The action type being signed
            **kwargs: Additional parameters to include in the payload

        Returns:
            The complete signed URL with signature parameter
        """
        payload = {
            'user_id': user_id,
            'action': action,
            'expires': (timezone.now() + timedelta(seconds=self.max_age)).isoformat(),
            **kwargs
        }

        signature = signing.dumps(payload)
        base_url = reverse(view_name)

        return f"{base_url}?sig={signature}"

    def validate_signature(self, signature: str) -> Optional[dict]:
        """
        Validates a URL signature and returns the payload.

        Args:
            signature: The signature string from the URL

        Returns:
            The decoded payload if valid, None if invalid or expired
        """
        try:
            payload = signing.loads(signature, max_age=self.max_age)
            return payload
        except signing.BadSignature:
            return None


class ItsDangerousSignedUrlService:
    """
    Alternative signed URL service using itsdangerous library.
    Provides more granular control and cross-framework compatibility.
    """

    def __init__(self, secret_key: str, salt: str = 'signed-urls', max_age_seconds: int = 3600):
        """
        Initialises the itsdangerous signed URL service.

        Args:
            secret_key: Secret key for signing (use Django's SECRET_KEY)
            salt: Salt value for additional security
            max_age_seconds: URL validity period in seconds (default: 1 hour)
        """
        self.serialiser = URLSafeTimedSerializer(secret_key, salt=salt)
        self.max_age = max_age_seconds

    def generate_signed_url(
        self,
        view_name: str,
        user_id: int,
        action: str,
        **kwargs: Any
    ) -> str:
        """
        Generates a signed URL using itsdangerous.

        Args:
            view_name: The Django view name to generate URL for
            user_id: The user ID to embed in the signature
            action: The action type being signed
            **kwargs: Additional parameters to include in the payload

        Returns:
            The complete signed URL with signature parameter
        """
        payload = {
            'user_id': user_id,
            'action': action,
            **kwargs
        }

        token = self.serialiser.dumps(payload)
        base_url = reverse(view_name)

        return f"{base_url}?token={token}"

    def validate_signature(self, token: str) -> Optional[dict]:
        """
        Validates a signed token and returns the payload.

        Args:
            token: The signed token from the URL

        Returns:
            The decoded payload if valid, None if invalid or expired
        """
        try:
            payload = self.serialiser.loads(token, max_age=self.max_age)
            return payload
        except (BadSignature, SignatureExpired):
            return None


# Usage in views:
# service = SignedUrlService()
# url = service.generate_signed_url('admin:sensitive-action', user.id, 'export')
#
# Or with itsdangerous:
# from django.conf import settings
# service = ItsDangerousSignedUrlService(settings.SECRET_KEY)
# url = service.generate_signed_url('admin:sensitive-action', user.id, 'export')
```

### views/secure_routes.py

```python
"""
secure_routes.py

Django views for handling signed URL requests.
Validates signatures before processing sensitive actions.
"""

from django.conf import settings
from django.http import JsonResponse, HttpRequest, HttpResponse
from django.views import View
from django.views.decorators.http import require_http_methods

from .services.signed_urls import SignedUrlService, ItsDangerousSignedUrlService


class SignedUrlView(View):
    """
    Base view for handling signed URL requests.
    Validates signatures before allowing access to sensitive actions.
    """

    def __init__(self, **kwargs):
        """Initialises the view with signed URL service."""
        super().__init__(**kwargs)
        self.signed_url_service = ItsDangerousSignedUrlService(settings.SECRET_KEY)

    def get(self, request: HttpRequest) -> HttpResponse:
        """
        Handles GET requests with signed URL validation.

        Args:
            request: The HTTP request object

        Returns:
            JSON response with action result or 404 if invalid
        """
        token = request.GET.get('token')

        if not token:
            return JsonResponse({'error': 'No token provided'}, status=404)

        payload = self.signed_url_service.validate_signature(token)

        if not payload:
            return JsonResponse({'error': 'Invalid or expired signature'}, status=404)

        # Process the signed action
        return self.process_signed_action(payload)

    def process_signed_action(self, payload: dict) -> HttpResponse:
        """
        Processes the validated signed action.
        Override this method in subclasses.

        Args:
            payload: The validated payload from the signature

        Returns:
            HTTP response for the action
        """
        return JsonResponse({
            'status': 'success',
            'action': payload.get('action'),
            'user_id': payload.get('user_id')
        })


@require_http_methods(["GET"])
def sensitive_export_view(request: HttpRequest) -> HttpResponse:
    """
    Example view for handling sensitive data export with signed URL.

    Args:
        request: The HTTP request object

    Returns:
        Export file or error response
    """
    service = SignedUrlService()
    signature = request.GET.get('sig')

    if not signature:
        return JsonResponse({'error': 'Invalid request'}, status=404)

    payload = service.validate_signature(signature)

    if not payload:
        return JsonResponse({'error': 'Invalid or expired link'}, status=404)

    # Validate user permissions
    user_id = payload.get('user_id')
    action = payload.get('action')

    # Process the export
    return JsonResponse({
        'status': 'Export generated',
        'user_id': user_id,
        'action': action
    })
```

### urls.py

```python
"""
URL configuration for signed routes.
"""

from django.urls import path
from .views import secure_routes

urlpatterns = [
    path(
        'secure/export/',
        secure_routes.sensitive_export_view,
        name='admin:sensitive-export'
    ),
    path(
        'secure/action/',
        secure_routes.SignedUrlView.as_view(),
        name='admin:signed-action'
    ),
]
```

---

## Token-Based Access - Django

### services/token_access.py

```python
"""
token_access.py

Handles token-based access to protected resources in Django.
Tokens are single-use and IP-bound for enhanced security.
Compatible with Django 6.x and Python 3.14.
"""

from datetime import timedelta
from typing import Optional, Dict, Any
import secrets

from django.core.cache import cache
from django.utils import timezone


class TokenAccessService:
    """
    Service for generating and validating single-use access tokens.
    Tokens are bound to IP addresses and expire after use or timeout.
    """

    def __init__(self, ttl_minutes: int = 30):
        """
        Initialises the token access service.

        Args:
            ttl_minutes: Token time-to-live in minutes (default: 30)
        """
        self.ttl = timedelta(minutes=ttl_minutes)

    def generate_access_token(
        self,
        user_id: int,
        resource: str,
        permissions: list[str],
        ip_address: str,
        **metadata: Any
    ) -> str:
        """
        Generates a secure access token for a resource.
        Token is bound to the requesting user and IP address.

        Args:
            user_id: The user ID requesting access
            resource: The resource identifier
            permissions: List of granted permissions
            ip_address: The client IP address
            **metadata: Additional metadata to store with the token

        Returns:
            The generated access token string
        """
        token = secrets.token_urlsafe(48)

        payload = {
            'user_id': user_id,
            'resource': resource,
            'permissions': permissions,
            'ip': ip_address,
            'created_at': timezone.now().isoformat(),
            **metadata
        }

        cache_key = f'secure_access:{token}'
        cache.set(cache_key, payload, timeout=int(self.ttl.total_seconds()))

        return token

    def validate_and_consume_token(
        self,
        token: str,
        ip_address: str
    ) -> Optional[Dict[str, Any]]:
        """
        Validates and consumes an access token.
        Token is invalidated after first use (single-use).

        Args:
            token: The access token to validate
            ip_address: The client IP address for validation

        Returns:
            The token payload if valid, None if invalid or expired
        """
        cache_key = f'secure_access:{token}'
        payload = cache.get(cache_key)

        if not payload:
            return None  # Token not found or expired

        # Validate IP binding
        if payload.get('ip') != ip_address:
            cache.delete(cache_key)
            return None

        # Single-use: delete after access
        cache.delete(cache_key)

        return payload

    def revoke_token(self, token: str) -> bool:
        """
        Revokes an access token before expiry.

        Args:
            token: The access token to revoke

        Returns:
            True if token was revoked, False if not found
        """
        cache_key = f'secure_access:{token}'
        return cache.delete(cache_key) > 0


# Usage in views:
# service = TokenAccessService()
# token = service.generate_access_token(
#     user_id=user.id,
#     resource='sensitive-data',
#     permissions=['read'],
#     ip_address=request.META.get('REMOTE_ADDR')
# )
```

### views/token_protected.py

```python
"""
token_protected.py

Views for handling token-protected resource access.
"""

from django.http import JsonResponse, HttpRequest, HttpResponse
from django.views.decorators.http import require_http_methods

from .services.token_access import TokenAccessService


@require_http_methods(["POST"])
def generate_access_token_view(request: HttpRequest) -> JsonResponse:
    """
    Generates an access token for a protected resource.

    Args:
        request: The HTTP request object

    Returns:
        JSON response with the access token
    """
    if not request.user.is_authenticated:
        return JsonResponse({'error': 'Unauthorised'}, status=401)

    service = TokenAccessService()

    # Extract request parameters
    resource = request.POST.get('resource')
    permissions = request.POST.getlist('permissions')

    if not resource:
        return JsonResponse({'error': 'Resource required'}, status=400)

    token = service.generate_access_token(
        user_id=request.user.id,
        resource=resource,
        permissions=permissions or ['read'],
        ip_address=request.META.get('REMOTE_ADDR')
    )

    return JsonResponse({
        'access_token': token,
        'expires_in': 1800  # 30 minutes
    })


@require_http_methods(["GET"])
def access_protected_resource_view(request: HttpRequest, token: str) -> HttpResponse:
    """
    Accesses a protected resource using a single-use token.

    Args:
        request: The HTTP request object
        token: The access token

    Returns:
        The protected resource or error response
    """
    service = TokenAccessService()

    payload = service.validate_and_consume_token(
        token=token,
        ip_address=request.META.get('REMOTE_ADDR')
    )

    if not payload:
        # Return 404 instead of 403 to hide resource existence
        return JsonResponse({'error': 'Not found'}, status=404)

    # Load and return the protected resource
    resource_id = payload.get('resource')
    permissions = payload.get('permissions', [])

    return JsonResponse({
        'status': 'success',
        'resource': resource_id,
        'permissions': permissions,
        'data': 'Protected resource data'
    })
```

---

## Signed URLs - Next.js

### lib/signed-urls.ts

```typescript
/**
 * signed-urls.ts
 *
 * Generates and validates signed URLs for sensitive actions in Next.js.
 * Uses Node.js crypto module for cryptographic signatures.
 * Compatible with Next.js 16.x and TypeScript 5.9.
 */

import crypto from 'crypto';

interface SignedUrlPayload {
  userId: string;
  action: string;
  resource?: string;
  expiresAt: string;
  [key: string]: any;
}

export class SignedUrlService {
  private readonly secretKey: string;
  private readonly algorithm: string = 'sha256';

  /**
   * Initialises the signed URL service.
   *
   * @param secretKey - Secret key for signing (use environment variable)
   */
  constructor(secretKey: string) {
    if (!secretKey) {
      throw new Error('Secret key is required for signed URLs');
    }
    this.secretKey = secretKey;
  }

  /**
   * Generates a signed URL for a sensitive action.
   * The URL includes a cryptographic signature and expiry timestamp.
   *
   * @param basePath - The base path for the URL
   * @param payload - The data to embed in the signature
   * @param expiryMinutes - URL validity period in minutes (default: 60)
   * @returns The complete signed URL with signature parameter
   */
  generateSignedUrl(
    basePath: string,
    payload: Omit<SignedUrlPayload, 'expiresAt'>,
    expiryMinutes: number = 60
  ): string {
    const expiresAt = new Date(Date.now() + expiryMinutes * 60 * 1000).toISOString();

    const fullPayload: SignedUrlPayload = {
      ...payload,
      expiresAt,
    };

    // Create signature
    const payloadString = JSON.stringify(fullPayload);
    const signature = crypto
      .createHmac(this.algorithm, this.secretKey)
      .update(payloadString)
      .digest('base64url');

    // Encode payload for URL
    const encodedPayload = Buffer.from(payloadString).toString('base64url');

    // Construct signed URL
    const url = new URL(basePath, process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000');
    url.searchParams.set('sig', signature);
    url.searchParams.set('data', encodedPayload);

    return url.toString();
  }

  /**
   * Validates a signed URL and returns the payload.
   * Checks both signature validity and expiry time.
   *
   * @param signature - The signature from the URL
   * @param encodedData - The encoded payload data
   * @returns The decoded payload if valid, null if invalid or expired
   */
  validateSignature(signature: string, encodedData: string): SignedUrlPayload | null {
    try {
      // Decode payload
      const payloadString = Buffer.from(encodedData, 'base64url').toString('utf-8');
      const payload: SignedUrlPayload = JSON.parse(payloadString);

      // Verify signature
      const expectedSignature = crypto
        .createHmac(this.algorithm, this.secretKey)
        .update(payloadString)
        .digest('base64url');

      if (signature !== expectedSignature) {
        return null; // Invalid signature
      }

      // Check expiry
      const expiresAt = new Date(payload.expiresAt);
      if (expiresAt < new Date()) {
        return null; // Expired
      }

      return payload;
    } catch (error) {
      return null; // Parsing or validation error
    }
  }
}

// Export singleton instance
export const signedUrlService = new SignedUrlService(
  process.env.SIGNED_URL_SECRET_KEY || process.env.SECRET_KEY || ''
);

/**
 * Helper function to generate signed URLs.
 *
 * @param path - The path to sign
 * @param userId - The user ID
 * @param action - The action being performed
 * @param expiryMinutes - Optional expiry time in minutes
 * @returns Signed URL string
 */
export function generateSignedUrl(
  path: string,
  userId: string,
  action: string,
  expiryMinutes?: number
): string {
  return signedUrlService.generateSignedUrl(
    path,
    { userId, action },
    expiryMinutes
  );
}
```

### app/api/secure/[action]/route.ts

```typescript
/**
 * route.ts
 *
 * API route for handling signed URL requests in Next.js.
 * Validates signatures before processing sensitive actions.
 */

import { NextRequest, NextResponse } from 'next/server';
import { signedUrlService } from '@/lib/signed-urls';

/**
 * Handles GET requests to signed URLs.
 * Validates the signature before allowing access.
 *
 * @param request - The Next.js request object
 * @param params - Route parameters
 * @returns JSON response with action result or error
 */
export async function GET(
  request: NextRequest,
  { params }: { params: { action: string } }
) {
  const searchParams = request.nextUrl.searchParams;
  const signature = searchParams.get('sig');
  const encodedData = searchParams.get('data');

  if (!signature || !encodedData) {
    return NextResponse.json(
      { error: 'Invalid request' },
      { status: 404 }
    );
  }

  // Validate signature
  const payload = signedUrlService.validateSignature(signature, encodedData);

  if (!payload) {
    return NextResponse.json(
      { error: 'Invalid or expired signature' },
      { status: 404 }
    );
  }

  // Verify action matches
  if (payload.action !== params.action) {
    return NextResponse.json(
      { error: 'Action mismatch' },
      { status: 404 }
    );
  }

  // Process the signed action
  return NextResponse.json({
    status: 'success',
    action: payload.action,
    userId: payload.userId,
    message: 'Signed action processed successfully',
  });
}
```

### middleware.ts

```typescript
/**
 * middleware.ts
 *
 * Next.js middleware for validating signed URLs.
 * Provides automatic signature validation for protected routes.
 */

import { NextRequest, NextResponse } from 'next/server';
import { signedUrlService } from '@/lib/signed-urls';

/**
 * Middleware to validate signed URLs on protected routes.
 *
 * @param request - The Next.js request object
 * @returns Response allowing or denying access
 */
export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;

  // Only apply to signed routes
  if (!pathname.startsWith('/api/secure/')) {
    return NextResponse.next();
  }

  const searchParams = request.nextUrl.searchParams;
  const signature = searchParams.get('sig');
  const encodedData = searchParams.get('data');

  if (!signature || !encodedData) {
    return NextResponse.json(
      { error: 'Unauthorised' },
      { status: 404 }
    );
  }

  // Validate signature
  const payload = signedUrlService.validateSignature(signature, encodedData);

  if (!payload) {
    return NextResponse.json(
      { error: 'Invalid or expired signature' },
      { status: 404 }
    );
  }

  // Add payload to request headers for downstream handlers
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-signed-payload', JSON.stringify(payload));

  return NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });
}

export const config = {
  matcher: '/api/secure/:path*',
};
```

---

## Token-Based Access - Next.js

### lib/token-access.ts

```typescript
/**
 * token-access.ts
 *
 * Handles token-based access to protected resources in Next.js.
 * Tokens are single-use and IP-bound for enhanced security.
 * Compatible with Next.js 16.x and TypeScript 5.9.
 */

import crypto from 'crypto';

interface TokenPayload {
  userId: string;
  resource: string;
  permissions: string[];
  ip: string;
  createdAt: string;
  [key: string]: any;
}

/**
 * In-memory token storage.
 * In production, use Redis or another distributed cache.
 */
const tokenStore = new Map<string, TokenPayload>();

export class TokenAccessService {
  private readonly ttlMinutes: number;

  /**
   * Initialises the token access service.
   *
   * @param ttlMinutes - Token time-to-live in minutes (default: 30)
   */
  constructor(ttlMinutes: number = 30) {
    this.ttlMinutes = ttlMinutes;
  }

  /**
   * Generates a secure access token for a resource.
   * Token is bound to the requesting user and IP address.
   *
   * @param userId - The user ID requesting access
   * @param resource - The resource identifier
   * @param permissions - Array of granted permissions
   * @param ip - The client IP address
   * @param metadata - Additional metadata to store
   * @returns The generated access token
   */
  async generateAccessToken(
    userId: string,
    resource: string,
    permissions: string[],
    ip: string,
    metadata?: Record<string, any>
  ): Promise<string> {
    const token = crypto.randomBytes(32).toString('hex');

    const payload: TokenPayload = {
      userId,
      resource,
      permissions,
      ip,
      createdAt: new Date().toISOString(),
      ...metadata,
    };

    // Store token
    tokenStore.set(token, payload);

    // Set expiry
    setTimeout(() => {
      tokenStore.delete(token);
    }, this.ttlMinutes * 60 * 1000);

    return token;
  }

  /**
   * Validates and consumes an access token.
   * Token is invalidated after first use (single-use).
   *
   * @param token - The access token to validate
   * @param ip - The client IP address for validation
   * @returns The token payload if valid, null otherwise
   */
  async validateAndConsumeToken(
    token: string,
    ip: string
  ): Promise<TokenPayload | null> {
    const payload = tokenStore.get(token);

    if (!payload) {
      return null; // Token not found or expired
    }

    // Validate IP binding
    if (payload.ip !== ip) {
      tokenStore.delete(token);
      return null;
    }

    // Single-use: delete after access
    tokenStore.delete(token);

    return payload;
  }

  /**
   * Revokes an access token before expiry.
   *
   * @param token - The access token to revoke
   * @returns True if token was revoked, false if not found
   */
  async revokeToken(token: string): Promise<boolean> {
    return tokenStore.delete(token);
  }
}

// Export singleton instance
export const tokenAccessService = new TokenAccessService(30);
```

### app/api/token/generate/route.ts

```typescript
/**
 * route.ts
 *
 * API endpoint for generating access tokens.
 */

import { NextRequest, NextResponse } from 'next/server';
import { tokenAccessService } from '@/lib/token-access';
import { getServerSession } from 'next-auth';

/**
 * Generates a new access token for a protected resource.
 *
 * @param request - The Next.js request object
 * @returns JSON response with the access token
 */
export async function POST(request: NextRequest) {
  const session = await getServerSession();

  if (!session?.user) {
    return NextResponse.json(
      { error: 'Unauthorised' },
      { status: 401 }
    );
  }

  const body = await request.json();
  const { resource, permissions } = body;

  if (!resource) {
    return NextResponse.json(
      { error: 'Resource required' },
      { status: 400 }
    );
  }

  // Get client IP
  const ip = request.headers.get('x-forwarded-for')?.split(',')[0] ||
             request.headers.get('x-real-ip') ||
             'unknown';

  const token = await tokenAccessService.generateAccessToken(
    session.user.id || session.user.email || 'unknown',
    resource,
    permissions || ['read'],
    ip
  );

  return NextResponse.json({
    access_token: token,
    expires_in: 1800, // 30 minutes
  });
}
```

### app/api/resource/[token]/route.ts

```typescript
/**
 * route.ts
 *
 * API endpoint for accessing protected resources with tokens.
 */

import { NextRequest, NextResponse } from 'next/server';
import { tokenAccessService } from '@/lib/token-access';

/**
 * Accesses a protected resource using a single-use token.
 *
 * @param request - The Next.js request object
 * @param params - Route parameters containing the token
 * @returns The protected resource or error response
 */
export async function GET(
  request: NextRequest,
  { params }: { params: { token: string } }
) {
  const { token } = params;

  // Get client IP
  const ip = request.headers.get('x-forwarded-for')?.split(',')[0] ||
             request.headers.get('x-real-ip') ||
             'unknown';

  const payload = await tokenAccessService.validateAndConsumeToken(token, ip);

  if (!payload) {
    // Return 404 instead of 403 to hide resource existence
    return NextResponse.json(
      { error: 'Not found' },
      { status: 404 }
    );
  }

  // Load and return the protected resource
  return NextResponse.json({
    status: 'success',
    resource: payload.resource,
    permissions: payload.permissions,
    userId: payload.userId,
    data: 'Protected resource data',
  });
}
```

---

## React Native - Consuming Signed URLs

### services/signedUrlClient.ts

```typescript
/**
 * signedUrlClient.ts
 *
 * React Native client for consuming signed URLs from backend APIs.
 * Handles signed URL requests with proper error handling.
 * Compatible with React Native 0.83.x and TypeScript 5.9.
 */

import { Platform } from 'react-native';

interface SignedUrlResponse {
  status: string;
  action: string;
  userId: string;
  data?: any;
  [key: string]: any;
}

interface AccessTokenResponse {
  access_token: string;
  expires_in: number;
}

export class SignedUrlClient {
  private readonly baseUrl: string;

  /**
   * Initialises the signed URL client.
   *
   * @param baseUrl - The base URL of the API
   */
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  /**
   * Fetches data from a signed URL.
   * Handles signature validation and error responses.
   *
   * @param signedUrl - The complete signed URL from the backend
   * @returns The response data from the signed endpoint
   * @throws Error if the request fails or signature is invalid
   */
  async fetchSignedUrl<T = SignedUrlResponse>(signedUrl: string): Promise<T> {
    try {
      const response = await fetch(signedUrl, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'User-Agent': `ReactNative/${Platform.OS}`,
        },
      });

      if (!response.ok) {
        // Handle 404 (invalid/expired signature)
        if (response.status === 404) {
          throw new Error('This link is invalid or has expired');
        }

        throw new Error(`Request failed with status ${response.status}`);
      }

      const data = await response.json();
      return data as T;
    } catch (error) {
      if (error instanceof Error) {
        throw error;
      }
      throw new Error('Failed to fetch signed URL');
    }
  }

  /**
   * Requests an access token for a protected resource.
   * Token can then be used to access the resource once.
   *
   * @param resource - The resource identifier
   * @param permissions - Array of requested permissions
   * @param authToken - User authentication token
   * @returns Access token response with token and expiry
   * @throws Error if token generation fails
   */
  async requestAccessToken(
    resource: string,
    permissions: string[],
    authToken: string
  ): Promise<AccessTokenResponse> {
    try {
      const response = await fetch(`${this.baseUrl}/api/token/generate`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${authToken}`,
        },
        body: JSON.stringify({
          resource,
          permissions,
        }),
      });

      if (!response.ok) {
        throw new Error(`Token generation failed with status ${response.status}`);
      }

      const data = await response.json();
      return data as AccessTokenResponse;
    } catch (error) {
      if (error instanceof Error) {
        throw error;
      }
      throw new Error('Failed to generate access token');
    }
  }

  /**
   * Accesses a protected resource using a single-use token.
   *
   * @param token - The access token
   * @returns The protected resource data
   * @throws Error if access fails or token is invalid
   */
  async accessProtectedResource<T = any>(token: string): Promise<T> {
    try {
      const response = await fetch(`${this.baseUrl}/api/resource/${token}`, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
        },
      });

      if (!response.ok) {
        if (response.status === 404) {
          throw new Error('Resource not found or access token is invalid');
        }

        throw new Error(`Access failed with status ${response.status}`);
      }

      const data = await response.json();
      return data as T;
    } catch (error) {
      if (error instanceof Error) {
        throw error;
      }
      throw new Error('Failed to access protected resource');
    }
  }
}

// Export singleton instance
const API_BASE_URL = process.env.EXPO_PUBLIC_API_URL || 'http://localhost:3000';
export const signedUrlClient = new SignedUrlClient(API_BASE_URL);
```

### components/SignedUrlHandler.tsx

```typescript
/**
 * SignedUrlHandler.tsx
 *
 * React Native component for handling signed URL navigation.
 * Displays loading states and error handling for signed URL access.
 */

import React, { useEffect, useState } from 'react';
import { View, Text, ActivityIndicator, StyleSheet } from 'react-native';
import { signedUrlClient } from '../services/signedUrlClient';

interface SignedUrlHandlerProps {
  signedUrl: string;
  onSuccess?: (data: any) => void;
  onError?: (error: Error) => void;
}

/**
 * Component that handles signed URL fetching with loading and error states.
 *
 * @param props - Component props
 * @returns React Native component
 */
export const SignedUrlHandler: React.FC<SignedUrlHandlerProps> = ({
  signedUrl,
  onSuccess,
  onError,
}) => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [data, setData] = useState<any>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);

        const response = await signedUrlClient.fetchSignedUrl(signedUrl);
        setData(response);

        if (onSuccess) {
          onSuccess(response);
        }
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : 'An error occurred';
        setError(errorMessage);

        if (onError && err instanceof Error) {
          onError(err);
        }
      } finally {
        setLoading(false);
      }
    };

    if (signedUrl) {
      fetchData();
    }
  }, [signedUrl]);

  if (loading) {
    return (
      <View style={styles.container}>
        <ActivityIndicator size="large" color="#0000ff" />
        <Text style={styles.loadingText}>Loading secure content...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.container}>
        <Text style={styles.errorText}>{error}</Text>
      </View>
    );
  }

  if (data) {
    return (
      <View style={styles.container}>
        <Text style={styles.successText}>Content loaded successfully</Text>
        <Text style={styles.dataText}>{JSON.stringify(data, null, 2)}</Text>
      </View>
    );
  }

  return null;
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorText: {
    fontSize: 16,
    color: '#ff0000',
    textAlign: 'center',
  },
  successText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#00aa00',
    marginBottom: 10,
  },
  dataText: {
    fontSize: 14,
    color: '#333',
    fontFamily: 'monospace',
  },
});
```

### hooks/useAccessToken.ts

```typescript
/**
 * useAccessToken.ts
 *
 * React Native hook for managing access token lifecycle.
 * Handles token generation and resource access with proper state management.
 */

import { useState, useCallback } from 'react';
import { signedUrlClient } from '../services/signedUrlClient';

interface UseAccessTokenResult {
  token: string | null;
  loading: boolean;
  error: string | null;
  generateToken: (resource: string, permissions: string[], authToken: string) => Promise<void>;
  accessResource: <T = any>() => Promise<T | null>;
  resetToken: () => void;
}

/**
 * Hook for managing access tokens and protected resource access.
 *
 * @returns Access token management functions and state
 */
export const useAccessToken = (): UseAccessTokenResult => {
  const [token, setToken] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  /**
   * Generates a new access token for a resource.
   */
  const generateToken = useCallback(
    async (resource: string, permissions: string[], authToken: string) => {
      try {
        setLoading(true);
        setError(null);

        const response = await signedUrlClient.requestAccessToken(
          resource,
          permissions,
          authToken
        );

        setToken(response.access_token);
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : 'Token generation failed';
        setError(errorMessage);
        setToken(null);
      } finally {
        setLoading(false);
      }
    },
    []
  );

  /**
   * Accesses the protected resource using the current token.
   * Token is consumed and cleared after use.
   */
  const accessResource = useCallback(async <T = any>(): Promise<T | null> => {
    if (!token) {
      setError('No token available');
      return null;
    }

    try {
      setLoading(true);
      setError(null);

      const data = await signedUrlClient.accessProtectedResource<T>(token);

      // Clear token after single use
      setToken(null);

      return data;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Resource access failed';
      setError(errorMessage);
      setToken(null);
      return null;
    } finally {
      setLoading(false);
    }
  }, [token]);

  /**
   * Resets the token and error state.
   */
  const resetToken = useCallback(() => {
    setToken(null);
    setError(null);
    setLoading(false);
  }, []);

  return {
    token,
    loading,
    error,
    generateToken,
    accessResource,
    resetToken,
  };
};
```

### Example usage in a screen

```typescript
/**
 * SecureContentScreen.tsx
 *
 * Example screen showing how to use signed URLs and access tokens.
 */

import React from 'react';
import { View, Button, Text, StyleSheet } from 'react-native';
import { SignedUrlHandler } from '../components/SignedUrlHandler';
import { useAccessToken } from '../hooks/useAccessToken';

export const SecureContentScreen: React.FC = () => {
  // Example signed URL from backend
  const signedUrl = 'https://api.example.com/api/secure/export?sig=abc123&data=xyz789';

  const { token, loading, error, generateToken, accessResource } = useAccessToken();

  const handleGenerateToken = async () => {
    // Replace with actual auth token from your auth system
    const authToken = 'user-auth-token';
    await generateToken('sensitive-data', ['read'], authToken);
  };

  const handleAccessResource = async () => {
    const data = await accessResource();
    if (data) {
      console.log('Resource data:', data);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Secure Content Access</Text>

      {/* Signed URL Example */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Signed URL Access:</Text>
        <SignedUrlHandler
          signedUrl={signedUrl}
          onSuccess={(data) => console.log('Signed URL data:', data)}
          onError={(err) => console.error('Signed URL error:', err)}
        />
      </View>

      {/* Access Token Example */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Token-Based Access:</Text>
        <Button
          title="Generate Access Token"
          onPress={handleGenerateToken}
          disabled={loading}
        />
        {token && (
          <View style={styles.tokenInfo}>
            <Text style={styles.tokenText}>Token: {token.substring(0, 20)}...</Text>
            <Button
              title="Access Protected Resource"
              onPress={handleAccessResource}
              disabled={loading}
            />
          </View>
        )}
        {error && <Text style={styles.error}>{error}</Text>}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  section: {
    marginBottom: 30,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 10,
  },
  tokenInfo: {
    marginTop: 10,
    padding: 10,
    backgroundColor: '#f0f0f0',
    borderRadius: 5,
  },
  tokenText: {
    fontSize: 12,
    fontFamily: 'monospace',
    marginBottom: 10,
  },
  error: {
    color: '#ff0000',
    marginTop: 10,
  },
});
```
