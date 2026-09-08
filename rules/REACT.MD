# React Rule

Component and hook conventions for React 18/19 apps (Next.js App Router
specifics live in `rules/NEXTJS.MD`).

## Components

- Components are arrow functions with named exports.
- Props are typed with `interface` (named `Props` or `<Feature>Props`).
- Use `React.FC<Props>` where applicable.

```tsx
import type { ReactNode } from "react";
import { Card } from "@/shared/components/ui/card";

interface StatusBadgeProps {
  label: string;
  tone?: "success" | "warning" | "danger";
}

export const StatusBadge = ({ label, tone = "success" }: StatusBadgeProps): ReactNode => {
  return <Card tone={tone}>{label}</Card>;
};
```

- Keep components under ~150 lines. Extract children when larger.
- Prefer composition (children, slots) over prop bloat.
- No nested component definitions inside a render body — define at module
  scope so they are not recreated every render.

## Hooks

- Custom hooks live in `hooks/` (or `services/` when they wrap data fetching)
  and are named `useXxx`.
- Hooks return derived state + the query (`{ ...query, items }`). See
  `rules/DATA-FETCHING.MD`.
- Dependency arrays are complete: never suppress the linter with
  `// eslint-disable-line`.
- Do not call hooks in loops or conditions.

## Client vs Server components

Rule of thumb: a component is a **Server Component by default**. Add
`"use client"` only when it needs hooks or event handlers.

```tsx
// Server component (page files) - no "use client"
export const ProductPage = async () => {
  return <ProductGrid />; // client child
};
```

- Never put `"use client"` in `page.tsx`/`layout.tsx` route files.
- Client components pass data down via props, not by fetching server-side.

## State

- URL state (filters, pagination, search) -> nuqs. See `rules/STATE-MANAGEMENT.MD`.
- Server cache state -> TanStack Query. See `rules/DATA-FETCHING.MD`.
- Ephemeral UI state (dialogs, toggles) -> `useState`/`useReducer`.

## Prop typing examples

```tsx
// Good
interface ProductCardProps {
  product: Product;
  onAddToCart: (productId: string) => void;
}

// Bad - inline anonymous types when reusable
export const ProductCard = (props: { product: Product; onAdd: (id: string) => void }) => {}
```