# Refactoring Examples

## Overview

Common refactoring patterns with before/after examples for improving code quality without changing behaviour.

## Refactoring Patterns

| Pattern                 | Purpose             | When to Use                              |
| ----------------------- | ------------------- | ---------------------------------------- |
| Extract Method/Function | Reduce duplication  | Repeated code blocks                     |
| Extract Service/Hook    | Separate concerns   | Business logic in controllers/components |
| Replace Conditional     | Simplify branching  | Complex if/switch statements             |
| Introduce DTO/Type      | Type safety         | Passing multiple related values          |
| Composition             | Improve reusability | Large monolithic components              |

---

## Table of Contents

- [Overview](#overview)
- [Refactoring Patterns](#refactoring-patterns)
- [Table of Contents](#table-of-contents)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
  - [Laravel Extract Service](#laravel-extract-service)
    - [Before](#before)
    - [After](#after)
  - [Laravel Polymorphism](#laravel-polymorphism)
    - [Before](#before-1)
    - [After](#after-1)
  - [Laravel DTO](#laravel-dto)
    - [Before](#before-2)
    - [After](#after-2)
- [Django/Wagtail Stack](#djangowagtail-stack)
  - [Django Extract Service](#django-extract-service)
    - [Before](#before-3)
    - [After](#after-3)
  - [Django Strategy](#django-strategy)
    - [Before](#before-4)
    - [After](#after-4)
  - [Django Dataclass](#django-dataclass)
    - [Before](#before-5)
    - [After](#after-5)
- [React/Next.js Stack](#reactnextjs-stack)
  - [Extract Custom Hook](#extract-custom-hook)
    - [Before](#before-6)
    - [After](#after-6)
  - [Component Composition](#component-composition)
    - [Before](#before-7)
    - [After](#after-7)
  - [State Machine Refactor](#state-machine-refactor)
    - [Before](#before-8)
    - [After](#after-8)
- [React Native Stack](#react-native-stack)
  - [RN Extract Hook](#rn-extract-hook)
    - [Before](#before-9)
    - [After](#after-9)
  - [RN Performance](#rn-performance)
    - [Before](#before-10)
    - [After](#after-10)
  - [Platform-Specific](#platform-specific)
    - [Before](#before-11)
    - [After](#after-11)


## TALL Stack (Laravel 12)

### Laravel Extract Service

#### Before

```php
<?php
// app/Http/Controllers/OrderController.php

namespace App\Http\Controllers;

use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Mail;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // Validation mixed with business logic
        $validated = $request->validate([
            'items' => 'required|array',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
        ]);

        // All business logic in controller
        DB::beginTransaction();

        try {
            $total = 0;
            $orderItems = [];

            foreach ($validated['items'] as $item) {
                $product = Product::findOrFail($item['product_id']);

                if ($product->stock < $item['quantity']) {
                    throw new \Exception("Insufficient stock for {$product->name}");
                }

                $product->decrement('stock', $item['quantity']);

                $lineTotal = $product->price * $item['quantity'];
                $total += $lineTotal;

                $orderItems[] = [
                    'product_id' => $product->id,
                    'quantity' => $item['quantity'],
                    'price' => $product->price,
                    'total' => $lineTotal,
                ];
            }

            $order = Order::create([
                'user_id' => auth()->id(),
                'total' => $total,
                'status' => 'pending',
            ]);

            $order->items()->createMany($orderItems);

            // Email logic in controller
            Mail::to(auth()->user())->send(new OrderConfirmation($order));

            DB::commit();

            return response()->json($order->load('items'), 201);
        } catch (\Exception $e) {
            DB::rollBack();
            return response()->json(['error' => $e->getMessage()], 422);
        }
    }
}
```

#### After

```php
<?php
// app/Http/Controllers/OrderController.php

namespace App\Http\Controllers;

use App\Http\Requests\StoreOrderRequest;
use App\Http\Resources\OrderResource;
use App\Services\OrderService;
use Illuminate\Http\JsonResponse;

final class OrderController extends Controller
{
    public function __construct(
        private readonly OrderService $orderService
    ) {}

    public function store(StoreOrderRequest $request): JsonResponse
    {
        $order = $this->orderService->createOrder(
            user: $request->user(),
            items: $request->validated('items')
        );

        return OrderResource::make($order)
            ->response()
            ->setStatusCode(201);
    }
}
```

```php
<?php
// app/Services/OrderService.php

namespace App\Services;

use App\DTOs\OrderItemDTO;
use App\Events\OrderCreated;
use App\Exceptions\InsufficientStockException;
use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

final readonly class OrderService
{
    public function __construct(
        private StockService $stockService,
        private PricingService $pricingService
    ) {}

    /**
     * Create a new order for the user.
     *
     * @param array<array{product_id: int, quantity: int}> $items
     * @throws InsufficientStockException
     */
    public function createOrder(User $user, array $items): Order
    {
        return DB::transaction(function () use ($user, $items) {
            $orderItems = $this->prepareOrderItems($items);

            $order = Order::create([
                'user_id' => $user->id,
                'total' => $orderItems->sum('total'),
                'status' => 'pending',
            ]);

            $order->items()->createMany(
                $orderItems->map(fn(OrderItemDTO $item) => $item->toArray())->all()
            );

            event(new OrderCreated($order));

            return $order->load('items.product');
        });
    }

    /**
     * @return Collection<int, OrderItemDTO>
     */
    private function prepareOrderItems(array $items): Collection
    {
        return collect($items)->map(function (array $item) {
            $product = Product::findOrFail($item['product_id']);

            $this->stockService->reserveStock($product, $item['quantity']);

            return new OrderItemDTO(
                productId: $product->id,
                quantity: $item['quantity'],
                price: $product->price,
                total: $this->pricingService->calculateLineTotal($product, $item['quantity'])
            );
        });
    }
}
```

### Laravel Polymorphism

#### Before

```php
<?php
// app/Services/NotificationService.php

class NotificationService
{
    public function send(User $user, string $type, array $data): void
    {
        // Complex conditional logic
        switch ($type) {
            case 'email':
                Mail::to($user->email)->send(new GenericNotification($data));
                break;

            case 'sms':
                $client = new TwilioClient();
                $client->messages->create($user->phone, [
                    'from' => config('services.twilio.from'),
                    'body' => $data['message'],
                ]);
                break;

            case 'push':
                $expo = new ExpoClient();
                $expo->send([
                    'to' => $user->push_token,
                    'title' => $data['title'],
                    'body' => $data['message'],
                ]);
                break;

            case 'slack':
                Http::post($user->slack_webhook, [
                    'text' => $data['message'],
                ]);
                break;

            default:
                throw new InvalidArgumentException("Unknown notification type: {$type}");
        }
    }
}
```

#### After

```php
<?php
// app/Contracts/NotificationChannel.php

namespace App\Contracts;

use App\Models\User;

interface NotificationChannel
{
    public function send(User $user, array $data): void;

    public function supports(User $user): bool;
}
```

```php
<?php
// app/Notifications/Channels/EmailChannel.php

namespace App\Notifications\Channels;

use App\Contracts\NotificationChannel;
use App\Mail\GenericNotification;
use App\Models\User;
use Illuminate\Support\Facades\Mail;

final readonly class EmailChannel implements NotificationChannel
{
    public function send(User $user, array $data): void
    {
        Mail::to($user->email)->send(new GenericNotification($data));
    }

    public function supports(User $user): bool
    {
        return !empty($user->email) && $user->email_notifications_enabled;
    }
}
```

```php
<?php
// app/Notifications/Channels/SmsChannel.php

namespace App\Notifications\Channels;

use App\Contracts\NotificationChannel;
use App\Models\User;
use Twilio\Rest\Client;

final readonly class SmsChannel implements NotificationChannel
{
    public function __construct(
        private Client $twilioClient
    ) {}

    public function send(User $user, array $data): void
    {
        $this->twilioClient->messages->create($user->phone, [
            'from' => config('services.twilio.from'),
            'body' => $data['message'],
        ]);
    }

    public function supports(User $user): bool
    {
        return !empty($user->phone) && $user->sms_notifications_enabled;
    }
}
```

```php
<?php
// app/Services/NotificationService.php

namespace App\Services;

use App\Contracts\NotificationChannel;
use App\Models\User;

final readonly class NotificationService
{
    /**
     * @param iterable<NotificationChannel> $channels
     */
    public function __construct(
        private iterable $channels
    ) {}

    public function send(User $user, array $data, ?array $channelTypes = null): void
    {
        foreach ($this->channels as $channel) {
            if ($channelTypes !== null && !in_array($channel::class, $channelTypes, true)) {
                continue;
            }

            if ($channel->supports($user)) {
                $channel->send($user, $data);
            }
        }
    }
}
```

### Laravel DTO

#### Before

```php
<?php
// Passing arrays around - no type safety

$userData = [
    'name' => $request->input('name'),
    'email' => $request->input('email'),
    'password' => $request->input('password'),
    'role' => $request->input('role', 'user'),
];

$user = $userService->createUser($userData);
```

#### After

```php
<?php
// app/DTOs/CreateUserDTO.php

namespace App\DTOs;

use App\Enums\UserRole;

final readonly class CreateUserDTO
{
    public function __construct(
        public string $name,
        public string $email,
        public string $password,
        public UserRole $role = UserRole::USER,
        public ?string $phone = null,
        public ?array $preferences = null,
    ) {}

    public static function fromRequest(Request $request): self
    {
        return new self(
            name: $request->validated('name'),
            email: $request->validated('email'),
            password: $request->validated('password'),
            role: UserRole::tryFrom($request->validated('role')) ?? UserRole::USER,
            phone: $request->validated('phone'),
            preferences: $request->validated('preferences'),
        );
    }

    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'email' => $this->email,
            'password' => Hash::make($this->password),
            'role' => $this->role->value,
            'phone' => $this->phone,
            'preferences' => $this->preferences,
        ];
    }
}
```

```php
<?php
// Usage with full type safety

$dto = CreateUserDTO::fromRequest($request);
$user = $userService->createUser($dto);
```

---

## Django/Wagtail Stack

### Django Extract Service

#### Before

```python
# apps/orders/views.py

from django.db import transaction
from rest_framework.views import APIView

class OrderView(APIView):
    def post(self, request):
        # All logic in view
        items_data = request.data.get('items', [])

        with transaction.atomic():
            total = Decimal('0')
            order_items = []

            for item in items_data:
                product = Product.objects.get(id=item['product_id'])

                if product.stock < item['quantity']:
                    return Response(
                        {'error': f'Insufficient stock for {product.name}'},
                        status=400
                    )

                product.stock -= item['quantity']
                product.save()

                line_total = product.price * item['quantity']
                total += line_total

                order_items.append({
                    'product': product,
                    'quantity': item['quantity'],
                    'price': product.price,
                })

            order = Order.objects.create(
                user=request.user,
                total=total,
                status='pending',
            )

            for item in order_items:
                OrderItem.objects.create(order=order, **item)

            # Send email
            send_mail(
                'Order Confirmation',
                f'Your order #{order.id} has been placed.',
                'noreply@example.com',
                [request.user.email],
            )

            return Response(OrderSerializer(order).data, status=201)
```

#### After

```python
# apps/orders/views.py

from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView

from .serializers import CreateOrderSerializer, OrderSerializer
from .services import OrderService


class OrderView(APIView):
    """API view for order operations."""

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.order_service = OrderService()

    def post(self, request) -> Response:
        """Create a new order."""
        serializer = CreateOrderSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        order = self.order_service.create_order(
            user=request.user,
            items=serializer.validated_data['items'],
        )

        return Response(
            OrderSerializer(order).data,
            status=status.HTTP_201_CREATED,
        )
```

```python
# apps/orders/services.py

from decimal import Decimal
from typing import Any

from django.db import transaction

from apps.orders.models import Order, OrderItem
from apps.orders.events import order_created
from apps.products.models import Product
from apps.products.services import StockService


class OrderService:
    """Service for order business logic."""

    def __init__(self):
        self.stock_service = StockService()

    @transaction.atomic
    def create_order(
        self,
        user: 'User',
        items: list[dict[str, Any]],
    ) -> Order:
        """Create a new order with the given items."""
        order_items = self._prepare_order_items(items)
        total = sum(item['total'] for item in order_items)

        order = Order.objects.create(
            user=user,
            total=total,
            status='pending',
        )

        for item in order_items:
            OrderItem.objects.create(order=order, **item)

        order_created.send(sender=self.__class__, order=order)

        return order

    def _prepare_order_items(
        self,
        items: list[dict[str, Any]],
    ) -> list[dict[str, Any]]:
        """Prepare order items with stock validation."""
        order_items = []

        for item in items:
            product = Product.objects.select_for_update().get(
                id=item['product_id']
            )

            self.stock_service.reserve_stock(product, item['quantity'])

            order_items.append({
                'product': product,
                'quantity': item['quantity'],
                'price': product.price,
                'total': product.price * item['quantity'],
            })

        return order_items
```

### Django Strategy

#### Before

```python
# apps/payments/services.py

class PaymentService:
    def process_payment(self, order, method: str, details: dict):
        if method == 'stripe':
            stripe.api_key = settings.STRIPE_SECRET_KEY
            charge = stripe.Charge.create(
                amount=int(order.total * 100),
                currency='gbp',
                source=details['token'],
            )
            return {'id': charge.id, 'status': charge.status}

        elif method == 'paypal':
            client = PayPalClient()
            result = client.create_order(
                intent='CAPTURE',
                purchase_units=[{
                    'amount': {'value': str(order.total), 'currency_code': 'GBP'}
                }],
            )
            return {'id': result.id, 'status': result.status}

        elif method == 'bank_transfer':
            reference = f'ORDER-{order.id}'
            return {'reference': reference, 'status': 'pending'}

        else:
            raise ValueError(f'Unknown payment method: {method}')
```

#### After

```python
# apps/payments/strategies/base.py

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Any

from apps.orders.models import Order


@dataclass
class PaymentResult:
    """Result of a payment attempt."""
    transaction_id: str
    status: str
    raw_response: dict[str, Any] | None = None


class PaymentStrategy(ABC):
    """Abstract base class for payment strategies."""

    @abstractmethod
    def process(self, order: Order, details: dict[str, Any]) -> PaymentResult:
        """Process the payment."""
        ...

    @abstractmethod
    def refund(self, transaction_id: str, amount: int) -> PaymentResult:
        """Refund a payment."""
        ...
```

```python
# apps/payments/strategies/stripe.py

import stripe
from django.conf import settings

from .base import PaymentStrategy, PaymentResult


class StripePaymentStrategy(PaymentStrategy):
    """Stripe payment strategy."""

    def __init__(self):
        stripe.api_key = settings.STRIPE_SECRET_KEY

    def process(self, order, details: dict) -> PaymentResult:
        charge = stripe.Charge.create(
            amount=int(order.total * 100),
            currency='gbp',
            source=details['token'],
            metadata={'order_id': order.id},
        )

        return PaymentResult(
            transaction_id=charge.id,
            status=charge.status,
            raw_response=dict(charge),
        )

    def refund(self, transaction_id: str, amount: int) -> PaymentResult:
        refund = stripe.Refund.create(
            charge=transaction_id,
            amount=amount,
        )

        return PaymentResult(
            transaction_id=refund.id,
            status=refund.status,
            raw_response=dict(refund),
        )
```

```python
# apps/payments/services.py

from typing import Type

from .strategies.base import PaymentStrategy, PaymentResult
from .strategies.stripe import StripePaymentStrategy
from .strategies.paypal import PayPalPaymentStrategy
from .strategies.bank_transfer import BankTransferStrategy


class PaymentService:
    """Service for processing payments."""

    STRATEGIES: dict[str, Type[PaymentStrategy]] = {
        'stripe': StripePaymentStrategy,
        'paypal': PayPalPaymentStrategy,
        'bank_transfer': BankTransferStrategy,
    }

    def process_payment(
        self,
        order: 'Order',
        method: str,
        details: dict,
    ) -> PaymentResult:
        """Process payment using the specified method."""
        strategy_class = self.STRATEGIES.get(method)

        if not strategy_class:
            raise ValueError(f'Unknown payment method: {method}')

        strategy = strategy_class()
        return strategy.process(order, details)
```

### Django Dataclass

#### Before

```python
# Passing dicts - no type safety

user_data = {
    'email': request.data.get('email'),
    'password': request.data.get('password'),
    'first_name': request.data.get('first_name'),
    'last_name': request.data.get('last_name'),
}

user = user_service.create_user(user_data)
```

#### After

```python
# apps/users/dtos.py

from dataclasses import dataclass, field
from enum import Enum
from typing import Any


class UserRole(str, Enum):
    USER = 'user'
    ADMIN = 'admin'
    MODERATOR = 'moderator'


@dataclass(frozen=True)
class CreateUserDTO:
    """Data transfer object for user creation."""

    email: str
    password: str
    first_name: str
    last_name: str
    role: UserRole = UserRole.USER
    phone: str | None = None
    preferences: dict[str, Any] = field(default_factory=dict)

    @classmethod
    def from_request(cls, data: dict[str, Any]) -> 'CreateUserDTO':
        """Create DTO from request data."""
        return cls(
            email=data['email'],
            password=data['password'],
            first_name=data['first_name'],
            last_name=data['last_name'],
            role=UserRole(data.get('role', 'user')),
            phone=data.get('phone'),
            preferences=data.get('preferences', {}),
        )

    def to_dict(self) -> dict[str, Any]:
        """Convert to dictionary for model creation."""
        return {
            'email': self.email,
            'first_name': self.first_name,
            'last_name': self.last_name,
            'role': self.role.value,
            'phone': self.phone,
            'preferences': self.preferences,
        }
```

```python
# Usage with full type safety

dto = CreateUserDTO.from_request(request.data)
user = user_service.create_user(dto)
```

---

## React/Next.js Stack

### Extract Custom Hook

#### Before

```tsx
// components/UserProfile.tsx

import { useState, useEffect } from 'react';

export function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  const updateUser = async (updates) => {
    try {
      const response = await fetch(`/api/users/${userId}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates),
      });
      if (!response.ok) throw new Error('Failed to update');
      const data = await response.json();
      setUser(data);
    } catch (err) {
      setError(err.message);
    }
  };

  if (loading) return <Loading />;
  if (error) return <Error message={error} />;

  return (
    <div>
      <h1>{user.name}</h1>
      {/* ... */}
    </div>
  );
}
```

#### After

```tsx
// hooks/useUser.ts

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import type { User, UpdateUserInput } from '@/types';

async function fetchUser(userId: string): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) throw new Error('Failed to fetch user');
  return response.json();
}

async function updateUser(userId: string, updates: UpdateUserInput): Promise<User> {
  const response = await fetch(`/api/users/${userId}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(updates),
  });
  if (!response.ok) throw new Error('Failed to update user');
  return response.json();
}

export function useUser(userId: string) {
  const queryClient = useQueryClient();

  const query = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000,
  });

  const mutation = useMutation({
    mutationFn: (updates: UpdateUserInput) => updateUser(userId, updates),
    onSuccess: (updatedUser) => {
      queryClient.setQueryData(['user', userId], updatedUser);
    },
  });

  return {
    user: query.data,
    isLoading: query.isLoading,
    error: query.error,
    updateUser: mutation.mutate,
    isUpdating: mutation.isPending,
  };
}
```

```tsx
// components/UserProfile.tsx

import { useUser } from '@/hooks/useUser';
import { Loading, ErrorMessage } from '@/components/ui';

interface UserProfileProps {
  userId: string;
}

export function UserProfile({ userId }: UserProfileProps) {
  const { user, isLoading, error, updateUser, isUpdating } = useUser(userId);

  if (isLoading) return <Loading />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      {/* Much cleaner component */}
    </div>
  );
}
```

### Component Composition

#### Before

```tsx
// components/ProductCard.tsx - Monolithic component

export function ProductCard({ product, variant, showQuickView, showWishlist }) {
  // 200+ lines of conditional rendering
  return (
    <div className={variant === 'compact' ? 'compact' : 'full'}>
      {variant === 'full' && (
        <div className="image-container">
          {/* Image logic */}
        </div>
      )}
      {variant === 'compact' && (
        <div className="small-image">
          {/* Different image logic */}
        </div>
      )}

      <h3>{product.name}</h3>

      {variant === 'full' && <p>{product.description}</p>}

      <span>{product.price}</span>

      {showWishlist && (
        <button onClick={handleWishlist}>
          {/* Wishlist logic */}
        </button>
      )}

      {showQuickView && variant === 'full' && (
        <button onClick={handleQuickView}>
          {/* Quick view logic */}
        </button>
      )}

      {/* More conditionals... */}
    </div>
  );
}
```

#### After

```tsx
// components/ProductCard/index.tsx

import { createContext, useContext, memo } from 'react';
import type { Product } from '@/types';

interface ProductCardContextValue {
  product: Product;
}

const ProductCardContext = createContext<ProductCardContextValue | null>(null);

function useProductCard() {
  const context = useContext(ProductCardContext);
  if (!context) throw new Error('Must be used within ProductCard');
  return context;
}

interface ProductCardProps {
  product: Product;
  children: React.ReactNode;
  className?: string;
}

export const ProductCard = memo(function ProductCard({
  product,
  children,
  className,
}: ProductCardProps) {
  return (
    <ProductCardContext.Provider value={{ product }}>
      <article className={className}>{children}</article>
    </ProductCardContext.Provider>
  );
});

// Composable sub-components
ProductCard.Image = function ProductImage({ size = 'medium' }) {
  const { product } = useProductCard();
  return <img src={product.imageUrl} alt={product.name} className={size} />;
};

ProductCard.Title = function ProductTitle() {
  const { product } = useProductCard();
  return <h3>{product.name}</h3>;
};

ProductCard.Description = function ProductDescription() {
  const { product } = useProductCard();
  return <p>{product.description}</p>;
};

ProductCard.Price = function ProductPrice() {
  const { product } = useProductCard();
  const formatted = new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: 'GBP',
  }).format(product.price / 100);
  return <span className="price">{formatted}</span>;
};

ProductCard.WishlistButton = function WishlistButton() {
  const { product } = useProductCard();
  // Wishlist logic isolated here
  return <button aria-label="Add to wishlist">Save</button>;
};

ProductCard.QuickView = function QuickView() {
  const { product } = useProductCard();
  // Quick view logic isolated here
  return <button>Quick View</button>;
};
```

```tsx
// Usage - Compose what you need

// Full card
<ProductCard product={product}>
  <ProductCard.Image size="large" />
  <ProductCard.Title />
  <ProductCard.Description />
  <ProductCard.Price />
  <ProductCard.WishlistButton />
  <ProductCard.QuickView />
</ProductCard>

// Compact card
<ProductCard product={product} className="compact">
  <ProductCard.Image size="small" />
  <ProductCard.Title />
  <ProductCard.Price />
</ProductCard>
```

### State Machine Refactor

#### Before

```tsx
// Complex boolean state management

function CheckoutForm() {
  const [isLoading, setIsLoading] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);
  const [isError, setIsError] = useState(false);
  const [error, setError] = useState(null);
  const [isValidating, setIsValidating] = useState(false);

  // Impossible states are possible (isLoading && isSuccess)
  // Easy to forget to reset states
}
```

#### After

```tsx
// hooks/useCheckoutMachine.ts

import { useMachine } from '@xstate/react';
import { createMachine, assign } from 'xstate';

type CheckoutContext = {
  error: string | null;
  orderId: string | null;
};

type CheckoutEvent =
  | { type: 'VALIDATE' }
  | { type: 'SUBMIT'; data: FormData }
  | { type: 'SUCCESS'; orderId: string }
  | { type: 'ERROR'; error: string }
  | { type: 'RETRY' };

const checkoutMachine = createMachine<CheckoutContext, CheckoutEvent>({
  id: 'checkout',
  initial: 'idle',
  context: {
    error: null,
    orderId: null,
  },
  states: {
    idle: {
      on: {
        VALIDATE: 'validating',
      },
    },
    validating: {
      invoke: {
        src: 'validateForm',
        onDone: 'ready',
        onError: {
          target: 'invalid',
          actions: assign({ error: (_, event) => event.data.message }),
        },
      },
    },
    invalid: {
      on: {
        VALIDATE: 'validating',
      },
    },
    ready: {
      on: {
        SUBMIT: 'submitting',
      },
    },
    submitting: {
      invoke: {
        src: 'submitOrder',
        onDone: {
          target: 'success',
          actions: assign({ orderId: (_, event) => event.data.orderId }),
        },
        onError: {
          target: 'error',
          actions: assign({ error: (_, event) => event.data.message }),
        },
      },
    },
    success: {
      type: 'final',
    },
    error: {
      on: {
        RETRY: 'ready',
      },
    },
  },
});

export function useCheckout() {
  const [state, send] = useMachine(checkoutMachine);

  return {
    // Clear state checks - no impossible states
    isIdle: state.matches('idle'),
    isValidating: state.matches('validating'),
    isReady: state.matches('ready'),
    isSubmitting: state.matches('submitting'),
    isSuccess: state.matches('success'),
    isError: state.matches('error'),

    error: state.context.error,
    orderId: state.context.orderId,

    validate: () => send({ type: 'VALIDATE' }),
    submit: (data: FormData) => send({ type: 'SUBMIT', data }),
    retry: () => send({ type: 'RETRY' }),
  };
}
```

---

## React Native Stack

### RN Extract Hook

#### Before

```tsx
// screens/ProductScreen.tsx

import { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator, Alert } from 'react-native';

export function ProductScreen({ route }) {
  const { productId } = route.params;
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [addingToCart, setAddingToCart] = useState(false);

  useEffect(() => {
    fetch(`${API_URL}/products/${productId}`)
      .then(res => res.json())
      .then(data => setProduct(data))
      .catch(err => Alert.alert('Error', err.message))
      .finally(() => setLoading(false));
  }, [productId]);

  const addToCart = async () => {
    setAddingToCart(true);
    try {
      await fetch(`${API_URL}/cart`, {
        method: 'POST',
        body: JSON.stringify({ productId, quantity: 1 }),
      });
      Alert.alert('Success', 'Added to cart');
    } catch (err) {
      Alert.alert('Error', err.message);
    } finally {
      setAddingToCart(false);
    }
  };

  // Component with mixed concerns
}
```

#### After

```tsx
// hooks/useProduct.ts

import { useQuery } from '@tanstack/react-query';
import { api } from '@/lib/api';
import type { Product } from '@/types';

export function useProduct(productId: string) {
  return useQuery({
    queryKey: ['product', productId],
    queryFn: () => api.get<Product>(`/products/${productId}`),
    staleTime: 5 * 60 * 1000,
  });
}
```

```tsx
// hooks/useCart.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/api';

interface AddToCartInput {
  productId: string;
  quantity: number;
}

export function useAddToCart() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (input: AddToCartInput) => api.post('/cart', input),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });
}
```

```tsx
// screens/ProductScreen.tsx

import { View, Text, Pressable, ActivityIndicator } from 'react-native';
import { useProduct } from '@/hooks/useProduct';
import { useAddToCart } from '@/hooks/useCart';
import { showToast } from '@/lib/toast';

export function ProductScreen({ route }) {
  const { productId } = route.params;
  const { data: product, isLoading, error } = useProduct(productId);
  const addToCart = useAddToCart();

  const handleAddToCart = () => {
    addToCart.mutate(
      { productId, quantity: 1 },
      {
        onSuccess: () => showToast('Added to cart'),
        onError: (err) => showToast(err.message, 'error'),
      }
    );
  };

  if (isLoading) return <ActivityIndicator />;
  if (error) return <ErrorView error={error} />;
  if (!product) return null;

  return (
    <View>
      <Text>{product.name}</Text>
      <Pressable onPress={handleAddToCart} disabled={addToCart.isPending}>
        <Text>{addToCart.isPending ? 'Adding...' : 'Add to Cart'}</Text>
      </Pressable>
    </View>
  );
}
```

### RN Performance

#### Before

```tsx
// Inefficient list rendering

export function ProductList({ products }) {
  return (
    <ScrollView>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onPress={() => navigate('Product', { id: product.id })}
        />
      ))}
    </ScrollView>
  );
}
```

#### After

```tsx
// components/ProductList.tsx

import { memo, useCallback, useMemo } from 'react';
import { FlatList, StyleSheet, View } from 'react-native';
import type { Product } from '@/types';
import { ProductCard } from './ProductCard';

interface ProductListProps {
  products: Product[];
  onProductPress: (product: Product) => void;
}

const ITEM_HEIGHT = 120;

export const ProductList = memo(function ProductList({
  products,
  onProductPress,
}: ProductListProps) {
  const keyExtractor = useCallback((item: Product) => item.id, []);

  const getItemLayout = useCallback(
    (_: unknown, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  const renderItem = useCallback(
    ({ item }: { item: Product }) => (
      <ProductCard product={item} onPress={onProductPress} />
    ),
    [onProductPress]
  );

  const ItemSeparator = useMemo(
    () =>
      function Separator() {
        return <View style={styles.separator} />;
      },
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
      initialNumToRender={8}
    />
  );
});

const styles = StyleSheet.create({
  separator: {
    height: 12,
  },
});
```

### Platform-Specific

#### Before

```tsx
// Messy platform conditionals everywhere

import { Platform, StyleSheet } from 'react-native';

export function Button({ children }) {
  return (
    <Pressable
      style={[
        styles.button,
        Platform.OS === 'ios' ? styles.iosButton : styles.androidButton,
      ]}
      android_ripple={Platform.OS === 'android' ? { color: 'rgba(0,0,0,0.1)' } : undefined}
    >
      <Text
        style={[
          styles.text,
          Platform.OS === 'ios' ? { fontWeight: '600' } : { fontWeight: 'bold' },
        ]}
      >
        {children}
      </Text>
    </Pressable>
  );
}
```

#### After

```tsx
// components/Button/Button.tsx

import { Pressable, Text, StyleSheet, PressableProps } from 'react-native';

export interface ButtonProps extends Omit<PressableProps, 'style'> {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
}

export function Button({ children, variant = 'primary', ...props }: ButtonProps) {
  return (
    <Pressable
      style={({ pressed }) => [
        styles.base,
        styles[variant],
        pressed && styles.pressed,
      ]}
      {...props}
    >
      <Text style={[styles.text, styles[`${variant}Text`]]}>{children}</Text>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  base: {
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  primary: {
    backgroundColor: '#2563EB',
  },
  secondary: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#2563EB',
  },
  pressed: {
    opacity: 0.8,
  },
  text: {
    fontSize: 16,
  },
  primaryText: {
    color: '#FFFFFF',
  },
  secondaryText: {
    color: '#2563EB',
  },
});
```

```tsx
// components/Button/Button.ios.tsx

import { Pressable, Text, StyleSheet } from 'react-native';
import { ButtonProps } from './Button';

export function Button({ children, variant = 'primary', ...props }: ButtonProps) {
  return (
    <Pressable
      style={({ pressed }) => [
        styles.base,
        styles[variant],
        pressed && styles.pressed,
      ]}
      {...props}
    >
      <Text style={[styles.text, styles[`${variant}Text`]]}>{children}</Text>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  base: {
    paddingVertical: 14,
    paddingHorizontal: 28,
    borderRadius: 12,
    alignItems: 'center',
  },
  primary: {
    backgroundColor: '#007AFF', // iOS blue
  },
  secondary: {
    backgroundColor: 'transparent',
  },
  pressed: {
    opacity: 0.7,
  },
  text: {
    fontSize: 17,
    fontWeight: '600',
  },
  primaryText: {
    color: '#FFFFFF',
  },
  secondaryText: {
    color: '#007AFF',
  },
});
```

```tsx
// components/Button/Button.android.tsx

import { Pressable, Text, StyleSheet } from 'react-native';
import { ButtonProps } from './Button';

export function Button({ children, variant = 'primary', ...props }: ButtonProps) {
  return (
    <Pressable
      style={[styles.base, styles[variant]]}
      android_ripple={{
        color: variant === 'primary' ? 'rgba(255,255,255,0.3)' : 'rgba(37,99,235,0.2)',
      }}
      {...props}
    >
      <Text style={[styles.text, styles[`${variant}Text`]]}>{children}</Text>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  base: {
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 4, // Material Design
    alignItems: 'center',
    elevation: 2,
  },
  primary: {
    backgroundColor: '#2563EB',
  },
  secondary: {
    backgroundColor: 'transparent',
    elevation: 0,
  },
  text: {
    fontSize: 14,
    fontWeight: 'bold',
    textTransform: 'uppercase',
    letterSpacing: 1,
  },
  primaryText: {
    color: '#FFFFFF',
  },
  secondaryText: {
    color: '#2563EB',
  },
});
```
