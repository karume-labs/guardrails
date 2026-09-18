# Mantine Styling & UI Rules

- Use Mantine components exclusively: `Button`, `Card`, `Group`, `Stack`, `Grid`.
- **No dot notation for sub-components.** Mantine has moved away from `Grid.Col`, `Card.Section`, etc. - always import the sub-component directly as a top-level named export. `Grid.Col`/`Card.Section` are deprecated; `GridCol`/`CardSection` are the current form:

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

- The dot-notation ban applies to any parent with exported sub-components. Use the top-level form: `GridCol` (not `Grid.Col`), `CardSection` (not `Card.Section`), `CarouselSlide` (not `Carousel.Slide`), `List` + `ListItem` (not `List.Item`), `SpotlightRoot` (not `Spotlight.Root`), `AccordionControl` (not `Accordion.Control`), `ComboboxOption` (not `Combobox.Option`), `TabsList`/`TabsTab`/`TabsPanel` (not `Tabs.List`/`Tabs.Tab`/`Tabs.Panel`).

- **No `className` or `style` on React components.** Use native Mantine style props: `mb`, `mt`, `p`, `px`, `bg`, `c`, `bd`, `fw`, `fz`, `tt`, `w`.

```tsx
// Good
<Button mb="sm" bg="darkAnthracite.2" c="white" bd="1px solid ...">
  Submit
</Button>

// Bad
<Button className="mb-2 bg-custom">Submit</Button>
```

- Use theme refs (`bg="var(--mantine-color-card)"`) for dynamic surfaces.
- Icons come from a single icon set (`@tabler/icons-react`). Follow icon stroke conventions (`size`, `stroke`).

## Spacing Rules & Best Practices

1. **No Inline Styles:** Remove all instances of `style={{ ... }}`.
2. **Minimize Style Props:** Avoid using Mantine style props (like margin or padding) as much as possible. Rely on layout components (like `Stack`, `Group`, `Flex`) and their native spacing properties (e.g., `gap`).
3. **No Directional Spacing:** Never use single-direction margin or padding props (`ml`, `mr`, `mt`, `mb`, `pl`, `pr`, `pt`, `pb`). 
4. **Only Axis/Global Spacing:** When spacing props are absolutely necessary, ONLY use the `x` (horizontal) and `y` (vertical) axes (i.e., `mx`, `my`, `px`, `py`) or global spacing (`m`, `p`).
5. **Divisible by 2 and 4:** Any hardcoded numerical value used for spacing or layout must be strictly divisible by 2 and 4 (e.g., `4`, `8`, `12`, `16`, `24`, `32`).

## Holistic UI Architecture

1. **Component Restructuring:** We have full freedom to refactor the UI structure itself to make the layouts cleaner and more robust. If a layout can be achieved more elegantly with a different Mantine component (e.g., swapping a complex `Box` layout for a `SimpleGrid` or `Paper`), we will do so.
2. **Semantic Mantine Components:** Ensure we are using the correct Mantine components for their intended semantic purpose. For instance, using `<Card>` or `<Paper>` instead of styling a `<Box>` to look like a card. Using `<Anchor>` for links instead of styling a `<Text component="a">`.
3. **Responsive Design native props:** Ensure responsive behavior leverages Mantine's native object syntax (e.g., `w={{ base: '100%', md: '50%' }}`) or hooks (`hiddenFrom`, `visibleFrom`) rather than messy manual styles.

## Common Patterns & Refactoring Strategies

### 1. Structural Borders, Backgrounds, and Radius
**Current Issue:** `<Box p="lg" bg="white" style={{ borderRadius: "var(--mantine-radius-md)", overflow: "hidden" }}>`
**Proposed Change:** Use `<Paper>` or `<Card>` components natively designed for these properties.
```tsx
// ❌ BAD
<Box p="lg" bg="white" style={{ borderRadius: "var(--mantine-radius-md)", overflow: "hidden" }}>

// ✅ GOOD
<Paper p="lg" bg="white" radius="md">
// Note: If clipping is needed, Card provides native overflow control.
```

### 2. Flexbox Properties (`flex`, `flexShrink`)
**Current Issue:** `<Box style={{ flex: 1 }}>` or `<Stack style={{ flexShrink: 0 }}>`
**Proposed Change:** Rely on parent layout containers like `<Group grow>` when possible, or use the native `flex` prop as a fallback.
```tsx
// ❌ BAD
<Stack style={{ flex: 1 }}>

// ✅ GOOD (Fallback)
<Stack flex={1}>
```

### 3. Spacing and Margins (The Strict Rules)
**Current Issue:** Using single-direction spacing (`ml`, `mt`) or arbitrary odd values (`ml={3}`, `mt={15}`).
**Proposed Change:** Use `Stack` or `Group` for component spacing (via `gap`). If spacing props are unavoidable, use `mx`, `my`, `px`, `py`, `m`, or `p` with values divisible by 4 (e.g., 4, 8, 12, 16).
```tsx
// ❌ BAD
<IconPlayerPlay style={{ marginLeft: 3 }} />
<IconPlayerPlay ml={3} />
<Box mt={10} pr={5} />

// ✅ GOOD (Using Layout)
<Group gap="xs">
  <Text>Play</Text>
  <IconPlayerPlay />
</Group>

// ✅ GOOD (Using proper axis & divisible values)
<Box my={12} px={16} />
```

### 4. Positioning (`position`, `top`, `zIndex`)
**Current Issue:** `<Box style={{ position: "sticky", top: 80, zIndex: 300 }}>`
**Proposed Change:** Use Mantine's native layout props (`pos`, `top`, `z`) only if absolutely necessary, ensuring values align with our numbering rules (e.g., `top={80}` is divisible by 4).
```tsx
// ❌ BAD
<Box style={{ position: "sticky", top: 80, zIndex: 300 }}>

// ✅ GOOD
<Box pos="sticky" top={80} z={300}>
```
