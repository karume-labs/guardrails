# General Rule

Baseline language and code-organization conventions that apply everywhere
(Next.js apps, API packages, shared packages).

## Functions

- Always use arrow functions. Never `function` declarations.
- Prefer `export const` (named exports) over `export default`. Default exports
  are only allowed where a framework requires them (e.g. `page.tsx`,
  `layout.tsx`, `route.ts`).

```ts
// Good
export const formatSlug = (title: string): string =>
  title.toLowerCase().replace(/\s+/g, "-");

// Bad
export default function formatSlug(title: string) {
  return title.toLowerCase().replace(/\s+/g, "-");
}
```

## TypeScript

- Strict mode always. No `any`, no `@ts-ignore`, no `as unknown` casts.
- Prefer `interface` for object shapes (prop types, DTOs) and `type` for
  unions, intersections, and mapped types.
- Derive types from Zod / TypeBox / Drizzle schemas instead of hand-maintaining
  duplicates (see `rules/API-CONTRACT.MD` and `rules/VALIDATION.MD`).

## Imports

- **Absolute path aliases only. Relative imports are forbidden.**
- Sort imports: external packages first, then internal aliases, then types.
- Never import deeply nested files by walking up `../../` folders.

```ts
// Good
import { useQuery } from "@tanstack/react-query";
import { gamesQueryKeys } from "@/features/games/keys";
import { orpcClient } from "@/shared/lib/orpc-client";

// Bad
import { gamesQueryKeys } from "./../../../../features/games/keys";
```

## File size limit

- **No file exceeds ~150 lines.** Hard requirement.
- When a file approaches the limit, split by responsibility:
  - a router into per-resource routers,
  - a component into subcomponents,
  - a types file into `types/` with focused slices.
- Do not silence the rule by removing newlines or minifying.

## Feature-based organization

- Code is grouped by feature, not by technical layer. Rails between the API
  and web apps mirror each other. See `rules/FEATURE-FOLDER.MD`.
- File names never use dot notation for roles (`auth.router.ts` is banned;
  use `auth/server/router.ts`).

## Environment variables

- Centralize env parsing per package in an `env/index.ts`. Never read
  `process.env` inline. See `rules/ENV-VARS.MD`.

## Always finish with verification

When done editing, run the project's lint, format, and typecheck commands
(e.g. `bun lint`, `bun format`, `bun typecheck`). Never leave the tree
unverified.