# Styling Rule

The UI library owns styling. **Embrace it and work within its constraints.** Choose
exactly one library per project (Mantine or shadcn) and stay consistent.

## Core principles

1. **Let the library do the work.** Use the library's primitives, variants,
   and style props. Rely exclusively on the library's styling mechanisms instead of appending `className`, inline `style`, or custom
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
- **React Native Reusables:** See [REACT-NATIVE-REUSABLES.MD](./REACT-NATIVE-REUSABLES.MD) for shadcn-style rules in React Native.

## Theme tokens

- Colors, spacing, fonts, and borders are defined once (Mantine `createTheme`
  / shadcn `components.json` + CSS variables). Components reference tokens,
  rather than raw hex values.
- Semantic tokens (`card`, `darkAnthracite`, `foreground`, `muted`) over
  positional ones, so dark mode and rebranding stay cheap.

## No Inline Styles

Replace inline styles (`style={{ ... }}`) with the UI library's primitives, variants, style props, or proper components (e.g., use `next/image` instead of `img` tags).

## Mobile Friendly Tables

Tables must be designed with mobile devices in mind. Use the following techniques to ensure readability and usability on small screens:

- **Horizontal Scrolling**: Wrap the table in a scrollable container so users can swipe through extra columns while keeping the layout intact.
- **Sticky First Column**: Lock the leftmost label column in place during horizontal scrolling so users always maintain context of what data they are reading.
- **Column Stacking**: Transform wide columns into vertical rows or cards, stacking information top-to-bottom instead of side-to-side.
- **Hiding Non-Essential Columns**: Use responsive breakpoints to hide secondary or low-priority columns on small screens, letting users focus on core data.
- **Progressive Disclosure**: Move deep details into popups, expandable accordions, or separate pages rather than cramming them into the main table view.

### Content and Styling Adjustments

- **Prioritize Key Data**: Display only the two or three most critical columns (like item name and price) by default.
- **Abbreviate and Iconize**: Shorten long text labels and replace repeated text words with simple icons (like a star or checkmark).
- **Reduce Font Size**: Use smaller body text sizes and bold key values instead of headers to improve quick scanning.
- **Simplify Rows**: Apply alternating background colors (zebra striping) and horizontal-only divider lines to guide the eye across compact rows.