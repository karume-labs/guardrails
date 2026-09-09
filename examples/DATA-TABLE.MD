# Data Table Example

TanStack Table rendered in a shared UI component, column defs built
by a factory, and filter state driven by **nuqs** URL params.

## 1. Shared DataTable wrapper

```tsx
// shared/components/ui/data-table.tsx
"use client";
import { useReactTable, flexRender, getCoreRowModel, type ColumnDef } from "@tanstack/react-table";

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
}

export const DataTable = <TData, TValue>({ columns, data }: DataTableProps<TData, TValue>) => {
  const table = useReactTable({ data, columns, getCoreRowModel: getCoreRowModel() });
  return (
    <table>
      <thead>
        {table.getHeaderGroups().map((hg) => (
          <tr key={hg.id}>
            {hg.headers.map((header) => (
              <th key={header.id}>
                {header.isPlaceholder
                  ? null
                  : flexRender(header.column.columnDef.header, header.getContext())}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id}>
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

## 2. Column factory

```tsx
// features/admin/components/GamesColumns.tsx
"use client";
import type { ColumnDef } from "@tanstack/react-table";

export type AdminGame = { id: string; title: string; isActive: boolean };

export const buildGamesColumns = ({
  onToggleActive,
  onDelete,
}: {
  onToggleActive: (game: AdminGame) => void;
  onDelete: (game: AdminGame) => void;
}): ColumnDef<AdminGame, unknown>[] => [
  { id: "title", accessorKey: "title", header: "Game" },
  {
    id: "active",
    accessorKey: "isActive",
    header: "Status",
    cell: ({ row }) => <span>{row.original.isActive ? "Active" : "Inactive"}</span>,
  },
  {
    id: "actions",
    header: "",
    cell: ({ row }) => (
      <span className="flex gap-2">
        <button onClick={() => onToggleActive(row.original)}>Toggle</button>
        <button onClick={() => onDelete(row.original)}>Delete</button>
      </span>
    ),
  },
];
```

## 3. View wired to nuqs URL state

```tsx
// features/admin/components/GamesView.tsx
"use client";
import { parseAsString, useQueryState } from "nuqs";
import { useAdminGames } from "@/features/admin/services/queries";
import { DataTable } from "@/shared/components/ui/data-table";
import { buildGamesColumns } from "@/features/admin/components/GamesColumns";

export const GamesView = () => {
  const [q, setQ] = useQueryState("q", parseAsString.withDefault(""));
  const { data, isLoading } = useAdminGames(q);
  const columns = buildGamesColumns({
    onToggleActive: (game) => void game,
    onDelete: (game) => void game,
  });

  return (
    <div>
      <input value={q} onChange={(e) => setQ(e.target.value || null)} placeholder="Search..." />
      <DataTable columns={columns} data={data ?? []} />
    </div>
  );
};
```

## 4. Debounced search with nuqs

```tsx
import { useQueryState } from "nuqs";
import { useEffect, useState } from "react";
import { parseAsString } from "nuqs";

export const useDebouncedSearchQuery = (delay = 300) => {
  const [q, setQ] = useQueryState("q", parseAsString.withDefault(""));
  const [local] = useState(q);
  useEffect(() => setQ(local || null), [delay]);
  return { q, setQ };
};
```

## Notes

- Keep filter/page/sort in the URL (nuqs) so deep links and back/forward work.
- `page`/`pageSize` should use `parseAsInteger` (see `rules/STATE-MANAGEMENT.MD`).
- Wrapper stays presentational; data + handlers come from the view/query layer.