# Testing Standards — PHPUnit + Pest

## Testing Pyramid
```
         [E2E Tests]         ← Few, slow, catch integration issues (Playwright)
       [Feature Tests]       ← Some, test HTTP endpoints (Pest + Laravel)
     [Unit Tests]            ← Many, fast, test isolated logic (Pest)
```

## Requirements
- Unit test coverage: **minimum 80%** (Services, Repositories)
- All new features must have tests
- All bug fixes must have a regression test
- Tests run in CI before any merge (`php artisan test --coverage`)

## Test File Organization
```
tests/
├── Unit/
│   ├── Services/
│   │   └── UserServiceTest.php
│   └── Repositories/
│       └── UserRepositoryTest.php
├── Feature/
│   ├── Api/
│   │   ├── UsersTest.php
│   │   └── OrdersTest.php
│   └── Auth/
│       └── LoginTest.php
└── e2e/                    # Playwright (JS)
    └── auth-flow.spec.ts
```

## Unit Test Example (Pest)
```php
// tests/Unit/Services/UserServiceTest.php
use App\Services\UserService;
use App\Repositories\UserRepository;
use App\Exceptions\AppException;

describe('UserService', function () {
    beforeEach(function () {
        $this->userRepo = Mockery::mock(UserRepository::class);
        $this->userService = new UserService($this->userRepo);
    });

    describe('findById', function () {
        it('returns user when found', function () {
            $mockUser = User::factory()->make(['id' => 1]);
            $this->userRepo->shouldReceive('findById')->with(1)->andReturn($mockUser);

            $result = $this->userService->findById(1);

            expect($result)->toBeArray()->toHaveKey('id', 1);
        });

        it('throws AppException when user not found', function () {
            $this->userRepo->shouldReceive('findById')->with(999)->andReturn(null);

            expect(fn () => $this->userService->findById(999))
                ->toThrow(AppException::class, 'User not found');
        });
    });
});
```

## Feature Test Example (Pest + Laravel HTTP)
```php
// tests/Feature/Api/UsersTest.php
use App\Models\User;

describe('GET /api/v1/users/{id}', function () {
    it('returns 200 with user data', function () {
        $user = User::factory()->create();

        $this->actingAs($user, 'sanctum')
            ->getJson("/api/v1/users/{$user->id}")
            ->assertOk()
            ->assertJson(['success' => true])
            ->assertJsonPath('data.id', $user->id);
    });

    it('returns 404 for unknown user', function () {
        $user = User::factory()->create();

        $this->actingAs($user, 'sanctum')
            ->getJson('/api/v1/users/99999')
            ->assertNotFound();
    });

    it('returns 401 without auth token', function () {
        $this->getJson('/api/v1/users/1')->assertUnauthorized();
    });
});
```

## Test Commands
```bash
# Run all tests
php artisan test

# Run with coverage report (requires Xdebug or PCOV)
php artisan test --coverage --min=80

# Run specific test file
php artisan test tests/Feature/Api/UsersTest.php

# Run with Pest (parallel execution)
./vendor/bin/pest --parallel

# Run E2E tests (Playwright)
npx playwright test

# Watch mode (Pest)
./vendor/bin/pest --watch
```

## Naming Conventions
- Test classes: `[ClassName]Test.php`
- `describe` blocks: match the class/method being tested
- `it` descriptions: `'returns user when found'`, `'throws exception when email duplicated'`

## Factory Usage
```php
// Create models for tests
$user  = User::factory()->create();                    // saved to DB
$admin = User::factory()->admin()->create();           // with state
$users = User::factory()->count(5)->create();          // multiple
$user  = User::factory()->make();                      // NOT saved to DB
```
