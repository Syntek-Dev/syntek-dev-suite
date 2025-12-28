# IP Address Security Service

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Overview

Secure IP address capture and storage for authentication events. IP addresses are considered PII under GDPR and must be protected with hashing (for lookups) and encryption (for audit display).

## Metadata

| Property            | Value                               |
| ------------------- | ----------------------------------- |
| **Example Version** | 2.0.0                               |
| **Last Updated**    | 2025-12                             |
| **Laravel**         | 12.x                                |
| **Django**          | 6.x                                 |
| **Next.js**         | 16.x                                |
| **React Native**    | 0.83.x                              |
| **Stacks**          | TALL, Django, React/Next.js, Mobile |

---


## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [IP Address Storage Strategy](#ip-address-storage-strategy)
- [IP Security Service - Laravel](#ip-security-service---laravel)
  - [app/Services/IpSecurityService.php](#appservicesipsecurityservicephp)
- [Auth Security Log Migration - Laravel](#auth-security-log-migration---laravel)
  - [database/migrations/xxxx\_create\_auth\_security\_logs\_table.php](#databasemigrationsxxxx_create_auth_security_logs_tablephp)
- [Login Controller Integration - Laravel](#login-controller-integration---laravel)
  - [app/Http/Controllers/LoginController.php](#apphttpcontrollerslogincontrollerphp)
- [IP Security Service - Django](#ip-security-service---django)
  - [apps/authentication/services/ip\_security.py](#appsauthenticationservicesip_securitypy)
- [Auth Security Log Model - Django](#auth-security-log-model---django)
  - [apps/authentication/models/auth\_security\_log.py](#appsauthenticationmodelsauth_security_logpy)
- [Login View Integration - Django](#login-view-integration---django)
  - [apps/authentication/views/login\_view.py](#appsauthenticationviewslogin_viewpy)
- [IP Security Service - Next.js](#ip-security-service---nextjs)
  - [lib/services/ip-security.ts](#libservicesip-securityts)
- [Auth Security Log Schema - Prisma](#auth-security-log-schema---prisma)
  - [prisma/schema.prisma (partial)](#prismaschemaprisma-partial)
- [Login API Route - Next.js](#login-api-route---nextjs)
  - [app/api/auth/login/route.ts](#appapiauthloginroutets)
- [IP Security Hook - React Native](#ip-security-hook---react-native)
  - [src/hooks/useIpSecurity.ts](#srchooksuseipsecurityts)
- [GraphQL Mutation - React Native](#graphql-mutation---react-native)
  - [src/graphql/mutations/auth.ts](#srcgraphqlmutationsauthts)
  - [src/screens/LoginScreen.tsx (Integration Example)](#srcscreensloginscreentsx-integration-example)


## IP Address Storage Strategy

| Purpose          | Storage Method        | Retention  |
| ---------------- | --------------------- | ---------- |
| Rate limiting    | Hashed (HMAC)         | 24 hours   |
| Login audit logs | Encrypted (AES-256)   | 90 days    |
| Security alerts  | Encrypted (AES-256)   | 1 year     |
| Analytics        | Hashed (irreversible) | Indefinite |

---

## IP Security Service - Laravel

### app/Services/IpSecurityService.php

```php
<?php

/**
 * IpSecurityService.php
 *
 * Handles secure capture and storage of IP addresses for authentication events.
 * IP addresses are considered PII under GDPR and must be protected accordingly.
 *
 * @package App\Services
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Services;

use Illuminate\Support\Facades\Crypt;
use Illuminate\Support\Facades\DB;

class IpSecurityService
{
    /**
     * Captures and stores IP address for an authentication event.
     *
     * Stores both hashed (for lookup) and encrypted (for display) versions
     * of the IP address along with event metadata.
     *
     * @param string $ip The IP address to capture
     * @param string $eventType The type of auth event (login, logout, failed_login, etc.)
     * @param int|null $userId The user ID if authenticated
     * @return array The stored IP data
     */
    public function captureAuthEvent(string $ip, string $eventType, ?int $userId = null): array
    {
        $ipData = [
            'ip_hash' => $this->hashIp($ip),
            'ip_encrypted' => $this->encryptIp($ip),
            'event_type' => $eventType,
            'user_id' => $userId,
            'user_agent' => request()->userAgent(),
            'created_at' => now(),
        ];

        DB::table('auth_security_logs')->insert($ipData);

        return $ipData;
    }

    /**
     * Generates an irreversible hash of the IP address for lookup/analytics.
     *
     * The hash cannot be reversed to obtain the original IP. Uses HMAC
     * with the application key for consistent hashing.
     *
     * @param string $ip The IP address to hash
     * @return string The hashed IP (64 characters)
     */
    public function hashIp(string $ip): string
    {
        return hash_hmac('sha256', $ip, config('app.key'));
    }

    /**
     * Encrypts the IP address for secure storage with later retrieval.
     *
     * Only users with appropriate permissions can decrypt. Uses Laravel's
     * built-in encryption (AES-256-CBC).
     *
     * @param string $ip The IP address to encrypt
     * @return string The encrypted IP
     */
    public function encryptIp(string $ip): string
    {
        return Crypt::encryptString($ip);
    }

    /**
     * Decrypts a previously encrypted IP address.
     *
     * Requires pii.access permission to call. Returns null on decryption
     * failure rather than throwing an exception.
     *
     * @param string $encryptedIp The encrypted IP address
     * @return string|null The decrypted IP or null on failure
     */
    public function decryptIp(string $encryptedIp): ?string
    {
        try {
            return Crypt::decryptString($encryptedIp);
        } catch (\Exception $e) {
            logger()->error('IP decryption failed', ['error' => $e->getMessage()]);
            return null;
        }
    }

    /**
     * Checks if an IP has suspicious activity based on hashed lookups.
     *
     * Queries the auth security logs for failed login attempts in the
     * past 24 hours. Returns true if 10 or more failures detected.
     *
     * @param string $ip The IP address to check
     * @return bool True if suspicious activity detected
     */
    public function isSuspiciousIp(string $ip): bool
    {
        $hash = $this->hashIp($ip);

        $failedAttempts = DB::table('auth_security_logs')
            ->where('ip_hash', $hash)
            ->where('event_type', 'failed_login')
            ->where('created_at', '>=', now()->subHours(24))
            ->count();

        return $failedAttempts >= 10;
    }
}
```

---

## Auth Security Log Migration - Laravel

### database/migrations/xxxx_create_auth_security_logs_table.php

```php
<?php

/**
 * Creates the auth_security_logs table for tracking authentication events.
 * IP addresses are stored both hashed (for lookup) and encrypted (for audit).
 *
 * @version Laravel 12.x / MariaDB 12.x
 */

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Creates the auth_security_logs table.
     *
     * Stores authentication events with PII-protected IP addresses.
     * IP addresses are hashed for lookups and encrypted for audit display.
     */
    public function up(): void
    {
        Schema::create('auth_security_logs', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->nullable()->constrained()->onDelete('set null');

            // IP address storage (PII protected)
            $table->string('ip_hash', 64)->index();      // For lookups
            $table->text('ip_encrypted');                 // For audit display

            // Event details
            $table->string('event_type', 50)->index();   // login, logout, failed_login, password_reset, mfa_enabled
            $table->string('user_agent', 500)->nullable();
            $table->json('metadata')->nullable();        // Additional event data

            $table->timestamp('created_at')->useCurrent();

            // Composite index for security queries
            $table->index(['ip_hash', 'event_type', 'created_at']);
        });
    }

    /**
     * Drops the auth_security_logs table.
     */
    public function down(): void
    {
        Schema::dropIfExists('auth_security_logs');
    }
};
```

---

## Login Controller Integration - Laravel

### app/Http/Controllers/LoginController.php

```php
<?php

/**
 * LoginController.php
 *
 * Handles authentication with IP security logging. Checks for suspicious
 * IP activity before allowing login attempts and logs all auth events.
 *
 * @package App\Http\Controllers
 * @version Laravel 12.x / PHP 8.4
 */

namespace App\Http\Controllers;

use App\Services\IpSecurityService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Creates a new LoginController instance.
     *
     * @param IpSecurityService $ipSecurity The IP security service
     */
    public function __construct(
        protected IpSecurityService $ipSecurity
    ) {}

    /**
     * Handles user login with security checks.
     *
     * Checks for suspicious IP activity before attempting authentication.
     * Logs all login attempts (successful and failed) with IP address.
     *
     * @param Request $request The HTTP request containing credentials
     * @return \Illuminate\Http\JsonResponse Login result
     */
    public function login(Request $request)
    {
        $ip = $request->ip();

        // Check for suspicious IP before attempting login
        if ($this->ipSecurity->isSuspiciousIp($ip)) {
            $this->ipSecurity->captureAuthEvent($ip, 'blocked_suspicious', null);

            return response()->json([
                'error' => 'Access temporarily restricted. Contact support if this persists.',
            ], 403);
        }

        // Attempt authentication
        if (Auth::attempt($request->only('email', 'password'))) {
            $user = Auth::user();

            // Log successful login with encrypted IP
            $this->ipSecurity->captureAuthEvent($ip, 'login', $user->id);

            return response()->json(['message' => 'Login successful']);
        }

        // Log failed login attempt
        $this->ipSecurity->captureAuthEvent($ip, 'failed_login', null);

        return response()->json(['error' => 'Invalid credentials'], 401);
    }

    /**
     * Handles user logout.
     *
     * Logs the logout event with IP address and invalidates the session.
     *
     * @param Request $request The HTTP request
     * @return \Illuminate\Http\JsonResponse Logout confirmation
     */
    public function logout(Request $request)
    {
        $this->ipSecurity->captureAuthEvent(
            $request->ip(),
            'logout',
            $request->user()->id
        );

        Auth::logout();
        $request->session()->invalidate();

        return response()->json(['message' => 'Logged out']);
    }
}
```

---

## IP Security Service - Django

### apps/authentication/services/ip_security.py

```python
"""
ip_security.py

Handles secure capture and storage of IP addresses for authentication events.
IP addresses are considered PII under GDPR and must be protected accordingly.

Version: Django 6.x / Python 3.14
"""

import hashlib
import hmac
import logging
from datetime import timedelta
from typing import Optional

from cryptography.fernet import Fernet
from django.conf import settings
from django.utils import timezone

from apps.authentication.models import AuthSecurityLog

logger = logging.getLogger(__name__)


class IpSecurityService:
    """
    Service for secure IP address handling in authentication events.

    Provides methods for hashing (irreversible lookup), encrypting
    (reversible audit), and suspicious activity detection.
    """

    def __init__(self) -> None:
        """Initialises the IP security service with encryption key."""
        self._fernet = Fernet(settings.ENCRYPTION_KEY)

    def capture_auth_event(
        self,
        ip: str,
        event_type: str,
        user_id: Optional[int] = None,
        user_agent: Optional[str] = None,
        metadata: Optional[dict] = None,
    ) -> AuthSecurityLog:
        """
        Captures and stores IP address for an authentication event.

        Stores both hashed (for lookup) and encrypted (for display) versions
        of the IP address along with event metadata.

        Args:
            ip: The IP address to capture
            event_type: The type of auth event (login, logout, failed_login, etc.)
            user_id: The user ID if authenticated
            user_agent: The user agent string from the request
            metadata: Additional event data as a dictionary

        Returns:
            AuthSecurityLog: The created log entry
        """
        return AuthSecurityLog.objects.create(
            ip_hash=self.hash_ip(ip),
            ip_encrypted=self.encrypt_ip(ip),
            event_type=event_type,
            user_id=user_id,
            user_agent=user_agent or "",
            metadata=metadata or {},
        )

    def hash_ip(self, ip: str) -> str:
        """
        Generates an irreversible hash of the IP address for lookup/analytics.

        The hash cannot be reversed to obtain the original IP. Uses HMAC
        with the secret key for consistent hashing.

        Args:
            ip: The IP address to hash

        Returns:
            str: The hashed IP (64 characters)
        """
        return hmac.new(
            settings.SECRET_KEY.encode(),
            ip.encode(),
            hashlib.sha256,
        ).hexdigest()

    def encrypt_ip(self, ip: str) -> str:
        """
        Encrypts the IP address for secure storage with later retrieval.

        Only users with appropriate permissions can decrypt. Uses Fernet
        symmetric encryption (AES-128-CBC with HMAC).

        Args:
            ip: The IP address to encrypt

        Returns:
            str: The encrypted IP as a base64 string
        """
        return self._fernet.encrypt(ip.encode()).decode()

    def decrypt_ip(self, encrypted_ip: str) -> Optional[str]:
        """
        Decrypts a previously encrypted IP address.

        Requires pii.access permission to call. Returns None on decryption
        failure rather than raising an exception.

        Args:
            encrypted_ip: The encrypted IP address

        Returns:
            Optional[str]: The decrypted IP or None on failure
        """
        try:
            return self._fernet.decrypt(encrypted_ip.encode()).decode()
        except Exception as e:
            logger.error("IP decryption failed: %s", str(e))
            return None

    def is_suspicious_ip(self, ip: str) -> bool:
        """
        Checks if an IP has suspicious activity based on hashed lookups.

        Queries the auth security logs for failed login attempts in the
        past 24 hours. Returns True if 10 or more failures detected.

        Args:
            ip: The IP address to check

        Returns:
            bool: True if suspicious activity detected
        """
        ip_hash = self.hash_ip(ip)
        threshold = timezone.now() - timedelta(hours=24)

        failed_attempts = AuthSecurityLog.objects.filter(
            ip_hash=ip_hash,
            event_type="failed_login",
            created_at__gte=threshold,
        ).count()

        return failed_attempts >= 10
```

---

## Auth Security Log Model - Django

### apps/authentication/models/auth_security_log.py

```python
"""
auth_security_log.py

Model for tracking authentication events with PII-protected IP addresses.
IP addresses are stored both hashed (for lookup) and encrypted (for audit).

Version: Django 6.x / PostgreSQL 18.x
"""

from django.contrib.auth import get_user_model
from django.db import models

User = get_user_model()


class AuthSecurityLog(models.Model):
    """
    Stores authentication events with secure IP address handling.

    Attributes:
        user: The user associated with the event (nullable for failed logins)
        ip_hash: HMAC-SHA256 hash of the IP for lookups (irreversible)
        ip_encrypted: Fernet-encrypted IP for audit display (reversible)
        event_type: Type of authentication event
        user_agent: Browser/client user agent string
        metadata: Additional event data as JSON
        created_at: Timestamp of the event
    """

    class EventType(models.TextChoices):
        """Enumeration of authentication event types."""

        LOGIN = "login", "Login"
        LOGOUT = "logout", "Logout"
        FAILED_LOGIN = "failed_login", "Failed Login"
        PASSWORD_RESET = "password_reset", "Password Reset"
        MFA_ENABLED = "mfa_enabled", "MFA Enabled"
        MFA_DISABLED = "mfa_disabled", "MFA Disabled"
        BLOCKED_SUSPICIOUS = "blocked_suspicious", "Blocked Suspicious"

    user = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name="security_logs",
    )
    ip_hash = models.CharField(max_length=64, db_index=True)
    ip_encrypted = models.TextField()
    event_type = models.CharField(
        max_length=50,
        choices=EventType.choices,
        db_index=True,
    )
    user_agent = models.CharField(max_length=500, blank=True)
    metadata = models.JSONField(default=dict, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        """Model metadata configuration."""

        db_table = "auth_security_logs"
        ordering = ["-created_at"]
        indexes = [
            models.Index(
                fields=["ip_hash", "event_type", "created_at"],
                name="ip_event_created_idx",
            ),
        ]

    def __str__(self) -> str:
        """Returns string representation of the log entry."""
        return f"{self.event_type} - {self.ip_hash[:16]}... - {self.created_at}"
```

---

## Login View Integration - Django

### apps/authentication/views/login_view.py

```python
"""
login_view.py

Handles authentication with IP security logging using Strawberry GraphQL.
Checks for suspicious IP activity before allowing login attempts.

Version: Django 6.x / Strawberry GraphQL 0.260+
"""

import strawberry
from django.contrib.auth import authenticate, login, logout
from strawberry.types import Info

from apps.authentication.services.ip_security import IpSecurityService


def get_client_ip(request) -> str:
    """
    Extracts the client IP address from the request.

    Handles X-Forwarded-For header for proxied requests.

    Args:
        request: The Django HTTP request

    Returns:
        str: The client IP address
    """
    x_forwarded_for = request.META.get("HTTP_X_FORWARDED_FOR")
    if x_forwarded_for:
        return x_forwarded_for.split(",")[0].strip()
    return request.META.get("REMOTE_ADDR", "")


@strawberry.type
class AuthResult:
    """Result type for authentication operations."""

    success: bool
    message: str


@strawberry.type
class AuthMutation:
    """GraphQL mutations for authentication operations."""

    @strawberry.mutation
    def login(self, info: Info, email: str, password: str) -> AuthResult:
        """
        Authenticates a user with IP security checks.

        Checks for suspicious IP activity before attempting authentication.
        Logs all login attempts (successful and failed) with IP address.

        Args:
            info: Strawberry request info
            email: User email address
            password: User password

        Returns:
            AuthResult: Success status and message
        """
        request = info.context.request
        ip = get_client_ip(request)
        user_agent = request.META.get("HTTP_USER_AGENT", "")
        ip_security = IpSecurityService()

        # Check for suspicious IP before attempting login
        if ip_security.is_suspicious_ip(ip):
            ip_security.capture_auth_event(
                ip=ip,
                event_type="blocked_suspicious",
                user_agent=user_agent,
            )
            return AuthResult(
                success=False,
                message="Access temporarily restricted. Contact support if this persists.",
            )

        # Attempt authentication
        user = authenticate(request, username=email, password=password)

        if user is not None:
            login(request, user)
            ip_security.capture_auth_event(
                ip=ip,
                event_type="login",
                user_id=user.id,
                user_agent=user_agent,
            )
            return AuthResult(success=True, message="Login successful")

        # Log failed login attempt
        ip_security.capture_auth_event(
            ip=ip,
            event_type="failed_login",
            user_agent=user_agent,
        )
        return AuthResult(success=False, message="Invalid credentials")

    @strawberry.mutation
    def logout_user(self, info: Info) -> AuthResult:
        """
        Logs out the current user.

        Logs the logout event with IP address and terminates the session.

        Args:
            info: Strawberry request info

        Returns:
            AuthResult: Success status and message
        """
        request = info.context.request
        ip = get_client_ip(request)
        user_agent = request.META.get("HTTP_USER_AGENT", "")
        ip_security = IpSecurityService()

        if request.user.is_authenticated:
            ip_security.capture_auth_event(
                ip=ip,
                event_type="logout",
                user_id=request.user.id,
                user_agent=user_agent,
            )
            logout(request)
            return AuthResult(success=True, message="Logged out successfully")

        return AuthResult(success=False, message="Not authenticated")
```

---

## IP Security Service - Next.js

### lib/services/ip-security.ts

```typescript
/**
 * ip-security.ts
 *
 * Handles secure capture and storage of IP addresses for authentication events.
 * IP addresses are considered PII under GDPR and must be protected accordingly.
 *
 * @version Next.js 16.x / TypeScript 5.9 / Node.js 24.x
 */

import { createHmac, createCipheriv, createDecipheriv, randomBytes } from 'crypto';
import { prisma } from '@/lib/prisma';

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY!;
const SECRET_KEY = process.env.SECRET_KEY!;

interface CaptureEventParams {
  ip: string;
  eventType: string;
  userId?: string | null;
  userAgent?: string | null;
  metadata?: Record<string, unknown>;
}

interface IpData {
  ipHash: string;
  ipEncrypted: string;
  eventType: string;
  userId: string | null;
  userAgent: string | null;
  createdAt: Date;
}

/**
 * Captures and stores IP address for an authentication event.
 *
 * Stores both hashed (for lookup) and encrypted (for display) versions
 * of the IP address along with event metadata.
 *
 * @param params - The event parameters including IP, type, and optional user data
 * @returns The stored IP data record
 */
export async function captureAuthEvent(params: CaptureEventParams): Promise<IpData> {
  const { ip, eventType, userId = null, userAgent = null, metadata = {} } = params;

  const ipData = {
    ipHash: hashIp(ip),
    ipEncrypted: encryptIp(ip),
    eventType,
    userId,
    userAgent,
    metadata,
    createdAt: new Date(),
  };

  await prisma.authSecurityLog.create({
    data: ipData,
  });

  return ipData;
}

/**
 * Generates an irreversible hash of the IP address for lookup/analytics.
 *
 * The hash cannot be reversed to obtain the original IP. Uses HMAC
 * with the secret key for consistent hashing.
 *
 * @param ip - The IP address to hash
 * @returns The hashed IP (64 characters)
 */
export function hashIp(ip: string): string {
  return createHmac('sha256', SECRET_KEY).update(ip).digest('hex');
}

/**
 * Encrypts the IP address for secure storage with later retrieval.
 *
 * Only users with appropriate permissions can decrypt. Uses AES-256-GCM
 * for authenticated encryption.
 *
 * @param ip - The IP address to encrypt
 * @returns The encrypted IP with IV prepended (base64)
 */
export function encryptIp(ip: string): string {
  const iv = randomBytes(16);
  const key = Buffer.from(ENCRYPTION_KEY, 'hex');
  const cipher = createCipheriv('aes-256-gcm', key, iv);

  let encrypted = cipher.update(ip, 'utf8', 'base64');
  encrypted += cipher.final('base64');
  const authTag = cipher.getAuthTag();

  // Combine IV + authTag + encrypted data
  return Buffer.concat([iv, authTag, Buffer.from(encrypted, 'base64')]).toString('base64');
}

/**
 * Decrypts a previously encrypted IP address.
 *
 * Requires pii.access permission to call. Returns null on decryption
 * failure rather than throwing an exception.
 *
 * @param encryptedIp - The encrypted IP address
 * @returns The decrypted IP or null on failure
 */
export function decryptIp(encryptedIp: string): string | null {
  try {
    const buffer = Buffer.from(encryptedIp, 'base64');
    const iv = buffer.subarray(0, 16);
    const authTag = buffer.subarray(16, 32);
    const encrypted = buffer.subarray(32);

    const key = Buffer.from(ENCRYPTION_KEY, 'hex');
    const decipher = createDecipheriv('aes-256-gcm', key, iv);
    decipher.setAuthTag(authTag);

    let decrypted = decipher.update(encrypted);
    decrypted = Buffer.concat([decrypted, decipher.final()]);

    return decrypted.toString('utf8');
  } catch (error) {
    console.error('IP decryption failed:', error);
    return null;
  }
}

/**
 * Checks if an IP has suspicious activity based on hashed lookups.
 *
 * Queries the auth security logs for failed login attempts in the
 * past 24 hours. Returns true if 10 or more failures detected.
 *
 * @param ip - The IP address to check
 * @returns True if suspicious activity detected
 */
export async function isSuspiciousIp(ip: string): Promise<boolean> {
  const ipHash = hashIp(ip);
  const twentyFourHoursAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);

  const failedAttempts = await prisma.authSecurityLog.count({
    where: {
      ipHash,
      eventType: 'failed_login',
      createdAt: {
        gte: twentyFourHoursAgo,
      },
    },
  });

  return failedAttempts >= 10;
}

/**
 * Extracts client IP from Next.js request headers.
 *
 * Handles X-Forwarded-For header for proxied requests and
 * falls back to connection remote address.
 *
 * @param headers - The request headers
 * @returns The client IP address
 */
export function getClientIp(headers: Headers): string {
  const xForwardedFor = headers.get('x-forwarded-for');
  if (xForwardedFor) {
    return xForwardedFor.split(',')[0].trim();
  }

  const xRealIp = headers.get('x-real-ip');
  if (xRealIp) {
    return xRealIp;
  }

  return '127.0.0.1';
}
```

---

## Auth Security Log Schema - Prisma

### prisma/schema.prisma (partial)

```prisma
// Auth Security Log Model
// Stores authentication events with PII-protected IP addresses.
// IP addresses are stored both hashed (for lookup) and encrypted (for audit).
//
// Version: Prisma 6.x / PostgreSQL 18.x

model AuthSecurityLog {
  id          String   @id @default(cuid())
  userId      String?  @map("user_id")
  user        User?    @relation(fields: [userId], references: [id], onDelete: SetNull)

  // IP address storage (PII protected)
  ipHash      String   @map("ip_hash") @db.VarChar(64)
  ipEncrypted String   @map("ip_encrypted") @db.Text

  // Event details
  eventType   String   @map("event_type") @db.VarChar(50)
  userAgent   String?  @map("user_agent") @db.VarChar(500)
  metadata    Json     @default("{}")

  createdAt   DateTime @default(now()) @map("created_at")

  @@index([ipHash])
  @@index([eventType])
  @@index([ipHash, eventType, createdAt])
  @@map("auth_security_logs")
}
```

---

## Login API Route - Next.js

### app/api/auth/login/route.ts

```typescript
/**
 * route.ts
 *
 * Login API route with IP security logging. Checks for suspicious
 * IP activity before allowing login attempts and logs all auth events.
 *
 * @version Next.js 16.x / TypeScript 5.9 / React 19.x
 */

import { NextRequest, NextResponse } from 'next/server';
import { signIn } from '@/lib/auth';
import {
  captureAuthEvent,
  isSuspiciousIp,
  getClientIp,
} from '@/lib/services/ip-security';

interface LoginRequest {
  email: string;
  password: string;
}

interface LoginResponse {
  success: boolean;
  message: string;
  user?: {
    id: string;
    email: string;
  };
}

/**
 * Handles POST requests for user login.
 *
 * Checks for suspicious IP activity before attempting authentication.
 * Logs all login attempts (successful and failed) with IP address.
 *
 * @param request - The incoming HTTP request
 * @returns JSON response with login result
 */
export async function POST(request: NextRequest): Promise<NextResponse<LoginResponse>> {
  const ip = getClientIp(request.headers);
  const userAgent = request.headers.get('user-agent');
  const body: LoginRequest = await request.json();

  // Check for suspicious IP before attempting login
  if (await isSuspiciousIp(ip)) {
    await captureAuthEvent({
      ip,
      eventType: 'blocked_suspicious',
      userAgent,
    });

    return NextResponse.json(
      {
        success: false,
        message: 'Access temporarily restricted. Contact support if this persists.',
      },
      { status: 403 }
    );
  }

  try {
    // Attempt authentication
    const result = await signIn('credentials', {
      email: body.email,
      password: body.password,
      redirect: false,
    });

    if (result?.error) {
      // Log failed login attempt
      await captureAuthEvent({
        ip,
        eventType: 'failed_login',
        userAgent,
      });

      return NextResponse.json(
        { success: false, message: 'Invalid credentials' },
        { status: 401 }
      );
    }

    // Get authenticated user
    const user = result.user;

    // Log successful login with encrypted IP
    await captureAuthEvent({
      ip,
      eventType: 'login',
      userId: user.id,
      userAgent,
    });

    return NextResponse.json({
      success: true,
      message: 'Login successful',
      user: {
        id: user.id,
        email: user.email,
      },
    });
  } catch (error) {
    console.error('Login error:', error);

    await captureAuthEvent({
      ip,
      eventType: 'failed_login',
      userAgent,
      metadata: { error: String(error) },
    });

    return NextResponse.json(
      { success: false, message: 'An error occurred' },
      { status: 500 }
    );
  }
}
```

---

## IP Security Hook - React Native

### src/hooks/useIpSecurity.ts

```typescript
/**
 * useIpSecurity.ts
 *
 * React Native hook for IP security operations.
 * Captures device information for authentication event logging via GraphQL API.
 *
 * @version React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x
 */

import { useCallback } from 'react';
import * as Network from 'expo-network';
import * as Device from 'expo-device';
import Constants from 'expo-constants';
import { useMutation } from '@apollo/client';
import { CAPTURE_AUTH_EVENT } from '@/graphql/mutations/auth';

interface CaptureEventParams {
  eventType: string;
  userId?: string | null;
  metadata?: Record<string, unknown>;
}

interface UseIpSecurityReturn {
  /** Captures an authentication event with device info */
  captureAuthEvent: (params: CaptureEventParams) => Promise<void>;
  /** Gets the device network information */
  getDeviceInfo: () => Promise<DeviceInfo>;
  /** Loading state for the mutation */
  loading: boolean;
}

interface DeviceInfo {
  ipAddress: string | null;
  deviceName: string | null;
  deviceModel: string | null;
  osVersion: string | null;
  appVersion: string | null;
}

/**
 * Hook for capturing authentication events with IP security.
 *
 * Gathers device information and sends authentication events
 * to the backend GraphQL API for secure logging.
 *
 * @returns Object with captureAuthEvent function and device info getter
 */
export function useIpSecurity(): UseIpSecurityReturn {
  const [captureEventMutation, { loading }] = useMutation(CAPTURE_AUTH_EVENT);

  /**
   * Retrieves current device and network information.
   *
   * Collects IP address, device name, model, OS version, and app version
   * for comprehensive authentication event logging.
   *
   * @returns Device information object
   */
  const getDeviceInfo = useCallback(async (): Promise<DeviceInfo> => {
    try {
      const networkState = await Network.getNetworkStateAsync();
      const ipAddress = await Network.getIpAddressAsync();

      return {
        ipAddress,
        deviceName: Device.deviceName,
        deviceModel: Device.modelName,
        osVersion: `${Device.osName} ${Device.osVersion}`,
        appVersion: Constants.expoConfig?.version ?? null,
      };
    } catch (error) {
      console.error('Failed to get device info:', error);
      return {
        ipAddress: null,
        deviceName: null,
        deviceModel: null,
        osVersion: null,
        appVersion: null,
      };
    }
  }, []);

  /**
   * Captures an authentication event with device information.
   *
   * Sends the event to the backend GraphQL API where IP addresses
   * are hashed and encrypted for GDPR compliance.
   *
   * @param params - Event parameters including type and optional user ID
   */
  const captureAuthEvent = useCallback(
    async (params: CaptureEventParams): Promise<void> => {
      const { eventType, userId = null, metadata = {} } = params;

      try {
        const deviceInfo = await getDeviceInfo();

        await captureEventMutation({
          variables: {
            input: {
              eventType,
              userId,
              ipAddress: deviceInfo.ipAddress,
              userAgent: buildUserAgent(deviceInfo),
              metadata: {
                ...metadata,
                deviceName: deviceInfo.deviceName,
                deviceModel: deviceInfo.deviceModel,
              },
            },
          },
        });
      } catch (error) {
        console.error('Failed to capture auth event:', error);
        // Fail silently - don't block auth flow for logging failures
      }
    },
    [captureEventMutation, getDeviceInfo]
  );

  return {
    captureAuthEvent,
    getDeviceInfo,
    loading,
  };
}

/**
 * Builds a user agent string from device information.
 *
 * Creates a consistent format for mobile device identification
 * in authentication logs.
 *
 * @param deviceInfo - The device information object
 * @returns Formatted user agent string
 */
function buildUserAgent(deviceInfo: DeviceInfo): string {
  const parts = [
    deviceInfo.deviceModel ?? 'Unknown Device',
    deviceInfo.osVersion ?? 'Unknown OS',
    deviceInfo.appVersion ? `App/${deviceInfo.appVersion}` : '',
  ].filter(Boolean);

  return parts.join(' ');
}
```

---

## GraphQL Mutation - React Native

### src/graphql/mutations/auth.ts

```typescript
/**
 * auth.ts
 *
 * GraphQL mutations for authentication operations.
 * Used by React Native app to communicate with the backend API.
 *
 * @version React Native 0.83.x / TypeScript 5.9
 */

import { gql } from '@apollo/client';

/**
 * Mutation to capture an authentication event.
 *
 * Sends device and IP information to the backend for secure storage.
 * The backend handles hashing and encryption of the IP address.
 */
export const CAPTURE_AUTH_EVENT = gql`
  mutation CaptureAuthEvent($input: AuthEventInput!) {
    captureAuthEvent(input: $input) {
      success
      message
    }
  }
`;

/**
 * Mutation for user login.
 *
 * Authenticates the user and returns tokens. The backend
 * automatically logs the event with IP security.
 */
export const LOGIN_MUTATION = gql`
  mutation Login($email: String!, $password: String!) {
    login(email: $email, password: $password) {
      success
      message
      user {
        id
        email
        name
      }
      tokens {
        accessToken
        refreshToken
        expiresAt
      }
    }
  }
`;

/**
 * Mutation for user logout.
 *
 * Invalidates the current session and logs the logout event.
 */
export const LOGOUT_MUTATION = gql`
  mutation Logout {
    logout {
      success
      message
    }
  }
`;

/**
 * GraphQL input type for authentication events.
 */
export interface AuthEventInput {
  eventType: string;
  userId?: string | null;
  ipAddress?: string | null;
  userAgent?: string | null;
  metadata?: Record<string, unknown>;
}
```

### src/screens/LoginScreen.tsx (Integration Example)

```tsx
/**
 * LoginScreen.tsx
 *
 * Login screen with IP security integration.
 * Captures authentication events for GDPR-compliant logging.
 *
 * @version React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x
 */

import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert } from 'react-native';
import { useMutation } from '@apollo/client';
import { LOGIN_MUTATION } from '@/graphql/mutations/auth';
import { useIpSecurity } from '@/hooks/useIpSecurity';
import { useAuth } from '@/hooks/useAuth';

/**
 * Login screen component with IP security logging.
 *
 * Captures login attempts (successful and failed) with device
 * information for security audit purposes.
 */
export function LoginScreen(): React.ReactElement {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { captureAuthEvent } = useIpSecurity();
  const { setTokens } = useAuth();
  const [login, { loading }] = useMutation(LOGIN_MUTATION);

  /**
   * Handles form submission for login.
   *
   * Attempts authentication and logs the result.
   */
  const handleLogin = async (): Promise<void> => {
    try {
      const result = await login({
        variables: { email, password },
      });

      if (result.data?.login.success) {
        // Capture successful login event
        await captureAuthEvent({
          eventType: 'login',
          userId: result.data.login.user.id,
        });

        // Store tokens
        await setTokens(result.data.login.tokens);
      } else {
        // Capture failed login event
        await captureAuthEvent({
          eventType: 'failed_login',
          metadata: { email },
        });

        Alert.alert('Login Failed', result.data?.login.message ?? 'Invalid credentials');
      }
    } catch (error) {
      // Capture error event
      await captureAuthEvent({
        eventType: 'failed_login',
        metadata: { email, error: String(error) },
      });

      Alert.alert('Error', 'An error occurred during login');
    }
  };

  return (
    <View className="flex-1 justify-center px-6 bg-white dark:bg-gray-900">
      <Text className="text-2xl font-bold text-center mb-8 text-gray-900 dark:text-white">
        Sign In
      </Text>

      <TextInput
        className="h-12 border border-gray-300 dark:border-gray-700 rounded-lg px-4 mb-4 text-gray-900 dark:text-white"
        placeholder="Email"
        placeholderTextColor="#9ca3af"
        value={email}
        onChangeText={setEmail}
        autoCapitalize="none"
        keyboardType="email-address"
      />

      <TextInput
        className="h-12 border border-gray-300 dark:border-gray-700 rounded-lg px-4 mb-6 text-gray-900 dark:text-white"
        placeholder="Password"
        placeholderTextColor="#9ca3af"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />

      <TouchableOpacity
        className="h-12 bg-blue-600 rounded-lg justify-center items-center"
        onPress={handleLogin}
        disabled={loading}
      >
        <Text className="text-white font-semibold text-lg">
          {loading ? 'Signing in...' : 'Sign In'}
        </Text>
      </TouchableOpacity>
    </View>
  );
}
```
