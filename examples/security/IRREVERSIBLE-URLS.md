# Irreversible URLs (UUIDs, Hashids, Single-Use Tokens)

## Overview

Irreversible URL patterns prevent enumeration attacks on resources. Sequential IDs like `/users/1`, `/users/2` allow attackers to discover all resources. These patterns replace predictable IDs with unpredictable identifiers.

## Metadata

| Property            | Value                                             |
| ------------------- | ------------------------------------------------- |
| **Example Version** | 2.0.0                                             |
| **Last Updated**    | 2025-12                                           |
| **Laravel**         | 12.x                                              |
| **PHP**             | 8.4                                               |
| **Django**          | 6.x                                               |
| **Python**          | 3.14                                              |
| **Next.js**         | 16.x                                              |
| **Node.js**         | 24.x                                              |
| **TypeScript**      | 5.9                                               |
| **React Native**    | 0.83.x                                            |
| **Stacks**          | TALL, Django/Wagtail, React/Next.js, React Native |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [URL Obfuscation Strategies](#url-obfuscation-strategies)
- [TALL Stack (Laravel 12.x / PHP 8.4)](#tall-stack-laravel-12x--php-84)
  - [UUID Trait - Laravel](#uuid-trait---laravel)
  - [app/Traits/HasPublicUuid.php](#apptraitshaspublicuuidphp)
  - [Usage in Model](#usage-in-model)
- [Hashids Service - Laravel](#hashids-service---laravel)
  - [app/Services/HashidService.php](#appserviceshashidservicephp)
  - [app/Http/Middleware/DecodeHashid.php](#apphttpmiddlewaredecodehashidphp)
- [Single-Use Token Service - Laravel](#single-use-token-service---laravel)
  - [app/Services/SingleUseTokenService.php](#appservicessingleusetokenservicephp)
- [Route Examples - Laravel](#route-examples---laravel)
  - [routes/web.php](#routeswebphp)
- [Django/Wagtail Stack (Django 6.x / Python 3.14)](#djangowagtail-stack-django-6x--python-314)
  - [UUID Mixin - Django](#uuid-mixin---django)
  - [mixins/uuid\_mixin.py](#mixinsuuid_mixinpy)
- [Hashids Service - NestJS](#hashids-service---nestjs)
  - [src/common/services/hashid.service.ts](#srccommonserviceshashidservicets)
  - [src/common/pipes/hashid.pipe.ts](#srccommonpipeshashidpipets)

## URL Obfuscation Strategies

| Strategy         | Use Case                         | Example                                              |
| ---------------- | -------------------------------- | ---------------------------------------------------- |
| **UUID v4**      | Public-facing resource IDs       | `/users/550e8400-e29b-41d4-a716-446655440000`        |
| **Hashids**      | Short, obfuscated IDs            | `/users/jR` (maps to ID 1)                           |
| **Signed URLs**  | Time-limited access              | `/download/file?signature=abc123&expires=1234567890` |
| **HMAC tokens**  | Single-use access                | `/verify/a1b2c3d4e5f6...`                            |
| **Random slugs** | Human-readable but unpredictable | `/invoice/XK7m9pLq2nR4`                              |

---

## TALL Stack (Laravel 12.x / PHP 8.4)

### UUID Trait - Laravel

### app/Traits/HasPublicUuid.php

```php
<?php

/**
 * Trait for UUID-based public identification.
 *
 * Models using this trait expose UUIDs instead of sequential IDs in URLs.
 * The internal auto-increment ID remains for database relationships.
 */

namespace App\Traits;

use Illuminate\Support\Str;

trait HasPublicUuid
{
    /**
     * Boot the trait to auto-generate UUID on model creation.
     *
     * @return void
     */
    protected static function bootHasPublicUuid(): void
    {
        static::creating(function ($model) {
            if (empty($model->uuid)) {
                $model->uuid = (string) Str::uuid();
            }
        });
    }

    /**
     * Returns the route key name for URL binding.
     * Laravel will use 'uuid' instead of 'id' for route model binding.
     *
     * @return string The column name to use for route binding
     */
    public function getRouteKeyName(): string
    {
        return 'uuid';
    }
}
```

### Usage in Model

```php
<?php

namespace App\Models;

use App\Traits\HasPublicUuid;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasPublicUuid;

    // Now /users/{user} expects a UUID, not an ID
    // Example: /users/550e8400-e29b-41d4-a716-446655440000
}
```

---

## Hashids Service - Laravel

### app/Services/HashidService.php

```php
<?php

/**
 * Hashids Service for obfuscating numeric IDs.
 *
 * Converts sequential IDs to short, unpredictable strings.
 * The mapping is reversible within the application but not externally guessable.
 */

namespace App\Services;

use Hashids\Hashids;

class HashidService
{
    protected Hashids $hashids;

    public function __construct()
    {
        // Use a unique salt per application - store in env
        $this->hashids = new Hashids(
            config('app.hashids_salt'),
            8, // Minimum length
            'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890'
        );
    }

    /**
     * Encodes an ID to an obfuscated string.
     *
     * @param int $id The database ID to encode
     * @return string The obfuscated ID string
     */
    public function encode(int $id): string
    {
        return $this->hashids->encode($id);
    }

    /**
     * Decodes an obfuscated string back to the original ID.
     *
     * @param string $hash The obfuscated ID string
     * @return int|null The original database ID or null if invalid
     */
    public function decode(string $hash): ?int
    {
        $decoded = $this->hashids->decode($hash);
        return $decoded[0] ?? null;
    }
}
```

### app/Http/Middleware/DecodeHashid.php

```php
<?php

/**
 * Middleware to auto-decode hashids in route parameters.
 *
 * Decodes hashid parameters before they reach the controller,
 * allowing transparent use of obfuscated IDs in URLs.
 */

namespace App\Http\Middleware;

use App\Services\HashidService;
use Closure;
use Illuminate\Http\Request;

class DecodeHashid
{
    public function __construct(protected HashidService $hashidService) {}

    /**
     * Decodes a hashid route parameter to the original ID.
     *
     * @param Request $request The HTTP request
     * @param Closure $next The next middleware
     * @param string $parameter The route parameter name to decode
     * @return mixed
     */
    public function handle(Request $request, Closure $next, string $parameter = 'id')
    {
        $hash = $request->route($parameter);

        if ($hash) {
            $id = $this->hashidService->decode($hash);

            if ($id === null) {
                abort(404); // Invalid hashid returns 404
            }

            $request->route()->setParameter($parameter, $id);
        }

        return $next($request);
    }
}
```

---

## Single-Use Token Service - Laravel

### app/Services/SingleUseTokenService.php

```php
<?php

/**
 * Single-Use Token Service
 *
 * Generates cryptographically secure, single-use tokens for sensitive actions.
 * Tokens are invalidated after use and cannot be predicted or reused.
 */

namespace App\Services;

use Illuminate\Support\Facades\Cache;

class SingleUseTokenService
{
    /**
     * Generates a single-use token for a specific action.
     * Token is cryptographically random and stored with metadata.
     *
     * @param string $action The action type (e.g., 'password-reset', 'email-verify')
     * @param array $data Additional data to store with the token
     * @param int $expiryMinutes Token validity period in minutes
     * @return string The generated token (64 characters)
     */
    public function generate(string $action, array $data, int $expiryMinutes = 60): string
    {
        // Generate cryptographically secure random token
        $token = bin2hex(random_bytes(32)); // 64 hex characters

        Cache::put(
            "single_use_token:{$action}:{$token}",
            [
                'data' => $data,
                'created_at' => now()->toIso8601String(),
                'ip' => request()->ip(),
            ],
            now()->addMinutes($expiryMinutes)
        );

        return $token;
    }

    /**
     * Validates and consumes a single-use token.
     * After validation, the token is immediately invalidated.
     *
     * @param string $action The action type the token was created for
     * @param string $token The token to validate
     * @return array|null The stored data if valid, null if invalid/expired/used
     */
    public function consume(string $action, string $token): ?array
    {
        $key = "single_use_token:{$action}:{$token}";
        $data = Cache::get($key);

        if (!$data) {
            return null; // Token not found, expired, or already used
        }

        // Immediately invalidate the token (single-use)
        Cache::forget($key);

        return $data['data'];
    }

    /**
     * Generates a URL with an embedded single-use token.
     *
     * @param string $routeName The named route to generate URL for
     * @param string $action The action type for the token
     * @param array $data Data to embed in the token
     * @param int $expiryMinutes Token validity period
     * @return string The full URL with token
     */
    public function generateUrl(
        string $routeName,
        string $action,
        array $data,
        int $expiryMinutes = 60
    ): string {
        $token = $this->generate($action, $data, $expiryMinutes);

        return route($routeName, ['token' => $token]);
    }
}
```

---

## Route Examples - Laravel

### routes/web.php

```php
<?php

use App\Http\Controllers\UserController;
use App\Http\Controllers\InvoiceController;
use App\Http\Controllers\DocumentController;
use App\Http\Controllers\VerificationController;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\Facades\URL;

// BAD - Predictable, enumerable
// Route::get('/users/{id}', [UserController::class, 'show']);  // /users/1, /users/2...
// Route::get('/invoices/{id}', [InvoiceController::class, 'show']);  // /invoices/1...

// GOOD - UUID-based (use HasPublicUuid trait)
Route::get('/users/{user}', [UserController::class, 'show']);
// Results in: /users/550e8400-e29b-41d4-a716-446655440000

// GOOD - Hashid-based
Route::get('/invoices/{invoice}', [InvoiceController::class, 'show'])
    ->middleware('decode-hashid:invoice');
// Results in: /invoices/jR9xKp

// GOOD - Signed URLs for sensitive downloads
Route::get('/documents/{document}/download', [DocumentController::class, 'download'])
    ->name('documents.download')
    ->middleware('signed');
// Generate with: URL::signedRoute('documents.download', ['document' => $doc->uuid])

// GOOD - Single-use tokens for verification
Route::get('/verify-email/{token}', [VerificationController::class, 'verify'])
    ->name('email.verify');
// Token is consumed on first use, cannot be reused
```

---

## Django/Wagtail Stack (Django 6.x / Python 3.14)

### UUID Mixin - Django

### mixins/uuid_mixin.py

```python
"""
uuid_mixin.py

Mixin for UUID-based public identification in Django models.
Models using this mixin expose UUIDs instead of sequential IDs in URLs.
"""

import uuid
from django.db import models


class UuidMixin(models.Model):
    """
    Abstract mixin that adds a UUID field for public identification.
    The internal auto-increment ID remains for database relationships.
    """

    uuid = models.UUIDField(
        default=uuid.uuid4,
        editable=False,
        unique=True,
        db_index=True,
    )

    class Meta:
        abstract = True

    def get_public_id(self) -> str:
        """
        Returns the public UUID identifier.

        Returns:
            The UUID as a string
        """
        return str(self.uuid)


# Usage in Model:
# class User(UuidMixin, AbstractUser):
#     pass
#
# # In URLs:
# path('users/<uuid:uuid>/', views.user_detail, name='user-detail')
```

---

## Hashids Service - NestJS

### src/common/services/hashid.service.ts

```typescript
/**
 * hashid.service.ts
 *
 * Service for obfuscating numeric IDs using Hashids.
 * Converts sequential IDs to short, unpredictable strings.
 */

import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import Hashids from 'hashids';

@Injectable()
export class HashidService {
  private hashids: Hashids;

  constructor(private configService: ConfigService) {
    const salt = this.configService.get<string>('HASHIDS_SALT', 'default-salt');
    const minLength = this.configService.get<number>('HASHIDS_MIN_LENGTH', 8);

    this.hashids = new Hashids(
      salt,
      minLength,
      'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890',
    );
  }

  /**
   * Encodes a numeric ID to an obfuscated string.
   *
   * @param id - The database ID to encode
   * @returns The obfuscated ID string
   */
  encode(id: number): string {
    return this.hashids.encode(id);
  }

  /**
   * Decodes an obfuscated string back to the original ID.
   *
   * @param hash - The obfuscated ID string
   * @returns The original database ID or null if invalid
   */
  decode(hash: string): number | null {
    const decoded = this.hashids.decode(hash);
    return decoded.length > 0 ? (decoded[0] as number) : null;
  }

  /**
   * Encodes multiple IDs to a single obfuscated string.
   *
   * @param ids - Array of database IDs to encode
   * @returns The obfuscated string containing all IDs
   */
  encodeMany(ids: number[]): string {
    return this.hashids.encode(...ids);
  }

  /**
   * Decodes an obfuscated string back to multiple IDs.
   *
   * @param hash - The obfuscated string
   * @returns Array of original database IDs
   */
  decodeMany(hash: string): number[] {
    return this.hashids.decode(hash) as number[];
  }
}
```

### src/common/pipes/hashid.pipe.ts

```typescript
/**
 * hashid.pipe.ts
 *
 * Pipe to decode hashid parameters in controller routes.
 * Transforms hashid strings to numeric IDs automatically.
 */

import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  NotFoundException,
} from '@nestjs/common';
import { HashidService } from '../services/hashid.service';

@Injectable()
export class HashidPipe implements PipeTransform<string, number> {
  constructor(private hashidService: HashidService) {}

  /**
   * Transforms a hashid string to a numeric ID.
   *
   * @param value - The hashid string from the route parameter
   * @param metadata - Argument metadata (unused)
   * @returns The decoded numeric ID
   * @throws NotFoundException if the hashid is invalid
   */
  transform(value: string, metadata: ArgumentMetadata): number {
    const id = this.hashidService.decode(value);

    if (id === null) {
      throw new NotFoundException(); // Return 404 for invalid hashids
    }

    return id;
  }
}

// Usage in controller:
// @Get(':id')
// findOne(@Param('id', HashidPipe) id: number) {
//   return this.usersService.findOne(id);
// }
```
