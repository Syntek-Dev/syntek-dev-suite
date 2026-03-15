# Data Structures

**Last Updated:** 15/03/2026
**Version:** 1.8.0
**Maintained By:** Development Team
**Language:** British English (en_GB)
**Timezone:** Europe/London
**Plugin Scope:** syntek-dev-suite (Python/Django, PHP/Laravel, TypeScript/React, React Native)

---

## Table of Contents

- [Overview](#overview)
- [The Principle](#the-principle)
- [Fundamental Structures and When to Use Them](#fundamental-structures-and-when-to-use-them)
  - [Lists and Arrays](#lists-and-arrays)
  - [Dictionaries and Maps](#dictionaries-and-maps)
  - [Sets](#sets)
  - [Tuples and Frozen Structures](#tuples-and-frozen-structures)
  - [Queues and Stacks](#queues-and-stacks)
  - [Trees](#trees)
  - [Graphs](#graphs)
- [Choosing the Right Structure](#choosing-the-right-structure)
- [Domain Modelling](#domain-modelling)
  - [Models Reflect the Business](#models-reflect-the-business)
  - [Value Objects](#value-objects)
  - [Enumerations and Status Fields](#enumerations-and-status-fields)
  - [Aggregates and Boundaries](#aggregates-and-boundaries)
  - [GraphQL Types as Domain Contracts](#graphql-types-as-domain-contracts)
  - [TypeScript Interfaces as Domain Contracts](#typescript-interfaces-as-domain-contracts)
- [Database Schema Design](#database-schema-design)
  - [Normalisation](#normalisation)
  - [Denormalisation](#denormalisation)
  - [Indexes](#indexes)
  - [Foreign Keys and Constraints](#foreign-keys-and-constraints)
  - [Migrations](#migrations)
  - [Soft Deletes](#soft-deletes)
  - [Polymorphic Relationships](#polymorphic-relationships)
  - [JSON Fields](#json-fields)
  - [Multi-Tenancy Patterns](#multi-tenancy-patterns)
- [Anti-Patterns](#anti-patterns)
  - [The God Dictionary](#the-god-dictionary)
  - [Stringly Typed Data](#stringly-typed-data)
  - [Primitive Obsession](#primitive-obsession)
  - [Parallel Collections](#parallel-collections)
  - [Nested Dicts as Poor Man's Objects](#nested-dicts-as-poor-mans-objects)
  - [The Mega-Model](#the-mega-model)
  - [Implicit Schema](#implicit-schema)
  - [Comma-Separated Values in a Column](#comma-separated-values-in-a-column)
  - [Boolean Blindness](#boolean-blindness)
  - [Overloaded Status Fields](#overloaded-status-fields)
- [Refactoring Toward Better Structures](#refactoring-toward-better-structures)
  - [Recognising the Signals](#recognising-the-signals)
  - [Refactoring Strategies](#refactoring-strategies)
- [Rules and Principles](#rules-and-principles)

---

## Overview

This guide covers how to choose, design, and maintain data structures across the syntek-dev-suite stack (Python/Django, PHP/Laravel, TypeScript/React, React Native). It starts with fundamental structures, moves through domain modelling and database schema design, and finishes with anti-patterns and refactoring guidance.

It exists because data structure decisions are the highest-leverage decisions in a codebase. Get the data right and the algorithms, queries, and UI components follow naturally. Get the data wrong and every layer of the application becomes a workaround.

Database examples cover both PostgreSQL and MariaDB/MySQL, noting differences where they matter.

---

## The Principle

Rob Pike and Linus Torvalds both express the same idea:

> "If you have chosen the right data structures and organised things well, the algorithms will almost always be self-evident." -- Rob Pike

> "Show me your tables, and I won't usually need your flowcharts; they'll be obvious." -- Linus Torvalds (via Fred Brooks)

This means: before writing logic, design the data. Before optimising an algorithm, question the data structure it operates on. Before adding a conditional, ask whether the structure should make the condition unnecessary.

---

## Fundamental Structures and When to Use Them

### Lists and Arrays

An ordered, indexed collection. Use when order matters and you need to iterate, slice, or access elements by position.

**Python:**

```python
# Good -- ordered collection where position matters
stages = ["draft", "review", "published", "archived"]
current_index = stages.index(article.status)
next_stage = stages[current_index + 1]
```

**PHP:**

```php
// Good -- ordered collection where position matters
$stages = ['draft', 'review', 'published', 'archived'];
$currentIndex = array_search($article->status, $stages);
$nextStage = $stages[$currentIndex + 1];
```

**TypeScript:**

```typescript
// Good -- ordered items rendered in sequence
const steps: Step[] = [
  { id: "contact", label: "Contact Details" },
  { id: "payment", label: "Payment Method" },
  { id: "confirm", label: "Confirmation" },
];
```

**When to avoid:** if you are frequently searching for an element by value (`if x in large_list` / `in_array($x, $largeArray)`), you need a set or dictionary instead. Linear search on a list is O(n); lookup in a set or dict is O(1).

### Dictionaries and Maps

An unordered (or insertion-ordered in Python 3.7+ and PHP 7.0+) collection of key-value pairs. Use when you need fast lookup by a known key. In PHP, associative arrays serve this role.

**Python:**

```python
# Good -- O(1) lookup by a known identifier
permissions_by_role = {
    "admin": {"read", "write", "delete", "manage_users"},
    "editor": {"read", "write"},
    "viewer": {"read"},
}

user_permissions = permissions_by_role[user.role]
```

**PHP:**

```php
// Good -- associative array for O(1) lookup
$permissionsByRole = [
    'admin'  => ['read', 'write', 'delete', 'manage_users'],
    'editor' => ['read', 'write'],
    'viewer' => ['read'],
];

$userPermissions = $permissionsByRole[$user->role];
```

**TypeScript:**

```typescript
// Good -- constant-time status lookup
const statusLabels: Record<OrderStatus, string> = {
  pending: "Awaiting Payment",
  paid: "Payment Received",
  shipped: "Dispatched",
  delivered: "Delivered",
};
```

**When to avoid:** if you are iterating over all keys and values every time you access the structure, you may not need a dictionary -- a list of objects might be clearer.

### Sets

An unordered collection of unique values. Use when you need fast membership testing, deduplication, or set operations (union, intersection, difference).

**Python:**

```python
# Good -- fast membership check and set operations
required_permissions = {"read", "write"}
user_permissions = {"read", "write", "delete"}

if required_permissions.issubset(user_permissions):
    allow_access()

# Deduplication
unique_tags = set(tag.lower() for tag in raw_tags)
```

**PHP:**

PHP has no native set type. Use `array_unique()` for deduplication and `array_flip()` for O(1) membership testing on large collections.

```php
// Good -- flip for fast membership check
$allowedRoles = array_flip(['admin', 'editor', 'moderator']);

if (isset($allowedRoles[$user->role])) {
    allowAccess();
}

// Deduplication
$uniqueTags = array_unique(array_map('strtolower', $rawTags));
```

**TypeScript:**

```typescript
// Good -- deduplication and fast lookup
const selectedIds = new Set<string>(["id_1", "id_3", "id_5"]);

if (selectedIds.has(item.id)) {
  // item is selected
}
```

**When to avoid:** if you need to preserve insertion order and also check membership, in Python a dict with dummy values (`dict.fromkeys(items)`) preserves order while providing O(1) lookup. In TypeScript, use a `Map` or maintain a parallel `Set` alongside an array if order matters for rendering.

### Tuples and Frozen Structures

An immutable, ordered collection. Use when a group of values belongs together and should not be modified after creation.

**Python:**

```python
# Good -- fixed structure, used as a dict key or in a set
Coordinate = tuple[float, float]
location: Coordinate = (51.5074, -0.1278)

# Named tuples for clarity
from typing import NamedTuple

class PageRange(NamedTuple):
    start: int
    end: int

visible = PageRange(start=1, end=25)
```

**PHP:**

PHP has no native tuple or immutable array. Use readonly classes (PHP 8.2+) to achieve immutability.

```php
readonly class Coordinate
{
    public function __construct(
        public float $latitude,
        public float $longitude,
    ) {}
}

$london = new Coordinate(51.5074, -0.1278);
// $london->latitude = 0; // Error -- cannot modify readonly property
```

**TypeScript:**

```typescript
// Good -- fixed-length, typed tuple
type LatLng = [latitude: number, longitude: number];
const london: LatLng = [51.5074, -0.1278];

// Use `as const` for literal tuples
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = (typeof ROLES)[number]; // "admin" | "editor" | "viewer"
```

**When to avoid:** if the structure has more than three or four fields, use a dataclass (Python) or an interface/type (TypeScript) instead. Tuples with many positional fields become unreadable.

### Queues and Stacks

**Queue (FIFO):** first in, first out. Use for task processing, message handling, and breadth-first traversal.

**Stack (LIFO):** last in, first out. Use for undo operations, depth-first traversal, and expression parsing.

**Python:**

```python
from collections import deque

# Queue -- task processing
task_queue: deque[Task] = deque()
task_queue.append(new_task)       # enqueue
next_task = task_queue.popleft()  # dequeue -- O(1)

# Stack -- undo history
undo_stack: list[Action] = []
undo_stack.append(action)         # push
last_action = undo_stack.pop()    # pop -- O(1)
```

**PHP:**

```php
// Queue -- SplQueue provides O(1) enqueue/dequeue
$queue = new SplQueue();
$queue->enqueue($newTask);
$nextTask = $queue->dequeue();

// Stack -- plain array
$stack = [];
$stack[] = $action;                    // push
$lastAction = array_pop($stack);       // pop -- O(1)
```

**Important:** never use `array_shift()` for a queue in PHP -- it is O(n) because every element is re-indexed. Use `SplQueue` instead.

**Important:** never use `list.pop(0)` for a queue in Python -- it is O(n) because every element shifts. Use `collections.deque` which provides O(1) operations at both ends.

**TypeScript:**

```typescript
// Queue pattern in React state
const [queue, setQueue] = useState<Notification[]>([]);

// Enqueue
setQueue((prev) => [...prev, newNotification]);

// Dequeue
const [next, ...rest] = queue;
setQueue(rest);
```

### Trees

A hierarchical structure where each node has zero or more children. Use for category hierarchies, nested navigation, comment threads, organisational structures, and file systems.

**Python (Django -- self-referential model):**

```python
class Category(models.Model):
    name = models.CharField(max_length=200)
    parent = models.ForeignKey(
        "self",
        null=True,
        blank=True,
        on_delete=models.CASCADE,
        related_name="children",
    )

    class Meta:
        verbose_name_plural = "categories"
```

For deep trees with frequent ancestor/descendant queries, consider `django-mptt` or `django-treebeard` which use materialised path or nested set algorithms for efficient tree queries without recursive CTEs.

**PHP (Laravel -- self-referential model):**

```php
class Category extends Model
{
    public function parent(): BelongsTo
    {
        return $this->belongsTo(Category::class, 'parent_id');
    }

    public function children(): HasMany
    {
        return $this->hasMany(Category::class, 'parent_id');
    }

    public function descendants(): HasMany
    {
        return $this->children()->with('descendants');
    }
}
```

For deep trees in Laravel, consider `staudenmeir/laravel-adjacency-list` for recursive CTE support. Note: MariaDB/MySQL supports recursive CTEs from version 10.2.2 / 8.0 respectively.

**TypeScript (recursive type):**

```typescript
interface TreeNode<T> {
  data: T;
  children: TreeNode<T>[];
}

interface NavItem {
  label: string;
  href: string;
}

type NavTree = TreeNode<NavItem>;
```

### Graphs

A structure where nodes (vertices) connect to other nodes via edges, with no hierarchy constraint. Use for social networks, dependency resolution, routing, and workflow systems where relationships are many-to-many and potentially cyclic.

Most web applications do not need an explicit graph data structure in application code. If relationships are stored in the database, the graph is implicit in foreign keys and join tables. Use an explicit graph structure only when you need to perform graph algorithms (shortest path, cycle detection, topological sort) in application memory.

**Python:**

```python
from collections import defaultdict

# Adjacency list -- simplest graph representation
dependencies: dict[str, set[str]] = defaultdict(set)
dependencies["syntek-auth"].add("syntek-crypto")
dependencies["syntek-auth"].add("syntek-core")
dependencies["syntek-crypto"].add("syntek-core")

# Topological sort for build order
from graphlib import TopologicalSorter

sorter = TopologicalSorter(dependencies)
build_order = list(sorter.static_order())
```

---

## Choosing the Right Structure

When selecting a data structure, answer these questions in order:

**1. What operations are most frequent?**

| Primary operation | Best structure | Python | PHP | TypeScript |
|-------------------|---------------|--------|-----|------------|
| Access by index / position | List / Array | `list` | `array` (indexed) | `Array` |
| Lookup by key | Dictionary / Map | `dict` | `array` (associative) | `Record` / `Map` |
| Membership check | Set | `set` | `array_flip()` + `isset()` | `Set` |
| Ordered iteration | List / Array | `list` | `array` | `Array` |
| FIFO processing | Queue | `deque` | `SplQueue` | `Array` (shift/push) |
| LIFO processing | Stack | `list` | `array` + `array_pop` | `Array` (push/pop) |
| Hierarchical navigation | Tree | self-referential model | self-referential model | recursive interface |
| Many-to-many with traversal | Graph | `dict[str, set]` | associative arrays | `Map<string, Set>` |

**2. Does mutability matter?**

If the data should not change after creation, use an immutable structure: `tuple`, `frozenset`, `NamedTuple`, `@dataclass(frozen=True)` in Python; `readonly` class or `readonly` properties in PHP 8.2+; `as const`, `Readonly<T>`, `ReadonlyArray<T>` in TypeScript. Immutability prevents accidental mutation bugs, makes data safe to use as dictionary keys or set members, and simplifies reasoning about state in React components.

**3. Is uniqueness required?**

If duplicates must be prevented, use a set (values) or a dictionary (keyed data). Do not enforce uniqueness by checking a list before inserting -- use a structure that guarantees it.

**4. How large will the collection grow?**

For small collections (under a few hundred elements), the difference between O(1) and O(n) is negligible. For large or unbounded collections (user records, event logs, search results), choose structures with the right complexity for your access patterns.

**5. Will this structure cross a boundary?**

If data will be serialised (to JSON, to the database, over an API), choose structures that serialise cleanly. Prefer plain dicts, lists, and dataclasses over custom objects with complex internal state. In TypeScript, prefer plain objects and arrays over `Map` and `Set` when the data will be passed as props or sent over the network.

---

## Domain Modelling

### Models Reflect the Business

A domain model should use the same language as the business. If stakeholders call it a "booking", the model is `Booking`, not `Reservation` or `Appointment`. If the business distinguishes between a "lead" and a "customer", those are two models, not one model with a `type` field.

**Python (Django):**

```python
# Good -- model names match the business language
class Booking(models.Model):
    guest = models.ForeignKey("Guest", on_delete=models.CASCADE)
    property = models.ForeignKey("Property", on_delete=models.CASCADE)
    check_in = models.DateField()
    check_out = models.DateField()
    status = models.CharField(max_length=20, choices=BookingStatus.choices)
```

**PHP (Laravel):**

```php
class Booking extends Model
{
    protected $fillable = ['guest_id', 'property_id', 'check_in', 'check_out', 'status'];

    protected $casts = [
        'check_in'  => 'date',
        'check_out' => 'date',
        'status'    => BookingStatus::class,
    ];

    public function guest(): BelongsTo
    {
        return $this->belongsTo(Guest::class);
    }

    public function property(): BelongsTo
    {
        return $this->belongsTo(Property::class);
    }
}
```

**Bad -- generic names that do not communicate intent:**

```python
# Bad -- what is an "item"? What is "data"?
class Item(models.Model):
    data = models.JSONField()
    type = models.CharField(max_length=50)
    ref = models.CharField(max_length=100)
```

### Value Objects

A value object is a small, immutable object defined by its attributes rather than its identity. Two value objects with the same attributes are considered equal. Use them for concepts like money, addresses, date ranges, and coordinates.

**Python:**

```python
from dataclasses import dataclass
from decimal import Decimal


@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("Amount cannot be negative")
        if len(self.currency) != 3:
            raise ValueError("Currency must be a 3-letter ISO code")

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError(f"Cannot add {self.currency} and {other.currency}")
        return Money(amount=self.amount + other.amount, currency=self.currency)


# Usage -- the structure enforces correctness
price = Money(amount=Decimal("19.99"), currency="GBP")
shipping = Money(amount=Decimal("3.50"), currency="GBP")
total = price.add(shipping)
```

**PHP:**

```php
readonly class Money
{
    public function __construct(
        public string $amount,
        public string $currency,
    ) {
        if (bccomp($this->amount, '0', 2) < 0) {
            throw new \InvalidArgumentException('Amount cannot be negative');
        }
        if (strlen($this->currency) !== 3) {
            throw new \InvalidArgumentException('Currency must be a 3-letter ISO code');
        }
    }

    public function add(self $other): self
    {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException("Cannot add {$this->currency} and {$other->currency}");
        }
        return new self(bcadd($this->amount, $other->amount, 2), $this->currency);
    }
}
```

**TypeScript:**

```typescript
interface Money {
  readonly amount: number;
  readonly currency: string;
}

interface DateRange {
  readonly start: Date;
  readonly end: Date;
}

function addMoney(a: Money, b: Money): Money {
  if (a.currency !== b.currency) {
    throw new Error(`Cannot add ${a.currency} and ${b.currency}`);
  }
  return { amount: a.amount + b.amount, currency: a.currency };
}
```

The benefit is that logic about "what is valid money?" lives inside the value object, not scattered across every function that handles prices.

### Enumerations and Status Fields

Use explicit enumerations for any field with a fixed set of valid values. Never use raw strings or magic numbers.

**Python (Django):**

```python
from django.db import models


class BookingStatus(models.TextChoices):
    PENDING = "pending", "Pending"
    CONFIRMED = "confirmed", "Confirmed"
    CHECKED_IN = "checked_in", "Checked In"
    CHECKED_OUT = "checked_out", "Checked Out"
    CANCELLED = "cancelled", "Cancelled"


class Booking(models.Model):
    status = models.CharField(
        max_length=20,
        choices=BookingStatus.choices,
        default=BookingStatus.PENDING,
    )

    def confirm(self):
        if self.status != BookingStatus.PENDING:
            raise ValueError(f"Cannot confirm a booking with status {self.status}")
        self.status = BookingStatus.CONFIRMED
        self.save(update_fields=["status"])
```

**PHP (Laravel -- backed enum, PHP 8.1+):**

```php
enum BookingStatus: string
{
    case Pending = 'pending';
    case Confirmed = 'confirmed';
    case CheckedIn = 'checked_in';
    case CheckedOut = 'checked_out';
    case Cancelled = 'cancelled';
}

class Booking extends Model
{
    protected $casts = ['status' => BookingStatus::class];

    public function confirm(): void
    {
        if ($this->status !== BookingStatus::Pending) {
            throw new \DomainException("Cannot confirm a booking with status {$this->status->value}");
        }
        $this->update(['status' => BookingStatus::Confirmed]);
    }
}
```

**TypeScript:**

```typescript
const BOOKING_STATUS = {
  PENDING: "pending",
  CONFIRMED: "confirmed",
  CHECKED_IN: "checked_in",
  CHECKED_OUT: "checked_out",
  CANCELLED: "cancelled",
} as const;

type BookingStatus = (typeof BOOKING_STATUS)[keyof typeof BOOKING_STATUS];
```

State transitions should be explicit methods on the model or service, not arbitrary string assignments. If `booking.status = "anything"` is possible, the structure is not protecting you.

### Aggregates and Boundaries

An aggregate is a cluster of domain objects treated as a single unit for data changes. External code interacts with the aggregate root, never directly with its internals. This enforces consistency.

**Python (Django):**

```python
class Order(models.Model):
    """Aggregate root. All modifications to order lines go through Order methods."""
    customer = models.ForeignKey("Customer", on_delete=models.CASCADE)
    status = models.CharField(max_length=20, choices=OrderStatus.choices)

    def add_line(self, product: "Product", quantity: int) -> "OrderLine":
        if self.status != OrderStatus.DRAFT:
            raise ValueError("Cannot modify a non-draft order")
        line = OrderLine.objects.create(
            order=self, product=product, quantity=quantity, unit_price=product.price,
        )
        return line

    def remove_line(self, line_id: int) -> None:
        if self.status != OrderStatus.DRAFT:
            raise ValueError("Cannot modify a non-draft order")
        self.lines.filter(id=line_id).delete()

    @property
    def total(self) -> Decimal:
        return sum(
            (line.unit_price * line.quantity for line in self.lines.all()),
            Decimal("0.00"),
        )


class OrderLine(models.Model):
    """Part of the Order aggregate. Never modify directly -- use Order methods."""
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name="lines")
    product = models.ForeignKey("Product", on_delete=models.PROTECT)
    quantity = models.PositiveIntegerField()
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)
```

**PHP (Laravel):**

```php
class Order extends Model
{
    public function lines(): HasMany
    {
        return $this->hasMany(OrderLine::class);
    }

    public function addLine(Product $product, int $quantity): OrderLine
    {
        if ($this->status !== OrderStatus::Draft) {
            throw new \DomainException('Cannot modify a non-draft order');
        }
        return $this->lines()->create([
            'product_id' => $product->id,
            'quantity'   => $quantity,
            'unit_price' => $product->price,
        ]);
    }

    public function removeLine(int $lineId): void
    {
        if ($this->status !== OrderStatus::Draft) {
            throw new \DomainException('Cannot modify a non-draft order');
        }
        $this->lines()->where('id', $lineId)->delete();
    }
}
```

The rule: if you find yourself writing `OrderLine::create(...)` or `OrderLine.objects.create(...)` outside of the `Order` model, the aggregate boundary is being violated.

### GraphQL Types as Domain Contracts

GraphQL types define the shape of data that clients can request. They should reflect the domain, not the database schema.

```python
import strawberry
from decimal import Decimal


@strawberry.type
class OrderType:
    id: strawberry.ID
    status: str
    total: Decimal
    lines: list["OrderLineType"]

    @strawberry.field
    def can_be_cancelled(self) -> bool:
        return self.status in ("pending", "confirmed")


@strawberry.type
class OrderLineType:
    product_name: str
    quantity: int
    line_total: Decimal
```

Note that `OrderLineType` exposes `product_name` and `line_total`, not `product_id` and `unit_price`. The GraphQL type is a domain contract, not a database mirror. Clients should not need to perform calculations or follow foreign keys.

### TypeScript Interfaces as Domain Contracts

On the frontend, define interfaces that match the domain, not the API response shape. If the API returns snake_case, transform at the boundary and use camelCase domain types throughout the application.

```typescript
// Domain type -- used throughout the app
interface Order {
  id: string;
  status: BookingStatus;
  total: number;
  lines: OrderLine[];
  canBeCancelled: boolean;
}

interface OrderLine {
  productName: string;
  quantity: number;
  lineTotal: number;
}

// API response type -- used only at the fetch boundary
interface OrderApiResponse {
  id: string;
  status: string;
  total: string;
  lines: {
    product_name: string;
    quantity: number;
    line_total: string;
  }[];
}

// Transform at the boundary
function toOrder(raw: OrderApiResponse): Order {
  return {
    id: raw.id,
    status: raw.status as BookingStatus,
    total: parseFloat(raw.total),
    canBeCancelled: ["pending", "confirmed"].includes(raw.status),
    lines: raw.lines.map((line) => ({
      productName: line.product_name,
      quantity: line.quantity,
      lineTotal: parseFloat(line.line_total),
    })),
  };
}
```

The transformation happens once, at the API boundary. Every component, hook, and utility function works with the clean domain type.

### Laravel API Resources as Domain Contracts

In Laravel, API Resources transform Eloquent models into domain-shaped JSON without leaking database structure.

```php
class OrderResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'               => $this->id,
            'status'           => $this->status->value,
            'total'            => $this->getTotal(),
            'can_be_cancelled' => in_array($this->status, [OrderStatus::Pending, OrderStatus::Confirmed]),
            'lines'            => OrderLineResource::collection($this->lines),
        ];
    }
}

class OrderLineResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'product_name' => $this->product->name,
            'quantity'     => $this->quantity,
            'line_total'   => bcmul($this->unit_price, (string) $this->quantity, 2),
        ];
    }
}
```

The resource exposes `product_name` and `line_total`, not `product_id` and `unit_price`. The API contract is a domain view, not a database dump.

---

## Database Schema Design

### PostgreSQL vs MariaDB/MySQL

Most principles in this section apply to both PostgreSQL and MariaDB/MySQL. Where they differ:

| Feature | PostgreSQL | MariaDB/MySQL |
|---------|-----------|---------------|
| JSON support | `jsonb` -- binary, indexable, GIN indexes | `JSON` -- native from MariaDB 10.2 / MySQL 5.7, fewer operators, no GIN indexes |
| Partial indexes | Supported (`WHERE` clause on index) | Not supported. Use generated columns + regular index as a workaround |
| Check constraints | Fully enforced | Enforced from MariaDB 10.2.1 / MySQL 8.0.16. Older versions parse but silently ignore them |
| Recursive CTEs | Supported | Supported from MariaDB 10.2.2 / MySQL 8.0 |
| UUID primary keys | `uuid` type, `gen_random_uuid()` | `CHAR(36)` or `BINARY(16)`. Use `ORDERED_UUID` in Laravel for InnoDB performance |
| Collation | Default `en_US.UTF-8` | Always set `utf8mb4` charset and `utf8mb4_unicode_ci` collation explicitly |

**MariaDB/MySQL-specific rules:**

- Always use the InnoDB storage engine. Never use MyISAM -- it does not support transactions, foreign keys, or row-level locking.
- Always set the charset to `utf8mb4` and collation to `utf8mb4_unicode_ci`. MySQL's `utf8` is not real UTF-8 -- it only supports 3-byte characters and will silently truncate emoji.
- InnoDB's clustered index uses the primary key for physical row order. Use auto-incrementing integers or `ORDERED_UUID` -- standard UUIDs (v4) cause write amplification due to random insertion patterns.
- MariaDB/MySQL defaults to `REPEATABLE READ` isolation with gap locking. This can cause unexpected deadlocks on range queries in concurrent workloads.

### Normalisation

Normalisation eliminates redundancy by ensuring each piece of data is stored in exactly one place. Follow these normal forms as a baseline:

**First Normal Form (1NF):** every column contains a single, atomic value. No lists, no comma-separated strings, no arrays of mixed types.

```sql
-- BAD: violates 1NF
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    name TEXT,
    tags TEXT  -- "music,outdoor,free"
);

-- GOOD: separate table
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    name TEXT
);

CREATE TABLE event_tags (
    event_id INTEGER REFERENCES events(id) ON DELETE CASCADE,
    tag TEXT NOT NULL,
    PRIMARY KEY (event_id, tag)
);
```

**Second Normal Form (2NF):** every non-key column depends on the entire primary key, not just part of it. This matters for composite keys.

**Third Normal Form (3NF):** every non-key column depends on the primary key and nothing else. No transitive dependencies.

```python
# BAD: city depends on postcode, not on the user
class User(models.Model):
    name = models.CharField(max_length=200)
    postcode = models.CharField(max_length=10)
    city = models.CharField(max_length=100)  # derived from postcode

# GOOD: separate the address or look up city from postcode
class User(models.Model):
    name = models.CharField(max_length=200)
    address = models.ForeignKey("Address", on_delete=models.SET_NULL, null=True)
```

**Laravel:**

```php
// BAD: city depends on postcode -- transitive dependency
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('postcode', 10);
    $table->string('city', 100); // derived from postcode
});

// GOOD: separate address table
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->foreignId('address_id')->nullable()->constrained()->nullOnDelete();
});
```

**Rule of thumb:** normalise by default. Denormalise deliberately when you have measured a performance problem (see below).

### Denormalisation

Denormalisation introduces controlled redundancy to avoid expensive joins or calculations. It is a performance optimisation, not a design shortcut.

**When to denormalise:**

- A value is read orders of magnitude more often than it is written, and computing it requires joining multiple tables.
- A dashboard or report query is unacceptably slow even with proper indexes.
- A field is needed for full-text search or filtering and reconstructing it from normalised data on every query is impractical.

**How to denormalise safely:**

```python
class Order(models.Model):
    # Normalised: lines contain the source of truth
    # Denormalised: cached total for fast reads
    cached_total = models.DecimalField(
        max_digits=10, decimal_places=2, default=Decimal("0.00"),
        help_text="Denormalised. Updated by recalculate_total().",
    )

    def recalculate_total(self) -> None:
        self.cached_total = (
            self.lines.aggregate(
                total=models.Sum(models.F("unit_price") * models.F("quantity"))
            )["total"]
            or Decimal("0.00")
        )
        self.save(update_fields=["cached_total"])
```

**Laravel:**

```php
class Order extends Model
{
    public function recalculateTotal(): void
    {
        $this->update([
            'cached_total' => $this->lines->sum(
                fn (OrderLine $line) => bcmul($line->unit_price, (string) $line->quantity, 2)
            ),
        ]);
    }
}

// Keep in sync with an observer
class OrderLineObserver
{
    public function saved(OrderLine $line): void { $line->order->recalculateTotal(); }
    public function deleted(OrderLine $line): void { $line->order->recalculateTotal(); }
}
```

**Rules:**

- Always document which field is denormalised and which method/signal/observer keeps it in sync.
- The denormalised field must never be the only source of truth. If it becomes inconsistent, the system must be able to recalculate it from the normalised data.
- Use Django signals, Laravel observers, or explicit service methods to update denormalised fields. Do not rely on application code remembering to call `recalculate_total()` -- make it automatic.

### Indexes

Indexes speed up reads at the cost of slower writes and additional storage. Add them deliberately, not speculatively.

**When to add an index:**

- A column appears in `WHERE`, `ORDER BY`, or `JOIN` clauses in queries that run frequently or against large tables.
- You have measured slow query performance with `EXPLAIN ANALYZE` (PostgreSQL) or `EXPLAIN FORMAT=JSON` (MariaDB/MySQL) and the query plan shows sequential/full table scans on the relevant column.

**When NOT to add an index:**

- The table is small (under a few thousand rows). The database will use a sequential/full scan regardless.
- The column has very low cardinality (e.g., a boolean `is_active` with roughly 50/50 distribution). A partial index (PostgreSQL) or a generated column with an index (MariaDB/MySQL) may help instead.
- You are guessing. Run `EXPLAIN ANALYZE` first.

**Django:**

```python
class Order(models.Model):
    customer = models.ForeignKey("Customer", on_delete=models.CASCADE)
    status = models.CharField(max_length=20, choices=OrderStatus.choices)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        indexes = [
            # Composite index for the most common query pattern
            models.Index(fields=["customer", "-created_at"], name="idx_order_customer_date"),
            # Partial index for active orders only
            models.Index(
                fields=["status"],
                name="idx_order_active",
                condition=models.Q(status__in=["pending", "confirmed", "shipped"]),
            ),
        ]
```

**Laravel:**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('customer_id')->constrained();
    $table->string('status', 20)->default('pending');
    $table->timestamps();

    $table->index(['customer_id', 'created_at'], 'idx_order_customer_date');
    $table->index('status', 'idx_order_status');
});
```

**MariaDB/MySQL index notes:**

- InnoDB automatically creates an index on every foreign key column.
- The leftmost prefix rule applies: an index on `(customer_id, created_at)` can satisfy queries on `customer_id` alone, but not on `created_at` alone.
- MariaDB/MySQL does not support partial indexes. Use generated columns and index those instead.

**Rules:**

- Every index must correspond to a real query pattern. If no query uses it, remove it.
- Composite indexes: put the most selective column first. The column that narrows the result set the most should be the leftmost in the index.
- Review indexes quarterly. As query patterns change, indexes may become redundant or new ones may be needed.

### Foreign Keys and Constraints

Use database-level constraints to enforce data integrity. Application-level validation is not a substitute -- it can be bypassed by direct database access, data migrations, and bugs.

```python
class OrderLine(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product = models.ForeignKey("Product", on_delete=models.PROTECT)  # prevent deleting products with orders
    quantity = models.PositiveIntegerField()  # enforces >= 0 at DB level
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=models.Q(quantity__gte=1),
                name="orderline_quantity_positive",
            ),
            models.UniqueConstraint(
                fields=["order", "product"],
                name="orderline_unique_product_per_order",
            ),
        ]
```

**Laravel:**

```php
Schema::create('order_lines', function (Blueprint $table) {
    $table->id();
    $table->foreignId('order_id')->constrained()->cascadeOnDelete();
    $table->foreignId('product_id')->constrained()->restrictOnDelete();
    $table->unsignedInteger('quantity');
    $table->decimal('unit_price', 10, 2);
    $table->unique(['order_id', 'product_id'], 'orderline_unique_product_per_order');
});

// For MariaDB 10.2.1+ / MySQL 8.0.16+:
DB::statement('ALTER TABLE order_lines ADD CONSTRAINT orderline_quantity_positive CHECK (quantity >= 1)');
```

**Note:** MariaDB enforces `CHECK` constraints from 10.2.1, MySQL from 8.0.16. Older versions silently ignore them.

**`on_delete` choices (Django) / foreign key actions (Laravel):**

| Django / Laravel | Use when |
|--------|----------|
| `CASCADE` / `cascadeOnDelete()` | The child has no meaning without the parent |
| `PROTECT` / `restrictOnDelete()` | Deletion should be prevented if children exist |
| `SET_NULL` / `nullOnDelete()` | The relationship is optional |
| `SET_DEFAULT` / (manual SQL) | Rare. The child should fall back to a default parent |
| `DO_NOTHING` / (no action) | Almost never. Only if you handle referential integrity in application code |

**Rules:**

- Every foreign key must have an explicit `on_delete`. Never rely on the default.
- Use `PROTECT` for any relationship where deleting the parent would lose important data.
- Use database-level `CheckConstraint` and `UniqueConstraint` for invariants that must always hold, regardless of how data enters the system.

### Migrations

Migrations are the version history of the database schema. They must be treated with the same care as application code.

**Rules:**

- Every model change produces a migration. Never modify the database schema manually.
- Migrations must be forwards-compatible. A deployment that rolls out new code before running migrations must not break. This means: add new columns as nullable or with defaults before code starts using them; deploy the code that uses the column; then tighten constraints in a later migration.
- Never modify a migration that has been applied to staging or production. Create a new migration instead.
- **Django:** data migrations (`RunPython`) must have both a forward and reverse function.
- **Laravel:** use separate migration files for schema changes and data changes. Data migrations should be idempotent.
- Test migrations against a real database instance (PostgreSQL via testcontainers, MariaDB/MySQL via testcontainers or a local instance). See `TESTING.md`.
- Large data migrations on production tables should be run as background tasks or batched operations to avoid locking the table. On MariaDB/MySQL, consider `pt-online-schema-change` or `gh-ost` for zero-downtime schema changes on large tables.

### Soft Deletes

Soft deletes mark a record as deleted without removing it from the database. Use them when you need to preserve data for audit, compliance, or undo purposes.

```python
class SoftDeleteModel(models.Model):
    deleted_at = models.DateTimeField(null=True, blank=True, db_index=True)

    class Meta:
        abstract = True

    def soft_delete(self):
        self.deleted_at = timezone.now()
        self.save(update_fields=["deleted_at"])

    def restore(self):
        self.deleted_at = None
        self.save(update_fields=["deleted_at"])


class SoftDeleteManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(deleted_at__isnull=True)


class Booking(SoftDeleteModel):
    objects = SoftDeleteManager()       # default: excludes soft-deleted
    all_objects = models.Manager()      # includes soft-deleted

    # ...
```

**Laravel:**

Laravel provides soft deletes out of the box via the `SoftDeletes` trait:

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Booking extends Model
{
    use SoftDeletes;
    // $booking->delete()        -- soft deletes (sets deleted_at)
    // $booking->restore()       -- restores
    // $booking->forceDelete()   -- hard deletes
    // Booking::withTrashed()    -- includes soft-deleted
    // Booking::onlyTrashed()    -- only soft-deleted
}
```

**Rules:**

- The default query scope must exclude soft-deleted records. Code that needs deleted records must explicitly use `all_objects`.
- Soft-deleted records must still respect unique constraints. Use partial unique indexes that exclude deleted records, or include `deleted_at` in the uniqueness check.
- Define a retention policy. Soft-deleted records should be hard-deleted after a defined period (e.g., 90 days) via a scheduled task.

### Polymorphic Relationships

When different types of objects share a relationship (e.g., comments on both articles and events), avoid the generic foreign key anti-pattern where possible.

**Prefer separate foreign keys with a constraint:**

```python
class Comment(models.Model):
    article = models.ForeignKey("Article", null=True, blank=True, on_delete=models.CASCADE)
    event = models.ForeignKey("Event", null=True, blank=True, on_delete=models.CASCADE)
    body = models.TextField()

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=(
                    models.Q(article__isnull=False, event__isnull=True)
                    | models.Q(article__isnull=True, event__isnull=False)
                ),
                name="comment_single_parent",
            ),
        ]
```

This approach gives you real foreign keys, real joins, and real referential integrity. `GenericForeignKey` (Django's content types framework) loses all of these.

**Laravel's `morphTo()`:** Laravel's polymorphic relationships (`morphTo`, `morphMany`) are convenient but store a string type discriminator and an integer ID without any foreign key constraint. The database cannot enforce referential integrity. Use polymorphic relationships only when the number of parent types is unbounded.

**Use `GenericForeignKey` (Django) or `morphTo` (Laravel) only when:** the number of possible parent types is unbounded and growing (e.g., an activity log that tracks changes to any model). Even then, consider whether a JSON field with a type discriminator is simpler.

### JSON Fields

PostgreSQL's `jsonb` and MariaDB/MySQL's `JSON` types are useful for semi-structured data that varies between records and does not need to be joined, filtered, or aggregated at the database level.

**Good uses:**

```python
class Integration(models.Model):
    """Each integration provider has different config shapes."""
    provider = models.CharField(max_length=50)
    config = models.JSONField(default=dict)
    # config might be {"api_key": "...", "webhook_url": "..."} for Stripe
    # or {"channel": "...", "bot_token": "..."} for Slack
```

**Bad uses:**

```python
# BAD: structured data that should be relational
class Order(models.Model):
    lines = models.JSONField()  # [{"product_id": 1, "qty": 2}, ...]
    # Cannot enforce foreign keys, cannot join, cannot index, cannot aggregate
```

**MariaDB/MySQL JSON notes:**

- MariaDB treats `JSON` as an alias for `LONGTEXT` with a `CHECK (JSON_VALID(...))` constraint.
- Neither MariaDB nor MySQL supports GIN indexes on JSON. Extract frequently queried values into generated columns and index those.
- For frequently queried JSON paths, use generated columns:

```sql
ALTER TABLE integrations
  ADD COLUMN provider_key VARCHAR(100) AS (JSON_UNQUOTE(JSON_EXTRACT(config, '$.api_key'))) STORED,
  ADD INDEX idx_provider_key (provider_key);
```

**Rules:**

- If you need to `WHERE`, `JOIN`, `ORDER BY`, or `AGGREGATE` on a value, it belongs in a column, not in a JSON field.
- Validate JSON structure at the application level using a schema (Pydantic, Laravel validation, Zod).
- Document the expected shape of every JSON field in the model's docstring or a dedicated schema file.

### Multi-Tenancy Patterns

When the same application serves multiple clients (tenants), the schema must enforce data isolation.

**Row-level isolation (shared schema):**

```python
class TenantModel(models.Model):
    tenant = models.ForeignKey("Tenant", on_delete=models.CASCADE, db_index=True)

    class Meta:
        abstract = True


class TenantManager(models.Manager):
    def for_tenant(self, tenant):
        return self.filter(tenant=tenant)


class Booking(TenantModel):
    objects = TenantManager()
    # ...

    class Meta:
        constraints = [
            models.UniqueConstraint(
                fields=["tenant", "reference"],
                name="booking_unique_ref_per_tenant",
            ),
        ]
```

**Laravel -- row-level isolation with global scope:**

```php
trait BelongsToTenant
{
    protected static function bootBelongsToTenant(): void
    {
        static::addGlobalScope('tenant', function (Builder $builder) {
            if ($tenantId = auth()->user()?->tenant_id) {
                $builder->where('tenant_id', $tenantId);
            }
        });

        static::creating(function (Model $model) {
            if ($tenantId = auth()->user()?->tenant_id) {
                $model->tenant_id = $tenantId;
            }
        });
    }
}

class Booking extends Model
{
    use BelongsToTenant;
}
```

**Rules:**

- Every query that returns tenant data must be scoped to the authenticated tenant. A missing tenant filter is a data breach.
- Unique constraints must be tenant-scoped. A booking reference that is unique globally is too restrictive; one that is unique per tenant is correct.
- Background jobs and management commands must explicitly receive and use a tenant context. Never assume a "current tenant" in code that runs outside a request.
- Consider schema-level isolation (separate schemas per tenant in PostgreSQL) for clients with strict regulatory requirements (e.g., data residency). This adds operational complexity but provides stronger isolation guarantees.

---

## Anti-Patterns

### The God Dictionary / God Array

A single dictionary (Python), associative array (PHP), or object (TypeScript) that accumulates every piece of state for a feature, passed around through multiple functions, growing new keys as requirements change.

```python
# Bad -- what keys does this dict have? What types? Nobody knows.
def process_order(context: dict) -> dict:
    context["total"] = sum(item["price"] for item in context["items"])
    context["tax"] = context["total"] * Decimal("0.2")
    context["processed"] = True
    return context
```

**PHP:**

```php
// Bad -- what keys does this array have?
function processOrder(array &$context): array {
    $context['total'] = collect($context['items'])->sum('price');
    $context['tax'] = $context['total'] * 0.2;
    $context['processed'] = true;
    return $context;
}
```

**Fix:** replace with a typed structure.

```python
@dataclass
class OrderContext:
    items: list[OrderItem]
    total: Decimal = Decimal("0.00")
    tax: Decimal = Decimal("0.00")
    processed: bool = False
```

### Stringly Typed Data

Using strings for values that have a finite set of valid options, or that represent structured data.

```python
# Bad
user.role = "admn"  # typo -- no error until runtime

# Good
user.role = UserRole.ADMIN  # typo caught by IDE and type checker
```

```php
// Bad
$user->role = 'admn'; // typo -- no error until runtime

// Good
$user->role = UserRole::Admin; // typo caught by IDE
```

```typescript
// Bad
type Status = string;

// Good
type Status = "pending" | "confirmed" | "cancelled";
```

### Primitive Obsession

Representing domain concepts as raw primitives (strings, numbers, booleans) instead of value objects.

```python
# Bad -- what unit is this? Pounds? Pence? Dollars?
def apply_discount(price: float, discount: float) -> float:
    return price - discount

# Good -- the type communicates the domain
def apply_discount(price: Money, discount: Money) -> Money:
    if price.currency != discount.currency:
        raise ValueError("Currency mismatch")
    return Money(amount=price.amount - discount.amount, currency=price.currency)
```

### Parallel Collections

Two or more lists that must be kept in sync by index.

```typescript
// Bad -- names[i] corresponds to emails[i]. What if they get out of sync?
const names: string[] = ["Alice", "Bob"];
const emails: string[] = ["alice@example.com", "bob@example.com"];

// Good -- one collection of structured objects
interface Contact {
  name: string;
  email: string;
}
const contacts: Contact[] = [
  { name: "Alice", email: "alice@example.com" },
  { name: "Bob", email: "bob@example.com" },
];
```

### Nested Dicts as Poor Man's Objects

Deeply nested dictionaries used in place of defined types.

```python
# Bad -- what is user["profile"]["preferences"]["notifications"]["email"]?
user = {
    "name": "Sam",
    "profile": {
        "preferences": {
            "notifications": {
                "email": True,
                "push": False,
            }
        }
    }
}

# Good -- defined types
@dataclass
class NotificationPreferences:
    email: bool = True
    push: bool = False

@dataclass
class UserProfile:
    notifications: NotificationPreferences = field(default_factory=NotificationPreferences)

@dataclass
class User:
    name: str
    profile: UserProfile = field(default_factory=UserProfile)
```

### The Mega-Model

A single Django model with 30+ fields covering multiple domain concepts.

```python
# Bad -- User model that is also a profile, billing record, and preferences store
class User(models.Model):
    email = models.EmailField()
    name = models.CharField(max_length=200)
    bio = models.TextField()
    avatar = models.ImageField()
    stripe_customer_id = models.CharField(max_length=100)
    subscription_plan = models.CharField(max_length=50)
    subscription_expires = models.DateTimeField()
    notification_email = models.BooleanField()
    notification_push = models.BooleanField()
    theme = models.CharField(max_length=20)
    language = models.CharField(max_length=10)
    # ... 20 more fields
```

**Fix:** split into focused models with one-to-one relationships.

**Django:**

```python
class User(models.Model):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=200)

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name="profile")
    bio = models.TextField(blank=True)
    avatar = models.ImageField(blank=True)

class Subscription(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name="subscription")
    stripe_customer_id = models.CharField(max_length=100)
    plan = models.CharField(max_length=50, choices=PlanChoices.choices)
    expires_at = models.DateTimeField()

class UserPreferences(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name="preferences")
    notification_email = models.BooleanField(default=True)
    notification_push = models.BooleanField(default=True)
    theme = models.CharField(max_length=20, default="system")
    language = models.CharField(max_length=10, default="en-gb")
```

**Laravel:**

```php
class User extends Model
{
    public function profile(): HasOne { return $this->hasOne(UserProfile::class); }
    public function subscription(): HasOne { return $this->hasOne(Subscription::class); }
    public function preferences(): HasOne { return $this->hasOne(UserPreferences::class); }
}
```

### Implicit Schema

Data structures whose shape is only known by reading the code that produces them, not by any type definition.

```typescript
// Bad -- what does fetchUser return? Who knows until you read the implementation.
const user = await fetchUser(id);
console.log(user.name); // might exist, might not

// Good -- the return type is explicit
async function fetchUser(id: string): Promise<User> { ... }
```

In Python, use type annotations on all function signatures. In TypeScript, avoid `any` -- use explicit interfaces. In PHP, use strict types, return type declarations, and typed properties.

### Comma-Separated Values in a Column

Storing multiple values in a single text column separated by commas.

```python
# Bad -- violates 1NF, cannot join, cannot index, cannot enforce integrity
class Article(models.Model):
    tags = models.CharField(max_length=500)  # "python,django,web"

# Good -- proper relationship
class Article(models.Model):
    tags = models.ManyToManyField("Tag", related_name="articles")

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
```

**Laravel:**

```php
// Bad -- violates 1NF
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('tags', 500); // "laravel,php,web"
});

// Good -- pivot table
Schema::create('article_tag', function (Blueprint $table) {
    $table->foreignId('article_id')->constrained()->cascadeOnDelete();
    $table->foreignId('tag_id')->constrained()->cascadeOnDelete();
    $table->primary(['article_id', 'tag_id']);
});
```

If you find yourself writing `.split(",")` or `explode(',', ...)` to read data from a model field, the schema is wrong.

### Boolean Blindness

Using a boolean when the actual domain has more than two states, or when the boolean's meaning is unclear at the call site.

```php
// Bad -- what does true, false mean here?
processOrder($order, true, false);

// Good -- use enums or named parameters
processOrder($order, priority: Priority::High, sendNotification: false);
```

```python
# Bad -- what does True mean here?
process_order(order, True, False)

# Good -- use enums or keyword arguments
process_order(order, priority=Priority.HIGH, send_notification=False)
```

```python
# Bad -- a boolean that will inevitably need a third state
class Order(models.Model):
    is_paid = models.BooleanField(default=False)
    # What about "partially paid"? "refunded"? "payment failed"?

# Good -- explicit status
class PaymentStatus(models.TextChoices):
    UNPAID = "unpaid"
    PENDING = "pending"
    PAID = "paid"
    PARTIALLY_REFUNDED = "partially_refunded"
    REFUNDED = "refunded"
    FAILED = "failed"

class Order(models.Model):
    payment_status = models.CharField(
        max_length=25, choices=PaymentStatus.choices, default=PaymentStatus.UNPAID,
    )
```

### Overloaded Status Fields

A single status field that encodes multiple independent dimensions.

```python
# Bad -- status encodes both payment state and fulfilment state
class Order(models.Model):
    status = models.CharField(max_length=20)
    # "paid_shipped", "paid_unshipped", "unpaid_unshipped", "refunded_returned"
    # Every combination is a new status. This does not scale.

# Good -- separate concerns into separate fields
class Order(models.Model):
    payment_status = models.CharField(max_length=20, choices=PaymentStatus.choices)
    fulfilment_status = models.CharField(max_length=20, choices=FulfilmentStatus.choices)
```

Two fields with 5 states each give you 25 combinations with 10 values. One combined field needs 25 separate values. Separate fields are easier to query, index, and reason about.

---

## Refactoring Toward Better Structures

### Recognising the Signals

The following code smells indicate that the data structure is wrong:

| Signal | Likely problem |
|--------|---------------|
| A function has many `if/elif` branches based on a `type` or `status` field | Use polymorphism or separate models |
| You are passing a dictionary through many functions, adding keys as you go | Replace with a dataclass or typed object |
| Two lists must be kept in sync by index | Merge into a single list of structured objects |
| A model has 20+ fields | Split into focused models with clear boundaries |
| You are writing `.split(",")` or `.join(",")` on a model field | Normalise into a related table |
| A function accepts `**kwargs` and uses string keys with no documentation | Define an explicit parameter type |
| The same `if` check appears in many places across the codebase | The missing check should be a constraint on the data structure itself |
| A test requires elaborate setup to construct the right shape of dict | The dict should be a typed structure with defaults |
| A GraphQL resolver or API serialiser reshapes data extensively | The domain model is too far from the API contract |
| You need a comment to explain what a variable contains | The type should be self-documenting |

### Refactoring Strategies

**1. Extract a value object.** When the same group of fields appears together across multiple models or functions (e.g., `amount` + `currency`, `start_date` + `end_date`), extract them into a value object (dataclass in Python, readonly class in PHP, interface in TypeScript).

**2. Replace dict with dataclass.** When a dictionary has a known, fixed shape, replace it with a dataclass (Python), readonly class / DTO (PHP), or interface (TypeScript). This gives you autocompletion, type checking, and documentation for free.

**3. Normalise the schema.** When a column contains multiple values (CSV, JSON array of IDs, pipe-separated strings), create a related table. Migrate data with a data migration that splits the values and creates rows in the new table.

**4. Split the mega-model.** When a model has too many fields, identify the distinct domain concepts within it (profile, billing, preferences, audit) and extract each into a separate model with a one-to-one relationship to the original.

**5. Replace boolean with enum.** When a boolean field is gaining exceptions or a third state is needed, replace it with a `TextChoices` enum (Django), backed enum (PHP 8.1+), or union type (TypeScript). Migrate existing data: `True` to the appropriate positive state, `False` to the appropriate negative state.

**6. Separate overloaded status.** When a status field encodes multiple independent dimensions, split it into separate fields -- one per dimension. Migrate existing combined values into the new separate fields.

**7. Introduce aggregate boundaries.** When external code directly creates, modifies, or deletes child objects (e.g., `OrderLine::create(...)` or `OrderLine.objects.create(...)` outside the `Order` model), move those operations into methods on the aggregate root. This is a code change, not a schema change, but it prevents the data structure from being put into an inconsistent state.

**General rule:** every refactoring that changes a data structure must include a migration (for database changes) and updated tests. Never change a data structure without verifying that all consumers of that structure still work correctly.

---

## Rules and Principles

1. Design the data before writing the logic. If the logic is complex, question the data structure.
2. Use the simplest structure that supports the required operations. Do not reach for a tree when a list will do.
3. Make the structure enforce correctness. If a value cannot be negative, use `PositiveIntegerField`, `unsignedInteger()`, or a `CHECK` constraint, not a validation function that might be bypassed.
4. Use explicit types. Every function parameter, return value, model field, and API response should have a declared type. No `any`, no untyped dicts, no `mixed` without documentation.
5. Name structures after the domain, not the implementation. `Booking`, not `DataRecord`. `PaymentStatus`, not `StatusEnum`.
6. Normalise by default. Denormalise deliberately, document it, and automate the synchronisation.
7. Every index must correspond to a measured query performance need. Do not add indexes speculatively.
8. Every foreign key must have an explicit `on_delete`. Every constraint must be enforced at the database level.
9. Transform data at boundaries. API responses are transformed into domain types at the fetch layer. Database records are transformed into domain objects at the ORM layer. The rest of the application works with clean domain types.
10. When the structure is wrong, fix the structure. Do not write clever code to work around a bad data model.
11. Always use InnoDB for MariaDB/MySQL. Always use `utf8mb4` charset with `utf8mb4_unicode_ci` collation.
12. Understand your database's constraint enforcement. MariaDB/MySQL `CHECK` constraints require version 10.2.1+ / 8.0.16+ -- verify your version and test that constraints are enforced, not silently ignored.
