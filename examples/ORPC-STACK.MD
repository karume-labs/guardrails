# oRPC Stack Example

Snippets adapted from `acme-stack`. Copy these shapes for a
Next.js + oRPC + TanStack Query project.

## 1. Server entry

```ts
// src/orpc/server.ts
import { os } from "@orpc/server";
export const osServer = os;
```

## 2. Typed input schemas

```ts
// src/features/games/types/index.ts
import { z } from "zod";

export const listGamesInputSchema = z
  .object({
    q: z.string().optional(),
    genre: z.string().optional(),
    serviceType: z.enum(["bundle", "airtime"]).optional(),
    cursor: z.number().int().min(0).default(0),
    limit: z.number().int().min(1).max(100).default(20),
  })
  .optional();

export type ListGamesInput = z.infer<typeof listGamesInputSchema>;
```

## 3. Feature router

```ts
// src/features/games/server/router.ts
import { osServer } from "@/orpc/server";
import { listGamesInputSchema } from "@/features/games/types";

export const gamesRouter = osServer.router({
  list: osServer
    .route({})
    .input(listGamesInputSchema)
    .handler(async ({ input }) => {
      const cursor = input?.cursor ?? 0;
      const limit = input?.limit ?? 20;
      // fetch rows with .limit(limit + 1) for next-cursor detection
      return { items: rows, nextCursor };
    }),
});
```

## 4. Root router

```ts
// src/orpc/root-router.ts
import { gamesRouter } from "@/features/games/server/router";
import { osServer } from "@/orpc/server";

export const rootRouter = osServer.router({
  health: osServer.route({}).handler(() => ({ status: "ok" })),
  games: gamesRouter,
});
export type RootRouter = typeof rootRouter;
```

## 5. Mount route (Next.js)

```ts
// src/app/api/server/[[...slug]]/route.ts
import { RPCHandler } from "@orpc/server/fetch";
import { rootRouter } from "@/orpc/root-router";

const rpcHandler = new RPCHandler(rootRouter);
export const dynamic = "force-dynamic";
const MOUNT_PATH = "/api/server";

const handle = async (req: Request) => {
  const url = new URL(req.url);
  if (!url.pathname.startsWith(MOUNT_PATH)) {
    return new Response("Not found", { status: 404 });
  }
  url.pathname = url.pathname.slice(MOUNT_PATH.length) || "/";
  const modifiedReq = new Request(url, {
    method: req.method,
    headers: req.headers,
    body: req.method === "GET" || req.method === "HEAD" ? undefined : req.body,
    duplex: "half",
  } as RequestInit & { duplex: "half" });
  const result = await rpcHandler.handle(modifiedReq);
  return result.matched ? result.response : new Response("Not found", { status: 404 });
};

export const GET = handle;
export const POST = handle;
```

## 6. Client + React Query utils

```ts
// src/shared/lib/orpc-client.ts
import { createORPCClient } from "@orpc/client";
import { RPCLink } from "@orpc/client/fetch";
import { createORPCReactQueryUtils } from "@orpc/react-query";
import type { RouterClient } from "@orpc/server";
import type { RootRouter } from "@/orpc/root-router";
import { env } from "@/shared/env";

const baseURL = env.NEXT_PUBLIC_API_BASE_URL || "http://localhost:3000";
const rpcLink = new RPCLink({ url: `${baseURL}/api/server` });

type Client = RouterClient<RootRouter>;

export const orpcClient = createORPCClient<Client>(rpcLink);
export const orpc = createORPCReactQueryUtils(orpcClient);