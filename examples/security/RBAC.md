# Role-Based Access Control (RBAC)

## Overview

Role-Based Access Control (RBAC) patterns for managing user permissions. Includes role and permission schemas, caching strategies, and middleware implementations.

## Metadata

| Property | Value |
|----------|-------|
| **Example Version** | 2.0.0 |
| **Last Updated** | 2025-12 |
| **TALL Stack** | Laravel 12.x / PHP 8.4 / MariaDB 12.x |
| **Django/Wagtail** | Django 6.x / Python 3.14 / PostgreSQL 18.x / Strawberry GraphQL |
| **React/Next.js** | Next.js 16.x / React 19.x / TypeScript 5.9 / Prisma 6.x |
| **React Native** | React Native 0.83.x / TypeScript 5.9 / NativeWind 4.x |
| **Stacks** | TALL, Django/Wagtail, Next.js, React Native |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Database Schema](#database-schema)
- [TALL Stack (Laravel) Implementation](#tall-stack-laravel-implementation)
- [Django/Wagtail Implementation](#djangowagtail-implementation)
- [React/Next.js Implementation](#reactnextjs-implementation)
- [React Native Implementation](#react-native-implementation)

## Database Schema

### MySQL/MariaDB

```sql
-- Roles table
CREATE TABLE roles (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,      -- 'admin', 'manager', 'user'
    display_name VARCHAR(100),
    description TEXT,
    is_system BOOLEAN DEFAULT FALSE,       -- Prevent deletion of system roles
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Permissions table
CREATE TABLE permissions (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,     -- 'users.create', 'posts.publish'
    display_name VARCHAR(100),
    description TEXT,
    group_name VARCHAR(50),                -- For UI grouping: 'users', 'posts'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Role-Permission pivot
CREATE TABLE role_permissions (
    role_id BIGINT UNSIGNED NOT NULL,
    permission_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
);

-- User-Role pivot
CREATE TABLE user_roles (
    user_id BIGINT UNSIGNED NOT NULL,
    role_id BIGINT UNSIGNED NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    assigned_by BIGINT UNSIGNED,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

-- Direct User-Permission (for overrides)
CREATE TABLE user_permissions (
    user_id BIGINT UNSIGNED NOT NULL,
    permission_id BIGINT UNSIGNED NOT NULL,
    granted BOOLEAN DEFAULT TRUE,          -- Can be used to deny specific permissions
    PRIMARY KEY (user_id, permission_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
);
```

---

## TALL Stack (Laravel) Implementation

## Permission Service - Laravel

### app/Services/PermissionService.php

```php
<?php

/**
 * PermissionService.php
 *
 * Handles permission and role checking with caching.
 * Provides methods for checking user permissions and roles.
 */

namespace App\Services;

use App\Models\User;
use Illuminate\Support\Facades\Cache;

class PermissionService
{
    protected int $cacheTtl = 3600; // 1 hour

    /**
     * Checks if a user has a specific permission.
     *
     * @param User $user The user to check
     * @param string $permission The permission name
     * @return bool True if user has the permission
     */
    public function userHasPermission(User $user, string $permission): bool
    {
        $permissions = $this->getUserPermissions($user);
        return in_array($permission, $permissions);
    }

    /**
     * Checks if a user has any of the specified permissions.
     *
     * @param User $user The user to check
     * @param array $permissions Array of permission names
     * @return bool True if user has any of the permissions
     */
    public function userHasAnyPermission(User $user, array $permissions): bool
    {
        $userPermissions = $this->getUserPermissions($user);
        return !empty(array_intersect($permissions, $userPermissions));
    }

    /**
     * Checks if a user has all of the specified permissions.
     *
     * @param User $user The user to check
     * @param array $permissions Array of permission names
     * @return bool True if user has all permissions
     */
    public function userHasAllPermissions(User $user, array $permissions): bool
    {
        $userPermissions = $this->getUserPermissions($user);
        return empty(array_diff($permissions, $userPermissions));
    }

    /**
     * Checks if a user has a specific role.
     *
     * @param User $user The user to check
     * @param string $role The role name
     * @return bool True if user has the role
     */
    public function userHasRole(User $user, string $role): bool
    {
        $roles = $this->getUserRoles($user);
        return in_array($role, $roles);
    }

    /**
     * Gets all permissions for a user (from roles + direct grants - denials).
     *
     * @param User $user The user to get permissions for
     * @return array Array of permission names
     */
    protected function getUserPermissions(User $user): array
    {
        return Cache::remember(
            "user_permissions:{$user->id}",
            $this->cacheTtl,
            function () use ($user) {
                // Get permissions from roles
                $rolePermissions = $user->roles()
                    ->with('permissions')
                    ->get()
                    ->pluck('permissions')
                    ->flatten()
                    ->pluck('name')
                    ->toArray();

                // Get direct user permissions
                $directPermissions = $user->permissions()
                    ->wherePivot('granted', true)
                    ->pluck('name')
                    ->toArray();

                // Get denied permissions
                $deniedPermissions = $user->permissions()
                    ->wherePivot('granted', false)
                    ->pluck('name')
                    ->toArray();

                // Merge and remove denied
                $allPermissions = array_unique(array_merge($rolePermissions, $directPermissions));
                return array_diff($allPermissions, $deniedPermissions);
            }
        );
    }

    /**
     * Gets all roles for a user.
     *
     * @param User $user The user to get roles for
     * @return array Array of role names
     */
    protected function getUserRoles(User $user): array
    {
        return Cache::remember(
            "user_roles:{$user->id}",
            $this->cacheTtl,
            fn() => $user->roles()->pluck('name')->toArray()
        );
    }

    /**
     * Clears cached permissions and roles for a user.
     * Call this when permissions or roles change.
     *
     * @param User $user The user to clear cache for
     */
    public function clearUserCache(User $user): void
    {
        Cache::forget("user_permissions:{$user->id}");
        Cache::forget("user_roles:{$user->id}");
    }
}
```

---

## Authorisation Middleware - Laravel

### app/Http/Middleware/CheckPermission.php

```php
<?php

/**
 * CheckPermission.php
 *
 * Middleware to check if user has required permissions.
 * Logs unauthorised access attempts.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use App\Services\PermissionService;

class CheckPermission
{
    public function __construct(protected PermissionService $permissions) {}

    /**
     * Handles permission checking for a request.
     *
     * @param Request $request The HTTP request
     * @param Closure $next The next middleware
     * @param string ...$permissions Required permissions (any match passes)
     * @return mixed
     */
    public function handle(Request $request, Closure $next, string ...$permissions)
    {
        $user = $request->user();

        if (!$user) {
            abort(401, 'Unauthorized');
        }

        // Check if user has any of the required permissions
        if (!$this->permissions->userHasAnyPermission($user, $permissions)) {
            // Log unauthorised access attempt
            logger()->warning('Unauthorised access attempt', [
                'user_id' => $user->id,
                'required_permissions' => $permissions,
                'path' => $request->path(),
                'ip' => $request->ip(),
            ]);

            abort(403, 'Forbidden');
        }

        return $next($request);
    }
}
```

### app/Http/Middleware/CheckRole.php

```php
<?php

/**
 * CheckRole.php
 *
 * Middleware to check if user has required role.
 * Logs unauthorised access attempts.
 */

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use App\Services\PermissionService;

class CheckRole
{
    public function __construct(protected PermissionService $permissions) {}

    /**
     * Handles role checking for a request.
     *
     * @param Request $request The HTTP request
     * @param Closure $next The next middleware
     * @param string ...$roles Required roles (any match passes)
     * @return mixed
     */
    public function handle(Request $request, Closure $next, string ...$roles)
    {
        $user = $request->user();

        if (!$user) {
            abort(401, 'Unauthorized');
        }

        $hasRole = false;
        foreach ($roles as $role) {
            if ($this->permissions->userHasRole($user, $role)) {
                $hasRole = true;
                break;
            }
        }

        if (!$hasRole) {
            logger()->warning('Unauthorised role access attempt', [
                'user_id' => $user->id,
                'required_roles' => $roles,
                'path' => $request->path(),
                'ip' => $request->ip(),
            ]);

            abort(403, 'Forbidden');
        }

        return $next($request);
    }
}
```

---

## Route Protection - Laravel

### routes/web.php

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\ProfileController;
use App\Http\Controllers\UserController;
use App\Http\Controllers\AdminController;
use App\Http\Controllers\SettingsController;
use App\Http\Controllers\SystemController;
use Illuminate\Support\Facades\Route;

// Public routes
Route::get('/', [HomeController::class, 'index']);

// Authenticated routes
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/profile', [ProfileController::class, 'show']);
});

// Permission-protected routes
Route::middleware(['auth', 'permission:users.view'])->group(function () {
    Route::get('/users', [UserController::class, 'index']);
});

Route::middleware(['auth', 'permission:users.create'])->group(function () {
    Route::post('/users', [UserController::class, 'store']);
});

Route::middleware(['auth', 'permission:users.edit,users.manage'])->group(function () {
    Route::put('/users/{user}', [UserController::class, 'update']);
});

// Role-protected routes
Route::middleware(['auth', 'role:admin'])->prefix(config('admin.path'))->group(function () {
    Route::get('/', [AdminController::class, 'dashboard']);
    Route::resource('settings', SettingsController::class);
});

// Super admin only
Route::middleware(['auth', 'role:super-admin'])->group(function () {
    Route::get('/system/logs', [SystemController::class, 'logs']);
    Route::post('/system/cache/clear', [SystemController::class, 'clearCache']);
});
```

---

## Policy-Based Authorisation - Laravel

### app/Policies/PostPolicy.php

```php
<?php

/**
 * PostPolicy.php
 *
 * Policy for authorising post-related actions.
 * Combines permission checks with ownership checks.
 */

namespace App\Policies;

use App\Models\User;
use App\Models\Post;
use App\Services\PermissionService;

class PostPolicy
{
    public function __construct(protected PermissionService $permissions) {}

    /**
     * Determines if user can view any posts.
     */
    public function viewAny(User $user): bool
    {
        return $this->permissions->userHasPermission($user, 'posts.view');
    }

    /**
     * Determines if user can view a specific post.
     * Published posts viewable with 'posts.view', unpublished only by author or admin.
     */
    public function view(User $user, Post $post): bool
    {
        // Published posts are viewable by anyone with view permission
        if ($post->is_published) {
            return $this->permissions->userHasPermission($user, 'posts.view');
        }

        // Unpublished posts only viewable by author or admin
        return $user->id === $post->user_id ||
               $this->permissions->userHasPermission($user, 'posts.view-unpublished');
    }

    /**
     * Determines if user can create posts.
     */
    public function create(User $user): bool
    {
        return $this->permissions->userHasPermission($user, 'posts.create');
    }

    /**
     * Determines if user can update a post.
     * Authors can edit their own posts with 'posts.edit-own'.
     */
    public function update(User $user, Post $post): bool
    {
        // Authors can edit their own posts
        if ($user->id === $post->user_id) {
            return $this->permissions->userHasPermission($user, 'posts.edit-own');
        }

        // Others need general edit permission
        return $this->permissions->userHasPermission($user, 'posts.edit');
    }

    /**
     * Determines if user can delete a post.
     * Authors can delete their own posts with 'posts.delete-own'.
     */
    public function delete(User $user, Post $post): bool
    {
        // Authors can delete their own posts
        if ($user->id === $post->user_id) {
            return $this->permissions->userHasPermission($user, 'posts.delete-own');
        }

        // Others need general delete permission
        return $this->permissions->userHasPermission($user, 'posts.delete');
    }

    /**
     * Determines if user can publish a post.
     */
    public function publish(User $user, Post $post): bool
    {
        return $this->permissions->userHasPermission($user, 'posts.publish');
    }
}
```

---

## Django/Wagtail Implementation

## Permission Service - Django

### services/permission_service.py

```python
"""
permission_service.py

Handles permission and role checking with caching.
Provides methods for checking user permissions and roles.
"""

from django.core.cache import cache
from django.contrib.auth import get_user_model

User = get_user_model()


class PermissionService:
    """
    Service for checking user permissions and roles.
    Uses caching to reduce database queries.
    """

    CACHE_TTL = 3600  # 1 hour

    def user_has_permission(self, user: User, permission: str) -> bool:
        """
        Checks if a user has a specific permission.

        Args:
            user: The user to check
            permission: The permission name

        Returns:
            True if user has the permission
        """
        permissions = self._get_user_permissions(user)
        return permission in permissions

    def user_has_any_permission(self, user: User, permissions: list[str]) -> bool:
        """
        Checks if a user has any of the specified permissions.

        Args:
            user: The user to check
            permissions: List of permission names

        Returns:
            True if user has any of the permissions
        """
        user_permissions = self._get_user_permissions(user)
        return bool(set(permissions) & set(user_permissions))

    def user_has_role(self, user: User, role: str) -> bool:
        """
        Checks if a user has a specific role.

        Args:
            user: The user to check
            role: The role name

        Returns:
            True if user has the role
        """
        roles = self._get_user_roles(user)
        return role in roles

    def _get_user_permissions(self, user: User) -> list[str]:
        """
        Gets all permissions for a user from cache or database.
        """
        cache_key = f"user_permissions:{user.id}"
        permissions = cache.get(cache_key)

        if permissions is None:
            # Get permissions from roles
            role_permissions = list(
                user.roles.values_list('permissions__name', flat=True)
            )

            # Get direct user permissions
            direct_permissions = list(
                user.user_permissions.filter(granted=True).values_list('permission__name', flat=True)
            )

            # Get denied permissions
            denied_permissions = list(
                user.user_permissions.filter(granted=False).values_list('permission__name', flat=True)
            )

            # Merge and remove denied
            all_permissions = set(role_permissions + direct_permissions)
            permissions = list(all_permissions - set(denied_permissions))

            cache.set(cache_key, permissions, self.CACHE_TTL)

        return permissions

    def _get_user_roles(self, user: User) -> list[str]:
        """
        Gets all roles for a user from cache or database.
        """
        cache_key = f"user_roles:{user.id}"
        roles = cache.get(cache_key)

        if roles is None:
            roles = list(user.roles.values_list('name', flat=True))
            cache.set(cache_key, roles, self.CACHE_TTL)

        return roles

    def clear_user_cache(self, user: User) -> None:
        """
        Clears cached permissions and roles for a user.
        """
        cache.delete(f"user_permissions:{user.id}")
        cache.delete(f"user_roles:{user.id}")
```

---

## Permission Decorators - Django

### decorators/permission_required.py

```python
"""
permission_required.py

Decorators for checking permissions on Django views.
Provides both function-based and class-based view decorators.
"""

from functools import wraps
from django.http import HttpResponseForbidden
from django.contrib.auth.decorators import login_required
from services.permission_service import PermissionService


def permission_required(*permissions):
    """
    Decorator to require specific permissions for a view.
    User must have at least one of the specified permissions.

    Args:
        *permissions: Variable number of permission names required

    Returns:
        Decorated view function

    Example:
        @permission_required('posts.view', 'posts.manage')
        def view_posts(request):
            ...
    """
    def decorator(view_func):
        @wraps(view_func)
        @login_required
        def wrapped_view(request, *args, **kwargs):
            permission_service = PermissionService()

            if not permission_service.user_has_any_permission(request.user, list(permissions)):
                return HttpResponseForbidden('Insufficient permissions')

            return view_func(request, *args, **kwargs)

        return wrapped_view
    return decorator


def role_required(*roles):
    """
    Decorator to require specific roles for a view.
    User must have at least one of the specified roles.

    Args:
        *roles: Variable number of role names required

    Returns:
        Decorated view function

    Example:
        @role_required('admin', 'manager')
        def admin_dashboard(request):
            ...
    """
    def decorator(view_func):
        @wraps(view_func)
        @login_required
        def wrapped_view(request, *args, **kwargs):
            permission_service = PermissionService()

            for role in roles:
                if permission_service.user_has_role(request.user, role):
                    return view_func(request, *args, **kwargs)

            return HttpResponseForbidden('Insufficient role')

        return wrapped_view
    return decorator
```

### views.py - Usage Example

```python
"""
views.py

Example Django views using permission decorators.
"""

from django.shortcuts import render
from decorators.permission_required import permission_required, role_required


@permission_required('posts.view')
def post_list(request):
    """
    View for listing posts.
    Requires 'posts.view' permission.
    """
    # View implementation
    return render(request, 'posts/list.html')


@permission_required('posts.create')
def post_create(request):
    """
    View for creating posts.
    Requires 'posts.create' permission.
    """
    # View implementation
    return render(request, 'posts/create.html')


@role_required('admin', 'manager')
def admin_dashboard(request):
    """
    Admin dashboard view.
    Requires 'admin' or 'manager' role.
    """
    # View implementation
    return render(request, 'admin/dashboard.html')
```

---

## Strawberry GraphQL Permissions

### graphql/permissions.py

```python
"""
permissions.py

Permission extensions for Strawberry GraphQL.
Provides field and type-level permission checking.
"""

from typing import Any
import strawberry
from strawberry.permission import BasePermission
from strawberry.types import Info
from services.permission_service import PermissionService


class IsAuthenticated(BasePermission):
    """
    Permission class to check if user is authenticated.
    """

    message = "User is not authenticated"

    def has_permission(self, source: Any, info: Info, **kwargs) -> bool:
        """
        Checks if the request has an authenticated user.

        Args:
            source: The source object
            info: GraphQL execution info containing request context
            **kwargs: Additional keyword arguments

        Returns:
            True if user is authenticated
        """
        request = info.context.get('request')
        return request and hasattr(request, 'user') and request.user.is_authenticated


class HasPermission(BasePermission):
    """
    Permission class to check if user has specific permissions.
    """

    def __init__(self, permissions: list[str]):
        """
        Initialises the permission checker with required permissions.

        Args:
            permissions: List of permission names (user needs at least one)
        """
        self.permissions = permissions
        self.message = f"User lacks required permissions: {', '.join(permissions)}"

    def has_permission(self, source: Any, info: Info, **kwargs) -> bool:
        """
        Checks if user has any of the required permissions.

        Args:
            source: The source object
            info: GraphQL execution info containing request context
            **kwargs: Additional keyword arguments

        Returns:
            True if user has at least one required permission
        """
        request = info.context.get('request')
        if not request or not hasattr(request, 'user') or not request.user.is_authenticated:
            return False

        permission_service = PermissionService()
        return permission_service.user_has_any_permission(request.user, self.permissions)


class HasRole(BasePermission):
    """
    Permission class to check if user has specific roles.
    """

    def __init__(self, roles: list[str]):
        """
        Initialises the role checker with required roles.

        Args:
            roles: List of role names (user needs at least one)
        """
        self.roles = roles
        self.message = f"User lacks required roles: {', '.join(roles)}"

    def has_permission(self, source: Any, info: Info, **kwargs) -> bool:
        """
        Checks if user has any of the required roles.

        Args:
            source: The source object
            info: GraphQL execution info containing request context
            **kwargs: Additional keyword arguments

        Returns:
            True if user has at least one required role
        """
        request = info.context.get('request')
        if not request or not hasattr(request, 'user') or not request.user.is_authenticated:
            return False

        permission_service = PermissionService()
        for role in self.roles:
            if permission_service.user_has_role(request.user, role):
                return True
        return False
```

### graphql/schema.py

```python
"""
schema.py

Example Strawberry GraphQL schema using permissions.
"""

import strawberry
from typing import List
from graphql.permissions import IsAuthenticated, HasPermission, HasRole


@strawberry.type
class Post:
    """GraphQL Post type."""
    id: int
    title: str
    content: str
    is_published: bool


@strawberry.type
class Query:
    """GraphQL Query root."""

    @strawberry.field(permission_classes=[IsAuthenticated, HasPermission(['posts.view'])])
    def posts(self) -> List[Post]:
        """
        Returns list of posts.
        Requires authentication and 'posts.view' permission.
        """
        # Query implementation
        return []

    @strawberry.field(permission_classes=[IsAuthenticated, HasRole(['admin', 'manager'])])
    def admin_stats(self) -> str:
        """
        Returns admin statistics.
        Requires 'admin' or 'manager' role.
        """
        return "Admin statistics"


@strawberry.type
class Mutation:
    """GraphQL Mutation root."""

    @strawberry.mutation(permission_classes=[IsAuthenticated, HasPermission(['posts.create'])])
    def create_post(self, title: str, content: str) -> Post:
        """
        Creates a new post.
        Requires authentication and 'posts.create' permission.
        """
        # Mutation implementation
        return Post(id=1, title=title, content=content, is_published=False)

    @strawberry.mutation(permission_classes=[IsAuthenticated, HasPermission(['posts.publish'])])
    def publish_post(self, post_id: int) -> Post:
        """
        Publishes a post.
        Requires authentication and 'posts.publish' permission.
        """
        # Mutation implementation
        return Post(id=post_id, title="", content="", is_published=True)


schema = strawberry.Schema(query=Query, mutation=Mutation)
```

---

## React/Next.js Implementation

## Next.js Middleware for Route Protection

### middleware.ts

```typescript
/**
 * middleware.ts
 *
 * Next.js middleware for protecting routes based on authentication and permissions.
 * Runs on the Edge runtime for optimal performance.
 */

import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { getToken } from 'next-auth/jwt';

/**
 * Route permission configuration.
 * Maps route patterns to required permissions.
 */
const ROUTE_PERMISSIONS: Record<string, string[]> = {
  '/dashboard': ['dashboard.view'],
  '/users': ['users.view'],
  '/users/create': ['users.create'],
  '/users/edit': ['users.edit'],
  '/admin': ['admin.access'],
  '/settings': ['settings.manage'],
};

/**
 * Route role configuration.
 * Maps route patterns to required roles.
 */
const ROUTE_ROLES: Record<string, string[]> = {
  '/admin': ['admin', 'super-admin'],
  '/moderator': ['moderator', 'admin'],
};

/**
 * Checks if user has any of the required permissions.
 *
 * @param userPermissions - Array of user's permission names
 * @param requiredPermissions - Array of required permission names
 * @returns True if user has at least one required permission
 */
function hasAnyPermission(
  userPermissions: string[],
  requiredPermissions: string[]
): boolean {
  return requiredPermissions.some((perm) => userPermissions.includes(perm));
}

/**
 * Checks if user has any of the required roles.
 *
 * @param userRoles - Array of user's role names
 * @param requiredRoles - Array of required role names
 * @returns True if user has at least one required role
 */
function hasAnyRole(userRoles: string[], requiredRoles: string[]): boolean {
  return requiredRoles.some((role) => userRoles.includes(role));
}

/**
 * Next.js middleware function for route protection.
 * Checks authentication, permissions, and roles.
 *
 * @param request - The incoming request
 * @returns NextResponse to continue or redirect
 */
export async function middleware(request: NextRequest) {
  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET,
  });

  const { pathname } = request.nextUrl;

  // Allow public routes
  if (
    pathname.startsWith('/api/auth') ||
    pathname.startsWith('/_next') ||
    pathname.startsWith('/static') ||
    pathname === '/login' ||
    pathname === '/'
  ) {
    return NextResponse.next();
  }

  // Check authentication
  if (!token) {
    const url = request.nextUrl.clone();
    url.pathname = '/login';
    url.searchParams.set('callbackUrl', pathname);
    return NextResponse.redirect(url);
  }

  // Check route permissions
  for (const [route, requiredPermissions] of Object.entries(ROUTE_PERMISSIONS)) {
    if (pathname.startsWith(route)) {
      const userPermissions = (token.permissions as string[]) || [];

      if (!hasAnyPermission(userPermissions, requiredPermissions)) {
        const url = request.nextUrl.clone();
        url.pathname = '/forbidden';
        return NextResponse.redirect(url);
      }
    }
  }

  // Check route roles
  for (const [route, requiredRoles] of Object.entries(ROUTE_ROLES)) {
    if (pathname.startsWith(route)) {
      const userRoles = (token.roles as string[]) || [];

      if (!hasAnyRole(userRoles, requiredRoles)) {
        const url = request.nextUrl.clone();
        url.pathname = '/forbidden';
        return NextResponse.redirect(url);
      }
    }
  }

  return NextResponse.next();
}

/**
 * Middleware configuration.
 * Specifies which routes the middleware should run on.
 */
export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * - api/auth (authentication endpoints)
     * - _next/static (static files)
     * - _next/image (image optimisation files)
     * - favicon.ico (favicon file)
     * - public folder
     */
    '/((?!api/auth|_next/static|_next/image|favicon.ico|public).*)',
  ],
};
```

---

## Permission Context Provider

### contexts/PermissionContext.tsx

```typescript
/**
 * PermissionContext.tsx
 *
 * React context for managing user permissions and roles.
 * Provides hooks for checking permissions throughout the application.
 */

'use client';

import React, { createContext, useContext, ReactNode } from 'react';
import { useSession } from 'next-auth/react';

/**
 * User permission and role data structure.
 */
interface PermissionContextType {
  permissions: string[];
  roles: string[];
  hasPermission: (permission: string) => boolean;
  hasAnyPermission: (permissions: string[]) => boolean;
  hasAllPermissions: (permissions: string[]) => boolean;
  hasRole: (role: string) => boolean;
  hasAnyRole: (roles: string[]) => boolean;
  isLoading: boolean;
}

const PermissionContext = createContext<PermissionContextType | undefined>(
  undefined
);

/**
 * Permission provider component.
 * Wraps the application to provide permission context.
 *
 * @param children - Child components
 * @returns Permission context provider
 */
export function PermissionProvider({ children }: { children: ReactNode }) {
  const { data: session, status } = useSession();

  const permissions = (session?.user?.permissions as string[]) || [];
  const roles = (session?.user?.roles as string[]) || [];
  const isLoading = status === 'loading';

  /**
   * Checks if user has a specific permission.
   *
   * @param permission - Permission name to check
   * @returns True if user has the permission
   */
  const hasPermission = (permission: string): boolean => {
    return permissions.includes(permission);
  };

  /**
   * Checks if user has any of the specified permissions.
   *
   * @param perms - Array of permission names
   * @returns True if user has at least one permission
   */
  const hasAnyPermission = (perms: string[]): boolean => {
    return perms.some((perm) => permissions.includes(perm));
  };

  /**
   * Checks if user has all of the specified permissions.
   *
   * @param perms - Array of permission names
   * @returns True if user has all permissions
   */
  const hasAllPermissions = (perms: string[]): boolean => {
    return perms.every((perm) => permissions.includes(perm));
  };

  /**
   * Checks if user has a specific role.
   *
   * @param role - Role name to check
   * @returns True if user has the role
   */
  const hasRole = (role: string): boolean => {
    return roles.includes(role);
  };

  /**
   * Checks if user has any of the specified roles.
   *
   * @param roleList - Array of role names
   * @returns True if user has at least one role
   */
  const hasAnyRole = (roleList: string[]): boolean => {
    return roleList.some((role) => roles.includes(role));
  };

  return (
    <PermissionContext.Provider
      value={{
        permissions,
        roles,
        hasPermission,
        hasAnyPermission,
        hasAllPermissions,
        hasRole,
        hasAnyRole,
        isLoading,
      }}
    >
      {children}
    </PermissionContext.Provider>
  );
}

/**
 * Hook to access permission context.
 *
 * @returns Permission context
 * @throws Error if used outside PermissionProvider
 */
export function usePermissions(): PermissionContextType {
  const context = useContext(PermissionContext);
  if (!context) {
    throw new Error('usePermissions must be used within PermissionProvider');
  }
  return context;
}
```

---

## Protected Component Wrapper

### components/PermissionGuard.tsx

```typescript
/**
 * PermissionGuard.tsx
 *
 * Component wrapper for permission-based rendering.
 * Conditionally renders children based on user permissions or roles.
 */

'use client';

import React, { ReactNode } from 'react';
import { usePermissions } from '@/contexts/PermissionContext';

/**
 * Props for PermissionGuard component.
 */
interface PermissionGuardProps {
  /** Child components to render if permission check passes */
  children: ReactNode;
  /** Required permissions (user needs at least one) */
  permissions?: string[];
  /** Required roles (user needs at least one) */
  roles?: string[];
  /** Require all permissions instead of any */
  requireAll?: boolean;
  /** Fallback component to render if permission check fails */
  fallback?: ReactNode;
}

/**
 * Permission guard component.
 * Renders children only if user has required permissions/roles.
 *
 * @param props - Component props
 * @returns Protected content or fallback
 *
 * @example
 * <PermissionGuard permissions={['posts.create']}>
 *   <CreatePostButton />
 * </PermissionGuard>
 */
export function PermissionGuard({
  children,
  permissions = [],
  roles = [],
  requireAll = false,
  fallback = null,
}: PermissionGuardProps) {
  const {
    hasAnyPermission,
    hasAllPermissions,
    hasAnyRole,
    isLoading,
  } = usePermissions();

  // Show nothing while loading
  if (isLoading) {
    return null;
  }

  // Check permissions
  if (permissions.length > 0) {
    const hasRequiredPermissions = requireAll
      ? hasAllPermissions(permissions)
      : hasAnyPermission(permissions);

    if (!hasRequiredPermissions) {
      return <>{fallback}</>;
    }
  }

  // Check roles
  if (roles.length > 0) {
    if (!hasAnyRole(roles)) {
      return <>{fallback}</>;
    }
  }

  return <>{children}</>;
}
```

### components/Example.tsx - Usage

```typescript
/**
 * Example.tsx
 *
 * Example component demonstrating PermissionGuard usage.
 */

'use client';

import React from 'react';
import { PermissionGuard } from '@/components/PermissionGuard';
import { usePermissions } from '@/contexts/PermissionContext';

export function ExampleDashboard() {
  const { hasPermission } = usePermissions();

  return (
    <div>
      <h1>Dashboard</h1>

      {/* Show button only if user has permission */}
      <PermissionGuard permissions={['posts.create']}>
        <button>Create Post</button>
      </PermissionGuard>

      {/* Show admin panel only for admin role */}
      <PermissionGuard roles={['admin']}>
        <div>Admin Panel</div>
      </PermissionGuard>

      {/* Show with fallback message */}
      <PermissionGuard
        permissions={['users.view']}
        fallback={<p>You don't have permission to view users.</p>}
      >
        <UserList />
      </PermissionGuard>

      {/* Programmatic check */}
      {hasPermission('settings.edit') && (
        <button>Edit Settings</button>
      )}
    </div>
  );
}

function UserList() {
  return <div>User list component</div>;
}
```

---

## Server-Side Permission Checks

### lib/permissions.ts

```typescript
/**
 * permissions.ts
 *
 * Server-side utilities for checking permissions.
 * Used in Server Components and API routes.
 */

import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

/**
 * User session with permissions and roles.
 */
interface UserSession {
  user: {
    id: string;
    email: string;
    permissions: string[];
    roles: string[];
  };
}

/**
 * Gets the current user's session on the server.
 *
 * @returns User session or null if not authenticated
 */
export async function getCurrentUser(): Promise<UserSession | null> {
  const session = await getServerSession(authOptions);
  return session as UserSession | null;
}

/**
 * Checks if current user has a specific permission.
 *
 * @param permission - Permission name to check
 * @returns True if user has the permission
 */
export async function hasPermission(permission: string): Promise<boolean> {
  const session = await getCurrentUser();
  if (!session?.user?.permissions) return false;
  return session.user.permissions.includes(permission);
}

/**
 * Checks if current user has any of the specified permissions.
 *
 * @param permissions - Array of permission names
 * @returns True if user has at least one permission
 */
export async function hasAnyPermission(
  permissions: string[]
): Promise<boolean> {
  const session = await getCurrentUser();
  if (!session?.user?.permissions) return false;
  return permissions.some((perm) =>
    session.user.permissions.includes(perm)
  );
}

/**
 * Checks if current user has a specific role.
 *
 * @param role - Role name to check
 * @returns True if user has the role
 */
export async function hasRole(role: string): Promise<boolean> {
  const session = await getCurrentUser();
  if (!session?.user?.roles) return false;
  return session.user.roles.includes(role);
}

/**
 * Requires specific permissions or throws unauthorised error.
 * Use in Server Actions and API routes.
 *
 * @param permissions - Required permission names
 * @throws Error if user lacks permissions
 */
export async function requirePermissions(permissions: string[]): Promise<void> {
  const hasPerms = await hasAnyPermission(permissions);
  if (!hasPerms) {
    throw new Error('Unauthorised: Insufficient permissions');
  }
}

/**
 * Requires specific role or throws unauthorised error.
 * Use in Server Actions and API routes.
 *
 * @param role - Required role name
 * @throws Error if user lacks role
 */
export async function requireRole(role: string): Promise<void> {
  const hasUserRole = await hasRole(role);
  if (!hasUserRole) {
    throw new Error('Unauthorised: Insufficient role');
  }
}
```

### app/api/posts/route.ts - API Route Example

```typescript
/**
 * route.ts
 *
 * Example API route with permission checking.
 */

import { NextRequest, NextResponse } from 'next/server';
import { requirePermissions } from '@/lib/permissions';

/**
 * GET /api/posts
 * Retrieves list of posts.
 * Requires 'posts.view' permission.
 */
export async function GET(request: NextRequest) {
  try {
    await requirePermissions(['posts.view']);

    // Fetch posts logic here
    const posts = [];

    return NextResponse.json(posts);
  } catch (error) {
    return NextResponse.json(
      { error: 'Unauthorised' },
      { status: 403 }
    );
  }
}

/**
 * POST /api/posts
 * Creates a new post.
 * Requires 'posts.create' permission.
 */
export async function POST(request: NextRequest) {
  try {
    await requirePermissions(['posts.create']);

    const body = await request.json();

    // Create post logic here
    const post = { id: 1, ...body };

    return NextResponse.json(post, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: 'Unauthorised' },
      { status: 403 }
    );
  }
}
```

---

## React Native Implementation

## Permission Context Provider - React Native

### contexts/PermissionContext.tsx

```typescript
/**
 * PermissionContext.tsx
 *
 * React Native context for managing user permissions and roles.
 * Provides hooks for checking permissions throughout the mobile app.
 */

import React, {
  createContext,
  useContext,
  useState,
  useEffect,
  ReactNode,
} from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

/**
 * User permission and role data structure.
 */
interface PermissionContextType {
  permissions: string[];
  roles: string[];
  hasPermission: (permission: string) => boolean;
  hasAnyPermission: (permissions: string[]) => boolean;
  hasAllPermissions: (permissions: string[]) => boolean;
  hasRole: (role: string) => boolean;
  hasAnyRole: (roles: string[]) => boolean;
  isLoading: boolean;
  refreshPermissions: () => Promise<void>;
}

const PermissionContext = createContext<PermissionContextType | undefined>(
  undefined
);

/**
 * Permission provider component for React Native.
 * Manages permissions from AsyncStorage and API.
 *
 * @param children - Child components
 * @returns Permission context provider
 */
export function PermissionProvider({ children }: { children: ReactNode }) {
  const [permissions, setPermissions] = useState<string[]>([]);
  const [roles, setRoles] = useState<string[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  /**
   * Loads permissions and roles from AsyncStorage.
   */
  const loadPermissions = async () => {
    try {
      const [storedPermissions, storedRoles] = await Promise.all([
        AsyncStorage.getItem('user_permissions'),
        AsyncStorage.getItem('user_roles'),
      ]);

      if (storedPermissions) {
        setPermissions(JSON.parse(storedPermissions));
      }

      if (storedRoles) {
        setRoles(JSON.parse(storedRoles));
      }
    } catch (error) {
      console.error('Failed to load permissions:', error);
    } finally {
      setIsLoading(false);
    }
  };

  /**
   * Refreshes permissions and roles from the API.
   */
  const refreshPermissions = async () => {
    try {
      // Fetch from your API
      const response = await fetch('/api/user/permissions');
      const data = await response.json();

      const newPermissions = data.permissions || [];
      const newRoles = data.roles || [];

      // Update state
      setPermissions(newPermissions);
      setRoles(newRoles);

      // Persist to AsyncStorage
      await Promise.all([
        AsyncStorage.setItem('user_permissions', JSON.stringify(newPermissions)),
        AsyncStorage.setItem('user_roles', JSON.stringify(newRoles)),
      ]);
    } catch (error) {
      console.error('Failed to refresh permissions:', error);
    }
  };

  useEffect(() => {
    loadPermissions();
  }, []);

  /**
   * Checks if user has a specific permission.
   *
   * @param permission - Permission name to check
   * @returns True if user has the permission
   */
  const hasPermission = (permission: string): boolean => {
    return permissions.includes(permission);
  };

  /**
   * Checks if user has any of the specified permissions.
   *
   * @param perms - Array of permission names
   * @returns True if user has at least one permission
   */
  const hasAnyPermission = (perms: string[]): boolean => {
    return perms.some((perm) => permissions.includes(perm));
  };

  /**
   * Checks if user has all of the specified permissions.
   *
   * @param perms - Array of permission names
   * @returns True if user has all permissions
   */
  const hasAllPermissions = (perms: string[]): boolean => {
    return perms.every((perm) => permissions.includes(perm));
  };

  /**
   * Checks if user has a specific role.
   *
   * @param role - Role name to check
   * @returns True if user has the role
   */
  const hasRole = (role: string): boolean => {
    return roles.includes(role);
  };

  /**
   * Checks if user has any of the specified roles.
   *
   * @param roleList - Array of role names
   * @returns True if user has at least one role
   */
  const hasAnyRole = (roleList: string[]): boolean => {
    return roleList.some((role) => roles.includes(role));
  };

  return (
    <PermissionContext.Provider
      value={{
        permissions,
        roles,
        hasPermission,
        hasAnyPermission,
        hasAllPermissions,
        hasRole,
        hasAnyRole,
        isLoading,
        refreshPermissions,
      }}
    >
      {children}
    </PermissionContext.Provider>
  );
}

/**
 * Hook to access permission context.
 *
 * @returns Permission context
 * @throws Error if used outside PermissionProvider
 */
export function usePermissions(): PermissionContextType {
  const context = useContext(PermissionContext);
  if (!context) {
    throw new Error('usePermissions must be used within PermissionProvider');
  }
  return context;
}
```

---

## usePermission Hook

### hooks/usePermission.ts

```typescript
/**
 * usePermission.ts
 *
 * Custom hook for permission-based logic in React Native components.
 * Provides simplified permission checking interface.
 */

import { usePermissions } from '@/contexts/PermissionContext';

/**
 * Permission check result.
 */
interface UsePermissionResult {
  /** Whether user has the required permission(s) */
  hasAccess: boolean;
  /** Whether permissions are still loading */
  isLoading: boolean;
  /** All user permissions */
  permissions: string[];
  /** All user roles */
  roles: string[];
}

/**
 * Hook for checking a single permission.
 *
 * @param permission - Permission name to check
 * @returns Permission check result
 *
 * @example
 * const { hasAccess, isLoading } = usePermission('posts.create');
 */
export function usePermission(permission: string): UsePermissionResult {
  const { hasPermission, isLoading, permissions, roles } = usePermissions();

  return {
    hasAccess: hasPermission(permission),
    isLoading,
    permissions,
    roles,
  };
}

/**
 * Hook for checking multiple permissions (any match).
 *
 * @param requiredPermissions - Array of permission names
 * @returns Permission check result
 *
 * @example
 * const { hasAccess } = useAnyPermission(['posts.edit', 'posts.manage']);
 */
export function useAnyPermission(
  requiredPermissions: string[]
): UsePermissionResult {
  const { hasAnyPermission, isLoading, permissions, roles } = usePermissions();

  return {
    hasAccess: hasAnyPermission(requiredPermissions),
    isLoading,
    permissions,
    roles,
  };
}

/**
 * Hook for checking if user has a specific role.
 *
 * @param role - Role name to check
 * @returns Permission check result
 *
 * @example
 * const { hasAccess } = useRole('admin');
 */
export function useRole(role: string): UsePermissionResult {
  const { hasRole, isLoading, permissions, roles } = usePermissions();

  return {
    hasAccess: hasRole(role),
    isLoading,
    permissions,
    roles,
  };
}
```

---

## Screen Guards

### components/PermissionGuard.tsx

```typescript
/**
 * PermissionGuard.tsx
 *
 * React Native component wrapper for permission-based rendering.
 * Conditionally renders children based on user permissions or roles.
 */

import React, { ReactNode } from 'react';
import { View, Text } from 'react-native';
import { usePermissions } from '@/contexts/PermissionContext';

/**
 * Props for PermissionGuard component.
 */
interface PermissionGuardProps {
  /** Child components to render if permission check passes */
  children: ReactNode;
  /** Required permissions (user needs at least one) */
  permissions?: string[];
  /** Required roles (user needs at least one) */
  roles?: string[];
  /** Require all permissions instead of any */
  requireAll?: boolean;
  /** Fallback component to render if permission check fails */
  fallback?: ReactNode;
  /** Show loading indicator while checking permissions */
  showLoading?: boolean;
}

/**
 * Permission guard component for React Native.
 * Renders children only if user has required permissions/roles.
 *
 * @param props - Component props
 * @returns Protected content, fallback, or loading indicator
 *
 * @example
 * <PermissionGuard permissions={['posts.create']}>
 *   <CreatePostButton />
 * </PermissionGuard>
 */
export function PermissionGuard({
  children,
  permissions = [],
  roles = [],
  requireAll = false,
  fallback = null,
  showLoading = true,
}: PermissionGuardProps) {
  const {
    hasAnyPermission,
    hasAllPermissions,
    hasAnyRole,
    isLoading,
  } = usePermissions();

  // Show loading indicator while permissions are being fetched
  if (isLoading && showLoading) {
    return (
      <View className="flex-1 items-center justify-center">
        <Text className="text-gray-500">Loading...</Text>
      </View>
    );
  }

  // Check permissions
  if (permissions.length > 0) {
    const hasRequiredPermissions = requireAll
      ? hasAllPermissions(permissions)
      : hasAnyPermission(permissions);

    if (!hasRequiredPermissions) {
      return <>{fallback}</>;
    }
  }

  // Check roles
  if (roles.length > 0) {
    if (!hasAnyRole(roles)) {
      return <>{fallback}</>;
    }
  }

  return <>{children}</>;
}

/**
 * Default fallback component for unauthorised access.
 */
export function UnauthorisedFallback() {
  return (
    <View className="flex-1 items-center justify-center p-4">
      <Text className="text-lg font-semibold text-red-600 mb-2">
        Access Denied
      </Text>
      <Text className="text-gray-600 text-center">
        You don't have permission to access this content.
      </Text>
    </View>
  );
}
```

### screens/ExampleScreen.tsx - Usage

```typescript
/**
 * ExampleScreen.tsx
 *
 * Example React Native screen demonstrating PermissionGuard usage.
 */

import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';
import { PermissionGuard, UnauthorisedFallback } from '@/components/PermissionGuard';
import { usePermission } from '@/hooks/usePermission';

export function ExampleScreen() {
  const { hasAccess: canCreatePosts } = usePermission('posts.create');

  return (
    <View className="flex-1 p-4">
      <Text className="text-2xl font-bold mb-4">Dashboard</Text>

      {/* Show button only if user has permission */}
      <PermissionGuard permissions={['posts.create']}>
        <TouchableOpacity className="bg-blue-500 p-4 rounded mb-4">
          <Text className="text-white text-center">Create Post</Text>
        </TouchableOpacity>
      </PermissionGuard>

      {/* Show admin section only for admin role */}
      <PermissionGuard
        roles={['admin']}
        fallback={<UnauthorisedFallback />}
      >
        <View className="bg-gray-100 p-4 rounded mb-4">
          <Text className="font-semibold">Admin Panel</Text>
        </View>
      </PermissionGuard>

      {/* Programmatic check */}
      {canCreatePosts && (
        <TouchableOpacity className="bg-green-500 p-4 rounded">
          <Text className="text-white text-center">Quick Create</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}
```

---

## Navigation Guards

### navigation/ProtectedNavigator.tsx

```typescript
/**
 * ProtectedNavigator.tsx
 *
 * Navigation wrapper that protects routes based on permissions.
 * Prevents unauthorised users from accessing restricted screens.
 */

import React from 'react';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { usePermissions } from '@/contexts/PermissionContext';
import { View, Text } from 'react-native';

import HomeScreen from '@/screens/HomeScreen';
import DashboardScreen from '@/screens/DashboardScreen';
import AdminScreen from '@/screens/AdminScreen';
import UsersScreen from '@/screens/UsersScreen';
import UnauthorisedScreen from '@/screens/UnauthorisedScreen';

const Stack = createNativeStackNavigator();

/**
 * Screen configuration with permission requirements.
 */
interface ProtectedRoute {
  name: string;
  component: React.ComponentType<any>;
  permissions?: string[];
  roles?: string[];
}

const routes: ProtectedRoute[] = [
  {
    name: 'Home',
    component: HomeScreen,
  },
  {
    name: 'Dashboard',
    component: DashboardScreen,
    permissions: ['dashboard.view'],
  },
  {
    name: 'Users',
    component: UsersScreen,
    permissions: ['users.view'],
  },
  {
    name: 'Admin',
    component: AdminScreen,
    roles: ['admin'],
  },
];

/**
 * Protected navigator component.
 * Only renders screens user has permission to access.
 *
 * @returns Protected navigation stack
 */
export function ProtectedNavigator() {
  const { hasAnyPermission, hasAnyRole, isLoading } = usePermissions();

  if (isLoading) {
    return (
      <View className="flex-1 items-center justify-center">
        <Text>Loading permissions...</Text>
      </View>
    );
  }

  /**
   * Checks if user can access a route.
   *
   * @param route - Route configuration
   * @returns True if user has access
   */
  const canAccessRoute = (route: ProtectedRoute): boolean => {
    // Public routes (no permissions required)
    if (!route.permissions && !route.roles) {
      return true;
    }

    // Check permissions
    if (route.permissions && !hasAnyPermission(route.permissions)) {
      return false;
    }

    // Check roles
    if (route.roles && !hasAnyRole(route.roles)) {
      return false;
    }

    return true;
  };

  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: true,
      }}
    >
      {routes.map((route) =>
        canAccessRoute(route) ? (
          <Stack.Screen
            key={route.name}
            name={route.name}
            component={route.component}
          />
        ) : null
      )}

      {/* Unauthorised screen for denied access attempts */}
      <Stack.Screen
        name="Unauthorised"
        component={UnauthorisedScreen}
        options={{ title: 'Access Denied' }}
      />
    </Stack.Navigator>
  );
}
```

### navigation/useProtectedNavigation.ts

```typescript
/**
 * useProtectedNavigation.ts
 *
 * Hook for navigation with permission checks.
 * Prevents navigation to unauthorised screens.
 */

import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { usePermissions } from '@/contexts/PermissionContext';

/**
 * Route permission mapping.
 */
const ROUTE_PERMISSIONS: Record<string, string[]> = {
  Dashboard: ['dashboard.view'],
  Users: ['users.view'],
  CreateUser: ['users.create'],
  EditUser: ['users.edit'],
  Settings: ['settings.manage'],
};

const ROUTE_ROLES: Record<string, string[]> = {
  Admin: ['admin'],
  Moderator: ['moderator', 'admin'],
};

/**
 * Hook for protected navigation.
 * Checks permissions before navigating.
 *
 * @returns Navigation functions with permission checks
 *
 * @example
 * const { navigateIfAllowed } = useProtectedNavigation();
 * navigateIfAllowed('Admin');
 */
export function useProtectedNavigation() {
  const navigation = useNavigation<NativeStackNavigationProp<any>>();
  const { hasAnyPermission, hasAnyRole } = usePermissions();

  /**
   * Checks if navigation to a route is allowed.
   *
   * @param routeName - Name of the route
   * @returns True if user can navigate to the route
   */
  const canNavigateTo = (routeName: string): boolean => {
    // Check permissions
    const requiredPermissions = ROUTE_PERMISSIONS[routeName];
    if (requiredPermissions && !hasAnyPermission(requiredPermissions)) {
      return false;
    }

    // Check roles
    const requiredRoles = ROUTE_ROLES[routeName];
    if (requiredRoles && !hasAnyRole(requiredRoles)) {
      return false;
    }

    return true;
  };

  /**
   * Navigates to a route if user has permission.
   * Otherwise navigates to Unauthorised screen.
   *
   * @param routeName - Name of the route
   * @param params - Navigation parameters
   */
  const navigateIfAllowed = (routeName: string, params?: any) => {
    if (canNavigateTo(routeName)) {
      navigation.navigate(routeName, params);
    } else {
      navigation.navigate('Unauthorised', { attemptedRoute: routeName });
    }
  };

  return {
    canNavigateTo,
    navigateIfAllowed,
    navigation,
  };
}
```

### screens/UnauthorisedScreen.tsx

```typescript
/**
 * UnauthorisedScreen.tsx
 *
 * Screen displayed when user attempts to access unauthorised content.
 */

import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';

export default function UnauthorisedScreen() {
  const navigation = useNavigation<NativeStackNavigationProp<any>>();

  return (
    <View className="flex-1 items-center justify-center p-6 bg-white">
      <View className="items-center">
        <Text className="text-6xl mb-4">🔒</Text>
        <Text className="text-2xl font-bold text-gray-900 mb-2">
          Access Denied
        </Text>
        <Text className="text-gray-600 text-center mb-8">
          You don't have permission to access this page.
          Please contact your administrator if you believe this is an error.
        </Text>

        <TouchableOpacity
          className="bg-blue-500 px-6 py-3 rounded-lg"
          onPress={() => navigation.navigate('Home')}
        >
          <Text className="text-white font-semibold">Go to Home</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
```
