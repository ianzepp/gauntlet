# Gauntlet AI Projects

This is the root directory for all Gauntlet AI program projects.

## About Gauntlet
- 10-week, high-intensity, AI-first program for senior engineers
- Focus: fine-tuning AI development and process skills

## Project Structure
- Each project lives in its own subdirectory under this root
- Projects may be independent or interdependent

## Tooling & Runtime
- **Bun only** — no npm. Use `bun` for all package management, scripts, and runtime.
- **Hono** as the standard backend/API framework
- **htmx** as the default UI approach (server-rendered HTML fragments); reach for React/Svelte only when a project genuinely needs rich client-side interactivity
- **SQLite** for speed of development, but always use DB libraries compatible with both SQLite and PostgreSQL (e.g., Drizzle, Knex, Kysely)

## Hono Project Template

### Project Structure
```
src/
├── index.ts                  # Entry point: env, init, start server
├── server.ts                 # createApp() and startServer() separated for testability
├── routes/                   # File-based routing: path mirrors URL
│   ├── root/GET.ts
│   ├── health/GET.ts
│   ├── auth/
│   │   ├── routes.ts         # Barrel export for route group
│   │   └── login/POST.ts
│   └── api/
│       └── things/
│           ├── routes.ts
│           ├── GET.ts
│           ├── POST.ts
│           └── :id/GET.ts
├── lib/
│   ├── middleware/            # One file per middleware concern
│   │   ├── index.ts          # Barrel export
│   │   ├── auth.ts
│   │   └── error-handler.ts
│   └── errors/
│       └── http-error.ts     # Structured HttpError class
└── spec/                     # Tests (bun:test)
```

### App Initialization
- Separate `createApp()` (returns Hono instance) from `startServer()` (calls `Bun.serve`) for testability
- `Bun.serve({ fetch: app.fetch, port })`
- Register graceful shutdown handlers (`SIGINT`, `SIGTERM`)

### Middleware Pipeline
Apply middleware per route group with `app.use('/path/*', ...)`. Standard order:
1. Body Parser
2. Auth (sets context for downstream)
3. Route Handler
4. Response formatting

Each middleware writes to `context.set(key, value)` — downstream reads it.

### Route Handlers
- Use wrapper functions (e.g., `withTransaction()`) to inject a clean interface: `{ system, params, query, body }`
- Routes return data directly — middleware handles enveloping and formatting
- Barrel-export route groups via `routes.ts` files

### Error Handling
- `HttpError` class with static factories: `HttpErrors.badRequest()`, `.notFound()`, `.unauthorized()`, etc.
- Throw `HttpError` in business logic; caught by `app.onError()` globally
- Consistent response shape: `{ success: boolean, error?: string, error_code?: string, data?: any }`
- Custom `app.notFound()` handler

### Testing
- Use `bun:test` — tests in `spec/` directory
- Serial execution (`--concurrency 1`) for tests that touch the database

## Conventions
- (Additional conventions to be defined as projects are added)
