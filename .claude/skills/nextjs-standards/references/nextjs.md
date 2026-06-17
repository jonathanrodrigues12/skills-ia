# Next.js Frontend Patterns Reference

## Component

Always split into two files inside `components/(feature)/component-name/`:

**index.tsx** — visual only, no business logic:
```typescript
'use client';
import { useComponentName } from './useComponentName';

export function ComponentName() {
  const { data, isLoading, onSubmit } = useComponentName();

  if (isLoading) return <div>Carregando...</div>;

  return (
    <div>
      {/* JSX here */}
    </div>
  );
}
```

**useComponentName.tsx** — all logic, state, queries:
```typescript
import { useQuery } from '@tanstack/react-query';
import { api } from '@/app/services/api';

export const useComponentName = () => {
  const { data, isLoading } = useQuery({
    queryKey: ['feature'],
    queryFn: () => api.get('/feature'),
  });

  const onSubmit = () => {};

  return { data, isLoading, onSubmit };
};
```

---

## Page (App Router)

```typescript
// app/(group)/route/page.tsx
'use client';
import Layout from '@/app/_layouts/root';
import FeatureComponent from '@/components/(feature)/feature-component';

export default function FeaturePage() {
  return (
    <Layout breadCrumbItems={[{ title: 'Feature', url: '/feature' }]}>
      <FeatureComponent />
    </Layout>
  );
}
```

---

## Form

Always use React Hook Form + Zod:

**useFeatureForm.tsx:**
```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { toast } from 'sonner';
import { api } from '@/app/services/api';

const schema = z.object({
  name: z.string().min(1, 'Nome é obrigatório'),
  email: z.string().email('E-mail inválido'),
  phone: z.string().min(10, 'Telefone inválido').optional(),
});

type FormData = z.infer<typeof schema>;

export const useFeatureForm = () => {
  const form = useForm<FormData>({
    resolver: zodResolver(schema),
    defaultValues: { name: '', email: '' },
  });

  const onSubmit = async (data: FormData) => {
    try {
      toast.loading('Salvando...', { id: 'save' });
      await api.post('/feature', data);
      toast.success('Salvo com sucesso!', { id: 'save' });
      form.reset();
    } catch {
      toast.error('Erro ao salvar.', { id: 'save' });
    }
  };

  return { form, onSubmit: form.handleSubmit(onSubmit), isSubmitting: form.formState.isSubmitting };
};
```

**index.tsx:**
```typescript
'use client';
import { useFeatureForm } from './useFeatureForm';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';

export function FeatureForm() {
  const { form, onSubmit, isSubmitting } = useFeatureForm();

  return (
    <Form {...form}>
      <form onSubmit={onSubmit} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nome</FormLabel>
              <FormControl>
                <Input data-testid="name-input" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" disabled={isSubmitting} data-testid="submit-button">
          {isSubmitting ? 'Salvando...' : 'Salvar'}
        </Button>
      </form>
    </Form>
  );
}
```

---

## Tailwind Class Merging

```typescript
import { cn } from '@/lib/utils';

<div className={cn(
  'base-classes',
  condition && 'conditional-classes',
  className,
)} />
```

---

## TanStack Query — Paginated

```typescript
const { data, isLoading } = useQuery({
  queryKey: ['feature', page, perPage],
  queryFn: () => api.get('/feature', { params: { page, perPage } }),
});
```

---

## Unit Tests (Vitest + React Testing Library)

```typescript
// __tests__/components/feature-component.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { FeatureComponent } from '@/components/(feature)/feature-component';

vi.mock('@/hooks/useAuth', () => ({
  useAuth: () => ({ isLoading: false }),
}));

describe('FeatureComponent', () => {
  it('deve renderizar corretamente', () => {
    render(<FeatureComponent />);
    expect(screen.getByRole('heading')).toBeInTheDocument();
  });

  it('deve exibir erro de validação', async () => {
    const user = userEvent.setup();
    render(<FeatureComponent />);
    await user.click(screen.getByTestId('submit-button'));
    await waitFor(() => {
      expect(screen.getByText(/obrigatório/i)).toBeInTheDocument();
    });
  });
});
```

**Test file locations:**
- `__tests__/components/` — component tests
- `__tests__/hooks/` — hook tests
- `__tests__/utils/` — utility tests

---

## E2E Tests (Cypress)

```typescript
// cypress/e2e/feature/feature.cy.ts
describe('Feature', () => {
  beforeEach(() => {
    cy.login('admin@example.com', 'admin123');
    cy.visit('/feature');
  });

  it('deve listar itens', () => {
    cy.getByTestId('feature-table').should('be.visible');
  });

  it('deve criar novo item', () => {
    cy.getByTestId('add-button').click();
    cy.getByTestId('name-input').type('Novo Item');
    cy.getByTestId('submit-button').click();
    cy.contains('Criado com sucesso').should('be.visible');
  });
});
```

**Custom commands:**
- `cy.login(email, password)` — authenticates using session cache
- `cy.getByTestId(testId)` — selects by `data-testid`

---

## Authentication

- NextAuth.js for session management
- Middleware for protected routes
- Tokens stored in httpOnly cookies

---

## Environment Variables

```env
NEXTAUTH_URL=
NEXTAUTH_SECRET=
NEXT_PUBLIC_API_URL=
```

---

## Orval — API Code Generation

Orval generates TypeScript types, API functions, and TanStack Query hooks from the backend Swagger (OpenAPI) spec. **Always use generated code from `src/gen/` — never hand-write API types or fetch functions.**

### Configuration (`orval.config.ts`)

```typescript
import { defineConfig } from 'orval';

export default defineConfig({
  api: {
    input: {
      target: process.env.NEXT_PUBLIC_API_URL + '/api-json',
    },
    output: {
      mode: 'tags-split',
      target: './src/gen/api.ts',
      schemas: './src/gen/model',
      client: 'react-query',
      httpClient: 'axios',
      override: {
        mutator: {
          path: './app/services/api.ts',
          name: 'api',
        },
      },
    },
  },
});
```

### Generated files (`src/gen/`)

| File | Content |
|---|---|
| `api.ts` | API functions and TanStack Query hooks |
| `api.zod.ts` | Zod schemas matching backend DTOs |
| `model/` | TypeScript types for all entities and DTOs |

### Using generated hooks in a component hook

```typescript
// components/(feature)/feature-list/useFeatureList.tsx
import { useGetFeature } from '@/src/gen/api';

export const useFeatureList = () => {
  const { data, isLoading } = useGetFeature();

  return { data: data?.data ?? [], isLoading };
};
```

### Using generated mutation hooks

```typescript
// components/(feature)/feature-form/useFeatureForm.tsx
import { useCreateFeature } from '@/src/gen/api';
import { CreateFeatureDto } from '@/src/gen/model';
import { toast } from 'sonner';

export const useFeatureForm = () => {
  const { mutateAsync, isPending } = useCreateFeature();

  const onSubmit = async (data: CreateFeatureDto) => {
    try {
      toast.loading('Salvando...', { id: 'save' });
      await mutateAsync({ data });
      toast.success('Salvo com sucesso!', { id: 'save' });
    } catch {
      toast.error('Erro ao salvar.', { id: 'save' });
    }
  };

  return { onSubmit, isPending };
};
```

### Using generated Zod schemas in forms

```typescript
import { useCreateFeatureBody } from '@/src/gen/api.zod';

const schema = useCreateFeatureBody();

const form = useForm({
  resolver: zodResolver(schema),
});
```

### When to regenerate

Run `pnpm orval` whenever the backend Swagger changes (new endpoints, updated DTOs, renamed fields). Commit the `src/gen/` changes together with the frontend code that uses them.

---

## Useful Commands

```bash
pnpm dev             # Dev server (Turbopack)
pnpm build           # Build
pnpm lint            # Lint
pnpm test            # Unit tests (Vitest)
pnpm test:watch      # Watch mode
pnpm test:coverage   # Coverage report
pnpm cy:open         # Cypress interactive
pnpm cy:run          # Cypress headless
pnpm orval           # Regenerate API types from Swagger
```
