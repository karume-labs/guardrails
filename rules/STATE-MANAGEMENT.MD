# State Management Rule

State is layered by durability. URL state via **nuqs**, server cache via
**TanStack Query**, ephemeral UI state via `useState`. Never invent a global
store for data that belongs in one of those layers.

## URL state (nuqs)

Anything a user should be able to **deep-link, bookmark, or share** lives in
the URL search params: search, filters, sort, pagination, active tab.

- Wrap the tree in `NuqsAdapter` (from `nuqs/adapters/next/app`).
- Read/write with `useQueryState` + typed parsers.

```tsx
const [q, setQ] = useQueryState("q", parseAsString.withDefault(""));
const [page, setPage] = useQueryState("page", parseAsInteger.withDefault(1));
```

- Use `.withDefault()` sparingly; prefer `clearOnDefault` so default values
  vanish from the URL.
- Parsers: `parseAsString`, `parseAsInteger`, `parseAsBoolean`, `parseAsArrayOf`.
- Set `null` to remove a param (e.g. `setSort(null)`).

## Server state (TanStack Query)

- Server data is never "local state" - it goes through the query layer with
  central keys. See `rules/DATA-FETCHING.MD`.
- Filters living in the URL become **query-key inputs**:

```ts
const [search] = useQueryState("search", parseAsString.withDefault(""));
const query = useProducts(search); // key: ["products", search]
```

## Ephemeral UI state

- Dialogs, toggles, hover, input caret - use `useState`/`useReducer` locally.
- Portal/confirm dialogs hold only the item being acted on, never copied data.

```ts
const [pendingDelete, setPendingDelete] = useState<AdminGame | null>(null);
```

## Rules of thumb

1. If it's in the URL, it drives a query key and can be removed with null.
2. If it's server-owned, it lives in TanStack cache with typed keys.
3. If it's transient UI, it stays in component state.
4. No `useContext` store for anything above. No zustand/jotai unless the
   project explicitly adds them.
5. Debounce URL writes for search inputs (300ms) to avoid spamming the URL
   on every keystroke; read the debounced value in the query key.