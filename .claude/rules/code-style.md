# Code Style Guide — PHP & Vue 3

## General Principles
- **Clarity over cleverness** — Write code that is easy to read and understand
- **Consistency** — Follow existing patterns in the codebase
- **DRY** — Don't Repeat Yourself, but don't over-abstract

---

## PHP (PSR-12 + Laravel Pint)

### Formatting
- Indentation: **4 spaces** (no tabs)
- Max line length: **120 characters**
- Always use **strict_types=1** at the top of files
- Use **single quotes** for plain strings, double quotes only for interpolation

```php
<?php

declare(strict_types=1);

namespace App\Services;
```

### Naming
```php
// Classes, Interfaces, Enums: PascalCase
class UserService {}
interface UserRepositoryInterface {}
enum OrderStatus: string {}

// Methods and variables: camelCase
public function findById(int $id): ?User {}
$currentUser = auth()->user();

// Constants: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';

// Files (PHP): PascalCase (always matches class name)
// UserService.php, CreateUserRequest.php

// Files (Vue): PascalCase
// UserCard.vue, AppSidebar.vue
```

### Methods
```php
// ✅ Good — single responsibility, typed
public function createUser(array $data): User
{
    return User::create($data);
}

// ✅ Use readonly properties for immutable data (PHP 8.1+)
class UserDTO
{
    public function __construct(
        public readonly string $name,
        public readonly string $email,
    ) {}
}

// ❌ Avoid — methods longer than 30 lines (extract helpers)
```

### Imports
```php
// Order: 1. PHP built-in classes, 2. External vendor, 3. App classes
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Cache;
use App\Models\User;
use App\Services\UserService;
```

---

## TypeScript / Vue 3

### Formatting
- Indentation: **4 spaces** in Vue templates, **2 spaces** in TS files
- Max line length: **100 characters**
- Use **single quotes** for strings
- Trailing commas in multi-line structures

### Vue SFC Structure Order
```vue
<script setup lang="ts">
// 1. Imports
// 2. Props/Emits
// 3. Reactive state (ref, reactive, computed)
// 4. Composables (useXxx)
// 5. Methods
// 6. Lifecycle hooks
</script>

<template>
  <!-- markup here -->
</template>
```

### Naming
```ts
// Variables and composables: camelCase
const userProfile = ref({});
const { user } = useAuth();

// Interfaces and Types: PascalCase
interface UserProfile { id: number; name: string; }
type OrderStatus = 'pending' | 'completed' | 'cancelled';

// Constants: UPPER_SNAKE_CASE
const MAX_ITEMS_PER_PAGE = 20;

// Vue components: PascalCase
// UserCard.vue, OrderSummary.vue
```

### Comments
```php
// ✅ Explain WHY, not WHAT
// We cache this for 1 hour because the catalog rarely changes
$products = Cache::remember('products:all', 3600, fn() => Product::all());

// ❌ Avoid obvious comments
// Get user
$user = User::find($id);
```

---

## Tooling
```bash
# PHP — auto-format with Laravel Pint (PSR-12 + Laravel style)
./vendor/bin/pint

# Check only (CI)
./vendor/bin/pint --test

# TypeScript/Vue — Prettier
npm run format

# Lint Vue/TS
npm run lint

# Type check
npm run types:check
```

## File Organization
- One class/service per file (PHP)
- One component per file (Vue)
- Group related files in feature folders
- Use index files for clean imports
