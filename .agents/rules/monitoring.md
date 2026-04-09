# Monitoring & Observability — Laravel

> Standards for logging, metrics, tracing, and alerting in a Laravel application.

## 🔭 The Three Pillars of Observability

| Pillar | Tool | Purpose |
|--------|------|---------|
| **Logs** | Laravel Log (Monolog) + Sentry | What happened |
| **Dev Inspection** | Laravel Telescope | Queries, jobs, requests (local/staging) |
| **Metrics** | Prometheus + Grafana (advanced) | How the system is behaving |
| **Traces** | Sentry Performance | Why something is slow |

---

## 📝 Logging Rules

### Log Levels
| Level | When to Use |
|-------|-------------|
| `emergency` | System is unusable |
| `critical` | Application crash, payment failure |
| `error` | Unexpected failure requiring attention |
| `warning` | Unexpected but recoverable situation |
| `info` | Normal significant events (job dispatched, order placed) |
| `debug` | Detailed debugging info (dev only) |

### Log Format — Structured with Context
```php
// ✅ Structured log — searchable and parseable
use Illuminate\Support\Facades\Log;

Log::info('Order placed', [
    'event'       => 'order.placed',
    'order_id'    => $order->id,
    'user_id'     => $order->user_id,
    'amount'      => $order->total,
    'duration_ms' => now()->diffInMilliseconds($startTime),
]);

// ❌ Unstructured log — cannot be queried
Log::info("Order {$orderId} placed by user {$userId}");
```

### What NOT to Log
```php
// ❌ Never log sensitive data
Log::info('Login', ['password' => $request->password]);       // NEVER
Log::info('Auth', ['token' => $request->bearerToken()]);      // NEVER
Log::info('Payment', ['card' => $request->card_number]);      // NEVER
```

### Log Channel Configuration (config/logging.php)
```php
'channels' => [
    'stack' => [
        'driver'   => 'stack',
        'channels' => ['daily', 'sentry'],
    ],
    'daily' => [
        'driver' => 'daily',
        'path'   => storage_path('logs/laravel.log'),
        'level'  => env('LOG_LEVEL', 'error'),
        'days'   => 14,
    ],
],
```

---

## 🔭 Laravel Telescope (Dev/Staging)

```bash
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate
```

Telescope provides automatic inspection of:
- HTTP requests and responses
- Eloquent queries (with N+1 detection)
- Queued jobs, failed jobs
- Cache hits/misses
- Scheduled tasks
- Exceptions

**Rule**: Enable Telescope only in `local` and `staging` environments. **Never in production.**

```php
// app/Providers/TelescopeServiceProvider.php
public function register(): void
{
    Telescope::night(); // dark mode

    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        if ($this->app->environment('local')) {
            return true;
        }
        return $entry->isReportableException() ||
               $entry->isFailedJob() ||
               $entry->isSlowQuery(); // > 100ms
    });
}
```

---

## 📊 Metrics

### Key Metrics to Track
```
# Pattern: {namespace}_{subsystem}_{name}_{unit}

http_request_duration_seconds     # histogram
http_requests_total               # counter
db_query_duration_seconds         # histogram
cache_hits_total / misses_total   # counters
queue_jobs_total (+ status label) # counter
queue_processing_duration_seconds # histogram
```

### Laravel Horizon (Queue Monitoring)
```bash
composer require laravel/horizon
php artisan horizon:install
php artisan horizon  # start the dashboard
```
Horizon provides: job throughput, failure rates, queue depth, worker status.

---

## 🚨 Alerting Rules

### Severity Levels
| Level | Response Time | Example |
|-------|--------------|---------|
| `critical` | Immediate | Service down, payment failures |
| `warning` | Within 30min | High error rate, slow queries |
| `info` | Business hours | Unusual traffic patterns |

### Sentry Integration (Production)
```bash
composer require sentry/sentry-laravel
php artisan sentry:publish --dsn=https://...
```

```php
// config/sentry.php
'sample_rate'             => env('SENTRY_TRACES_SAMPLE_RATE', 0.1),
'profiles_sample_rate'    => env('SENTRY_PROFILES_SAMPLE_RATE', 0.1),
'send_default_pii'        => false, // never send PII
```

### Health Check Route
```php
// routes/web.php
Route::get('/health', function () {
    return response()->json([
        'status'   => 'ok',
        'database' => DB::connection()->getPdo() ? 'connected' : 'error',
        'cache'    => Cache::store('redis')->set('health', 1) ? 'connected' : 'error',
        'time'     => now()->toIso8601String(),
    ]);
})->name('health');
```

---

## 🔍 Performance Monitoring

### Slow Query Detection
```php
// AppServiceProvider::boot()
DB::listen(function ($query) {
    if ($query->time > 1000) { // > 1 second
        Log::warning('Slow query detected', [
            'sql'      => $query->sql,
            'duration' => $query->time . 'ms',
        ]);
    }
});
```

### N+1 Detection (dev)
```php
// AppServiceProvider::boot() — dev only
if (app()->environment('local')) {
    \Illuminate\Database\Eloquent\Model::preventLazyLoading();
}
```
