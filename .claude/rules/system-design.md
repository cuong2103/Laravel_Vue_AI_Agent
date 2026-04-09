# System Design Rules

> Principles from "System Design Interview" (Alex Xu), "Designing Data-Intensive Applications" (Martin Kleppmann), and industry best practices.

## 🏗️ Core Principles

### CAP Theorem
- A distributed system can only guarantee 2 of 3: **Consistency**, **Availability**, **Partition Tolerance**
- **CP systems** (sacrifice availability): MySQL, ZooKeeper → use when data correctness is critical
- **AP systems** (sacrifice consistency): Cassandra, DynamoDB → use for high availability

### Design for Failure
- Every external call CAN fail — design for it
- Use **circuit breakers** to prevent cascade failures
- Use **bulkhead pattern** to isolate failures
- Implement **graceful degradation** (serve cached/partial data when dependencies fail)

---

## 📐 Scalability Patterns

### Horizontal vs Vertical Scaling
```
Vertical  → More CPU/RAM on one machine (limited ceiling)
Horizontal → More machines (preferred for production)
```

### Load Balancing
- **Round Robin** — equal distribution
- **Least Connections** — route to least busy server
- **Consistent Hashing** — for cache/session affinity (minimize remapping on scale)

### Caching Strategies
| Strategy | Use Case |
|----------|----------|
| **Cache-Aside** (Lazy Loading) | Read-heavy, content changes frequently |
| **Write-Through** | Write-heavy, data must be consistent |
| **Write-Behind** (Write-Back) | High write throughput, some lag acceptable |
| **Read-Through** | Cache is always up to date |

```php
// Cache-Aside (Laravel default pattern)
$product = Cache::remember("product:{$id}", 3600, fn () => Product::find($id));
```

### Database Scaling
- **Read Replicas** — scale read-heavy workloads (configure in `config/database.php`)
- **Sharding** — partition data horizontally by shard key
- **CQRS** — separate read model and write model

---

## 🔄 Async Patterns

### Message Queues (Laravel Queue)
Use queues when:
- Processing can be deferred
- Tasks are slow/expensive (email, PDF gen, notifications)
- You need to decouple producers from consumers

```
Producer → [Laravel Queue / Redis] → Consumer(s)
```

### Event-Driven Architecture (Laravel Events)
```php
// Publish domain events instead of direct service calls
event(new OrderPlaced($order));

// Other listeners subscribe independently
class SendOrderConfirmationEmail
{
    public function handle(OrderPlaced $event): void
    {
        Mail::to($event->order->user)->send(new OrderConfirmation($event->order));
    }
}
```

### Saga Pattern (Distributed Transactions)
- **Choreography** — each service reacts to events (no central coordinator)
- **Orchestration** — a saga orchestrator tells each service what to do

---

## 🗄️ Database Design (MySQL + Eloquent)

### Normalization vs Denormalization
- **Normalize** (OLTP) — avoid data duplication, use JOINs via Eloquent relationships
- **Denormalize** (OLAP/Read-heavy) — duplicate data to eliminate JOINs

### Indexing Rules
```php
// ✅ Index columns used in WHERE, JOIN, ORDER BY
$table->index(['email']);
$table->index(['user_id', 'created_at']); // composite
$table->unique('email');

// ❌ Don't index every column — indexes slow down writes
```

### N+1 Query Prevention (Eloquent)
```php
// ❌ N+1: one query per user
$orders = Order::all();
foreach ($orders as $order) {
    echo $order->user->name; // queries DB on every iteration!
}

// ✅ Eager loading — single query with JOIN
$orders = Order::with(['user', 'items.product'])->get();
```

---

## 🌐 API Design Patterns

### Rate Limiting Algorithms
| Algorithm | Best For |
|-----------|----------|
| **Token Bucket** | Burst traffic allowed (API gateways) |
| **Leaky Bucket** | Smooth/constant output rate |
| **Fixed Window** | Simple, but spiky at window boundaries |
| **Sliding Window** | Most accurate, higher memory cost |

Laravel's `throttle` middleware uses sliding window rate limiting.

### Idempotency
- GET, PUT, DELETE must be idempotent
- POST operations: use **idempotency keys** for payment/critical actions

### Pagination Patterns
```php
// Offset pagination (simple, use for small datasets)
$users = User::paginate(20); // ?page=2&per_page=20

// Cursor pagination (✅ preferred for large datasets)
$users = User::cursorPaginate(20);
```

---

## 🔒 Reliability

### Circuit Breaker States
```
CLOSED → OPEN (after N failures) → HALF-OPEN (probe) → CLOSED
```

### Retry Strategy (Laravel Queue)
```php
// Exponential backoff with jitter — built into Laravel Queue
class ProcessPayment implements ShouldQueue
{
    public int $tries = 3;
    public array $backoff = [10, 30, 60]; // seconds between retries
}
```

### Health Checks
```
GET /health       → Basic liveness (is the service running?)
GET /health/ready → Readiness (is the service ready to serve traffic?)
```

---

## 📊 Back-of-Envelope Estimation

| Resource | Approximate Speed |
|----------|------------------|
| L1 cache reference | 0.5 ns |
| RAM access | 100 ns |
| Redis GET | < 1 ms |
| SSD random read | 150 μs |
| MySQL indexed query | 1–5 ms |
| Network round trip (same DC) | 0.5 ms |
| Network round trip (cross continent) | 150 ms |

> Rule of thumb: Prefer Redis (in-memory) over DB for anything needing < 1ms latency
