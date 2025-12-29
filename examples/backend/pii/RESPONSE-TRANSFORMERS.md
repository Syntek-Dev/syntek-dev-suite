# PII Response Transformers

**Last Updated**: 29/12/2025
**Version**: 1.3.1
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Metadata

| Property         | Value         |
| ---------------- | ------------- |
| **Version**      | 2.0.0         |
| **Last Updated** | December 2025 |
| **Status**       | Stable        |

## Framework Versions Tested

| Framework          | Version  | Tested Date |
| ------------------ | -------- | ----------- |
| Laravel            | 12.x     | 20/12/2025  |
| Django             | 6.x      | 20/12/2025  |
| Strawberry GraphQL | 0.250+   | 20/12/2025  |
| Next.js            | 16.x     | 20/12/2025  |
| PHP                | 8.4      | 20/12/2025  |
| Python             | 3.14     | 20/12/2025  |
| Node.js            | 24.x LTS | 20/12/2025  |
| TypeScript         | 5.9      | 20/12/2025  |

---

## Table of Contents

- [Metadata](#metadata)
- [Framework Versions Tested](#framework-versions-tested)
- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Laravel (TALL Stack)](#laravel-tall-stack)
  - [API Resources](#api-resources)
  - [Alternative: Conditional Attributes](#alternative-conditional-attributes)
  - [Usage Example](#usage-example)
- [Django/Wagtail (Strawberry GraphQL)](#djangowagtail-strawberry-graphql)
- [Node.js/TypeScript (NestJS GraphQL)](#nodejstypescript-nestjs-graphql)


## Overview

Response transformers filter PII fields from API responses based on the requesting user's permissions. Only users with `pii.access` (or `pii_access`) permission should see sensitive fields like email, phone, and full name.

**Pattern:** Check permission in the transformer/resolver, return `null` if permission denied.

**Key Principles:**
- Server-side filtering only (never send PII to unauthorised clients)
- Explicit permission checks before exposing PII
- Consistent null/undefined return for denied access
- Type-safe implementations where possible

---

## Laravel (TALL Stack)

### API Resources

Laravel API Resources provide a clean way to transform models for API responses with permission-based filtering.

```php
<?php
/**
 * UserResource.php
 *
 * Transforms User model for API responses.
 * Filters PII based on the requesting user's permissions.
 *
 * Laravel 12.x / PHP 8.4
 */

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transforms the user model for API response.
     * Only includes PII fields if the requester has pii.access permission.
     *
     * @param Request $request The incoming request
     * @return array<string, mixed> The transformed user data
     */
    public function toArray(Request $request): array
    {
        $data = [
            'id' => $this->public_uuid,
            'username' => $this->username,
            'avatar_url' => $this->avatar_url,
            'created_at' => $this->created_at->toIso8601String(),
        ];

        // Only include PII if requester has permission
        if ($request->user()?->hasPermission('pii.access')) {
            $data['email'] = $this->email;
            $data['full_name'] = $this->full_name;
            $data['phone'] = $this->phone;
        }

        return $data;
    }
}
```

### Alternative: Conditional Attributes

```php
<?php
/**
 * UserResource.php (Alternative using conditional attributes)
 *
 * Demonstrates using Laravel's conditional methods for cleaner syntax.
 *
 * Laravel 12.x / PHP 8.4
 */

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transforms the user model for API response.
     * Uses Laravel's when() method for conditional attribute inclusion.
     *
     * @param Request $request The incoming request
     * @return array<string, mixed> The transformed user data
     */
    public function toArray(Request $request): array
    {
        $canAccessPii = $request->user()?->hasPermission('pii.access') ?? false;

        return [
            'id' => $this->public_uuid,
            'username' => $this->username,
            'avatar_url' => $this->avatar_url,
            'created_at' => $this->created_at->toIso8601String(),

            // Conditionally include PII fields
            $this->mergeWhen($canAccessPii, [
                'email' => $this->email,
                'full_name' => $this->full_name,
                'phone' => $this->phone,
            ]),
        ];
    }

    /**
     * Checks if the authenticated user can access PII.
     *
     * @param Request $request The incoming request
     * @return bool True if user has pii.access permission
     */
    protected function canAccessPii(Request $request): bool
    {
        return $request->user()?->hasPermission('pii.access') ?? false;
    }
}
```

### Usage Example

```php
<?php
/**
 * UserController.php
 *
 * Example controller using UserResource.
 */

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    /**
     * Display the specified user.
     *
     * @param User $user The user to display
     * @return JsonResponse The JSON response with transformed user data
     */
    public function show(User $user): JsonResponse
    {
        return UserResource::make($user)->response();
    }

    /**
     * Display a listing of users.
     *
     * @return JsonResponse The JSON response with collection of users
     */
    public function index(): JsonResponse
    {
        $users = User::paginate(15);

        return UserResource::collection($users)->response();
    }
}
```

---

## Django/Wagtail (Strawberry GraphQL)

```python
"""
schema/types/user_type.py

GraphQL type for User model with PII field resolution.
Filters PII based on the requesting user's permissions.
"""

import strawberry
from strawberry.types import Info
from typing import Optional
from datetime import datetime


@strawberry.type
class UserType:
    """
    GraphQL type for User model.
    PII fields resolve to None unless the requester has pii_access permission.
    """

    public_uuid: str
    username: str
    avatar_url: Optional[str]
    created_at: datetime

    @strawberry.field
    def email(self, info: Info) -> Optional[str]:
        """
        Resolves user email only if requester has pii_access permission.

        Args:
            info: GraphQL resolver info containing request context

        Returns:
            str | None: The email address or None if permission denied
        """
        request = info.context.get('request')
        if request and request.user.has_perm('accounts.pii_access'):
            return self._email
        return None

    @strawberry.field
    def full_name(self, info: Info) -> Optional[str]:
        """
        Resolves user full name only if requester has pii_access permission.

        Args:
            info: GraphQL resolver info containing request context

        Returns:
            str | None: The full name or None if permission denied
        """
        request = info.context.get('request')
        if request and request.user.has_perm('accounts.pii_access'):
            return self._full_name
        return None

    @strawberry.field
    def phone(self, info: Info) -> Optional[str]:
        """
        Resolves user phone only if requester has pii_access permission.

        Args:
            info: GraphQL resolver info containing request context

        Returns:
            str | None: The phone number or None if permission denied
        """
        request = info.context.get('request')
        if request and request.user.has_perm('accounts.pii_access'):
            return self._phone
        return None
```

---

## Node.js/TypeScript (NestJS GraphQL)

```typescript
/**
 * user.resolver.ts
 *
 * GraphQL resolver for User type with PII field resolution.
 * Filters PII based on the requesting user's permissions.
 */

import { Resolver, Query, ResolveField, Parent, Context } from '@nestjs/graphql';
import { UserType } from './user.type';
import { User } from '../entities/user.entity';
import { GqlContext } from '../types/context';

@Resolver(() => UserType)
export class UserResolver {
  /**
   * Resolves user email only if requester has pii.access permission.
   *
   * @param user - The parent User entity
   * @param ctx - GraphQL context containing request and user
   * @returns The email address or null if permission denied
   */
  @ResolveField(() => String, { nullable: true })
  email(@Parent() user: User, @Context() ctx: GqlContext): string | null {
    if (ctx.user?.permissions?.includes('pii.access')) {
      return user.email;
    }
    return null;
  }

  /**
   * Resolves user full name only if requester has pii.access permission.
   *
   * @param user - The parent User entity
   * @param ctx - GraphQL context containing request and user
   * @returns The full name or null if permission denied
   */
  @ResolveField(() => String, { nullable: true })
  fullName(@Parent() user: User, @Context() ctx: GqlContext): string | null {
    if (ctx.user?.permissions?.includes('pii.access')) {
      return user.fullName;
    }
    return null;
  }

  /**
   * Resolves user phone only if requester has pii.access permission.
   *
   * @param user - The parent User entity
   * @param ctx - GraphQL context containing request and user
   * @returns The phone number or null if permission denied
   */
  @ResolveField(() => String, { nullable: true })
  phone(@Parent() user: User, @Context() ctx: GqlContext): string | null {
    if (ctx.user?.permissions?.includes('pii.access')) {
      return user.phone;
    }
    return null;
  }
}
```
