# Auth Rule

Authentication and authorization live at the **API boundary**, not in
components. Two stacks: Elysia middleware and oRPC server guards
Session/auth setup uses Better Auth or the platform's native
auth; never hand-roll sessions.

## Auth setup

- Auth config lives in `features/auth/services/config.ts` (Elysia) or
  `features/auth/server/...` (oRPC). Instantiate once, export the instance.
- Route all auth hooks through it — never re-create auth clients in
  handlers or components.

```ts
// Elysia: apps/api/src/features/auth/services/config.ts
export const auth = betterAuth({ /* ...config */ });

// oRPC: src/features/auth/server/better-auth.ts
export { auth };
```

## Authorization middleware

Guard **admin routers** (and any protected route) with a shared middleware
that (1) resolves the session from request headers, (2) checks role +
verification state, (3) throws before the handler runs.

```ts
// Elysia: features/admin/middleware
import { ForbiddenError } from "@api/features/shared/errors";

export const isAdmin = new Elysia().derive(async ({ request }) => {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session || session.user.role !== "admin" || !session.user.emailVerified) {
    throw new ForbiddenError("Action forbidden or email not verified");
  }
  return { session };
});
```

```ts
// oRPC: src/features/admin/server/require-admin.ts
export const requireAdmin = async () => {
  const session = await auth.api.getSession({ /* headers */ });
  if (!session?.user) throw new Error("Unauthorized");
  if (session.user.role !== "admin") throw new Error("Forbidden");
  return session.user;
};

export const runAsAdmin = async <T>(fn: () => Promise<T>): Promise<T> => {
  await requireAdmin();
  return fn();
};
```

## Application

- **Elysia:** chain `.use(isAdmin)` before the protected CRUD routes. Place
  the guard after the public routes of the router.
- **oRPC:** wrap handler bodies with `runAsAdmin(async () => { ... })`, or
  reuse the guard in shared sub-routers.
- **Never** gate in the web UI alone — always enforce server-side.

## Error mapping

- Throw typed errors (`UnauthorizedError`, `ForbiddenError`); the global
  `onError` plugin maps them to 401/403 envelopes. See
  `examples/ERROR-HANDLING.MD`.
- Do not leak whether a user exists vs. has the wrong role — return a
  generic 401/403 message.

## Web side

- Client components check the authenticated session for UI only (show/hide
  buttons); the API remains the authority.
- Protected pages redirect server-side when the session is missing.