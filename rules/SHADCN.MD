# shadcn projects (shadcn style)

- shadcn components are generated into `src/shared/components/ui/`. Do not edit the generated primitives by hand.
- **Never edit shadcn-generated files.** Files produced by `shadcn` (everything
  inside `src/shared/components/ui/`) are strictly read-only. Do not modify
  them for any reason unless the user explicitly asks you to. If you need
  different behavior, create a wrapper component or add a new variant in your
  own code - never patch the generated source.
- Compose compositions by **props and variants** (`variant`, `size`), not by overriding internal styles.
- Styling stays in the component definitions. When you must extend, prefer a new variant in the `cva()` config over ad-hoc `className` soup.
- Utility helpers (`cn`, `cva`, `tailwind-merge`) are fine - they are the shadcn mechanism. The ban on `className` applies to **Mantine** projects.
