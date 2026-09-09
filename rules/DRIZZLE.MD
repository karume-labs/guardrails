# Drizzle Rule

PostgreSQL via **Drizzle ORM**, colocated with feature folders.

## Layout

The db package mirrors `apps/api/src/features` in
`packages/db/src/<feature>/schema/index.ts`.

```
packages/db/src/
  <feature>/schema/index.ts   # pgTable + relations for the feature
  client.ts                   # single drizzle client, all schemas
  env/index.ts                # validated DATABASE_URL etc.
  utils.ts                    # lifecycleTimestamps and shared column defs
  migrations/                 # drizzle-kit generated SQL
  seed/<feature>/             # feature seeding pattern (rules/SEEDING.MD)
```

## Table definitions

- Every table is `pgTable("snake_case", { ... })` with `uuid` PK
  `defaultRandom()`, and snake_case column names (`category_id`).
- Spread `lifecycleTimestamps()` (`createdAt`/`updatedAt`) on tables that
  need them.

```ts
export const categories = pgTable("categories", {
  id: uuid("id").primaryKey().defaultRandom(),
  name: varchar("name").unique().notNull(),
  slug: varchar("slug").unique().notNull(),
  ...lifecycleTimestamps(),
});
```

## Relations

- Define a `relations` export per table for every FK. Name the relation
  object `<table>Relations` and keep it next to its table.

```ts
export const subcategoriesRelations = relations(
  subcategories,
  ({ one, many }) => ({
    category: one(categories, {
      fields: [subcategories.categoryId],
      references: [categories.id],
    }),
    products: many(products),
  }),
);
```

- FK columns `references(() => parentTable.id, { onDelete: "cascade" })`.
- Keep `onDelete` semantics explicit — do not omit cascade on child tables.

## Client

A single `client.ts` imports every feature schema, flattens them, and exports
one `drizzle` instance. Consumers import `{ db }` only from `@db/client`.

```ts
// client.ts
import * as products from "@db/products/schema";
const schema = { ...products, ...reviews };
export const db = drizzle(queryClient, { schema });
```

## CLI / migrations

- `drizzle.config.ts` globs `./src/*/schema/*.ts`, outputs to
  `./src/migrations`, and reads `DATABASE_URL` from the validated env.
- Generated migrations are committed and never edited by hand.
- Scripts live in the db package: `db:generate`, `db:push`, `db:migrate`,
  `db:studio`, `db:seed`.

## Conventions

- Money columns are `decimal(..., { precision: 10, scale: 2 })`, not floats.
- Booleans default explicitly (`isOnSale: boolean("is_on_sale").default(false)`).
- Use `uniqueIndex`/`.unique()` for slug- or code-uniqueness to mirror the
  domain constraint.
- Query shapes are projected with `.select({...})` — never `select() *`
  unless the whole row is truly needed.