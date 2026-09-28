# Feature Folder Rule

Every project is organized **feature-first**. A feature is a domain slice
(auth, products, reviews, admin, cart, ...) that owns all of its code, not a
technical concern.

## Structure

Backend and frontend feature folders **mirror each other** by name:

```
apps/api/src/features/
  auth/
    router/          # elysia routers (elysia) or oRPC routers
    schema/          # drizzle schema for the feature
    services/        # business logic used by routers
    types/           # zod/typebox validators (the API contract)
    middleware/

apps/web/src/features/
  auth/
    components/
    services/        # useQuery / useMutation hooks
    types/
    keys/            # tanstack query keys
```

## Organization rules

- **Bundled by responsibility, not role.** `services/`, `schema/`, `router/`
  live inside the feature, colocated with the domain.
- **Consolidate single-file folders.** Collapse a folder into a single file (`constants.ts`) if it would only contain one file.
- **Use standard filenames.** Put role files in
  subdirectories: `auth/router/index.ts` or `auth/routers/index.ts`.
- **Seeding lives exclusively in the db package**, keeping it isolated from app code. See
  `rules/SEEDING.MD` for the seeding pattern: feature seeders in
  `packages/db/src/<feature>/seed/index.ts` composed from a
  `data.json` of building blocks, deterministic seeded-random,
  a root `packages/db/src/seed/index.ts` entry, `--<feature>`
  flags, and `bun --cwd packages/db db:seed` in root scripts.
- **Shared primitives** that cross features go in `shared/` (e.g.
  `shared/lib/`, `shared/components/ui/`). Their names are still
  feature-agnostic.

## Rails between API and web

- A page in `apps/web` talking to `apps/api` should have a matching feature
  name on both sides so an agent can jump between them with zero guesswork.
- API feature types (Zod/Typebox schemas) are the contract; web features
  import the shared validators and their inferred types, relying on them exclusively instead of redefining them.
- Public and admin surface differ: keep public reads lean, and gate admin
  routers behind the auth middleware rather than scattering guards.

## Naming

- Full naming system in `rules/NAMING.MD`. Summary:
  - Folders: kebab-case (`feature-folder`).
  - Files (general): kebab-case (`use-catalog-filters.ts`).
  - Component files: PascalCase (`GamesView.tsx`).
  - Symbols: camelCase functions/hooks, PascalCase components/types.
- Domain files use dot-notation only for database tables and types where
  Drizzle infers them (e.g. `productVariants`); maintain standard naming for role-files without dot-notation.

## When a feature grows

- Split sub-routers per resource (`admin/games-router.ts`) instead of a single
  giant `admin/router.ts`.
- Split component files into `components/` subfolders when the top-level file
  would exceed 150 lines.