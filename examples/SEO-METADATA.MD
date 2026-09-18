# SEO Metadata Example

A root metadata config applied via the
Next.js Metadata API, plus a `SEOConfig` component that injects
JSON-LD structured data.

## 1. Shared metadata config

```ts
// features/shared/components/metadataConfig.ts
import type { Metadata } from "next";
import { env } from "@web/features/shared/env";

const appUrl = env.NEXT_PUBLIC_APP_URL;

export const metadataConfig: Metadata = {
  metadataBase: new URL(appUrl),
  title: {
    default: "Acme | Premium Luxury Reselling",
    template: "%s | Acme",
  },
  description: "Shop high-quality affordable luxury items.",
  openGraph: {
    title: "Acme",
    description: "Dress for less while making money doing it.",
    url: appUrl,
    siteName: "Acme",
    locale: "en_US",
    type: "website",
    images: [{ url: `${appUrl}/images/shared/og-image.png`, width: 1200, height: 630 }],
  },
  twitter: {
    card: "summary_large_image",
    images: [`${appUrl}/images/shared/og-image.png`],
  },
  robots: {
    index: true,
    follow: true,
    googleBot: { index: true, follow: true, "max-image-preview": "large" },
  },
  alternates: { canonical: appUrl },
};
```

## 2. Per-page metadata

```ts
// app/products/page.tsx
import type { Metadata } from "next";
import { metadataConfig } from "@web/features/shared/components/metadataConfig";

export const metadata: Metadata = {
  ...metadataConfig,
  title: "Products",
  description: "Browse the full catalog.",
  alternates: { canonical: `${metadataConfig.metadataBase}/products` },
};
```

## 3. Structured data (JSON-LD)

```tsx
// features/shared/components/SEOConfig.tsx
"use client";
import { env } from "@web/features/shared/env";

const appUrl = env.NEXT_PUBLIC_APP_URL;

const websiteSchema = {
  "@context": "https://schema.org",
  "@type": "WebSite",
  name: "Acme",
  url: appUrl,
  potentialAction: {
    "@type": "SearchAction",
    target: `${appUrl}/products?search={search_term_string}`,
    "query-input": "required name=search_term_string",
  },
};

const SEOConfig = () => (
  <>
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(websiteSchema) }}
    />
  </>
);

export default SEOConfig;
```

## 4. Usage in layout

```tsx
// app/layout.tsx
import SEOConfig from "@web/features/shared/components/SEOConfig";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <SEOConfig />
        {children}
      </body>
    </html>
  );
}
```

## Notes

- `metadataConfig` stays a shared non-component module; pages spread it
  and override `title`/`description`.
- Keep OpenGraph + Twitter + canonical in the base config so every page
  inherits it.
- JSON-LD is per-page opt-in (`SEOConfig`) - add Organization,
  BreadcrumbList, or Product schemas via the same pattern.