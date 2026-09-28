# Naming Rule

One naming system across the codebase so agents (and humans) can predict file
locations without searching.

## Folders

- **kebab-case** everywhere: `feature-folder`, `shared/lib`, `routers/`.
- Folder names are singular (`cart/`, `user/`) unless inherently plural
  (`migrations/`).
- **Consolidate single-file folders.** Collapse a folder into a single file if it would only contain one file (e.g., use `constants.ts` instead of `constants/index.ts`).

## Files

- **kebab-case** for all non-component files: `router.ts` ->
  `routers/index.ts`, guards in `can-access-user.ts`, keys in
  `keys/index.ts`.
- Component files keep **PascalCase** (`GamesView.tsx`, `SEOConfig.tsx`)
  so JSX imports read naturally. The exported component must match the
  filename.
- Hooks/utilities/services/hooks files are kebab-case
  (`use-catalog-filters.ts`, `format-slug.ts`).

| Kind                   | Case      | Example                        |
| :--------------------- | :-------- | :----------------------------- |
| Folders                | kebab     | `features/shared/components`   |
| Files (general)        | kebab     | `useQueryErrorToast.ts` -> `use-query-error-toast.ts` |
| Component files        | PascalCase| `GamesView.tsx`                |
| Exported components    | PascalCase| `export const GamesView`       |
| Hooks / utils / services| kebab   | `use-catalog-filters.ts`       |

## Symbols

- **Functions/consts:** camelCase (`formatSlug`, `useAdminMetrics`).
- **Components/types/interfaces:** PascalCase (`StatusBadge`, `AdminGame`).
- **Constants:** UPPER_SNAKE (`MAX_PAGE_SIZE`) for literals, camelCase still
  fine for derived values.
- **Database columns (Drizzle):** snake_case (`createdAt` via helper,
  `category_id` in DB).

## Exports

- Named exports only (`export const`). Default export reserved for Next.js
  route files (`page.tsx`, `layout.tsx`).
- Re-export through feature index files (`features/<f>/index.ts`) to shorten
  import paths; ensure dependency graphs between features remain strictly unidirectional.

## Anti-patterns

- Use full words rather than abbreviations that aren't the domain language (`usr` vs `user`).
- Keep related components unified or properly split by role rather than spreading one hook across several similarly named files -
  split by role folder (`components/`, `services/`, `types/`).