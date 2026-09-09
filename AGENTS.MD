# AGENTS.MD

This repository is the **single source of truth** for coding standards across all
projects in the organization. Read it fully before writing code.

## How to use this

- Read `rules/*.MD` in order. Each file is self-contained and under 150 lines.
- Consult `examples/*.MD` for canonical, copy-ready code patterns.
- When a rule and a refactor conflict, follow the rule. If a rule is ambiguous,
  preserve the surrounding code's existing conventions.

## Rules index

| File                                        | Scope                                        |
| :------------------------------------------ | :------------------------------------------- |
| `rules/GENERAL.MD`                          | Functions, exports, imports, file size       |
| `rules/NAMING.MD`                           | Folders, files, symbols naming               |
| `rules/FEATURE-FOLDER.MD`                   | Feature-based folder architecture            |
| `rules/SEEDING.MD`                          | Database seeding (acme pattern)             |
| `rules/DRIZZLE.MD`                          | Drizzle tables, relations, client, migrations|
| `rules/ENV-VARS.MD`                         | Typed env parsing per package                |
| `rules/VALIDATION.MD`                       | TypeBox (Elysia) vs Zod (elsewhere)          |
| `rules/AUTH.MD`                             | Better Auth, admin guards, protected routes  |
| `rules/REACT.MD`                            | Components, props, server/client split       |
| `rules/NEXTJS.MD`                           | App Router, routing, metadata                |
| `rules/STYLING.MD`                          | Mantine + shadcn, no style overrides         |
| `rules/DATA-FETCHING.MD`                    | TanStack Query, keys, service colocation     |
| `rules/API-CONTRACT.MD`                     | End-to-end type safety (shared principles)   |
| `rules/ORPC.MD`                           | oRPC folder structure + client + React Query |
| `rules/ELYSIA-EDEN.MD`                    | Elysia + Eden standalone API stack           |
| `rules/STATE-MANAGEMENT.MD`                 | nuqs, URL-driven state                       |
| `rules/DATA-TABLES.MD`                      | TanStack Table + nuqs data grids             |
| `rules/SEO.MD`                              | SEO config component, OG metadata            |
| `rules/TOOLING.MD`                          | Biome, typecheck, lefthook, CI               |

## Examples index

| File                                  | Copy-ready pattern                              |
| :------------------------------------ | :---------------------------------------------- |
| `examples/ORPC-STACK.MD`              | Full oRPC server -> client -> React-Query stack |
| `examples/ELYSIA-EDEN.MD`             | Elysia app assembly + treaty client             |
| `examples/TANSTACK-QUERY.MD`          | keys.ts + queries.ts + mutations.ts             |
| `examples/DATA-TABLE.MD`              | TanStack Table columns + view with nuqs         |
| `examples/SEO-METADATA.MD`            | SEO component and generateMetadata              |
| `examples/ERROR-HANDLING.MD`          | Typed error classes + global onError plugin      |

## Hard rules (non-negotiable)

1. No file exceeds `~150` lines. Split on growth.
2. No relative imports. Absolute path aliases only.
3. Components and functions are arrow functions with named exports.
4. Every project is organized feature-first (rails between API and web).
5. Let the UI library own styling. No `className`/`style` overrides.
6. Server state goes through TanStack Query; every rule in
   `rules/API-CONTRACT.MD` and `rules/ORPC.MD` is mandatory for
   end-to-end type safety.