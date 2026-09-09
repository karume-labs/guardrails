# Fallbacks Rule

Strict discipline around `||`/`??` fallbacks and type casts. With end-to-end
type safety, re-validated data, and schema defaults, most "just in case"
fallbacks and casts are **unnecessary noise** — and the agent habit of adding
them everywhere is forbidden.

## Core principle

If a value is guaranteed by the type system (validated env, API contract
response, schema with a default), **do not add a fallback or a cast**. Trust
the type. Adding `?? ""`, `|| {}`, or `as Something` there hides real bugs and
dilutes the contract.

## Trust validated env

Env is parsed once through a schema with defaults and hard-fails on invalid
values. Consumers get a fully-typed, non-null `env`.

```ts
// Bad — env is already validated; the fallback is dead code
const baseURL = env.NEXT_PUBLIC_API_URL || "http://localhost:3000";

// Good — use the validated value directly
export const orpcLink = new RPCLink({ url: `${env.NEXT_PUBLIC_API_URL}/api/server` });
```

If a var is genuinely optional, mark it in the schema (`Type.Optional`,
`z.optional`) and consume it with the schema-provided default or `env.X` —
not with an inline `|| "hardcoded"`.

## Trust the API contract

Responses are typed end-to-end (oRPC/eden). Handler outputs and query results
arrive typed — never re-cast or re-fallback them.

```ts
// Bad — data is typed; the cast and fallback are noise
const res = await api.admin.metrics.get();
return res.data as unknown as AdminStats;

// Good — the contract delivers the type; surface errors via res.error
const res = await api.admin.metrics.get();
if (res.error) throw res.error;
return res.data;
```

## Banned patterns

- `x || fallback` / `x ?? fallback` where `x` is non-nullable by its type
  (validated env, `.notNull()` columns, contract-typed responses).
- `as` casts on already-contract-typed values. Includes
  `as unknown as T`, `as Record<string, unknown>` for validated bodies, and
  double casts.
- `as any` in any form. If a cast feels needed, the **schema/contract is
  wrong** — fix the source, don't paper over it.

## Allowed fallbacks

Fallbacks are fine where the value is **genuinely absent**, never where the
type already guarantees it:

- Optional-access before first load: `data?.pages.flatMap(...) ?? []` (query
  data may be undefined) — legitimate, keep it.
- Optional API fields that the schema marks optional.
- Local UI state initialization.
- Query params with `.withDefault()` in nuqs (the sanctioned default
  mechanism) — not `?? ""` at the read site.

## Defaults live in the schema

Prefer schema-level defaults over scattered `||` fallbacks:

```ts
// Schema (single source) — default lives here
limit: z.number().int().min(1).max(100).default(20)

// Consumer — no fallback needed
orpcClient.games.list({ limit })   // limit is always a number
```

## Review checklist

Before finishing a change, scan for:

1. `|| ""` / `?? 0` / `?? []` on validated env, `.notNull()` data, or
   contract-typed responses.
2. `as` / `as unknown as` / `as any` on already-typed values.
3. Hand-rolled fallbacks that duplicate a schema default.