# ApplyHarbor — Project Structure (MVP v1)

## Legend

🔵 = Backend (Fastify + Prisma + PostgreSQL)  
🟢 = Frontend (Nuxt 3 + Tailwind)  
⚪ = Shared / Root / Infra

---

```
applyharbor/
├─ ⚪ README.md
├─ ⚪ .gitignore
├─ ⚪ .editorconfig
├─ ⚪ .nvmrc
├─ ⚪ .env.example
├─ ⚪ docker-compose.yml
├─ ⚪ docs/
│  ├─ MVP_SPECIFICATION.md
│  ├─ project_roadmap.md
│  └─ diagrams/
├─ ⚪ apps/
│  ├─ 🔵 api/
│  │  ├─ package.json
│  │  ├─ tsconfig.json
│  │  ├─ .env.example
│  │  ├─ prisma/
│  │  │  ├─ schema.prisma
│  │  │  └─ migrations/
│  │  ├─ src/
│  │  │  ├─ app.ts                     # build Fastify instance (register plugins/routes)
│  │  │  ├─ server.ts                  # start server
│  │  │  ├─ config/
│  │  │  │  └─ env.ts                  # typed env validation
│  │  │  ├─ plugins/
│  │  │  │  ├─ cors.ts                 # exact origin + credentials true
│  │  │  │  ├─ prisma.ts               # Prisma client decorator
│  │  │  │  ├─ auth.ts                 # auth utilities + request context user
│  │  │  │  └─ rateLimit.ts            # in-memory fixed window + daily quotas
│  │  │  ├─ lib/
│  │  │  │  ├─ errors.ts               # API error contract { error, message }
│  │  │  │  ├─ cookies.ts              # cookie options (dev vs prod cross-site)
│  │  │  │  ├─ crypto.ts               # hashing helpers (password + refresh token)
│  │  │  │  └─ time.ts                 # expiry helpers
│  │  │  ├─ modules/
│  │  │  │  ├─ auth/
│  │  │  │  │  ├─ routes.ts            # /auth/register|login|refresh|logout
│  │  │  │  │  ├─ service.ts           # rotation + reuse detection
│  │  │  │  │  ├─ schemas.ts           # Fastify JSON schemas
│  │  │  │  │  └─ tokens.ts            # access jwt + opaque refresh token structure
│  │  │  │  ├─ me/
│  │  │  │  │  ├─ routes.ts            # GET /me
│  │  │  │  │  └─ schemas.ts
│  │  │  │  ├─ applications/
│  │  │  │  │  ├─ routes.ts            # CRUD /applications
│  │  │  │  │  ├─ service.ts           # ownership enforcement (userId scoping)
│  │  │  │  │  └─ schemas.ts
│  │  │  │  ├─ profile/
│  │  │  │  │  ├─ routes.ts            # GET/PUT /profile
│  │  │  │  │  ├─ service.ts
│  │  │  │  │  └─ schemas.ts
│  │  │  │  └─ ai/
│  │  │  │     ├─ routes.ts            # POST /ai/analyze, /ai/cover-letter
│  │  │  │     ├─ service.ts           # retry once on upstream failure, overwrite on success
│  │  │  │     ├─ provider.ts          # LLM adapter (timeouts, metadata-only logs)
│  │  │  │     └─ validation.ts        # Zod schemas + constraint checks
│  │  │  └─ routes.ts                  # register module routes
│  │  └─ test/
│  │     ├─ setup.ts
│  │     └─ auth.integration.test.ts   # refresh rotation + reuse detection
│  │
│  └─ 🟢 web/
│     ├─ package.json
│     ├─ nuxt.config.ts
│     ├─ tsconfig.json
│     ├─ .env.example
│     ├─ app.vue
│     ├─ pages/
│     │  ├─ index.vue                  # SSR landing page (single scrollable)
│     │  ├─ login.vue
│     │  ├─ register.vue
│     │  └─ app/
│     │     ├─ index.vue
│     │     ├─ applications/
│     │     │  ├─ index.vue
│     │     │  └─ [id].vue
│     │     └─ settings.vue
│     ├─ server/
│     │  └─ middleware/
│     │     └─ app-auth.ts             # SSR protection for /app/** (primary gate)
│     ├─ features/
│     │  ├─ auth/
│     │  │  ├─ api.ts                  # /me + /auth/* clients (credentials: include)
│     │  │  ├─ session.ts              # current user state
│     │  │  └─ refreshSingleFlight.ts  # prevents parallel refresh storms
│     │  ├─ applications/
│     │  │  ├─ api.ts
│     │  │  ├─ types.ts
│     │  │  └─ composables.ts
│     │  ├─ profile/
│     │  │  ├─ api.ts
│     │  │  ├─ types.ts
│     │  │  └─ composables.ts
│     │  └─ ai/
│     │     ├─ api.ts
│     │     ├─ types.ts
│     │     └─ ui.ts                   # empty states + result rendering helpers
│     ├─ components/
│     │  └─ ui/                        # dumb UI primitives only
│     └─ lib/
│        ├─ apiFetch.ts                # enforces credentials + error mapping + retry policy
│        ├─ errors.ts                  # maps API error codes to UX
│        └─ dates.ts
│
└─ ⚪ pnpm-workspace.yaml
```

## Notes (important)

- No `packages/shared-types/` until it’s clearly needed.
- No Nuxt global route middleware for auth yet (avoid loops and double-refresh).
- SSR auth gate lives in `web/server/middleware/app-auth.ts`.
