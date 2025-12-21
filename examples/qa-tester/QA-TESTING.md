# QA Testing Examples

## Overview

QA testing examples covering functional, integration, security, and accessibility testing across all technology stacks.

## Test Categories

| Category | Purpose | Tools |
|----------|---------|-------|
| Functional | Verify features work correctly | Pest, Pytest, Playwright, Detox |
| Integration | Test component interactions | API tests, E2E tests |
| Security | Find vulnerabilities | OWASP ZAP, Security scanners |
| Accessibility | Ensure WCAG compliance | axe, Lighthouse |
| Performance | Measure and optimise speed | k6, Artillery, Lighthouse |

---

## Table of Contents

- [Overview](#overview)
- [Test Categories](#test-categories)
- [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
- [Django/Wagtail Stack](#djangowagtail-stack)
- [React/Next.js Stack](#reactnextjs-stack)
- [React Native Stack](#react-native-stack)

## TALL Stack (Laravel 12)

### Laravel Functional Testing

```php
<?php
// tests/Feature/OrderProcessTest.php

use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use App\Enums\OrderStatus;
use Illuminate\Support\Facades\Queue;

describe('Order Process', function () {
    describe('Creating an order', function () {
        it('creates order with valid products', function () {
            $user = User::factory()->create();
            $products = Product::factory(3)->create(['stock' => 10]);

            $response = $this->actingAs($user)->postJson('/api/orders', [
                'items' => $products->map(fn($p) => [
                    'product_id' => $p->id,
                    'quantity' => 2,
                ])->toArray(),
                'shipping_address' => [
                    'line1' => '123 Test Street',
                    'city' => 'London',
                    'postcode' => 'SW1A 1AA',
                    'country' => 'GB',
                ],
            ]);

            $response->assertCreated()
                ->assertJsonStructure([
                    'data' => [
                        'id',
                        'status',
                        'total',
                        'items' => [
                            '*' => ['product_id', 'quantity', 'price'],
                        ],
                    ],
                ]);

            // Verify stock reduced
            $products->each(function ($product) {
                expect($product->fresh()->stock)->toBe(8);
            });
        });

        it('rejects order with insufficient stock', function () {
            $user = User::factory()->create();
            $product = Product::factory()->create(['stock' => 1]);

            $response = $this->actingAs($user)->postJson('/api/orders', [
                'items' => [
                    ['product_id' => $product->id, 'quantity' => 5],
                ],
                'shipping_address' => [
                    'line1' => '123 Test Street',
                    'city' => 'London',
                    'postcode' => 'SW1A 1AA',
                    'country' => 'GB',
                ],
            ]);

            $response->assertUnprocessable()
                ->assertJsonPath('errors.items.0', 'Insufficient stock for ' . $product->name);
        });

        it('requires authentication', function () {
            $product = Product::factory()->create();

            $response = $this->postJson('/api/orders', [
                'items' => [
                    ['product_id' => $product->id, 'quantity' => 1],
                ],
            ]);

            $response->assertUnauthorized();
        });
    });

    describe('Order status transitions', function () {
        it('allows valid status transitions', function () {
            $order = Order::factory()->create(['status' => OrderStatus::PENDING]);

            $response = $this->actingAs($order->user)->patchJson("/api/orders/{$order->id}/status", [
                'status' => 'confirmed',
            ]);

            $response->assertOk();
            expect($order->fresh()->status)->toBe(OrderStatus::CONFIRMED);
        });

        it('rejects invalid status transitions', function () {
            $order = Order::factory()->create(['status' => OrderStatus::DELIVERED]);

            $response = $this->actingAs($order->user)->patchJson("/api/orders/{$order->id}/status", [
                'status' => 'pending',
            ]);

            $response->assertUnprocessable()
                ->assertJsonPath('message', 'Cannot transition from delivered to pending');
        });
    });

    describe('Order cancellation', function () {
        it('refunds payment when cancelled', function () {
            Queue::fake();

            $order = Order::factory()
                ->paid()
                ->create(['status' => OrderStatus::CONFIRMED]);

            $response = $this->actingAs($order->user)->deleteJson("/api/orders/{$order->id}");

            $response->assertNoContent();
            expect($order->fresh()->status)->toBe(OrderStatus::CANCELLED);

            Queue::assertPushed(\App\Jobs\ProcessRefund::class);
        });

        it('restores stock when cancelled', function () {
            $product = Product::factory()->create(['stock' => 5]);
            $order = Order::factory()
                ->hasItems(1, ['product_id' => $product->id, 'quantity' => 3])
                ->create(['status' => OrderStatus::CONFIRMED]);

            $this->actingAs($order->user)->deleteJson("/api/orders/{$order->id}");

            expect($product->fresh()->stock)->toBe(8);
        });
    });
});
```

### Laravel API Testing

```php
<?php
// tests/Feature/Api/ProductApiTest.php

use App\Models\Product;
use App\Models\User;

describe('Product API', function () {
    describe('GET /api/products', function () {
        it('returns paginated products', function () {
            Product::factory(25)->create();

            $response = $this->getJson('/api/products');

            $response->assertOk()
                ->assertJsonCount(15, 'data')
                ->assertJsonStructure([
                    'data' => [
                        '*' => ['id', 'name', 'price', 'description'],
                    ],
                    'meta' => ['current_page', 'last_page', 'per_page', 'total'],
                    'links' => ['first', 'last', 'prev', 'next'],
                ]);
        });

        it('filters by category', function () {
            Product::factory(5)->create(['category' => 'electronics']);
            Product::factory(3)->create(['category' => 'clothing']);

            $response = $this->getJson('/api/products?category=electronics');

            $response->assertOk()
                ->assertJsonCount(5, 'data');

            collect($response->json('data'))->each(function ($product) {
                expect($product['category'])->toBe('electronics');
            });
        });

        it('searches by name', function () {
            Product::factory()->create(['name' => 'Blue Widget']);
            Product::factory()->create(['name' => 'Red Widget']);
            Product::factory()->create(['name' => 'Green Gadget']);

            $response = $this->getJson('/api/products?search=widget');

            $response->assertOk()
                ->assertJsonCount(2, 'data');
        });

        it('sorts by price', function () {
            Product::factory()->create(['price' => 5000]);
            Product::factory()->create(['price' => 1000]);
            Product::factory()->create(['price' => 3000]);

            $response = $this->getJson('/api/products?sort=price&direction=asc');

            $prices = collect($response->json('data'))->pluck('price');
            expect($prices->toArray())->toBe([1000, 3000, 5000]);
        });
    });

    describe('POST /api/products', function () {
        it('creates product as admin', function () {
            $admin = User::factory()->admin()->create();

            $response = $this->actingAs($admin)->postJson('/api/products', [
                'name' => 'New Product',
                'description' => 'A great product',
                'price' => 2999,
                'stock' => 100,
                'category' => 'electronics',
            ]);

            $response->assertCreated()
                ->assertJsonPath('data.name', 'New Product');

            $this->assertDatabaseHas('products', ['name' => 'New Product']);
        });

        it('rejects creation by non-admin', function () {
            $user = User::factory()->create();

            $response = $this->actingAs($user)->postJson('/api/products', [
                'name' => 'New Product',
                'price' => 2999,
            ]);

            $response->assertForbidden();
        });

        it('validates required fields', function () {
            $admin = User::factory()->admin()->create();

            $response = $this->actingAs($admin)->postJson('/api/products', []);

            $response->assertUnprocessable()
                ->assertJsonValidationErrors(['name', 'price', 'stock']);
        });
    });
});
```

### Laravel Security Testing

```php
<?php
// tests/Security/AuthenticationSecurityTest.php

use App\Models\User;
use Illuminate\Support\Facades\Hash;

describe('Authentication Security', function () {
    describe('Password Security', function () {
        it('stores passwords hashed', function () {
            $user = User::factory()->create([
                'password' => Hash::make('SecurePass123!'),
            ]);

            expect($user->password)->not->toBe('SecurePass123!');
            expect(Hash::check('SecurePass123!', $user->password))->toBeTrue();
        });

        it('rejects weak passwords', function () {
            $response = $this->postJson('/api/register', [
                'name' => 'Test User',
                'email' => 'test@example.com',
                'password' => '123456',
                'password_confirmation' => '123456',
            ]);

            $response->assertUnprocessable()
                ->assertJsonValidationErrors(['password']);
        });
    });

    describe('Brute Force Protection', function () {
        it('rate limits login attempts', function () {
            $user = User::factory()->create();

            foreach (range(1, 6) as $i) {
                $response = $this->postJson('/api/login', [
                    'email' => $user->email,
                    'password' => 'wrongpassword',
                ]);
            }

            $response->assertTooManyRequests();
        });

        it('tracks failed login attempts', function () {
            $user = User::factory()->create();

            foreach (range(1, 3) as $i) {
                $this->postJson('/api/login', [
                    'email' => $user->email,
                    'password' => 'wrongpassword',
                ]);
            }

            expect($user->fresh()->failed_login_attempts)->toBe(3);
        });
    });

    describe('Session Security', function () {
        it('regenerates session on login', function () {
            $user = User::factory()->create([
                'password' => Hash::make('password'),
            ]);

            $oldSession = session()->getId();

            $this->postJson('/api/login', [
                'email' => $user->email,
                'password' => 'password',
            ]);

            expect(session()->getId())->not->toBe($oldSession);
        });

        it('invalidates session on logout', function () {
            $user = User::factory()->create();

            $this->actingAs($user);
            $token = $user->createToken('test')->plainTextToken;

            $this->withToken($token)->postJson('/api/logout');

            $this->withToken($token)->getJson('/api/user')
                ->assertUnauthorized();
        });
    });

    describe('SQL Injection Prevention', function () {
        it('escapes search input', function () {
            $maliciousInput = "'; DROP TABLE users; --";

            $response = $this->getJson('/api/products?search=' . urlencode($maliciousInput));

            $response->assertOk();
            $this->assertDatabaseHas('users', []); // Table still exists
        });
    });

    describe('XSS Prevention', function () {
        it('escapes user input in responses', function () {
            $admin = User::factory()->admin()->create();
            $maliciousName = '<script>alert("XSS")</script>';

            $this->actingAs($admin)->postJson('/api/products', [
                'name' => $maliciousName,
                'price' => 1000,
                'stock' => 10,
            ]);

            $response = $this->getJson('/api/products');

            $productName = $response->json('data.0.name');
            expect($productName)->not->toContain('<script>');
        });
    });
});
```

---

## Django/Wagtail Stack

### Django Functional Testing

```python
# apps/orders/tests/test_order_process.py

import pytest
from decimal import Decimal
from django.urls import reverse
from rest_framework import status

from apps.orders.models import Order, OrderItem
from apps.orders.enums import OrderStatus
from apps.products.models import Product
from apps.users.tests.factories import UserFactory
from apps.products.tests.factories import ProductFactory


@pytest.mark.django_db
class TestOrderProcess:
    """Functional tests for order processing."""

    class TestCreateOrder:
        """Tests for order creation."""

        def test_creates_order_with_valid_products(
            self, authenticated_client, user
        ):
            """Should create order with valid product data."""
            products = ProductFactory.create_batch(3, stock=10)
            url = reverse('api:orders-list')

            response = authenticated_client.post(url, {
                'items': [
                    {'product_id': p.id, 'quantity': 2}
                    for p in products
                ],
                'shipping_address': {
                    'line1': '123 Test Street',
                    'city': 'London',
                    'postcode': 'SW1A 1AA',
                    'country': 'GB',
                },
            }, format='json')

            assert response.status_code == status.HTTP_201_CREATED
            assert 'id' in response.data
            assert len(response.data['items']) == 3

            # Verify stock reduced
            for product in products:
                product.refresh_from_db()
                assert product.stock == 8

        def test_rejects_order_with_insufficient_stock(
            self, authenticated_client
        ):
            """Should reject order when stock is insufficient."""
            product = ProductFactory(stock=1)
            url = reverse('api:orders-list')

            response = authenticated_client.post(url, {
                'items': [{'product_id': product.id, 'quantity': 5}],
                'shipping_address': {
                    'line1': '123 Test Street',
                    'city': 'London',
                    'postcode': 'SW1A 1AA',
                    'country': 'GB',
                },
            }, format='json')

            assert response.status_code == status.HTTP_400_BAD_REQUEST
            assert 'stock' in str(response.data).lower()

        def test_requires_authentication(self, api_client):
            """Should require authentication."""
            product = ProductFactory()
            url = reverse('api:orders-list')

            response = api_client.post(url, {
                'items': [{'product_id': product.id, 'quantity': 1}],
            }, format='json')

            assert response.status_code == status.HTTP_401_UNAUTHORIZED

    class TestOrderStatusTransitions:
        """Tests for order status transitions."""

        def test_allows_valid_status_transitions(
            self, authenticated_client, user
        ):
            """Should allow valid status transitions."""
            from apps.orders.tests.factories import OrderFactory

            order = OrderFactory(user=user, status=OrderStatus.PENDING)
            url = reverse('api:orders-status', args=[order.id])

            response = authenticated_client.patch(url, {
                'status': 'confirmed',
            }, format='json')

            assert response.status_code == status.HTTP_200_OK
            order.refresh_from_db()
            assert order.status == OrderStatus.CONFIRMED

        def test_rejects_invalid_status_transitions(
            self, authenticated_client, user
        ):
            """Should reject invalid status transitions."""
            from apps.orders.tests.factories import OrderFactory

            order = OrderFactory(user=user, status=OrderStatus.DELIVERED)
            url = reverse('api:orders-status', args=[order.id])

            response = authenticated_client.patch(url, {
                'status': 'pending',
            }, format='json')

            assert response.status_code == status.HTTP_400_BAD_REQUEST
            assert 'transition' in str(response.data).lower()
```

### Django API Testing

```python
# apps/products/tests/test_api.py

import pytest
from django.urls import reverse
from rest_framework import status

from apps.products.models import Product
from apps.products.tests.factories import ProductFactory
from apps.users.tests.factories import UserFactory


@pytest.mark.django_db
class TestProductAPI:
    """API tests for product endpoints."""

    class TestListProducts:
        """Tests for GET /api/products/."""

        @pytest.fixture
        def url(self) -> str:
            return reverse('api:products-list')

        def test_returns_paginated_products(self, api_client, url):
            """Should return paginated product list."""
            ProductFactory.create_batch(25)

            response = api_client.get(url)

            assert response.status_code == status.HTTP_200_OK
            assert len(response.data['results']) == 20  # Default page size
            assert 'count' in response.data
            assert 'next' in response.data

        def test_filters_by_category(self, api_client, url):
            """Should filter products by category."""
            ProductFactory.create_batch(5, category='electronics')
            ProductFactory.create_batch(3, category='clothing')

            response = api_client.get(url, {'category': 'electronics'})

            assert response.status_code == status.HTTP_200_OK
            assert len(response.data['results']) == 5
            for product in response.data['results']:
                assert product['category'] == 'electronics'

        def test_searches_by_name(self, api_client, url):
            """Should search products by name."""
            ProductFactory(name='Blue Widget')
            ProductFactory(name='Red Widget')
            ProductFactory(name='Green Gadget')

            response = api_client.get(url, {'search': 'widget'})

            assert response.status_code == status.HTTP_200_OK
            assert len(response.data['results']) == 2

        def test_sorts_by_price(self, api_client, url):
            """Should sort products by price."""
            ProductFactory(price=Decimal('50.00'))
            ProductFactory(price=Decimal('10.00'))
            ProductFactory(price=Decimal('30.00'))

            response = api_client.get(url, {'ordering': 'price'})

            prices = [p['price'] for p in response.data['results']]
            assert prices == sorted(prices)

    class TestCreateProduct:
        """Tests for POST /api/products/."""

        @pytest.fixture
        def url(self) -> str:
            return reverse('api:products-list')

        def test_creates_product_as_admin(self, admin_client, url):
            """Should allow admin to create products."""
            response = admin_client.post(url, {
                'name': 'New Product',
                'description': 'A great product',
                'price': '29.99',
                'stock': 100,
                'category': 'electronics',
            }, format='json')

            assert response.status_code == status.HTTP_201_CREATED
            assert response.data['name'] == 'New Product'
            assert Product.objects.filter(name='New Product').exists()

        def test_rejects_creation_by_non_admin(
            self, authenticated_client, url
        ):
            """Should reject product creation by non-admin."""
            response = authenticated_client.post(url, {
                'name': 'New Product',
                'price': '29.99',
            }, format='json')

            assert response.status_code == status.HTTP_403_FORBIDDEN

        def test_validates_required_fields(self, admin_client, url):
            """Should validate required fields."""
            response = admin_client.post(url, {}, format='json')

            assert response.status_code == status.HTTP_400_BAD_REQUEST
            assert 'name' in response.data
            assert 'price' in response.data
```

### Django Security Testing

```python
# apps/core/tests/test_security.py

import pytest
from django.urls import reverse
from django.contrib.auth.hashers import check_password
from rest_framework import status

from apps.users.models import User
from apps.users.tests.factories import UserFactory


@pytest.mark.django_db
class TestAuthenticationSecurity:
    """Security tests for authentication."""

    class TestPasswordSecurity:
        """Tests for password security."""

        def test_stores_passwords_hashed(self, db):
            """Should store passwords as hashes."""
            user = UserFactory(password='SecurePass123!')

            assert user.password != 'SecurePass123!'
            assert check_password('SecurePass123!', user.password)

        def test_rejects_weak_passwords(self, api_client):
            """Should reject weak passwords."""
            url = reverse('api:register')

            response = api_client.post(url, {
                'email': 'test@example.com',
                'password': '123456',
                'password_confirm': '123456',
                'first_name': 'Test',
                'last_name': 'User',
            }, format='json')

            assert response.status_code == status.HTTP_400_BAD_REQUEST
            assert 'password' in response.data

    class TestBruteForceProtection:
        """Tests for brute force protection."""

        def test_rate_limits_login_attempts(self, api_client):
            """Should rate limit login attempts."""
            user = UserFactory()
            url = reverse('api:login')

            for _ in range(6):
                response = api_client.post(url, {
                    'email': user.email,
                    'password': 'wrongpassword',
                }, format='json')

            assert response.status_code == status.HTTP_429_TOO_MANY_REQUESTS

    class TestSQLInjectionPrevention:
        """Tests for SQL injection prevention."""

        def test_escapes_search_input(self, api_client):
            """Should escape malicious search input."""
            url = reverse('api:products-list')
            malicious_input = "'; DROP TABLE products; --"

            response = api_client.get(url, {'search': malicious_input})

            assert response.status_code == status.HTTP_200_OK
            # Verify table still exists
            from apps.products.models import Product
            assert Product.objects.count() >= 0

    class TestXSSPrevention:
        """Tests for XSS prevention."""

        def test_escapes_user_input_in_responses(self, admin_client):
            """Should escape potentially malicious input."""
            url = reverse('api:products-list')
            malicious_name = '<script>alert("XSS")</script>'

            admin_client.post(url, {
                'name': malicious_name,
                'price': '10.00',
                'stock': 10,
                'category': 'test',
            }, format='json')

            response = admin_client.get(url)

            for product in response.data['results']:
                assert '<script>' not in product['name']
```

---

## React/Next.js Stack

### Next.js Component Testing

```typescript
// tests/components/ProductCard.test.tsx

import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { ProductCard } from '@/components/ProductCard';
import type { Product } from '@/types';

const mockProduct: Product = {
  id: '1',
  name: 'Test Product',
  description: 'A great test product',
  price: 2999,
  imageUrl: '/images/product.jpg',
  category: 'electronics',
  stock: 10,
};

describe('ProductCard', () => {
  describe('Display', () => {
    it('renders product information correctly', () => {
      render(<ProductCard product={mockProduct} />);

      expect(screen.getByText('Test Product')).toBeInTheDocument();
      expect(screen.getByText('£29.99')).toBeInTheDocument();
      expect(screen.getByRole('img')).toHaveAttribute('alt', 'Test Product');
    });

    it('shows out of stock badge when stock is 0', () => {
      const outOfStock = { ...mockProduct, stock: 0 };
      render(<ProductCard product={outOfStock} />);

      expect(screen.getByText('Out of Stock')).toBeInTheDocument();
    });

    it('shows low stock warning when stock < 5', () => {
      const lowStock = { ...mockProduct, stock: 3 };
      render(<ProductCard product={lowStock} />);

      expect(screen.getByText('Only 3 left!')).toBeInTheDocument();
    });
  });

  describe('Interactions', () => {
    it('calls onAddToCart when button clicked', async () => {
      const user = userEvent.setup();
      const onAddToCart = vi.fn();

      render(<ProductCard product={mockProduct} onAddToCart={onAddToCart} />);

      await user.click(screen.getByRole('button', { name: /add to cart/i }));

      expect(onAddToCart).toHaveBeenCalledWith(mockProduct, 1);
    });

    it('disables add to cart when out of stock', () => {
      const outOfStock = { ...mockProduct, stock: 0 };
      render(<ProductCard product={outOfStock} onAddToCart={vi.fn()} />);

      expect(screen.getByRole('button', { name: /out of stock/i })).toBeDisabled();
    });

    it('navigates to product page on click', async () => {
      const user = userEvent.setup();
      const onNavigate = vi.fn();

      render(<ProductCard product={mockProduct} onNavigate={onNavigate} />);

      await user.click(screen.getByRole('article'));

      expect(onNavigate).toHaveBeenCalledWith(`/products/${mockProduct.id}`);
    });
  });

  describe('Accessibility', () => {
    it('has accessible name and role', () => {
      render(<ProductCard product={mockProduct} />);

      const card = screen.getByRole('article');
      expect(card).toHaveAccessibleName(/test product/i);
    });

    it('provides price in accessible format', () => {
      render(<ProductCard product={mockProduct} />);

      const price = screen.getByText('£29.99');
      expect(price).toHaveAttribute('aria-label', 'Price: 29 pounds 99 pence');
    });
  });
});
```

### Next.js Integration Testing

```typescript
// tests/integration/checkout.test.tsx

import { render, screen, waitFor, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { CheckoutPage } from '@/app/checkout/page';
import { CartProvider } from '@/contexts/CartContext';
import { server } from '@/mocks/server';
import { rest } from 'msw';

function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <CartProvider>{ui}</CartProvider>
    </QueryClientProvider>
  );
}

describe('Checkout Integration', () => {
  beforeEach(() => {
    // Reset cart state
    localStorage.clear();
  });

  describe('Complete checkout flow', () => {
    it('processes order successfully', async () => {
      const user = userEvent.setup();

      // Add items to cart first
      localStorage.setItem('cart', JSON.stringify({
        items: [
          { productId: '1', quantity: 2, price: 2999, name: 'Product 1' },
          { productId: '2', quantity: 1, price: 4999, name: 'Product 2' },
        ],
      }));

      renderWithProviders(<CheckoutPage />);

      // Fill shipping details
      await user.type(screen.getByLabelText(/address line 1/i), '123 Test Street');
      await user.type(screen.getByLabelText(/city/i), 'London');
      await user.type(screen.getByLabelText(/postcode/i), 'SW1A 1AA');

      // Fill payment details
      await user.type(screen.getByLabelText(/card number/i), '4242424242424242');
      await user.type(screen.getByLabelText(/expiry/i), '12/25');
      await user.type(screen.getByLabelText(/cvc/i), '123');

      // Submit order
      await user.click(screen.getByRole('button', { name: /place order/i }));

      // Verify success
      await waitFor(() => {
        expect(screen.getByText(/order confirmed/i)).toBeInTheDocument();
      });

      expect(screen.getByText(/order #/i)).toBeInTheDocument();
    });

    it('shows validation errors for invalid input', async () => {
      const user = userEvent.setup();

      localStorage.setItem('cart', JSON.stringify({
        items: [{ productId: '1', quantity: 1, price: 2999 }],
      }));

      renderWithProviders(<CheckoutPage />);

      // Submit without filling form
      await user.click(screen.getByRole('button', { name: /place order/i }));

      await waitFor(() => {
        expect(screen.getByText(/address is required/i)).toBeInTheDocument();
        expect(screen.getByText(/card number is required/i)).toBeInTheDocument();
      });
    });

    it('handles payment failure gracefully', async () => {
      const user = userEvent.setup();

      // Mock payment failure
      server.use(
        rest.post('/api/orders', (req, res, ctx) => {
          return res(
            ctx.status(400),
            ctx.json({ error: 'Payment declined' })
          );
        })
      );

      localStorage.setItem('cart', JSON.stringify({
        items: [{ productId: '1', quantity: 1, price: 2999 }],
      }));

      renderWithProviders(<CheckoutPage />);

      // Fill valid details
      await user.type(screen.getByLabelText(/address line 1/i), '123 Test Street');
      await user.type(screen.getByLabelText(/city/i), 'London');
      await user.type(screen.getByLabelText(/postcode/i), 'SW1A 1AA');
      await user.type(screen.getByLabelText(/card number/i), '4000000000000002');
      await user.type(screen.getByLabelText(/expiry/i), '12/25');
      await user.type(screen.getByLabelText(/cvc/i), '123');

      await user.click(screen.getByRole('button', { name: /place order/i }));

      await waitFor(() => {
        expect(screen.getByRole('alert')).toHaveTextContent(/payment declined/i);
      });

      // Cart should still have items
      expect(screen.getByText(/Product 1/i)).toBeInTheDocument();
    });
  });
});
```

### Next.js Accessibility Testing

```typescript
// tests/accessibility/pages.test.tsx

import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { describe, it, expect } from 'vitest';

import { HomePage } from '@/app/page';
import { ProductPage } from '@/app/products/[id]/page';
import { CheckoutPage } from '@/app/checkout/page';

expect.extend(toHaveNoViolations);

describe('Accessibility', () => {
  describe('HomePage', () => {
    it('has no accessibility violations', async () => {
      const { container } = render(<HomePage />);
      const results = await axe(container);
      expect(results).toHaveNoViolations();
    });
  });

  describe('ProductPage', () => {
    it('has no accessibility violations', async () => {
      const { container } = render(
        <ProductPage params={{ id: '1' }} />
      );
      const results = await axe(container);
      expect(results).toHaveNoViolations();
    });
  });

  describe('CheckoutPage', () => {
    it('has no accessibility violations', async () => {
      const { container } = render(<CheckoutPage />);
      const results = await axe(container);
      expect(results).toHaveNoViolations();
    });

    it('has proper form labels', () => {
      const { container } = render(<CheckoutPage />);

      const inputs = container.querySelectorAll('input');
      inputs.forEach((input) => {
        expect(input).toHaveAccessibleName();
      });
    });

    it('has proper heading hierarchy', () => {
      const { container } = render(<CheckoutPage />);

      const headings = container.querySelectorAll('h1, h2, h3, h4, h5, h6');
      const levels = Array.from(headings).map((h) =>
        parseInt(h.tagName.charAt(1))
      );

      // Check headings don't skip levels
      for (let i = 1; i < levels.length; i++) {
        expect(levels[i] - levels[i - 1]).toBeLessThanOrEqual(1);
      }
    });
  });

  describe('Keyboard Navigation', () => {
    it('allows full keyboard navigation', async () => {
      const { container } = render(<CheckoutPage />);

      const focusableElements = container.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );

      focusableElements.forEach((element) => {
        expect(element).toBeVisible();
        // Verify focus styles exist
        element.focus();
        expect(document.activeElement).toBe(element);
      });
    });
  });
});
```

---

## React Native Stack

### Device Testing

```typescript
// e2e/checkout.e2e.ts

import { by, device, element, expect } from 'detox';

describe('Checkout Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  describe('Complete Purchase', () => {
    it('should complete checkout successfully', async () => {
      // Navigate to product
      await element(by.id('product-1')).tap();
      await expect(element(by.id('product-detail'))).toBeVisible();

      // Add to cart
      await element(by.id('add-to-cart-button')).tap();
      await expect(element(by.text('Added to cart'))).toBeVisible();

      // Go to cart
      await element(by.id('cart-tab')).tap();
      await expect(element(by.id('cart-screen'))).toBeVisible();
      await expect(element(by.id('cart-item-1'))).toBeVisible();

      // Proceed to checkout
      await element(by.id('checkout-button')).tap();

      // Fill shipping address
      await element(by.id('address-input')).typeText('123 Test Street');
      await element(by.id('city-input')).typeText('London');
      await element(by.id('postcode-input')).typeText('SW1A 1AA');

      // Fill payment
      await element(by.id('card-number-input')).typeText('4242424242424242');
      await element(by.id('expiry-input')).typeText('1225');
      await element(by.id('cvc-input')).typeText('123');

      // Place order
      await element(by.id('place-order-button')).tap();

      // Verify success
      await expect(element(by.id('order-confirmation'))).toBeVisible();
      await expect(element(by.text('Order Confirmed'))).toBeVisible();
    });

    it('should show error for declined card', async () => {
      // Add product and go to checkout
      await element(by.id('product-1')).tap();
      await element(by.id('add-to-cart-button')).tap();
      await element(by.id('cart-tab')).tap();
      await element(by.id('checkout-button')).tap();

      // Fill details
      await element(by.id('address-input')).typeText('123 Test Street');
      await element(by.id('city-input')).typeText('London');
      await element(by.id('postcode-input')).typeText('SW1A 1AA');

      // Use declined card
      await element(by.id('card-number-input')).typeText('4000000000000002');
      await element(by.id('expiry-input')).typeText('1225');
      await element(by.id('cvc-input')).typeText('123');

      await element(by.id('place-order-button')).tap();

      // Verify error
      await expect(element(by.id('error-message'))).toBeVisible();
      await expect(element(by.text('Payment declined'))).toBeVisible();
    });
  });

  describe('Form Validation', () => {
    beforeEach(async () => {
      await element(by.id('product-1')).tap();
      await element(by.id('add-to-cart-button')).tap();
      await element(by.id('cart-tab')).tap();
      await element(by.id('checkout-button')).tap();
    });

    it('should show validation errors for empty fields', async () => {
      await element(by.id('place-order-button')).tap();

      await expect(element(by.text('Address is required'))).toBeVisible();
      await expect(element(by.text('Card number is required'))).toBeVisible();
    });

    it('should validate postcode format', async () => {
      await element(by.id('postcode-input')).typeText('invalid');
      await element(by.id('place-order-button')).tap();

      await expect(element(by.text('Invalid postcode'))).toBeVisible();
    });
  });
});
```

### RN Integration Testing

```typescript
// src/__tests__/integration/cart.test.tsx

import React from 'react';
import { renderHook, act, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useCart } from '@/hooks/useCart';
import { CartProvider } from '@/contexts/CartContext';
import { server } from '@/mocks/server';
import { rest } from 'msw';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      <CartProvider>{children}</CartProvider>
    </QueryClientProvider>
  );
};

describe('Cart Integration', () => {
  describe('useCart hook', () => {
    it('adds item to cart', async () => {
      const { result } = renderHook(() => useCart(), {
        wrapper: createWrapper(),
      });

      await act(async () => {
        await result.current.addItem({
          productId: '1',
          quantity: 2,
          price: 2999,
          name: 'Test Product',
        });
      });

      expect(result.current.items).toHaveLength(1);
      expect(result.current.items[0].quantity).toBe(2);
      expect(result.current.total).toBe(5998);
    });

    it('updates item quantity', async () => {
      const { result } = renderHook(() => useCart(), {
        wrapper: createWrapper(),
      });

      await act(async () => {
        await result.current.addItem({
          productId: '1',
          quantity: 1,
          price: 2999,
          name: 'Test Product',
        });
      });

      await act(async () => {
        await result.current.updateQuantity('1', 5);
      });

      expect(result.current.items[0].quantity).toBe(5);
      expect(result.current.total).toBe(14995);
    });

    it('removes item from cart', async () => {
      const { result } = renderHook(() => useCart(), {
        wrapper: createWrapper(),
      });

      await act(async () => {
        await result.current.addItem({
          productId: '1',
          quantity: 1,
          price: 2999,
          name: 'Test Product',
        });
      });

      await act(async () => {
        await result.current.removeItem('1');
      });

      expect(result.current.items).toHaveLength(0);
      expect(result.current.total).toBe(0);
    });

    it('syncs cart with server', async () => {
      const { result } = renderHook(() => useCart(), {
        wrapper: createWrapper(),
      });

      await act(async () => {
        await result.current.addItem({
          productId: '1',
          quantity: 1,
          price: 2999,
          name: 'Test Product',
        });
      });

      // Wait for sync
      await waitFor(() => {
        expect(result.current.isSynced).toBe(true);
      });
    });

    it('handles sync failure gracefully', async () => {
      server.use(
        rest.post('/api/cart/sync', (req, res, ctx) => {
          return res(ctx.status(500));
        })
      );

      const { result } = renderHook(() => useCart(), {
        wrapper: createWrapper(),
      });

      await act(async () => {
        await result.current.addItem({
          productId: '1',
          quantity: 1,
          price: 2999,
          name: 'Test Product',
        });
      });

      // Cart should still work locally
      expect(result.current.items).toHaveLength(1);
      expect(result.current.syncError).toBeTruthy();
    });
  });
});
```

### RN Accessibility Testing

```typescript
// src/__tests__/accessibility/components.test.tsx

import React from 'react';
import { render, screen } from '@testing-library/react-native';
import { ProductCard } from '@/components/ProductCard';
import { Button } from '@/components/ui/Button';
import { TextInput } from '@/components/ui/TextInput';

describe('Accessibility', () => {
  describe('ProductCard', () => {
    const mockProduct = {
      id: '1',
      name: 'Test Product',
      price: 2999,
      imageUrl: 'https://example.com/image.jpg',
    };

    it('has accessible role', () => {
      render(<ProductCard product={mockProduct} onPress={jest.fn()} />);

      const card = screen.getByRole('button');
      expect(card).toBeTruthy();
    });

    it('has accessible label', () => {
      render(<ProductCard product={mockProduct} onPress={jest.fn()} />);

      const card = screen.getByLabelText(/test product/i);
      expect(card).toBeTruthy();
    });

    it('announces price correctly', () => {
      render(<ProductCard product={mockProduct} onPress={jest.fn()} />);

      const price = screen.getByAccessibilityHint(/price/i);
      expect(price.props.accessibilityLabel).toContain('29');
      expect(price.props.accessibilityLabel).toContain('99');
    });
  });

  describe('Button', () => {
    it('has correct accessibility state when disabled', () => {
      render(<Button disabled>Submit</Button>);

      const button = screen.getByRole('button');
      expect(button.props.accessibilityState.disabled).toBe(true);
    });

    it('has correct accessibility state when loading', () => {
      render(<Button loading>Submit</Button>);

      const button = screen.getByRole('button');
      expect(button.props.accessibilityState.busy).toBe(true);
    });
  });

  describe('TextInput', () => {
    it('has associated label', () => {
      render(<TextInput label="Email" placeholder="Enter email" />);

      const input = screen.getByLabelText('Email');
      expect(input).toBeTruthy();
    });

    it('announces error state', () => {
      render(
        <TextInput
          label="Email"
          error="Invalid email address"
        />
      );

      const input = screen.getByLabelText('Email');
      expect(input.props.accessibilityState.invalid).toBe(true);
    });

    it('provides error hint', () => {
      render(
        <TextInput
          label="Email"
          error="Invalid email address"
        />
      );

      const errorText = screen.getByText('Invalid email address');
      expect(errorText.props.accessibilityRole).toBe('alert');
    });
  });

  describe('Screen Reader Announcements', () => {
    it('announces loading states', () => {
      const { rerender } = render(<Button>Submit</Button>);

      rerender(<Button loading>Submit</Button>);

      // In a real test, you'd check AccessibilityInfo.announceForAccessibility was called
      const button = screen.getByRole('button');
      expect(button.props.accessibilityLabel).toContain('Loading');
    });
  });
});
```
