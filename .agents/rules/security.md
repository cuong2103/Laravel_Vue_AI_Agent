# Security Rules — Laravel

## 🚨 CRITICAL — Never Violate These

- **Never** hardcode secrets, API keys, passwords, or tokens in source code
- **Never** commit `.env` files to version control
- **Never** log sensitive data (passwords, tokens, PII)
- **Never** use `eval()` or dynamic code execution with user input
- **Always** validate and sanitize all user inputs via FormRequest

## Environment Variables
```php
// ✅ Always use environment variables for secrets
$dbPassword = config('database.connections.mysql.password'); // reads from .env
$appKey = config('app.key');

// ❌ Never hardcode secrets
$dbPassword = 'mypassword123';
```

## Input Validation (FormRequest)
```php
// ✅ Validate all incoming data with FormRequest
class LoginRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'email'    => ['required', 'string', 'email', 'max:255'],
            'password' => ['required', 'string', 'min:8', 'max:128'],
        ];
    }
}

// Sanitize HTML to prevent XSS when storing user-generated content
use Illuminate\Support\Str;

$cleanContent = strip_tags($request->input('content'));
// Or use a package like mews/purifier for rich HTML
```

## Authentication — Laravel Sanctum
```php
// Hash passwords — Laravel does this automatically via 'hashed' cast
protected $casts = ['password' => 'hashed'];

// Manual hashing
use Illuminate\Support\Facades\Hash;
$hashed = Hash::make($password); // uses bcrypt with 12 rounds (from BCRYPT_ROUNDS in .env)

// Rate limit auth endpoints
Route::middleware(['throttle:5,1'])->group(function () {
    Route::post('/login', [AuthController::class, 'login']);
    Route::post('/register', [AuthController::class, 'register']);
});
```

## Authorization — Gates & Policies
```php
// ✅ Check permissions on every protected route using Policies
// app/Policies/PostPolicy.php
class PostPolicy
{
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }
}

// In controller
$this->authorize('delete', $post);

// Or in route middleware
Route::delete('/posts/{post}', [PostController::class, 'destroy'])
    ->middleware(['auth:sanctum', 'can:delete,post']);
```

## HTTP Security Headers
```php
// Laravel includes CSRF protection by default for web routes
// For API routes using Sanctum, use stateless tokens

// Add security headers via middleware
// app/Http/Middleware/SecurityHeaders.php
public function handle($request, Closure $next)
{
    $response = $next($request);
    $response->headers->set('X-Content-Type-Options', 'nosniff');
    $response->headers->set('X-Frame-Options', 'DENY');
    $response->headers->set('X-XSS-Protection', '1; mode=block');
    return $response;
}
```

## Rate Limiting
```php
// routes/api.php
Route::middleware(['throttle:api'])->group(function () {
    Route::apiResource('users', UserController::class);
});

// Custom rate limit in RouteServiceProvider
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip());
});
```

## SQL Injection Prevention
- Always use Eloquent or the Query Builder — parameterized automatically
- If raw queries are needed, use `DB::select('SELECT * FROM users WHERE id = ?', [$id])`
- **Never** concatenate user input into SQL strings

## Mass Assignment Protection
```php
// ✅ Always define $fillable or $guarded on models
class User extends Model
{
    // Option 1: allowlist (preferred)
    protected $fillable = ['name', 'email', 'password'];

    // Option 2: blocklist (use with care)
    // protected $guarded = ['id', 'role'];
}
```

## Dependency Security
```bash
# Regularly audit PHP dependencies
composer audit

# Audit JS dependencies
npm audit
npm audit fix
```
