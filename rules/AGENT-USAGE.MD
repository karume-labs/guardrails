# Agent Usage Rule

How an AI agent loads and applies these guardrails when working in a project.

## Why this file

Reading correct, complete rules at the start of a task changes the code an
agent produces. This file defines the loading order and the task workflow so
the same behavior happens regardless of which agent (opencode, Cursor, Claude,
Codex, ...) is running.

## 1. Surface the rules into context

Agents only follow rules that are actually in their context. Do **not**
instruct the model to "clone the guardrails repo". Instead, make the content
available:

- **Recommended:** mirror `rules/*.MD` + `examples/*.MD` into the working
  repo under `.agents/` (or `.github/`/docs), and add a pointer in the repo's
  `AGENTS.md`.
- **Synchronized:** git submodule or a `bun`/scripted copy job keeps them in
  sync across projects. Update all workspaces when guardrails change.
- **Strict trade-off:** injecting the full set costs context tokens. On large
  tasks, load only the rules for the files being touched (see load order).

## 2. Load order (before writing code)

1. Read `AGENTS.MD` - the index + hard rules.
2. Read the **always-on** set:
   - `GENERAL.MD` (baseline TS, imports, arrow functions, 150-line cap)
   - `NAMING.MD` (kebab-case, PascalCase components)
   - `FEATURE-FOLDER.MD` (feature-first rails between API and web)
   - `FALLBACKS.MD` (no `??`/`||`/`as` on already-typed values)
   - `TOOLING.MD` and `ENV-VARS.MD` (commands + typed env)
3. Read the **stack-specific** file(s) for the project:
   - Elysia standalone API → `ELYSIA-EDEN.MD` + `VALIDATION.MD` (TypeBox)
   - oRPC + Next.js → `ORPC.MD` + `VALIDATION.MD` (Zod)
   - Mantine UI → `STYLING.MD` (Mantine sections)
   - shadcn UI → `STYLING.MD` (shadcn sections)
4. Read **task-dependent** files when the task touches that concern:
   `REACT.MD`, `NEXTJS.MD`, `DATA-FETCHING.MD`, `STATE-MANAGEMENT.MD`,
   `DATA-TABLES.MD`, `SEO.MD`, `AUTH.MD`, `DRIZZLE.MD`, `SEEDING.MD`,
   `API-CONTRACT.MD`.
5. Mirror shapes from `examples/*.MD` that match the task.

## 3. Task workflow

1. **Survey first.** Read the relevant rules, then the surrounding repo code.
   Match existing conventions instead of guessing.
2. **Apply while writing.**
   - No relative imports; absolute aliases only.
   - Arrow functions + named exports (`export const`).
   - Keep files under 150 lines; split on growth.
   - All data fetching through the typed client + TanStack Query.
   - No `??`/`||`/`as` on validated env or contract-typed data
     (`rules/FALLBACKS.MD`).
   - `process.env` only inside `env/index.ts`; consume the validated `env`
     object everywhere else.
   - Filter/page/sort state lives in nuqs URL params.
   - UI library owns styling - no `className`/`style` overrides.
3. **Verify before finishing.** Run the Tooling checks:
   `bun format`, `bun lint`, `bun typecheck` (exact commands may differ per
   project - check its package.json).
4. **Report honestly.** If a rule could not be followed (framework constraint,
   missing config), say so explicitly rather than silently deviating.

## 4. Precedence

- Project-local `AGENTS.md` / `.agents/RULES.md` > guardrails. The project
  may tighten or override a stacked rule; defaults here are the floor.
- Guardrails > generic assistant behavior. These rules win over "best
  practices" the model invented on its own.
- If two guardrail rules conflict, the more specific one wins (e.g. a
  stack-specific file beats `GENERAL.MD`).

## 5. When the repo evolves

- Rules are split so a 150-line cap applies per file. When adding a rule,
  update `AGENTS.MD` (the index) and `README.MD` (the human summary).
- Re-run the guardrails synchronisation (section 1) after changes so every
  consuming project picks them up.