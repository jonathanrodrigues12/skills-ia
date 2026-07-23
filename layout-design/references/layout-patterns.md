# Layout & Proportion Pattern Reference

## Container & page width

```typescript
// Standard page shell — capped width, centered, responsive side padding
<div className="mx-auto w-full max-w-7xl px-4 md:px-6 lg:px-8">
  {/* page content */}
</div>
```

| Content type | Max width |
|---|---|
| Narrow form / auth screen | `max-w-md` (28rem) |
| Article / reading content | `max-w-2xl` – `max-w-3xl` |
| Standard app page | `max-w-6xl` – `max-w-7xl` |
| Full dashboard / data-dense screen | `max-w-none` inside a padded shell, or no cap |

Pick one width per page type and reuse it — don't let sibling pages in the same section use
different caps.

---

## 12-column grid

```typescript
// Section-level grid, 12 columns, responsive collapse
<div className="grid grid-cols-1 gap-6 md:grid-cols-12">
  <div className="md:col-span-8">{/* main content */}</div>
  <div className="md:col-span-4">{/* sidebar */}</div>
</div>
```

```typescript
// Card grid — let column count respond to breakpoint, not a fixed count everywhere
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  {items.map((item) => <Card key={item.id} {...item} />)}
</div>
```

Common column splits for a 12-col grid: `8/4` (content + sidebar), `4/4/4` (three even
sections), `3/9` (nav + content), `6/6` (even split). Pick the split for a reason (content
weight), not by default.

---

## Typographic scale

Use a consistent ratio-based scale — don't invent sizes between steps:

| Role | Class | Size |
|---|---|---|
| Display / hero | `text-4xl md:text-5xl font-bold tracking-tight` | 36px → 48px |
| Page title (h1) | `text-2xl md:text-3xl font-semibold tracking-tight` | 24px → 30px |
| Section title (h2) | `text-xl font-semibold` | 20px |
| Card/subsection title (h3) | `text-lg font-medium` | 18px |
| Body | `text-sm` (dense UI) or `text-base` (marketing/content) | 14px / 16px |
| Caption / helper text | `text-xs text-muted-foreground` | 12px |

Line height: pair large display/heading sizes with `tracking-tight` (tighter letter-spacing
reads better at scale); leave body text at default tracking. Never mix more than 3 heading
levels visually on one screen.

---

## Spacing rhythm (proportional grouping)

```typescript
<section className="flex flex-col gap-8">          {/* between unrelated sections */}
  <div className="flex flex-col gap-6">              {/* between related groups */}
    <div className="flex flex-col gap-4">             {/* between fields in a group */}
      <div className="flex items-center gap-2">        {/* icon + label, tightest */}
        <Icon className="size-4" />
        <span>Label</span>
      </div>
    </div>
  </div>
</section>
```

Rule: each nesting level of visual grouping steps the gap down one notch
(`gap-8 → gap-6 → gap-4 → gap-2`). If two adjacent groups use the same gap as their own
internal spacing, the hierarchy reads as flat — deliberately widen the outer gap.

---

## Aspect ratios for media

```typescript
import { AspectRatio } from '@/components/ui/aspect-ratio';

<AspectRatio ratio={16 / 9} className="overflow-hidden rounded-md">
  <img src={src} alt={alt} className="h-full w-full object-cover" />
</AspectRatio>

// Avatars / thumbnails — always square via aspect-square, sized by the scale, never by
// eyeballed height
<div className="aspect-square size-10 overflow-hidden rounded-full">
  <img src={avatarUrl} alt="" className="h-full w-full object-cover" />
</div>
```

| Content | Ratio |
|---|---|
| Avatar / thumbnail | `aspect-square` |
| Banner / hero image | `aspect-[21/9]` or `aspect-video` (16:9) |
| Video embed | `aspect-video` |
| Chart container | `aspect-[16/9]` or a fixed `h-[...]` only when the chart library requires
  an explicit pixel height — cap width via the parent, not the chart itself |

---

## Alignment discipline

```typescript
// Card grid — every card's internal elements align to the same baseline regardless of
// content length: header, body, footer pinned via flex, not floating based on content height
<Card className="flex h-full flex-col">
  <CardHeader>{/* title */}</CardHeader>
  <CardContent className="flex-1">{/* body, grows to fill */}</CardContent>
  <CardFooter>{/* actions, pinned to bottom */}</CardFooter>
</Card>
```

```typescript
// Form rows — labels and inputs share the same left edge across all fields; use grid
// instead of independent flex rows when labels vary in width
<div className="grid grid-cols-[120px_1fr] items-center gap-x-4 gap-y-3">
  <label>Nome</label>
  <Input />
  <label>E-mail</label>
  <Input />
</div>
```

Never let one card in a grid be taller/shorter purely due to content length while siblings
stay short — use `h-full` + flex (`flex-1` on the growing region) so footers/actions align
across the row.

---

## Optical vs. mathematical alignment

Icons rarely have the same visual weight as text at the same box height. When an icon+text
pair looks "off" despite `items-center`:

```typescript
// Slightly reduce icon size relative to text, rather than nudging position
<div className="flex items-center gap-2">
  <Icon className="size-4 shrink-0" />       {/* icon slightly smaller than line-height */}
  <span className="text-sm leading-none">Label</span>
</div>
```

Reach for `leading-none` on the text and a slightly undersized icon before reaching for
manual offset utilities (`mt-px`, `relative top-[1px]`) — those are a last resort and should
be commented with why if used.

---

## Symmetry check (before calling a layout done)

- Do the left/right padding of the page shell match, or is one side wider by accident?
- Do sibling cards/sections in a grid share the same internal padding?
- If a two-column split has a divider, is the gap equal on both sides of it?
- Does the last row of a card grid look intentional when the item count doesn't divide evenly
  (e.g. use `col-span` on the last item, or accept the gap — don't stretch it to fill)?
