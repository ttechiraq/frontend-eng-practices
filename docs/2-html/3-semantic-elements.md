---
sidebar_position: 3
---

# Semantic elements

MUST prefer semantic elements over generic containers to improve accessibility, SEO, and maintainability.

## Landmarks and layout

- Use one `<main>` per page for primary content.
- Use `<header>`, `<nav>`, `<aside>`, and `<footer>` to define regions.
- Avoid nesting multiple `<main>` elements; use `<section>`/`<article>` inside instead.

:::tip[good]
```html
<header>...</header>
<nav aria-label="Primary">...</nav>
<main>
  <article>...</article>
</main>
<footer>...</footer>
```
:::

:::warning[bad]
```html
<div class="top"></div>
<div class="menu"></div>
<div class="content"></div>
<div class="bottom"></div>
```
:::

## Sectioning content

- Use `<article>` for self-contained content that can stand alone (e.g., blog post, card).
- Use `<section>` for thematic grouping within a page or article; include a heading.
- Use `<figure>` and `<figcaption>` for media with a caption.
- Use `<aside>` for tangential content (related links, tips, ads).

## Text-level semantics

- `<strong>` for importance, `<em>` for emphasis.
- `<code>`, `<kbd>`, `<samp>`, `<var>` for inline code and UI text.
- `<time>` with `datetime` for machine-readable dates.
- `<abbr>` with `title` for abbreviations.
- `<mark>` to highlight relevant text in context.

## Headings

- Exactly one `<h1>` per page; descend levels sequentially.
- Each `<section>` or `<article>` SHOULD start with a heading.
- MUST NOT use heading tags purely for sizing; use CSS for presentation.

## Lists and tables

- Use `<ul>`/`<ol>`/`<dl>` for lists; avoid `<br>` for list formatting.
- Use `<table>` only for tabular data. Always include `<thead>`, `<tbody>`, and `<th scope="col|row">` where appropriate.

## Forms preview

Prefer native controls with labels; see the forms chapter for detailed rules.

## Avoiding div/span soup

- `<div>` is acceptable for layout primitives (grid/flex wrappers) when no semantic element fits.
- `<span>` is for inline grouping without inherent meaning.
- When an element conveys structure or meaning, choose the semantic element instead.
