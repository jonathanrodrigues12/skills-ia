---
name: system-design
description: >
  Guide architecture decisions when planning a new feature, module, or integration that spans
  backend and frontend. Use this skill whenever the user asks to design, plan, or scaffold a
  feature end-to-end, decide how data should flow between NestJS and Next.js, define an API
  contract, decide where state should live (server vs client), or asks "how should we structure
  this" / "how should this talk to the backend" before writing code. Complements
  [nestjs-standards] and [nextjs-standards] — this skill decides the shape of the system, those
  decide the shape of the code within each layer, and [ux-design] decides how it's presented.
---

# System Design Standards

Before generating code for a non-trivial feature, decide the shape of the system first:
data ownership, API contract, and integration points. Read `references/architecture.md` for
worked patterns. Skipping this step for anything beyond a single component is how features end
up with duplicated state, N+1 requests, or logic split across the wrong layer.

## How to apply this skill

1. **Define the resource/entity** — what does it represent, who owns it (which backend module),
   what's its lifecycle (created/updated/soft-deleted)?
2. **Define the contract before the implementation** — sketch the DTO shape (request/response)
   and endpoint(s) first. This becomes the Swagger spec, which Orval turns into frontend types —
   get this right once instead of hand-syncing types later.
3. **Decide state ownership per piece of data**:
   - Persisted / shared across users → backend + TanStack Query (server state).
   - Ephemeral UI state (open/closed, selected tab, form draft) → local `useState` in the
     component's hook, never TanStack Query or global state.
   - Cross-cutting client state actually needed app-wide (auth session, theme) → existing
     context/providers — don't introduce a new global store for feature-local state.
4. **Map the request lifecycle**: which layer validates (DTO + class-validator), which layer
   authorizes (CASL policies), which layer owns business rules (service), which layer only
   renders (component).
5. **Identify failure modes up front**: what happens on 404, 403, validation error, network
   error — and which UI state (from `ux-design`) each maps to.
6. Only after 1–5 are decided, generate code following `nestjs-standards` /
   `nextjs-standards`.

## Critical rules

- One backend module owns each resource — no other module reaches into its repository
  directly; cross-module access goes through the owning module's service.
- The API contract (DTOs + Swagger) is the single source of truth for shapes shared between
  layers — the frontend never hand-writes a type that mirrors a backend DTO; it consumes the
  Orval-generated one.
- Authorization (CASL policies) is enforced in the backend controller/guard — never trust a
  frontend-only check (hiding a button) as the actual access control.
- Don't add a new state-management library or pattern to solve a problem TanStack Query +
  local state already solves.
- Pagination, filtering, and sorting are server-side for any list that can grow unbounded —
  never load-all-then-filter-in-the-client for data of unknown size.
- Idempotent operations (e.g. "mark as read") tolerate retries; non-idempotent ones
  (e.g. "create payment") need explicit duplicate-submission guards (disable button while
  pending, idempotency key if the operation has real-world side effects).
- When a feature touches more than one backend module, write down the call direction
  (which module calls which) before coding — avoid circular module dependencies.

## Layer this with other skills

- Backend implementation details → `nestjs-standards`.
- Frontend implementation details → `nextjs-standards`.
- Visual/interaction design of the resulting screens → `ux-design`.
- Company-wide baseline conventions → `company-standards`.
