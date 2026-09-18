# oRPC Rule

Folder structure for the oRPC codebase.
oRPC embeds RPC procedures directly into Next.js routes.

## Directory layout

```
src/
  orpc/
    server.ts               # export const osServer = os
    root-router.ts          # aggregate feature routers
  features/<feature>/
    server/
      router.ts             # osServer.router({ ... })
      users-router.ts       # per-resource sub-routers
    types/
      index.ts              # zod input schemas + inferred types
    services/
      queries.ts            # useQuery hooks via orpcClient
      mutations.ts          # useMutation hooks via orpcClient
    keys/
      index.ts              # typed query keys
  app/
    api/
      server/
        [[...slug]]/route.ts  # RPCHandler mount (GET + POST)
  shared/
    lib/
      orpc-client.ts        # typed client + react-query utils
```

## Router composition

Feature routers are nested under the root router by name, mirroring
the URL surface. Each router uses `osServer.router({ procName:
osServer.route().input(schema).handler(fn) })`.

```ts
// root-router.ts
import { osServer } from "@/orpc/server";
import { gamesRouter } from "@/features/games/server/router";

export const rootRouter = osServer.router({
  health: osServer.route({}).handler(() => ({ status: "ok" })),
  games: gamesRouter,
  // ... other feature routers
});
export type RootRouter = typeof rootRouter;
```

- The mount route `app/api/server/[[...slug]]/route.ts` uses `RPCHandler`.
- It is framework-agnostic and works inside Next.js App Router.
- The client derives types from `RootRouter` via `RouterClient<RootRouter>`.

## Input schemas

- Put all input Zod schemas in `features/<feature>/types/index.ts`.
- Procedures declare `.input(schema)` - infer with `z.infer`.
- Keep per-resource schemas separate (`listGamesInputSchema`,
  `leaderboardInputSchema`) but co-locate them in the types file.

## Client + React Query

- `shared/lib/orpc-client.ts` creates `createORPCClient` +
  `createORPCReactQueryUtils`.
- `orpc.<feature>.<proc>.queryOptions({ input })` keeps typed
  query-key + function in one call site (preferred over manual
  `queryKey`/`queryFn`).
- For manual control, import `orpcClient` and call
  `orpcClient.<feature>.<proc>(params)` directly.
- Keep hooks in `services/queries.ts` and `services/mutations.ts`.
- Invalidations use the typed `queryKey` from `keys/index.ts`.

## Per-resource sub-routers

When a feature owns many resources, split sub-routers (`games-router.ts`,
`users-router.ts`) and aggregate in `server/router.ts`. This keeps
each file under 150 lines.