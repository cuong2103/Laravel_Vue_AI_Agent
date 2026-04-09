# AI Agent Configuration — Laravel Base

> **⚠️ SYSTEM BOOTSTRAP — READ THIS FIRST**
>
> Before ANY response, you MUST:
> 1. Read **all** rules in `.claude/rules/`
> 2. Select the **correct agent** from `.claude/agents/`
> 3. Apply the **layered architecture** (Controller → Service → Repository)
>
> ❌ If any step is skipped → your response is **INVALID**.
>
> ✅ Confirm system is ready by stating: **"System loaded — [Agent Name] active."**

---

## Project Overview
This is a **Laravel 12 + Vue 3 + Inertia.js** application. All agents MUST follow the rules and tech stack below.

---

## ⚡ Entry Trigger (Run Before Every Response)

```
STEP 1 — Read Rules:
  Required: tech-stack.md, project-structure.md, laravel.md
  Based on task: database.md, security.md, api-conventions.md,
                 error-handling.md, testing.md, naming-conventions.md

STEP 2 — Select Agent:
  Backend task?   → .claude/agents/backend.md
  Frontend task?  → .claude/agents/frontend.md
  Architecture?   → .claude/agents/systems-architect.md
  Tests?          → .claude/agents/qa.md
  UI/UX?          → .claude/agents/ui-ux-designer.md
  Copy/SEO?       → .claude/agents/copywriter-seo.md
  Planning?       → .claude/agents/project-manager.md

STEP 3 — Apply Architecture:
  Route → Controller (thin) → Service (logic) → Repository (data) → DB
  NEVER put business logic in controllers.
  NEVER write raw SQL.

STEP 4 — Confirm:
  State: "System loaded — [Agent Name] active."
  Then proceed with response.
```

---

## Mandatory Rules
All rules in `.claude/rules/` are **mandatory**:

| File | What it defines |
|------|----------------|
| `laravel.md` | **Laravel-specific patterns** — MVC, thin controllers, Services, Eloquent, FormRequest, API Resources |
| `tech-stack.md` | Approved tech (Laravel 12, Vue 3, MySQL, Redis) |
| `project-structure.md` | Folder layout and layered architecture |
| `database.md` | Eloquent query patterns, transactions, migrations |
| `error-handling.md` | Laravel exception classes and global handler |
| `security.md` | Sanctum auth, Policies, FormRequest, rate limiting |
| `api-conventions.md` | REST conventions, response envelope |
| `testing.md` | PHPUnit + Pest + Playwright standards |
| `naming-conventions.md` | Cache keys, DB names, queues, env vars |
| `clean-code.md` | SOLID, functions, PHP 8.3 patterns |
| `code-style.md` | PSR-12 + Prettier formatting |
| `monitoring.md` | Telescope, Sentry, structured logging |
| `system-design.md` | CAP, caching, scaling patterns |
| `git-workflow.md` | Branching, commit format |

---

## Available Agents

| Agent | File | Invoke When |
|-------|------|-------------|
| � Backend Developer | `agents/backend.md` | APIs, services, Eloquent, jobs |
| �️ Frontend Developer | `agents/frontend.md` | Vue components, Inertia pages, Pinia |
| 📋 Project Manager | `agents/project-manager.md` | User stories, sprint planning |
| 🏗️ Systems Architect | `agents/systems-architect.md` | ADRs, system design, scalability |
| 🎨 UI/UX Designer | `agents/ui-ux-designer.md` | Design system, wireframes |
| ✅ QA Engineer | `agents/qa.md` | Pest/Playwright tests, bug reports |
| ✍️ Copywriter/SEO | `agents/copywriter-seo.md` | Page copy, meta tags, SEO |

---

## Agent Behavior Rules
- Never generate code without first reading the relevant rules
- Always explain changes BEFORE making them
- Prefer incremental changes over large rewrites
- If uncertain about architecture → read `systems-architect.md` first
- Test assumptions before acting
