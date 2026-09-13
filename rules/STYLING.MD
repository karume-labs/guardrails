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

## UI Libraries

- **Mantine:** See [MANTINE.MD](./MANTINE.MD) for Mantine-specific styling and UI rules.
- **shadcn:** See [SHADCN.MD](./SHADCN.MD) for shadcn-specific styling rules.

## Theme tokens

- Colors, spacing, fonts, and borders are defined once (Mantine `createTheme`
  / shadcn `components.json` + CSS variables). Components reference tokens,
  never raw hex.
- Semantic tokens (`card`, `darkAnthracite`, `foreground`, `muted`) over
  positional ones, so dark mode and rebranding stay cheap.