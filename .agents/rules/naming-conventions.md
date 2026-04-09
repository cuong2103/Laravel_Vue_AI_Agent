# Naming Conventions — Laravel + Vue + MySQL

> Standard naming rules for cache keys, database identifiers, queues, events, environment variables, and more.

## 🔑 Cache Key Naming

### Format
```
{app}:{version}:{entity}:{identifier}:{variant}
```

### Rules
- Use **colons** (`:`) as separators
- Use **lowercase snake_case** for each segment
- Always prefix with app/service name to avoid collision
- Include version for easy cache invalidation

### Examples
```
# User data
myapp:v1:user:123:profile
myapp:v1:user:123:permissions

# Lists / collections
myapp:v1:users:active:list
myapp:v1:products:category:electronics:page:1

# Sessions
myapp:v1:session:abc123xyz

# Rate limiting
myapp:v1:rate_limit:user:123:api
myapp:v1:rate_limit:ip:192.168.1.1

# Temporary locks (mutex)
myapp:v1:lock:payment:order:99999
```

### TTL Conventions
| Data Type | Recommended TTL |
|-----------|----------------|
| User session | 7 days |
| Auth tokens | 15 minutes |
| User profile | 1 hour |
| Product catalog | 6 hours |
| Config/settings | 24 hours |
| Rate limit windows | 15 minutes |
| Temporary locks | 30 seconds |

```php
// Laravel Cache usage with TTL
Cache::remember("myapp:v1:user:{$id}:profile", 3600, fn () => User::find($id));
Cache::put("myapp:v1:rate_limit:ip:{$ip}", 1, 900); // 15 min
```

---

## 🗄️ Database Naming (MySQL)

### Tables
```sql
-- snake_case, plural nouns
users
order_items
product_categories
user_role_mappings    -- junction tables
```

### Columns
```sql
id                    -- primary key (always 'id', bigint unsigned)
user_id               -- foreign key: {referenced_table_singular}_id
created_at            -- timestamps (managed by Laravel automatically)
updated_at
deleted_at            -- soft delete (SoftDeletes trait)
is_active             -- booleans: is_, has_, can_
has_verified_email
email                 -- data fields: plain descriptive name
full_name
```

### Laravel Migration Conventions
```php
// Indexes
$table->index(['email']);
$table->index(['user_id', 'created_at']);  // composite
$table->unique('email');

// Foreign keys
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

---

## 📨 Laravel Queue — Job & Event Naming

### Queue Job Class Names
```php
// Pattern: Verb + Noun (PascalCase)
SendWelcomeEmail::class
ProcessPayment::class
GenerateInvoicePdf::class
ResizeProductImage::class
```

### Queue Names
```
# Pattern: {entity}-{action} (kebab-case)
emails
payments
notifications
reports
```

### Event Class Names (Domain Events)
```php
// Pattern: Noun + PastTenseVerb (PascalCase)
UserRegistered::class
OrderPlaced::class
PaymentReceived::class
OrderFulfilled::class
ProductStockDepleted::class
```

---

## 🌍 Environment Variables

### Rules
- **UPPER_SNAKE_CASE** for all env vars
- Laravel standard: `APP_*`, `DB_*`, `CACHE_*`, `QUEUE_*`

### Standard Variables
```bash
# App
APP_NAME=myapp
APP_ENV=production
APP_KEY=base64:...
APP_URL=https://myapp.com
LOG_LEVEL=warning

# Database (MySQL)
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp_production
DB_USERNAME=myapp_user
DB_PASSWORD=...

# Cache & Queue
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
CACHE_STORE=redis
QUEUE_CONNECTION=redis

# Auth
BCRYPT_ROUNDS=12
SANCTUM_STATEFUL_DOMAINS=localhost:3000

# External Services
MAIL_MAILER=smtp
MAIL_HOST=smtp.resend.com
MAIL_PORT=465
MAIL_USERNAME=resend
MAIL_PASSWORD=...
MAIL_FROM_ADDRESS=no-reply@myapp.com

AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_DEFAULT_REGION=ap-southeast-1
AWS_BUCKET=myapp-uploads

SENTRY_LARAVEL_DSN=...
```

---

## 📁 PHP File & Folder Naming

```
# Classes: PascalCase
UserService.php
OrderRepository.php
CreateUserRequest.php
SendWelcomeEmail.php     # Jobs

# Folders: PascalCase (Laravel convention)
app/Http/Controllers/
app/Services/
app/Repositories/
app/Models/
app/Jobs/

# Test files: PascalCase + Test suffix
tests/Unit/Services/UserServiceTest.php
tests/Feature/Api/UsersTest.php
```

---

## 📁 Vue/TypeScript File & Folder Naming

```
# Components: PascalCase.vue
UserCard.vue
OrderSummary.vue
AppSidebar.vue

# Pages (Inertia): PascalCase.vue
resources/js/pages/Dashboard.vue
resources/js/pages/users/Index.vue
resources/js/pages/users/Create.vue
resources/js/pages/users/Edit.vue

# Composables: use + PascalCase
composables/useAuth.ts
composables/useCart.ts

# Pinia Stores: use + PascalCase + Store
stores/useCartStore.ts
stores/useUserStore.ts
```

---

## 🌐 URL / Route Naming

```php
// REST: plural nouns, kebab-case, Laravel resource naming
Route::apiResource('users', UserController::class);
// Generates: GET /users, POST /users, GET /users/{user}, PUT/PATCH /users/{user}, DELETE /users/{user}

// Custom named routes
Route::get('/users/{user}/orders', [OrderController::class, 'index'])->name('users.orders.index');

// Auth
Route::post('/auth/login', [AuthController::class, 'login'])->name('auth.login');
Route::post('/auth/logout', [AuthController::class, 'logout'])->name('auth.logout');
```
