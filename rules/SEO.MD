# SEO Rule

SEO is configured declaratively through the **Next.js Metadata API**, backed
by a shared metadata config and per-page overrides. Structured data is
injected via a small `SEOConfig` component.

## Metadata API

- Every page exports `metadata` or `generateMetadata`. Never hand-write
  `<head>` tags.
- A shared `metadataConfig` module defines the site-wide default:
  `metadataBase`, `title.template`, `description`, `openGraph`,
  `twitter`, `robots`, `alternates.canonical`.

```ts
// features/shared/components/metadataConfig.ts
export const metadataConfig: Metadata = {
  metadataBase: new URL(env.NEXT_PUBLIC_APP_URL),
  title: { default: "Acme", template: "%s | Acme" },
  description: "...",
  openGraph: { title: "Acme", url: appUrl, images: [...] },
  twitter: { card: "summary_large_image", images: [...] },
  alternates: { canonical: appUrl },
};
```

- Pages spread it and override the specific fields (`title`,
  `description`, `alternates`). `generateMetadata` is used when metadata
  depends on route params (dynamic products/categories).

## Structured data (JSON-LD)

- Inject JSON-LD with the `SEOConfig` component using
  `dangerouslySetInnerHTML` + `JSON.stringify`. It is the accepted way to
  emit `application/ld+json` in React.
- Keep schemas at the root layout: `WebSite` (with `SearchAction`) and
  `Organization` (+ `BreadcrumbList`). Feature pages add entity schemas
  (`Product`, `CollectionPage`) via the same component.

```tsx
const websiteSchema = {
  "@context": "https://schema.org",
  "@type": "WebSite",
  url: appUrl,
  potentialAction: { "@type": "SearchAction", target: `${appUrl}/products?search={q}` },
};
const SEOConfig = () => (
  <script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(websiteSchema) }} />
);
```

## Indexing

- Keep `robots` open (`index`, `follow`) for public pages; admin/auth pages
  set `noindex`. Never leave public pages as `noindex`.

## Client components

- `page.tsx` stays a server component - metadata + SEOConfig render server-side.
- The `SEOConfig` component can be `"use client"` if it reads env; keep its
  markup minimal so it never becomes a hydration bottleneck.

## When a rule file grows

Split per-page schemas (Product/Article JSON-LD) into
`features/seo/...` rather than growing this file past 150 lines.