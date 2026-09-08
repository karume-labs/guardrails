# Guardrail

Organization-wide coding standards, written for both humans and AI agents.
The rules are spread across focused markdown files (each `~150` lines max) so
agents can load exactly the guidance they need for a given task.

## What this covers

- **React 19 + Next.js App Router** - component patterns, server/client split
- **UI libraries** - Mantine and shadcn, with the rule that the library owns
  styling and you never override it
- **End-to-end type safety** - oRPC (preferred) and Elysia via Eden Treaty
- **Server state** - TanStack Query, centralized query keys
- **URL state** - nuqs (search params as the single source of truth)
- **Data grids** - TanStack Table backed by nuqs
- **SEO** - reusable SEO config component with OG/Twitter/JSON-LD metadata

## Repository layout

```
guardrail/
├── AGENTS.MD          # Agent entry point. Read first.
├── README.MD          # This file.
├── rules/             # One discipline per file.
└── examples/          # Canonical, copy-ready code patterns.
```

## How to evolve these rules

- Keep every rule file under ~150 lines. If a file grows, split the topic into
  a new focused file and update `AGENTS.MD`.
- Prefer concrete examples over abstract guidance. An LLM behaves more
  reliably when given a correct snippet to mirror.
- When the stack changes (e.g. a new ORM or UI library), add a rule file rather
  than bolting it onto an existing one.