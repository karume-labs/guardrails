# Elysia + Eden Rule

Folder structure derived from the `acme` monorepo. Use this stack
when the API runs as a standalone Bun server behind a proxy.

## Directory layout

```
apps/api/src/
  index.ts                # new Elysia(), .use(featureRouters), .listen()
  env/index.ts            # parse env for the server
  features/<feature>/
    routers/index.ts      # Elysia instance with prefix
    routers/<resource>.ts # per-resource sub-routers
    services/             # business logic used by routers
    schema/               # drizzle ORM schema
    types/                # zod/typebox validators (the contract)
    middleware/           # shared guard middleware
  features/shared/
    errors/index.ts       # typed error classes + Elysia onError plugin
    utils/                # logger, storage, helpers
    env/index.ts
  app.ts                  # optional app assembly export (type App)
apps/web/src/features/shared/api/index.ts   # treaty<App> client
packages/db/src/seed/<feature>/seed/index.ts
```

## Router composition

Feature routers are Elysia instances with a `prefix`. Aggregate in
the server entry via `.use(router)`. Export `type App = typeof app`
for the client.

```ts
// apps/api/src/index.ts
export const app = new Elysia()
  .use(errorHandlerPlugin)
  .use(cors({ origin: env.FRONTEND_URL, credentials: true }))
  .use(productsRouter)
  .use(reviewsRouter)
  .use(cartRouter)
  .get("/health", () => ({ status: "ok" }))
  .listen(env.PORT);

export type App = typeof app;
```

- Public reads live directly on `app` (health, settings).
- Admin routes live under feature routers gated by middleware.
- Each route validates `query`, `body`, `params`, and `response` with
  the shared validator schemas.

## Client

- The web app instantiates `treaty<App>(url, { fetcher })` once in
  `features/shared/api`.
- The custom `fetcher` adds `credentials: "include"`, stringifies
  JSON bodies, and normalizes HTTP/network failures into a stable
  `{ success: false, code, message }` envelope.
- Components call `api.<feature>.<method>();` and handle
  `if (res.error)` first.

## Error handling

- Typed error classes (`BadRequestError`, `NotFoundError`, etc.) with
  `status` + `code`.
- Register them via a plugin `.error({ ... })` + `.onError(...)`
  handler. In production strip internals; return a generic message.
- See `examples/ERROR-HANDLING.MD` for the canonical wiring.

## Sub-routers

Split per-resource into `routers/<resource>.ts` (`games-router.ts`,
`users-router.ts`) and aggregate in `routers/index.ts`. This keeps
each file under 150 lines.

## Response contracts

- Declare `response` per route where the shape differs from the body
  (e.g. upload responses, 404s).
- Return `{ success: false, message }` on application errors inside
  the handler with `set.status = 404` where appropriate.