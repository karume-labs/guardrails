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
| `rules/FEATURE-FOLDER.MD`                   | Feature-based folder architecture            |
| `rules/REACT.MD`                            | Components, props, server/client split       |
| `rules/NEXTJS.MD`                           | App Router, routing, metadata                |
| `rules/STYLING.MD`                          | Mantine + shadcn, no style overrides         |
| `rules/DATA-FETCHING.MD`                    | TanStack Query, keys, service colocation     |
| `rules/API-CONTRACT.MD`                     | oRPC + Elysia Eden end-to-end type safety    |
| `rules/STATE-MANAGEMENT.MD`                 | nuqs, URL-driven state                       |
| `rules/DATA-TABLES.MD`                      | TanStack Table + nuqs data grids             |
| `rules/SEO.MD`                              | SEO config component, OG metadata            |

## Examples index

| File                                  | Copy-ready pattern                              |
| :------------------------------------ | :---------------------------------------------- |
| `examples/ORPC-STACK.MD`              | Full oRPC server -> client -> React-Query stack |
| `examples/TANSTACK-QUERY.MD`          | keys.ts + queries.ts + mutations.ts             |
| `examples/DATA-TABLE.MD`              | TanStack Table columns + view with nuqs         |
| `examples/SEO-METADATA.MD`            | SEO component and generateMetadata              |

## Hard rules (non-negotiable)

1. No file exceeds `~150` lines. Split on growth.
2. No relative imports. Absolute path aliases only.
3. Components and functions are arrow functions with named exports.
4. Every project is organized feature-first (rails between API and web).
5. Let the UI library own styling. No `className`/`style` overrides.
6. Server state goes through TanStack Query; every rule in `rules/API-CONTRACT.MD`
   is mandatory for end-to-end type safety.