# React Native Reusables projects (shadcn style)

- React Native Reusables components are generated into `components/ui/` (or a similar ui folder). Treat the generated primitives as read-only.
- **Treat generated files as read-only.** Files produced by `react-native-reusables` are strictly read-only. Leave them unmodified unless the user explicitly asks you to. If you need different behavior, create a wrapper component or add a new variant in your own code rather than patching the generated source.
- Compose compositions by **props and variants** (`variant`, `size`), not by overriding internal styles.
- Styling stays in the component definitions. When you must extend, prefer a new variant in the `cva()` config over ad-hoc style object soup.
- Utility helpers (like `cn`, `cva`, etc.) are fine - they are the standard mechanism.
