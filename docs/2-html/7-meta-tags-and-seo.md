---
sidebar_position: 7
---

# Meta tags and SEO

Define essential metadata for usability, sharing, and crawlability. Keep head clean—avoid unnecessary meta tags.

## Essentials

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Page title | Product</title>
<meta name="description" content="Concise, action-oriented summary (≤ 160 chars)." />
```

- **Title**: Unique per page; under ~60 characters.
- **Description**: Human-readable summary; avoid keyword stuffing.
- **Language**: Set on `<html lang="...">`.

## Canonical and indexing

- Provide a canonical URL to prevent duplicate-content issues.
- Use robots meta for page-level control (`index`, `noindex`, `follow`, `nofollow`).

```html
<link rel="canonical" href="https://example.com/page" />
<meta name="robots" content="index,follow" />
```

## Open Graph (social sharing)

```html
<meta property="og:type" content="article" />
<meta property="og:title" content="Page title" />
<meta property="og:description" content="Share-friendly summary." />
<meta property="og:url" content="https://example.com/page" />
<meta property="og:image" content="https://example.com/img/og-image.jpg" />
```

Recommendations:
- Image: 1200x630 (1.91:1), absolute URL.
- Keep titles under ~60 characters; descriptions 2–4 sentences.

## Twitter Cards

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Page title" />
<meta name="twitter:description" content="Share-friendly summary." />
<meta name="twitter:image" content="https://example.com/img/og-image.jpg" />
```

## Structured data (JSON-LD)

Use JSON-LD for rich results; keep schemas minimal and accurate.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Page title",
  "description": "Concise summary.",
  "url": "https://example.com/page",
  "datePublished": "2025-01-15",
  "dateModified": "2025-01-15"
}
</script>
```

## Performance considerations

- Avoid duplicate or conflicting meta tags.
- Keep head lean; large scripts in head MUST use `defer` unless strictly necessary.
- Preload only truly critical assets (see performance chapter).
