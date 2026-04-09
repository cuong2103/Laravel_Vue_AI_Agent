# Technology Stack — Selection & Standards

> This rule defines the **approved tech stack** for all projects. When starting a new project or proposing a new dependency, follow the decision criteria below.

---

## 🗂️ Quick Reference — Approved Stack

| Layer | Primary Choice | Alternative | Avoid |
|-------|---------------|-------------|-------|
| **Backend Framework** | Laravel 12 (PHP 8.3) | — | CodeIgniter, Symfony (unless needed) |
| **Frontend — SPA/Inertia** | Vue 3 + Inertia.js | — | React, Angular |
| **Frontend — Standalone SPA** | Vue 3 + Vite | Nuxt.js (SSR/SEO) | Next.js (for this stack) |
| **UI Components** | Reka UI (shadcn-vue) | PrimeVue | Bootstrap (too heavy) |
| **Styling** | Tailwind CSS v4 | CSS Modules | Styled-components |
| **State Management** | Pinia (global) + ref/reactive (local) | — | Vuex (legacy) |
| **Data Fetching** | Inertia shared props (default) | TanStack Query for Vue | Axios alone |
| **API Style** | REST (default) / Inertia (fullstack) | — | GraphQL (unless needed) |
| **Language (PHP)** | PHP 8.3 (typed properties, enums, fibers) | — | PHP < 8.2 |
| **Language (JS)** | TypeScript 5+ | — | Plain JavaScript |
| **Database** | MySQL 8 | PostgreSQL | SQLite (prod) |
| **ORM** | Eloquent (Laravel built-in) | — | Doctrine, raw PDO |
| **Validation** | FormRequest (Laravel) | — | Manual array validation |
| **Cache** | Redis (Laravel Cache facade) | — | Memcached |
| **Queue** | Laravel Queue (Redis driver) | — | BullMQ (Node-only) |
| **Auth** | Laravel Sanctum (API tokens) / Fortify (web) | Laravel Passport (OAuth2) | JWT manual |
| **File Storage** | AWS S3 / Cloudflare R2 (Laravel Filesystem) | Local disk (dev only) | |
| **Email** | Resend / SMTP via Laravel Mail | Mailgun | SendGrid (expensive) |
| **Search** | MySQL FULLTEXT (start here) | Meilisearch / Algolia | Elasticsearch (unless needed) |
| **Monitoring** | Laravel Telescope (dev) + Sentry (prod) | Datadog | — |
| **Logging** | Laravel Log (Monolog, structured JSON) | — | error_log() |
| **Testing** | PHPUnit + Pest | — | Codeception |
| **E2E Testing** | Playwright | Cypress | Selenium |
| **CI/CD** | GitHub Actions | — | Jenkins (legacy) |
| **Containerization** | Docker + Docker Compose | Laravel Sail | — |
| **Deployment** | Forge + DigitalOcean/AWS | Railway, Fly.io | Manual SSH |
| **API Docs** | Swagger / OpenAPI 3.0 (scribe) | — | Postman collections only |

---

## 🖥️ Frontend — Choosing the Right Approach

### Decision Table

| Criteria | Vue 3 + Inertia.js | Nuxt.js (SSR) | Vue 3 + Vite (pure SPA) |
|----------|-------------------|---------------|------------------------|
| **Use case** | Laravel fullstack app | Marketing/SEO-heavy | Admin panel, dashboard |
| **SEO** | ⚠️ SSR optional via Inertia | ✅ Native SSR | ❌ Client-only |
| **Backend** | Laravel (required) | Any | Any REST API |
| **Complexity** | Low | Medium | Low |
| **Auth** | Laravel Fortify/Sanctum | Nuxt Auth | Manual JWT |

> **Rule**: This project uses **Vue 3 + Inertia.js** as the default fullstack approach.

---

## 🗄️ Database — MySQL + Eloquent

### Why MySQL
- Battle-tested, widely supported by hosting providers
- Excellent Laravel/Eloquent support
- JSON column type for flexible data
- Full-text search built-in

### Eloquent Model Conventions
```php
// app/Models/User.php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = ['name', 'email', 'password'];

    protected $hidden = ['password', 'remember_token'];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password'          => 'hashed',
    ];

    // Relationships
    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }
}
```

### Migration Conventions
```php
// database/migrations/2024_01_01_000001_create_users_table.php
Schema::create('users', function (Blueprint $table) {
    $table->id();                         // ✅ Auto-increment bigint PK
    $table->string('email')->unique();
    $table->string('name');
    $table->string('password');
    $table->enum('role', ['user', 'admin'])->default('user');
    $table->timestamp('email_verified_at')->nullable();
    $table->rememberToken();
    $table->softDeletes();                // deleted_at
    $table->timestamps();                 // created_at, updated_at

    $table->index(['email']);             // ✅ Index on searched columns
});
```

### Migration Workflow
```bash
# Create migration
php artisan make:migration add_role_to_users_table

# Run migrations
php artisan migrate

# Rollback last migration
php artisan migrate:rollback

# Fresh migration + seed
php artisan migrate:fresh --seed
```

---

## ⚡ Cache — Redis + Laravel Cache

### Why Redis
- Sub-millisecond latency
- Powers caching + queues + rate limiting + sessions

### Cache Helper Pattern
```php
// Using Laravel Cache facade
use Illuminate\Support\Facades\Cache;

// Get or set (cache-aside pattern)
$user = Cache::remember("user:{$id}:profile", 3600, function () use ($id) {
    return User::find($id);
});

// Invalidate
Cache::forget("user:{$id}:profile");
Cache::tags(['users'])->flush();
```

---

## 📨 Queue — Laravel Queue (Redis)

### Queue Job Pattern
```php
// Dispatch a job
SendWelcomeEmail::dispatch($user);
SendWelcomeEmail::dispatch($user)->onQueue('emails')->delay(now()->addMinutes(5));

// Run queue worker
php artisan queue:work redis --queue=emails,default --tries=3
```

### Queue Decision
| Use Case | Solution |
|---------|----------|
| Emails, notifications, PDFs | Laravel Queue (Redis driver) |
| Scheduled tasks | Laravel Scheduler (`app/Console/Kernel.php`) |
| High-throughput events/streaming | Consider Kafka (advanced) |

---

## 📄 Documentation

### API Documentation — OpenAPI / Swagger
```bash
composer require knuckleswtf/scribe
php artisan scribe:generate
```

- Every API endpoint MUST have OpenAPI annotations or docblocks
- Mount at `/api-docs`

### README Template (mandatory for every service)
```markdown
# Service Name

## What it does (1-2 sentences)

## Tech Stack
- Runtime: PHP 8.3
- Framework: Laravel 12
- Database: MySQL 8 (Eloquent)
- Cache: Redis
- Frontend: Vue 3 + Inertia.js

## Quick Start
\`\`\`bash
cp .env.example .env
composer install && npm install
php artisan key:generate
php artisan migrate --seed
npm run dev
\`\`\`

## Environment Variables → see .env.example
## API Documentation → /api-docs
## Architecture → docs/architecture.md
```

---

## ✅ Technology Decision Process

When **proposing a new library or technology**, evaluate against these criteria:

| Criterion | Questions to ask |
|-----------|-----------------|
| **Necessity** | Does an approved alternative already solve this? |
| **Maintenance** | Stars > 1k? Last commit < 6 months? |
| **Laravel Compatibility** | Does it integrate with Laravel DI/Service Container? |
| **PHP Version** | Is it compatible with PHP 8.3+? |
| **License** | Is it MIT/Apache? (No GPL in commercial products) |
| **Security** | `composer audit` — zero high/critical vulnerabilities |
| **Community** | Active issues/discussions? Laracasts coverage? |
