---
name: nextjs-standards
description: >
  Enforce Next.js frontend development standards when writing, reviewing, or generating code.
  Use this skill whenever the user asks to create a component, hook, page, form, test, or any
  piece of frontend code in a Next.js App Router project with shadcn/ui and TanStack Query.
  Also trigger when scaffolding a feature, refactoring existing frontend code, or setting up
  forms, authentication, state management, or E2E tests.
  Apply this skill even when the user doesn't say "follow standards" — consistency across the
  frontend codebase is the goal.
---

# Next.js Frontend Standards

## Stack

| Area | Standard |
|---|---|
| Framework | Next.js 16+ (App Router) / TypeScript 5+ |
| UI | shadcn/ui + Radix UI |
| Styling | Tailwind CSS 4+ |
| Forms | React Hook Form + Zod |
| State | TanStack Query (React Query) |
| HTTP | Axios |
| Auth | NextAuth.js |
| Unit Tests | Vitest + React Testing Library |
| E2E Tests | Cypress |
| API Code Gen | Orval (from Swagger → `src/gen/`) |

Read the full patterns in `references/nextjs.md` before producing any code.

## How to apply this skill

1. Read the relevant section in `references/nextjs.md` for the artifact requested (component, hook, page, form, test).
2. When creating a new feature component, always scaffold both files: `index.tsx` (visual) and `useComponentName.tsx` (logic).
3. Generate code that follows naming conventions and folder structure exactly.
4. Do not add comments explaining what the code does — only comment when the WHY is non-obvious.

## Critical rules

- Components **must** separate visual (`index.tsx`) from logic (`useComponentName.tsx`)
- `index.tsx` always has `'use client'` at the top
- Forms always use React Hook Form + Zod with `zodResolver`
- Frontend errors surface via `sonner` toast: `toast.loading` → `toast.success` / `toast.error` with the same ID
- Use `cn()` from `@/lib/utils` for Tailwind class merging
- Always use shadcn/ui components first — never raw HTML for UI elements
- Never use `any` — type everything
- Use `data-testid` attributes on all interactive elements for stable E2E selectors
- **Always use generated code from `src/gen/`** — never hand-write API types, fetch functions, or Zod schemas that mirror backend DTOs; run `pnpm orval` to regenerate when the backend Swagger changes

## File naming

| Pattern | Convention |
|---|---|
| Files & folders | `kebab-case` |
| React components (export) | `PascalCase` |
| Custom hooks | `use[NomePascalCase].tsx` |
| Classes & interfaces | `PascalCase` |
| Variables & functions | `camelCase` |
| Constants | `SCREAMING_SNAKE_CASE` |

## Folder structure

```
components/(feature)/component-name/
├── index.tsx          # Visual only — no business logic
└── useComponentName.tsx  # All logic, state, queries

app/(route-group)/route/
├── layout.tsx
└── page.tsx
```
