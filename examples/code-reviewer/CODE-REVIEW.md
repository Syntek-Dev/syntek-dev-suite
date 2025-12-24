# Code Review Examples

## Overview

Code review guidelines, checklists, and examples for maintaining code quality across all technology stacks.

## Table of Contents

- [Code Review Examples](#code-review-examples)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Review Checklists](#review-checklists)
    - [General Checklist](#general-checklist)
  - [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
    - [Laravel Controller Review](#laravel-controller-review)
      - [Before (Needs Improvement)](#before-needs-improvement)
      - [After (Improved)](#after-improved)
    - [Laravel Service Review](#laravel-service-review)
      - [Before (Needs Improvement)](#before-needs-improvement-1)
      - [After (Improved)](#after-improved-1)
    - [Laravel Security Review](#laravel-security-review)
  - [Django/Wagtail Stack](#djangowagtail-stack)
    - [Django View Review](#django-view-review)
      - [Before (Needs Improvement)](#before-needs-improvement-2)
      - [After (Improved)](#after-improved-2)
    - [Django Model Review](#django-model-review)
      - [Before (Needs Improvement)](#before-needs-improvement-3)
      - [After (Improved)](#after-improved-3)
    - [Django Security Review](#django-security-review)
  - [React/Next.js Stack](#reactnextjs-stack)
    - [Component Review](#component-review)
      - [Before (Needs Improvement)](#before-needs-improvement-4)
      - [After (Improved)](#after-improved-4)
    - [Hook Review](#hook-review)
      - [Before (Needs Improvement)](#before-needs-improvement-5)
      - [After (Improved)](#after-improved-5)
    - [Performance Review](#performance-review)
  - [React Native Stack](#react-native-stack)
    - [RN Component Review](#rn-component-review)
      - [Before (Needs Improvement)](#before-needs-improvement-6)
      - [After (Improved)](#after-improved-6)
    - [Native Module Review](#native-module-review)
    - [RN Performance Review](#rn-performance-review)


## Review Checklists

### General Checklist

- [ ] Code follows project style guidelines
- [ ] No hardcoded secrets or credentials
- [ ] Proper error handling in place
- [ ] Unit tests cover new functionality
- [ ] Documentation updated if needed
- [ ] No unnecessary complexity
- [ ] Performance considerations addressed
- [ ] Security vulnerabilities checked

---


## TALL Stack (Laravel 12)

### Laravel Controller Review

#### Before (Needs Improvement)

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        // Issue: No validation
        // Issue: Business logic in controller
        // Issue: No type hints
        // Issue: Raw password storage risk

        $user = new User();
        $user->name = $request->name;
        $user->email = $request->email;
        $user->password = $request->password; // Security issue!
        $user->role = $request->role; // Mass assignment vulnerability
        $user->save();

        // Issue: Exposing all user data
        return response()->json($user);
    }

    public function update(Request $request, $id)
    {
        // Issue: No authorisation check
        // Issue: No type hints on $id
        $user = User::find($id);
        $user->update($request->all()); // Mass assignment vulnerability

        return response()->json($user);
    }
}
```

#### After (Improved)

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use App\Http\Requests\UpdateUserRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use App\Services\UserService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

final class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService
    ) {}

    public function index(): AnonymousResourceCollection
    {
        $users = User::query()
            ->with(['profile', 'roles'])
            ->paginate();

        return UserResource::collection($users);
    }

    public function store(StoreUserRequest $request): JsonResponse
    {
        $user = $this->userService->createUser($request->validated());

        return UserResource::make($user)
            ->response()
            ->setStatusCode(201);
    }

    public function show(User $user): UserResource
    {
        $this->authorize('view', $user);

        return UserResource::make($user->load(['profile', 'roles']));
    }

    public function update(UpdateUserRequest $request, User $user): UserResource
    {
        $this->authorize('update', $user);

        $user = $this->userService->updateUser($user, $request->validated());

        return UserResource::make($user);
    }

    public function destroy(User $user): JsonResponse
    {
        $this->authorize('delete', $user);

        $this->userService->deleteUser($user);

        return response()->json(null, 204);
    }
}
```

### Laravel Service Review

#### Before (Needs Improvement)

```php
<?php
// app/Services/PaymentService.php

namespace App\Services;

use App\Models\Payment;
use Stripe\Stripe;

class PaymentService
{
    public function processPayment($userId, $amount)
    {
        // Issue: No type hints
        // Issue: Hardcoded API key
        // Issue: No error handling
        // Issue: No transaction

        Stripe::setApiKey(''); // Security issue!

        $charge = \Stripe\Charge::create([
            'amount' => $amount,
            'currency' => 'gbp',
        ]);

        $payment = new Payment();
        $payment->user_id = $userId;
        $payment->amount = $amount;
        $payment->stripe_id = $charge->id;
        $payment->save();

        return $payment;
    }
}
```

#### After (Improved)

```php
<?php
// app/Services/PaymentService.php

namespace App\Services;

use App\DTOs\CreatePaymentDTO;
use App\Events\PaymentProcessed;
use App\Exceptions\PaymentFailedException;
use App\Models\Payment;
use App\Models\User;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Stripe\Exception\ApiErrorException;
use Stripe\StripeClient;

final readonly class PaymentService
{
    public function __construct(
        private StripeClient $stripe
    ) {}

    /**
     * Process a payment for the given user.
     *
     * @throws PaymentFailedException
     */
    public function processPayment(User $user, CreatePaymentDTO $dto): Payment
    {
        return DB::transaction(function () use ($user, $dto) {
            try {
                $charge = $this->stripe->charges->create([
                    'amount' => $dto->amountInPence,
                    'currency' => 'gbp',
                    'customer' => $user->stripe_customer_id,
                    'description' => $dto->description,
                    'metadata' => [
                        'user_id' => $user->id,
                        'order_id' => $dto->orderId,
                    ],
                ]);

                $payment = Payment::create([
                    'user_id' => $user->id,
                    'amount' => $dto->amountInPence,
                    'currency' => 'GBP',
                    'stripe_charge_id' => $charge->id,
                    'status' => $charge->status,
                    'description' => $dto->description,
                ]);

                event(new PaymentProcessed($payment));

                Log::info('Payment processed successfully', [
                    'payment_id' => $payment->id,
                    'user_id' => $user->id,
                    'amount' => $dto->amountInPence,
                ]);

                return $payment;
            } catch (ApiErrorException $e) {
                Log::error('Stripe payment failed', [
                    'user_id' => $user->id,
                    'error' => $e->getMessage(),
                    'code' => $e->getStripeCode(),
                ]);

                throw new PaymentFailedException(
                    message: 'Payment processing failed',
                    previous: $e
                );
            }
        });
    }
}
```

### Laravel Security Review

```php
<?php
// Security Review Checklist for Laravel

/**
 * SECURITY REVIEW CHECKLIST
 *
 * 1. Input Validation
 *    - [ ] All user input validated using Form Requests
 *    - [ ] Validation rules are strict and appropriate
 *    - [ ] File uploads validated for type, size, and content
 *
 * 2. Authentication & Authorisation
 *    - [ ] Routes protected with appropriate middleware
 *    - [ ] Policies used for resource authorisation
 *    - [ ] Rate limiting applied to sensitive endpoints
 *
 * 3. SQL Injection Prevention
 *    - [ ] Using Eloquent or Query Builder (no raw SQL)
 *    - [ ] Parameterised queries if raw SQL is necessary
 *    - [ ] No user input in orderBy/groupBy without whitelist
 *
 * 4. XSS Prevention
 *    - [ ] Output escaped in Blade templates
 *    - [ ] Using {!! !!} only for trusted content
 *    - [ ] Content-Security-Policy headers set
 *
 * 5. CSRF Protection
 *    - [ ] @csrf token in all forms
 *    - [ ] API routes use token authentication
 *
 * 6. Mass Assignment Protection
 *    - [ ] $fillable or $guarded defined on models
 *    - [ ] Using validated() from Form Requests
 *
 * 7. Sensitive Data Handling
 *    - [ ] Passwords hashed with Hash::make()
 *    - [ ] Sensitive data encrypted at rest
 *    - [ ] API keys in environment variables
 *    - [ ] No secrets in version control
 */

// Example: Secure file upload handling
namespace App\Http\Controllers;

use App\Http\Requests\UploadDocumentRequest;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;

final class DocumentController extends Controller
{
    public function store(UploadDocumentRequest $request)
    {
        $file = $request->file('document');

        // Generate secure filename
        $filename = Str::uuid() . '.' . $file->getClientOriginalExtension();

        // Store in private disk (not publicly accessible)
        $path = $file->storeAs(
            'documents/' . auth()->id(),
            $filename,
            'private'
        );

        return response()->json([
            'path' => $path,
            'message' => 'Document uploaded successfully',
        ], 201);
    }
}
```

---

## Django/Wagtail Stack

### Django View Review

#### Before (Needs Improvement)

```python
# apps/users/views.py

from django.http import JsonResponse
from django.views import View
from .models import User
import json

class UserView(View):
    def post(self, request):
        # Issue: No input validation
        # Issue: No authentication check
        # Issue: SQL injection risk
        # Issue: No error handling

        data = json.loads(request.body)

        # Raw SQL - vulnerable!
        from django.db import connection
        cursor = connection.cursor()
        cursor.execute(f"INSERT INTO users (name, email) VALUES ('{data['name']}', '{data['email']}')")

        return JsonResponse({'status': 'created'})

    def get(self, request, user_id):
        # Issue: No type conversion
        # Issue: Exposes all fields
        user = User.objects.filter(id=user_id).values()[0]
        return JsonResponse(user)
```

#### After (Improved)

```python
# apps/users/views.py

from rest_framework import status
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

from .models import User
from .serializers import UserSerializer, CreateUserSerializer
from .services import UserService


class UserListCreateView(APIView):
    """API view for listing and creating users."""

    permission_classes = [IsAuthenticated]

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.user_service = UserService()

    def get(self, request) -> Response:
        """List all users with pagination."""
        users = User.objects.select_related('profile').all()
        serializer = UserSerializer(users, many=True)
        return Response(serializer.data)

    def post(self, request) -> Response:
        """Create a new user."""
        serializer = CreateUserSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        user = self.user_service.create_user(**serializer.validated_data)

        return Response(
            UserSerializer(user).data,
            status=status.HTTP_201_CREATED
        )


class UserDetailView(APIView):
    """API view for user detail operations."""

    permission_classes = [IsAuthenticated]

    def get_object(self, pk: int) -> User:
        """Get user or raise 404."""
        from django.shortcuts import get_object_or_404
        return get_object_or_404(
            User.objects.select_related('profile'),
            pk=pk
        )

    def get(self, request, pk: int) -> Response:
        """Retrieve a user."""
        user = self.get_object(pk)
        self.check_object_permissions(request, user)
        return Response(UserSerializer(user).data)

    def put(self, request, pk: int) -> Response:
        """Update a user."""
        user = self.get_object(pk)
        self.check_object_permissions(request, user)

        serializer = UserSerializer(user, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()

        return Response(serializer.data)

    def delete(self, request, pk: int) -> Response:
        """Delete a user."""
        user = self.get_object(pk)
        self.check_object_permissions(request, user)
        user.delete()

        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Django Model Review

#### Before (Needs Improvement)

```python
# apps/products/models.py

from django.db import models

class Product(models.Model):
    # Issue: No field constraints
    # Issue: No indexes
    # Issue: No validation
    # Issue: Inconsistent naming

    Name = models.CharField(max_length=500)  # PascalCase
    desc = models.TextField()  # Abbreviated
    price = models.FloatField()  # Should be Decimal
    qty = models.IntegerField()  # Abbreviated
    created = models.DateTimeField(auto_now_add=True)

    class Meta:
        pass  # No ordering, no indexes
```

#### After (Improved)

```python
# apps/products/models.py

from decimal import Decimal
from django.core.validators import MinValueValidator
from django.db import models
from django.utils.translation import gettext_lazy as _


class Product(models.Model):
    """Product model representing items for sale."""

    class Status(models.TextChoices):
        DRAFT = 'draft', _('Draft')
        ACTIVE = 'active', _('Active')
        DISCONTINUED = 'discontinued', _('Discontinued')

    name = models.CharField(
        _('name'),
        max_length=255,
        db_index=True,
    )
    slug = models.SlugField(
        _('slug'),
        max_length=255,
        unique=True,
    )
    description = models.TextField(
        _('description'),
        blank=True,
    )
    price = models.DecimalField(
        _('price'),
        max_digits=10,
        decimal_places=2,
        validators=[MinValueValidator(Decimal('0.01'))],
    )
    quantity = models.PositiveIntegerField(
        _('quantity'),
        default=0,
    )
    status = models.CharField(
        _('status'),
        max_length=20,
        choices=Status.choices,
        default=Status.DRAFT,
        db_index=True,
    )
    created_at = models.DateTimeField(
        _('created at'),
        auto_now_add=True,
    )
    updated_at = models.DateTimeField(
        _('updated at'),
        auto_now=True,
    )

    class Meta:
        verbose_name = _('product')
        verbose_name_plural = _('products')
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['name', 'status']),
            models.Index(fields=['created_at']),
        ]

    def __str__(self) -> str:
        return self.name

    @property
    def is_available(self) -> bool:
        """Check if product is available for purchase."""
        return self.status == self.Status.ACTIVE and self.quantity > 0

    def reduce_stock(self, quantity: int) -> None:
        """Reduce stock by specified quantity."""
        if quantity > self.quantity:
            raise ValueError('Insufficient stock')
        self.quantity -= quantity
        self.save(update_fields=['quantity', 'updated_at'])
```

### Django Security Review

```python
# apps/core/security_review.py

"""
SECURITY REVIEW CHECKLIST FOR DJANGO

1. Input Validation
   - [ ] Serializers validate all input data
   - [ ] File uploads validated (type, size, content)
   - [ ] URL parameters validated and typed

2. Authentication & Authorisation
   - [ ] Views use appropriate permission classes
   - [ ] Object-level permissions checked
   - [ ] Session security settings configured

3. SQL Injection Prevention
   - [ ] Using ORM queries (no raw SQL)
   - [ ] Parameterised queries if raw SQL necessary
   - [ ] No user input in extra()/raw()

4. XSS Prevention
   - [ ] Templates auto-escape by default
   - [ ] mark_safe() used only for trusted content
   - [ ] Content-Security-Policy headers set

5. CSRF Protection
   - [ ] CsrfViewMiddleware enabled
   - [ ] @csrf_exempt used sparingly with justification
   - [ ] API uses token authentication

6. Sensitive Data
   - [ ] Passwords hashed with make_password()
   - [ ] Secrets in environment variables
   - [ ] DEBUG=False in production
"""

from django.conf import settings
from django.core.exceptions import ImproperlyConfigured


def check_security_settings() -> list[str]:
    """Check Django security settings and return warnings."""
    warnings = []

    if settings.DEBUG:
        warnings.append('DEBUG is True - disable in production')

    if not settings.SECRET_KEY or settings.SECRET_KEY == 'insecure-dev-key':
        warnings.append('SECRET_KEY is not properly configured')

    if not settings.SECURE_SSL_REDIRECT:
        warnings.append('SECURE_SSL_REDIRECT should be True in production')

    if not settings.SESSION_COOKIE_SECURE:
        warnings.append('SESSION_COOKIE_SECURE should be True')

    if not settings.CSRF_COOKIE_SECURE:
        warnings.append('CSRF_COOKIE_SECURE should be True')

    if 'django.middleware.security.SecurityMiddleware' not in settings.MIDDLEWARE:
        warnings.append('SecurityMiddleware is not enabled')

    return warnings
```

---

## React/Next.js Stack

### Component Review

#### Before (Needs Improvement)

```tsx
// components/UserList.tsx

import { useState, useEffect } from 'react';

// Issue: No TypeScript types
// Issue: No error handling
// Issue: No loading state
// Issue: Inline styles
// Issue: No memoisation

export default function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => setUsers(data));
  }, []);

  return (
    <div style={{ padding: '20px' }}>
      {users.map((user: any) => (
        <div key={user.id} style={{ border: '1px solid black', margin: '10px' }}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
}
```

#### After (Improved)

```tsx
// components/UserList/UserList.tsx

'use client';

import { memo, useCallback } from 'react';
import { useUsers } from '@/hooks/useUsers';
import { UserCard } from './UserCard';
import { UserListSkeleton } from './UserListSkeleton';
import { ErrorMessage } from '@/components/ui/ErrorMessage';
import type { User } from '@/types';
import styles from './UserList.module.css';

interface UserListProps {
  initialUsers?: User[];
  onUserSelect?: (user: User) => void;
}

export const UserList = memo(function UserList({
  initialUsers,
  onUserSelect,
}: UserListProps) {
  const { users, isLoading, error, refetch } = useUsers({
    initialData: initialUsers,
  });

  const handleUserClick = useCallback(
    (user: User) => {
      onUserSelect?.(user);
    },
    [onUserSelect]
  );

  if (isLoading) {
    return <UserListSkeleton count={5} />;
  }

  if (error) {
    return (
      <ErrorMessage
        title="Failed to load users"
        message={error.message}
        onRetry={refetch}
      />
    );
  }

  if (!users?.length) {
    return (
      <div className={styles.empty}>
        <p>No users found</p>
      </div>
    );
  }

  return (
    <div className={styles.container} role="list" aria-label="User list">
      {users.map((user) => (
        <UserCard
          key={user.id}
          user={user}
          onClick={handleUserClick}
        />
      ))}
    </div>
  );
});
```

### Hook Review

#### Before (Needs Improvement)

```tsx
// hooks/useAuth.ts

import { useState } from 'react';

// Issue: No TypeScript
// Issue: No error handling
// Issue: Side effects in wrong place
// Issue: No token refresh

export function useAuth() {
  const [user, setUser] = useState(null);

  const login = async (email, password) => {
    const res = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
    const data = await res.json();
    localStorage.setItem('token', data.token); // Direct localStorage access
    setUser(data.user);
  };

  return { user, login };
}
```

#### After (Improved)

```tsx
// hooks/useAuth.ts

import { useCallback, useMemo } from 'react';
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { authApi } from '@/lib/api/auth';
import { tokenStorage } from '@/lib/storage/tokens';
import type { User, LoginCredentials, AuthState } from '@/types/auth';

interface UseAuthReturn extends AuthState {
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  refreshToken: () => Promise<void>;
}

export function useAuth(): UseAuthReturn {
  const queryClient = useQueryClient();

  const {
    data: user,
    isLoading,
    error,
  } = useQuery({
    queryKey: ['auth', 'user'],
    queryFn: authApi.getCurrentUser,
    retry: false,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  const loginMutation = useMutation({
    mutationFn: authApi.login,
    onSuccess: async (data) => {
      await tokenStorage.setTokens(data.accessToken, data.refreshToken);
      queryClient.setQueryData(['auth', 'user'], data.user);
    },
    onError: (error) => {
      console.error('Login failed:', error);
    },
  });

  const logoutMutation = useMutation({
    mutationFn: authApi.logout,
    onSettled: async () => {
      await tokenStorage.clearTokens();
      queryClient.clear();
    },
  });

  const refreshMutation = useMutation({
    mutationFn: async () => {
      const refreshToken = await tokenStorage.getRefreshToken();
      if (!refreshToken) throw new Error('No refresh token');
      return authApi.refreshToken(refreshToken);
    },
    onSuccess: async (data) => {
      await tokenStorage.setTokens(data.accessToken, data.refreshToken);
    },
  });

  const login = useCallback(
    async (credentials: LoginCredentials) => {
      await loginMutation.mutateAsync(credentials);
    },
    [loginMutation]
  );

  const logout = useCallback(async () => {
    await logoutMutation.mutateAsync();
  }, [logoutMutation]);

  const refreshToken = useCallback(async () => {
    await refreshMutation.mutateAsync();
  }, [refreshMutation]);

  const authState = useMemo(
    (): AuthState => ({
      user: user ?? null,
      isLoading,
      isAuthenticated: !!user,
      error: error instanceof Error ? error : null,
    }),
    [user, isLoading, error]
  );

  return {
    ...authState,
    login,
    logout,
    refreshToken,
  };
}
```

### Performance Review

```tsx
// Performance Review Checklist for React/Next.js

/**
 * PERFORMANCE REVIEW CHECKLIST
 *
 * 1. Component Rendering
 *    - [ ] Components memoised with React.memo where appropriate
 *    - [ ] Callbacks memoised with useCallback
 *    - [ ] Expensive computations memoised with useMemo
 *    - [ ] No unnecessary re-renders (check with React DevTools)
 *
 * 2. Data Fetching
 *    - [ ] Using React Query or SWR for caching
 *    - [ ] Appropriate stale times configured
 *    - [ ] Parallel fetching where possible
 *    - [ ] Proper loading and error states
 *
 * 3. Bundle Size
 *    - [ ] Dynamic imports for large components
 *    - [ ] Tree-shaking friendly imports
 *    - [ ] No unused dependencies
 *    - [ ] Images optimised with next/image
 *
 * 4. Core Web Vitals
 *    - [ ] LCP < 2.5s
 *    - [ ] FID < 100ms
 *    - [ ] CLS < 0.1
 */

// Example: Optimised component with proper memoisation
import { memo, useMemo, useCallback } from 'react';
import dynamic from 'next/dynamic';

// Lazy load heavy component
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,
});

interface DataTableProps {
  data: DataItem[];
  onSort: (column: string) => void;
  onFilter: (filters: Filter[]) => void;
}

export const DataTable = memo(function DataTable({
  data,
  onSort,
  onFilter,
}: DataTableProps) {
  // Memoise expensive computation
  const sortedData = useMemo(() => {
    return [...data].sort((a, b) => a.name.localeCompare(b.name));
  }, [data]);

  // Memoise callback to prevent child re-renders
  const handleRowClick = useCallback(
    (item: DataItem) => {
      console.log('Row clicked:', item.id);
    },
    []
  );

  return (
    <table>
      <tbody>
        {sortedData.map((item) => (
          <DataRow
            key={item.id}
            item={item}
            onClick={handleRowClick}
          />
        ))}
      </tbody>
    </table>
  );
});
```

---

## React Native Stack

### RN Component Review

#### Before (Needs Improvement)

```tsx
// components/ProductCard.tsx

import { View, Text, Image, TouchableOpacity } from 'react-native';

// Issue: No TypeScript
// Issue: Inline styles
// Issue: No accessibility
// Issue: No image optimisation

export default function ProductCard({ product, onPress }) {
  return (
    <TouchableOpacity onPress={() => onPress(product)}>
      <View style={{ padding: 10, backgroundColor: 'white' }}>
        <Image
          source={{ uri: product.image }}
          style={{ width: 100, height: 100 }}
        />
        <Text style={{ fontSize: 16, fontWeight: 'bold' }}>{product.name}</Text>
        <Text>{product.price}</Text>
      </View>
    </TouchableOpacity>
  );
}
```

#### After (Improved)

```tsx
// components/ProductCard/ProductCard.tsx

import React, { memo, useCallback } from 'react';
import {
  View,
  Text,
  Pressable,
  StyleSheet,
  AccessibilityProps,
} from 'react-native';
import { Image } from 'expo-image';
import type { Product } from '@/types';

interface ProductCardProps extends AccessibilityProps {
  product: Product;
  onPress: (product: Product) => void;
  testID?: string;
}

export const ProductCard = memo(function ProductCard({
  product,
  onPress,
  testID,
  ...accessibilityProps
}: ProductCardProps) {
  const handlePress = useCallback(() => {
    onPress(product);
  }, [product, onPress]);

  const formattedPrice = new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: 'GBP',
  }).format(product.price / 100);

  return (
    <Pressable
      onPress={handlePress}
      style={({ pressed }) => [
        styles.container,
        pressed && styles.pressed,
      ]}
      testID={testID}
      accessible
      accessibilityRole="button"
      accessibilityLabel={`${product.name}, ${formattedPrice}`}
      accessibilityHint="Double tap to view product details"
      {...accessibilityProps}
    >
      <Image
        source={{ uri: product.imageUrl }}
        style={styles.image}
        contentFit="cover"
        placeholder={product.blurhash}
        transition={200}
        cachePolicy="memory-disk"
      />
      <View style={styles.content}>
        <Text
          style={styles.name}
          numberOfLines={2}
          ellipsizeMode="tail"
        >
          {product.name}
        </Text>
        <Text style={styles.price}>{formattedPrice}</Text>
        {product.originalPrice && (
          <Text style={styles.originalPrice}>
            Was {new Intl.NumberFormat('en-GB', {
              style: 'currency',
              currency: 'GBP',
            }).format(product.originalPrice / 100)}
          </Text>
        )}
      </View>
    </Pressable>
  );
});

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#FFFFFF',
    borderRadius: 12,
    padding: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  pressed: {
    opacity: 0.9,
    transform: [{ scale: 0.98 }],
  },
  image: {
    width: '100%',
    aspectRatio: 1,
    borderRadius: 8,
    backgroundColor: '#F5F5F5',
  },
  content: {
    marginTop: 12,
  },
  name: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1A1A1A',
    lineHeight: 22,
  },
  price: {
    fontSize: 18,
    fontWeight: '700',
    color: '#2563EB',
    marginTop: 4,
  },
  originalPrice: {
    fontSize: 14,
    color: '#6B7280',
    textDecorationLine: 'line-through',
    marginTop: 2,
  },
});
```

### Native Module Review

```tsx
// Review checklist for native modules

/**
 * NATIVE MODULE REVIEW CHECKLIST
 *
 * 1. Error Handling
 *    - [ ] All native calls wrapped in try-catch
 *    - [ ] Errors properly propagated to JS
 *    - [ ] Fallbacks for unsupported platforms
 *
 * 2. Performance
 *    - [ ] Heavy operations run on background thread
 *    - [ ] Batch operations where possible
 *    - [ ] Memory properly managed (no leaks)
 *
 * 3. Type Safety
 *    - [ ] TypeScript definitions provided
 *    - [ ] Native types mapped correctly
 *    - [ ] Null/undefined handling
 *
 * 4. Platform Consistency
 *    - [ ] Behaviour consistent across iOS/Android
 *    - [ ] Platform-specific code isolated
 *    - [ ] Feature detection used
 */

// Example: Properly wrapped native module
import { NativeModules, Platform } from 'react-native';

interface BiometricModule {
  isAvailable(): Promise<boolean>;
  authenticate(reason: string): Promise<boolean>;
  getBiometricType(): Promise<'fingerprint' | 'face' | 'iris' | 'none'>;
}

const { BiometricAuth } = NativeModules as { BiometricAuth: BiometricModule };

export async function checkBiometricAvailability(): Promise<boolean> {
  try {
    if (!BiometricAuth) {
      console.warn('BiometricAuth module not available');
      return false;
    }
    return await BiometricAuth.isAvailable();
  } catch (error) {
    console.error('Failed to check biometric availability:', error);
    return false;
  }
}

export async function authenticateWithBiometrics(
  reason: string = 'Authenticate to continue'
): Promise<{ success: boolean; error?: string }> {
  try {
    if (!BiometricAuth) {
      return { success: false, error: 'Biometrics not supported' };
    }

    const isAvailable = await BiometricAuth.isAvailable();
    if (!isAvailable) {
      return { success: false, error: 'Biometrics not available' };
    }

    const success = await BiometricAuth.authenticate(reason);
    return { success };
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Authentication failed';
    return { success: false, error: message };
  }
}
```

### RN Performance Review

```tsx
// Performance Review Checklist for React Native

/**
 * PERFORMANCE REVIEW CHECKLIST
 *
 * 1. List Rendering
 *    - [ ] Using FlatList/FlashList for long lists
 *    - [ ] getItemLayout provided for fixed-height items
 *    - [ ] keyExtractor returns stable keys
 *    - [ ] renderItem properly memoised
 *
 * 2. Images
 *    - [ ] Using expo-image or fast-image
 *    - [ ] Images properly sized (not oversized)
 *    - [ ] Caching configured
 *    - [ ] Placeholders for loading states
 *
 * 3. Animations
 *    - [ ] Using Reanimated for smooth animations
 *    - [ ] useNativeDriver where possible
 *    - [ ] No layout thrashing during animations
 *
 * 4. JavaScript Performance
 *    - [ ] Expensive computations memoised
 *    - [ ] Large data transformations off main thread
 *    - [ ] Avoiding anonymous functions in render
 */

// Example: Optimised FlatList implementation
import React, { memo, useCallback, useMemo } from 'react';
import { FlatList, StyleSheet, View, ListRenderItem } from 'react-native';
import type { Product } from '@/types';
import { ProductCard } from './ProductCard';

interface ProductListProps {
  products: Product[];
  onProductPress: (product: Product) => void;
}

const ITEM_HEIGHT = 200;

export const ProductList = memo(function ProductList({
  products,
  onProductPress,
}: ProductListProps) {
  // Memoise keyExtractor
  const keyExtractor = useCallback((item: Product) => item.id, []);

  // Memoise getItemLayout for fixed-height items
  const getItemLayout = useCallback(
    (_: unknown, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  // Memoise renderItem
  const renderItem: ListRenderItem<Product> = useCallback(
    ({ item }) => (
      <ProductCard
        product={item}
        onPress={onProductPress}
      />
    ),
    [onProductPress]
  );

  // Memoise separator
  const ItemSeparator = useMemo(
    () => () => <View style={styles.separator} />,
    []
  );

  return (
    <FlatList
      data={products}
      keyExtractor={keyExtractor}
      renderItem={renderItem}
      getItemLayout={getItemLayout}
      ItemSeparatorComponent={ItemSeparator}
      removeClippedSubviews
      maxToRenderPerBatch={10}
      windowSize={5}
      initialNumToRender={5}
      contentContainerStyle={styles.container}
    />
  );
});

const styles = StyleSheet.create({
  container: {
    padding: 16,
  },
  separator: {
    height: 16,
  },
});
```
