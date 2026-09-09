# Data Tables Rule

Data grids use **TanStack Table** (headless, presentational) and keep
pagination/sort/search in URL state via **nuqs**.

## Architecture

- A shared, presentational `DataTable` wrapper (`shared/components/ui/`)
  renders `columns` + `data` with `useReactTable`. No business logic inside.
- Column definitions are built by a **per-feature factory**
  (`<Feature>Columns.tsx`) that returns `ColumnDef<T, unknown>[]` and receives
  action callbacks as props.
- A **view** component owns state: it reads nuqs URL params, calls the query
  hooks, and passes columns + data down.

```
features/admin/
  components/
    GamesColumns.tsx   # buildGamesColumns({ onEdit, onDelete, ... })
    GamesView.tsx      # nuqs state + queries + <DataTable>
```

## Column factories

```ts
// GamesColumns.tsx
export const buildGamesColumns = ({
  onToggleActive,
  onDelete,
}: {
  onToggleActive: (g: AdminGame) => void;
  onDelete: (g: AdminGame) => void;
}): ColumnDef<AdminGame, unknown>[] => [
  { id: "title", accessorKey: "title", header: "Game" },
  { id: "active", accessorKey: "isActive", header: "Status" },
  {
    id: "actions",
    cell: ({ row }) => (
      <Group gap="xs">
        <Button onClick={() => onToggleActive(row.original)}>Toggle</Button>
        <Button onClick={() => onDelete(row.original)}>Delete</Button>
      </Group>
    ),
  },
];
```

## View with URL-driven pagination

```tsx
// GamesView.tsx
"use client";
const [page, setPage] = useQueryState("page", parseAsInteger.withDefault(1));
const [pageSize, setPageSize] = useQueryState("pageSize", parseAsInteger.withDefault(20));
const [q, setQ] = useQueryState("q", parseAsString.withDefault(""));

const { data, totalRecords } = useAdminGames(page, pageSize, q);

<DataTable
  columns={columns}
  data={data ?? []}
  totalRecords={totalRecords}
  page={page}
  pageSize={pageSize}
  onPageChange={setPage}
  onPageSizeChange={(n) => { setPageSize(n); setPage(1); }}
/>
```

## Server-side pagination

- Pass `page`/`pageSize` + filters into the query hooks; they map to cursor
  or offset pagination on the server (see `rules/DATA-FETCHING.MD`).
- Mapping `page` -> `offset` happens inside the query hook, not in the table.

## Defaults

- Table renders a skeleton while `isLoading`.
- Empty state shows a "No results" card with a clear-filters action.
- Row click (when enabled) skips actions columns and sortable headers.