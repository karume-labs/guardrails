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
- **No fallbacks or casts on already-typed values.** With end-to-end type
  safety, contract-typed data and validated env need neither `??`/`||` nor
  `as`. See `rules/FALLBACKS.MD`.

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

- **No pure barrel files.** A file whose only purpose is importing things
  from other modules and re-exporting them (e.g. an `index.ts` that is just
  `export { … } from "./foo"`) is banned. Every file must contain real logic,
  types, or declarations - not merely act as an import relay.

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

## Banned Elements

- **Emojis**: Do not use emojis in the UI or codebase unless explicitly requested by the user.
- **Gradients**: Avoid using gradients in styling unless specifically asked to use a gradient.
- **Historical Code Comments**: Do not write code comments that describe the history of changes, refactoring processes, or previous instructions (e.g., `// moved from utils.ts`, `// during the auth refactor`, `// Lauren said you should never do this`). Code comments should only explain complex code logic.

## Always finish with verification

When done editing, run the project's lint, format, and typecheck commands
(e.g. `bun lint`, `bun format`, `bun typecheck`). Never leave the tree
unverified.