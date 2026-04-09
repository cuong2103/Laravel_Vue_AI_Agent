# Clean Code — PHP & Vue Rules

> Source: Clean Code principles applied to PHP 8.3 and Vue 3

## 📦 Variables

### ✅ Use meaningful, pronounceable names
```php
// ❌ Bad
$yyyymmdstr = now()->format('Y/m/d');

// ✅ Good
$currentDate = now()->format('Y/m/d');
```

### ✅ Same vocabulary for same type
```php
// ❌ getUserInfo(), getClientData(), getCustomerRecord()
// ✅ getUser(), findUser()
```

### ✅ Use searchable names (no magic numbers)
```php
// ❌ cache($key, fn() => ..., 86400);
const SECONDS_PER_DAY = 86400;
Cache::remember($key, SECONDS_PER_DAY, fn() => ...); // ✅
```

### ✅ Avoid mental mapping — be explicit
```php
// ❌ $orders->each(fn($o) => dispatch($o));
// ✅ $orders->each(fn($order) => dispatch($order));
```

### ✅ Don't add redundant context
```php
// ❌ class Order { public $orderStatus; public $orderTotal; }
// ✅ class Order { public $status; public $total; }
```

---

## 🔧 Functions & Methods

### ✅ 2 parameters or fewer — use DTO/array for more
```php
// ❌ function createUser(string $name, string $email, string $password, string $role) {}
// ✅ function createUser(array $data): User {}
// ✅ function createUser(CreateUserDTO $dto): User {}
```

### ✅ Functions should do ONE thing
```php
// ❌ emailUsers() — queries DB + checks active + sends email in one method
// ✅ emailActiveUsers() calls getActiveUsers() + sendEmail() separately
```

### ✅ Method names should say what they do
```php
// ❌ $date->add(1);          → unclear what is added
// ✅ $date->addMonth(1);     → crystal clear
```

### ✅ No flag parameters — split into separate methods
```php
// ❌ function createFile(string $name, bool $isTemp) { if ($isTemp) ... }
// ✅ function createFile(string $name): void {}
// ✅ function createTempFile(string $name): void {}
```

### ✅ Avoid Side Effects — return new data
```php
// ✅ Return new collection instead of mutating global state
public function addItem(Collection $cart, CartItem $item): Collection
{
    return $cart->push($item);
}
```

### ✅ Remove dead code immediately — use git for history

---

## 🏛️ Classes

### ✅ Use method chaining (builder pattern)
```php
// Eloquent Query Builder naturally supports this
$users = User::where('is_active', true)
    ->where('role', 'admin')
    ->orderByDesc('created_at')
    ->paginate(20);
```

### ✅ Prefer composition over inheritance
> "Favor has-a over is-a" — inject dependencies via constructor

---

## 🧱 SOLID Principles

| Principle | Rule |
|-----------|------|
| **S** — Single Responsibility | One class = one job. `UserService` handles business logic, not DB queries |
| **O** — Open/Closed | Add new behavior via new classes, not modifying existing ones |
| **L** — Liskov Substitution | Subclasses must be substitutable for their base class |
| **I** — Interface Segregation | Clients shouldn't depend on interfaces they don't use |
| **D** — Dependency Inversion | Depend on abstractions, not concretions — use Laravel DI container |

```php
// ✅ D — Dependency Inversion (Laravel DI)
class UserController extends Controller
{
    public function __construct(
        private UserService $userService  // injected by Laravel DI container
    ) {}
}

// Register in AppServiceProvider if needed
$this->app->bind(UserRepositoryInterface::class, UserRepository::class);
```

---

## ⚡ PHP Modern Patterns

### ✅ Use named arguments for clarity
```php
// ✅ Clear what each argument does
Cache::remember(key: "user:{$id}", ttl: 3600, callback: fn() => User::find($id));
```

### ✅ Use match expressions over switch
```php
// ✅
$label = match($status) {
    'pending'   => 'Chờ xử lý',
    'completed' => 'Hoàn thành',
    'cancelled' => 'Đã huỷ',
    default     => 'Không xác định',
};
```

### ✅ Use enums for fixed value sets (PHP 8.1+)
```php
enum OrderStatus: string
{
    case Pending   = 'pending';
    case Completed = 'completed';
    case Cancelled = 'cancelled';
}
```

---

## 🗒️ Comments

### ✅ Only comment business logic complexity
### ❌ Never leave commented-out code
### ❌ No journal comments (use git log instead)
```php
// ✅ Good comment — explains WHY
// We retry 3x because the payment gateway has intermittent timeouts
public int $tries = 3;
```

---

## 📐 Formatting (PHP)
- Indentation: **4 spaces** (PSR-12)
- Max line length: **120 characters**
- PascalCase for classes, camelCase for methods/variables, UPPER_SNAKE for constants
- Use Laravel Pint for automatic formatting: `./vendor/bin/pint`
