# Skills IA

Skills do Claude Code com padrões de desenvolvimento. Instale em qualquer projeto para que o Claude gere código consistente com a stack e convenções.

## Skills disponíveis

| Skill | Descrição |
|---|---|
| `nestjs-standards` | Padrões de backend: entidades, DTOs, services, controllers, módulos |
| `nextjs-standards` | Padrões de frontend: componentes, hooks, páginas, formulários, testes e geração de código com Orval |
| `company-standards` | As duas skills acima em um único pacote |
| `ux-design` | Padrões de UX/UI: hierarquia, estados de loading/empty/error, confirmação de ações destrutivas, acessibilidade e motion |
| `layout-design` | Padrões de layout e proporção: grid, largura de container, escala tipográfica, ritmo de espaçamento e aspect ratio |
| `system-design` | Padrões de arquitetura de feature ponta a ponta: contrato de API, ownership de estado (server vs. client) e integração backend↔frontend |

---

## Instalação

### Pré-requisito

Ter o [Claude Code](https://claude.ai/code) instalado.

---

### Via skills.sh (recomendado)

O [skills.sh](https://skills.sh) é um marketplace de skills para agentes de IA. Use o comando abaixo para instalar diretamente do repositório:

**Backend (NestJS)**
```bash
npx skills add jonathanrodrigues12/skills-ia/nestjs-standards
```

**Frontend (Next.js)**
```bash
npx skills add jonathanrodrigues12/skills-ia/nextjs-standards
```

**Ambos (pacote completo)**
```bash
npx skills add jonathanrodrigues12/skills-ia/company-standards
```

**Design (UX)**
```bash
npx skills add jonathanrodrigues12/skills-ia/ux-design
```

**Layout e proporção**
```bash
npx skills add jonathanrodrigues12/skills-ia/layout-design
```

**Arquitetura de features (system design)**
```bash
npx skills add jonathanrodrigues12/skills-ia/system-design
```

---

### Via Claude Code CLI

**Backend (NestJS)**
```bash
claude skill install https://raw.githubusercontent.com/jonathanrodrigues12/skills-ia/main/nestjs-standards/SKILL.md
```

**Frontend (Next.js)**
```bash
claude skill install https://raw.githubusercontent.com/jonathanrodrigues12/skills-ia/main/nextjs-standards/SKILL.md
```

**Ambos (pacote completo)**
```bash
claude skill install https://raw.githubusercontent.com/jonathanrodrigues12/skills-ia/main/company-standards/SKILL.md
```

**Design (UX)**
```bash
claude skill install https://raw.githubusercontent.com/jonathanrodrigues12/skills-ia/main/ux-design/SKILL.md
```

**Layout e proporção**
```bash
claude skill install https://raw.githubusercontent.com/jonathanrodrigues12/skills-ia/main/layout-design/SKILL.md
```

**Arquitetura de features (system design)**
```bash
claude skill install https://raw.githubusercontent.com/jonathanrodrigues12/skills-ia/main/system-design/SKILL.md
```

---

## Como funciona

Após instalar, o Claude passa a aplicar automaticamente os padrões da empresa ao gerar código. Não é necessário nenhum comando especial — basta pedir normalmente:

> "Cria um módulo de produtos no backend"
> "Cria um componente de listagem de pedidos"
> "Cria um formulário de cadastro de cliente"
> "Melhora a UX dessa tela, tá muito apertada"
> "Ajusta o layout pra ficar mais proporcional"
> "Como devo estruturar essa feature entre backend e frontend?"

O Claude vai seguir a estrutura de pastas, nomenclatura, padrões de entidade, DTO, controller, service, hooks, formulários e testes definidos nas skills — além de hierarquia visual, grid, proporção e arquitetura de dados quando as skills de design/arquitetura entrarem em ação.

---

## Estrutura do repositório

```
nestjs-standards/        # Skill de backend
├── SKILL.md
└── references/
    └── nestjs.md
nextjs-standards/        # Skill de frontend
├── SKILL.md
└── references/
    └── nextjs.md
company-standards/       # Skill completa (backend + frontend)
├── SKILL.md
└── references/
    └── standards.md
ux-design/               # Skill de UX/UI: hierarquia, estados, acessibilidade
├── SKILL.md
└── references/
    └── ux-patterns.md
layout-design/           # Skill de layout e proporção: grid, escala, aspect ratio
├── SKILL.md
└── references/
    └── layout-patterns.md
system-design/           # Skill de arquitetura de feature ponta a ponta
├── SKILL.md
└── references/
    └── architecture.md
```
