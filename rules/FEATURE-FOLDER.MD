# Feature Folder Rule

Every project is organized **feature-first**. A feature is a domain slice
(auth, products, reviews, admin, cart, ...) that owns all of its code, not a
technical concern.

## Structure

Backend and frontend feature folders **mirror each other** by name:

```
apps/api/src/features/
  auth/
    router/
    schema/
    services/
    types/
  products/
    router/
    schema/
    services/
    types/
    seed/

apps/web/src/features/
  auth/
    components/
    services/        # useQuery / useMutation hooks
    types/
    keys/            # tanstack query keys
  products/
    components/
    services/
    types/
    keys/
```

## Organization rules

- **Bundled by responsibility, not role.** `services/`, `schema/`, `router/`
  live inside the feature, colocated with the domain.
- **No dot-notation filenames.** `auth.router.ts` is banned. Put role files in
  subdirectories: `auth/router/index.ts` or `auth/routers/index.ts`.
- **Seeding lives in the db package**, not in app code:
  `packages/db/src/seed/[feature]/seed/index.ts`.
- **Shared primitives** that cross features go in `shared/` (e.g.
  `shared/lib/`, `shared/components/ui/`). Their names are still
  feature-agnostic.

## Rails between API and web

- A page in `apps/web` talking to `apps/api` should have a matching feature
  name on both sides so an agent can jump between them with zero guesswork.
- API feature types (Zod schemas) are the contract; web features import the
  generated types, never redefine them.

## Naming

- Use kebab-case for folder names (`feature-folder`), PascalCase for component
  files, camelCase for hooks/utilities/services.
- Folder names are singular (`cart/`, `user/`) unless inherently plural.

## When a feature grows

- Split sub-routers per resource (`admin/games-router.ts`) instead of a single
  giant `admin/router.ts`.
- Split component files into `components/` subfolders when the top-level file
  would exceed 150 lines.