# Elysia + Eden Example

Snippets adapted from `acme`. Copy these shapes for a standalone
Bun API + Next.js web app backed by Elysia + Eden Treaty.

## 1. Feature router

```ts
// apps/api/src/features/products/routers/index.ts
import { listProducts } from "@api/features/products/services";
import {
  createProductBody,
  productListQuery,
} from "@shared/validators/products";
import { Elysia, t } from "elysia";

export const productsRouter = new Elysia({ prefix: "/products" })
  .get("/", async ({ query }) => listProducts(query), {
    query: productListQuery,
  })
  .get("/:slug", async ({ params: { slug } }) => getProductBySlug(slug), {
    params: t.Object({ slug: t.String() }),
  })
  // guarded section: middleware on the shared chain
  .use(isAdmin)
  .post("/", async ({ body }) => createProduct(body as Record<string, unknown>), {
    body: createProductBody,
  });
```

## 2. App assembly

```ts
// apps/api/src/index.ts
import { productsRouter } from "@api/features/products/routers";
import { env } from "@api/features/shared/env";
import { errorHandlerPlugin } from "@api/features/shared/errors";
import { cors } from "@elysiajs/cors";
import { Elysia } from "elysia";

export const app = new Elysia()
  .use(errorHandlerPlugin)
  .use(cors({ origin: env.FRONTEND_URL, credentials: true }))
  .use(productsRouter)
  .get("/health", () => ({ status: "ok" }))
  .listen(env.PORT);

export type App = typeof app;
```

## 3. Eden client with normalized errors

```ts
// apps/web/src/features/shared/api/index.ts
import { treaty } from "@elysiajs/eden";
import type { App } from "@api/root";
import { env } from "@web/features/shared/env";

const client = treaty<App>(env.NEXT_PUBLIC_API_URL, {
  fetcher: async (url, options) => {
    options = { ...options, credentials: "include" };
    if (options.body && typeof options.body === "object") {
      options.body = JSON.stringify(options.body);
      options.headers = { ...options.headers, "Content-Type": "application/json" };
    }
    const response = await fetch(url, options);
    if (!response.ok) {
      const errorData = await response.json().catch(() => null);
      return Response.json(
        {
          success: false,
          error: {
            code: errorData?.error?.code || "UNKNOWN_ERROR",
            message: errorData?.error?.message || "Request failed.",
          },
        },
        { status: response.status },
      );
    }
    return response;
  },
});

export const api = client;
```

## 4. Consuming with TanStack Query

```ts
// apps/web/src/features/admin/services/queries.ts
"use client";
import { useQuery } from "@tanstack/react-query";
import { api } from "@web/features/shared/api";
import { queryKeys } from "@web/features/shared/keys";

export const useAdminMetrics = () =>
  useQuery({
    queryKey: queryKeys.admin.metrics,
    queryFn: async () => {
      const res = await api.admin.metrics.get();
      if (res.error) throw res.error;
      return res.data;
    },
  });
```