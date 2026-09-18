# Tooling Rule

One toolchain per project, enforced by lint + format + typecheck before any
commit lands.

## Toolchain (across the org)

| Tool                | Role                              | Command                    |
| :------------------ | :-------------------------------- | :------------------------- |
| Biome               | Lint + format (+ organize imports) | `bunx biome check --write` |
| TypeScript (`tsc`)  | Typecheck (strict)                | `bun typecheck`            |
| Lefthook            | Git hooks (pre-commit)            | auto on commit             |
| Bun                 | Package manager + runner          | `bun install`, `bun run`   |

## Biome configuration

- Enable `vcs` + `useIgnoreFile` so gitignore is respected.
- Enable framework domains (`next`, `react`) and strict rules:
  - `noUnusedImports: error`, `useImportType: error`.
  - Spell out `lineWidth` (80), `indentStyle` (space), `indentWidth` (2).
- Organize imports with the biome formatter so import order is stable.

## Commands per package

Every package carries the same three scripts in `package.json`:

```jsonc
{
  "scripts": {
    "lint": "biome check",
    "format": "biome format --write",
    "typecheck": "tsc --noEmit"
  }
}
```

Root `package.json` exposes `bun lint`, `bun format`, `bun typecheck`
that fan out to all workspaces.

## Pre-commit (Lefthook)

```yaml
# lefthook.yml
pre-commit:
  commands:
    check:
      glob: "*.{js,ts,cjs,mjs,d.cts,d.mts,jsx,tsx,json,jsonc}"
      run: bunx biome check --write --no-errors-on-unmatched --files-ignore-unknown=true {staged_files}
      stage_fixed: true
    typecheck:
      run: bun typecheck
```

- `stage_fixed: true` re-stages files biome rewrites - never commit an
  unformatted tree.
- `bun typecheck` runs over the whole workspace, so keep it fast (~a few
  seconds) or scope it per commit.

## Post-task verification

After finishing a task:

1. `bun format` - fix formatting.
2. `bun lint` - fix lint issues (imports, types).
3. `bun typecheck` - prove the type graph is sound.

This is mandatory before declaring work done; the pre-commit hook is a
backstop, not the primary gate.

## CI

- The deploy workflow runs `bun install`, `bun lint`, `bun typecheck`,
  `bun build` before ship. See `.github/workflows/deploy.yml`.
- Failing lint/typecheck blocks deploy - never bypass with `--no-verify`.