# GUARDRAILS

Organization-wide coding standards, written for both humans and AI agents.
The rules are spread across focused markdown files (each `~150` lines max) so
agents can load exactly the guidance they need for a given task.

## What this covers

- **React 19 + Next.js App Router** - component patterns, server/client split
- **UI libraries** - Mantine and shadcn, with the rule that the library owns
  styling and you never override it (incl. no dot notation for Mantine
  sub-components)
- **End-to-end type safety** - oRPC and Elysia via Eden Treaty (separate stacks,
  separate rules)
- **Database** - Drizzle tables/relations/migrations and the `acme` seeding
  pattern (feature seeders, data.json, root entry)
- **Auth** - Better Auth, admin guards, protected routes
- **Validation** - TypeBox for Elysia, Zod everywhere else
- **Server state** - TanStack Query, centralized query keys
- **URL state** - nuqs (search params as the single source of truth)
- **Data grids** - TanStack Table backed by nuqs
- **SEO** - reusable SEO config component with OG/Twitter/JSON-LD metadata
- **Tooling** - Biome, typecheck, lefthook, CI gates

## Repository layout

```
guardrails/
├── AGENTS.MD          # Agent entry point. Read first.
├── README.MD          # This file.
├── rules/             # One discipline per file.
└── examples/          # Canonical, copy-ready code patterns.
```

## Rules at a glance

- `GENERAL.MD` — baseline TypeScript and code-organization conventions.
- `NAMING.MD` — folders, files, and symbol naming (kebab-case, PascalCase).
- `FEATURE-FOLDER.MD` — feature-first folders; API and web mirror each other.
- `SEEDING.MD` — database seeding pattern (db-package only, data.json).
- `DRIZZLE.MD` — tables, relations, client, migrations.
- `ENV-VARS.MD` — validated env parsing per package.
- `VALIDATION.MD` — TypeBox for Elysia, Zod elsewhere.
- `AUTH.MD` — Better Auth, admin guards, protected routes.
- `REACT.MD` — arrow functions, named exports, props typing.
- `NEXTJS.MD` — App Router, routing, metadata.
- `STYLING.MD` — Mantine or shadcn; no `className`/`style` overrides.
- `DATA-FETCHING.MD` — TanStack Query, query keys, colocation.
- `API-CONTRACT.MD` — shared end-to-end type safety principles and error envelope.
- `ORPC.MD` — oRPC stack (folder structure, root router, client).
- `ELYSIA-EDEN.MD` — Elysia + Eden standalone API stack.
- `STATE-MANAGEMENT.MD` — nuqs URL state.
- `DATA-TABLES.MD` — TanStack Table + nuqs.
- `SEO.MD` — SEO config component and metadata.
- `TOOLING.MD` — Biome, typecheck, lefthook, CI.

## How to evolve these rules

- Keep every rule file under ~150 lines. If a file grows, split the topic into
  a new focused file and update `AGENTS.MD`.
- Prefer concrete examples over abstract guidance. An LLM behaves more
  reliably when given a correct snippet to mirror.
- When the stack changes (e.g. a new ORM or UI library), add a rule file rather
  than bolting it onto an existing one.