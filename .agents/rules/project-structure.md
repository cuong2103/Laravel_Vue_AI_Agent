# Project Structure

## Standard Folder Layout (Laravel 12 + Vue 3 + Inertia.js)

```
project-root/
├── .claude/                    # Claude AI Agent configuration
│   ├── agents/                 # Sub-agent definitions
│   ├── rules/                  # Mandatory rules for AI
│   ├── settings.json           # Project-level settings
│   └── CLAUDE.md               # Main AI instructions
│
├── .cursor/                    # Cursor IDE rules
│   └── rules/                  # .mdc rule files
│
├── .agents/                    # Antigravity / Gemini
│   ├── agents/
│   └── rules/
│
├── app/                        # Laravel application code
│   ├── Console/                # Artisan commands
│   ├── Exceptions/             # Custom exception classes
│   │   └── AppException.php
│   ├── Http/
│   │   ├── Controllers/        # Route handlers (thin layer)
│   │   ├── Middleware/         # HTTP middleware
│   │   └── Requests/           # FormRequest validation classes
│   ├── Jobs/                   # Queue jobs
│   ├── Mail/                   # Mailable classes
│   ├── Models/                 # Eloquent models
│   ├── Providers/              # Service providers
│   ├── Repositories/           # Data access layer
│   └── Services/               # Business logic layer
│
├── bootstrap/                  # App bootstrap files
├── config/                     # Configuration files
├── database/
│   ├── factories/              # Model factories (for testing)
│   ├── migrations/             # Database schema migrations
│   └── seeders/                # Database seeders
│
├── docs/                       # Documentation
│   ├── api/                    # API documentation
│   └── architecture/           # Architecture diagrams + ADRs
│
├── public/                     # Web root (index.php, assets)
├── resources/
│   ├── css/
│   │   └── app.css             # Tailwind CSS entry point
│   ├── js/
│   │   ├── app.ts              # Inertia/Vue entry point
│   │   ├── components/         # Shared Vue components
│   │   │   └── ui/             # Base UI components (Reka UI)
│   │   ├── composables/        # Vue composables (useForm, useAuth...)
│   │   ├── layouts/            # Inertia page layouts
│   │   ├── pages/              # Inertia page components
│   │   │   ├── Dashboard.vue
│   │   │   ├── auth/
│   │   │   └── settings/
│   │   ├── stores/             # Pinia stores
│   │   └── types/              # TypeScript type definitions
│   └── views/
│       └── app.blade.php       # Single Blade template (Inertia root)
│
├── routes/
│   ├── web.php                 # Web routes (Inertia)
│   ├── api.php                 # API routes (JSON/Sanctum)
│   └── console.php             # Artisan schedule
│
├── storage/                    # Logs, cache, uploaded files
├── tests/
│   ├── Feature/                # HTTP / integration tests (Pest)
│   │   ├── Api/
│   │   └── Auth/
│   └── Unit/                   # Unit tests (Pest)
│       └── Services/
│
├── .env                        # Local environment (gitignored)
├── .env.example                # Template committed to git
├── artisan                     # Laravel CLI
├── composer.json
├── package.json
├── vite.config.ts
└── README.md
```

## Layered Architecture
```
Request → Routes → Middleware → Controller → Service → Repository → Database
                                       ↓
                                  FormRequest (validation)
```

- **Routes** (`routes/web.php`, `routes/api.php`): URL mapping only, no logic
- **Controllers** (`app/Http/Controllers/`): thin — inject Service, call method, return response
- **FormRequests** (`app/Http/Requests/`): input validation, authorization
- **Services** (`app/Services/`): business logic, orchestration, no HTTP awareness
- **Repositories** (`app/Repositories/`): data access, Eloquent queries only
- **Models** (`app/Models/`): Eloquent relationships, casts, scopes

## PHP File Naming
- Classes: `PascalCase.php` (`UserService.php`, `OrderRepository.php`)
- Test files: `[Name]Test.php` or `[name].test.ts` for Vue
- Controllers: `[Resource]Controller.php` (`UserController.php`)
- FormRequests: `[Verb][Resource]Request.php` (`CreateUserRequest.php`)
- Jobs: `[Action][Resource].php` (`SendWelcomeEmail.php`)

## Vue/TypeScript File Naming
- Components: `PascalCase.vue` (`UserCard.vue`, `OrderSummary.vue`)
- Pages: `PascalCase.vue` in `resources/js/pages/`
- Composables: `use[Name].ts` (`useAuth.ts`, `useCart.ts`)
- Stores: `use[Name]Store.ts` (`useCartStore.ts`)
- Types: `[name].d.ts` or `[name].ts` in `types/`

## Environment Files
- `.env` — Local development (gitignored)
- `.env.example` — Template committed to git
- `.env.testing` — Test environment (gitignored)
- Set production values in CI/CD pipeline secrets, never commit
