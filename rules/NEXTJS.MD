# Next.js Rule

App Router conventions for the web app.

## Routing

- Use the App Router: `app/` with `page.tsx`, `layout.tsx`, `loading.tsx`,
  `error.tsx`, `not-found.tsx`.
- Route groups `(marketing)`, `(dashboard)`, `(admin)` group layouts without
  adding URL segments.
- Route files (`page.tsx`, `layout.tsx`) are **Server Components** and
  `export default` (framework requirement).
- Dynamic routes: `[id]` for required, `[[...slug]]` for optional
  catch-alls. Reserve the app-router metadata everywhere.

## Data and rendering

- Prefer Server Components for the initial payload: fetch inside the page and
  pass primitives/serialized props down to client children.
- Mark any page that renders personalized/dynamic data as `force-dynamic` or
  use `revalidate`/`fetch` cache options explicitly. Never rely on implicit
  caching.
- All client-side data fetching goes through TanStack Query via the type-safe
  API client. Raw `fetch()` in components is banned (see
  `rules/API-CONTRACT.MD`).

## Links and images

- Use `next/link` for navigation and `next/image` for images (or the
  equivalent in the configured UI kit). Never bare `<a href>` for internal
  routes or raw `<img>` for optimized assets.
- The ORPC/API mount route is the only place that should leak HTTP concerns:
  `app/api/server/[[...slug]]/route.ts` mirrors ORPC procedures.

## Metadata and SEO

- Every page exports `generateMetadata` (or `metadata`). SEO rules and the
  reusable SEO component live in `rules/SEO.MD`.
- Use the `Metadata` API, never hand-roll `<head>` tags.

## Client/server file boundaries

- Keep `"use client"` at the top of the file, `"use server"` for server
  actions. Exception: never `"use client"` on route files (handled in
  ReactRule).

## Layouts & providers

- App providers (TanStack Provider, nuqs adapter) belong in the root layout or
  a Providers component — wrap only the subtree that consumes them.
- Auth-guarded sections (`/admin`) wrap their UI in a layout that runs the
  guard server-side before rendering.

## Env and config

- Centralized env parsing only (`apps/web/src/features/shared/env` or
  equivalent). No `process.env.NEXT_PUBLIC_*` inline.
- tsconfig path aliases: `@web/*` (or `@/*`) and `@api/*`. Relative imports
  are forbidden.