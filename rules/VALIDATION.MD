# Validation Rule

Schemas are the **single source of truth** for API contracts. Which library
you use depends on the stack:

| Stack                        | Validator | Why                                  |
| :--------------------------- | :-------- | :----------------------------------- |
| Elysia API (standalone Bun)  | **TypeBox** | Ships with Elysia; built-in `t` types + Value utilities |
| oRPC (Next.js) / frontend /  | **Zod**   | First-class oRPC + React wrapper support |
| frontend shared schemas      |           | `z.infer` powers client types       |

**Rule:** zod only when NOT using Elysia; TypeBox otherwise (for the server).

## TypeBox (Elysia routes)

- Define schemas in `features/<feature>/types` (or the shared validators
  package), compose via `Type.Object`.
- Elysia validates `query`, `params`, `body`, `response` automatically when
  you pass the schema to the route config.
- Derive static types with `Static<typeof schema>`:
  `export type Product = Static<typeof productSchema>`.

```ts
// @shared/validators/products
import { Type } from "elysia";
export const productListQuery = Type.Object({
  page: Type.Optional(Type.Integer({ minimum: 1 })),
  limit: Type.Optional(Type.Number({ default: 20 })),
});
```

## Zod (oRPC + frontend/db)

- Define input schemas in `features/<feature>/types`.
- oRPC `.input(schema)` validates automatically; `z.infer<typeof schema>`
  provides the handler and client types.
- Reuse the same schema package between `apps/api` and `apps/web` so the
  contract is literally one definition, not a mirror.

```ts
// src/features/games/types/index.ts
import { z } from "zod";
export const leaderboardInputSchema = z.object({ gameId: z.string().uuid() });
export type LeaderboardInput = z.infer<typeof leaderboardInputSchema>;
```

## Rules

1. Never inline ad-hoc validators in handler bodies — schemas live in
   `types/` (or a shared `validators` package) and are reused on both sides.
2. Prefer derived types (`Static`/`z.infer`) over hand-written interfaces to
   avoid drift.
3. Strip validation internals from error responses in production (TypeBox
   `Value.Errors`, failed payloads) — do not leak schema structure. Envelope
   shape in `examples/ERROR-HANDLING.MD`.
4. Booleans: Elysia `t.Boolean()` / Zod `z.boolean()` only — never accept
   `"true"`/`"false"` strings.
5. Keep numeric ranges explicit (`minimum`/`maximum`); default rather than
   force callers to pass every field.