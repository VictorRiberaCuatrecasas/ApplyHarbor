# ApplyHarbor

ApplyHarbor is an AI-powered job application tracker built to demonstrate **production-grade authentication architecture, SSR protection, and secure AI integration**.

This project prioritizes architectural correctness, security discipline, and interview-level explainability over feature breadth.

---

## Overview

ApplyHarbor is intentionally designed as a portfolio-grade system that showcases:

- HttpOnly cookie-based authentication
- Refresh token rotation with reuse detection
- SSR-protected private routes
- Strict per-user data ownership enforcement
- Defensive AI integration with schema validation
- Clear API contracts and vertical-slice execution

The goal is not feature volume — it is architectural maturity.

---

## Tech Stack

### Frontend

- Nuxt 3 (SSR enabled)
- TypeScript (strict mode)
- Tailwind CSS

### Backend

- Fastify
- TypeScript
- Prisma
- PostgreSQL

---

## Authentication Model

- Access + refresh tokens stored in **HttpOnly cookies**
- Short-lived access tokens
- Long-lived refresh tokens
- Refresh token rotation
- Reuse detection
- Refresh tokens hashed in the database
- Session revocation on logout
- No localStorage/sessionStorage usage for authentication

---

## AI Integration

- REST endpoints:
  - `POST /ai/analyze`
  - `POST /ai/cover-letter`
- Zod schema validation of AI responses
- AI treated as untrusted input
- Overwrite-on-save model (no chat interface)
- Per-user rate limiting
- Structured 422 / 502 error handling

---

# Architecture Highlights

## 1. Refresh Token Rotation with Reuse Detection

Each refresh operation:

- Invalidates the previous refresh token
- Issues a new refresh token
- Detects reuse attempts
- Revokes compromised sessions

This prevents replay attacks and demonstrates real-world authentication architecture beyond simple JWT handling.

---

## 2. SSR Authentication Gating

Private routes (`/app/**`) are protected at both:

- **Server-side (SSR)**
- **Client-side navigation**

Flow:

1. SSR calls `GET /me`
2. If `401` → attempt `/auth/refresh`
3. If still `401` → redirect to `/login`
4. Client mirrors the same logic for navigation

This ensures private content is never rendered without authentication.

---

## 3. Strict Ownership Enforcement

All database queries are scoped by `userId`.

- No resource is ever fetched without ownership validation
- Route parameters are validated
- Authorization checks occur before business logic

This prevents cross-user data access vulnerabilities.

---

## 4. Defensive AI Integration

AI responses are:

- Schema-validated using Zod
- Stored in structured form
- Overwritten rather than appended
- Rate-limited per user

AI is treated as probabilistic and untrusted input.

---

# Routing Model

- `/` → Public SSR landing page
- `/login`, `/register` → Public authentication routes
- `/app/**` → Private application routes (SSR + client protected)

---

# MVP Scope

## Authentication

- Register
- Login
- Refresh rotation
- Reuse detection
- Logout
- Canonical `GET /me`

## Applications

- Create
- List (paginated)
- Get by ID
- Update
- Soft delete
- Strict ownership enforcement

## Profile

- `GET /profile`
- `PUT /profile`

## AI

- Application analysis
- Cover letter generation
- Schema-validated responses
- Per-user rate limiting

## Dashboard

- Server-side aggregations:
  - Application count
  - Response rate
  - Interview rate

## Landing Page

- SSR marketing page at `/`

---

# Security Guarantees

- Tokens never accessible to JavaScript
- No localStorage/sessionStorage for auth
- Strict CORS origin matching
- SameSite cookie strategy
- Refresh token hashing in DB
- Input validation on every route
- No sensitive error leakage
- In-memory rate limiting (MVP v1)

---

# Testing Strategy

- **Unit tests** for pure logic
- **Integration tests** for:
  - Auth flows
  - Refresh rotation & reuse detection
  - Ownership enforcement
- **Minimal E2E tests** for:
  - Login flow
  - SSR private route gating

Critical authentication paths are always tested.

---

# Source of Truth

Contract precedence:

1. `MVP_SPECIFICATION.md` — defines API and system behavior
2. `project-roadmap.md` — defines implementation order
3. `scaffold.md` — structural reference

If roadmap conflicts with spec, the specification defines the contract.

---

# Local Development

## Requirements

- Node 18+
- Docker
- PostgreSQL (via Docker Compose)

## Setup

```bash
cp .env.example .env
docker compose up -d
pnpm install
pnpm dev
```

---

## Runtime

Backend runs on:  
`http://localhost:3001`

Frontend runs on:  
`http://localhost:3000`

---

## Roadmap

The execution roadmap is documented in:

`docs/project-roadmap.md`

Build order:

Auth → SSR → Domain → AI → Dashboard → Landing → Hardening

Hard problems are solved first to reduce long-term architectural risk.

---

## License

MIT License © 2026 Víctor Ribera
