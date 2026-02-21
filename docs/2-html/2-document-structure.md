---
sidebar_position: 2
---

# Document structure

Every HTML document MUST be valid, declare its encoding and language, and opt in to responsive behavior. A well-structured document improves accessibility for assistive technologies, enables correct rendering across browsers, and provides a foundation for SEO (Search Engine Optimization).

## Minimal document skeleton

Every page starts from this baseline:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Page title</title>
  </head>
  <body>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </body>
</html>
```

### DOCTYPE

`<!DOCTYPE html>` MUST be the first line of every HTML file. It tells the browser to render the page in **standards mode** rather than quirks mode. Without it, browsers fall back to legacy rendering behavior that produces inconsistent results across engines.

### Language

The `lang` attribute MUST be set on the `<html>` element (e.g., `lang="en"`, `lang="es"`, `lang="fr"`). This allows screen readers to choose the correct pronunciation rules and helps search engines understand the page language. Set it to your project's primary locale.

### Charset

The charset declaration MUST be `UTF-8` and MUST appear as the first element inside `<head>`. UTF-8 (Unicode Transformation Format — 8-bit) supports virtually all characters and symbols across languages. Placing it first ensures the browser knows the encoding before parsing any other content.

### Viewport

The viewport meta tag MUST include `width=device-width, initial-scale=1`. This instructs mobile browsers to set the viewport width to the device's screen width and start at 1x zoom. Without it, mobile browsers render at a desktop-sized viewport and shrink the result, making text unreadable and touch targets too small.

### Title

The `<title>` element MUST be present and unique per page. It appears in the browser tab, bookmarks, search engine results, and is announced by screen readers when a page loads. Keep titles concise and descriptive — under ~60 characters to avoid truncation in search results.

## Head ordering

Elements inside `<head>` SHOULD follow this order for maximum effectiveness:

1. **Charset** — Must come first so the browser decodes all subsequent content correctly.
2. **Viewport** — Establishes responsive behavior before any rendering begins.
3. **Title** — Required for all pages; appears in browser UI immediately.
4. **Description and other meta tags** — SEO and social sharing metadata.
5. **Canonical and resource hint links** — `<link rel="canonical">`, `preconnect`, `preload` (see the performance and SEO chapters).
6. **Stylesheets** — CSS files the browser needs for rendering.
7. **Scripts** — Rare in `<head>`; use `defer` to avoid blocking the HTML parser.

This ordering ensures the browser has critical information (encoding, viewport, title) before it encounters render-blocking resources.

## Body landmarks

The `<body>` SHOULD be organized with semantic landmark elements that assistive technologies use to build a navigable page outline:

- `<header>` — Page-level banner, typically containing the site logo and primary navigation.
- `<main>` — The primary content area. There MUST be exactly one `<main>` per page.
- `<footer>` — Page-level footer for legal, contact, and secondary navigation links.

Nested `<header>` and `<footer>` elements inside `<article>` or `<section>` are valid and represent section-level regions.

:::warning[bad]

```html
<body>
  <div class="header">...</div>
  <div class="container">
    <div class="content">...</div>
  </div>
  <div class="footer">...</div>
</body>
```

:::

:::tip[good]

```html
<body>
  <header>
    <nav aria-label="Primary">...</nav>
  </header>
  <main>
    <article>...</article>
  </main>
  <footer>
    <nav aria-label="Footer">...</nav>
  </footer>
</body>
```

:::

## Heading outline

Headings (`<h1>` through `<h6>`) create the document outline that screen readers and search engines use to understand page structure.

- Use exactly one `<h1>` per page for the top-level heading.
- Descend levels sequentially — do not skip from `<h1>` to `<h3>`.
- Each `<section>` or `<article>` SHOULD begin with a heading at the appropriate level.
- **MUST NOT** use heading tags for visual sizing; use CSS (Cascading Style Sheets) for presentation.
- **SHOULD** keep headings meaningful and concise. They serve as a scannable table of contents for assistive technology users.

:::warning[bad]

```html
<h1>Welcome</h1>
<h3>Our Services</h3>  <!-- Skipped h2 -->
<h5>Contact Us</h5>    <!-- Skipped h4 -->
```

:::

:::tip[good]

```html
<h1>Welcome</h1>
<h2>Our Services</h2>
<h3>Web Development</h3>
<h3>Mobile Apps</h3>
<h2>Contact Us</h2>
```

:::

## Source order and layout

The DOM (Document Object Model) order determines the reading order for screen readers and the tab order for keyboard navigation:

- **SHOULD** ensure DOM order matches the intended reading order. When CSS Flexbox or Grid reorders elements visually, assistive technologies still follow the DOM order, which can create confusing mismatches.
- **MUST** provide explicit `width` and `height` attributes on images and embedded content to prevent CLS (Cumulative Layout Shift) — unexpected visual movement as resources load.

## Internationalization

- **SHOULD** set `dir="rtl"` on `<html>` when rendering right-to-left (RTL) languages such as Arabic or Hebrew. This affects text direction, layout mirroring, and punctuation placement.
- Use `lang` on individual elements when mixing languages within a document. For example, a French quote inside an English page should be wrapped with `lang="fr"` so screen readers switch pronunciation.

```html
<html lang="en" dir="ltr">
  <body>
    <p>The French word <span lang="fr">bonjour</span> means hello.</p>
  </body>
</html>
```
