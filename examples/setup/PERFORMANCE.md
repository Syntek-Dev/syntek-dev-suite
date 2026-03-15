# Performance

**Last Updated:** 15/03/2026
**Version:** 1.8.0
**Maintained By:** Development Team
**Language:** British English (en_GB)
**Timezone:** Europe/London
**Plugin Scope:** syntek-dev-suite (Python/Django, PHP/Laravel, TypeScript/React, React Native)

---

## Table of Contents

- [Overview](#overview)
- [The Rules](#the-rules)
- [Database Query Optimisation](#database-query-optimisation)
  - [N+1 Queries](#n1-queries)
  - [Query Analysis](#query-analysis)
  - [Indexing Strategy](#indexing-strategy)
  - [Query Patterns to Avoid](#query-patterns-to-avoid)
- [Caching](#caching)
  - [Cache Hierarchy](#cache-hierarchy)
  - [Application-Level Caching](#application-level-caching)
  - [HTTP Caching](#http-caching)
  - [Cache Invalidation](#cache-invalidation)
- [Frontend Performance](#frontend-performance)
  - [Bundle Size](#bundle-size)
  - [Code Splitting and Lazy Loading](#code-splitting-and-lazy-loading)
  - [Rendering Strategy](#rendering-strategy)
  - [React Performance Patterns](#react-performance-patterns)
- [Image and Asset Optimisation](#image-and-asset-optimisation)
- [API and Network Performance](#api-and-network-performance)
- [Background Jobs and Queues](#background-jobs-and-queues)
- [Connection Pooling](#connection-pooling)
- [Mobile Performance (React Native)](#mobile-performance-react-native)
- [Monitoring and Measurement](#monitoring-and-measurement)
- [Load Testing](#load-testing)
- [Performance Checklist](#performance-checklist)

---

## Overview

Performance work follows the same philosophy as the coding principles: measure first, then optimise. Do not guess where bottlenecks are. Do not add caching, indexing, or memoisation "just in case." Measure, identify the bottleneck, fix it, then measure again.

This guide covers practical performance patterns across the syntek-dev-suite stack. It applies to database queries, caching, frontend rendering, API design, background processing, and infrastructure.

---

## The Rules

1. **Do not optimise without measuring.** (Pike Rule 1, Pike Rule 2.)
2. **Premature optimisation is the wrong abstraction applied too early.** Wait until a real performance problem exists, then fix the actual bottleneck.
3. **The fastest code is code that does not run.** Eliminate unnecessary work before optimising the work that remains.
4. **Performance budgets are constraints, not goals.** Set them, enforce them in CI, and treat violations as bugs.
5. **User-perceived performance matters more than server-side metrics.** A page that loads in 200ms but shows nothing for 2 seconds is slower than a page that loads in 500ms with a meaningful first paint.

---

## Database Query Optimisation

### N+1 Queries

The N+1 problem is the most common performance issue in ORM-based applications. It occurs when a query for N items triggers N additional queries to load a related object for each item.

**Django — detecting and fixing N+1:**

```python
# BAD: N+1 — one query for orders, then one query per order to get the customer
orders = Order.objects.all()
for order in orders:
    print(order.customer.name)  # hits the database for each order

# GOOD: select_related for ForeignKey / OneToOne (SQL JOIN)
orders = Order.objects.select_related("customer").all()

# GOOD: prefetch_related for ManyToMany / reverse ForeignKey (separate query + Python join)
orders = Order.objects.prefetch_related("lines", "lines__product").all()
```

**Laravel — detecting and fixing N+1:**

```php
// BAD: N+1
$orders = Order::all();
foreach ($orders as $order) {
    echo $order->customer->name; // query per iteration
}

// GOOD: eager loading
$orders = Order::with(['customer', 'lines.product'])->get();

// Prevent N+1 in development
// In AppServiceProvider::boot():
Model::preventLazyLoading(! app()->isProduction());
```

**Laravel's `preventLazyLoading()`** throws an exception whenever a relationship is lazy-loaded in non-production environments. Enable this in every project — it catches N+1 problems at development time.

**Django's `nplusone` library** or `django-debug-toolbar` serve the same purpose — flag lazy-loaded relationships during development.

### Query Analysis

Always use `EXPLAIN` before adding indexes or rewriting queries.

**PostgreSQL:**

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC;
```

Look for: `Seq Scan` (full table scan — may need an index), `Nested Loop` (check for N+1), `Sort` (check for missing index on sort column).

**MariaDB/MySQL:**

```sql
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC;
```

Look for: `type: ALL` (full table scan), `Using filesort` (sorting without index), `Using temporary` (temporary table for grouping).

### Indexing Strategy

See `DATA-STRUCTURES.md` — Indexes for the full indexing guide. Performance-specific additions:

- **Covering indexes** (PostgreSQL, MariaDB/MySQL): if a query only needs columns that are all in the index, the database can answer the query from the index alone without reading the table. For hot queries, consider adding `INCLUDE` columns (PostgreSQL) or including all selected columns in a composite index.
- **Index-only scans**: verify with `EXPLAIN` that your most frequent queries achieve index-only scans.
- **Unused indexes**: indexes that are never used by queries slow down writes without benefiting reads. Audit periodically with `pg_stat_user_indexes` (PostgreSQL) or `sys.schema_unused_indexes` (MySQL 8.0).

### Query Patterns to Avoid

- **SELECT \***: select only the columns you need. Fewer columns = smaller result set = faster transfer.
- **Unbounded queries**: every collection query must have a `LIMIT`. See `API-DESIGN.md` — Pagination.
- **Queries in loops**: if you need data for each item in a list, use a single query with `IN (...)` or a `JOIN`, not a query per item.
- **COUNT(\*) for existence checks**: if you only need to know whether a record exists, use `EXISTS` (SQL), `.exists()` (Django), or `->exists()` (Laravel) instead of counting all matches.

```python
# BAD: counts all matching rows
if Order.objects.filter(customer=customer, status="pending").count() > 0:

# GOOD: stops at the first match
if Order.objects.filter(customer=customer, status="pending").exists():
```

```php
// BAD
if (Order::where('customer_id', $customerId)->where('status', 'pending')->count() > 0)

// GOOD
if (Order::where('customer_id', $customerId)->where('status', 'pending')->exists())
```

---

## Caching

### Cache Hierarchy

Apply caching at the layer closest to the consumer, falling back to deeper layers:

1. **Browser cache** — static assets, HTTP cache headers.
2. **CDN cache** — edge-cached HTML, API responses, assets.
3. **Application cache** — computed values, serialised query results (Redis, Memcached).
4. **Database query cache** — query results cached by the ORM or database (use sparingly — invalidation is hard).

### Application-Level Caching

**Django:**

```python
from django.core.cache import cache

def get_dashboard_stats(tenant_id: int) -> dict:
    cache_key = f"dashboard_stats:{tenant_id}"
    stats = cache.get(cache_key)
    if stats is None:
        stats = _compute_dashboard_stats(tenant_id)
        cache.set(cache_key, stats, timeout=300)  # 5 minutes
    return stats
```

**Laravel:**

```php
use Illuminate\Support\Facades\Cache;

function getDashboardStats(int $tenantId): array
{
    return Cache::remember(
        "dashboard_stats:{$tenantId}",
        now()->addMinutes(5),
        fn () => computeDashboardStats($tenantId)
    );
}
```

**Rules:**

- Always include the tenant ID (or user ID, or relevant scope) in cache keys. A shared cache key for tenant-scoped data is a data breach.
- Set explicit TTL (time to live) on every cached value. Never cache without an expiry.
- Use a consistent key naming convention: `{resource}:{scope}:{identifier}` (e.g., `orders:tenant_42:recent`).
- Cache serialised data, not ORM objects. ORM objects carry database connections, lazy-loaded relationships, and state that does not serialise cleanly.

### HTTP Caching

Set appropriate cache headers on API and asset responses:

```
# Static assets — cache aggressively with content hashing
Cache-Control: public, max-age=31536000, immutable

# API responses — no caching by default
Cache-Control: no-store

# API responses that can be cached — short TTL with revalidation
Cache-Control: private, max-age=60, must-revalidate
ETag: "abc123"
```

**Rules:**

- Static assets with content hashes in the filename (`app.a1b2c3.js`) should be cached for one year with `immutable`.
- API responses that return user-specific data must use `Cache-Control: private` or `no-store`.
- Never cache responses that contain authentication tokens or sensitive data.

### Cache Invalidation

Cache invalidation is the hardest problem. Minimise its complexity:

- **Time-based expiry** (TTL) is the simplest strategy. Use it where stale data is acceptable for a short period.
- **Event-based invalidation**: clear the cache when the underlying data changes. Use Django signals, Laravel observers, or explicit service methods.
- **Cache tags** (Laravel): group related cache entries under a tag and flush them all at once.

```php
// Laravel — tagged cache
Cache::tags(['tenant:42', 'orders'])->put('orders:recent', $orders, 300);

// Invalidate all order caches for tenant 42
Cache::tags(['tenant:42', 'orders'])->flush();
```

- **Never cache and forget.** Every cached value must have either a TTL or an explicit invalidation trigger. If you cannot define when a cached value becomes stale, do not cache it.

---

## Frontend Performance

### Bundle Size

- Set a performance budget for JavaScript bundle size. Recommended maximum: **200 KB gzipped** for the initial page load.
- Enforce the budget in CI. Fail the build if the bundle exceeds the budget.
- Use `source-map-explorer` or `webpack-bundle-analyzer` to identify large dependencies.
- Before adding a dependency, check its size with [bundlephobia](https://bundlephobia.com). A 50 KB library for a 20-line function is not acceptable (see `CODING-PRINCIPLES.md` — Dependencies).

### Code Splitting and Lazy Loading

Split the bundle so that users only download the code they need for the current page:

```typescript
// React — lazy load routes
const OrdersPage = lazy(() => import("./pages/OrdersPage"));
const SettingsPage = lazy(() => import("./pages/SettingsPage"));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/orders" element={<OrdersPage />} />
        <Route path="/settings" element={<SettingsPage />} />
      </Routes>
    </Suspense>
  );
}
```

**Rules:**

- Lazy load every route-level component.
- Lazy load heavy libraries (chart libraries, rich text editors, PDF generators) that are not needed on every page.
- Use `Suspense` with a meaningful loading state (skeleton screen, not a spinner).
- Do not lazy load components that are visible above the fold on the initial page load.

### Rendering Strategy

Choose the right rendering strategy for each page:

| Strategy | Use when | Framework |
|----------|----------|-----------|
| Static Generation (SSG) | Content rarely changes (marketing, docs) | Next.js `generateStaticParams` |
| Server-Side Rendering (SSR) | Content is personalised or changes frequently | Next.js server components, Inertia.js |
| Client-Side Rendering (CSR) | Highly interactive content behind authentication | React SPA |
| Incremental Static Regeneration (ISR) | Content changes periodically (blog, catalogue) | Next.js `revalidate` |

**Rules:**

- Default to server-side rendering for SEO-critical pages.
- Default to client-side rendering for authenticated, interactive pages.
- Use static generation for content that does not change between requests.

### React Performance Patterns

- **Avoid unnecessary re-renders.** Use `React.memo()` for components that receive the same props frequently. Do not memo everything — only components where re-rendering is measurably expensive.
- **Stabilise references.** Use `useMemo` for expensive computations and `useCallback` for event handlers passed to memoised children. Do not use them for simple values — the overhead of memoisation can exceed the cost of recalculation.
- **Virtualise long lists.** For lists with hundreds or thousands of items, use `react-window` or `@tanstack/virtual` to render only the visible items.
- **Debounce expensive operations.** Search inputs, resize handlers, and scroll handlers should be debounced (200–300ms).

---

## Image and Asset Optimisation

- Use modern formats: **WebP** for photographs, **SVG** for icons and illustrations, **AVIF** where browser support is sufficient.
- Serve responsive images with `srcset` and `sizes` attributes. Do not serve a 2000px image to a 400px viewport.
- Lazy load images below the fold: `<img loading="lazy" />`.
- Set explicit `width` and `height` attributes (or aspect-ratio in CSS) to prevent layout shift (CLS).
- Compress images at build time or via a CDN image transformation service.
- Strip EXIF metadata from user-uploaded images for both privacy and file size. See `SECURITY.md` — File Upload Security.

---

## API and Network Performance

- **Minimise request count.** Prefer one request that returns all needed data over multiple requests. GraphQL is well-suited to this.
- **Compress responses.** Enable gzip or Brotli compression on all text-based responses (JSON, HTML, CSS, JS).
- **Use connection keep-alive.** HTTP/2 multiplexing eliminates the need for domain sharding.
- **Avoid over-fetching.** Return only the fields the consumer needs. In REST, consider sparse fieldsets (`?fields=id,status,total`). In GraphQL, clients inherently request only what they need.
- **Set timeouts.** Every outbound HTTP call must have a connect timeout (5s) and read timeout (30s). Never make unbounded HTTP calls.

---

## Background Jobs and Queues

Move slow or unreliable work out of the request/response cycle:

- **Email sending** — queue it. A failed email send should not fail the user's request.
- **Webhook delivery** — queue with retries and exponential back-off.
- **Report generation** — queue and notify the user when complete.
- **Image processing** — queue and serve a placeholder until processing is complete.
- **Third-party API calls** — queue if the response is not needed immediately.

**Laravel:**

```php
// Dispatch to queue
SendOrderConfirmation::dispatch($order)->onQueue('emails');

// Job class
class SendOrderConfirmation implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $backoff = 60; // seconds between retries

    public function __construct(private Order $order) {}

    public function handle(): void
    {
        Mail::to($this->order->customer->email)->send(new OrderConfirmed($this->order));
    }
}
```

**Django (Celery):**

```python
from celery import shared_task

@shared_task(bind=True, max_retries=3, default_retry_delay=60)
def send_order_confirmation(self, order_id: int) -> None:
    try:
        order = Order.objects.select_related("customer").get(id=order_id)
        send_mail(...)
    except Exception as exc:
        self.retry(exc=exc)
```

**Rules:**

- Every queued job must be idempotent. If it runs twice, the result is the same as running once.
- Every queued job must have a `max_retries` and a `backoff` strategy.
- Failed jobs must be logged and monitored. A failed job that nobody notices is worse than no job at all.
- Do not pass large objects to queued jobs. Pass IDs and let the job fetch the data.

---

## Connection Pooling

Database connections are expensive to establish. Use connection pooling to reuse them.

**Django:**

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "CONN_MAX_AGE": 600,  # reuse connections for 10 minutes
        "CONN_HEALTH_CHECKS": True,  # verify connections before reuse (Django 4.1+)
        # ... other settings
    }
}
```

For higher connection volumes, use PgBouncer as an external connection pooler between Django and PostgreSQL.

**Laravel:**

Laravel's database connection pooling is managed through the config. For MariaDB/MySQL, connection reuse is handled by the persistent connections setting:

```php
// config/database.php
'mysql' => [
    'driver' => 'mysql',
    'options' => [
        PDO::ATTR_PERSISTENT => true, // reuse connections
    ],
    // ...
],
```

For high-traffic applications, use ProxySQL or PgBouncer as external poolers.

**Rules:**

- Set `CONN_MAX_AGE` (Django) or persistent connections (Laravel) in all environments.
- Monitor connection counts. If the database is approaching its connection limit, add an external pooler rather than increasing the limit.
- Close connections explicitly in long-running processes (management commands, queue workers) that may hold connections indefinitely.

---

## Mobile Performance (React Native)

- **Minimise bridge crossings.** In React Native, communication between JavaScript and native code crosses a "bridge" that has overhead. Batch operations where possible.
- **Use FlatList, not ScrollView**, for long lists. FlatList only renders visible items.
- **Avoid inline functions in render.** Create stable references with `useCallback` to prevent unnecessary re-renders of list items.
- **Optimise images.** Use `react-native-fast-image` for caching. Serve appropriately sized images for the device's screen density.
- **Profile with Flipper or React Native DevTools.** Use the performance monitor to identify janky frames (below 60 FPS).
- **Minimise JS bundle size.** Use Hermes engine (enabled by default in Expo SDK 50+) for faster startup and lower memory usage.

---

## Monitoring and Measurement

You cannot improve what you do not measure.

### Metrics to track

| Metric | Target | Tool |
|--------|--------|------|
| Time to First Byte (TTFB) | < 200ms | Server monitoring |
| Largest Contentful Paint (LCP) | < 2.5s | Lighthouse, Web Vitals |
| First Input Delay (FID) | < 100ms | Web Vitals |
| Cumulative Layout Shift (CLS) | < 0.1 | Lighthouse, Web Vitals |
| API response time (p95) | < 500ms | APM (Sentry, New Relic) |
| Database query time (p95) | < 50ms | APM, slow query log |
| JS bundle size (gzipped) | < 200 KB initial | CI budget check |
| Error rate | < 0.1% | Error tracking (Sentry) |

### Production monitoring

- Enable slow query logging in PostgreSQL (`log_min_duration_statement = 200`) and MariaDB/MySQL (`slow_query_log = 1`, `long_query_time = 0.2`).
- Use an APM tool (Sentry Performance, New Relic, Datadog) to track request durations, database query counts, and external API call times.
- Set up alerts for p95 response time exceeding the target and error rate exceeding the threshold.

---

## Load Testing

See `TESTING.md` — Performance and Load Testing for tools and examples. Additional performance-specific guidance:

### When to load test

- Before any release that changes database queries, caching, or serialisation in a high-traffic path.
- After adding a new tenant or significantly increasing the user base.
- When introducing a new backing service.
- When a production incident is traced to a performance regression.

### Load testing rules

- Test against a staging environment with production-like data volume. Do not test against an empty database.
- Establish baselines for critical endpoints. A regression of more than 20% in p95 response time requires investigation.
- Test with realistic concurrency patterns, not just maximum throughput. Simulate ramp-up, sustained load, and spike scenarios.
- Include database-heavy operations (reports, exports, search) in load tests, not just lightweight endpoints.

---

## Performance Checklist

Before deploying performance-sensitive changes:

- [ ] N+1 queries are eliminated (`select_related`/`prefetch_related` in Django, `with()` in Laravel)
- [ ] Lazy loading prevention is enabled in development (`preventLazyLoading` in Laravel, `nplusone` in Django)
- [ ] All collection endpoints are paginated with enforced limits
- [ ] `EXPLAIN ANALYZE` confirms that hot queries use indexes, not full table scans
- [ ] Cache keys include tenant/user scope where data is scoped
- [ ] Cached values have explicit TTL or invalidation triggers
- [ ] JavaScript bundle size is within the 200 KB gzipped budget
- [ ] Route-level components are lazy loaded
- [ ] Images use modern formats, responsive sizes, and `loading="lazy"` below the fold
- [ ] Background jobs are idempotent with retries and back-off
- [ ] Database connection pooling is configured
- [ ] API responses are compressed (gzip/Brotli)
- [ ] Core Web Vitals targets are met (LCP < 2.5s, FID < 100ms, CLS < 0.1)
