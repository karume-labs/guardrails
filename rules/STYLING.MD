# Styling Rule

The UI library owns styling. **Never override it, never fight it.** Choose
exactly one library per project (Mantine or shadcn) and stay consistent.

## Core principles

1. **Let the library do the work.** Use the library's primitives, variants,
   and style props. Do not append `className`, inline `style`, or custom
   CSS/SCSS to fight the framework.
2. **No TailwindCSS in Mantine projects.** `tailwind` classes and the
   `tailwind-merge`/`clsx` helper chains are banned where Mantine is the UI
   library.
3. **Custom styling lives in the theme**, not in the component. Register new
   colors, radii, and tokens once in the theme provider and reference them
   from everywhere.

## Mantine projects (acme style)

- Use Mantine components exclusively: `Button`, `Card`, `Group`, `Stack`,
  `Grid`.
- **No dot notation for sub-components.** Mantine has moved away from
  `Grid.Col`, `Card.Section`, etc. — always import the sub-component
  directly as a top-level named export. `Grid.Col`/`Card.Section` are
  deprecated; `GridCol`/`CardSection` are the current form:

```tsx
// Good
import { Grid, GridCol, Card, CardSection } from "@mantine/core";
<Card>
  <CardSection>...</CardSection>
</Card>
<Grid>
  <GridCol span={6}>...</GridCol>
</Grid>

// Bad
<Grid>
  <Grid.Col span={6} />
</Grid>
```

- The dot-notation ban applies to any parent with exported sub-components.
  Use the top-level form:
  `GridCol` (not `Grid.Col`), `CardSection` (not `Card.Section`),
  `CarouselSlide` (not `Carousel.Slide`), `List` + `ListItem`
  (not `List.Item`), `SpotlightRoot` (not `Spotlight.Root`),
  `AccordionControl` (not `Accordion.Control`),
  `ComboboxOption` (not `Combobox.Option`),
  `TabsList`/`TabsTab`/`TabsPanel` (not `Tabs.List`/`Tabs.Tab`/`Tabs.Panel`).

- **No `className` or `style` on React components.** Use native Mantine style
  props: `mb`, `mt`, `p`, `px`, `bg`, `c`, `bd`, `fw`, `fz`, `tt`, `w`.

```tsx
// Good
<Button mb="sm" bg="darkAnthracite.2" c="white" bd="1px solid ...">
  Submit
</Button>

// Bad
<Button className="mb-2 bg-custom">Submit</Button>
```

- Use theme refs (`bg="var(--mantine-color-card)"`) for dynamic surfaces.
- Icons come from a single icon set (`@tabler/icons-react`). Follow icon
  stroke conventions (`size`, `stroke`).

## shadcn projects (acme style)

- shadcn components are generated into `src/shared/components/ui/`. Do not
  edit the generated primitives by hand.
- Compose compositions by **props and variants** (`variant`, `size`), not by
  overriding internal styles.
- Styling stays in the component definitions. When you must extend, prefer a
  new variant in the `cva()` config over ad-hoc `className` soup.
- Utility helpers (`cn`, `cva`, `tailwind-merge`) are fine — they are the
  shadcn mechanism. The ban on `className` applies to **Mantine** projects.

## Theme tokens

- Colors, spacing, fonts, and borders are defined once (Mantine `createTheme`
  / shadcn `components.json` + CSS variables). Components reference tokens,
  never raw hex.
- Semantic tokens (`card`, `darkAnthracite`, `foreground`, `muted`) over
  positional ones, so dark mode and rebranding stay cheap.