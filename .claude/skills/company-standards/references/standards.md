# Full Development Standards Reference

## Table of Contents
1. [Stack](#stack)
2. [Folder Structure — Backend](#folder-structure--backend)
3. [Folder Structure — Frontend](#folder-structure--frontend)
4. [Backend Patterns](#backend-patterns)
   - [Entity](#entity)
   - [DTO](#dto)
   - [Controller](#controller)
   - [Service](#service)
   - [Module](#module)
5. [Frontend Patterns](#frontend-patterns)
   - [Component](#component)
   - [Hook](#hook)
   - [Page (App Router)](#page-app-router)
   - [Form](#form)
6. [API Patterns](#api-patterns)
7. [Authentication](#authentication)
8. [Testing](#testing)
9. [Environment Variables](#environment-variables)
10. [Useful Commands](#useful-commands)

---

## Stack

### Backend
- NestJS 11+, TypeScript 5+
- TypeORM + PostgreSQL
- JWT + Passport (auth), CASL (authorization)
- class-validator + class-transformer (validation)
- @nestjs/swagger (docs)
- Jest (tests)

### Frontend
- Next.js 16+ (App Router), TypeScript 5+
- shadcn/ui + Radix UI (components)
- Tailwind CSS 4+ (styling)
- React Hook Form + Zod (forms)
- TanStack Query / React Query (server state)
- Axios (HTTP)
- NextAuth.js (auth session)
- Vitest + React Testing Library (unit tests)
- Cypress (E2E tests)

---

## Folder Structure — Backend

```
src/
├── auth/
│   ├── dto/
│   ├── google/
│   └── mfa/
├── casl/
│   └── guard/
├── common/
│   ├── entity/        # EntityBase, RoleBase
│   ├── enums/
│   ├── paginations/
│   └── util/
├── database/
│   └── seeds/
├── env/
├── jwt/
├── mailer/
├── storage/
│   ├── dto/
│   ├── interfaces/
│   └── services/
├── users/
│   └── dto/
└── [feature]/
    ├── dto/
    ├── interfaces/
    ├── repositories/
    ├── [feature].controller.ts
    ├── [feature].service.ts
    ├── [feature].module.ts
    └── [feature].entity.ts
```

---

## Folder Structure — Frontend

```
├── @types/
├── app/
│   ├── (route-group)/
│   │   └── route/
│   │       ├── layout.tsx
│   │       └── page.tsx
│   ├── api/
│   │   └── auth/
│   ├── services/
│   └── _layouts/
├── components/
│   ├── (feature)/
│   │   └── component-name/
│   │       ├── index.tsx          # Visual only
│   │       └── useComponentName.tsx  # Logic only
│   ├── global/
│   │   ├── form/
│   │   └── icons/
│   └── ui/                        # shadcn/ui components
├── hooks/
├── lib/
└── src/
    └── gen/                       # Orval-generated code
```

---

## Backend Patterns

### Entity

```typescript
import { EntityBase } from '@/common/entity/entity-base';
import { Column, Entity } from 'typeorm';

@Entity('table_name')           // snake_case table name
export class MyEntity extends EntityBase {
  @Column({ length: 255 })
  name: string;

  @Column({ name: 'field_name' })  // snake_case in DB
  fieldName: string;               // camelCase in code

  @Column({ name: 'is_active', default: true })
  isActive: boolean;
}
```

**Rules:**
- Always extend `EntityBase` (provides id, created_at, updated_at, deleted_at)
- Table names: `snake_case`
- DB columns: `{ name: 'snake_case' }` with camelCase TS property
- Use soft delete via `deleted_at` — never hard delete

### DTO

```typescript
import { ApiProperty } from '@nestjs/swagger';
import { IsEmail, IsNotEmpty, IsOptional, IsString } from 'class-validator';

export class CreateUserDto {
  @ApiProperty()
  @IsNotEmpty({ message: 'Name is required' })
  @IsString()
  name: string;

  @ApiProperty()
  @IsEmail({}, { message: 'Invalid email' })
  email: string;

  @ApiProperty({ required: false })
  @IsOptional()
  @IsString()
  photoUrl?: string;
}

export class UpdateUserDto {
  @ApiProperty({ required: false })
  @IsOptional()
  @IsString()
  name?: string;
}
```

**Rules:**
- All fields annotated with `@ApiProperty` for Swagger docs
- Use class-validator decorators with Portuguese error messages
- Create separate `Create*Dto` and `Update*Dto`
- File: `dto/create-[feature].dto.ts`, `dto/update-[feature].dto.ts`

### Controller

```typescript
import { Body, Controller, Delete, Get, Param, Post, Put, UseGuards } from '@nestjs/common';
import { ApiBearerAuth, ApiOkResponse, ApiOperation, ApiTags } from '@nestjs/swagger';
import { PoliciesGuard } from '@/casl/guard/policies.guard';
import { FeatureService } from './feature.service';
import { CreateFeatureDto } from './dto/create-feature.dto';
import { UpdateFeatureDto } from './dto/update-feature.dto';
import { FeatureEntity } from './feature.entity';

@UseGuards(PoliciesGuard)
@ApiBearerAuth()
@ApiTags('feature-name')
@Controller('feature-name')
export class FeatureController {
  constructor(private readonly featureService: FeatureService) {}

  @ApiOperation({ summary: 'List all features' })
  @ApiOkResponse({ description: 'Feature list', type: [FeatureEntity] })
  @Get()
  async findAll(): Promise<FeatureEntity[]> {
    return this.featureService.findAll();
  }

  @ApiOperation({ summary: 'Get feature by id' })
  @ApiOkResponse({ description: 'Feature found', type: FeatureEntity })
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<FeatureEntity> {
    return this.featureService.findOne(id);
  }

  @ApiOperation({ summary: 'Create feature' })
  @ApiOkResponse({ description: 'Feature created', type: FeatureEntity })
  @Post()
  async create(@Body() dto: CreateFeatureDto): Promise<FeatureEntity> {
    return this.featureService.create(dto);
  }

  @ApiOperation({ summary: 'Update feature' })
  @ApiOkResponse({ description: 'Feature updated', type: FeatureEntity })
  @Put(':id')
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateFeatureDto,
  ): Promise<FeatureEntity> {
    return this.featureService.update(id, dto);
  }

  @ApiOperation({ summary: 'Delete feature' })
  @Delete(':id')
  async remove(@Param('id') id: string): Promise<void> {
    return this.featureService.remove(id);
  }
}
```

### Service

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { FeatureEntity } from './feature.entity';
import { CreateFeatureDto } from './dto/create-feature.dto';
import { UpdateFeatureDto } from './dto/update-feature.dto';

@Injectable()
export class FeatureService {
  constructor(
    @InjectRepository(FeatureEntity)
    private readonly repository: Repository<FeatureEntity>,
  ) {}

  async findAll(): Promise<FeatureEntity[]> {
    return this.repository.find({ where: { deletedAt: null } });
  }

  async findOne(id: string): Promise<FeatureEntity> {
    const entity = await this.repository.findOne({ where: { id } });
    if (!entity) throw new NotFoundException('Feature not found');
    return entity;
  }

  async create(dto: CreateFeatureDto): Promise<FeatureEntity> {
    const entity = this.repository.create(dto);
    return this.repository.save(entity);
  }

  async update(id: string, dto: UpdateFeatureDto): Promise<FeatureEntity> {
    const entity = await this.findOne(id);
    Object.assign(entity, dto);
    return this.repository.save(entity);
  }

  async remove(id: string): Promise<void> {
    const entity = await this.findOne(id);
    await this.repository.softDelete(entity.id);
  }
}
```

**Error handling:**
```typescript
throw new NotFoundException('Resource not found');
throw new ForbiddenException('Access denied');
throw new BadRequestException('Validation error');
throw new ConflictException('Resource already exists');
```

### Module

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { FeatureController } from './feature.controller';
import { FeatureService } from './feature.service';
import { FeatureEntity } from './feature.entity';

@Module({
  imports: [TypeOrmModule.forFeature([FeatureEntity])],
  controllers: [FeatureController],
  providers: [FeatureService],
  exports: [FeatureService],
})
export class FeatureModule {}
```

---

## Frontend Patterns

### Component

Always split into two files in `components/(feature)/component-name/`:

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
    queryFn: () => api.getAll(),
  });

  const onSubmit = () => {
    // handler
  };

  return { data, isLoading, onSubmit };
};
```

### Page (App Router)

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

### Form

Always use React Hook Form + Zod:

```typescript
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { toast } from 'sonner';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';

const schema = z.object({
  name: z.string().min(1, 'Nome é obrigatório'),
  email: z.string().email('E-mail inválido'),
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
      await api.create(data);
      toast.success('Salvo com sucesso!', { id: 'save' });
    } catch {
      toast.error('Erro ao salvar.', { id: 'save' });
    }
  };

  return { form, onSubmit: form.handleSubmit(onSubmit) };
};
```

### Tailwind class merging

```typescript
import { cn } from '@/lib/utils';

<div className={cn(
  'base-classes',
  condition && 'conditional-classes',
  className,
)} />
```

---

## API Patterns

### Paginated response (backend)

```typescript
interface PaginationResponse<T> {
  data: T[];
  meta: {
    page: number;
    perPage: number;
    total: number;
  };
}
```

### Paginated query (frontend)

```typescript
const { data, isLoading } = useQuery({
  queryKey: ['feature', page, perPage],
  queryFn: () => api.getAll({ page, perPage }),
});
```

---

## Authentication

### Backend
- JWT with Passport (`@nestjs/passport`)
- `PoliciesGuard` for route protection (CASL)
- MFA support (TOTP)
- OAuth via Google

### Frontend
- NextAuth.js for session management
- Middleware for protected routes
- Tokens stored in httpOnly cookies

---

## Testing

### Unit tests (Vitest + React Testing Library)

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
});
```

**Test file locations:**
- `__tests__/components/` — component tests
- `__tests__/hooks/` — hook tests
- `__tests__/utils/` — utility tests

**Best practices:**
- Use `data-testid` attributes for stable E2E selectors
- Mock external dependencies (APIs, services)
- Test behavior, not implementation
- Name tests as "deve [ação esperada]"
- Each test should be independent

### E2E (Cypress)

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
});
```

**Custom commands available:**
- `cy.login(email, password)` — authenticates using session cache
- `cy.getByTestId(testId)` — selects element by `data-testid`

---

## Environment Variables

### Backend (.env)
```
DATABASE_HOST=
DATABASE_PORT=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_NAME=
JWT_SECRET=
JWT_EXPIRES_IN=
MAIL_HOST=
MAIL_USER=
MAIL_PASS=
STORAGE_PROVIDER=   # gcs or s3
```

### Frontend (.env.local)
```
NEXTAUTH_URL=
NEXTAUTH_SECRET=
NEXT_PUBLIC_API_URL=
```

---

## Useful Commands

### Backend
```bash
pnpm start:dev       # Dev server
pnpm build           # Build
pnpm test            # Tests (Jest)
pnpm seed            # Run seeders
pnpm make:entity     # Scaffold new entity
```

### Frontend
```bash
pnpm dev             # Dev server (Turbopack)
pnpm build           # Build
pnpm lint            # Lint
pnpm test            # Unit tests (Vitest)
pnpm test:watch      # Watch mode
pnpm test:coverage   # Coverage report
pnpm cy:open         # Cypress interactive
pnpm cy:run          # Cypress headless
pnpm cy:run:ci       # Cypress for CI/CD
pnpm orval           # Regenerate API types from Swagger
```
