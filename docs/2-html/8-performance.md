---
sidebar_position: 8
---

# Performance

Web performance directly impacts user experience, conversion rates, and search rankings. This chapter focuses on HTML-level techniques to optimize for **Core Web Vitals** — the three metrics Google uses to assess real-world user experience:

- **LCP (Largest Contentful Paint)** — Measures perceived load speed. The time until the largest visible element (hero image, heading block) finishes rendering. Target: **≤ 2.5 seconds**.
- **INP (Interaction to Next Paint)** — Measures responsiveness. The delay between a user interaction (click, tap, keypress) and the next visual update. Target: **≤ 200 milliseconds**.
- **CLS (Cumulative Layout Shift)** — Measures visual stability. The total amount of unexpected layout movement during the page lifecycle. Target: **≤ 0.1**.

## Resource hints

Resource hints are `<link>` elements in `<head>` that inform the browser about resources it will need, allowing it to start fetching or connecting earlier than it otherwise would.

### preconnect

Establishes an early connection (DNS + TCP + TLS) to a cross-origin server. Use for origins that serve critical resources loaded early in the page lifecycle.

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
```

- Add `crossorigin` when the resource will be fetched with CORS (Cross-Origin Resource Sharing), such as fonts.
- Limit to 2–4 origins to avoid connection contention.

### dns-prefetch

Performs only DNS (Domain Name System) resolution — lighter than `preconnect`. Use as a fallback for browsers that do not support `preconnect`, or for lower-priority cross-origin connections.

```html
<link rel="dns-prefetch" href="https://analytics.example.com" />
```

### preload

Declares a high-priority fetch for a resource the browser needs immediately but would not discover until later in the parsing process. Always specify the `as` attribute so the browser sets the correct request headers and caching policy.

```html
<link rel="preload" as="style" href="/css/critical.css" />
<link rel="preload" as="font" href="/fonts/inter.woff2" type="font/woff2" crossorigin />
<link rel="preload" as="image" href="/img/hero.avif" fetchpriority="high" />
```

Common `as` values: `style`, `script`, `font`, `image`, `fetch`.

:::warning[bad]

```html
<!-- Preloading too many resources wastes bandwidth and delays critical ones -->
<link rel="preload" as="image" href="/img/footer-logo.png" />
<link rel="preload" as="image" href="/img/sidebar-ad.jpg" />
<link rel="preload" as="image" href="/img/author-avatar.png" />
```

:::

:::tip[good]

```html
<!-- Preload only the hero image that determines LCP -->
<link rel="preload" as="image" href="/img/hero.avif" fetchpriority="high" />
```

:::

### prefetch

Fetches a resource that will likely be needed on the **next** page navigation. The browser downloads it at low priority during idle time.

```html
<link rel="prefetch" href="/next-page.html" />
<link rel="prefetch" as="script" href="/js/next-page-bundle.js" />
```

Use sparingly — prefetching consumes bandwidth on the user's device, which is especially costly on metered mobile connections.

## Script loading

How you load JavaScript has a major impact on page performance. Scripts without `async` or `defer` block HTML parsing, delaying rendering.

### async vs defer

| Attribute | Download | Execution | Order guaranteed | Use for |
|---|---|---|---|---|
| (none) | Blocks parsing | Blocks parsing | Yes | Avoid — rarely needed |
| `async` | Parallel to parsing | As soon as downloaded (pauses parsing) | No | Independent scripts (analytics, ads) |
| `defer` | Parallel to parsing | After HTML is fully parsed | Yes | Application scripts that depend on DOM |

```html
<!-- Application script — defer ensures DOM is ready and execution order is preserved -->
<script src="/js/app.js" defer></script>

<!-- Analytics — async because it is independent and order does not matter -->
<script src="https://analytics.example.com/tracker.js" async></script>
```

**SHOULD** default to `defer` for application scripts. Use `async` only for truly independent scripts. Avoid inline `document.write()` — it forces synchronous parsing and is incompatible with `async`/`defer`.

### Module scripts

ES module scripts (`type="module"`) are deferred by default, making them a good choice for modern applications.

```html
<script type="module" src="/js/app.mjs"></script>
```

## Styles and render blocking

CSS is render-blocking — the browser will not paint anything until it has downloaded and parsed all stylesheets in `<head>`. Minimizing CSS impact is critical for LCP.

- Keep CSS files small and cacheable. Avoid CSS `@import` declarations — each `@import` creates an additional sequential request.
- **Inline critical CSS** — Embed only the minimal CSS needed for above-the-fold content directly in a `<style>` tag in `<head>`. Load the full stylesheet asynchronously.
- Load remaining styles with standard `<link rel="stylesheet">`. Avoid inserting stylesheets via JavaScript when possible — JS-inserted styles are invisible to the browser's preload scanner.

```html
<head>
  <!-- Inline critical CSS for immediate rendering -->
  <style>
    body { margin: 0; font-family: system-ui, sans-serif; }
    .hero { min-height: 50vh; }
  </style>

  <!-- Full stylesheet loaded normally -->
  <link rel="stylesheet" href="/css/main.css" />
</head>
```

## Images and media

Image optimization is typically the largest performance opportunity on content-heavy pages:

- Use responsive images with `srcset`/`sizes` and modern formats (AVIF, WebP) — see the media chapter for details.
- **MUST** set `width` and `height` attributes on `<img>` to prevent CLS.
- Use `loading="lazy"` for below-the-fold images to defer their download until they approach the viewport.
- Use `fetchpriority="high"` on the hero/LCP image to tell the browser it is the most important resource.
- Do NOT lazy-load the LCP image — this delays the most critical visual element.

## Core Web Vitals optimization

### LCP optimization

The LCP element is usually a hero image, background image, or large heading block. To optimize:

- **Reduce TTFB (Time to First Byte)** — Use a CDN (Content Delivery Network), enable server-side caching, and minimize redirects.
- **Preload the LCP resource** — If it is an image, use `<link rel="preload" as="image">`.
- **Eliminate render-blocking resources** — Defer non-critical CSS and JS.
- **Optimize fonts** — Use `font-display: swap` or `font-display: optional` to avoid invisible text while fonts load.

### INP optimization

INP measures how quickly the page responds to user interactions:

- **Defer heavy JavaScript** — Break long tasks (>50ms) into smaller chunks using `requestIdleCallback`, `scheduler.yield()`, or `setTimeout`.
- **Prefer native controls** — Native `<select>`, `<dialog>`, `<details>` elements perform better than JavaScript-heavy custom equivalents.
- **Minimize main-thread work** — Offload heavy computation to Web Workers where feasible.

### CLS optimization

- **Reserve space** for images, ads, and embeds using explicit dimensions or CSS `aspect-ratio`.
- **Avoid injecting content above existing content** — Banners, cookie notices, and dynamically loaded elements should not push page content down.
- **Use `font-display: swap` carefully** — While it prevents invisible text, it can cause layout shift if the web font metrics differ significantly from the fallback. Consider `font-display: optional` or font metric overrides for CLS-sensitive pages.

## Critical rendering path

The critical rendering path is the sequence of steps the browser takes to convert HTML, CSS, and JavaScript into rendered pixels:

1. Parse HTML → build the DOM (Document Object Model).
2. Parse CSS → build the CSSOM (CSS Object Model).
3. Combine DOM + CSSOM → render tree.
4. Layout → calculate element positions and sizes.
5. Paint → render pixels to the screen.

To minimize time to first paint:

- Keep HTML lean — avoid excessive `<head>` bloat and deeply nested DOM structures.
- Minimize the number of render-blocking resources (CSS files, synchronous scripts).
- Serve compressed assets using gzip or Brotli compression.
- Enable HTTP caching with sensible `Cache-Control` headers (`max-age`, `ETag`, `immutable` for hashed assets).

## Diagnostics

Performance monitoring should combine lab testing (controlled conditions) with field data (real users):

- **Lab tools** — [Lighthouse](https://developer.chrome.com/docs/lighthouse/), [WebPageTest](https://www.webpagetest.org/), Chrome DevTools Performance panel.
- **Field data (RUM)** — Real User Monitoring via the [web-vitals](https://github.com/GoogleChrome/web-vitals) JavaScript library, Google Search Console Core Web Vitals report, or third-party RUM (Real User Monitoring) services.
- **CI integration** — Run Lighthouse in CI/CD (Continuous Integration / Continuous Delivery) pipelines to catch performance regressions before deployment.
