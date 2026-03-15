# Architecture Patterns

**Last Updated:** 15/03/2026
**Version:** 1.8.0
**Maintained By:** Development Team
**Language:** British English (en_GB)
**Timezone:** Europe/London
**Plugin Scope:** syntek-dev-suite (Python/Django, PHP/Laravel, TypeScript/React, React Native)

---

## Table of Contents

- [Overview](#overview)
- [Service Layer](#service-layer)
- [Middleware and Request Pipeline](#middleware-and-request-pipeline)
- [State Management (Frontend)](#state-management-frontend)
- [Routing Conventions](#routing-conventions)
- [Background Job Patterns](#background-job-patterns)
- [Email and Notification Patterns](#email-and-notification-patterns)
- [File Processing Pipelines](#file-processing-pipelines)
- [Project Structure](#project-structure)

---

## Overview

This guide covers architectural patterns that are not covered by the other guides in this suite. It fills the gap between domain modelling (`DATA-STRUCTURES.md`), API contracts (`API-DESIGN.md`), and performance (`PERFORMANCE.md`).

It is deliberately slim. Each section establishes the pattern and the rules — not an exhaustive reference, but enough for a developer or agent to make consistent decisions.

---

## Service Layer

Business logic lives in service classes or modules, not in views, controllers, serialisers, models, or components. This is the most important architectural boundary in the application.

### The rule

- **Views/Controllers** handle HTTP concerns: receiving requests, validating input, returning responses.
- **Services** handle business logic: orchestrating domain operations, enforcing business rules, coordinating between models.
- **Models** handle data: persistence, relationships, simple computed properties. Models do not call external services, send emails, or dispatch events.
- **Serialisers/Resources** handle transformation: converting between internal and external representations.

### Django

```python
# services/order_service.py
class OrderService:
    def create_order(self, customer: Customer, items: list[OrderItemInput]) -> Order:
        """Business logic: create an order with validation and side effects."""
        if not items:
            raise ValueError("Cannot create an order with no items")

        order = Order.objects.create(customer=customer, status=OrderStatus.DRAFT)
        for item in items:
            order.add_line(product=item.product, quantity=item.quantity)
        order.recalculate_total()

        send_order_confirmation.delay(order.id)  # async side effect
        return order
```

```python
# views.py — thin controller
class OrderCreateView(APIView):
    def post(self, request):
        serialiser = CreateOrderSerialiser(data=request.data)
        serialiser.is_valid(raise_exception=True)

        service = OrderService()
        order = service.create_order(
            customer=request.user,
            items=serialiser.validated_data["items"],
        )

        return Response(OrderSerialiser(order).data, status=201)
```

### Laravel

```php
// Services/OrderService.php
class OrderService
{
    public function createOrder(Customer $customer, array $items): Order
    {
        if (empty($items)) {
            throw new \DomainException('Cannot create an order with no items');
        }

        return DB::transaction(function () use ($customer, $items) {
            $order = Order::create([
                'customer_id' => $customer->id,
                'status'      => OrderStatus::Draft,
            ]);

            foreach ($items as $item) {
                $order->addLine($item['product'], $item['quantity']);
            }

            $order->recalculateTotal();
            SendOrderConfirmation::dispatch($order);

            return $order;
        });
    }
}
```

```php
// Controllers/OrderController.php — thin controller
class OrderController extends Controller
{
    public function store(CreateOrderRequest $request, OrderService $service): JsonResponse
    {
        $order = $service->createOrder(
            customer: $request->user(),
            items: $request->validated()['items'],
        );

        return OrderResource::make($order)->response()->setStatusCode(201);
    }
}
```

### Rules

- If a view/controller method exceeds 10-15 lines of logic (beyond input validation and response construction), extract it to a service.
- Services may call other services. Keep the dependency graph acyclic.
- Services must not access `request` directly. Pass the data they need as arguments. This makes them testable without HTTP.
- Wrap multi-step operations in database transactions. If any step fails, all steps roll back.
- Side effects (emails, webhooks, event dispatch) happen at the end of the service method, after the primary operation succeeds.

---

## Middleware and Request Pipeline

Middleware processes requests before they reach the controller and responses before they reach the client. Order matters.

### Recommended middleware order

```
1. Security headers (CSP, HSTS, X-Frame-Options)
2. CORS
3. Request logging / request ID injection
4. Authentication (verify token, populate user)
5. Tenant resolution (identify tenant from subdomain/header)
6. Rate limiting
7. CSRF verification (stateful routes only)
8. Authorisation (route-level permission check)
9. Input validation (via form request / serialiser — often in the controller)
```

### Rules

- Authentication middleware must run before authorisation, rate limiting, and tenant resolution — you need to know who the user is before checking permissions or scoping data.
- Tenant resolution middleware must run before any service or query that accesses tenant-scoped data.
- Do not put business logic in middleware. Middleware handles cross-cutting concerns (security, logging, auth, rate limiting), not domain rules.
- Global middleware applies to every request. Route-specific middleware applies to groups of routes. Do not apply expensive middleware globally if it is only needed on a subset of routes.

### Laravel middleware groups

```php
// bootstrap/app.php or Kernel.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->api([
        'throttle:60,1',
        ResolveTenant::class,
    ]);
})
```

### Django middleware

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.middleware.common.CommonMiddleware",
    "corsheaders.middleware.CorsMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "apps.tenants.middleware.TenantMiddleware",
    # custom middleware below framework middleware
]
```

---

## State Management (Frontend)

State is the primary source of complexity in frontend applications. Minimise the amount of state and keep it in the right place.

### State categories

| Category | Where to store | Examples |
|----------|---------------|----------|
| **Server state** | React Query / TanStack Query / SWR | API data: orders, users, products |
| **URL state** | URL parameters and search params | Filters, pagination, sort order, active tab |
| **Form state** | Local component state or form library | Input values, validation errors, dirty flags |
| **UI state** | Local component state | Modal open/closed, dropdown expanded, sidebar collapsed |
| **Global UI state** | React Context (sparingly) | Theme, locale, toast notifications |

### Rules

- **Server state is not component state.** Do not store API responses in `useState`. Use a server state library (TanStack Query is recommended) that handles caching, background refetching, and stale data.
- **URL is the source of truth for navigation state.** Filters, pagination, sort order, and tab selections belong in the URL, not in React state. This makes the state shareable, bookmarkable, and browser-history-friendly.
- **Avoid global state.** Most state is local to a component or a small subtree. Reach for React Context only for truly global concerns (theme, authenticated user, locale). If you need `useContext` in more than 5 components, consider whether the state should be in the URL or in a server state library instead.
- **Do not use Redux** unless the application has genuinely complex client-side state that cannot be modelled as server state, URL state, or local state. For most CRUD applications, Redux adds complexity without value.
- **Lift state only as high as necessary.** If two sibling components need the same state, lift it to their parent — not to the nearest context provider and definitely not to a global store.

### React Query example

```typescript
// hooks/useOrders.ts
function useOrders(filters: OrderFilters) {
  return useQuery({
    queryKey: ["orders", filters],
    queryFn: () => getOrders(filters),
    staleTime: 60_000, // consider data fresh for 1 minute
  });
}

// In the component
function OrdersPage() {
  const [searchParams] = useSearchParams();
  const filters = parseOrderFilters(searchParams);
  const { data, isLoading, error } = useOrders(filters);

  // ...
}
```

### React Native state

The same rules apply. Use TanStack Query for server state. Use React Navigation's params for screen-level state. Avoid AsyncStorage for state management — it is a persistence mechanism, not a state store.

---

## Routing Conventions

### Backend (REST)

All REST routes follow the resource-oriented conventions in `API-DESIGN.md`. Additionally:

**Laravel:**

```php
// routes/api.php — resource routes
Route::apiResource('orders', OrderController::class);
Route::apiResource('orders.lines', OrderLineController::class)->shallow();

// Nested actions
Route::post('orders/{order}/confirm', [OrderController::class, 'confirm']);
```

**Django:**

```python
# urls.py
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register("orders", OrderViewSet, basename="order")

urlpatterns = [
    path("api/v1/", include(router.urls)),
]
```

### Frontend (React)

- One route file per feature area. Do not put all routes in a single file.
- Lazy load every page-level component (see `PERFORMANCE.md`).
- Use nested routes for layout inheritance (shared sidebars, headers).
- Use path parameters for resource identity (`/orders/:id`), query parameters for filters and pagination (`/orders?status=confirmed&page=2`).

---

## Background Job Patterns

See `PERFORMANCE.md` — Background Jobs and Queues for when to use queues and implementation examples. This section covers architectural patterns.

### Job classification

| Type | Description | Queue | Retry |
|------|------------|-------|-------|
| **Fire-and-forget** | No result needed (email, webhook, audit log) | Default | Yes, with backoff |
| **Deferred result** | Result needed later (report generation, export) | Dedicated queue | Yes, notify on failure |
| **Scheduled** | Runs on a cron schedule (daily digest, cleanup) | Scheduled | Yes, alert on repeated failure |
| **Chained** | Sequence of dependent steps (payment → invoice → email) | Default | Each step retries independently |

### Rules

- Each job has a single, clear responsibility. A job that sends an email and updates a database and calls an API is doing too many things.
- Chain jobs explicitly rather than having one job trigger the next implicitly. `ProcessPayment -> GenerateInvoice -> SendReceipt` is clearer than `ProcessPayment` silently dispatching the others.
- Failed jobs must be monitored. Set up alerts for job failure rates exceeding a threshold.
- Long-running jobs (> 30 seconds) must send heartbeats or progress updates to prevent the queue from considering them dead.
- Jobs that interact with external APIs must implement circuit breaker patterns for repeated failures.

---

## Email and Notification Patterns

### Architecture

- Emails and notifications are always dispatched via queued jobs, never sent synchronously in the request cycle.
- Use the framework's notification system (Laravel Notifications, Django signals + custom notification service).
- Each notification type has a single class/function that encapsulates the content, recipients, and delivery channels.

### Laravel

```php
class OrderConfirmedNotification extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(private Order $order) {}

    public function via(object $notifiable): array
    {
        return ['mail', 'database'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject("Order #{$this->order->reference} confirmed")
            ->greeting("Hi {$notifiable->name},")
            ->line("Your order has been confirmed.")
            ->action('View Order', url("/orders/{$this->order->id}"));
    }
}

// Dispatch
$customer->notify(new OrderConfirmedNotification($order));
```

### Django

```python
# notifications/order_confirmed.py
from django.core.mail import send_mail

@shared_task
def send_order_confirmed(order_id: int) -> None:
    order = Order.objects.select_related("customer").get(id=order_id)
    send_mail(
        subject=f"Order #{order.reference} confirmed",
        message=f"Hi {order.customer.name}, your order has been confirmed.",
        from_email="orders@example.com",
        recipient_list=[order.customer.email],
    )
```

### Rules

- Never put email content (subject lines, body text) in controllers or services. It belongs in dedicated notification/mail classes.
- All emails must have both HTML and plain-text versions.
- Use a logging or null mailer in development and test environments. Never send real emails in tests.
- For transactional emails (order confirmations, password resets), send immediately via queue. For marketing or digest emails, use a scheduled job.

---

## File Processing Pipelines

When handling user-uploaded files that require processing (image resizing, document conversion, video transcoding):

### Pattern

1. **Accept** — validate and store the original file. Return immediately with a "processing" status.
2. **Queue** — dispatch a processing job with the file path and desired transformations.
3. **Process** — the job performs the transformation, stores the result, and updates the record's status.
4. **Notify** — if the user is waiting, notify via WebSocket, polling, or email.

### Rules

- Store originals permanently. Store processed versions as derived artefacts that can be regenerated.
- Process files in a background job, never in the request cycle. Large files will timeout the request.
- Set file size limits at the web server level, the application framework level, and the storage layer. See `SECURITY.md` — File Upload Security.
- Use streaming uploads for large files to avoid loading the entire file into memory.
- Clean up temporary files in a `finally` block or equivalent, even if processing fails.

---

## Project Structure

### Django

```
project/
├── apps/
│   ├── orders/
│   │   ├── models.py
│   │   ├── services.py
│   │   ├── serialisers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── tasks.py         # Celery tasks
│   │   ├── signals.py
│   │   └── tests/
│   ├── payments/
│   └── users/
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── testing.py
│   │   └── production.py
│   ├── urls.py
│   └── celery.py
├── templates/
├── static/
└── manage.py
```

### Laravel

```
project/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Requests/        # Form requests (validation)
│   │   ├── Resources/       # API resources (transformation)
│   │   └── Middleware/
│   ├── Models/
│   ├── Services/            # Business logic
│   ├── Jobs/                # Queued jobs
│   ├── Notifications/
│   ├── Observers/
│   ├── Enums/
│   └── Exceptions/
├── config/
├── database/
│   ├── migrations/
│   ├── factories/
│   └── seeders/
├── resources/
│   └── views/
├── routes/
│   ├── api.php
│   └── web.php
└── tests/
    ├── Unit/
    └── Feature/
```

### React (Next.js / SPA)

```
project/
├── src/
│   ├── app/                 # Next.js App Router pages
│   │   ├── orders/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/              # Reusable UI primitives (Button, Input, Modal)
│   │   └── features/        # Feature-specific components (OrderTable, BookingCard)
│   ├── hooks/               # Custom React hooks
│   ├── lib/
│   │   ├── api/             # API client and typed endpoint functions
│   │   └── utils/           # Pure utility functions
│   ├── types/               # TypeScript interfaces and types
│   └── styles/
├── public/
└── tests/
```

### Rules

- Group by feature/domain, not by technical layer. An `orders/` directory containing its model, service, controller, and tests is easier to navigate than separate `models/`, `services/`, `controllers/` directories each containing files from every feature.
- The exception: small projects with few features can use layer-based grouping. Switch to feature-based grouping when any layer directory exceeds 10 files.
- Keep test files adjacent to or mirroring the source structure.
- Configuration files live in a `config/` directory, not scattered across the project root.
