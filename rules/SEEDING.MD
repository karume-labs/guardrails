# Seeding Rule

Database seeding follows a feature-based pattern. Seeding is a
db-package concern - never in app or client code.

## Location

A feature's seed logic lives in:

```
packages/db/src/<feature>/seed/index.ts
packages/db/src/<feature>/seed/data.json
```

The root seed entry is `packages/db/src/seed/index.ts`. It
composes every feature seeder and exposes the single `./seed`
package export.

## Data-driven construction

Each seeder composes its records from a `data.json` file of
raw building blocks (openers, bodies, names, rating profiles).
A deterministic seeded-random helper produces repeatable output
given a prefix, so reseeding is idempotent for a given prefix.

```ts
// packages/db/src/reviews/seed/index.ts
import seedData from "@db/reviews/seed/data.json";

const seededRandom = (seedStr: string): (() => number) => { ... };

export const seedReviews = async () => {
  const count = await db.$count(reviews);
  if (count > 0) return; // skip already-seeded
  // compose records from seedData building blocks ...
  await db.insert(reviews).values(recordsToInsert);
};
```

- Each feature exports a named `seed<Feature>()` async function.
- Guard against re-seeding: bail early if the target table has rows.
- Deterministic seeding means `data.json` + prefix reproduce the
  same dataset every run.

## Root entry and flags

The root `packages/db/src/seed/index.ts` aggregates all feature
seeders and accepts `--<feature>` flags so you can seed one slice
at a time:

```
bun run db:seed                  # seed all
bun run db:seed --reviews       # seed reviews only
bun run db:seed --force         # clear data, then seed all
bun run db:seed:fresh           # db:migrate && db:seed --force
```

## Package exports and scripts

- The db package exposes `"./seed": "./src/seed/index.ts"`
  as a package export.
- Root `package.json` delegates via `bun --cwd packages/db db:seed`.
- Frontend client code must never seed the database.

## Seeder composition order

Seed in dependency order: auth/roles first, then catalog entities,
then reviews/ratings, then orders/cart sessions. The root entry
reflects this order.