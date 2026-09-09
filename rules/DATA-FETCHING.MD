# Data Fetching Rule

All server state goes through **TanStack Query**. Every request between the
frontend and backend goes through the **type-safe API client** (oRPC or Eden)
— raw `fetch()` in components is banned.

## Query keys

Centralized per feature in `keys/` (or `keys.ts`), with `as const` for
exhaustive type safety. Never inline query keys in hooks.

```ts
// features/games/keys/index.ts
export const gamesQueryKeys = {
  all: ["games"] as const,
  list: (genre: string, q: string, serviceType?: "bundle" | "airtime") =>
    ["games", "list", genre, q, serviceType] as const,
  leaderboard: (gameId: string) => ["games", "leaderboard", gameId] as const,
};
```

- Key shape mirrors the query params (pagination, search, filters) so
  invalidations are precise.
- `all` seeds invalidation: `invalidateQueries({ queryKey: gamesQueryKeys.all })`
  invalidates every game query; a specific key invalidates one slice.

## Colocation

Query hooks live in `features/<feature>/services/queries.ts`, mutations in
`services/mutations.ts`, both `"use client"`. The typed client import
depends on your stack — see `rules/ORPC.MD` (oRPC `orpcClient`) or
`rules/ELYSIA-EDEN.MD` (Eden `api`).

```ts
// queries.ts
"use client";
import { useQuery } from "@tanstack/react-query";
import { gamesQueryKeys } from "@/features/games/keys";
import { orpcClient } from "@/shared/lib/orpc-client";

export const useGameLeaderboard = (gameId: string) =>
  useQuery({
    queryKey: gamesQueryKeys.leaderboard(gameId),
    queryFn: () => orpcClient.games.leaderboard({ gameId }),
    enabled: !!gameId,
  });
```

## Mutations

- `mutationFn` calls the type-safe client; call params are typed by the
  contract.
- On success, invalidate the affected keys. Toast errors through the shared
  `toastError` helper.

```ts
// mutations.ts
"use client";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { gamesQueryKeys } from "@/features/games/keys";
import { orpcClient } from "@/shared/lib/orpc-client";

export const useRecordGameSession = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (input: { gameId: string; durationSeconds: number }) =>
      orpcClient.games.recordSession(input),
    onSuccess: () =>
      queryClient.invalidateQueries({ queryKey: gamesQueryKeys.all }),
  });
};
```

## Infinite queries

- Use `useInfiniteQuery` for cursors/scroll feeds.
- `getNextPageParam: (lastPage) => lastPage.nextCursor`; feed `nextCursor`
  back as `pageParam` in `queryFn`.
- Flatten pages into a derived value, never mutate query data:

```ts
export const useGamesFlat = (genre: string, q: string) => {
  const query = useGames(genre, q);
  const items = query.data?.pages.flatMap((p) => p.items) ?? [];
  return { ...query, items };
};
```

## Error toasts

- Wrap error surfacing in a reusable `useQueryErrorToast` hook that fires once
  per error transition, not per render.
- Surface a stable user-facing message; never dump raw API errors.

## Conventions

- `queryFn` returns `res.data` and throws (or checks `res.error`) so React
  Query can manage state. Never return the wrapper {
  error, data } as data.
- Set `enabled: false` for queries that need a precondition (auth, selected
  id).
- Use `staleTime`/`refetchInterval` deliberately; default to sensible values
  (e.g. 5m stale for catalog, short refetch for live leaderboards).