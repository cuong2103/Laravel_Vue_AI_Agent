# Database Rules — Eloquent & MySQL

## General Rules
- **Never** write raw SQL strings directly in business logic
- Always use **Eloquent ORM** or the Query Builder
- All database calls must be wrapped in proper error handling
- Use **DB transactions** for multi-step operations

## Eloquent Query Best Practices
```php
// ✅ Select only needed fields
$user = User::select('id', 'email', 'name')->find($id);

// ❌ Avoid selecting all fields when only a few are needed
$user = User::find($id); // loads all columns

// ✅ Use pagination for lists
$users = User::where('is_active', true)
    ->orderByDesc('created_at')
    ->paginate(20);

// ✅ Use eager loading to avoid N+1 queries
$orders = Order::with(['user', 'items.product'])->get();

// ❌ N+1 problem
$orders = Order::all();
foreach ($orders as $order) {
    echo $order->user->name; // queries DB on every iteration!
}

// ✅ Chunk for large datasets instead of loading all at once
User::chunk(200, function (Collection $users) {
    foreach ($users as $user) {
        // process user
    }
});
```

## Transactions
```php
// ✅ Use transactions for atomic operations
use Illuminate\Support\Facades\DB;

DB::transaction(function () use ($orderData, $productId) {
    $order = Order::create($orderData);
    Product::where('id', $productId)->decrement('stock', 1);
    return $order;
});

// With manual control
DB::beginTransaction();
try {
    $order = Order::create($orderData);
    Product::where('id', $productId)->decrement('stock', 1);
    DB::commit();
} catch (\Exception $e) {
    DB::rollBack();
    throw $e;
}
```

## Migrations
- Always use migration files, never modify the database schema directly
- Migration files are version-controlled and immutable
- Run migrations in CI/CD before deploying

```bash
# Create a new migration
php artisan make:migration create_orders_table
php artisan make:migration add_status_to_orders_table

# Run pending migrations
php artisan migrate

# Production (run in CI/CD)
php artisan migrate --force

# Reset and re-run (dev only)
php artisan migrate:fresh --seed
```

## Repository Pattern
```php
// app/Repositories/UserRepository.php
namespace App\Repositories;

use App\Models\User;
use Illuminate\Pagination\LengthAwarePaginator;

class UserRepository
{
    public function findById(int $id): ?User
    {
        return User::find($id);
    }

    public function findByEmail(string $email): ?User
    {
        return User::where('email', $email)->first();
    }

    public function paginate(int $perPage = 20): LengthAwarePaginator
    {
        return User::orderByDesc('created_at')->paginate($perPage);
    }

    public function create(array $data): User
    {
        return User::create($data);
    }

    public function update(User $user, array $data): User
    {
        $user->update($data);
        return $user->fresh();
    }

    public function softDelete(User $user): void
    {
        $user->delete();
    }
}
```

## Naming Conventions
- Tables: **snake_case** plural (`user_profiles`, `order_items`)
- Columns: **snake_case** (`created_at`, `user_id`)
- Indexes: `idx_[table]_[column]`
- Foreign keys: defined in migrations, follow Eloquent conventions (`user_id`)

## Security
- Never log query results containing sensitive data (passwords, tokens)
- Use Eloquent (parameterized queries automatically) — never concatenate user input into SQL
- Use `$fillable` or `$guarded` on all models to prevent mass assignment
- Apply scopes for multi-tenant row-level access
