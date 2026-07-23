# System Design Pattern Reference

## Feature design worksheet

For any feature spanning backend + frontend, fill this out (mentally or in the plan) before
coding:

```
Resource:        <name, e.g. "invoice">
Owning module:    <backend module that owns the table/repository>
Lifecycle:        create / update / soft-delete / list / detail
Who can access:   <roles/policies — CASL ability>
Server state:     <what's fetched via TanStack Query, cache key shape>
Client state:     <what's local-only: dialog open, selected row, draft>
Failure modes:    404 / 403 / 422 (validation) / network — mapped to which UI state
```

---

## Contract-first flow

```
1. Define DTOs in the backend (create-x.dto.ts / update-x.dto.ts) with class-validator +
   @ApiProperty.
2. Backend exposes the endpoint, Swagger reflects the DTOs automatically.
3. Run `pnpm orval` on the frontend — generates types, API functions, TanStack Query hooks,
   and Zod schemas in src/gen/.
4. Frontend hooks (useFeatureName.tsx) consume the generated hooks — never hand-roll a
   fetch call or a type that mirrors the DTO.
```

Changing a field name/shape always starts on the backend DTO — never patch a generated
frontend type by hand; regenerate instead.

---

## State ownership decision table

| Data | Owner | Mechanism |
|---|---|---|
| List of records from DB | Backend | TanStack Query (`useQuery`) |
| Form draft before submit | Local | `useForm` state (React Hook Form) |
| "Is this dialog open" | Local | `useState` in the component's hook |
| Selected table row(s) | Local | `useState`, lifted only if a sibling needs it |
| Auth session / current user | Global | NextAuth session, existing `useAuth` |
| Optimistic mutation result | Backend (cache) | TanStack Query `onMutate` / `invalidateQueries` |

Rule of thumb: if a page refresh should lose it, it's local state. If another user's
action should be reflected in it, it's server state via TanStack Query — never duplicated
into a separate client store.

---

## Request lifecycle mapping

```
Client (index.tsx)
  → useFeatureForm() [validates shape via Zod, same rules as backend DTO intent]
    → generated mutation hook (src/gen/api.ts)
      → HTTP request
        → Controller: @UseGuards(PoliciesGuard) — authorization
          → DTO validation (class-validator) — shape/format
            → Service — business rules, calls Repository
              → Repository/TypeORM — persistence
        ← Response (DTO) or NestJS exception (404/403/400)
      ← Orval-typed response or typed error
    ← onSuccess: invalidateQueries / toast.success
    ← onError: toast.error, keep form data for retry
  → UI reflects loading/empty/error/success (see ux-design skill)
```

Validation exists in two places on purpose: Zod on the frontend for fast feedback, DTO
validators on the backend as the actual authority — never skip the backend one because the
frontend already checks.

---

## Cross-module access

```typescript
// Wrong — reaching into another module's repository directly
// invoice.service.ts
constructor(@InjectRepository(Customer) private customerRepo: Repository<Customer>) {}

// Right — go through the owning module's service
// invoice.module.ts
imports: [CustomersModule]

// invoice.service.ts
constructor(private readonly customersService: CustomersService) {}

async createInvoice(dto: CreateInvoiceDto) {
  const customer = await this.customersService.findOneOrFail(dto.customerId);
  // ...
}
```

Export the service (not the repository) from the owning module for other modules to import.

---

## Unbounded lists — server-side pagination

```typescript
// Backend: controller accepts page/perPage/sort/filter query params, service applies them
// via TypeORM's skip/take + order + where — never `findAll()` then slice in memory.

// Frontend: page/perPage are state that drives the query key, not a client-side slice
// of an already-fetched full list.
const [page, setPage] = useState(1);
const { data } = useGetFeature({ page, perPage: 20 });
```

---

## Duplicate-submission guards

```typescript
// Disable the trigger while the mutation is in flight — the minimum guard for every
// non-idempotent action.
<Button disabled={isPending} onClick={onSubmit} data-testid="submit-button">
  {isPending ? 'Salvando...' : 'Salvar'}
</Button>
```

For operations with real external side effects (payments, emails sent), pass an
idempotency key generated once per form session, not per click, so a retried request
after a timeout doesn't duplicate the side effect.
