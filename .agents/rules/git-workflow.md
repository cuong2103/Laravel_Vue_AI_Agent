# Git Workflow

## Branch Strategy (Git Flow)

```
main          — Production-ready code only
develop       — Integration branch for features
feature/*     — New features
fix/*         — Bug fixes
hotfix/*      — Urgent production fixes
release/*     — Release preparation
```

## Branch Naming
```
feature/user-authentication
feature/payment-integration
fix/login-redirect-bug
fix/order-calculation-issue
hotfix/critical-security-patch
release/v1.2.0
```

## Commit Message Format (Conventional Commits)

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

### Types
| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change (Pint/Prettier) |
| `refactor` | Code restructure, no feature/fix |
| `test` | Adding or fixing tests |
| `chore` | Build process, dependencies |
| `perf` | Performance improvement |

### Examples
```
feat(auth): add Laravel Sanctum token refresh

fix(orders): correct total price calculation when discount applied

docs(api): add Scribe annotations to user endpoints

test(users): add Pest unit tests for UserService::findById

chore: upgrade Laravel to 12.5.0

style: run pint formatter on app/Services/
```

## Pull Request Rules
- PRs must reference an issue: `Closes #123`
- Minimum 1 reviewer approval required
- All CI checks must pass (`php artisan test --coverage`, `npm run lint:check`)
- No direct commits to `main` or `develop`
- PR title must follow conventional commit format

## Commit Best Practices
- Commit frequently with small, focused changes
- Each commit should be a single logical change
- Never commit: `.env` files, secrets, `vendor/`, `node_modules/`
- Always run tests before committing: `php artisan test`

## Tags & Releases
```bash
# Tag a release
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```
