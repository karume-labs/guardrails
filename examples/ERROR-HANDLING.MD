# Error Handling Example

Pattern from `acme` (Elysia) and `acme` (Eden client).
Typed error classes flow through a single global handler and a
normalizing client fetcher, so every caller sees one envelope.

## 1. Typed error classes

```ts
// features/shared/errors/index.ts
export class BadRequestError extends Error {
  status = 400;
  code = "BAD_REQUEST";
  constructor(message = "Bad request") {
    super(message);
  }
}

export class UnauthorizedError extends Error {
  status = 401;
  code = "UNAUTHORIZED";
  constructor(message = "Authentication required") {
    super(message);
  }
}

export class ForbiddenError extends Error {
  status = 403;
  code = "FORBIDDEN";
  constructor(message = "Action forbidden") {
    super(message);
  }
}

export class NotFoundError extends Error {
  status = 404;
  code = "NOT_FOUND";
  constructor(message = "Requested resource not found") {
    super(message);
  }
}

export class TooManyRequestsError extends Error {
  status = 429;
  code = "TOO_MANY_REQUESTS";
  constructor(message = "Too many requests. Please slow down.") {
    super(message);
  }
}
```

## 2. Global onError plugin

```ts
// features/shared/errors/index.ts (continued)
import { env } from "@api/env";
import { Elysia } from "elysia";

export const errorHandlerPlugin = new Elysia({ name: "error-handler" })
  .error({
    BAD_REQUEST: BadRequestError,
    UNAUTHORIZED: UnauthorizedError,
    FORBIDDEN: ForbiddenError,
    NOT_FOUND: NotFoundError,
    TOO_MANY_REQUESTS: TooManyRequestsError,
  })
  .onError(({ code, error, set }) => {
    const isProduction = env.NODE_ENV === "production";
    const err = error instanceof Error ? error : null;
    const message = err?.message ?? String(error);

    switch (code) {
      case "BAD_REQUEST":
        set.status = 400;
        return { success: false, code: "BAD_REQUEST", message };
      case "UNAUTHORIZED":
        set.status = 401;
        return { success: false, code: "UNAUTHORIZED", message };
      case "FORBIDDEN":
        set.status = 403;
        return { success: false, code: "FORBIDDEN", message };
      case "NOT_FOUND":
        set.status = 404;
        return { success: false, code: "NOT_FOUND", message };
      case "TOO_MANY_REQUESTS":
        set.status = 429;
        return { success: false, code: "TOO_MANY_REQUESTS", message };
      case "VALIDATION":
        set.status = 422;
        return {
          success: false,
          code: "VALIDATION_ERROR",
          message: "The provided parameters failed validation constraints.",
          errors: isProduction
            ? undefined
            : (error as { all?: Array<{ path: string; message: string }> }).all,
        };
      default:
        set.status = 500;
        return {
          success: false,
          code: "INTERNAL_SERVER_ERROR",
          message: isProduction
            ? "An unexpected error occurred. Please contact support."
            : message,
        };
    }
  });
```

## 3. Throwing from handlers

```ts
// inside a router handler
.handler(async ({ input }) => {
  const user = await findById(input.id);
  if (!user) throw new NotFoundError("User not found");
  return user;
})
```

## 4. Client-side normalization (Eden)

The `fetcher` wraps transport errors into the same envelope — see
`examples/ELYSIA-EDEN.MD` section 3. Callers never catch raw fetch
failures:

```ts
const res = await api.admin.metrics.get();
if (res.error) throw res.error;   // normalized  { code, message }
return res.data;
```

## Notes

- Validation errors are stripped of internals in production to avoid leaking
  schema structure.
- Never throw bare `Error` from services without a `code`; the plugin maps
  unknown errors to 500 automatically.