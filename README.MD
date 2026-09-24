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
- **Database** - Drizzle tables/relations/migrations and the feature-based
  seeding pattern (feature seeders, data.json, root entry)
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
├── AGENTS.md          # Always-on agent entry point. Read first.
├── README.MD          # This file.
├── .agents/skills/    # Codex-discoverable task workflow
├── rules/             # One discipline per file.
└── examples/          # Canonical, copy-ready code patterns.
```

## Agent entry points

- `AGENTS.md` contains the always-on project guidance and rule index.
- `.agents/skills/karume-guardrails/SKILL.md` routes a task to the smallest
  relevant set of rules and examples.
- `.claude/skills/karume-guardrails` points to the same skill directory for
  Claude Code. The two agents use different project skill discovery paths, but
  there is only one skill file to maintain.

Claude Code reads `AGENTS.md` as a fallback when no `CLAUDE.md` exists. A
project that already has `CLAUDE.md` should keep its existing entry point and
point it at the guardrails as appropriate.

## Rules at a glance

- `AGENT-USAGE.MD` - how agents load and apply these rules (read first).
- `GENERAL.MD` - baseline TypeScript and code-organization conventions.
- `FALLBACKS.MD` - no `??`/`||`/`as` on already-typed or validated values.
- `NAMING.MD` - folders, files, and symbol naming (kebab-case, PascalCase).
- `FEATURE-FOLDER.MD` - feature-first folders; API and web mirror each other.
- `SEEDING.MD` - database seeding pattern (db-package only, data.json).
- `DRIZZLE.MD` - tables, relations, client, migrations.
- `ENV-VARS.MD` - validated env parsing per package.
- `VALIDATION.MD` - TypeBox for Elysia, Zod elsewhere.
- `AUTH.MD` - Better Auth, admin guards, protected routes.
- `REACT.MD` - arrow functions, named exports, props typing.
- `NEXTJS.MD` - App Router, routing, metadata.
- `STYLING.MD` - Mantine or shadcn; no `className`/`style` overrides.
- `DATA-FETCHING.MD` - TanStack Query, query keys, colocation.
- `API-CONTRACT.MD` - shared end-to-end type safety principles and error envelope.
- `ORPC.MD` - oRPC stack (folder structure, root router, client).
- `ELYSIA-EDEN.MD` - Elysia + Eden standalone API stack.
- `STATE-MANAGEMENT.MD` - nuqs URL state.
- `DATA-TABLES.MD` - TanStack Table + nuqs.
- `SEO.MD` - SEO config component and metadata.
- `TOOLING.MD` - Biome, typecheck, lefthook, CI.

## How to evolve these rules

- Keep every rule file under ~150 lines. If a file grows, split the topic into
  a new focused file and update `AGENTS.md`.
- Prefer concrete examples over abstract guidance. An LLM behaves more
  reliably when given a correct snippet to mirror.
- When the stack changes (e.g. a new ORM or UI library), add a rule file rather
  than bolting it onto an existing one.
