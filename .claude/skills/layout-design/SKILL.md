---
name: layout-design
description: >
  Enforce visual proportion, grid, and alignment standards when laying out a page, screen,
  dashboard, or component composition in the Next.js frontend. Use this skill whenever the user
  asks to fix/build a layout, align elements, size a grid, balance whitespace, make a screen
  "look proportional" / "less cramped" / "more balanced", set up columns, or decide container
  widths, image aspect ratios, or typographic scale. Complements [ux-design] (which governs
  hierarchy, states, and interaction) — this skill governs the geometry: grid, proportion,
  alignment, and scale.
---

# Layout & Proportion Standards

Getting a screen to *feel* right is mostly geometry: consistent grid, a real typographic scale,
correct aspect ratios, and disciplined alignment — not eyeballed pixel values. Read
`references/layout-patterns.md` for the concrete Tailwind implementation of every rule below.

## How to apply this skill

1. Before placing elements, decide the grid: how many columns, what gutter, what max container
   width. Don't let ad-hoc flex/margins substitute for a grid on any screen with more than one
   column of content.
2. Pick sizes from a scale (spacing, type, radii) — never an arbitrary one-off value chosen by
   eye. If nothing in the scale fits, that's a sign the design needs a step in-between, not an
   excuse to invent one value.
3. Align everything to the grid/baseline: edges of cards, form fields, buttons, and text blocks
   share the same start/end lines. Nothing floats a few pixels off out of "it looked fine".
4. Fixed-ratio media (avatars, thumbnails, banners, charts) uses `aspect-*` utilities — never a
   hardcoded pixel height that breaks proportion on resize.
5. Check the result against `references/layout-patterns.md` before calling a layout done —
   in particular the alignment and whitespace-ratio checks.
6. Layer with `ux-design` for states/interaction and `nextjs-standards` for file structure —
   this skill only governs the geometry of the JSX these produce.

## Critical rules (never skip these)

- Every multi-column page uses an explicit grid (`grid grid-cols-12` or a simpler
  `grid-cols-{n}` for the section) with a defined gutter (`gap-4`/`gap-6`/`gap-8`) — never
  independent floated/margined blocks pretending to align.
- Container width is capped and centered (`mx-auto max-w-{screen-xl|7xl|...}`) with consistent
  side padding (`px-4 md:px-6 lg:px-8`) — content never spans raw viewport width on large
  screens.
- Typography follows a defined scale (see reference) — never a one-off `text-[15px]` or
  `text-[22px]` outside the scale.
- Whitespace grows with hierarchy: tighter gaps within a group (`gap-2`), medium gaps between
  related groups (`gap-4`/`gap-6`), larger gaps between unrelated sections (`gap-8`/`gap-12`).
  Never uniform spacing everywhere — that flattens hierarchy.
- Fixed-ratio content (images, video, chart containers, avatars) always uses `aspect-square`,
  `aspect-video`, or an explicit `aspect-[w/h]` — never `h-[123px]` guessed by eye.
- Optical alignment over mathematical alignment for icons/text: when an icon and text look
  misaligned despite equal box heights, nudge with `items-center` + line-height adjustments
  before reaching for manual `mt-px` hacks — and only as a last resort.
- Keep a single unit of measure per axis: don't mix `rem`-based Tailwind spacing with raw `px`
  margins in the same layout.
- Symmetry is intentional: if one side of a split layout has padding/margin, the equivalent
  side does too, unless there's a clear content reason not to (e.g. a sidebar vs. main content
  with deliberately different rhythm).

## Layer this with other skills

- Interaction states, feedback, accessibility → `ux-design`.
- File/component structure → `nextjs-standards`.
- Feature architecture / data flow → `system-design`.
- Company-wide baseline conventions → `company-standards`.
