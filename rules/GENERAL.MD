# General Rule

Baseline language and code-organization conventions that apply everywhere
(Next.js apps, API packages, shared packages).

## Functions

- Exclusively use arrow functions instead of `function` declarations.
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

- Strict mode always. Maintain strong typing instead of using `any`, `@ts-ignore`, or `as unknown` casts.
- Prefer `interface` for object shapes (prop types, DTOs) and `type` for
  unions, intersections, and mapped types.
- Derive types from Zod / TypeBox / Drizzle schemas instead of hand-maintaining
  duplicates (see `rules/API-CONTRACT.MD` and `rules/VALIDATION.MD`).
- **Trust already-typed values natively.** With end-to-end type
  safety, contract-typed data and validated env need neither `??`/`||` nor
  `as`. See `rules/FALLBACKS.MD`.

## Imports

- **Exclusively use absolute path aliases for imports.**
- Sort imports: external packages first, then internal aliases, then types.
- Import using absolute path aliases instead of walking up `../../` folders.

```ts
// Good
import { useQuery } from "@tanstack/react-query";
import { gamesQueryKeys } from "@/features/games/keys";
import { orpcClient } from "@/shared/lib/orpc-client";

// Bad
import { gamesQueryKeys } from "./../../../../features/games/keys";
```

- **Ensure files contain real logic.** A file whose only purpose is importing things
  from other modules and re-exporting them (e.g. an `index.ts` that is just
  `export { … } from "./foo"`) should be consolidated. Every file must contain real logic,
  types, or declarations - not merely act as an import relay.

## File size limit

- **Keep file size under ~150 lines.** Hard requirement.
- When a file approaches the limit, split by responsibility:
  - a router into per-resource routers,
  - a component into subcomponents,
  - a types file into `types/` with focused slices.
- Maintain proper formatting and readability when splitting files.

## Feature-based organization

- Code is grouped by feature, not by technical layer. Rails between the API
  and web apps mirror each other. See `rules/FEATURE-FOLDER.MD`.
- File names maintain standard folder separation for roles (use `auth/server/router.ts` instead of `auth.router.ts`).

## Environment variables

- Centralize env parsing per package in an `env/index.ts`. Read from the parsed
  `env` object instead of reading `process.env` inline. See `rules/ENV-VARS.MD`.

## Element Guidelines

- **Emojis**: Use standard UI elements instead of emojis unless explicitly requested.
- **Gradients**: Use solid colors instead of gradients in styling unless specifically requested.
- **Code Comments**: Focus comments exclusively on explaining complex logic rather than historical changes (e.g., omit `// moved from utils.ts`, `// during the auth refactor`).

## Dependencies and node_modules

- **Never edit code inside `node_modules` directly.** Any direct edits will be lost on the next reinstall. If a package needs modification, use a patch tool (like `patch-package` or `pnpm patch`), find an alternative package, or implement a local workaround.

## Always finish with verification

When done editing, run the project's lint, format, and typecheck commands
(e.g. `bun lint`, `bun format`, `bun typecheck`). Ensure the tree is always
verified before committing.