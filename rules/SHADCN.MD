# shadcn projects (shadcn style)

- shadcn components are generated into `src/shared/components/ui/`. Do not edit the generated primitives by hand.
- Compose compositions by **props and variants** (`variant`, `size`), not by overriding internal styles.
- Styling stays in the component definitions. When you must extend, prefer a new variant in the `cva()` config over ad-hoc `className` soup.
- Utility helpers (`cn`, `cva`, `tailwind-merge`) are fine — they are the shadcn mechanism. The ban on `className` applies to **Mantine** projects.
