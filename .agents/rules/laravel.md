# Laravel Rules — Mandatory Patterns

> This rule is **always required** for any backend task. Read before writing any PHP code.

## 🏗️ MVC Pattern — Laravel Specific

Laravel follows MVC. Your layered architecture is:
```
Route → Controller (thin) → Service (business logic) → Repository (data) → Database
                          ↓
                    FormRequest (validation)
                    API Resource (response transform)
```

---

## ✅ Controllers MUST Be Thin

A controller only:
1. Receives the request
2. Delegates to a Service
3. Returns a response

```php
// ✅ Correct — thin controller
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());
        return response()->json(['success' => true, 'data' => $user], 201);
    }
}

// ❌ Wrong — controller doing business logic
class UserController extends Controller
{
    public function store(Request $request): JsonResponse
    {
        // ❌ Validation directly in controller
        $request->validate([...]);

        // ❌ Business logic in controller
        if (User::where('email', $request->email)->exists()) {
            return response()->json(['error' => 'Email taken'], 409);
        }

        // ❌ Direct Eloquent in controller
        $user = User::create([...$request->all(), 'password' => Hash::make($request->password)]);
        return response()->json(['data' => $user]);
    }
}
```

---

## ✅ Business Logic MUST Be in Services

```php
// app/Services/UserService.php
namespace App\Services;

class UserService
{
    public function __construct(private UserRepository $userRepository) {}

    public function create(array $data): array
    {
        // ✅ Business rules here
        if ($this->userRepository->findByEmail($data['email'])) {
            throw new AppException('Email already in use', 409, 'EMAIL_CONFLICT');
        }
        $data['password'] = Hash::make($data['password']);
        return $this->userRepository->create($data)->toArray();
    }
}
```

---

## ✅ Always Use Eloquent ORM

```php
// ✅ Eloquent
$users = User::where('is_active', true)->orderByDesc('created_at')->paginate(20);
$order = Order::with(['user', 'items.product'])->findOrFail($id);
User::create($data);         // mass assignment (requires $fillable)
$user->update(['name' => $name]);
$user->delete();             // soft delete if SoftDeletes trait is used

// ❌ Never raw SQL string concatenation
DB::statement("SELECT * FROM users WHERE email = '$email'"); // SQL injection risk!

// ✅ If raw SQL is truly needed — use bindings
DB::select('SELECT * FROM users WHERE email = ?', [$email]);
```

---

## ✅ Always Use FormRequest for Validation

```php
// app/Http/Requests/CreateUserRequest.php
class CreateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // or: auth()->check()
    }

    public function rules(): array
    {
        return [
            'email'    => ['required', 'email', 'max:255', 'unique:users,email'],
            'name'     => ['required', 'string', 'min:2', 'max:100'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.unique' => 'This email is already registered.',
        ];
    }
}

// ❌ Never validate manually in controller:
// $request->validate([...]); // use FormRequest class instead
```

---

## ✅ Use API Resources for Response Transformation

```php
// app/Http/Resources/UserResource.php
class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'email'      => $this->email,
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}

// Usage in controller
return new UserResource($user);
return UserResource::collection($users);

// With pagination
return UserResource::collection($users)->response()->setStatusCode(200);
```

---

## ✅ Model Requirements

Every Eloquent model MUST have:
```php
class User extends Model
{
    use HasFactory;          // for tests
    use SoftDeletes;         // if soft-deletable

    protected $fillable = ['name', 'email', 'password'];  // mass assignment protection

    protected $hidden = ['password', 'remember_token'];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password'          => 'hashed',      // auto-hash on set (Laravel 10+)
    ];
}
```

---

## ✅ Route Conventions

```php
// routes/api.php

// Resource routes (preferred)
Route::apiResource('users', UserController::class);
Route::apiResource('users.orders', UserOrderController::class)->shallow();

// Protected routes
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('orders', OrderController::class);
    Route::post('payments/{payment}/refund', [PaymentController::class, 'refund']);
});

// Rate limiting
Route::middleware(['auth:sanctum', 'throttle:60,1'])->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

---

## ✅ Quick Checklist

Before submitting any backend code:
- [ ] Controller is thin (no business logic, no direct Eloquent)
- [ ] Business logic is in a Service class
- [ ] Validation is in a FormRequest class
- [ ] Response uses API Resource transformer
- [ ] Eloquent model has `$fillable`
- [ ] Route is protected with `auth:sanctum` if needed
- [ ] Errors throw `AppException` with proper status code
- [ ] Sensitive routes have `throttle` middleware
- [ ] Feature test written (Pest)
