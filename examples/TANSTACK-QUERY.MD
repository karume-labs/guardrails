# TanStack Query Example

The complete service-layer pattern: keys + queries + mutations.
Use these with either endpoint stack (oRPC or Eden) by swapping
the client import.

## 1. Typed query keys

```ts
// features/games/keys/index.ts
export const gamesQueryKeys = {
  all: ["games"] as const,
  list: (genre: string, q: string, serviceType?: "bundle" | "airtime") =>
    ["games", "list", genre, q, serviceType] as const,
  leaderboard: (gameId: string) => ["games", "leaderboard", gameId] as const,
  profile: (msisdn: string) => ["games", "profile", msisdn] as const,
};
```

## 2. Finite queries

```ts
// features/games/services/queries.ts
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

## 3. Infinite (cursor) query

```ts
import { useInfiniteQuery } from "@tanstack/react-query";

export const useGames = (genre: string, q: string, serviceType?: "bundle" | "airtime") =>
  useInfiniteQuery({
    queryKey: gamesQueryKeys.list(genre, q, serviceType),
    queryFn: ({ pageParam = 0 }) =>
      orpcClient.games.list({
        genre,
        q: q || undefined,
        serviceType,
        cursor: pageParam,
        limit: 10,
      }),
    initialPageParam: 0,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    staleTime: 1000 * 60 * 5,
  });

export const useGamesFlat = (genre: string, q: string) => {
  const query = useGames(genre, q);
  const items = query.data?.pages.flatMap((p) => p.items) ?? [];
  return { ...query, items };
};
```

## 4. Mutations with invalidation

```ts
// features/games/services/mutations.ts
"use client";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { gamesQueryKeys } from "@/features/games/keys";
import { orpcClient } from "@/shared/lib/orpc-client";

export const useRecordGameSession = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (input: { gameId: string; durationSeconds: number }) =>
      orpcClient.games.recordSession(input),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: gamesQueryKeys.all });
    },
    onError: (error) => toastError("Save failed", error.message),
  });
};
```

## 5. oRPC query-options sugar

Prefer colocating key + fn when the stack exposes `queryOptions`:

```ts
import { useQuery } from "@tanstack/react-query";
import { orpc } from "@/shared/lib/orpc-client";

export const useLeaderboard = (gameName: string) =>
  useQuery({
    ...orpc.safaricom.leaderboard.queryOptions({ input: { gameName } }),
    refetchInterval: 30_000,
  });
```

## 6. One-time error toast

```ts
// shared/lib/useQueryErrorToast.ts
"use client";
import { useEffect, useRef } from "react";

export const useQueryErrorToast = (error: boolean, title: string, description?: string) => {
  const notified = useRef(false);
  useEffect(() => {
    if (error && !notified.current) {
      notified.current = true;
      toastError(title, description ?? "Please check your connection and try again.");
      return;
    }
    if (!error) notified.current = false;
  }, [error, title, description]);
};
```