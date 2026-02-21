---
sidebar_position: 7
---

# Meta tags and SEO

Meta tags provide metadata about the HTML document to browsers, search engines, and social media platforms. SEO (Search Engine Optimization) encompasses the practices that improve a page's visibility and ranking in search engine results. Effective metadata is concise, accurate, and avoids unnecessary tags.

## Essential meta tags

Every page MUST include these fundamental meta elements:

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Page Title | Brand</title>
<meta name="description" content="Concise, action-oriented summary of the page content (aim for ≤ 160 characters)." />
```

### Title

The `<title>` element is the single most important on-page SEO signal and the text displayed in browser tabs, bookmarks, and search engine result pages (SERPs).

- **MUST** be unique per page.
- **SHOULD** be under ~60 characters to avoid truncation in search results.
- **SHOULD** follow a consistent format such as `Primary Topic | Brand Name` or `Primary Topic — Brand Name`.
- Avoid keyword stuffing — write for humans first.

### Description

The `<meta name="description">` content appears as the snippet below the title in search results.

- **SHOULD** be 120–160 characters — long enough to be informative, short enough to avoid truncation.
- Write a human-readable summary that encourages clicks. Describe what the user will find on the page.
- Avoid duplicating the same description across multiple pages.

### Language

Set `lang` on the `<html>` element (e.g., `lang="en"`, `lang="es"`). This is covered in the document structure chapter but is also important for SEO — search engines use it to serve the correct language version to users.

## Canonical URLs

A canonical URL tells search engines which version of a page is the "primary" one when the same content is accessible at multiple URLs (e.g., with/without trailing slash, with query parameters, HTTP vs HTTPS).

```html
<link rel="canonical" href="https://example.com/page" />
```

- **SHOULD** include a canonical link on every page.
- The `href` MUST be an absolute URL.
- Self-referencing canonicals (pointing to the current page) are valid and recommended.

## Robots and indexing

The robots meta tag provides page-level instructions to search engine crawlers about whether to index the page and follow its links.

```html
<!-- Default behavior: index the page and follow links -->
<meta name="robots" content="index,follow" />

<!-- Prevent indexing but follow links -->
<meta name="robots" content="noindex,follow" />

<!-- Prevent indexing and link following -->
<meta name="robots" content="noindex,nofollow" />
```

Common directives:

| Directive | Meaning |
|---|---|
| `index` | Allow the page to appear in search results (default). |
| `noindex` | Prevent the page from appearing in search results. |
| `follow` | Allow the crawler to follow links on the page (default). |
| `nofollow` | Prevent the crawler from following links on the page. |
| `noarchive` | Prevent search engines from showing a cached copy. |
| `nositelinkssearchbox` | Prevent a sitelinks search box from appearing in search results. |

For site-wide crawl control, use `robots.txt`. The robots meta tag is for page-level overrides.

## Open Graph (social sharing)

[Open Graph](https://ogp.me/) (OG) meta tags control how a page appears when shared on social media platforms (Facebook, LinkedIn, Slack, Discord, etc.). Without them, platforms generate previews from page content, which is often inaccurate.

```html
<meta property="og:type" content="article" />
<meta property="og:title" content="Page Title" />
<meta property="og:description" content="A share-friendly summary of the content." />
<meta property="og:url" content="https://example.com/page" />
<meta property="og:image" content="https://example.com/img/og-image.jpg" />
<meta property="og:site_name" content="Brand Name" />
```

Recommendations:
- **og:image** — Use an absolute URL. Recommended size is 1200x630 pixels (1.91:1 aspect ratio) for optimal display across platforms.
- **og:title** — Keep under ~60 characters.
- **og:description** — 2–4 concise sentences.
- **og:type** — Use `website` for homepages, `article` for content pages, `product` for e-commerce.

## Twitter Cards

Twitter (X) uses its own card meta tags, with fallback to Open Graph tags when Twitter-specific tags are absent.

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Page Title" />
<meta name="twitter:description" content="A share-friendly summary." />
<meta name="twitter:image" content="https://example.com/img/og-image.jpg" />
<meta name="twitter:site" content="@brandhandle" />
```

Card types:
- `summary` — Small thumbnail with title and description.
- `summary_large_image` — Large image banner with title and description (most common for articles).

## Favicon and app icons

Favicons appear in browser tabs, bookmarks, and app shortcuts. Modern best practice is to provide a minimal set:

```html
<link rel="icon" href="/favicon.ico" sizes="32x32" />
<link rel="icon" href="/img/icon.svg" type="image/svg+xml" />
<link rel="apple-touch-icon" href="/img/apple-touch-icon.png" />
```

- `favicon.ico` — Legacy support; 32x32 ICO file.
- SVG icon — Scales to any size; supports dark mode via CSS `prefers-color-scheme` media queries inside the SVG.
- Apple touch icon — 180x180 PNG for iOS home screen shortcuts.

## Structured data (JSON-LD)

Structured data helps search engines understand page content and can produce **rich results** (enhanced search listings with images, ratings, dates, FAQ dropdowns, etc.).

JSON-LD (JavaScript Object Notation for Linked Data) is the recommended format by Google. Embed it in a `<script type="application/ld+json">` block in the `<head>` or `<body>`.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Building Accessible Forms",
  "description": "A guide to creating forms that work for everyone.",
  "url": "https://example.com/guides/accessible-forms",
  "datePublished": "2025-06-01",
  "dateModified": "2025-06-15",
  "author": {
    "@type": "Organization",
    "name": "Engineering Team"
  },
  "image": "https://example.com/img/forms-guide.jpg"
}
</script>
```

Common schema types: `Article`, `FAQPage`, `BreadcrumbList`, `Product`, `Organization`, `WebSite`. Validate with [Google's Rich Results Test](https://search.google.com/test/rich-results) or [Schema.org Validator](https://validator.schema.org/).

## Performance considerations

- Avoid duplicate or conflicting meta tags — they confuse crawlers and waste bytes.
- Keep `<head>` lean. Every element in `<head>` is processed before the browser starts rendering the body.
- Large scripts in `<head>` MUST use `defer` to avoid blocking the HTML parser (see the performance chapter).
- Preload only truly critical assets; excessive preload hints can backfire by competing for bandwidth.
