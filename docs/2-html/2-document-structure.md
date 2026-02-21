---
sidebar_position: 2
---

# Document structure

MUST produce valid, semantic HTML documents that declare encoding, language, and responsive behavior up front.

## Minimal document skeleton

Use a consistent starting point:

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

Key requirements:
- **DOCTYPE** MUST be `<!DOCTYPE html>` to enable standards mode.
- **Language** MUST be set on `<html>` (`lang="en"` or project locale).
- **Charset** MUST be UTF-8 declared first in `<head>`.
- **Viewport** MUST include `width=device-width, initial-scale=1` for responsive layouts.
- **Title** MUST be present, unique per page.

## Head ordering

In `<head>`, prefer this order:
1) Charset
2) Viewport
3) Title
4) Description and other meta tags
5) Canonical and preconnect/preload links (see performance/SEO sections)
6) Stylesheets
7) Scripts that must be in head (rare; otherwise `defer`)

## Body landmarks

Wrap the primary content in `<main>` (one per page). Use `<header>` and `<footer>` for page-level regions; nest section-level headers/footers inside articles/sections as needed.

:::warning[bad]
```html
<body>
  <div class="container">
    <div class="content">...</div>
  </div>
</body>
```
:::

:::tip[good]
```html
<body>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
</body>
```
:::

## Heading outline

Use exactly one `<h1>` per page for the top-level heading. Nest subsections with descending levels (`h2`, `h3`, ...). Do not skip levels.

- **SHOULD** keep headings meaningful and concise.
- **MUST NOT** use headings for styling; use CSS instead.

## Source order and layout

- **SHOULD** ensure DOM order matches reading order. CSS (Flex/Grid) should not reorder core reading flows that impact accessibility.
- **MUST** avoid layout shifts by providing dimensions for media (see media and images).

## Internationalization

- **SHOULD** set `dir` on `<html>` when rendering right-to-left languages.
- Use `lang` on elements when mixing languages inside a document.
