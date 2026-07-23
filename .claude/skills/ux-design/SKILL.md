---
name: ux-design
description: >
  Enforce UX/UI design standards when building or reviewing frontend layout, screens, and
  components. Use this skill whenever the user asks to create a page, screen, layout, dashboard,
  form flow, empty state, loading state, or any visual/interaction design decision in the Next.js
  frontend. Also trigger when the user asks to "improve the UX", review a layout, fix spacing/
  hierarchy/responsiveness, or make a screen "look better" or "more professional". Works directly
  on top of the existing frontend stack (shadcn/ui + Tailwind) — apply together with
  [nextjs-standards] whenever generating actual component code.
---

# UX / UI Design Standards

You are responsible for how screens *look and feel*, not just whether they compile. Layout,
hierarchy, spacing, and feedback states are as much a part of "correct" frontend code as the
logic is. When in doubt, prefer the patterns in `references/ux-patterns.md` over ad-hoc styling.

## How to apply this skill

1. Before laying out a screen, identify: primary action, secondary actions, and what the user
   needs to see first. Design hierarchy before markup.
2. Reuse shadcn/ui primitives and existing layout wrappers (`@/app/_layouts/root`) — never
   hand-roll a component shadcn already provides (dialogs, dropdowns, tables, tooltips…).
3. Every async view needs three explicit states: loading, empty, and error — never just the
   happy path. Loading uses skeletons over spinners where content shape is known.
4. Every destructive action (delete, revoke, cancel) needs a confirmation step
   (`AlertDialog`) before firing the mutation.
5. Check the result against the checklist in `references/ux-patterns.md` before calling a
   layout done.
6. When generating actual component files, follow `nextjs-standards` for file split
   (`index.tsx` / `useComponentName.tsx`) — this skill governs what goes inside the JSX and
   why, not the file structure.

## Critical rules (never skip these)

- Never introduce a new color, spacing value, or font size outside the Tailwind theme scale —
  no arbitrary values like `mt-[13px]` unless matching a fixed external constraint (e.g. a
  pixel-perfect asset).
- Maintain one clear primary action per screen/section — visually distinct (`Button` default
  variant); everything else is `outline`/`ghost`/`secondary` or a text link.
- Respect an 8px spacing rhythm (Tailwind's default scale is already built on 4px/8px — stick
  to it: `gap-2`, `gap-4`, `gap-6`, `gap-8`, not arbitrary values).
- Never rely on color alone to convey meaning (errors, status, required fields) — pair color
  with an icon, label, or text.
- All interactive elements must have a visible focus state and a minimum 44×44px touch target
  on mobile.
- Every form field with an error shows the error inline via `FormMessage`, not only via toast.
- Design and build mobile-first: base classes target mobile, breakpoints (`sm:`, `md:`,
  `lg:`) layer on enhancements — never the reverse.
- Never ship a table/list without an empty state and, when the dataset can grow, pagination or
  virtualization.
- Motion is purposeful and short (150–250ms, `ease-out`) — never decorative animation that
  delays the user from acting.

## Layer this with other skills

- Structural/file conventions → `nextjs-standards`.
- General company-wide conventions → `company-standards`.
- Cross-cutting feature architecture (API shape, state ownership) → `system-design`.
