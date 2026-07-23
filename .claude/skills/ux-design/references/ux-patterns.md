# UX / UI Pattern Reference

## Visual hierarchy checklist

Before writing JSX, answer:
- What is the ONE primary action on this screen?
- What does the user look at first, second, third?
- What can be collapsed, deferred, or moved to a secondary view (dialog, tab, accordion)?

```typescript
// Primary vs secondary actions
<div className="flex items-center gap-2">
  <Button data-testid="save-button">Salvar</Button>
  <Button variant="outline" data-testid="cancel-button">Cancelar</Button>
</div>
```

---

## Page skeleton (list/dashboard screen)

```typescript
// app/(group)/route/page.tsx
'use client';
import Layout from '@/app/_layouts/root';
import FeatureList from '@/components/(feature)/feature-list';

export default function FeaturePage() {
  return (
    <Layout breadCrumbItems={[{ title: 'Feature', url: '/feature' }]}>
      <div className="flex flex-col gap-6">
        <div className="flex items-center justify-between">
          <div>
            <h1 className="text-2xl font-semibold tracking-tight">Feature</h1>
            <p className="text-sm text-muted-foreground">
              Descrição curta do que essa tela faz.
            </p>
          </div>
          <Button data-testid="add-button">Adicionar</Button>
        </div>
        <FeatureList />
      </div>
    </Layout>
  );
}
```

---

## Loading / empty / error states

Every data-driven component handles all three — never only the happy path:

```typescript
export function FeatureList() {
  const { data, isLoading, isError } = useFeatureList();

  if (isLoading) {
    return (
      <div className="flex flex-col gap-2">
        <Skeleton className="h-10 w-full" />
        <Skeleton className="h-10 w-full" />
        <Skeleton className="h-10 w-full" />
      </div>
    );
  }

  if (isError) {
    return (
      <div className="flex flex-col items-center gap-2 py-12 text-center">
        <p className="text-sm text-muted-foreground">
          Não foi possível carregar os dados.
        </p>
        <Button variant="outline" size="sm" data-testid="retry-button">
          Tentar novamente
        </Button>
      </div>
    );
  }

  if (!data?.length) {
    return (
      <div className="flex flex-col items-center gap-2 py-12 text-center">
        <p className="text-sm font-medium">Nenhum item encontrado</p>
        <p className="text-sm text-muted-foreground">
          Cadastre o primeiro item para começar.
        </p>
        <Button size="sm" data-testid="empty-add-button">Adicionar</Button>
      </div>
    );
  }

  return (
    <div className="rounded-md border">
      {/* table/list content */}
    </div>
  );
}
```

Use `Skeleton` (shape matches final content), never a bare spinner, whenever the layout of the
loaded content is known ahead of time. Reserve spinners for actions with unknown/short duration
(button submit state).

---

## Destructive action confirmation

```typescript
<AlertDialog>
  <AlertDialogTrigger asChild>
    <Button variant="destructive" size="sm" data-testid="delete-button">
      Excluir
    </Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Excluir item?</AlertDialogTitle>
      <AlertDialogDescription>
        Essa ação não pode ser desfeita. O item será removido permanentemente.
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel data-testid="delete-cancel">Cancelar</AlertDialogCancel>
      <AlertDialogAction
        onClick={onConfirmDelete}
        data-testid="delete-confirm"
        className="bg-destructive text-destructive-foreground"
      >
        Excluir
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

---

## Spacing & sizing scale

Stick to Tailwind's default scale — do not invent arbitrary values:

| Use | Class |
|---|---|
| Tight group (icon + label) | `gap-1` / `gap-2` |
| Related fields in a form | `gap-4` |
| Sections within a card | `gap-6` |
| Page sections | `gap-8` |
| Page padding (mobile) | `p-4` |
| Page padding (desktop) | `md:p-8` |

Card/section radius: `rounded-md` (default) or `rounded-lg` for prominent containers —
never mix radii within the same page.

---

## Responsive layout (mobile-first)

```typescript
// base = mobile, then layer enhancements
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  {/* cards */}
</div>

<div className="flex flex-col gap-4 md:flex-row md:items-center md:justify-between">
  {/* header that stacks on mobile, row on desktop */}
</div>
```

Test every screen at three widths before calling it done: 375px (mobile), 768px (tablet),
1440px (desktop).

---

## Forms — feedback and validation

- Inline error under the field via `FormMessage` (always) **and** a toast on submit failure
  (only for the overall submit, not per-field).
- Disable the submit button while `isSubmitting`/`isPending`, and swap its label
  (`"Salvando..."`).
- Group related fields visually (`fieldset`-like `div` with `gap-4`), separate unrelated
  groups with a divider or extra spacing (`gap-8`).

---

## Accessibility checklist

- Every interactive element is reachable and operable via keyboard (`Tab`, `Enter`, `Esc`)
  — shadcn/Radix primitives handle this by default, don't override `tabIndex` unnecessarily.
- Every icon-only button has an `aria-label` or wraps in a `Tooltip` with visible text.
- Color contrast meets WCAG AA (4.5:1 for text) — rely on the existing shadcn theme tokens
  (`text-foreground`, `text-muted-foreground`) rather than custom grays.
- Form inputs always pair with a `FormLabel` (not just a placeholder).
- Focus states are never removed (`focus:outline-none` without a replacement ring is not
  acceptable).

---

## Motion

```typescript
// Prefer Tailwind transition utilities over custom animation for micro-interactions
<div className="transition-colors duration-150 ease-out hover:bg-accent" />
```

Keep durations between 150–250ms for hover/press feedback; 300–400ms for
enter/exit transitions (dialogs, sheets). Anything longer feels sluggish; anything
shorter feels jarring.
