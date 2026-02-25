# ApplyHarbor — MVP v1 Execution Roadmap

This roadmap defines the execution order for ApplyHarbor MVP v1.

The goal is to:

- Minimize architectural rework
- Implement the hardest and most differentiating parts early
- Deliver vertical slices that are independently testable
- Stay fully aligned with MVP_SPECIFICATION.md
- Maximize portfolio and interview signaling value

The roadmap is structured in **phases**, each composed of **vertical slices**.

---

# Phase 0 — Project Foundation

## Objective

Establish a production-grade technical baseline before implementing business logic.

---

## 0.1 Monorepo + Infrastructure Setup

**Tasks**

- Monorepo structure (`apps/api`, `apps/web`)
- Docker Compose with PostgreSQL
- `.env.example` for API and Web
- ESLint + Prettier
- TypeScript strict mode enabled
- Base README
- Basic CI workflow (lint + test)

**Why first**

Refactoring structure mid-project introduces instability and weakens architectural consistency.  
Strong structure from day one signals maturity and reduces friction later.

---

## 0.2 Backend Skeleton (Fastify Core)

**Tasks**

- Fastify server bootstrap
- `/health` route
- Global error handler (no sensitive leakage)
- Request validation structure
- Logger configuration
- Graceful shutdown handling

**Why here**

Every future API route depends on consistent validation and error behavior.  
Establishing this early prevents inconsistent patterns later.

---

## 0.3 Prisma Initialization

**Tasks**

- Prisma setup
- Initial migration system
- Initial models:
  - `User`
  - `Session` (for refresh tokens)
- DB connection validation

**Why here**

Authentication depends on persistence.  
Auth must be built on a real database model, not mocks.

---

# Phase 1 — Authentication (Core Architectural Differentiator)

Authentication is the strongest architectural signal of the project.  
It should be implemented early and correctly.

---

## 1.1 Registration

**Tasks**

- `POST /auth/register`
- Argon2 password hashing
- Unique email constraint
- Validation
- Proper 409 conflict handling

**Why first**

This is the simplest part of the auth flow and provides a controlled entry point.

---

## 1.2 Login + Token Issuance

**Tasks**

- Short-lived access token
- Long-lived refresh token
- HttpOnly cookies
- Secure flags (dev vs production)
- Session record in DB
- Refresh token hashed in DB

**Why here**

This establishes the cookie strategy and token lifecycle early.  
Everything else depends on this being correct.

---

## 1.3 Refresh Rotation + Reuse Detection

**Tasks**

- `POST /auth/refresh`
- Refresh token rotation
- Invalidate old token
- Detect token reuse
- Session revocation logic

**Why now**

This is the advanced part that differentiates the project from junior-level implementations.  
Implementing it later increases integration risk.

---

## 1.4 Logout

**Tasks**

- Session revocation
- Cookie clearing
- Idempotent behavior

---

## 1.5 GET /me (Canonical Authentication Check)

**Tasks**

- Access token validation
- Return authenticated user info

**Why critical**

This becomes the canonical authentication check used by SSR and client-side logic.

---

# Phase 2 — Nuxt App Shell + SSR Protection

Now authentication is integrated into the frontend.

---

## 2.1 Nuxt Setup

**Tasks**

- Nuxt 3 project
- Tailwind integration
- Route structure:
  - `/`
  - `/login`
  - `/register`
  - `/app/**`

---

## 2.2 Centralized API Client

**Tasks**

- Centralized fetch wrapper
- Single-flight refresh handling
- Retry-once behavior for 401
- Redirect to login on failure

**Why here**

Without this abstraction, authentication logic becomes scattered and fragile.

---

## 2.3 SSR Protection for `/app/**`

**Tasks**

- On SSR:
  - Call `/me`
  - If 401 → attempt `/auth/refresh`
  - If still 401 → redirect to login
- Mirror logic client-side

**Why before feature UI**

Every private page depends on this.  
It must be built once and built correctly.

---

# Phase 3 — Applications Domain (Core Product Logic)

Now real product functionality begins.

---

## 3.1 Application Model

**Tasks**

- Prisma model
- Status enum
- Timestamps

---

## 3.2 Create Application

**Tasks**

- `POST /applications`
- Validation
- Scope by `userId`

---

## 3.3 List Applications

**Tasks**

- `GET /applications`
- Scoped by `userId`
- Pagination

---

## 3.4 Get Single Application

**Tasks**

- `GET /applications/:id`
- Ownership enforcement

---

## 3.5 Update Application

**Tasks**

- `PATCH /applications/:id`
- Strict validation
- Ownership enforcement

---

## 3.6 Delete Application

**Tasks**

- Soft delete preferred
- Ownership enforcement

**Why CRUD before AI**

AI features depend on applications being stored and correctly scoped.

---

# Phase 4 — Profile

**Tasks**

- `GET /profile`
- `PUT /profile`
- Preferences for AI behavior

**Why here**

Low complexity.  
Good confidence builder before introducing AI.

---

# Phase 5 — AI Integration (Controlled and Validated)

AI is probabilistic and unstable.  
It must be integrated only after core stability is achieved.

---

## 5.1 AI Analysis

**Tasks**

- `POST /ai/analyze`
- Request includes `applicationId`
- Load application by `applicationId`
- Ownership validation (application must belong to the authenticated user)
- AI call
- Zod schema validation of AI response
- Overwrite analysis in DB
- Per-user rate limiting

---

## 5.2 Cover Letter Generation

**Tasks**

- `POST /ai/cover-letter`
- Request includes `applicationId`
- Load application by `applicationId`
- Ownership validation
- Structured schema validation
- Store output
- Proper 422 and 502 handling

**Why later**

AI introduces instability and complexity.  
Core product integrity must be stable first.

---

# Phase 6 — Dashboard & Metrics

**Tasks**

- `GET /dashboard`
- Server-side aggregations:
  - Application count
  - Response rate
  - Interview rate

**Why here**

Demonstrates backend computation maturity and domain understanding.

---

# Phase 7 — Landing Page (`/`)

**Tasks**

- SSR marketing page
- Smooth scrolling sections
- Features
- Pricing placeholder

**Why last**

It does not affect system integrity or core architecture.

---

# Phase 8 — Testing & Hardening

## Backend

- Unit tests (services)
- Integration tests (auth flow, ownership)
- Refresh reuse detection tests
- Rate limit tests

## Frontend

- SSR authentication flow test
- Login flow E2E
- Application creation E2E

---

# Phase 9 — Production Readiness

**Tasks**

- Strict CORS origin matching
- Secure cookie flags in production
- Environment validation
- Logging strategy review
- Final architecture documentation in README

---

# Strategic Rationale for This Order

## 1. Hard Problems First

Authentication and SSR gating are the most complex and architecturally significant components.  
Solving them early reduces long-term refactor risk.

---

## 2. Dependency-Safe Layering

The project is built in this dependency chain:

Auth  
→ SSR  
→ Domain  
→ AI  
→ Marketing

No unstable layer is built on top of another unstable layer.

---

## 3. Vertical Slices Over Horizontal Layers

Each phase produces a demonstrable milestone:

- Auth working
- Private app accessible
- CRUD operational
- AI integrated
- Dashboard computed

This maintains momentum and technical clarity.

---

## 4. Interview Positioning

Completing through Phase 3 already signals strong backend and auth competence.  
Completing through Phase 5 signals architectural maturity.  
Everything after that is polish and completeness.