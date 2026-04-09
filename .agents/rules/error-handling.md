# Error Handling — Laravel

## Core Principles
- **Never swallow errors silently** — always log or rethrow
- Use Laravel's centralized exception handler
- Return consistent error responses from the API
- Distinguish between operational errors (expected) and programmer errors (bugs)

## Custom Exception Class
```php
// app/Exceptions/AppException.php
namespace App\Exceptions;

use Exception;

class AppException extends Exception
{
    public function __construct(
        string $message,
        int $statusCode = 500,
        public readonly string $code = 'INTERNAL_ERROR',
    ) {
        parent::__construct($message, $statusCode);
    }

    public function statusCode(): int
    {
        return $this->getCode();
    }

    public function render(): \Illuminate\Http\JsonResponse
    {
        return response()->json([
            'success' => false,
            'error'   => [
                'code'    => $this->code,
                'message' => $this->getMessage(),
            ],
        ], $this->statusCode());
    }
}
```

## Throwing Errors
```php
// ✅ Use AppException for known operational errors
if (!$user) {
    throw new AppException('User not found', 404, 'USER_NOT_FOUND');
}

if (!$this->authorize('delete', $post)) {
    throw new AppException('Forbidden', 403, 'ACCESS_DENIED');
}
```

## Laravel Global Exception Handler
```php
// app/Exceptions/Handler.php (bootstrap/app.php in Laravel 12)
use App\Exceptions\AppException;
use Illuminate\Foundation\Configuration\Exceptions;

->withExceptions(function (Exceptions $exceptions) {
    // Render AppException as JSON
    $exceptions->render(function (AppException $e, $request) {
        if ($request->expectsJson()) {
            return $e->render();
        }
    });

    // Convert validation exceptions to JSON envelope
    $exceptions->render(function (\Illuminate\Validation\ValidationException $e, $request) {
        if ($request->expectsJson()) {
            return response()->json([
                'success' => false,
                'error'   => [
                    'code'    => 'VALIDATION_ERROR',
                    'message' => 'The given data was invalid.',
                    'details' => $e->errors(),
                ],
            ], 422);
        }
    });
})
```

## Validation Errors (FormRequest)
```php
// Laravel FormRequest automatically returns 422 with errors
// In the controller, just type-hint the FormRequest — no try/catch needed:
public function store(CreateUserRequest $request): JsonResponse
{
    $user = $this->userService->create($request->validated());
    return response()->json(['success' => true, 'data' => $user], 201);
}

// Customize messages in FormRequest:
public function messages(): array
{
    return [
        'email.unique'   => 'This email is already registered.',
        'email.required' => 'Email address is required.',
    ];
}
```

## Logging
```php
// ✅ Use Laravel Log facade with context
use Illuminate\Support\Facades\Log;

Log::error('Payment failed', [
    'user_id'    => $userId,
    'order_id'   => $orderId,
    'error'      => $e->getMessage(),
]);

Log::info('Order created', ['order_id' => $order->id]);

// ✅ Log levels: emergency, alert, critical, error, warning, notice, info, debug
```

## Handling Async Errors (Jobs)
```php
// app/Jobs/SendWelcomeEmail.php
class SendWelcomeEmail implements ShouldQueue
{
    public int $tries = 3;
    public int $backoff = 10; // seconds between retries

    public function failed(\Throwable $exception): void
    {
        Log::error('SendWelcomeEmail job failed', [
            'user_id' => $this->user->id,
            'error'   => $exception->getMessage(),
        ]);

        // Optionally notify Sentry, Slack, etc.
    }
}
```
