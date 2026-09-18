# API Contract Rule

End-to-end type safety is mandatory. The contract (input schemas + inferred
return types) lives once - shared between server and client - so you never
hand-write a client type that duplicates the server.

Two stacks are in use. **They have different folder structures. Pick one per
project and follow its dedicated rule file**:

| Stack          | Rule file                    | Client                     | Use case                  |
| :------------- | :--------------------------- | :------------------------- | :------------------------ |
| oRPC           | `rules/ORPC.MD`              | `@orpc/*` + React Query    | Type-safe RPC embedded in Next.js |
| Elysia + Eden  | `rules/ELYSIA-EDEN.MD`       | `@elysiajs/eden` `treaty`  | Standalone Bun API, Caddy proxy |

## Shared principles (both stacks)

1. **One source of truth for types.** Server input validators (Zod or TypeBox)
   are the single source for request/response shapes; client code consumes
   inferred types. Never redeclare a DTO that the server owns.
2. **No raw `fetch()`.** All data fetching goes through the typed client.
3. **Derive types, don't duplicate.** `z.infer<typeof schema>` /
   `Static<typeof schema>` / Drizzle `$inferSelect` produce the types.
4. **Stable error envelope.** Failures return
   `{ success: false, code, message }`; client transport errors are normalized
   into the same shape so callers do `if (res.error)` once.
5. **Gate admin routes at the boundary** via shared auth middleware/permission
   check, not scattered in components.

## Error handling

- Custom error classes carry `status` + `code` (`BadRequestError` 400,
  `UnauthorizedError` 401, `ForbiddenError` 403, `NotFoundError` 404,
  `TooManyRequestsError` 429).
- A global `onError` handler maps class -> HTTP status + envelope. Strip
  validation internals; in production return a fixed generic message.
- Client wrapper converts network failures to
  `{ code: "CLIENT_NETWORK_ERROR", message: "Unable to connect..." }` so
  components always see the same envelope.

See `examples/ERROR-HANDLING.MD` for the canonical wiring.

## Post-declaration

After defining routes/procedures, run the project's `lint`, `format`, and
`typecheck` to prove the contract type-checks end to end.