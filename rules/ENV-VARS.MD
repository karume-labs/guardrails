# Environment Variables Rule

Environment variables are parsed **once per package**, validated on module
load, and exported as a typed `env` object. Confine raw `process.env` access exclusively to
these files.

## Layout

One env module per package (or shared wherever the package is consumed):

```
apps/api/src/features/shared/env/index.ts    # server env
apps/web/src/features/shared/env/index.ts    # client env (NEXT_PUBLIC_*)
packages/db/src/env/index.ts                 # db env
```

## Validation

Each package defines a schema and validates on boot:

- **Server / shared (no Elysia):** zod, parsed once.

```ts
// apps/web/src/features/shared/env/index.ts
import { z } from "zod";

export const env = z.object({
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "production", "test"]),
}).parse(process.env);
```

- **Elysia backend / db:** TypeBox, with `Value.Check` + `Value.Convert`,
  hard-failing on invalid values (see `rules/VALIDATION.MD`).

```ts
// packages/db/src/env, apps/api/src/env
import { Type, Value } from "@sinclair/typebox";

const envSchema = Type.Object({
  DATABASE_URL: Type.String({ pattern: "^(postgres://|postgresql://)" }),
  PORT: Type.Number({ default: 4000 }),
});
const data = { ...process.env };
Value.Default(envSchema, data);
Value.Convert(envSchema, data);
if (!Value.Check(envSchema, data)) {
  console.error("Invalid environment variables");
  process.exit(1);
}
export const env = data as Static<typeof envSchema>;
```

## Loading

- Use `bun --cwd <package> ...` so each env file can resolve its own
  `.env` (dotenv `config({ path: resolve(__dirname, "../../.env") })` when
  needed).
- `.env.example` documents every key the schema expects.

## Rules

1. **`process.env` is restricted to env modules only** (`env/index.ts` and
   their direct validators). Restrict `process.env` reads exclusively to env modules. Everywhere else, import the validated
   `env` object and read from it.
2. Default only harmless values (`PORT`, `NODE_ENV`) and omit defaults for secrets.
3. Prefix client-exposed vars with `NEXT_PUBLIC_` and ensure they contain no secrets.
4. `NODE_ENV` drives prod-only behavior (error leaking, logging) - read it
   from `env`, rather than from `process.env` at call sites.
5. Env values are validated and non-null by type - rely on their guaranteed presence rather than adding a `|| "..."` /
   `?? "..."` fallback at the call site (see `rules/FALLBACKS.MD`).