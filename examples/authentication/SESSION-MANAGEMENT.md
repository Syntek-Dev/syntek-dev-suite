# Session Management

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Overview

Secure session management implementations across all four technology stacks, including session configuration, multi-device logout, rate limiting, and password reset flows. Each implementation follows security best practices for the respective platform.

## Metadata

| Property               | Value                                                              |
| ---------------------- | ------------------------------------------------------------------ |
| **Example Version**    | 2.0.0                                                              |
| **Last Updated**       | 2025-12                                                            |
| **Stacks**             | TALL (Laravel 12.x), Django 6.x, Next.js 16.x, React Native 0.83.x |
| **Language Standards** | British English                                                    |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [TALL Stack - Laravel 12.x](#tall-stack---laravel-12x)
  - [Secure Session Configuration - Laravel](#secure-session-configuration---laravel)
  - [config/session.php](#configsessionphp)
- [Session Controller - Laravel](#session-controller---laravel)
  - [app/Http/Controllers/SessionController.php](#apphttpcontrollerssessioncontrollerphp)
- [Rate Limiting Middleware - Laravel](#rate-limiting-middleware---laravel)
  - [app/Http/Middleware/ThrottleLogins.php](#apphttpmiddlewarethrottleloginsphp)
- [Password Reset Controller - Laravel](#password-reset-controller---laravel)
  - [app/Http/Controllers/PasswordResetController.php](#apphttpcontrollerspasswordresetcontrollerphp)
- [Django/Wagtail Stack - Django 6.x](#djangowagtail-stack---django-6x)
  - [Session Configuration - Django](#session-configuration---django)
    - [config/settings/base.py](#configsettingsbasepy)
  - [Session Management Views - Django](#session-management-views---django)
    - [apps/accounts/views/session\_views.py](#appsaccountsviewssession_viewspy)
  - [Rate Limiting Middleware - Django](#rate-limiting-middleware---django)
    - [apps/accounts/middleware/rate\_limit.py](#appsaccountsmiddlewarerate_limitpy)
  - [Password Reset Views - Django](#password-reset-views---django)
    - [apps/accounts/views/password\_reset\_views.py](#appsaccountsviewspassword_reset_viewspy)


## TALL Stack - Laravel 12.x

### Secure Session Configuration - Laravel

### config/session.php

```php
<?php

/**
 * Session configuration with security-focused defaults.
 *
 * Ensures sessions are encrypted, HTTP-only, and use secure cookies
 * in accordance with OWASP security recommendations.
 *
 * @package Laravel 12.x
 * @category Security
 * @version 2.0.0
 */

return [
    'lifetime' => 120,                    // 2 hours session lifetime
    'expire_on_close' => false,           // Persist sessions across browser restarts
    'encrypt' => true,                    // Encrypt session data
    'secure' => env('SESSION_SECURE', true),  // HTTPS only in production
    'http_only' => true,                  // Prevent JavaScript access to cookies
    'same_site' => 'lax',                 // CSRF protection via SameSite policy
];
```

---

## Session Controller - Laravel

### app/Http/Controllers/SessionController.php

```php
<?php

/**
 * SessionController.php
 *
 * Handles session management including logout, multi-device logout,
 * and listing active sessions for security monitoring.
 */

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\DB;

class SessionController extends Controller
{
    /**
     * Logs out the current session.
     *
     * Invalidates the session and regenerates the CSRF token to prevent
     * session fixation attacks.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse Logout confirmation
     */
    public function logout(Request $request)
    {
        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return response()->json(['message' => 'Logged out successfully']);
    }

    /**
     * Logs out from all devices except the current one.
     *
     * Invalidates all other browser sessions and revokes all API tokens.
     * Requires password confirmation for security.
     *
     * @param Request $request The HTTP request containing password confirmation
     * @return \Illuminate\Http\JsonResponse Confirmation message
     */
    public function logoutAllDevices(Request $request)
    {
        $request->validate(['password' => 'required|string']);

        Auth::logoutOtherDevices($request->password);

        // Invalidate all tokens (for API auth)
        $request->user()->tokens()->delete();

        return response()->json(['message' => 'Logged out from all other devices']);
    }

    /**
     * Lists all active sessions for the authenticated user.
     *
     * Returns session information including IP address, user agent,
     * and last activity timestamp for security monitoring.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse List of active sessions
     */
    public function activeSessions(Request $request)
    {
        $sessions = DB::table('sessions')
            ->where('user_id', $request->user()->id)
            ->select('id', 'ip_address', 'user_agent', 'last_activity')
            ->get();

        return response()->json(['sessions' => $sessions]);
    }
}
```

---

## Rate Limiting Middleware - Laravel

### app/Http/Middleware/ThrottleLogins.php

```php
<?php

/**
 * ThrottleLogins.php
 *
 * Rate limiting middleware for login attempts. Tracks failed attempts
 * by IP address and blocks further attempts after threshold exceeded.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;

class ThrottleLogins
{
    /**
     * Handles the incoming request with rate limiting.
     *
     * Tracks login attempts by IP address. After 5 failed attempts,
     * blocks further attempts for 15 minutes. Clears the counter
     * on successful authentication.
     *
     * @param Request $request The HTTP request
     * @param Closure $next The next middleware
     * @return mixed The response
     */
    public function handle(Request $request, Closure $next)
    {
        $key = 'login_attempts:' . $request->ip();
        $attempts = Cache::get($key, 0);

        if ($attempts >= 5) {
            $ttl = Cache::get("{$key}:ttl", 300);
            return response()->json([
                'error' => 'Too many login attempts. Try again later.',
                'retry_after' => $ttl,
            ], 429);
        }

        $response = $next($request);

        if ($response->status() === 401) {
            Cache::put($key, $attempts + 1, now()->addMinutes(15));
            Cache::put("{$key}:ttl", 900, now()->addMinutes(15));
        } else {
            Cache::forget($key);
        }

        return $response;
    }
}
```

---

## Password Reset Controller - Laravel

### app/Http/Controllers/PasswordResetController.php

```php
<?php

/**
 * PasswordResetController.php
 *
 * Handles secure password reset flow with token-based verification.
 * Prevents email enumeration by returning consistent responses.
 */

namespace App\Http\Controllers;

use App\Models\User;
use App\Notifications\PasswordResetNotification;
use App\Rules\StrongPassword;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class PasswordResetController extends Controller
{
    /**
     * Sends a password reset link to the provided email.
     *
     * Generates a secure token, stores the hash, and sends the reset email.
     * Always returns a success message to prevent email enumeration attacks.
     *
     * @param Request $request The HTTP request containing email
     * @return \Illuminate\Http\JsonResponse Success message
     */
    public function sendResetLink(Request $request)
    {
        $request->validate(['email' => 'required|email']);

        // Always return success to prevent email enumeration
        $user = User::where('email', $request->email)->first();

        if ($user) {
            $token = Str::random(64);

            DB::table('password_resets')->updateOrInsert(
                ['email' => $user->email],
                [
                    'token' => Hash::make($token),
                    'created_at' => now(),
                ]
            );

            // Send email with token (expires in 1 hour)
            $user->notify(new PasswordResetNotification($token));
        }

        return response()->json([
            'message' => 'If an account exists, a reset link has been sent.',
        ]);
    }

    /**
     * Resets the user password using the provided token.
     *
     * Validates the token, enforces strong password requirements,
     * updates the password, and invalidates all existing sessions.
     *
     * @param Request $request The HTTP request containing token, email, and new password
     * @return \Illuminate\Http\JsonResponse Success or error message
     */
    public function reset(Request $request)
    {
        $request->validate([
            'token' => 'required|string',
            'email' => 'required|email',
            'password' => ['required', 'confirmed', new StrongPassword()],
        ]);

        $record = DB::table('password_resets')
            ->where('email', $request->email)
            ->first();

        if (!$record || !Hash::check($request->token, $record->token)) {
            return response()->json(['error' => 'Invalid or expired token'], 400);
        }

        if (Carbon::parse($record->created_at)->addHour()->isPast()) {
            return response()->json(['error' => 'Token has expired'], 400);
        }

        $user = User::where('email', $request->email)->firstOrFail();
        $user->password = Hash::make($request->password);
        $user->save();

        // Invalidate all sessions
        Auth::logoutOtherDevices($request->password);

        DB::table('password_resets')->where('email', $request->email)->delete();

        return response()->json(['message' => 'Password reset successfully']);
    }
}
```

---

## Django/Wagtail Stack - Django 6.x

### Session Configuration - Django

#### config/settings/base.py

```python
"""
Session configuration for Django 6.x with security-focused defaults.

Configures django.contrib.sessions middleware with secure cookie settings,
database-backed session storage, and appropriate expiration policies.

Package: Django 6.x
Category: Security
Version: 2.0.0
"""

# Session configuration
SESSION_ENGINE = 'django.contrib.sessions.backends.db'  # Database-backed sessions
SESSION_COOKIE_SECURE = True  # HTTPS only
SESSION_COOKIE_HTTPONLY = True  # Prevent JavaScript access
SESSION_COOKIE_SAMESITE = 'Lax'  # CSRF protection
SESSION_COOKIE_AGE = 7200  # 2 hours in seconds
SESSION_SAVE_EVERY_REQUEST = False  # Only save on modification
SESSION_EXPIRE_AT_BROWSER_CLOSE = False  # Persist across browser restarts

# Security settings
CSRF_COOKIE_SECURE = True  # HTTPS only for CSRF tokens
CSRF_COOKIE_HTTPONLY = True  # Prevent JavaScript access to CSRF token
CSRF_COOKIE_SAMESITE = 'Lax'  # CSRF protection

# Session cleanup (run via cron: python manage.py clearsessions)
SESSION_COOKIE_NAME = 'sessionid'
```

---

### Session Management Views - Django

#### apps/accounts/views/session_views.py

```python
"""
Session management views for Django 6.x.

Handles session logout, multi-device logout, and listing active sessions
for security monitoring purposes.

Package: Django 6.x
Category: Authentication
Version: 2.0.0
"""

from django.contrib.auth import logout
from django.contrib.auth.decorators import login_required
from django.contrib.sessions.models import Session
from django.http import JsonResponse
from django.utils import timezone
from django.views.decorators.http import require_http_methods
from django.views.decorators.csrf import csrf_protect
import json


@login_required
@require_http_methods(["POST"])
@csrf_protect
def logout_view(request):
    """
    Logs out the current session.

    Invalidates the current session and clears authentication data
    to prevent session fixation attacks.

    Args:
        request: The HTTP request object

    Returns:
        JsonResponse: Logout confirmation message
    """
    logout(request)
    return JsonResponse({'message': 'Logged out successfully'})


@login_required
@require_http_methods(["POST"])
@csrf_protect
def logout_all_devices(request):
    """
    Logs out from all devices except the current one.

    Invalidates all other sessions for the authenticated user.
    Requires password confirmation for security.

    Args:
        request: The HTTP request containing password confirmation

    Returns:
        JsonResponse: Confirmation message or error
    """
    try:
        data = json.loads(request.body)
        password = data.get('password')

        if not password:
            return JsonResponse(
                {'error': 'Password is required'},
                status=400
            )

        # Verify password
        if not request.user.check_password(password):
            return JsonResponse(
                {'error': 'Invalid password'},
                status=403
            )

        # Get current session key to preserve it
        current_session_key = request.session.session_key

        # Delete all other sessions for this user
        user_sessions = Session.objects.filter(
            expire_date__gte=timezone.now()
        )

        deleted_count = 0
        for session in user_sessions:
            data = session.get_decoded()
            if data.get('_auth_user_id') == str(request.user.id):
                if session.session_key != current_session_key:
                    session.delete()
                    deleted_count += 1

        return JsonResponse({
            'message': f'Logged out from {deleted_count} other device(s)'
        })

    except json.JSONDecodeError:
        return JsonResponse(
            {'error': 'Invalid JSON data'},
            status=400
        )


@login_required
@require_http_methods(["GET"])
def active_sessions(request):
    """
    Lists all active sessions for the authenticated user.

    Returns session information including last activity timestamp
    for security monitoring purposes.

    Args:
        request: The HTTP request object

    Returns:
        JsonResponse: List of active sessions
    """
    user_sessions = []
    current_session_key = request.session.session_key

    # Find all sessions for this user
    all_sessions = Session.objects.filter(
        expire_date__gte=timezone.now()
    )

    for session in all_sessions:
        data = session.get_decoded()
        if data.get('_auth_user_id') == str(request.user.id):
            user_sessions.append({
                'session_key': session.session_key[:8] + '...',  # Truncated for security
                'is_current': session.session_key == current_session_key,
                'expire_date': session.expire_date.isoformat(),
            })

    return JsonResponse({
        'sessions': user_sessions,
        'total': len(user_sessions)
    })
```

---

### Rate Limiting Middleware - Django

#### apps/accounts/middleware/rate_limit.py

```python
"""
Rate limiting middleware for login attempts in Django 6.x.

Tracks failed login attempts by IP address and blocks further attempts
after threshold exceeded using Django's cache framework.

Package: Django 6.x
Category: Security
Version: 2.0.0
"""

from django.core.cache import cache
from django.http import JsonResponse
from django.utils.deprecation import MiddlewareMixin
import time


class ThrottleLoginMiddleware(MiddlewareMixin):
    """
    Rate limiting middleware for login attempts.

    Tracks login attempts by IP address. After 5 failed attempts,
    blocks further attempts for 15 minutes. Clears the counter
    on successful authentication.
    """

    MAX_ATTEMPTS = 5
    LOCKOUT_DURATION = 900  # 15 minutes in seconds

    def process_request(self, request):
        """
        Handles the incoming request with rate limiting.

        Only applies to login endpoints. Checks the number of failed
        attempts and returns 429 Too Many Requests if threshold exceeded.

        Args:
            request: The HTTP request object

        Returns:
            JsonResponse: Error response if rate limit exceeded, None otherwise
        """
        # Only apply to login endpoint
        if request.path != '/api/auth/login/' or request.method != 'POST':
            return None

        ip_address = self.get_client_ip(request)
        cache_key = f'login_attempts:{ip_address}'

        attempts = cache.get(cache_key, 0)

        if attempts >= self.MAX_ATTEMPTS:
            ttl = cache.ttl(cache_key)
            return JsonResponse({
                'error': 'Too many login attempts. Try again later.',
                'retry_after': ttl if ttl else self.LOCKOUT_DURATION,
            }, status=429)

        return None

    def process_response(self, request, response):
        """
        Processes the response to track failed login attempts.

        Increments the failed attempt counter on 401 responses,
        clears it on successful authentication (200/201).

        Args:
            request: The HTTP request object
            response: The HTTP response object

        Returns:
            HttpResponse: The original response
        """
        # Only apply to login endpoint
        if request.path != '/api/auth/login/' or request.method != 'POST':
            return response

        ip_address = self.get_client_ip(request)
        cache_key = f'login_attempts:{ip_address}'

        if response.status_code == 401:
            # Increment failed attempts
            attempts = cache.get(cache_key, 0)
            cache.set(cache_key, attempts + 1, self.LOCKOUT_DURATION)
        elif response.status_code in [200, 201]:
            # Clear failed attempts on successful login
            cache.delete(cache_key)

        return response

    def get_client_ip(self, request):
        """
        Extracts the client IP address from the request.

        Checks X-Forwarded-For header for proxied requests,
        falls back to REMOTE_ADDR.

        Args:
            request: The HTTP request object

        Returns:
            str: The client IP address
        """
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0].strip()
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

---

### Password Reset Views - Django

#### apps/accounts/views/password_reset_views.py

```python
"""
Password reset views for Django 6.x.

Handles secure password reset flow with token-based verification
using Django's built-in password reset functionality.

Package: Django 6.x
Category: Authentication
Version: 2.0.0
"""

from django.contrib.auth import get_user_model
from django.contrib.auth.tokens import default_token_generator
from django.contrib.sites.shortcuts import get_current_site
from django.core.mail import send_mail
from django.http import JsonResponse
from django.template.loader import render_to_string
from django.utils.encoding import force_bytes, force_str
from django.utils.http import urlsafe_base64_encode, urlsafe_base64_decode
from django.views.decorators.http import require_http_methods
from django.views.decorators.csrf import csrf_protect
from django.conf import settings
import json

User = get_user_model()


@require_http_methods(["POST"])
@csrf_protect
def send_reset_link(request):
    """
    Sends a password reset link to the provided email.

    Generates a secure token using Django's default token generator,
    and sends the reset email. Always returns a success message to
    prevent email enumeration attacks.

    Args:
        request: The HTTP request containing email

    Returns:
        JsonResponse: Success message (always, for security)
    """
    try:
        data = json.loads(request.body)
        email = data.get('email')

        if not email:
            return JsonResponse(
                {'error': 'Email is required'},
                status=400
            )

        # Always return success to prevent email enumeration
        user = User.objects.filter(email=email).first()

        if user:
            # Generate token
            token = default_token_generator.make_token(user)
            uid = urlsafe_base64_encode(force_bytes(user.pk))

            # Build reset URL
            current_site = get_current_site(request)
            reset_url = f"{request.scheme}://{current_site.domain}/reset-password/{uid}/{token}/"

            # Send email
            subject = 'Password Reset Request'
            message = render_to_string('accounts/password_reset_email.html', {
                'user': user,
                'reset_url': reset_url,
                'site_name': current_site.name,
            })

            send_mail(
                subject,
                message,
                settings.DEFAULT_FROM_EMAIL,
                [user.email],
                fail_silently=False,
            )

        return JsonResponse({
            'message': 'If an account exists, a reset link has been sent.'
        })

    except json.JSONDecodeError:
        return JsonResponse(
            {'error': 'Invalid JSON data'},
            status=400
        )


@require_http_methods(["POST"])
@csrf_protect
def reset_password(request):
    """
    Resets the user password using the provided token.

    Validates the token using Django's token generator, enforces
    password validation rules, updates the password, and invalidates
    all existing sessions.

    Args:
        request: The HTTP request containing uidb64, token, and new password

    Returns:
        JsonResponse: Success or error message
    """
    try:
        data = json.loads(request.body)
        uidb64 = data.get('uidb64')
        token = data.get('token')
        password = data.get('password')
        password_confirm = data.get('password_confirm')

        if not all([uidb64, token, password, password_confirm]):
            return JsonResponse(
                {'error': 'All fields are required'},
                status=400
            )

        if password != password_confirm:
            return JsonResponse(
                {'error': 'Passwords do not match'},
                status=400
            )

        # Decode user ID
        try:
            uid = force_str(urlsafe_base64_decode(uidb64))
            user = User.objects.get(pk=uid)
        except (TypeError, ValueError, OverflowError, User.DoesNotExist):
            return JsonResponse(
                {'error': 'Invalid or expired token'},
                status=400
            )

        # Validate token
        if not default_token_generator.check_token(user, token):
            return JsonResponse(
                {'error': 'Invalid or expired token'},
                status=400
            )

        # Validate password strength (uses Django's password validators)
        from django.contrib.auth.password_validation import validate_password
        from django.core.exceptions import ValidationError

        try:
            validate_password(password, user)
        except ValidationError as e:
            return JsonResponse(
                {'error': list(e.messages)},
                status=400
            )

        # Set new password
        user.set_password(password)
        user.save()

        # Invalidate all sessions for this user
        from django.contrib.sessions.models import Session
        from django.utils import timezone

        for session in Session.objects.filter(expire_date__gte=timezone.now()):
            data = session.get_decoded()
            if data.get('_auth_user_id') == str(user.id):
                session.delete()

        return JsonResponse({
            'message': 'Password reset successfully. Please log in with your new password.'
        })

    except json.JSONDecodeError:
        return JsonResponse(
            {'error': 'Invalid JSON data'},
            status=400
        )
