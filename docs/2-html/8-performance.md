---
sidebar_position: 8
---

# Performance

Optimize for Core Web Vitals (LCP, INP, CLS). Keep HTML discoverable to the preload scanner and minimize render-blocking resources.

## Resource hints

- `preconnect` for critical third-party origins (fonts, APIs) used early.
- `dns-prefetch` for many low-priority cross-origins.
- `preload` for render-critical assets discovered late by default (fonts, hero images, critical CSS); always set `as`.
- `prefetch` for likely next-page assets.

```html
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link rel="preload" as="style" href="/css/critical.css" />
<link rel="preload" as="image" href="/img/hero.avif" fetchpriority="high" />
```

## Scripts

- Default to `defer` for scripts that run after parse.
- Use `async` for independent scripts that do not rely on DOM order.
- Avoid inline `document.write` and synchronous third-party embeds.
- Place non-critical scripts after critical content; avoid blocking the parser.

```html
<script src="/js/app.js" defer></script>
<script src="https://example.com/analytics.js" async></script>
```

## Styles

- Keep CSS small and cacheable; avoid `@import` in CSS.
- Inline only the minimal critical CSS needed for above-the-fold content.
- Load remaining styles with standard `<link rel="stylesheet">`; avoid JS-inserted styles when possible to keep them discoverable.

## Images and media

- Use responsive images and modern formats (see media chapter).
- Set dimensions to prevent CLS.
- Use `loading="lazy"` for non-critical media; `fetchpriority="high"` for hero media.

## Core Web Vitals focus

- **LCP**: Optimize hero image/font loading; reduce TTFB with caching and CDN.
- **INP**: Defer heavy JS; avoid long tasks; prefer native controls over JS-heavy custom UI.
- **CLS**: Reserve space for images/ads; avoid late-loading fonts without fallback strategies (font-display swap/optional).

## Critical rendering path

- Keep HTML lean; avoid excessive head bloat.
- Minimize number of render-blocking resources; combine when it reduces requests without harming cacheability.
- Serve compressed (gzip/brotli) assets; enable HTTP caching with sensible max-age and ETag.

## Diagnostics

- Use Lighthouse or WebPageTest to profile LCP/INP/CLS regressions.
- Track real-user monitoring (RUM) for Core Web Vitals in production.
